## 介绍
Redis hash是一个string类型的field和value的映射表.一个key可对应多个field，一个field对应一个value。
将一个对象存储为hash类型，较于每个字段都存储成string类型更能节省内存。新建一个hash对象时开始是用zipmap(又称为small hash)来存储的。
这个zipmap其实并不是hash table，但是zipmap相比正常的hash实现可以节省不少hash本身需要的一些元数据存储开销。
尽管zipmap的添加，删除，查找都是O(n)，但是由于一般对象的field数量都不太多。
所以使用zipmap也是很快的,也就是说添加删除平均还是O(1)。如果field或者value的大小超出一定限制后，Redis会在内部自动将zipmap替换成正常的hash实现。

## 操作
### 1. hset
```
HSET key field value
```
将哈希表key中的域field的值设为value。如果key不存在，一个新的哈希表被创建并进行hset操作。如果域field已经存在于哈希表中，旧值将被覆盖。

### 2. hget
```
HGET key field
```
返回哈希表key中指定的field的值。

### 3. hsetnx
```
HSETNX key field value
```
将哈希表key中的域field的值设置为value，当且仅当域field不存在。若域field已经存在，该操作无效。如果key不存在，一个新哈希表被创建并执行hsetnx命令。

### 4. hmset
```
HMSET key field value [field value ...]
```
同时将多个field - value(域-值)对设置到哈希表key中。此命令会覆盖哈希表中已存在的域。如果key不存在，一个空哈希表被创建并执行hmset操作。

```
redis> MSET key1 "Hello" key2 "World"
"OK"
redis> GET key1
"Hello"
redis> GET key2
"World"
redis> 
```

### 5. hmget
```
HMGET key field [field ...]
```
返回哈希表key中，一个或多个给定域的值。如果给定的域不存在于哈希表，那么返回一个nil值。因为不存在的key被当作一个空哈希表来处理，所以对一个不存在的key进行hmget操作将返回一个只带有nil值的表。

### 6. hgetall
```
HGETALL key
```

返回哈希表key中，所有的域和值。在返回值里，紧跟每个域名(field name)之后是域的值(value)，所以返回值的长度是哈希表大小的两倍。

### 7. hdel
```
HDEL key field [field ...]
```
删除哈希表key中的一个或多个指定域，不存在的域将被忽略。

### 8. hlen
```
HLEN key
```
返回哈希表key对应的field的数量。

### 9. hexists
```
HEXISTS key field
```
查看哈希表key中，给定域field是否存在。

### 10. hkeys
```
HKEYS key
```
获得哈希表中key对应的所有field。

### 11. hvals
```
HVALS key
```
获得哈希表中key对应的所有values。

### 12. hincrby
```
redis> HSET myhash field 5
(integer) 1
redis> HINCRBY myhash field 1
(integer) 6
redis> HINCRBY myhash field -1
(integer) 5
redis> HINCRBY myhash field -10
(integer) -5
redis> 
```
为哈希表key中的域field的值加上增量increment。增量也可以为负数，相当于对给定域进行减法操作。如果key不存在，一个新的哈希表被创建并执行hincrby命令。如果域field不存在，那么在执行命令前，域的值被初始化为0。对一个储存字符串值的域field执行hincrby命令将造成一个错误。本操作的值限制在64位(bit)有符号数字表示之内。

