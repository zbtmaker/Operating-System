# Spring Bean初始化- BeanPostProcessor



BeanPostPorcessor

```java
public interface BeanPostProcessor {

   /**
    * 这个方法在什么时候被调用呢？这个方法在populateBean方法设置属性之后会调用这个方法。
    * 如果这个方法返回null ,那么后面的BeanPostProcessor就不会触发了。
    */
   @Nullable
   default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
      return bean;
   }

   /**
    * 在任何 bean 初始化回调（如 InitializingBean 的 afterPropertiesSet 或自定义 init 方法）之			* 后，将此 BeanPostProcessor 应用于给定的新 bean 实例。 bean 将已填充属性值。返回的 bean 实例		* 可能是原始的包装器。在使用 FactoryBean 的情况下，将为 FactoryBean 实例和由 FactoryBean 创建
    * 的对象（从 Spring 2.0 开始）调用此回调。后处理器可以通过相应的 bean instanceof FactoryBean
    * 检查来决定是否应用到 FactoryBean 或创建的对象或两者。与所有其他 BeanPostProcessor 回调相比，
    * 此回调也将在由 InstantiationAwareBeanPostProcessor.postProcessBeforeInstantiation 方法
    * 触发的短路后调用。
    */
   @Nullable
   default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
      return bean;
   }

}
```



InstantiationAwareBeanPostProcessor

```java
public interface InstantiationAwareBeanPostProcessor extends BeanPostProcessor {

   /**
    * 在目标 bean 被实例化之前应用这个 BeanPostProcessor。返回的 bean 对象可能是要使用的代理而不是目标
    * bean，从而有效地抑制了目标 bean 的默认实例化。如果这个方法返回了一个非空的对象，那么bean的创建过程就
    * 会短路。唯一应用的进一步处理是来自配置的 BeanPostProcessors 的 postProcessAfterInitialization
    * 回调。此回调将应用于 bean 定义及其 bean 类，以及工厂方法定义，在这种情况下，返回的 bean 类型将在此处
    * 传递。后处理器可以实现扩展的 SmartInstantiationAwareBeanPostProcessor 接口，以预测它们将在此处
    * 返回的 bean 对象的类型。默认实现返回 null。
    */
   @Nullable
   default Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
      return null;
   }

   /**
    * 在通过构造函数或工厂方法实例化 bean 之后，但在 Spring 属性填充（来自显式属性或自动装配）发生之前执行
    * 操作。这是在给定 bean 实例上执行自定义字段注入的理想回调，就在 Spring 的自动装配开始之前。默认返回
    * true
    */
   default boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
      return true;
   }

   /**
    * 在工厂将给定的属性值应用到给定的 bean 之前对其进行后处理，而不需要任何属性描述符。如果实现提供自定义
    * postProcessPropertyValues 实现，则应返回 null（默认值），否则应返回 pvs。在此接口的未来版本中
    *（删除了 postProcessPropertyValues），默认实现将直接返回给定的 pvs。
    */
   @Nullable
   default PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName)
         throws BeansException {

      return null;
   }

   /**
    * 在工厂将给定的属性值应用于给定的 bean 之前对其进行后处理。允许检查是否所有依赖项都已满足，例如基于
    * bean 属性设置器上的“Required”注释。还允许替换要应用的属性值，通常通过基于原始 PropertyValues 创建
    * 新的MutablePropertyValues 实例，添加或删除特定值。默认实现按原样返回给定的 pv。
    */
   @Deprecated
   @Nullable
   default PropertyValues postProcessPropertyValues(
         PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName) throws BeansException {

      return pvs;
   }

}
```





AbstractAutoProxyCreator

```java
public abstract class AbstractAutoProxyCreator extends ProxyProcessorSupport implements SmartInstantiationAwareBeanPostProcessor, BeanFactoryAware {
    @Nullable
    protected static final Object[] DO_NOT_PROXY = null;
    protected static final Object[] PROXY_WITHOUT_ADDITIONAL_INTERCEPTORS = new Object[0];
    protected final Log logger = LogFactory.getLog(this.getClass());
    private AdvisorAdapterRegistry advisorAdapterRegistry = GlobalAdvisorAdapterRegistry.getInstance();
    private boolean freezeProxy = false;
    private String[] interceptorNames = new String[0];
    private boolean applyCommonInterceptorsFirst = true;
    @Nullable
    private TargetSourceCreator[] customTargetSourceCreators;
    @Nullable
    private BeanFactory beanFactory;
    private final Set<String> targetSourcedBeans = Collections.newSetFromMap(new ConcurrentHashMap(16));
    private final Map<Object, Object> earlyProxyReferences = new ConcurrentHashMap(16);
    private final Map<Object, Class<?>> proxyTypes = new ConcurrentHashMap(16);
    private final Map<Object, Boolean> advisedBeans = new ConcurrentHashMap(256);

    public AbstractAutoProxyCreator() {
    }

    public void setFrozen(boolean frozen) {
        this.freezeProxy = frozen;
    }

    public boolean isFrozen() {
        return this.freezeProxy;
    }

    public void setAdvisorAdapterRegistry(AdvisorAdapterRegistry advisorAdapterRegistry) {
        this.advisorAdapterRegistry = advisorAdapterRegistry;
    }

    public void setCustomTargetSourceCreators(TargetSourceCreator... targetSourceCreators) {
        this.customTargetSourceCreators = targetSourceCreators;
    }

    public void setInterceptorNames(String... interceptorNames) {
        this.interceptorNames = interceptorNames;
    }

    public void setApplyCommonInterceptorsFirst(boolean applyCommonInterceptorsFirst) {
        this.applyCommonInterceptorsFirst = applyCommonInterceptorsFirst;
    }

    public void setBeanFactory(BeanFactory beanFactory) {
        this.beanFactory = beanFactory;
    }

    @Nullable
    protected BeanFactory getBeanFactory() {
        return this.beanFactory;
    }

    @Nullable
    public Class<?> predictBeanType(Class<?> beanClass, String beanName) {
        if (this.proxyTypes.isEmpty()) {
            return null;
        } else {
            Object cacheKey = this.getCacheKey(beanClass, beanName);
            return (Class)this.proxyTypes.get(cacheKey);
        }
    }

    @Nullable
    public Constructor<?>[] determineCandidateConstructors(Class<?> beanClass, String beanName) {
        return null;
    }

    public Object getEarlyBeanReference(Object bean, String beanName) {
        Object cacheKey = this.getCacheKey(bean.getClass(), beanName);
        this.earlyProxyReferences.put(cacheKey, bean);
        return this.wrapIfNecessary(bean, beanName, cacheKey);
    }

    public Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) {
        Object cacheKey = this.getCacheKey(beanClass, beanName);
        if (!StringUtils.hasLength(beanName) || !this.targetSourcedBeans.contains(beanName)) {
            if (this.advisedBeans.containsKey(cacheKey)) {
                return null;
            }

            if (this.isInfrastructureClass(beanClass) || this.shouldSkip(beanClass, beanName)) {
                this.advisedBeans.put(cacheKey, Boolean.FALSE);
                return null;
            }
        }

        TargetSource targetSource = this.getCustomTargetSource(beanClass, beanName);
        if (targetSource != null) {
            if (StringUtils.hasLength(beanName)) {
                this.targetSourcedBeans.add(beanName);
            }

            Object[] specificInterceptors = this.getAdvicesAndAdvisorsForBean(beanClass, beanName, targetSource);
            Object proxy = this.createProxy(beanClass, beanName, specificInterceptors, targetSource);
            this.proxyTypes.put(cacheKey, proxy.getClass());
            return proxy;
        } else {
            return null;
        }
    }

    public PropertyValues postProcessProperties(PropertyValues pvs, Object bean, String beanName) {
        return pvs;
    }

    public Object postProcessAfterInitialization(@Nullable Object bean, String beanName) {
        if (bean != null) {
            Object cacheKey = this.getCacheKey(bean.getClass(), beanName);
            if (this.earlyProxyReferences.remove(cacheKey) != bean) {
                return this.wrapIfNecessary(bean, beanName, cacheKey);
            }
        }

        return bean;
    }

    protected Object getCacheKey(Class<?> beanClass, @Nullable String beanName) {
        if (StringUtils.hasLength(beanName)) {
            return FactoryBean.class.isAssignableFrom(beanClass) ? "&" + beanName : beanName;
        } else {
            return beanClass;
        }
    }

    protected Object wrapIfNecessary(Object bean, String beanName, Object cacheKey) {
        if (StringUtils.hasLength(beanName) && this.targetSourcedBeans.contains(beanName)) {
            return bean;
        } else if (Boolean.FALSE.equals(this.advisedBeans.get(cacheKey))) {
            return bean;
        } else if (!this.isInfrastructureClass(bean.getClass()) && !this.shouldSkip(bean.getClass(), beanName)) {
            Object[] specificInterceptors = this.getAdvicesAndAdvisorsForBean(bean.getClass(), beanName, (TargetSource)null);
            if (specificInterceptors != DO_NOT_PROXY) {
                this.advisedBeans.put(cacheKey, Boolean.TRUE);
                Object proxy = this.createProxy(bean.getClass(), beanName, specificInterceptors, new SingletonTargetSource(bean));
                this.proxyTypes.put(cacheKey, proxy.getClass());
                return proxy;
            } else {
                this.advisedBeans.put(cacheKey, Boolean.FALSE);
                return bean;
            }
        } else {
            this.advisedBeans.put(cacheKey, Boolean.FALSE);
            return bean;
        }
    }

    protected boolean isInfrastructureClass(Class<?> beanClass) {
        boolean retVal = Advice.class.isAssignableFrom(beanClass) || Pointcut.class.isAssignableFrom(beanClass) || Advisor.class.isAssignableFrom(beanClass) || AopInfrastructureBean.class.isAssignableFrom(beanClass);
        if (retVal && this.logger.isTraceEnabled()) {
            this.logger.trace("Did not attempt to auto-proxy infrastructure class [" + beanClass.getName() + "]");
        }

        return retVal;
    }

    protected boolean shouldSkip(Class<?> beanClass, String beanName) {
        return AutoProxyUtils.isOriginalInstance(beanName, beanClass);
    }

    @Nullable
    protected TargetSource getCustomTargetSource(Class<?> beanClass, String beanName) {
        if (this.customTargetSourceCreators != null && this.beanFactory != null && this.beanFactory.containsBean(beanName)) {
            TargetSourceCreator[] var3 = this.customTargetSourceCreators;
            int var4 = var3.length;

            for(int var5 = 0; var5 < var4; ++var5) {
                TargetSourceCreator tsc = var3[var5];
                TargetSource ts = tsc.getTargetSource(beanClass, beanName);
                if (ts != null) {
                    if (this.logger.isTraceEnabled()) {
                        this.logger.trace("TargetSourceCreator [" + tsc + "] found custom TargetSource for bean with name '" + beanName + "'");
                    }

                    return ts;
                }
            }
        }

        return null;
    }

    protected Object createProxy(Class<?> beanClass, @Nullable String beanName, @Nullable Object[] specificInterceptors, TargetSource targetSource) {
        if (this.beanFactory instanceof ConfigurableListableBeanFactory) {
            AutoProxyUtils.exposeTargetClass((ConfigurableListableBeanFactory)this.beanFactory, beanName, beanClass);
        }

        ProxyFactory proxyFactory = new ProxyFactory();
        proxyFactory.copyFrom(this);
        if (proxyFactory.isProxyTargetClass()) {
            if (Proxy.isProxyClass(beanClass)) {
                Class[] var6 = beanClass.getInterfaces();
                int var7 = var6.length;

                for(int var8 = 0; var8 < var7; ++var8) {
                    Class<?> ifc = var6[var8];
                    proxyFactory.addInterface(ifc);
                }
            }
        } else if (this.shouldProxyTargetClass(beanClass, beanName)) {
            proxyFactory.setProxyTargetClass(true);
        } else {
            this.evaluateProxyInterfaces(beanClass, proxyFactory);
        }

        Advisor[] advisors = this.buildAdvisors(beanName, specificInterceptors);
        proxyFactory.addAdvisors(advisors);
        proxyFactory.setTargetSource(targetSource);
        this.customizeProxyFactory(proxyFactory);
        proxyFactory.setFrozen(this.freezeProxy);
        if (this.advisorsPreFiltered()) {
            proxyFactory.setPreFiltered(true);
        }

        ClassLoader classLoader = this.getProxyClassLoader();
        if (classLoader instanceof SmartClassLoader && classLoader != beanClass.getClassLoader()) {
            classLoader = ((SmartClassLoader)classLoader).getOriginalClassLoader();
        }

        return proxyFactory.getProxy(classLoader);
    }

    protected boolean shouldProxyTargetClass(Class<?> beanClass, @Nullable String beanName) {
        return this.beanFactory instanceof ConfigurableListableBeanFactory && AutoProxyUtils.shouldProxyTargetClass((ConfigurableListableBeanFactory)this.beanFactory, beanName);
    }

    protected boolean advisorsPreFiltered() {
        return false;
    }

    protected Advisor[] buildAdvisors(@Nullable String beanName, @Nullable Object[] specificInterceptors) {
        Advisor[] commonInterceptors = this.resolveInterceptorNames();
        List<Object> allInterceptors = new ArrayList();
        if (specificInterceptors != null) {
            if (specificInterceptors.length > 0) {
                allInterceptors.addAll(Arrays.asList(specificInterceptors));
            }

            if (commonInterceptors.length > 0) {
                if (this.applyCommonInterceptorsFirst) {
                    allInterceptors.addAll(0, Arrays.asList(commonInterceptors));
                } else {
                    allInterceptors.addAll(Arrays.asList(commonInterceptors));
                }
            }
        }

        int i;
        if (this.logger.isTraceEnabled()) {
            int nrOfCommonInterceptors = commonInterceptors.length;
            i = specificInterceptors != null ? specificInterceptors.length : 0;
            this.logger.trace("Creating implicit proxy for bean '" + beanName + "' with " + nrOfCommonInterceptors + " common interceptors and " + i + " specific interceptors");
        }

        Advisor[] advisors = new Advisor[allInterceptors.size()];

        for(i = 0; i < allInterceptors.size(); ++i) {
            advisors[i] = this.advisorAdapterRegistry.wrap(allInterceptors.get(i));
        }

        return advisors;
    }

    private Advisor[] resolveInterceptorNames() {
        BeanFactory bf = this.beanFactory;
        ConfigurableBeanFactory cbf = bf instanceof ConfigurableBeanFactory ? (ConfigurableBeanFactory)bf : null;
        List<Advisor> advisors = new ArrayList();
        String[] var4 = this.interceptorNames;
        int var5 = var4.length;

        for(int var6 = 0; var6 < var5; ++var6) {
            String beanName = var4[var6];
            if (cbf == null || !cbf.isCurrentlyInCreation(beanName)) {
                Assert.state(bf != null, "BeanFactory required for resolving interceptor names");
                Object next = bf.getBean(beanName);
                advisors.add(this.advisorAdapterRegistry.wrap(next));
            }
        }

        return (Advisor[])advisors.toArray(new Advisor[0]);
    }

    protected void customizeProxyFactory(ProxyFactory proxyFactory) {
    }

    @Nullable
    protected abstract Object[] getAdvicesAndAdvisorsForBean(Class<?> beanClass, String beanName, @Nullable TargetSource customTargetSource) throws BeansException;
}
```