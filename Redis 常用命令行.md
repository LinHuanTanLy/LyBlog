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

### 查看key对应的值

get 【key名】

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



