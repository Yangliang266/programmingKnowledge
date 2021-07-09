## dubbo中的扩展点

### 一 扩展点的特征

> 在类级别标准@SPI(RndomLoadBalance.NAME)
>
> 括号内表示当前扩展点的默认扩展点

### 二 扩展点的类型

#### 指定名称扩展点

```java
LoadBalance load=ExtensionLoader.
    getExtensionLoader(LoadBalance.class).
    getExtension(random);
```

- 找到protocol的全路径名称，在META-INFO/dubbo/internal 或者META-INFO/dubbo/ 或者META-INFO/dubbo/service路径下
- 在指定文件中找到name对应的实现类

#### 自定义扩展点

```java
LoadBalance load=ExtensionLoader.
    getExtensionLoader(LoadBalance.class).
    getExtension(random);
```

##### 实现

> 1 新建继承AbstractLoadBalance的类
>
> 2 META-INFO/dubbo +文件名（带有@SPI类的全路径，即要实现的spi的）
>
>    key=自定义名称， value=新建类的全路径
>
> 3 在@Dubbo注解中定义loadbalance=key

##### 原理

- `getExtensionClasses`返回一个map对象，key=name（扩展点名字），clazz=name(对应的扩展类)
- 假设当前加载的扩展名字是：random, 那么此时clazz=RandomLoadBalance  

##### 源码分析

```java
private T createExtension(String name, boolean wrap) {
        Class<?> clazz = (Class)this.getExtensionClasses().get(name);
        if (clazz == null) {
            throw this.findException(name);
        } else {
            try {
                T instance = EXTENSION_INSTANCES.get(clazz);
                if (instance == null) {
                    EXTENSION_INSTANCES.putIfAbsent(clazz, clazz.newInstance());
                    instance = EXTENSION_INSTANCES.get(clazz);
                }
				// 会对存在依赖注入的扩展点进行依赖注入
                this.injectExtension(instance);
                ...
                    ...
    }
```

> META-INF/dubbo/internal  

```properties
random=org.apache.dubbo.rpc.cluster.loadbalance.RandomLoadBalance
roundrobin=org.apache.dubbo.rpc.cluster.loadbalance.RoundRobinLoadBalance
leastactive=org.apache.dubbo.rpc.cluster.loadbalance.LeastActiveLoadBalance
consistenthash=org.apache.dubbo.rpc.cluster.loadbalance.ConsistentHashLoadBalance 
shortestresponse=org.apache.dubbo.rpc.cluster.loadbalance.ShortestResponseLoadBalance
```

> 把上面这个文件中的内容，解析出来以Map的形式存储  

```java
private Map<String, Class<?>> getExtensionClasses() {
        Map<String, Class<?>> classes = (Map)this.cachedClasses.get();
        if (classes == null) {
            synchronized(this.cachedClasses) {
                classes = (Map)this.cachedClasses.get();
                if (classes == null) {
                    classes = this.loadExtensionClasses();
                    this.cachedClasses.set(classes);
                }
            }
        }

        return classes;
    }
```

- 根据默认配置的查找路径进行查找并解析

- strategies 对应的是不同扫描路径下的策略

  > META-INF/dubbo
  > META-INF/dubbo/internal
  > META-INF/services  

```java
private Map<String, Class<?>> loadExtensionClasses() {
        this.cacheDefaultExtensionName();
        Map<String, Class<?>> extensionClasses = new HashMap();
        LoadingStrategy[] var2 = strategies;
        int var3 = var2.length;

        for(int var4 = 0; var4 < var3; ++var4) {
            LoadingStrategy strategy = var2[var4];
            this.loadDirectory(extensionClasses, strategy.directory(), this.type.getName(), strategy.preferExtensionClassLoader(), strategy.overridden(), strategy.excludedPackages());
            this.loadDirectory(extensionClasses, strategy.directory(), this.type.getName().replace("org.apache", "com.alibaba"), strategy.preferExtensionClassLoader(), strategy.overridden(), strategy.excludedPackages());
        }

        return extensionClasses;
    }
```

返回的结果如下 :

![image-20201127151400285](https://yliang.oss-cn-shanghai.aliyuncs.com/img/programming/image-20201127151400285.png)



```java
if (CollectionUtils.isNotEmpty(wrapperClassesList)) {
    Iterator var6 = wrapperClassesList.iterator();

    label37:
    while(true) {
        Class wrapperClass;
        Wrapper wrapper;
        do {
            if (!var6.hasNext()) {
                break label37;
            }

            wrapperClass = (Class)var6.next();
            wrapper = (Wrapper)wrapperClass.getAnnotation(Wrapper.class);
        } while(wrapper != null && (!ArrayUtils.contains(wrapper.matches(), name) || ArrayUtils.contains(wrapper.mismatches(), name)));

        instance = this.injectExtension(wrapperClass.getConstructor(this.type).newInstance(instance));
    }
}
```

```properties
filter=org.apache.dubbo.rpc.protocol.ProtocolFilterWrapper
listener=org.apache.dubbo.rpc.protocol.ProtocolListenerWrapper
mock=org.apache.dubbo.rpc.support.MockProtocol
dubbo=org.apache.dubbo.rpc.protocol.dubbo.DubboProtocol
injvm=org.apache.dubbo.rpc.protocol.injvm.InjvmProtocol
http=org.apache.dubbo.rpc.protocol.http.HttpProtocol
rmi=org.apache.dubbo.rpc.protocol.rmi.RmiProtocol
hessian=org.apache.dubbo.rpc.protocol.hessian.HessianProtocol
org.apache.dubbo.rpc.protocol.webservice.WebServiceProtocol
thrift=org.apache.dubbo.rpc.protocol.thrift.ThriftProtocol
native-thrift=org.apache.dubbo.rpc.protocol.nativethrift.ThriftProtocol
memcached=org.apache.dubbo.rpc.protocol.memcached.MemcachedProtocol
redis=org.apache.dubbo.rpc.protocol.redis.RedisProtocol
rest=org.apache.dubbo.rpc.protocol.rest.RestProtocol
xmlrpc=org.apache.dubbo.xml.rpc.protocol.xmlrpc.XmlRpcProtocol
grpc=org.apache.dubbo.rpc.protocol.grpc.GrpcProtocol
registry=org.apache.dubbo.registry.integration.RegistryProtocol
service-discovery-registry=org.apache.dubbo.registry.client.ServiceDiscoveryRegistryProtocol
qos=org.apache.dubbo.qos.protocol.QosProtocolWrapper
```

> 比如上面的扩展点，以Wrapper为后缀，最终则以ProtocolFilterWrapper(ProtocolListenerWrapper(QosProtocolWrapper(DubboProtocol)))  显示

##### 总结

> 加载指定路径下的文件内容，保存到集合中
> 会对存在依赖注入的扩展点进行依赖注入
> 会对存在Wrapper类的扩展点，实现扩展点的包装  



#### 自适应扩展点

> 在运行期间，根据上下文来决定当前返回哪个扩展点  

```java
LoadBalance load=ExtensionLoader.
    getExtensionLoader(LoadBalance.class).
    getAdaptiveExtension();
```

##### 自适应扩展点的标识  

- @Adaptive
  - 该注解可以声明在类级别上
  - 也可以声明在方法级别

- 实现原理
  - 如果修饰在类级别，那么直接返回修饰的类
  - 如果修饰在方法界别，动态创建一个代理类（javassist）  

##### 源码分析

> dubbo自始至终都以url传递参数，url在动态的改变，所以不管前置协议是什么，都统一生成Protocol$Adaptive，再去匹配对应的协议

```java
Protocol protocol =
ExtensionLoader.getExtensionLoader(Protocol.class).getAdaptiveExtension();
dubbo:// -> Protocol$Adaptive -> DubboProtocol
registry:// -> Protocol$Adaptive -> RegistryProtocol
provider://  
```



```java
public T getAdaptiveExtension() {
        Object instance = this.cachedAdaptiveInstance.get();
        if (instance == null) {
            ...
                ...

            synchronized(this.cachedAdaptiveInstance) {
                instance = this.cachedAdaptiveInstance.get();
                if (instance == null) {
                    try {
                        instance = this.createAdaptiveExtension();
                        this.cachedAdaptiveInstance.set(instance);
                    }
                    ...
                        ...
                }
            }
        }
```

```java
private Class<?> getAdaptiveExtensionClass() {
        this.getExtensionClasses();
        return this.cachedAdaptiveClass != null ? this.cachedAdaptiveClass : (this.cachedAdaptiveClass = this.createAdaptiveExtensionClass());
    }
```

```java
private Class<?> createAdaptiveExtensionClass() {
        String code = (new AdaptiveClassCodeGenerator(this.type, this.cachedDefaultName)).generate();
        ClassLoader classLoader = findClassLoader();
        Compiler compiler = (Compiler)getExtensionLoader(Compiler.class).getAdaptiveExtension();
        return compiler.compile(code, classLoader);
    }
```

##### 总结

> 如果当前加载的扩展点存在自适应的类，那么直接返回
> 否则，会动态创建一个字节码，然后进行返回.



#### 激活扩展点

> 相当于Spring中的conditional ， 多为dubbo内部过滤使用

##### 使用

```java
ExtensionLoader extensionLoader=ExtensionLoader.getExtensionLoader(Filter.class);
URL url=new URL("","",0);
url=url.addParameter("cache","cache");
List<Filter> filters=extensionLoader.getActivateExtension(url,"cache");
System.out.println(filters.size()); //10
```

##### 实现条件  

> 只要url参数中包含CACHE_KEY,那么 CacheFilter就会被激活  

##### 扩展点标识

```java
@Activate(group = {CONSUMER, PROVIDER}, value = CACHE_KEY)
public class CacheFilter implements Filter 
```

