### 添加



set key val

setnx key val 

> 如果key存在 返回0 不进行操作，不存在进行操作 返回1



incr key 

> 当key的值是num类型 加1，返回加1后的值

incrby key 30 

> 递增30，如果key不存在，则初始化key的值为0，递增30

decr key 

> 当key的值是num类型 减1，返回减1后的值

decrby key 30 

> 递减30，如果key不存在，则初始化key的值为0，递减30



append key val 

> 将val添加到，原有的key对应的val的末尾

setrange key 6 val

> 把key对应val索引位置为6之后的字符替换为val

### 批量添加

mset key1 val1 key2 val2





### 查询

get key 

> 查询对应的val

getall 

> 查询所有key val

strlen key

> 查询对应key val的长度

getrange key 6

> 查询key对应val索引为6之后的字符



### 批量获取

 mget key1 key2



