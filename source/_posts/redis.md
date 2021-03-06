---
title: redis
tags: redis
date: 2018-12-06 18:10:43
---

# redis是什么
redis是一个开源的，使用C语言编写的，支持网络交互的、可基于内存也可持久化的Key-Value数据库

# 安装过程
参考另一篇文章`linux`

# linux操作redis

## 连接redis
`./redis-cli`
`127.0.0.1:6379>`

## 配置redis
redis的配置文件位于Redis安装目录下，文件名为 redis.conf，可以通过CONFIG命令查看或者设置配置项。
```
127.0.0.1:6379> config get loglevel
1) "loglevel"
2) "notice"
```
使用`*`获取所有配置项
`config get *`
很多配置，这里就不贴了。[参考这里](https://www.cnblogs.com/ruanraun/p/redis.html)
可以通过修改redis.conf文件或者使用config set 命令来修改配置
```
127.0.0.1:6379> config set loglevel 'notice'
OK
```

## 如何停止/启动/重启redis服务
如果是用apt-get或者yum install安装的redis，可以直接通过下面的命令停止/启动/重启redis
```
/etc/init.d/redis-server stop
/etc/init.d/redis-server start
/etc/init.d/redis-server restart
```
如果是通过源码安装的redis，可以通过redis的客户端程序redis-cli的shutdown命令来停止redis

`redis-cli -h 127.0.0.1 -p 6379 shutdown`
或
`./redis-cli shutdown`
没错，虽然启动的是server，但是这里停止的是`redis-cli`
如果上述方式都没有成功停止redis，则可以使用终极武器 kill -9

启动redis：
`cd src`
`./redis-server`
上面这种启动redis使用的是默认配置，也可以通过启动参数告诉redis使用指定配置
`./redis-server ../redis.conf`

让redis后台运行
`./redis-server &`
然后输入`bg`

# redis命令
[参考redis中文官网](https://www.redis.net.cn/)

## key
### del
del用于删除已存在的键，不存在的 key 会被忽略
`del key_name`
返回被删除key的数量

### dump
DUMP 命令用于序列化给定 key ，并返回被序列化的值
`DUMP KEY_NAME`
如果 key 不存在，那么返回 nil 。 否则，返回序列化之后的值。

### exist
EXISTS 命令用于检查给定 key 是否存在。
`EXISTS KEY_NAME`
若 key 存在返回 1 ，否则返回 0 。

### expire
Expire 命令用于设置 key 的过期时间。key 过期后将不再可用。
`Expire KEY_NAME TIME_IN_SECONDS`
设置成功返回1， 当key不存在或者不能为key设置过期时间时返回0。

### expireat
Expireat 命令用于以 UNIX 时间戳(unix timestamp)格式设置 key 的过期时间。key 过期后将不再可用。
`Expireat KEY_NAME TIME_IN_UNIX_TIMESTAMP`
设置成功返回1，不成功或不存在返回0。
示例：
`EXPIREAT w3ckey 1293840000`

### keys
Keys 命令用于查找所有符合给定模式 pattern 的 key
`KEYS PATTERN`
返回符合给定模式的 key 列表 (Array)。

查找以w3c开头的key：
`keys w3c*`
查找所有的key
`keys *`

### move
move命令用于将当前数据库的key移动到给定的数据库db当中。
`MOVE KEY_NAME DESTINATION_DATABASE`
移动成功返回 1 ，失败则返回 0 。

示例：
```
SELECT 0     # redis默认使用数据库 0，为了清晰起见，这里再显式指定一次。
set w3c redis  #设置一个key
move w3c 1   #转移到1库
exist w3c  #发现w3c已经没有了
select 1 #选择1库
exist w3c #发现w3c已经转移到1库中
```
1. 不存在的key无法转移
2. 当源数据库和目标数据库有相同的 key 时，key无法转移。

### persist
persist命令用于移除给定 key 的过期时间，使得 key 永不过期。
`PERSIST KEY_NAME`
当过期时间移除成功时，返回 1 。 如果 key 不存在或 key 没有设置过期时间，返回 0 。

### pttl
Pttl 命令以毫秒为单位返回 key 的剩余过期时间。
`PTTL KEY_NAME`
当 key 不存在时，返回 -2 。 当 key 存在但没有设置剩余生存时间时，返回 -1 。 否则，以毫秒为单位，返回 key 的剩余生存时间。

### TTL
TTL 命令以秒为单位返回 key 的剩余过期时间。
`TTL KEY_NAME`
当 key 不存在时，返回 -2 。 当 key 存在但没有设置剩余生存时间时，返回 -1 。 否则，以秒为单位，返回 key 的剩余生存时间。

### randomkey
RANDOMKEY 命令从当前数据库中随机返回一个 key 。
`randomkey`
当数据库不为空时，返回一个 key 。 当数据库为空时，返回 nil 。

### rename
Rename 命令用于修改 key 的名称 。
`RENAME OLD_KEY_NAME NEW_KEY_NAME`
改名成功时提示 OK ，失败时候返回一个错误。
当 OLD_KEY_NAME 和 NEW_KEY_NAME 相同，或者 OLD_KEY_NAME 不存在时，返回一个错误。 
当 NEW_KEY_NAME 已经存在时， RENAME 命令将覆盖旧值。

### renamenx
Renamenx 命令用于在新的 key 不存在时修改 key 的名称 。
`RENAMENX OLD_KEY_NAME NEW_KEY_NAME`
修改成功时，返回 1 。 如果 NEW_KEY_NAME 已经存在，返回 0 。

### type
Type 命令用于返回 key 所储存的值的类型。
`TYPE KEY_NAME `
返回 key 的数据类型，数据类型有：
```
none (key不存在)
string (字符串)
list (列表)
set (集合)
zset (有序集)
hash (哈希表)
```

## string
### set
SET 命令用于设置给定 key 的值。如果 key 已经存储其他值， SET 就覆写旧值，且无视类型。
`SET KEY_NAME VALUE`
从 Redis 2.6.12 版本开始， SET 在设置操作成功完成时，才返回 OK 。

### get
Get 命令用于获取指定 key 的值。
`GET KEY_NAME`
如果 key 不存在，返回 nil 。如果key 储存的值不是字符串类型，返回一个错误。

### getrange
getrange命令用于获取存储在指定 key 中字符串的子字符串。字符串的截取范围由 start 和 end 两个偏移量决定(包括 start 和 end 在内)。
`getrange KEY_NAME start end`

示例：首先，设置 mykey 的值并截取字符串。
```
redis 127.0.0.1:6379> SET mykey "This is my test key"
OK
redis 127.0.0.1:6379> GETRANGE mykey 0 3
"This"
redis 127.0.0.1:6379> GETRANGE mykey 0 -1
"This is my test key"
```

### getset
getset 命令用于设置指定 key 的值，并返回 key 旧的值。
`getset key_name value`
返回给定 key 的旧值。 当 key 没有旧值时，即 key 不存在时，返回 nil 。
当 key 存在但不是字符串类型时，返回一个错误。
示例：首先，设置 mykey 的值并截取字符串。
```
redis 127.0.0.1:6379> GETSET mynewkey "This is my test key"
(nil)
redis 127.0.0.1:6379> GETSET mynewkey "This is my new value to test getset"
"This is my test key"
```

### getbit
getbit 命令用于对 key 所储存的字符串值，获取指定偏移量上的位(bit)。
`getbit key_name offset`
返回字符串值指定偏移量上的位(bit)。
当偏移量 OFFSET 比字符串值的长度大，或者 key 不存在时，返回 0 。

### setbit
setbit命令用于对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit)。
`setbit key_name offset`
返回指定偏移量原来储存的位。

### mget
mget命令返回所有(一个或多个)给定 key 的值。 如果给定的 key 里面，有某个 key 不存在，那么这个 key 返回特殊值 nil 。
`MGET KEY1 KEY2 .. KEYN`

### setex
setex 命令为指定的 key 设置值及其过期时间。如果 key 已经存在， SETEX 命令将会替换旧的值。
`setex key_name timeout value`
设置成功时返回 OK 。
说白了就是在设置一个值的时候直接就指定了过期时间。

### setnx
setnx(SET if Not eXists)命令在指定的 key 不存在时，为 key 设置指定的值。
`setnx key_name value`
设置成功，返回 1 。 设置失败，返回 0 。
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

### setrange
setrange 命令用指定的字符串覆盖给定 key 所储存的字符串值，覆盖的位置从偏移量 offset 开始。
`SETRANGE KEY_NAME OFFSET VALUE`
返回被修改后的字符串长度。
就是从指定位置修改字符串的内容。

### strlen
strlen 命令用于获取指定 key 所储存的字符串值的长度。当 key 储存的不是字符串值时，返回一个错误。
`strlen key_name`
返回字符串值的长度。 当 key 不存在时，返回 0。

### mset
mset 命令用于同时设置一个或多个 key-value 对。
`MSET key1 value1 key2 value2 .. keyN valueN `
总是返回OK

### msetnx
msetnx 命令用于所有给定 key 都不存在时，同时设置一个或多个 key-value 对。
`MSETNX key1 value1 key2 value2 .. keyN valueN `
当所有 key 都成功设置，返回 1 。 如果所有给定 key 都设置失败(至少有一个 key 已经存在)，那么返回 0 。

msetnx是原子性操作，只要有一个值失败，所有的值都不会被设置。

### psetex
psetex 命令以毫秒为单位设置 key 的生存时间。
`PSETEX key1 EXPIRY_IN_MILLISECONDS value1 `
设置成功时返回 OK 。

### incr
incr 命令将 key 中储存的数字值增一。
如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 INCR 操作。
如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。
本操作的值限制在 64 位(bit)有符号数字表示之内。
`incr key_name`
返回执行incr命令之后 key 的值。

### incrby
incrby命令将 key 中储存的数字加上指定的增量值。
如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 INCRBY 命令。
如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。
本操作的值限制在 64 位(bit)有符号数字表示之内。
`incr key_name incr_amount`
返回加上指定的增量值之后， key 的值。

### incrbyfloat
incrbyfloat命令为 key 中所储存的值加上指定的浮点数增量值。
`incrbyfloat key_name incr_amount`
返回执行命令之后 key 的值。

用 SET 设置的值可以是指数符号`314e-2`，但执行 INCRBYFLOAT 之后格式会被改成非指数符号`3.14`
SET 设置的值小数部分可以是 0`3.0`，但 INCRBYFLOAT 会将无用的 0 忽略掉，有需要的话，将浮点变为整数`4`

### decr
decr 命令将 key 中储存的数字值减一。
如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 DECR 操作。
如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。
本操作的值限制在 64 位(bit)有符号数字表示之内。
`decr key_name`
返回执行命令之后 key 的值。

### decrby
decrby 命令将 key 所储存的值减去指定的减量值。
如果 key 不存在，那么 key 的值会先被初始化为 0 ，然后再执行 DECRBY 操作。
如果值包含错误的类型，或字符串类型的值不能表示为数字，那么返回一个错误。
本操作的值限制在 64 位(bit)有符号数字表示之内。
`decrby key_name decrement_amount`
返回减去指定减量值之后， key 的值。

### append
append 命令用于为指定的 key 追加值。
如果 key 已经存在并且是一个字符串， APPEND 命令将 value 追加到 key 原来的值的末尾。
如果 key 不存在， APPEND 就简单地将给定 key 设为 value ，就像执行 SET key value 一样。
`append key_name new_value`
返回追加指定值之后， key 中字符串的长度。

## hash
### hdel
hdel 命令用于删除哈希表 key 中的一个或多个指定字段，不存在的字段将被忽略。
`HDEL KEY_NAME FIELD1.. FIELDN `
被成功删除字段的数量，不包括被忽略的字段。

### hexists
hexists 命令用于查看哈希表的指定字段是否存在。
`HEXISTS KEY_NAME FIELD_NAME `
如果哈希表含有给定字段，返回 1 。 如果哈希表不含有给定字段，或 key 不存在，返回 0 。

### hget
hget 命令用于返回哈希表中指定字段的值。
`HGET KEY_NAME FIELD_NAME `
返回给定字段的值。如果给定的字段或 key 不存在时，返回 nil 。

### hgetall
hgetall 命令用于返回哈希表中，所有的字段和值。
在返回值里，紧跟每个字段名(field name)之后是字段的值(value)，所以返回值的长度是哈希表大小的两倍。
`HGETALL KEY_NAME `
以列表形式返回哈希表的字段及字段值。 若 key 不存在，返回空列表。

### hincrby
hincrby 命令用于为哈希表中的字段值加上指定增量值。
增量也可以为负数，相当于对指定字段进行减法操作。
如果哈希表的 key 不存在，一个新的哈希表被创建并执行 HINCRBY 命令。
如果指定的字段不存在，那么在执行命令前，字段的值被初始化为 0 。
对一个储存字符串值的字段执行 HINCRBY 命令将造成一个错误。
本操作的值被限制在 64 位(bit)有符号数字表示之内。
`HINCRBY KEY_NAME FIELD_NAME INCR_BY_NUMBER `

### hincrbyfloat
hincrbyfloat 命令用于为哈希表中的字段值加上指定浮点数增量值。
如果指定的字段不存在，那么在执行命令前，字段的值被初始化为 0 。
`HINCRBYFLOAT KEY_NAME FIELD_NAME INCR_BY_NUMBER `
返回执行 Hincrbyfloat 命令之后，哈希表中字段的值。

### hkeys
hkeys 命令用于获取哈希表中的所有字段名。
`HKEYS KEY_NAME FIELD_NAME INCR_BY_NUMBER `
包含哈希表中所有字段的列表。 当 key 不存在时，返回一个空列表。

### hlen
hlen 命令用于获取哈希表中字段的数量。
`HLEN KEY_NAME `
返回哈希表中字段的数量。 当 key 不存在时，返回 0 。

### hmget
hmget 命令用于返回哈希表中，一个或多个给定字段的值。
如果指定的字段不存在于哈希表，那么返回一个 nil 值。
`HMGET KEY_NAME FIELD1...FIELDN `
返回一个包含多个给定字段关联值的表，表值的排列顺序和指定字段的请求顺序一样。

### hmset
hmset 命令用于同时将多个 field-value (字段-值)对设置到哈希表中。
此命令会覆盖哈希表中已存在的字段。
如果哈希表不存在，会创建一个空哈希表，并执行 HMSET 操作。
`HMSET KEY_NAME FIELD1 VALUE1 ...FIELDN VALUEN`

### hset
hset 命令用于为哈希表中的字段赋值 。
如果哈希表不存在，一个新的哈希表被创建并进行 HSET 操作。
如果字段已经存在于哈希表中，旧值将被覆盖。
`HSET KEY_NAME FIELD VALUE`
返回如果字段是哈希表中的一个新建字段，并且值设置成功，返回 1 。 如果哈希表中域字段已经存在且旧值已被新值覆盖，返回 0 。


### hsetnx
hsetnx 命令用于为哈希表中不存在的的字段赋值 。
如果哈希表不存在，一个新的哈希表被创建并进行 HSET 操作。
如果字段已经存在于哈希表中，操作无效。
如果 key 不存在，一个新哈希表被创建并执行 HSETNX 命令。
`HSETNX KEY_NAME FIELD VALUE`
设置成功，返回 1 。 如果给定字段已经存在且没有操作被执行，返回 0 。

### hvals
hvals 命令返回哈希表所有字段的值。
`HVALS KEY_NAME FIELD VALUE`
返回一个包含哈希表中所有值的表。 当 key 不存在时，返回一个空表。

# JAVA操作redis基本使用
## 初始化
``` java
public class RedisClient {

private Jedis jedis;// 非切片客户端连接
private JedisPool jedisPool;// 非切片连接池
private ShardedJedis shardedJedis;// 切片客户端连接
private ShardedJedisPool shardedJedisPool;// 切片连接池

public RedisClient() {
	initialPool();
	initialShardedPool();
	shardedJedis = shardedJedisPool.getResource();
	jedis = jedisPool.getResource();
}

//初始化非切片连接池
private void initialPool() {
	// 池基本配置
	JedisPoolConfig config = new JedisPoolConfig();
	//可用连接实例的最大数目，默认值为8
	config.setMaxActive(20);
	//控制一个pool最多有多少个状态为idle(空闲的)的jedis实例，默认值也是8
	config.setMaxIdle(5);
	//等待可用连接的最大时间，单位毫秒，默认值为-1，表示永不超时。如果超过等待时间，则直接抛出JedisConnectionException
	config.setMaxWait(1000l);
	config.setTestOnBorrow(false);

	jedisPool = new JedisPool(config, "127.0.0.1", 6379);
}

//初始化切片池
private void initialShardedPool() {
	// 池基本配置
	JedisPoolConfig config = new JedisPoolConfig();
	config.setMaxActive(20);
	config.setMaxIdle(5);
	config.setMaxWait(1000l);
	config.setTestOnBorrow(false);
	// slave链接
	List<JedisShardInfo> shards = new ArrayList<JedisShardInfo>();
	shards.add(new JedisShardInfo("127.0.0.1", 6379, "master"));

	// 构造池
	shardedJedisPool = new ShardedJedisPool(config, shards);
}

public void show() {
	KeyOperate();
	StringOperate();
	ListOperate();
	SetOperate();
	SortedSetOperate();
	HashOperate();
	jedisPool.returnResource(jedis);
	shardedJedisPool.returnResource(shardedJedis);
}
```

## key
```java
/**
 * key 功能 
 * 1.清空所有数据 jedis.flushDB()
 * 2.判断键是否存在 shardedJedis.exists("key1")
 * 3.新增键值对 shardedJedis.set("key001", "value001")
 * 4.删除某个key,若key不存在，则忽略该命令 jedis.del("key002")
 * 5.设置 key001的过期时间 jedis.expire("key001", 5)
 * 6.查看某个key的剩余生存时间,单位【秒】.永久生存或者不存在的都返回-1  jedis.ttl("key001")
 * 7.移除某个key的生存时间  jedis.persist("key001")
 * 8.查看key所储存的值的类型 jedis.type("key001")
 * 
 */
private void KeyOperate() {
	System.out.println("======================key==========================");
	// 清空数据
	System.out.println("清空库中所有数据：" + jedis.flushDB());
	// 判断key否存在
	System.out.println("判断key999键是否存在：" + shardedJedis.exists("key999"));
	System.out.println("新增key001,value001键值对：" + shardedJedis.set("key001", "value001"));
	System.out.println("判断key001是否存在：" + shardedJedis.exists("key001"));
	// 输出系统中所有的key
	System.out.println("新增key002,value002键值对：" + shardedJedis.set("key002", "value002"));
	System.out.println("系统中所有键如下：");
	Set<String> keys = jedis.keys("*");
	Iterator<String> it = keys.iterator();
	while (it.hasNext()) {
		String key = it.next();
		System.out.println(key);
	}
	// 删除某个key,若key不存在，则忽略该命令。
	System.out.println("系统中删除key002: " + jedis.del("key002"));
	System.out.println("判断key002是否存在：" + shardedJedis.exists("key002"));
	// 设置 key001的过期时间
	System.out.println("设置 key001的过期时间为5秒:" + jedis.expire("key001", 5));
	try {
		Thread.sleep(2000);
	} catch (InterruptedException e) {
	}
	// 查看某个key的剩余生存时间,单位【秒】.永久生存或者不存在的都返回-1
	System.out.println("查看key001的剩余生存时间：" + jedis.ttl("key001"));
	// 移除某个key的生存时间
	System.out.println("移除key001的生存时间：" + jedis.persist("key001"));
	System.out.println("查看key001的剩余生存时间：" + jedis.ttl("key001"));
	// 查看key所储存的值的类型
	System.out.println("查看key所储存的值的类型：" + jedis.type("key001"));
	/*
	 * 一些其他方法：1、修改键名：jedis.rename("key6", "key0");
	 * 2、将当前db的key移动到给定的db当中：jedis.move("foo", 1)
	 */
}
```

## string
```java
/**
 * String功能
 * 1.一次性新增key201,key202  jedis.mset("key201", "value201", "key202", "value202")
 * 2.一次性删除key201,key202  jedis.del(new String[] { "key201", "key202" }
 * 3.一次性获取key201,key202  jedis.mget("key201", "key202")
 * 4.新增键值对时防止覆盖原先值  shardedJedis.setnx("key302", "value302_new")
 * 5.获取原值，更新为新值一步完成 shardedJedis.getSet("key302", "value302-after-getset")
 * 6.获取key302对应值中的子串   shardedJedis.getrange("key302", 5, 7)
 * 
 */
private void StringOperate() {
	System.out.println("======================String_1==========================");
	// 清空数据
	System.out.println("清空库中所有数据：" + jedis.flushDB());

	System.out.println("=============增=============");
	jedis.set("key001", "value001");
	jedis.set("key002", "value002");
	jedis.set("key003", "value003");
	System.out.println("已新增的3个键值对如下：");
	System.out.println(jedis.get("key001"));
	System.out.println(jedis.get("key002"));
	System.out.println(jedis.get("key003"));

	System.out.println("=============删=============");
	System.out.println("删除key003键值对：" + jedis.del("key003"));
	System.out.println("获取key003键对应的值：" + jedis.get("key003"));

	System.out.println("=============改=============");
	// 1、直接覆盖原来的数据
	System.out.println("直接覆盖key001原来的数据：" + jedis.set("key001", "value001-update"));
	System.out.println("获取key001对应的新值：" + jedis.get("key001"));
	// 2、直接覆盖原来的数据
	System.out.println("在key002原来值后面追加：" + jedis.append("key002", "+appendString"));
	System.out.println("获取key002对应的新值" + jedis.get("key002"));

	System.out.println("=============增，删，查（多个）=============");
	/**
	 * mset,mget同时新增，修改，查询多个键值对 等价于： jedis.set("name","ssss");
	 * jedis.set("jarorwar","xxxx");
	 */
	System.out.println("一次性新增key201,key202,key203,key204及其对应值："
			+ jedis.mset("key201", "value201", "key202", "value202", "key203", "value203", "key204", "value204"));
	System.out.println(
			"一次性获取key201,key202,key203,key204各自对应的值：" + jedis.mget("key201", "key202", "key203", "key204"));
	System.out.println("一次性删除key201,key202：" + jedis.del(new String[] { "key201", "key202" }));
	System.out.println(
			"一次性获取key201,key202,key203,key204各自对应的值：" + jedis.mget("key201", "key202", "key203", "key204"));
	System.out.println();

	// jedis具备的功能shardedJedis中也可直接使用，下面测试一些前面没用过的方法
	System.out.println("======================String_2==========================");
	// 清空数据
	System.out.println("清空库中所有数据：" + jedis.flushDB());

	System.out.println("=============新增键值对时防止覆盖原先值=============");
	System.out.println("原先key301不存在时，新增key301：" + shardedJedis.setnx("key301", "value301"));
	System.out.println("原先key302不存在时，新增key302：" + shardedJedis.setnx("key302", "value302"));
	System.out.println("当key302存在时，尝试新增key302：" + shardedJedis.setnx("key302", "value302_new"));
	System.out.println("获取key301对应的值：" + shardedJedis.get("key301"));
	System.out.println("获取key302对应的值：" + shardedJedis.get("key302"));

	System.out.println("=============超过有效期键值对被删除=============");
	// 设置key的有效期，并存储数据
	System.out.println("新增key303，并指定过期时间为2秒" + shardedJedis.setex("key303", 2, "key303-2second"));
	System.out.println("获取key303对应的值：" + shardedJedis.get("key303"));
	try {
		Thread.sleep(3000);
	} catch (InterruptedException e) {
	}
	System.out.println("3秒之后，获取key303对应的值：" + shardedJedis.get("key303"));

	System.out.println("=============获取原值，更新为新值一步完成=============");
	System.out.println("key302原值：" + shardedJedis.getSet("key302", "value302-after-getset"));
	System.out.println("key302新值：" + shardedJedis.get("key302"));

	System.out.println("=============获取子串=============");
	System.out.println("获取key302对应值中的子串：" + shardedJedis.getrange("key302", 5, 7));
}
```

## list
```java
/**
 * List功能
 * 1.
 * 2.
 * 3.
 * 4.
 * 5.
 * 6.
 * 7.
 * 8.
 * 
 */
private void ListOperate() {
	System.out.println("======================list==========================");
	// 清空数据
	System.out.println("清空库中所有数据：" + jedis.flushDB());

	System.out.println("=============增=============");
	shardedJedis.lpush("stringlists", "vector");
	shardedJedis.lpush("stringlists", "ArrayList");
	shardedJedis.lpush("stringlists", "vector");
	shardedJedis.lpush("stringlists", "vector");
	shardedJedis.lpush("stringlists", "LinkedList");
	shardedJedis.lpush("stringlists", "MapList");
	shardedJedis.lpush("stringlists", "SerialList");
	shardedJedis.lpush("stringlists", "HashList");
	shardedJedis.lpush("numberlists", "3");
	shardedJedis.lpush("numberlists", "1");
	shardedJedis.lpush("numberlists", "5");
	shardedJedis.lpush("numberlists", "2");
	System.out.println("所有元素-stringlists：" + shardedJedis.lrange("stringlists", 0, -1));
	System.out.println("所有元素-numberlists：" + shardedJedis.lrange("numberlists", 0, -1));

	System.out.println("=============删=============");
	// 删除列表指定的值 ，第二个参数为删除的个数（有重复时），后add进去的值先被删，类似于出栈
	System.out.println("成功删除指定元素个数-stringlists：" + shardedJedis.lrem("stringlists", 2, "vector"));
	System.out.println("删除指定元素之后-stringlists：" + shardedJedis.lrange("stringlists", 0, -1));
	// 删除区间以外的数据
	System.out.println("删除下标0-3区间之外的元素：" + shardedJedis.ltrim("stringlists", 0, 3));
	System.out.println("删除指定区间之外元素后-stringlists：" + shardedJedis.lrange("stringlists", 0, -1));
	// 列表元素出栈
	System.out.println("出栈元素：" + shardedJedis.lpop("stringlists"));
	System.out.println("元素出栈后-stringlists：" + shardedJedis.lrange("stringlists", 0, -1));

	System.out.println("=============改=============");
	// 修改列表中指定下标的值
	shardedJedis.lset("stringlists", 0, "hello list!");
	System.out.println("下标为0的值修改后-stringlists：" + shardedJedis.lrange("stringlists", 0, -1));
	System.out.println("=============查=============");
	// 数组长度
	System.out.println("长度-stringlists：" + shardedJedis.llen("stringlists"));
	System.out.println("长度-numberlists：" + shardedJedis.llen("numberlists"));
	// 排序
	/*
	 * list中存字符串时必须指定参数为alpha，如果不使用SortingParams，而是直接使用sort("list")，
	 * 会出现"ERR One or more scores can't be converted into double"
	 */
	SortingParams sortingParameters = new SortingParams();
	sortingParameters.alpha();
	sortingParameters.limit(0, 3);
	System.out.println("返回排序后的结果-stringlists：" + shardedJedis.sort("stringlists", sortingParameters));
	System.out.println("返回排序后的结果-numberlists：" + shardedJedis.sort("numberlists"));
	// 子串： start为元素下标，end也为元素下标；-1代表倒数一个元素，-2代表倒数第二个元素
	System.out.println("子串-第二个开始到结束：" + shardedJedis.lrange("stringlists", 1, -1));
	// 获取列表指定下标的值
	System.out.println("获取下标为2的元素：" + shardedJedis.lindex("stringlists", 2) + "\n");
}
```
## set
```java
/**
 * Set功能
 * 1.
 * 2.
 * 3.
 * 4.
 * 5.
 * 6.
 * 7.
 * 8.
 * 
 */
private void SetOperate() {

	System.out.println("======================set==========================");
	// 清空数据
	System.out.println("清空库中所有数据：" + jedis.flushDB());

	System.out.println("=============增=============");
	System.out.println("向sets集合中加入元素element001：" + jedis.sadd("sets", "element001"));
	System.out.println("向sets集合中加入元素element002：" + jedis.sadd("sets", "element002"));
	System.out.println("向sets集合中加入元素element003：" + jedis.sadd("sets", "element003"));
	System.out.println("向sets集合中加入元素element004：" + jedis.sadd("sets", "element004"));
	System.out.println("查看sets集合中的所有元素:" + jedis.smembers("sets"));
	System.out.println();

	System.out.println("=============删=============");
	System.out.println("集合sets中删除元素element003：" + jedis.srem("sets", "element003"));
	System.out.println("查看sets集合中的所有元素:" + jedis.smembers("sets"));
	/*
	 * System.out.println("sets集合中任意位置的元素出栈："+jedis.spop("sets"));//注：
	 * 出栈元素位置居然不定？--无实际意义
	 * System.out.println("查看sets集合中的所有元素:"+jedis.smembers("sets"));
	 */
	System.out.println();

	System.out.println("=============改=============");
	System.out.println();

	System.out.println("=============查=============");
	System.out.println("判断element001是否在集合sets中：" + jedis.sismember("sets", "element001"));
	System.out.println("循环查询获取sets中的每个元素：");
	Set<String> set = jedis.smembers("sets");
	Iterator<String> it = set.iterator();
	while (it.hasNext()) {
		Object obj = it.next();
		System.out.println(obj);
	}
	System.out.println();

	System.out.println("=============集合运算=============");
	System.out.println("sets1中添加元素element001：" + jedis.sadd("sets1", "element001"));
	System.out.println("sets1中添加元素element002：" + jedis.sadd("sets1", "element002"));
	System.out.println("sets1中添加元素element003：" + jedis.sadd("sets1", "element003"));
	System.out.println("sets1中添加元素element002：" + jedis.sadd("sets2", "element002"));
	System.out.println("sets1中添加元素element003：" + jedis.sadd("sets2", "element003"));
	System.out.println("sets1中添加元素element004：" + jedis.sadd("sets2", "element004"));
	System.out.println("查看sets1集合中的所有元素:" + jedis.smembers("sets1"));
	System.out.println("查看sets2集合中的所有元素:" + jedis.smembers("sets2"));
	System.out.println("sets1和sets2交集：" + jedis.sinter("sets1", "sets2"));
	System.out.println("sets1和sets2并集：" + jedis.sunion("sets1", "sets2"));
	System.out.println("sets1和sets2差集：" + jedis.sdiff("sets1", "sets2"));// 差集：set1中有，set2中没有的元素

}
```

## SortedSet
```java
/**
 * SortedSet功能
 * 1.
 * 2.
 * 3.
 * 4.
 * 5.
 * 6.
 * 7.
 * 8.
 * 
 */
private void SortedSetOperate() {
	System.out.println("======================zset==========================");
	// 清空数据
	System.out.println(jedis.flushDB());

	System.out.println("=============增=============");
	System.out.println("zset中添加元素element001：" + shardedJedis.zadd("zset", 7.0, "element001"));
	System.out.println("zset中添加元素element002：" + shardedJedis.zadd("zset", 8.0, "element002"));
	System.out.println("zset中添加元素element003：" + shardedJedis.zadd("zset", 2.0, "element003"));
	System.out.println("zset中添加元素element004：" + shardedJedis.zadd("zset", 3.0, "element004"));
	System.out.println("zset集合中的所有元素：" + shardedJedis.zrange("zset", 0, -1));// 按照权重值排序
	System.out.println();

	System.out.println("=============删=============");
	System.out.println("zset中删除元素element002：" + shardedJedis.zrem("zset", "element002"));
	System.out.println("zset集合中的所有元素：" + shardedJedis.zrange("zset", 0, -1));
	System.out.println();

	System.out.println("=============改=============");
	System.out.println();

	System.out.println("=============查=============");
	System.out.println("统计zset集合中的元素中个数：" + shardedJedis.zcard("zset"));
	System.out.println("统计zset集合中权重某个范围内（1.0——5.0），元素的个数：" + shardedJedis.zcount("zset", 1.0, 5.0));
	System.out.println("查看zset集合中element004的权重：" + shardedJedis.zscore("zset", "element004"));
	System.out.println("查看下标1到2范围内的元素值：" + shardedJedis.zrange("zset", 1, 2));

}
```
	
## hash
```java
/**
 * Hash功能
 * 1.
 * 2.
 * 3.
 * 4.
 * 5.
 * 6.
 * 7.
 * 8.
 * 
 */
private void HashOperate() {
	System.out.println("======================hash==========================");
	// 清空数据
	System.out.println(jedis.flushDB());

	System.out.println("=============增=============");
	System.out.println("hashs中添加key001和value001键值对：" + shardedJedis.hset("hashs", "key001", "value001"));
	System.out.println("hashs中添加key002和value002键值对：" + shardedJedis.hset("hashs", "key002", "value002"));
	System.out.println("hashs中添加key003和value003键值对：" + shardedJedis.hset("hashs", "key003", "value003"));
	System.out.println("新增key004和4的整型键值对：" + shardedJedis.hincrBy("hashs", "key004", 4l));
	System.out.println("hashs中的所有值：" + shardedJedis.hvals("hashs"));
	System.out.println();

	System.out.println("=============删=============");
	System.out.println("hashs中删除key002键值对：" + shardedJedis.hdel("hashs", "key002"));
	System.out.println("hashs中的所有值：" + shardedJedis.hvals("hashs"));
	System.out.println();

	System.out.println("=============改=============");
	System.out.println("key004整型键值的值增加100：" + shardedJedis.hincrBy("hashs", "key004", 100l));
	System.out.println("hashs中的所有值：" + shardedJedis.hvals("hashs"));
	System.out.println();

	System.out.println("=============查=============");
	System.out.println("判断key003是否存在：" + shardedJedis.hexists("hashs", "key003"));
	System.out.println("获取key004对应的值：" + shardedJedis.hget("hashs", "key004"));
	System.out.println("批量获取key001和key003对应的值：" + shardedJedis.hmget("hashs", "key001", "key003"));
	System.out.println("获取hashs中所有的key：" + shardedJedis.hkeys("hashs"));
	System.out.println("获取hashs中所有的value：" + shardedJedis.hvals("hashs"));
	System.out.println();

}
```
	
# 操作命令
## 连接操作命令
quit：关闭连接（connection）
auth：简单密码认证
help cmd： 查看cmd帮助，例如：help quit

## 持久化
save：将数据同步保存到磁盘
bgsave：将数据异步保存到磁盘
lastsave：返回上次成功将数据保存到磁盘的Unix时戳
shundown：将数据同步保存到磁盘，然后关闭服务

## 远程服务控制
info：提供服务器的信息和统计
monitor：实时转储收到的请求
slaveof：改变复制策略设置
config：在运行时配置Redis服务器

## 对value操作的命令
exists(key)：确认一个key是否存在
del(key)：删除一个key
type(key)：返回值的类型
keys(pattern)：返回满足给定pattern的所有key
randomkey：随机返回key空间的一个
keyrename(oldname, newname)：重命名key
dbsize：返回当前数据库中key的数目
expire：设定一个key的活动时间（s）
ttl：获得一个key的活动时间
select(index)：按索引查询
move(key, dbindex)：移动当前数据库中的key到dbindex数据库
flushdb：删除当前选择数据库中的所有key
flushall：删除所有数据库中的所有key

## String
set(key, value)：给数据库中名称为key的string赋予值value
get(key)：返回数据库中名称为key的string的value
getset(key, value)：给名称为key的string赋予上一次的value
mget(key1, key2,…, key N)：返回库中多个string的value
setnx(key, value)：添加string，名称为key，值为value
setex(key, time, value)：向库中添加string，设定过期时间time
mset(key N, value N)：批量设置多个string的值
msetnx(key N, value N)：如果所有名称为key i的string都不存在
incr(key)：名称为key的string增1操作
incrby(key, integer)：名称为key的string增加integer
decr(key)：名称为key的string减1操作
decrby(key, integer)：名称为key的string减少integer
append(key, value)：名称为key的string的值附加value
substr(key, start, end)：返回名称为key的string的value的子串

## List 
rpush(key, value)：在名称为key的list尾添加一个值为value的元素
lpush(key, value)：在名称为key的list头添加一个值为value的 元素
llen(key)：返回名称为key的list的长度
lrange(key, start, end)：返回名称为key的list中start至end之间的元素
ltrim(key, start, end)：截取名称为key的list
lindex(key, index)：返回名称为key的list中index位置的元素
lset(key, index, value)：给名称为key的list中index位置的元素赋值
lrem(key, count, value)：删除count个key的list中值为value的元素
lpop(key)：返回并删除名称为key的list中的首元素
rpop(key)：返回并删除名称为key的list中的尾元素
blpop(key1, key2,… key N, timeout)：lpop命令的block版本。
brpop(key1, key2,… key N, timeout)：rpop的block版本。
rpoplpush(srckey, dstkey)：返回并删除名称为srckey的list的尾元素，并将该元素添加到名称为dstkey的list的头部

## Set
sadd(key, member)：向名称为key的set中添加元素member
srem(key, member) ：删除名称为key的set中的元素member
spop(key) ：随机返回并删除名称为key的set中一个元素
smove(srckey, dstkey, member) ：移到集合元素
scard(key) ：返回名称为key的set的基数
sismember(key, member) ：member是否是名称为key的set的元素
sinter(key1, key2,…key N) ：求交集
sinterstore(dstkey, (keys)) ：求交集并将交集保存到dstkey的集合
sunion(key1, (keys)) ：求并集
sunionstore(dstkey, (keys)) ：求并集并将并集保存到dstkey的集合
sdiff(key1, (keys)) ：求差集
sdiffstore(dstkey, (keys)) ：求差集并将差集保存到dstkey的集合
smembers(key) ：返回名称为key的set的所有元素
srandmember(key) ：随机返回名称为key的set的一个元素

## Hash
hset(key, field, value)：向名称为key的hash中添加元素field
hget(key, field)：返回名称为key的hash中field对应的value
hmget(key, (fields))：返回名称为key的hash中field i对应的value
hmset(key, (fields))：向名称为key的hash中添加元素field 
hincrby(key, field, integer)：将名称为key的hash中field的value增加integer
hexists(key, field)：名称为key的hash中是否存在键为field的域
hdel(key, field)：删除名称为key的hash中键为field的域
hlen(key)：返回名称为key的hash中元素个数
hkeys(key)：返回名称为key的hash中所有键
hvals(key)：返回名称为key的hash中所有键对应的value
hgetall(key)：返回名称为key的hash中所有的键（field）及其对应的value

# Spring Boot 使用RedisTemplate 存储键值出现乱码
最近使用spring-data-redis RedisTemplate 操作redis时发现存储在redis中的key不是设置的string值，前面还多出了许多类似\xac\xed\x00\x05t\x00这种字符串，如下
```
127.0.0.1:6379> keys *
1) "\xac\xed\x00\x05t\x00\x04pass"
2) "\xac\xed\x00\x05t\x00\x04name"
3) "name"
```

解决：使用StringRedisTemplate，而不是RedisTemplate。

# 通过notify-keyspace-events实现订单自动关闭功能
修改Redis配置文件，开启过期通知功能。
在配置文件中搜索`notify`，找到`# notify-keyspace-events Ex`这一行，将注释去掉。
同时注释掉下面一行`notify-keyspace-events ""`，
为了节约cup资源，事件通知默认是没有开启的，即`notify-keyspace-events ""`

修改之后在需要重启redis，然后通过cli查看配置是否生效，`config get notify-keyspace-events`，如果显示Ex，就生效了。

# Redis缓存穿透、缓存雪崩和缓存击穿
缓存处理流程
前台请求，后台先从缓存中取数据，取到直接返回结果，取不到时从数据库中取，数据库取到更新缓存，并返回结果，数据库也没取到，那直接返回空结果。

- 缓存穿透
缓存穿透是指缓存和数据库中都没有的数据，而用户不断发起请求，如发起为id为“-1”的数据或id为特别大不存在的数据。这时的用户很可能是攻击者，攻击会导致数据库压力过大。
解决方案：
接口层增加校验，如用户鉴权校验，id做基础校验，id<=0的直接拦截；
从缓存取不到的数据，在数据库中也没有取到，这时也可以将key-value对写为key-null，缓存有效时间可以设置短点，如30秒（设置太长会导致正常情况也没法使用）。这样可以防止攻击用户反复用同一个id暴力攻击。

- 缓存雪崩
缓存雪崩是指缓存中数据大批量到过期时间，而查询数据量巨大，引起数据库压力过大甚至down机。和缓存击穿不同的是，缓存击穿指并发查同一条数据，缓存雪崩是不同数据都过期了，很多数据都查不到从而查数据库。
解决方案：
缓存数据的过期时间设置随机，防止同一时间大量数据过期现象发生。
如果缓存数据库是分布式部署，将热点数据均匀分布在不同搞得缓存数据库中。
设置热点数据永远不过期。

- 缓存击穿
缓存击穿，是指一个key非常热点，在不停的扛着大并发，大并发集中对这一个点进行访问，当这个key在失效的瞬间，持续的大并发就穿破缓存，直接请求数据库，就像在一个屏障上凿开了一个洞。缓存击穿指并发查同一条数据。
解决方案：
设置热点数据永远不过期。
加互斥锁，互斥锁参考代码如下：
```java
public static String getData(String key) throws InterruptedException{
	//从缓存中读取数据
	String result = getDataFromRedis(key);
	//缓存中不存在数据
	if(null == result){
		//获取锁，获取成功，取数据库获取数据
		if(reenLock.tryLock()){
			//从数据库获取数据
			result = getDataFrom Mysql(key);
			//更新缓存数据
			if(null != result){
				setDataToCache(key,result);
			}
			//释放锁
			reenLock.unlock();
		}else{//获取锁失败
			//暂停100ms再重新获取数据
			Thread.sleep(100);
			result = getData(key);
		}
	}
	return result;
}
```
说明：
- 缓存中有数据，直接走上述代码13行后就返回结果了
- 缓存中没有数据，第1个进入的线程，获取锁并从数据库去取数据，没释放锁之前，其他并行进入的线程会等待100ms，再重新去缓存取数据。这样就防止都去数据库重复取数据，重复往缓存中更新数据情况出现。
- 当然这是简化处理，理论上如果能根据key值加锁就更好了，就是线程A从数据库取key1的数据并不妨碍线程B取key2的数据，上面代码明显做不到这点。

# redis哨兵
哨兵是一个独立的进程，作为进程，它会独立运行。其原理是哨兵通过发送命令，等待Redis服务器响应，从而监控运行的多个Redis实例。
![](redis/1.jpg)

通过发送命令，让Redis服务器返回监控其运行状态，包括主服务器和从服务器。
当哨兵监测到master宕机，会自动将slave切换成master，然后通过发布订阅模式通知其他的从服务器，修改配置文件，让它们切换主机。
然而一个哨兵进程对Redis服务器进行监控，可能会出现问题，为此，我们可以使用多个哨兵进行监控。各个哨兵之间还会进行监控，这样就形成了多哨兵模式。
故障切换（failover）的过程。假设主服务器宕机，哨兵1先检测到这个结果，系统并不会马上进行failover过程，仅仅是哨兵1主观的认为主服务器不可用，这个现象成为主观下线。当后面的哨兵也检测到主服务器不可用，并且数量达到一定值时，那么哨兵之间就会进行一次投票，投票的结果由一个哨兵发起，进行failover操作。切换成功后，就会通过发布订阅模式，让各个哨兵把自己监控的从服务器实现切换主机，这个过程称为客观下线。这样对于客户端而言，一切都是透明的。

配置3个哨兵和1主2从的Redis服务器来演示这个过程。

|服务类型|是否是主服务器|IP地址|端口|
|-|-|-|
|Redis|是|192.168.11.128|6379|
|Redis|否|192.168.11.129|6379|
|Redis|否|192.168.11.130|6379|
|Sentinel|-|192.168.11.128|26379|
|Sentinel|-|192.168.11.129|26379|
|Sentinel|-|192.168.11.130|26379|

![](redis/2.jpg)
首先配置Redis的主从服务器，修改redis.conf文件如下
```
# 使得Redis服务器可以跨网络访问
bind 0.0.0.0
# 设置密码
requirepass "123456"
# 指定主服务器，注意：有关slaveof的配置只是配置从服务器，主服务器不需要配置
slaveof 192.168.11.128 6379
# 主服务器密码，注意：有关slaveof的配置只是配置从服务器，主服务器不需要配置
masterauth 123456
```
上述内容主要是配置Redis服务器，从服务器比主服务器多一个slaveof的配置和密码。
配置3个哨兵，每个哨兵的配置都是一样的。在Redis安装目录下有一个sentinel.conf文件，copy一份进行修改。
```
# 禁止保护模式
protected-mode no
# 配置监听的主服务器，这里sentinel monitor代表监控，mymaster代表服务器的名称，可以自定义，192.168.11.128代表监控的主服务器，6379代表端口，2代表只有两个或两个以上的哨兵认为主服务器不可用的时候，才会进行failover操作。
sentinel monitor mymaster 192.168.11.128 6379 2
# sentinel author-pass定义服务的密码，mymaster是服务名称，123456是Redis服务器密码
# sentinel auth-pass <master-name> <password>
sentinel auth-pass mymaster 123456
```
上述关闭了保护模式，便于测试。
有了上述的修改，我们可以进入Redis的安装目录的src目录，通过下面的命令启动服务器和哨兵
```
# 启动Redis服务器进程
./redis-server ../redis.conf
# 启动哨兵进程
./redis-sentinel ../sentinel.conf
```
注意启动的顺序。首先是主机（192.168.11.128）的Redis服务进程，然后启动从机的服务进程，最后启动3个哨兵的服务进程。

## Java中使用哨兵模式
```java
/**
 * 测试Redis哨兵模式
 * @author liu
 */
public class TestSentinels {
    @SuppressWarnings("resource")
    @Test
    public void testSentinel() {
        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
        jedisPoolConfig.setMaxTotal(10);
        jedisPoolConfig.setMaxIdle(5);
        jedisPoolConfig.setMinIdle(5);
        // 哨兵信息
        Set<String> sentinels = new HashSet<>(Arrays.asList("192.168.11.128:26379",
                "192.168.11.129:26379","192.168.11.130:26379"));
        // 创建连接池
        JedisSentinelPool pool = new JedisSentinelPool("mymaster", sentinels,jedisPoolConfig,"123456");
        // 获取客户端
        Jedis jedis = pool.getResource();
        // 执行两个命令
        jedis.set("mykey", "myvalue");
        String value = jedis.get("mykey");
        System.out.println(value);
    }
}
```
上面是通过Jedis进行使用的，同样也可以使用Spring进行配置RedisTemplate使用。
```
<bean id = "poolConfig" class="redis.clients.jedis.JedisPoolConfig">
	<!-- 最大空闲数 -->
	<property name="maxIdle" value="50"></property>
	<!-- 最大连接数 -->
	<property name="maxTotal" value="100"></property>
	<!-- 最大等待时间 -->
	<property name="maxWaitMillis" value="20000"></property>
</bean>

<bean id="connectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
	<constructor-arg name="poolConfig" ref="poolConfig"></constructor-arg>
	<constructor-arg name="sentinelConfig" ref="sentinelConfig"></constructor-arg>
	<property name="password" value="123456"></property>
</bean>

<!-- JDK序列化器 -->
<bean id="jdkSerializationRedisSerializer" class="org.springframework.data.redis.serializer.JdkSerializationRedisSerializer"></bean>

<!-- String序列化器 -->
<bean id="stringRedisSerializer" class="org.springframework.data.redis.serializer.StringRedisSerializer"></bean>

<bean id="redisTemplate" class="org.springframework.data.redis.core.RedisTemplate">
	<property name="connectionFactory" ref="connectionFactory"></property>
	<property name="keySerializer" ref="stringRedisSerializer"></property>
	<property name="defaultSerializer" ref="stringRedisSerializer"></property>
	<property name="valueSerializer" ref="jdkSerializationRedisSerializer"></property>
</bean>

<!-- 哨兵配置 -->
<bean id="sentinelConfig" class="org.springframework.data.redis.connection.RedisSentinelConfiguration">
	<!-- 服务名称 -->
	<property name="master">
		<bean class="org.springframework.data.redis.connection.RedisNode">
			<property name="name" value="mymaster"></property>
		</bean>
	</property>
	<!-- 哨兵服务IP和端口 -->
	<property name="sentinels">
		<set>
			<bean class="org.springframework.data.redis.connection.RedisNode">
				<constructor-arg name="host" value="192.168.11.128"></constructor-arg>
				<constructor-arg name="port" value="26379"></constructor-arg>
			</bean>
			<bean class="org.springframework.data.redis.connection.RedisNode">
				<constructor-arg name="host" value="192.168.11.129"></constructor-arg>
				<constructor-arg name="port" value="26379"></constructor-arg>
			</bean>
			<bean class="org.springframework.data.redis.connection.RedisNode">
				<constructor-arg name="host" value="192.168.11.130"></constructor-arg>
				<constructor-arg name="port" value="26379"></constructor-arg>
			</bean>
		</set>
	</property>
</bean>
```
## 哨兵模式的其他配置项

|配置项|参数类型|作用|
|-|-|-|
|port|整数|启动哨兵进程端口|
|dir|文件夹目录 | 哨兵进程服务临时文件夹，默认为/tmp，要保证有可写入的权限|
|sentinel down-after-milliseconds | <服务名称><毫秒数（整数）> | 指定哨兵在监控Redis服务时，当Redis服务在一个默认毫秒数内都无法回答时，单个哨兵认为的主观下线时间，默认为30000（30秒）|
|sentinel parallel-syncs | <服务名称><服务器数（整数）> | 指定可以有多少个Redis服务同步新的主机，一般而言，这个数字越小同步时间越长，而越大，则对网络资源要求越高|
|sentinel failover-timeout | <服务名称><毫秒数（整数）> | 指定故障切换允许的毫秒数，超过这个时间，就认为故障切换失败，默认为3分钟|
|sentinel notification-script | <服务名称><脚本路径> | 指定sentinel检测到该监控的redis实例指向的实例异常时，调用的报警脚本。该配置项可选，比较常用|

sentinel down-after-milliseconds配置项只是一个哨兵在超过规定时间依旧没有得到响应后，会自己认为主机不可用。对于其他哨兵而言，并不是这样认为。哨兵会记录这个消息，当拥有认为主观下线的哨兵达到sentinel monitor所配置的数量时，就会发起一次投票，进行failover，此时哨兵会重写Redis的哨兵配置文件，以适应新场景的需要。


# Docker实现Redis主从配置、哨兵模式
使用docker下载redis镜像，默认下载最新版本。
`docker pull redis`

创建文件夹
```
cd /home
mkdir docker
cd docker
mkdir redis
cd redis

mkdir redis-6379-data
mkdir redis-6380-data
mkdir redis-6381-data
mkdir sentinel-26379-data
mkdir sentinel-26380-data
mkdir sentinel-26381-data
```

创建自定义配置文件
```
touch redis-6379.conf
touch redis-6380.conf
touch redis-6381.conf
```
其中redis-6379.conf是master服务器配置文件，redis-6380.conf、redis-6381.conf 是slave服务器配置文件。

编辑6379配置文件，输入如下内容：
```
port 6379
logfile "redis-6379.log"
dir /data
appendonly yes
appendfilename appendonly.aof
masterauth 123456
requirepass 123456
```

编辑6380：
```
port 6380
logfile "redis-6380.log"
dir /data
appendonly yes
appendfilename appendonly.aof
slaveof 192.168.43.188 6379
masterauth 123456
requirepass 123456
```

编辑6381：
```
port 6381
logfile "redis-6381.log"
dir /data
appendonly yes
appendfilename appendonly.aof
slaveof 192.168.43.188 6379
masterauth 123456
requirepass 123456
```

然后启动三个redis容器：
```
docker run -p 6379:6379 --net=host --restart=always --name redis-6379 -v /home/docker/redis/redis-6379.conf:/etc/redis/redis-6379.conf -v /home/docker/redis/redis-6379-data:/data -d redis redis-server /etc/redis/redis-6379.conf
docker run -p 6380:6380 --net=host --restart=always --name redis-6380 -v /home/docker/redis/redis-6380.conf:/etc/redis/redis-6380.conf -v /home/docker/redis/redis-6380-data:/data -d redis redis-server /etc/redis/redis-6380.conf
docker run -p 6381:6381 --net=host --restart=always --name redis-6381 -v /home/docker/redis/redis-6381.conf:/etc/redis/redis-6381.conf -v /home/docker/redis/redis-6381-data:/data -d redis redis-server /etc/redis/redis-6381.conf
```
命令解释：
`-v /home/docker/redis/redis-6379-data:/data`
该命令是将redis容器中，redis配置文件中指定的数据存储目录/data下文件的内容共享到宿主机/home/docker/redis/redis-6379-data目录下。
为何必须挂载`/data`目录？有状态容器都有数据持久化需求，在容器的生命周期内，数据持久化是持续的，包括容器在被停止后，但当容器被删除后，数据也随之被删除了，因此Docker 采用 volume （卷）的形式来向容器提供持久化存储，如果不设置该命令，数据库中的数据会默认保存在redis容器中的/data目录下，这样当执行 docker rm 容器id/容器name 命令，会丢失数据库中的数据。

查看容器状态：
`docker ps -a`

如果STATUS状态是Up表示成功启动容器，如果STATUS状态是Exited (1) 或者 Restarting (1) 表示未能正常启动，这时候我们需要查看redis容器日志修改配置文件内容重新运行容器
`docker logs redis-6379|less`
查找出问题后停止redis容器
`docker stop redis-6379`
删除redis容器
`docker rm redis-6379`
然后重新执行步骤6。

启动redis容器后，分别观察三个redis容器内部情况
`docker exec -it redis-6379 /bin/bash`
`redis-cli`
`127.0.0.1:6379> auth 123456`
`info replication`

master会有如下显示
```
# Replication
role:master
connected_slaves:2
slave0:ip=172.17.0.1,port=6379,state=online,offset=707,lag=0
slave1:ip=172.17.0.1,port=6379,state=online,offset=707,lag=0
master_replid:db3135b2f4cba8e6c1eb4be290b9dbfc2ec3b6d0
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:707
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:707
```

slave显示：
```
role:slave
master_host:192.168.43.188
master_port:6379
master_link_status:up
master_last_io_seconds_ago:8
master_sync_in_progress:0
slave_repl_offset:833
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:db3135b2f4cba8e6c1eb4be290b9dbfc2ec3b6d0
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:833
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:833
```

看到主机role:master、connected_slaves:2，两台从机role:slave、master_host有地址，可以确认redis主从模式已经建立成功。
下面可以使用`RedisDesktopManager`或分别进入各个redis容器，来测试数据是否同步。测试过程我就略了。

## 哨兵模式
主机上创建哨兵配置文件
```
touch sentinel-26379.conf
touch sentinel-26380.conf
touch sentinel-26381.conf
```
编辑sentinel-26379.conf
```
port 26379
dir "/data"
logfile "sentinel-26379.log"
sentinel monitor mymaster 192.168.43.188 6379 2
sentinel down-after-milliseconds mymaster 10000
sentinel failover-timeout mymaster 60000
sentinel auth-pass mymaster 123456
```
编辑sentinel-26380.conf
```
port 26380
dir "/data"
logfile "sentinel-26380.log"
sentinel monitor mymaster 192.168.43.188 6379 2
sentinel down-after-milliseconds mymaster 10000
sentinel failover-timeout mymaster 60000
sentinel auth-pass mymaster 123456
```
编辑sentinel-26381.conf
```
port 26381
dir "/data"
logfile "sentinel-26381.log"
sentinel monitor mymaster 192.168.43.188 6379 2
sentinel down-after-milliseconds mymaster 10000
sentinel failover-timeout mymaster 60000
sentinel auth-pass mymaster 123456
```

启动哨兵：
```
docker run -p 26379:26379 --restart=always --name sentinel-26379 -v/home/docker/redis/sentinel-26379.conf:/etc/redis/sentinel.conf -v /home/docker/redis/sentinel-26379-data:/data -d redis redis-sentinel /etc/redis/sentinel.conf
docker run -p 26380:26380 --restart=always --name sentinel-26380 -v/home/docker/redis/sentinel-26380.conf:/etc/redis/sentinel.conf -v /home/docker/redis/sentinel-26380-data:/data -d redis redis-sentinel /etc/redis/sentinel.conf
docker run -p 26381:26381 --restart=always --name sentinel-26381 -v/home/docker/redis/sentinel-26381.conf:/etc/redis/sentinel.conf -v /home/docker/redis/sentinel-26381-data:/data -d redis redis-sentinel /etc/redis/sentinel.conf
```
查看哨兵容器状态
`docker ps -a`

至此，哨兵模式已经建立起来，查看哨兵信息
```
[root@localhost redis]# docker exec -it sentinel-26379 /bin/bash
root@zhangqian527halbin:/data# redis-cli -p 26379
127.0.0.1:26379>  info sentinel
# Sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=192.168.43.188:6379,slaves=2,sentinels=3
```
可以看到，现在端口号为:6379的redis服务器是master服务器，这时我们关闭端口号为6379的redis-6379容器测试哨兵配置是否生效
`docker stop redis-6379`

然后再查看哨兵信息，哨兵模式下，当master服务器宕机之后，哨兵自动会在从slave redis服务器里面投票选举一个master服务器来，这个master服务器也可以进行读写操作
```
[root@localhost redis]# docker exec -it sentinel-26379 /bin/bash
root@zhangqian527halbin:/data# redis-cli -p 26379
127.0.0.1:26379>  info sentinel
sentinel_masters:1
sentinel_tilt:0
sentinel_running_scripts:0
sentinel_scripts_queue_length:0
sentinel_simulate_failure_flags:0
master0:name=mymaster,status=ok,address=192.168.43.188:6380,slaves=2,sentinels=3
```
可以看到端口号为:6380的redis服务器已经变成master服务器了，说明哨兵配置成功。

[参考](https://www.jianshu.com/p/ce1d78cd368a)

# Redis SETNX 命令实现分布式锁
起因：
最开始我们是单服务，注册时候总是出现重复的手机号用户。
于是加了同步锁`synchronized`，解决了问题。

后来用了阿里云SLB分布式服务。又出现了重复注册的问题，想到用redis的SETNX来解决问题。
现在分布式的应用场景很多，为了保持数据的一致性，经常碰到需要对资源加锁的情形。利用redis来实现分布式锁就是其中的一种实现方案。
命令各式：
`SETNX key value`

将 key 的值设为 value ，当且仅当 key 不存在。
若给定的 key 已经存在，则 SETNX 不做任何动作。
SETNX 是『SET if Not eXists』(如果不存在，则 SET)的简写。
设置成功，返回1。
设置失败，返回0。
利用SETNX的特性，很容易的想到，在需要加锁的时候，调用SETNX命令，如果返回了1，表示设置成功，获得了当前锁，之后做一些想要的操作，完成之后调用DEL命令释放锁。
因为java的try/catch特性，即使程序报错，我们也可以在finally块释放锁，这样不会因为程序报错导致所无法释放的情况。
示例代码：
```
try {
	Boolean result = redisTemplate.opsForValue().setIfAbsent("reg_" + newUser.getMobile(),newUser.getMobile(),30,TimeUnit.SECONDS);
	if(result){
		//插入新用户
		usersMapper.insertSelective(newUser);
		redisTemplate.delete("reg_" + newUser.getMobile());
	}
} catch (java.lang.Exception e) {
	e.printStackTrace();
} finally {
	redisTemplate.delete("reg_" + newUser.getMobile());
}
```
幂等性：
HTTP/1.1中对幂等性的定义是：一次和多次请求某一个资源对于资源本身应该具有同样的结果（网络超时等问题除外）。也就是说，其任意多次执行对资源本身所产生的影响均与一次执行的影响相同。

很多复杂的事务要分布进行，但它们组成了一个整体，要么整体生效，要么整体失效。这种思想反应到数据库上，就是多条SQL语句，要么所有执行成功，要么所有执行失败。

数据库事务由严格的定义，它必须满足4个特性：
原子性(Atomicity),一致性(consistency),隔离性(Isolation),持久性(Durability)。

原子性：
表示组成一个事务的多个数据库操作是一个不可分割的原子单元，只有所有的操作执行成功，整个事务才提交。事务中的任何一个数据库操作失败，已经执行的任何操作都必须被撤销，让数据库返回初始状态。

一致性：
事务操作成功后，数据库所处的状态和他的业务规则是一致的，即数据不会被破坏。如A账户转账100元到B账户，不管操作成功与否，A和B账户的存款总额是不变的。

隔离性：
在并发数据操作时，不同的事务拥有各自的数据空间，他们的操作不会对对方产生敢逃。准确地说，并非要求做到完全无干扰。数据库规定了多种事务隔离界别，不同的隔离级别对应不用的干扰成都，隔离级别越高，数据一致性越好，但并发行越弱。

持久性：
一旦事务提交成功后，事务中所有的数据操作都必须被持久化到数据库中。即使在事务提交后，数据库马上崩溃，在数据库重启时，也必须保证能够通过某种机制恢复数据。
在这些事务特性中，数据“一致性”时最终目标，其他特性都是为达到这个目标而采取的措施、要求或手段。
数据库管理系统一般采用重执行日志来保证原子性、一致性和持久性。重执行日志记录了数据库变化的每一个动作，数据库在一个事务中执行一部分操作后发生错误退出，数据库即可根据重执行日志撤销已经执行的操作。对于已经提交的事务即使数据库崩溃，在重启数据库时也能后根据日志对尚未持久化的数据进行相应的重执行操作。

[参考](https://blog.csdn.net/harleylau/article/details/85856774)