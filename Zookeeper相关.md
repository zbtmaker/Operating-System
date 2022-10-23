# Dubbo 使用
## Zookeeper相关

### 使用
#### zookeeper Server启动

```bash
zoubaitao@zoubaitaodeMacBook-Pro ~ % zkServer start
ZooKeeper JMX enabled by default
Using config: /usr/local/etc/zookeeper/zoo.cfg
Starting zookeeper ... STARTED
```

#### zookeeper Client启动
```bash
zoubaitao@zoubaitaodeMacBook-Pro ~ % zkCli
Connecting to localhost:2181
Welcome to ZooKeeper!
JLine support is enabled
[zk: localhost:2181(CONNECTING) 0] 
WATCHER::

WatchedEvent state:SyncConnected type:None path:null
```

#### zookeeper Server关闭

```bash
zoubaitao@zoubaitaodeMacBook-Pro zookeeper % zkServer stop
ZooKeeper JMX enabled by default
Using config: /usr/local/etc/zookeeper/zoo.cfg
Stopping zookeeper ... /usr/local/Cellar/zookeeper/3.7.0_1/libexec/bin/zkServer.sh: line 219: kill: (15158) - No such process
STOPPED
```

## JDK SPI 
JDK 提供了SPI机制，具体是什么可以后续通过阅读文章给出一个合理的解释
我们首先提供一个接口
```java
package com.zbt.spi;

public interface DemoService {
    void doNothing();
}
```
然后这个接口有两个实现，DemoServiceOneImpl、DemoServiceTwoImpl
DemoServiceOneImpl
```java
package com.zbt.spi.impl;

import com.zbt.spi.DemoService;
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class DemoServiceOneImpl implements DemoService {
    @Override
    public void doNothing() {
        log.info("invoke DemoServiceOneImpl method");
    }
}
```
DemoServiceTwoImpl
```java
package com.zbt.spi.impl;

import com.zbt.spi.DemoService;
import lombok.extern.slf4j.Slf4j;

@Slf4j
public class DemoServiceTwoImpl implements DemoService {
    @Override
    public void doNothing() {
        log.info("invoke DemoServiceTwoImpl method");
    }
}
```
同时需要在src/main/resources/META-INF/services这个路径下创建一个com.zbt.spi.DemoService文件，文件中提供实现了这个接口的实例，如下所示
```properties
com.zbt.spi.impl.DemoServiceOneImpl
com.zbt.spi.impl.DemoServiceTwoImpl
```
然后我们通过JDK提供的ServiceLoader来加载对应的类
```java
package com.zbt.spi;

import java.util.ServiceLoader;

public class JdkSpiMain {

    public static void main(String[] args) {
        ServiceLoader<DemoService> serviceLoader = ServiceLoader.load(DemoService.class);
        for (DemoService demoService : serviceLoader) {
            demoService.doNothing();
        }
    }
}
```
这样就能实现通过JDK提供的SPi机制将所有实现了DemoService接口的实现类加载出来。这里还有一个疑问就是为什么需要这样一个机制，已经有的使用方式是jdk.sql.Driver接口，不同的数据库创建链接的方式不同，都需要实现这个Driver接口（具体内容后续补充）。

## Dubbo SPI
相比较JDK提供的SPI一次性加载接口所有的实现类，Dubbo SPI提供的方式灵活的多，可以通过按需加载的方式，Dubbo SPI在JDK SPI基础上进行了扩展，那么是如何扩展的呢？
JDK SPI 只是加载/META-INF/services这个目录里面的文件，但是Dubbo SPI额外增加了三个目录。
```properties
META-INF/services/
META-INF/dubbo/
META-INF/dubbo/internal/
META-INF/dubbo/external/
```
加载不同的目录使用了策略模式，其中
```java
public interface LoadingStrategy extends Prioritized {

    String directory();

    default boolean preferExtensionClassLoader() {
        return false;
    }

    default String[] excludedPackages() {
        return null;
    }
 
    default String[] includedPackages() {
        // default match all
        return null;
    }
 
    default String[] includedPackagesInCompatibleType() {
        // default match all
        return null;
    }
 
    default String[] onlyExtensionClassLoaderPackages() {
        return new String[]{};
    }
 
    default boolean overridden() {
        return false;
    }

    default String getName() {
        return this.getClass().getSimpleName();
    }
 
    String ALL = "ALL";
}

```
然后提供了四个子类，这几个子类提供的directory和getPriority方法
* DubboExternalLoadingStrategy
* DubboInternalLoadingStrategy
* DubboLoadingStrategy
* ServicesLoadingStrategy

这几个策略类的加载的优先级为DubboExternalLoadingStrategy、DubboInternalLoadingStrategy、DubboLoadingStrategy、ServicesLoadingStrategy。
我们可以看一下dubbo.internal中的文件

```properties
adaptive=org.apache.dubbo.common.compiler.support.AdaptiveCompiler
jdk=org.apache.dubbo.common.compiler.support.JdkCompiler
javassist=org.apache.dubbo.common.compiler.support.JavassistCompiler
```
我们可以看出，Dubbo SPI和原生的JDK SPI的不同之处在于其配置文件是key-value键值对的方式，如果想要加载某一个key对应的类，那么只需要指定对应类，然后通过指定对应的key就可以了。

我们来看一下SPI的灵魂类，ExtensionLoader类。


