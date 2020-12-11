# Dubbo服务的注册流程

## 服务发布步骤

- 注解

  ```java
  @DubboService(
          loadbalance = "random",
          cluster = "failover",
          retries = 2)
  ```

- 注解扫描

  ```java
  @DubboComponentScan
  ```

### dubbo服务发布有两种形式

* xml 形式 <dubbo:service>
* @DubboService/ @Service

### 一 Dubbo注解的解析流程

`@DubboComponentScan` -> `DubboComponentScanRegistrar` 

> 把什么注册到了Spring IOC容器？

#### 1 DubboComponentScanRegistrar

```java
public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
    Set<String> packagesToScan = this.getPackagesToScan(importingClassMetadata);
    this.registerServiceAnnotationBeanPostProcessor(packagesToScan, registry);
    DubboBeanUtils.registerCommonBeans(registry);
}
```

```java
private void registerServiceAnnotationBeanPostProcessor(Set<String> packagesToScan, BeanDefinitionRegistry registry) {
    // 拿到bean注解的定义
    BeanDefinitionBuilder builder = BeanDefinitionBuilder.rootBeanDefinition(ServiceAnnotationBeanPostProcessor.class);、
    // 添加构造参数    
    builder.addConstructorArgValue(packagesToScan);
    builder.setRole(2);
    AbstractBeanDefinition beanDefinition = builder.getBeanDefinition();
    BeanDefinitionReaderUtils.registerWithGeneratedName(beanDefinition, registry);
}

//  ServiceAnnotationBeanPostProcessor 服务注解bean处理器
```

> `ServiceAnnotationBeanPostProcessor` 在被加载完成后，初始化下面方法

#### 2 ServiceClassPostProcessor

```java
public void postProcessBeanDefinitionRegistry(BeanDefinitionRegistry registry) throws BeansException {
    // 构置基本构造bean - 包含dubbo监听 这个bean会在spring 容器的上下文装载完成之后，触发监听   
    BeanRegistrar.registerInfrastructureBean(registry, "dubboBootstrapApplicationListener", DubboBootstrapApplicationListener.class);
        Set<String> resolvedPackagesToScan = this.resolvePackagesToScan(this.packagesToScan);
        if (!CollectionUtils.isEmpty(resolvedPackagesToScan)) {
            // 服务注册bean
            this.registerServiceBeans(resolvedPackagesToScan, registry);
        } else if (this.logger.isWarnEnabled()) {
            this.logger.warn("packagesToScan is empty , ServiceBean registry will be ignored!");
        }

    }
```



`registerServiceBeans`

```java
private void registerServiceBeans(Set<String> packagesToScan, BeanDefinitionRegistry registry) {
    DubboClassPathBeanDefinitionScanner scanner = new DubboClassPathBeanDefinitionScanner(registry, this.environment, this.resourceLoader);
    BeanNameGenerator beanNameGenerator = this.resolveBeanNameGenerator(registry);
    scanner.setBeanNameGenerator(beanNameGenerator);
    // 扫描含有dubboService service(alibaba) service(apache)的注解
    serviceAnnotationTypes.forEach((annotationType) -> {
        scanner.addIncludeFilter(new AnnotationTypeFilter(annotationType));
    });
    Iterator var5 = packagesToScan.iterator();

    while(true) {
        while(var5.hasNext()) {
            String packageToScan = (String)var5.next();
            scanner.scan(new String[]{packageToScan}); 
            // 扫描所有有DubboService注解的bean
            Set<BeanDefinitionHolder> beanDefinitionHolders = this.findServiceBeanDefinitionHolders(scanner, packageToScan, registry, beanNameGenerator);
            if (!CollectionUtils.isEmpty(beanDefinitionHolders)) {
                Iterator var8 = beanDefinitionHolders.iterator();
                while(var8.hasNext()) {
                    // 服务发布的元数据
                    BeanDefinitionHolder beanDefinitionHolder = (BeanDefinitionHolder)var8.next();
                    // 注册服务bean
                    this.registerServiceBean(beanDefinitionHolder, registry, scanner);
                }

                ....
                    .....
        }

        return;
    }
}
```



`registerServiceBean`

注册和服务有关的信息，实际上都在@DubboService  

* 服务以什么协议发布
* 服务的负载均衡策略
* 服务的容错策略
* 服务发布端口

> 回答：把什么注册到了Spring IOC容器？

> 以上的配置信息都放置在org.apache.dubbo.config.spring.ServiceBean 上,注入到Spring IOC容器中

```java
private void registerServiceBean(BeanDefinitionHolder beanDefinitionHolder, BeanDefinitionRegistry registry, DubboClassPathBeanDefinitionScanner scanner) {
        Class<?> beanClass = this.resolveClass(beanDefinitionHolder);
        Annotation service = this.findServiceAnnotation(beanClass);
    	// 解析注解参数 
        AnnotationAttributes serviceAnnotationAttributes = AnnotationUtils.getAnnotationAttributes(service, false, false);
        Class<?> interfaceClass = DubboAnnotationUtils.resolveServiceInterfaceClass(serviceAnnotationAttributes, beanClass);
        String annotatedServiceBeanName = beanDefinitionHolder.getBeanName();
    	// 构建服务bean定义
        AbstractBeanDefinition serviceBeanDefinition = this.buildServiceBeanDefinition(service, serviceAnnotationAttributes, interfaceClass, annotatedServiceBeanName);
        String beanName = this.generateServiceBeanName(serviceAnnotationAttributes, interfaceClass);
        if (scanner.checkCandidate(beanName, serviceBeanDefinition)) {
            // 将一个 dubbo中提供的ServiceBean注入到Spring IOC容器
            registry.registerBeanDefinition(beanName, serviceBeanDefinition);
            ...
                ...

    }
```



`buildServiceBeanDefinition`

```java
private AbstractBeanDefinition buildServiceBeanDefinition(Annotation serviceAnnotation, AnnotationAttributes serviceAnnotationAttributes, Class<?> interfaceClass, String annotatedServiceBeanName) {
    	// ServiceBean 就是上个方法的 serviceBeanDefinition 
    	// 把服务的配置注入其中
    	// 加载完成后。初始化servicebean的方法
        BeanDefinitionBuilder builder = BeanDefinitionBuilder.rootBeanDefinition(ServiceBean.class);
        AbstractBeanDefinition beanDefinition = builder.getBeanDefinition();
        MutablePropertyValues propertyValues = beanDefinition.getPropertyValues();
        String[] ignoreAttributeNames = (String[])ObjectUtils.of(new String[]{"provider", "monitor", "application", "module", "registry", "protocol", "interface", "interfaceName", "parameters"});
        propertyValues.addPropertyValues(new AnnotationPropertyValuesAdapter(serviceAnnotation, this.environment, ignoreAttributeNames));
        this.addPropertyReference(builder, "ref", annotatedServiceBeanName);
    	// 添加一系列的dubboservice的配置信息
        builder.addPropertyValue("interface", interfaceClass.getName());
        builder.addPropertyValue("parameters", this.convertParameters(serviceAnnotationAttributes.getStringArray("parameters")));
        ...
            ...
    }
```

> registry.registerBeanDefinition(beanName, serviceBeanDefinition);
>
> 最终通过上述代码，将一个 dubbo中提供的ServiceBean注入到Spring IOC容器

### 二 ServiceBean的初始化阶段

```java
public ServiceBean() {
    super();
    this.service = null;
}
```

> 当ServiceBean初始化完成之后，会调用下面的方法.

```java
@Override
public void afterPropertiesSet() throws Exception {
    if (StringUtils.isEmpty(getPath())) {
        if (StringUtils.isNotEmpty(beanName)
            && StringUtils.isNotEmpty(getInterface())
            && beanName.startsWith(getInterface())) {
            setPath(beanName);
        }
    }
}
```



### 三 触发监听

#### DubboBootstrapApplicationListener

> spring的事件监听，在上述将dubboservice的配置信息注入到ServiceBean成功后，由于DubboBootstrapApplicationListener在被上述方法postProcessBeanDefinitionRegistry进行注册，进而监听后触发下面方法

```java
private void onContextRefreshedEvent(ContextRefreshedEvent event) {
    this.dubboBootstrap.start();
}
```

##### 猜想流程

* 元数据/远程配置信息的初始化
* 前面的配置拼接url（）
* 如果是dubbo协议，则启动netty server
* 服务注册

### 四 服务的发布与注册

> 一个服务发布的次数取决于协议的配置数

#### DubboBootstrap

```java
public DubboBootstrap start() {
    if (this.started.compareAndSet(false, true)) {
        this.ready.set(false);
        this.initialize();
        if (this.logger.isInfoEnabled()) {
            this.logger.info(NAME + " is starting...");
        }

        // 服务的发布
        this.exportServices();
        if (!this.isOnlyRegisterProvider() || this.hasExportedServices()) {
            this.exportMetadataService();
            // 服务的注册
            this.registerServiceInstance();
        }

        this.referServices();
        ...
            ...
    }

    return this;
}
```

```java
private void exportServices() {
    this.configManager.getServices().forEach((sc) -> {
        // 遍历serviceBean
        ServiceConfig serviceConfig = (ServiceConfig)sc;
        serviceConfig.setBootstrap(this);
        // 异步发布
        if (this.exportAsync) {
            ExecutorService executor = this.executorRepository.getServiceExporterExecutor();
            Future<?> future = executor.submit(() -> {
                sc.export();
                this.exportedServices.add(sc);
            });
            this.asyncExportingFutures.add(future);
        } else {
            sc.export();
            this.exportedServices.add(sc);
        }

    });
}
```

> 注意：ServiceConfig和ServiceBean的区别
>
> ServiceBean是继承ServiceConfig 存储服务配置信息

#### ServiceConfig

`export`

> 遍历所有dubbo服务，进行服务发布.

```xml
<dubbo:service beanName="ServiceBean:com.itcrazy.springboot.dubbo.springbootdubboprovider.services.IDemoService" />
<dubbo:service beanName="ServiceBean:com.itcrazy.springboot.dubbo.ISayHelloService" />
dubbo://ip:port?com.itcrazy.springboot.dubbo.springbootdubboprovider.services.IDemoService&xxx&xxx
dubbo://ip:port?com.itcrazy.springboot.dubbo.ISayHelloService&xxx&xxx
```

```java
public synchronized void export() {
    if (this.shouldExport()) {
        if (this.bootstrap == null) {
            this.bootstrap = DubboBootstrap.getInstance();
            this.bootstrap.initialize();
        }
		// 服务元数据的添加
        this.checkAndUpdateSubConfigs();
        this.serviceMetadata.setVersion(this.getVersion());
        this.serviceMetadata.setGroup(this.getGroup());
        this.serviceMetadata.setDefaultGroup(this.getGroup());
        this.serviceMetadata.setServiceType(this.getInterfaceClass());
        this.serviceMetadata.setServiceInterfaceName(this.getInterface());
        this.serviceMetadata.setTarget(this.getRef());
        if (this.shouldDelay()) {
            // 延迟启动 避免springboot还没启动完成， dubbo启动产生错误
            DELAY_EXPORT_EXECUTOR.schedule(this::doExport, (long)this.getDelay(), TimeUnit.MILLISECONDS);
        } else {
            // 发布
            this.doExport();
        }
		// 发布后的动作
        this.exported();
    }
}
```

```java
protected synchronized void doExport() {
    ...
    // 服务发布
    this.doExportUrls();
    ...
}
```



`doExportUrls`

> 主要流程，根据开发者配置的协议列表，遍历协议列表逐项进行发布。

```java
private void doExportUrls() {
    // 建立缓存 服务发布的一些数据会放入其中
    ServiceRepository repository = ApplicationModel.getServiceRepository();
    ServiceDescriptor serviceDescriptor = repository.registerService(this.getInterfaceClass());
    repository.registerProvider(this.getUniqueServiceName(), this.ref, serviceDescriptor, this, this.serviceMetadata);
    // 服务注册的url
    List<URL> registryURLs = ConfigValidationUtils.loadRegistries(this, true);
    Iterator var4 = this.protocols.iterator();

    while(var4.hasNext()) {
        // 如果有多个协议，则一个协议发布一次
        ProtocolConfig protocolConfig = (ProtocolConfig)var4.next();
        String pathKey = URL.buildKey((String)this.getContextPath(protocolConfig).map((p) -> {
            return p + "/" + this.path;
        }).orElse(this.path), this.group, this.version);
        repository.registerService(pathKey, this.interfaceClass);
        this.serviceMetadata.setServiceKey(pathKey);
        this.doExportUrlsFor1Protocol(protocolConfig, registryURLs);
    }

}
```



`doExportUrlsFor1Protocol`

* 生成url
* 根据url中配置的协议类型，调用指定协议进行服务的发布
  * 启动服务
  * 注册服务

```java
private void doExportUrlsFor1Protocol(ProtocolConfig protocolConfig, List<URL> registryURLs) {
        String name = protocolConfig.getName();
        if (StringUtils.isEmpty(name)) {
            name = "dubbo";
        }
		// 存储所有配置信息
    	* dubbo:service
            * dubbo:method
            	* dubbo:argument
        Map<String, String> map = new HashMap();
        map.put("side", "provider");
        appendRuntimeParameters(map);
        AbstractConfig.appendParameters(map, this.getMetrics());
        AbstractConfig.appendParameters(map, this.getApplication());
        AbstractConfig.appendParameters(map, this.getModule());
        AbstractConfig.appendParameters(map, this.provider);
        AbstractConfig.appendParameters(map, protocolConfig);
        AbstractConfig.appendParameters(map, this);
        MetadataReportConfig metadataReportConfig = this.getMetadataReportConfig();
        if (metadataReportConfig != null && metadataReportConfig.isValid()) {
            map.putIfAbsent("metadata-type", "remote");
        }

        String scope;
       
    	...
            ...

        String host;
        ...
            ...

        this.serviceMetadata.getAttachments().putAll(map);
        host = this.findConfigedHosts(protocolConfig, registryURLs, map);
        Integer port = this.findConfigedPorts(protocolConfig, name, map);
    	// 获取装配好的dubbo服务url地址
        URL url = new URL(name, host, port, (String)this.getContextPath(protocolConfig).map((p) -> {
            return p + "/" + this.path;
        }).orElse(this.path), map);
    
    
    	// 扩展点spi
        if (ExtensionLoader.getExtensionLoader(ConfiguratorFactory.class).hasExtension(url.getProtocol())) {
            url = ((ConfiguratorFactory)ExtensionLoader.getExtensionLoader(ConfiguratorFactory.class).getExtension(url.getProtocol())).getConfigurator(url).configure(url);
        }

        scope = url.getParameter("scope");
        if (!"none".equalsIgnoreCase(scope)) {
            if (!"remote".equalsIgnoreCase(scope)) {
                this.exportLocal(url);
            }

            if (!"local".equalsIgnoreCase(scope)) {
                if (CollectionUtils.isNotEmpty(registryURLs)) {
                    var10 = registryURLs.iterator();

                    while(var10.hasNext()) {
                        URL registryURL = (URL)var10.next();
                        if (!"injvm".equalsIgnoreCase(url.getProtocol())) {
                            url = url.addParameterIfAbsent("dynamic", registryURL.getParameter("dynamic"));
                            URL monitorUrl = ConfigValidationUtils.loadMonitor(this, registryURL);
                            if (monitorUrl != null) {
                                url = url.addParameterAndEncoded("monitor", monitorUrl.toFullString());
                            }

                            if (logger.isInfoEnabled()) {
                                if (url.getParameter("register", true)) {
                                    logger.info("Register dubbo service " + this.interfaceClass.getName() + " url " + url + " to registry " + registryURL);
                                } else {
                                    logger.info("Export dubbo service " + this.interfaceClass.getName() + " to url " + url);
                                }
                            }

                            String proxy = url.getParameter("proxy");
                            if (StringUtils.isNotEmpty(proxy)) {
                                registryURL = registryURL.addParameter("proxy", proxy);
                            }

                            Invoker<?> invoker = PROXY_FACTORY.getInvoker(this.ref, this.interfaceClass, registryURL.addParameterAndEncoded("export", url.toFullString()));
                            DelegateProviderMetaDataInvoker wrapperInvoker = new DelegateProviderMetaDataInvoker(invoker, this);
                            Exporter<?> exporter = PROTOCOL.export(wrapperInvoker);
                            this.exporters.add(exporter);
                        }
                    }
                } else {
                    if (logger.isInfoEnabled()) {
                        logger.info("Export dubbo service " + this.interfaceClass.getName() + " to url " + url);
                    }

                    Invoker<?> invoker = PROXY_FACTORY.getInvoker(this.ref, this.interfaceClass, url);
                    DelegateProviderMetaDataInvoker wrapperInvoker = new DelegateProviderMetaDataInvoker(invoker, this);
                    Exporter<?> exporter = PROTOCOL.export(wrapperInvoker);
                    this.exporters.add(exporter);
                }

                WritableMetadataService metadataService = WritableMetadataService.getExtension(url.getParameter("metadata-type", "local"));
                if (metadataService != null) {
                    metadataService.publishServiceDefinition(url);
                }
            }
        }

        this.urls.add(url);
    }
```



远程发布和本地发布

`injvm`的调用

- 判断是否需要发布到远程服务

