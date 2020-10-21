# SETNX key value

只在键 `key` 不存在的情况下， 将键 `key` 的值设置为 `value` 。

若键 `key` 已经存在， 则 `SETNX` 命令不做任何动作。

`SETNX` 是『SET if Not eXists』(如果不存在，则 SET)的简写。

```
redis> EXISTS job                # job 不存在
(integer) 0

redis> SETNX job "programmer"    # job 设置成功
(integer) 1

redis> SETNX job "code-farmer"   # 尝试覆盖 job ，失败
(integer) 0

redis> GET job                   # 没有被覆盖
"programmer"
```

# SETEX key seconds value

将键 `key` 的值设置为 `value` ， 并将键 `key` 的生存时间设置为 `seconds` 秒钟。

如果键 `key` 已经存在， 那么 `SETEX` 命令将覆盖已有的值。

在键 `key` 不存在的情况下执行 `SETEX` ：

```redis
redis> SETEX cache_user_id 60 10086
OK

redis> GET cache_user_id  # 值
"10086"

redis> TTL cache_user_id  # 剩余生存时间
(integer) 49
```

键 `key` 已经存在， 使用 `SETEX` 覆盖旧值：

```redis
redis> SET cd "timeless"
OK

redis> SETEX cd 3000 "goodbye my love"
OK

redis> GET cd
"goodbye my love"

redis> TTL cd
(integer) 2997
```

# GET key

## 返回值

如果键 `key` 不存在， 那么返回特殊值 `nil` ； 否则， 返回键 `key` 的值。

如果键 `key` 的值并非字符串类型， 那么返回一个错误， 因为 `GET` 命令只能用于字符串值。

对不存在的键 `key` 或是字符串类型的键 `key` 执行 `GET` 命令：

```
redis> GET db
(nil)

redis> SET db redis
OK

redis> GET db
"redis"
```

对不是字符串类型的键 `key` 执行 `GET` 命令：

```
redis> DEL db
(integer) 1

redis> LPUSH db redis mongodb mysql
(integer) 3

redis> GET db
(error) ERR Operation against a key holding the wrong kind of value
```