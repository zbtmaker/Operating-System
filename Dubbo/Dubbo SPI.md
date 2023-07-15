# Dubbo SPI
最近这几天在看Dubbo源码学习的时候发现对与ExtensionLoader这个机制虽然了解了皮毛，但是不了解这个框架的骨骼。
在阅读Dubbo源码的时候总是会看到@SPI和@Adaptive这两个注解，@SPI注解理解起来相对简单很多，因为@SPI注解就是基于Java的SPI机制做了相应的优化。但是@Adaptive这个注解就不是很清楚为什么要这么做了。

连续看了两天的Dubbo SPI机制，这个框架其实不是万能的，但是基于Dubbo的URL，可以说的上是把这个框架变成了一个很灵活的框架。确实很方便自适应扩展。

其实在看懂Dubbo SPI之前，其实还是还要了解JDK自带的SPI机制，Dubbo的 SPI主要分三部分构成，第一部分是
* ExtensionLoader 
* ExtensionLoaderFactory
* @SPI、@Adaptive、@Active三个注解

ExtensionLoader
* 每一个接口都会生成对应的ExtensionLoader，在创建这个类的Extension Loader之前会创建一个ExtensionLoaderFactory；

* 针对@SPI注解，通过调用ExtensionLoader.getDefaultExtension可以获取默认的@SPI注解接口指定的默认实现类，具体流程可以参考@SPI解析；

* 针对@Adaptive注解，通过ExtensionLoader.getAdaptiveExtension方法获取生成一个中间类，生成的类名为{ClassName}$Adaptive，根据URL中的参数获取接口真正的实现类，具体的流程可以参考@Adaptive解析；

* 针对@Active注解 ，通过ExtensionLoader.getActivateExtension获取对应的类集合，具体流程可以参考@Active解析


## ExtensionLoader 
在ExtensionLoader中
```java
public class ExtensionLoaderTest extends TestCase {
    @Test
    public void test_getDefaultExtension() throws Exception {
        SimpleExt ext = ExtensionLoader.getExtensionLoader(SimpleExt.class).getDefaultExtension();
        assertThat(ext, instanceOf(SimpleExtImpl1.class));

        String name = ExtensionLoader.getExtensionLoader(SimpleExt.class).getDefaultExtensionName();
        assertEquals("impl1", name);
    }
}
```
同时看一下这里通过ExtensionLoader类创建的SimpleExt接口以及实现类。
```java
@SPI("impl1")
public interface SimpleExt {
    // @Adaptive example, do not specify a explicit key.
    @Adaptive
    String echo(URL url, String s);

    @Adaptive({"key1", "key2"})
    String yell(URL url, String s);

    // no @Adaptive
    String bang(URL url, int i);
}
/**
 * SimpleExt接口实现类
 **/
public class SimpleExtImpl1 implements SimpleExt {
    public String echo(URL url, String s) {
        return "Ext1Impl1-echo";
    }

    public String yell(URL url, String s) {
        return "Ext1Impl1-yell";
    }

    public String bang(URL url, int i) {
        return "bang1";
    }
}
```
SimpleExt增加了注解@SPI，同时在其方法存在@Adaptive注解。@Adaptive注解在方法上会给这个类带来什么不一样的结果。

```java
    @SuppressWarnings("unchecked")
    public static <T> ExtensionLoader<T> getExtensionLoader(Class<T> type) {
        if (type == null) {
            throw new IllegalArgumentException("Extension type == null");
        }
        // type一定是一个接口
        if (!type.isInterface()) {
            throw new IllegalArgumentException("Extension type (" + type + ") is not an interface!");
        }
        // 接口上面一定要有@SPI注解
        if (!withExtensionAnnotation(type)) {
            throw new IllegalArgumentException("Extension type (" + type +
                    ") is not an extension, because it is NOT annotated with @" + SPI.class.getSimpleName() + "!");
        }
        // 从本次缓存中获取ExtensionLoader
        ExtensionLoader<T> loader = (ExtensionLoader<T>) EXTENSION_LOADERS.get(type);
        if (loader == null) {
            // 创建一个ExtensionLoader
            EXTENSION_LOADERS.putIfAbsent(type, new ExtensionLoader<T>(type));
            loader = (ExtensionLoader<T>) EXTENSION_LOADERS.get(type);
        }
        return loader;
    }
```
这个ExtensionLoader的构造器是一个private类型，我们可以看一下这个构造器
```java
    private ExtensionLoader(Class<?> type) {
        this.type = type;
        // 如果需要创建的类型不是ExtensionFactory，那么就需要先创建ExtensionLoaderFactory的扩展类
        objectFactory = (type == ExtensionFactory.class ? null : ExtensionLoader.getExtensionLoader(ExtensionFactory.class).getAdaptiveExtension());
    }
```
当我们要创建一个非ExtensionLoaderFactory的ExtensionLoader时，会首先创建一个ExtensionLoaderFactory，然后调用ExtensionLoader的getAdaptiveExtension方法。看一下getAdaptivgeExtension方法主要做了什么（**注意此时是ExtensionLoaderFactory的Extension Loader在调用getAdaptiveExtension方法，不要迷惑，因为每一个接口都需要创建自己的ExtensionLoader**）

```java
    public T getAdaptiveExtension() {
        // 从缓存中获取
        Object instance = cachedAdaptiveInstance.get();
        if (instance == null) {
            if (createAdaptiveInstanceError == null) {
                // 加锁
                synchronized (cachedAdaptiveInstance) {
                    // 继续从缓存中获取
                    instance = cachedAdaptiveInstance.get();
                    if (instance == null) {
                        try {
                            // 创建对应的扩展类
                            instance = createAdaptiveExtension();
                            cachedAdaptiveInstance.set(instance);
                        } catch (Throwable t) {
                            createAdaptiveInstanceError = t;
                            throw new IllegalStateException("Failed to create adaptive instance: " + t.toString(), t);
                        }
                    }
                }
            } else {
                throw new IllegalStateException("Failed to create adaptive instance: " + createAdaptiveInstanceError.toString(), createAdaptiveInstanceError);
            }
        }

        return (T) instance;
    }
    private T createAdaptiveExtension() {
        try {
            return injectExtension((T) getAdaptiveExtensionClass().newInstance());
        } catch (Exception e) {
            throw new IllegalStateException("Can't create adaptive extension " + type + ", cause: " + e.getMessage(), e);
        }
    }
    private Class<?> getAdaptiveExtensionClass() {
        getExtensionClasses();
        // 这里cachedAdaptiveClass一定不为空，因为META-INFO/dubbo/internal目录下存在AdaptiveExtensionLoaderFactory映射关系
        if (cachedAdaptiveClass != null) {
            return cachedAdaptiveClass;
        }
        // 在加载ExtensionLoaderFactory类的扩展类时因为AdaptiveExtensionLoaderFactory类有@Adaptive注解，所以在上面就会返回，不会走到这个逻辑。在加载其他类的时候如果接口实现类上面没有@Adaptive注解，那么就会执行这一块逻辑
        return cachedAdaptiveClass = createAdaptiveExtensionClass();
    }
```
这里因为是ExtensionLoaderFactory的ExtensionLoader，所以会先从以下四个目录中加载ExtenionLoaderFactory的扩展类。
* META-INF/dubbo/internal/org.apache.dubbo.common.extension.ExtensionLoaderFactory
* META-INF/dubbo/internal/com.alibaba.dubbo.common.extension.ExtensionLoaderFactory
* META-INF/dubbo/org.apache.dubbo.common.extension.ExtensionLoaderFactory
* META-INF/dubbo/com.alibaba.dubbo.common.extension.ExtensionLoaderFactory
* META-INF/services/org.apache.dubbo.common.extension.ExtensionLoaderFactory
* META-INF/services/internal/com.alibaba.dubbo.common.extension.ExtensionLoaderFactory

ExtensionLoaderFactory的扩展类主要存储在META-INF/dubbo/internal/com.alibaba.dubbo.common.extension.ExtensionLoaderFactory
```bash
adaptive=org.apache.dubbo.common.extension.factory.AdaptiveExtensionFactory
spi=org.apache.dubbo.common.extension.factory.SpiExtensionFactory
```
我们看到ExtensionLoaderFactory类主要有adaptive和spi两个扩展类，而且AdaptiveExtensionLoaderFactory这个类上面有@Adaptive注解。
```java
    private Map<String, Class<?>> getExtensionClasses() {
        Map<String, Class<?>> classes = cachedClasses.get();
        if (classes == null) {
            synchronized (cachedClasses) {
                classes = cachedClasses.get();
                if (classes == null) {
                    classes = loadExtensionClasses();
                    cachedClasses.set(classes);
                }
            }
        }
        return classes;
    }

    // synchronized in getExtensionClasses
    private Map<String, Class<?>> loadExtensionClasses() {
        cacheDefaultExtensionName();

        Map<String, Class<?>> extensionClasses = new HashMap<>();
        // 加载META-INF/internal/org.apache.*
        loadDirectory(extensionClasses, DUBBO_INTERNAL_DIRECTORY, type.getName());
        // 加载META-INFO/internal/com.alibaba.*
        loadDirectory(extensionClasses, DUBBO_INTERNAL_DIRECTORY, type.getName().replace("org.apache", "com.alibaba"));
        // 加载META-INFO/dubbo/org.apache.*
        loadDirectory(extensionClasses, DUBBO_DIRECTORY, type.getName());
        // 加载META-INFO/dubbo/com.alibaba.*
        loadDirectory(extensionClasses, DUBBO_DIRECTORY, type.getName().replace("org.apache", "com.alibaba"));
        // 加载META-INFO/services/org.apache.*
        loadDirectory(extensionClasses, SERVICES_DIRECTORY, type.getName());
        // 加载META-INFO/services/com.alibaba.*
        loadDirectory(extensionClasses, SERVICES_DIRECTORY, type.getName().replace("org.apache", "com.alibaba"));
        return extensionClasses;
    }

    private void loadDirectory(Map<String, Class<?>> extensionClasses, String dir, String type) {
        String fileName = dir + type;
        try {
            Enumeration<java.net.URL> urls;
            ClassLoader classLoader = findClassLoader();
            if (classLoader != null) {
                urls = classLoader.getResources(fileName);
            } else {
                urls = ClassLoader.getSystemResources(fileName);
            }
            if (urls != null) {
                while (urls.hasMoreElements()) {
                    java.net.URL resourceURL = urls.nextElement();
                    loadResource(extensionClasses, classLoader, resourceURL);
                }
            }
        } catch (Throwable t) {
            logger.error("Exception occurred when loading extension class (interface: " +
                    type + ", description file: " + fileName + ").", t);
        }
    }
    private void loadResource(Map<String, Class<?>> extensionClasses, ClassLoader classLoader, java.net.URL resourceURL) {
        try {
            try (BufferedReader reader = new BufferedReader(new InputStreamReader(resourceURL.openStream(), StandardCharsets.UTF_8))) {
                String line;
                while ((line = reader.readLine()) != null) {
                    final int ci = line.indexOf('#');
                    if (ci >= 0) {
                        line = line.substring(0, ci);
                    }
                    line = line.trim();
                    if (line.length() > 0) {
                        try {
                            String name = null;
                            int i = line.indexOf('=');
                            if (i > 0) {
                                name = line.substring(0, i).trim();
                                line = line.substring(i + 1).trim();
                            }
                            if (line.length() > 0) {
                                loadClass(extensionClasses, resourceURL, Class.forName(line, true, classLoader), name);
                            }
                        } catch (Throwable t) {
                            IllegalStateException e = new IllegalStateException("Failed to load extension class (interface: " + type + ", class line: " + line + ") in " + resourceURL + ", cause: " + t.getMessage(), t);
                            exceptions.put(line, e);
                        }
                    }
                }
            }
        } catch (Throwable t) {
            logger.error("Exception occurred when loading extension class (interface: " +
                    type + ", class file: " + resourceURL + ") in " + resourceURL, t);
        }
    }
    private void loadClass(Map<String, Class<?>> extensionClasses, java.net.URL resourceURL, Class<?> clazz, String name) throws NoSuchMethodException {
        if (!type.isAssignableFrom(clazz)) {
            throw new IllegalStateException("Error occurred when loading extension class (interface: " +
                    type + ", class line: " + clazz.getName() + "), class "
                    + clazz.getName() + " is not subtype of interface.");
        }
        if (clazz.isAnnotationPresent(Adaptive.class)) {
            cacheAdaptiveClass(clazz);
        } else if (isWrapperClass(clazz)) {
            cacheWrapperClass(clazz);
        } else {
            clazz.getConstructor();
            if (StringUtils.isEmpty(name)) {
                name = findAnnotationName(clazz);
                if (name.length() == 0) {
                    throw new IllegalStateException("No such extension name for the class " + clazz.getName() + " in the config " + resourceURL);
                }
            }

            String[] names = NAME_SEPARATOR.split(name);
            if (ArrayUtils.isNotEmpty(names)) {
                cacheActivateClass(clazz, names[0]);
                for (String n : names) {
                    cacheName(clazz, n);
                    saveInExtensionClass(extensionClasses, clazz, n);
                }
            }
        }
    }
```
上面一大串逻辑执行完成以后，就将ExtensionLoaderFactory的实现类从文件中加载到JVM了，因为Adaptive ExtensionLoaderFactory类上面增加了@Adaptive注解，因此getAdaptiveExtensionClass方法返回的Class就是AdaptiveExtensionLoaderFactory，同时会执行newInstacnce方法来构造对象，创建的对象就是AdaptiveExtensionLoaderFactory对象。
```java
    private T createAdaptiveExtension() {
        try {
            return injectExtension((T) getAdaptiveExtensionClass().newInstance());
        } catch (Exception e) {
            throw new IllegalStateException("Can't create adaptive extension " + type + ", cause: " + e.getMessage(), e);
        }
    }
    private Class<?> getAdaptiveExtensionClass() {
        getExtensionClasses();
        // 这里cachedAdaptiveClass一定不为空，因为META-INFO/dubbo/internal目录下存在AdaptiveExtensionLoaderFactory映射关系
        if (cachedAdaptiveClass != null) {
            return cachedAdaptiveClass;
        }
        // 在加载ExtensionLoaderFactory类的扩展类时因为AdaptiveExtensionLoaderFactory类有@Adaptive注解，所以在上面就会返回，不会走到这个逻辑。在加载其他类的时候如果接口实现类上面没有@Adaptive注解，那么就会执行这一块逻辑
        return cachedAdaptiveClass = createAdaptiveExtensionClass();
    }
```
我们可以看一下AdaptiveExtensionLoaderFactory类代码
```java
@Adaptive
public class AdaptiveExtensionFactory implements ExtensionFactory {

    private final List<ExtensionFactory> factories;

    public AdaptiveExtensionFactory() {
        // 此时获取的ExtensionLoader就是AdaptiveExtensionLoaderFactory的ExtensionLoader
        ExtensionLoader<ExtensionFactory> loader = ExtensionLoader.getExtensionLoader(ExtensionFactory.class);
        List<ExtensionFactory> list = new ArrayList<ExtensionFactory>();
        for (String name : loader.getSupportedExtensions()) {
            list.add(loader.getExtension(name));
        }
        // factories集合保存的是SpiExtensionLoaderFactory
        factories = Collections.unmodifiableList(list);
    }

    @Override
    public <T> T getExtension(Class<T> type, String name) {
        for (ExtensionFactory factory : factories) {
            T extension = factory.getExtension(type, name);
            if (extension != null) {
                return extension;
            }
        }
        return null;
    }
}
```
上面ExtensionLoaderFactory.getExtensionAdaptiveClass()执行完成之后，其实将文件中的spi ExtensionLoaderFactory保存在AdaptiveExtensionLoaderFactory的factories集合当中。然后再回到非ExtensionLoaderFactory的ExtensionLoader获取如下代码所示：
```java
    private ExtensionLoader(Class<?> type) {
        this.type = type;
        objectFactory = (type == ExtensionFactory.class ? null : ExtensionLoader.getExtensionLoader(ExtensionFactory.class).getAdaptiveExtension());
    }

    @SuppressWarnings("unchecked")
    public static <T> ExtensionLoader<T> getExtensionLoader(Class<T> type) {
        if (type == null) {
            throw new IllegalArgumentException("Extension type == null");
        }
        if (!type.isInterface()) {
            throw new IllegalArgumentException("Extension type (" + type + ") is not an interface!");
        }
        if (!withExtensionAnnotation(type)) {
            throw new IllegalArgumentException("Extension type (" + type +
                    ") is not an extension, because it is NOT annotated with @" + SPI.class.getSimpleName() + "!");
        }

        ExtensionLoader<T> loader = (ExtensionLoader<T>) EXTENSION_LOADERS.get(type);
        if (loader == null) {
            EXTENSION_LOADERS.putIfAbsent(type, new ExtensionLoader<T>(type));
            loader = (ExtensionLoader<T>) EXTENSION_LOADERS.get(type);
        }
        return loader;
    }
```
那么上面第一步其实只是完成了ExtensionLoaderFactory加载，以及普通类的ExtensionLoader类的创建，比方SimpleExt接口的Extension Loader创建好了，但是在properties文件并没有加载。
## 一、@SPI
我们这里仍然使用SimpleExt接口作为测试用例：
```java
@SPI("impl1")
public interface SimpleExt {
    // @Adaptive example, do not specify a explicit key.
    @Adaptive
    String echo(URL url, String s);

    @Adaptive({"key1", "key2"})
    String yell(URL url, String s);

    // no @Adaptive
    String bang(URL url, int i);
}
```
测试用例
```java
    @Test
    public void test_getDefaultExtension() throws Exception {
        SimpleExt ext = ExtensionLoader.getExtensionLoader(SimpleExt.class).getDefaultExtension();
        assertThat(ext, instanceOf(SimpleExtImpl1.class));

        String name = ExtensionLoader.getExtensionLoader(SimpleExt.class).getDefaultExtensionName();
        assertEquals("impl1", name);
    }
```
我们在上面 SimpleExt类的ExtensionLoader获取上讲过，一个类的ExtensionLoader类在初始化之前会初始化ExtensionLoaderFactory类的ExtensionLoader。创建SimpleExt的ExtensionLoader之后，没有将文件中的映射关系添加到JVM。需要调用ExtensionLoader.getDefaultExtension才会加载对应的映射关系并创建对应的实现类。
```java
    public T getDefaultExtension() {
        getExtensionClasses();
        if (StringUtils.isBlank(cachedDefaultName) || "true".equals(cachedDefaultName)) {
            return null;
        }
        return getExtension(cachedDefaultName);
    }

    private Map<String, Class<?>> getExtensionClasses() {
        Map<String, Class<?>> classes = cachedClasses.get();
        if (classes == null) {
            synchronized (cachedClasses) {
                classes = cachedClasses.get();
                if (classes == null) {
                    // 这个在上面加载ExtensionLoaderFactory的扩展类已经讲过，就是从四个文件中读取对应的映射关系
                    classes = loadExtensionClasses();
                    cachedClasses.set(classes);
                }
            }
        }
        return classes;
    }

    public T getExtension(String name) {
        if (StringUtils.isEmpty(name)) {
            throw new IllegalArgumentException("Extension name == null");
        }
        if ("true".equals(name)) {
            return getDefaultExtension();
        }
        final Holder<Object> holder = getOrCreateHolder(name);
        Object instance = holder.get();
        if (instance == null) {
            synchronized (holder) {
                instance = holder.get();
                if (instance == null) {
                    instance = createExtension(name);
                    holder.set(instance);
                }
            }
        }
        return (T) instance;
    }
    private T createExtension(String name) {
        Class<?> clazz = getExtensionClasses().get(name);
        if (clazz == null) {
            throw findException(name);
        }
        try {
            T instance = (T) EXTENSION_INSTANCES.get(clazz);
            if (instance == null) {
                EXTENSION_INSTANCES.putIfAbsent(clazz, clazz.newInstance());
                instance = (T) EXTENSION_INSTANCES.get(clazz);
            }
            injectExtension(instance);
            Set<Class<?>> wrapperClasses = cachedWrapperClasses;
            if (CollectionUtils.isNotEmpty(wrapperClasses)) {
                for (Class<?> wrapperClass : wrapperClasses) {
                    instance = injectExtension((T) wrapperClass.getConstructor(type).newInstance(instance));
                }
            }
            return instance;
        } catch (Throwable t) {
            throw new IllegalStateException("Extension instance (name: " + name + ", class: " +
                    type + ") couldn't be instantiated: " + t.getMessage(), t);
        }
    }

    private T injectExtension(T instance) {
        try {
            if (objectFactory != null) {
                for (Method method : instance.getClass().getMethods()) {
                    if (isSetter(method)) {
                        /**
                         * Check {@link DisableInject} to see if we need auto injection for this property
                         */
                        if (method.getAnnotation(DisableInject.class) != null) {
                            continue;
                        }
                        Class<?> pt = method.getParameterTypes()[0];
                        if (ReflectUtils.isPrimitives(pt)) {
                            continue;
                        }
                        try {
                            String property = getSetterProperty(method);
                            Object object = objectFactory.getExtension(pt, property);
                            if (object != null) {
                                method.invoke(instance, object);
                            }
                        } catch (Exception e) {
                            logger.error("Failed to inject via method " + method.getName()
                                    + " of interface " + type.getName() + ": " + e.getMessage(), e);
                        }
                    }
                }
            }
        } catch (Exception e) {
            logger.error(e.getMessage(), e);
        }
        return instance;
    }  
```



## 二、@Adaptive
看这个注解，首先是通过Dubbo源码的测试类，一步一步Debug的方式来看调用方式，然后再慢慢理解为什么需要这么设计。
Dubbo的SPI机制主要是ExtensionLoader和ExtensionLoaderFactory这两个类来实现。
* ExtensionLoader主要负责加载类，并将创建好的实例缓存在Map当中
* ExtensionLoaderFactory类是ExtensionLoader类的工厂类，那么这个类主要做什么呢？顾名思义Factory类就是为了创建ExtensionLoader服务的。
  
我们首先看一下Dubbo的一个关于ExtensionLoader的测试用例


这里首先创建一个ExtensionLoader\<ExtensionLoaderFactory\>的扩展类，然后会调用ExtensionLoader的getAdaptiveExtension方法

我们接下来看一下createAdaptiveExtension方法，这里首先会调用getExtensionClass方法，首先是从META-INFO/这个目录下加载name对应的Class，然后进行初始化，
```java
    private T createAdaptiveExtension() {
        try {
            return injectExtension((T) getAdaptiveExtensionClass().newInstance());
        } catch (Exception e) {
            throw new IllegalStateException("Can't create adaptive extension " + type + ", cause: " + e.getMessage(), e);
        }
    }
    private Class<?> getAdaptiveExtensionClass() {
        getExtensionClasses();
        // 这里cachedAdaptiveClass一定不为空，因为META-INFO/dubbo/internal目录下存在AdaptiveExtensionLoaderFactory映射关系
        if (cachedAdaptiveClass != null) {
            return cachedAdaptiveClass;
        }
        return cachedAdaptiveClass = createAdaptiveExtensionClass();
    }
```
我们首先来看一下getExtensionClasses，然后调用createAdaptiveExtensionClass方法


这里就是在加载AdaptiveExtensionClass的时候，会使用javaassist创建对应的代码。
```java
    private Class<?> createAdaptiveExtensionClass() {
        String code = new AdaptiveClassCodeGenerator(type, cachedDefaultName).generate();
        ClassLoader classLoader = findClassLoader();
        org.apache.dubbo.common.compiler.Compiler compiler = ExtensionLoader.getExtensionLoader(org.apache.dubbo.common.compiler.Compiler.class).getAdaptiveExtension();
        return compiler.compile(code, classLoader);
    }
```

# @Active
