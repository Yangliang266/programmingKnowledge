### 配置类

```java
@ConfigurationProperties(prefix = "spring.redisson",ignoreUnknownFields = false)
@Data
public class RedissonProperties {

    private String address; //连接地址

    private int database;

    /**
     * 等待节点回复命令的时间。该时间从命令发送成功时开始计时
     */
    private int timeout;

    private String password;

    private RedissonPoolProperties pool;
}

@Data
public class RedissonPoolProperties {

    private int maxIdle; /**连接池中的最大空闲连接**/

    private int minIdle;  /**最小连接数**/

    private int maxActive;/**连接池最大连接数**/

    private int maxWait;/**连接池最大阻塞等待时间**/

}
```



### 自动导入类

```java
@Configuration
// 使redisson配置生效，使RedissonProperties加载到bean容器中
@EnableConfigurationProperties(RedissonProperties.class)
// 存在reddison包才能加载
@ConditionalOnClass({Redisson.class})
public class RedissonAutoConfiguration {

    @Autowired
    RedissonProperties redissonProperties;

    @Bean
    RedissonClient redissonClient() {
        Config config = new Config();
        String node = redissonProperties.getAddress();
        node = node.startsWith("redis://") ? node : "redis://" + node;
        SingleServerConfig serverConfig = config.useSingleServer()
                .setAddress(node)
                .setTimeout(redissonProperties.getTimeout())
                .setConnectionMinimumIdleSize(redissonProperties.getPool().getMinIdle());
        if (StringUtils.isNotBlank(redissonProperties.getPassword())) {
            serverConfig.setPassword(redissonProperties.getPassword());
        }
        return Redisson.create(config);
    }

}
```

