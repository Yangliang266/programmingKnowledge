### 添加



hset key field val

hsetnx key field val 

> 如果field 存在 返回0 ，进行值的覆盖，不存在进行操作 返回1



hincrby key field 30 

> 递增30，如果field不存在，则初始化field的值为0，递增30



### 批量添加

hmset key1 field1 val field2 val2





### 删除

hdel key field





### 查询

hget key 

> 查询key对应field的val

hgetall 

> 查询所有key field val

hlen

> hash域的数量

hstrlen key

> 查询对应key field val的长度

hexists key field

> 查询key对应的field域是否存在

hkeys key

> 查询对应key的所有域

hvals key

> 查询对应key中域的所有值

### 批量获取

 hmget key field1 val1 field2 val2



