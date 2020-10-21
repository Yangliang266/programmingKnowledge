### @Configration

> 定义：直接导入单个bean，类似xml，<Bean name="...">配置

```java
@Configuration
public class FormatAutoConfig {

    @Bean
    @Primary
    @ConditionalOnMissingClass("com.alibaba.fastjson.JSON")
    public TemplateFormat stringFormat() {
        return new StringFormatImp();
    }

    @Bean
    @ConditionalOnClass(name = "com.alibaba.fastjson.JSON")
    public TemplateFormat jsonFormat() {
        return new JsonFormatImp();
    }
}
```



### @import

> 定义：导入外部资源包，此资源必须是已经被控制反转

1. 参数导入
2. 实现 ImportSelector 自定义规则
3. ImportBeanDefinitionRegistry 手动添加到IoC

```java
@EnableConfigurationProperties({FormatProperties.class})
@Import(FormatAutoConfig.class)
@Configuration
public class TempFormatProcessor {

    @Bean
    public TempDoFormat tempDoFormat(TemplateFormat templateFormat, FormatProperties formatProperties) {
        return new TempDoFormat(templateFormat,formatProperties);
    }

}
```





### @componetScan

> 定义：默认扫描@Service，@Repository，@Controller，@Bean的注解   添加到IOC

```java
@SpringBootApplication
@ComponentScan(basePackages = "com.itcrazy.starter")
public class DemoStarterApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoStarterApplication.class, args);
    }

}
```



SpringBoot在写启动类的时候如果不使用@ComponentScan指明对象扫描范围，默认指扫描当前启动类所在的包里的对象，如果当前启动类没有包，则在启动时会报错：Your ApplicationContext is unlikely to start due to a @ComponentScan of the default package错误。 因为启动类不能直接放在main/java文件夹下，必须要建一个包把它放进去或者使用@ComponentScan指明要扫描的包。代码示例如下：