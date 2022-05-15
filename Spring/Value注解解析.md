# @Value注解解析
## Value注解源码
之前在看BeanFactoryPostProcessor接口的实现类的时候，一直以为这个接口的主要目的是在Bean初始化之前修改Bean的定义，也就是修改类还未初始化之前的BeanDefinition。
因为@Value注解可以通过占位符的方式将配置文件的值替换占位符，而正好之前的PropertySourcesPlaceholderConfigurer是负责XML文件中的占位符替换问题，所以在阅读源码时自然而然会认为@value注解也是在BeanFactoryPostProcessor的postProcessBeanFactory中将占位符替换成实际的值。但是debug代码之后返现完全不是。可以看一下Value注解的源码。
```java
/**
 * Annotation used at the field or method/constructor parameter level that indicates a default value expression for the annotated element.
Typically used for expression-driven or property-driven dependency injection. Also supported for dynamic resolution of handler method arguments — for example, in Spring MVC.
A common use case is to inject values using #{systemProperties.myProp} style SpEL (Spring Expression Language) expressions. Alternatively, values may be injected using ${my.app.myProp} style property placeholders.
Note that actual processing of the @Value annotation is performed by a BeanPostProcessor which in turn means that you cannot use @Value within BeanPostProcessor or BeanFactoryPostProcessor types. Please consult the javadoc for the AutowiredAnnotationBeanPostProcessor class (which, by default, checks for the presence of this annotation).
 */
@Target({ElementType.FIELD, ElementType.METHOD, ElementType.PARAMETER, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Value {

	/**
	 * The actual value expression such as <code>#{systemProperties.myProp}</code>
	 * or property placeholder such as <code>${my.app.myProp}</code>.
	 */
	String value();

}
```
@Value注解的源码发现这个注解是由BeanPostProcessor借口中的实现类AutowiredAnnotationBeanPostProcessor处理的。此外，@Value注解的源码中也提醒使用者不要在在任何实现了BeanPostProcessor、BeanFactoryPostProcessor中使用@Value注解。
## AutowiredAnnotationBeanPostProcessor
这里我只是将与Value注解相关的源码贴出来，
```java

	@Override
	public void postProcessMergedBeanDefinition(RootBeanDefinition beanDefinition, Class<?> beanType, String beanName) {
        // 这个方法是在getBean->doCreateBean->populateBean之后
		InjectionMetadata metadata = findAutowiringMetadata(beanName, beanType, null);
		metadata.checkConfigMembers(beanDefinition);
	}

    /**
     *
     */
    private InjectionMetadata findAutowiringMetadata(String beanName, Class<?> clazz, @Nullable PropertyValues pvs) {
		// Fall back to class name as cache key, for backwards compatibility with custom callers.
		String cacheKey = (StringUtils.hasLength(beanName) ? beanName : clazz.getName());
		// Quick check on the concurrent map first, with minimal locking.
        // 从缓存中获取已经解析了@ Value注解和@Autowire注解需要
		InjectionMetadata metadata = this.injectionMetadataCache.get(cacheKey);
		if (InjectionMetadata.needsRefresh(metadata, clazz)) {
			synchronized (this.injectionMetadataCache) {
				metadata = this.injectionMetadataCache.get(cacheKey);
				if (InjectionMetadata.needsRefresh(metadata, clazz)) {
					if (metadata != null) {
						metadata.clear(pvs);
					}
                    // 具体的解析工作交给了buildAutowirngMetadata方法
					metadata = buildAutowiringMetadata(clazz);
					this.injectionMetadataCache.put(cacheKey, metadata);
				}
			}
		}
		return metadata;
	}

	private InjectionMetadata buildAutowiringMetadata(Class<?> clazz) {
        // 类在初始化的时候
		if (!AnnotationUtils.isCandidateClass(clazz, this.autowiredAnnotationTypes)) {
			return InjectionMetadata.EMPTY;
		}

		List<InjectionMetadata.InjectedElement> elements = new ArrayList<>();
		Class<?> targetClass = clazz;

		do {
			final List<InjectionMetadata.InjectedElement> currElements = new ArrayList<>();

			ReflectionUtils.doWithLocalFields(targetClass, field -> {
				MergedAnnotation<?> ann = findAutowiredAnnotation(field);
				if (ann != null) {
					if (Modifier.isStatic(field.getModifiers())) {
						if (logger.isInfoEnabled()) {
							logger.info("Autowired annotation is not supported on static fields: " + field);
						}
						return;
					}
					boolean required = determineRequiredStatus(ann);
					currElements.add(new AutowiredFieldElement(field, required));
				}
			});

			ReflectionUtils.doWithLocalMethods(targetClass, method -> {
				Method bridgedMethod = BridgeMethodResolver.findBridgedMethod(method);
				if (!BridgeMethodResolver.isVisibilityBridgeMethodPair(method, bridgedMethod)) {
					return;
				}
				MergedAnnotation<?> ann = findAutowiredAnnotation(bridgedMethod);
				if (ann != null && method.equals(ClassUtils.getMostSpecificMethod(method, clazz))) {
					if (Modifier.isStatic(method.getModifiers())) {
						if (logger.isInfoEnabled()) {
							logger.info("Autowired annotation is not supported on static methods: " + method);
						}
						return;
					}
					if (method.getParameterCount() == 0) {
						if (logger.isInfoEnabled()) {
							logger.info("Autowired annotation should only be used on methods with parameters: " +
									method);
						}
					}
					boolean required = determineRequiredStatus(ann);
					PropertyDescriptor pd = BeanUtils.findPropertyForMethod(bridgedMethod, clazz);
					currElements.add(new AutowiredMethodElement(method, required, pd));
				}
			});

			elements.addAll(0, currElements);
			targetClass = targetClass.getSuperclass();
		}
		while (targetClass != null && targetClass != Object.class);

		return InjectionMetadata.forElements(elements, clazz);
	}
```