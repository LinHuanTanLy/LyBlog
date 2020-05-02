[toc]



# Redis 常用命令行

## 基本知识

支持5种数据类型；单线程；

* string
* hash
* list
* set
* zset
* 

## 通用命令

### 启动redis

redis-server.exe redis.windows.conf

### 登录redis

redis-cli -p 6379

### 设置key

set 【key名】 【key值】

### 批量设置Key

mset 【key1】【value1】【key2】【values2】...

### 所有的key都不存在时，设置key，要么同时成功，要么同时失败

MSETNX 【key1】【value1】 【key2】 【value2】...



### 查看key对应的值

get 【key名】

### 批量查看key对应的值

mget 【key1】【key2】...

### 查看所有key

 key *

### 删除key

del 【key名】

### 清除所有数据

FLUSHALL

### 移动key到其他数据库

move  【1 数据库名字】

### 设置过期时间

expire 【key名】【时间，默认s】

### 当key不存在时，设置对应的key

SETNX 【key名】 【key值】

### 为key设置value值，并且，设置过期时间 

SETEX 【key名】【过期时间】【key值】

### 查看还有多久过期

 ttl 【key名字】

### 查看key对应的类型

key 【key名】

# String类型命令

### 追加数据

 append 【key名】【追加的数据】

### 获取数据的长度

strlen 【key名】

### 自增1

incr 【key名】

### 自减1

decr【key名】

### 自增步长

INCRBY  【key名】【步长】

### 自减步长

DECRBY  【key名】【步长】

### 截取字符串

GETRANGE  【开始位置】 【偏移量】

### 替换字符串

SETRANGE 【key名】【开始位置】 【要替换的值】

### 返回给定 key 的旧值，key 不存在时，返回 nil ，当 key 存在但不是字符串类型时，返回一个错误

 GETSET 【key】 【value】

# List相关命令行

### 从列表左侧头部添加数据

lpush 【list名】【key】

### 从列表右侧头部添加数据

rpush 【list名】【key】

### 从给左侧头部取出一个元素

lpop 【list名】

### 从给右侧尾部取出一个元素

rpop 【list名】

### 取出指定范围的元素

lrange 【list名】【开始位置】【偏移量】

### 删除列表指定元素

### Lrem 【list名】【删除个数】【value】



| 删除个数count > 0 | 从表头开始向表尾搜索，移除与 VALUE 相等的元素，数量为 COUNT  |
| :---------------: | ------------------------------------------------------------ |
| 删除个数count < 0 | 从表尾开始向表头搜索，移除与 VALUE 相等的元素，数量为 COUNT 的绝对值。 |
|     count = 0     | 移除表中所有与 VALUE 相等的值                                |



### 获取第几个坐标下的值

lindex 【list名】【index】

### 获取列表长度

llen 【list名】

修改坐标下的值

lset 【key名】【下标】【值】

### 修剪列表

ltrim 【key名】【开始位置】【偏移量】

# hash类型命令

# set类型命令

# 事务相关

> 本质：一组命令的集合！一个事务中的所有命令都会被序列化，在事务执行的过程中，会按照顺序执行！
>
> 一次性、顺序性、排他性。

> Redis单条命令是保存原子性的，但是事务不保证原子性 。

## 事务的执行

1. 开启事务
2. 命令入队
3. 执行任务

```shell
multi // 开启任务

xxx
xxx

exec // 执行事务
```





## 事务的放弃

1. 开启事务
2. 命令入队
3. 放弃任务

```shell
multi // 开启任务

xxx
xxx

DISCARD // 执行事务
```

