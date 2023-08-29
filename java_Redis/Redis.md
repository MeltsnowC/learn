# Redis

Redis 是一种基于内存的数据库，对数据的读写操作都是在内存中完成，因此**读写速度非常快**，常用于**缓存，消息队列、分布式锁等场景**。

Redis 提供了多种数据类型来支持不同的业务场景，比如 String(字符串)、Hash(哈希)、 List (列表)、Set(集合)、Zset(有序集合)、Bitmaps（位图）、HyperLogLog（基数统计）、GEO（地理信息），并且对数据类型的操作都是**原子性**的，因为执行命令由单线程负责的，不存在并发竞争的问题。

除此之外，Redis 还支持**事务 、持久化、Lua 脚本、多种集群方案（主从复制模式、哨兵模式）、发布/订阅模式，过期删除机制**等等

## 一、相关命令

|      redis启动和关闭      |           `/redis-server /etc/Myredis/redis.conf`            |
| :-----------------------: | :----------------------------------------------------------: |
|          cli登录          |          `redis-cli -p 6379`  <br />  `author 密码`          |
|        查看进程：         |                    `ps -ef  | grep redis`                    |
|         杀死进程          |                       `kill -9 进程号`                       |
|       关闭redis进程       | `redis-cli -p 6379 shutdown`<br/>	`redis-cli -a 密码 shutdown` |
| 查看当前数据库的key的数量 |                           `dbsize`                           |
|        清空当前库         |                          `flushall`                          |
|        切换数据库         |                        `select 0-15`                         |




## 二、Redis和Memcatched区别：

|          |  Memcatched  |           Redis            |
| :------: | :----------: | :------------------------: |
| 数据类型 | 单一数据类型 |         多数据类型         |
|  持久化  | 不支持持久化 | 支持持久化，可以保存至硬盘 |
|          |  多线程+锁   |     单线程+多路IO复用      |

memcatched不支持多数据类型，不支持持久化，只能保存在内存中，不能保存至硬盘，

memcatched使用的是多线程+锁的形式。
Redis是单线程+多路IO复用

### 多路IO复用

​	多路复用是指使用一个线程来检查多个文件描述符(Socket)的就绪状态，比如调用select和poll函数，传入多个文件描述符，如果有一个文件描述符就绪，则返回，否则阻塞直到超时。得到就绪状态后进行真正的操作可以在同一个线程里执行，也可以启动线程执行(比如使用线程池)

## 三、Key键操作

|         命令         |                             描述                             |
| :------------------: | :----------------------------------------------------------: |
|    keys  parttern    |        查看当前库中符合parttern的键，*表示取所有的键         |
| exists key [key1...] |              判断键是否存在，返回存在的键的个数              |
|       type key       |                       查看键key的类型                        |
|   del key [key...]   |                       删除指定的键值对                       |
|  unlink key [key..]  |                非阻塞删除键键值对（异步删除）                |
|  expire key second   |                  设置key的过期时间,单位是秒                  |
|       ttl key        | 查看key还有多久过期，-1永不过期(未设置过期时间就是-1)，-2已经过期 |
|      select did      |                          切换数据库                          |
|        dbsize        |                     查看当前库有多少key                      |
|       flushdb        |                          清空当前库                          |
|       flushall       |                          清空所有库                          |

## 四、常用数据类型

### 4.1 Redis字符串(String)

String是Redis最基本的数据类型，和memcatch的类型一样，一个key对应一个value

String类型是二进制安全的类型，意味着Redis的String可以包含任何数据，例如jpg图片或者序列号的对象

一个Redis中字符串value最大可以是512M

#### 4.1.1 常用命令

|                          命令                          |                             描述                             |
| :----------------------------------------------------: | :----------------------------------------------------------: |
| set key value [NX\|XX] [EX seconds \| PX milliseconds] | 将key-value存入redis,NX表示key不存在时，可以存入，否则返回nil; <br />XX表示key存在的时候，可以存入，覆盖之前的value值。<br />EX：设置key的超时秒数；PX：超时毫秒，与EX互斥 |
|                        get key                         |                       获取key对应的值                        |
|                   append key value1                    |    将value1追加到key原来的value末尾，返回现在的value长度     |
|                       strlen key                       |                    获取key对应value的长度                    |
|                    setNX key value                     |      只有当key不存在的时候，才添加key和value，返回0或1       |
|                       `incr key`                       | 将key中存储的value值自增1，只对integer操作，如果为空，新增为1 *<font color=red>**原子操作**</font>* |
|                       `decr key`                       | 将key中存储的value值自减1，只对integer操作*<font color=red>**原子操作**</font>* |
|                  incrby key increment                  | 将key中存储的value增加一个increment步长 *<font color=red>**原子操作**</font>* |
|                  decrby key decrement                  | 将key中存储的value值自减一个decrement步长 *<font color=red>**原子操作**</font>* |
|      mset key value [key1 value1 key2 value2 ...]      |                 同时设置一个或多个key value                  |
|                mget key [key1 key2...]                 |         同时获取多个key对应的value，不存在的返回nil          |
|            msetnx key value [key value...]             | 同时设置多个key value，仅当所有指定的key都不存在时才能添加成功<br /> *<font color=red>**原子性，一个失败都失败**</font>* |
|                 getrange key start end                 | 获取值的子串，从start开始到end结束，包含边界，数字类型的值也有效 |
|               setrange key offset value1               | 使用value1覆盖key所存储的value值，从offset开始，包含这个位置。例如：`k1=hello ;setrange k1 1 www的结果为k1=hwwwo` |
|                setex key seconds value                 |             设置key-value值的同时，设置过期时间              |
|                    getset key value                    |               以旧换新，设置新值的同时返回旧值               |

> 原子操作:zap:指不会被线程调度机制打断的操作；
>
> 这种操作一旦开始，就一直运行直到结束，中间不会有任何context switch(切换到另外一个线程)。
> 	（1）在单线程中，能够在单条指令中完成的操作都是原子操作，因为中断只发生在指令之间。
> 	（2）在多线程中，不能被其他进程(线程)打断的操作叫做原子操作。
> Redis的原子操作主要得益于Redis单线程
>
> > Java中的i++是否是原子操作？
> > i=0;两个线程分别对i执行++100次，值为多少？
> >
> > 答：不是原子操作，值为2-200之间都有可能。每个线程都有自己的线程缓存，每次写之前都会先从主内存中取值，然后修改，再写回主内存。

#### 4.1.2 数据结构

String的数据结构为简单动态字符串(Simple Dynamic String ,缩写SDS)，是可以修改的字符串，内部结构的实现类似与Java的ArrayList，采用预分配冗余空间的方式来减少内存的频繁分配。

![image-20230323183012266](E:\笔记\java_Redis\image\redis-4.1.2.png)

如图所示，内部为当前字符串实际分配的空间Capacity一般要高于实际字符串长度len,当字符串长度小于1M时，扩容都是加倍现有的空间，如果超过了1M，扩容时，只会多扩容1M的空间，需要注意的是字符串的长度最长为512M。

### 4.2 Redis列表(List)

单键多值，value是一个列表，

redis列表是一个简单的字符串列表，按照插入顺序排序，可以添加一个元素到列表的头部(左边)或者尾部(右边)。

它的底层实际是一个双向链表，对两端的操作性能很高，通过索引下标的操作中间的节点性能会较差。

![image](E:\笔记\java_Redis\image\redis-4.2.1.png)

#### 4.2.1 常用命令

|                   命令                   |                             描述                             |
| :--------------------------------------: | :----------------------------------------------------------: |
|       lpush key element [element]        |           从左边向key对应的队列中插入一个或多个值            |
|       rpush key element [element]        |           从右边向key对应的队列中插入一个或多个值            |
|              lpop key count              | 取出key对应的最左边的值，count表示取出几个值，<br />如果count大于key对应列表的大小，则会取出列表中所有元素。<br />当取出所有元素后，key也将不存在了，<br /><font color=red>值在键在，值尽键亡</font>，不指定count时，默认取出一个元素 |
|              rpop key count              | 取出key对应的最右边的count个值，如果count大于列表大小，<br />取出所有值，<font color=red>值在键在，值尽键亡</font>不指定count时，默认取出一个元素 |
|       rpoplpush source destination       | 从source键的右侧取出一个值然后从左侧添加到destination键对应的列表 |
|           lrange key start end           | 按照索引下标获取元素(从左到右)  0表示左边第一个，-1表示右边第一个 |
|             lindex key index             | 按照索引下标index获取key对应列表中指定索引位置的元素(从左到右) |
|                 llen key                 | 获取key对应列表的长度(只能用于计算列表长度，不能计算字符串值) |
| linsert key BEFORE\|AFTER value newvalue | 在key对应的列表中，将newvalue插入到value之前或之后。<br />返回值是新的长度，失败时返回-1 |
|          lrem key count element          | 从key的列表中从左边删除count个element，(可用于删除重复元素)，返回成功删除的个数 |
|           lset key index value           |      将key中索引为index的值更改为value。也就是替换操作       |

#### 4.2.2 数据结构

list的数据结构是一个快速链表也叫quickList

首先在列表元素较少的情况下会使用一块连续的内存存储，这个结构是ziplist，也就是压缩列表。

它将所有元素紧挨着一起存储，分配的是一块连续内存。

当数量比较多的时候，会改变为quickList.

因为普通的链表需要的附加指针空间太大，会比较浪费空间，比如这个列表中只存储int类型的数据，结构上还需要额外的两个指针prev和next。

![image](E:\笔记\java_Redis\image\redis-4.2.2.png)

Redis将链表和ziplist结合起来组成了quickList，也就是将多个zipList使用双向指针串起来使用，这样既满足了快捷的插入删除的特性，也不会出现太大 的空间冗余。

### 4.3 Redis集合(Set)

Redis set对外提供的功能与list类似是一个列表的功能，特殊之处在于，set可以自动排重，当需要存储列表数据，但又不希望包含重复数据时，set是一个很好的选择。且内部元素是无序的。并且提供了判断某个成员是否在一个set集合的重要接口。这也是list所不能提供的。

Redis的set是String类型的一个无序集合，底层其实是一个value为null的hash表，所以添加、删除、查找的复杂度都说O(1)

#### 4.3.1 常用命令

|              命令               |                            描述                            |
| :-----------------------------: | :--------------------------------------------------------: |
|     sadd key member[member]     |  将一个或多个member元素添加到集合key中，已经存在的被忽略   |
|          smembers key           |                   查看集合key中的所有值                    |
|      sismember key member       |    判断member是否是集合key中的元素，不是返回0，是返回1     |
|            scard key            |                  返回集合key中的元素个数                   |
|    srem key member [member]     |       删除key中的一个或多个元素，返回成功删除的个数        |
|         spop key count          |  随机从key的集合中取出count个元素，并在集合中删除对应元素  |
|      srandmember key count      | 随机从key的集合中取出count个元素，不会在集合中删除对应元素 |
| smove source destination member |     将集合source中的元素member移动到destination集合中      |
|       sinter key [key...]       |                 返回多个集合key的交集元素                  |
|       sunion key [key...]       |                 返回多个集合key的并集元素                  |
|       sdiff key [key1...]       |       返回集合key和其他集合的差集，即key中独有的元素       |

#### 4.3.2 数据结构

set的数据结构是dict字典，字典是用哈希表实现的。

Java中的HashSet的内部实现是使用的hashMap，只不过所有的value都指向同一个对象。

redis的set结构也一样，它的内部也使用hash结构，所有的value都指向同一个内布值。

### **4.4**  Redis哈希(Hash)

Redis hash 是一个键值对集合。

Redis hash是一个string类型的field和value的映射表，hash特别适合用于存储对象。

类似Java里面的Map<String,Object>

用户ID为查找的key，存储的value用户对象包含姓名，年龄，生日等信息，如果用普通的key/value结构来存储，主要有以下2种存储方式：

|                                                              |                                                              |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| <img src="E:\笔记\java_Redis\image\redis-4.4.1-1.png" alt="image-20230325150901541" style="zoom:90%;"  />  <br />缺点：每次修改用户的某个属性需要，先反序列化改好后再序列化回去。开销较大。 | ![image-20230325150932382](E:\笔记\java_Redis\image\redis-4.4.1-2.png)<br />用户ID数据冗余，数据关系分散 |
| ![image-20230325151828390](E:\笔记\java_Redis\image\redis-4.4.1_3.png) | **通过 key(用户ID) + field(属性标签) 就可以操作对应属性数据了，既不需要重复存储数据，也不会带来序列化和并发修改控制的问题** |

#### 4.4.1 常用命令

|                  命令                  |                             描述                             |
| :------------------------------------: | :----------------------------------------------------------: |
| hset  key field value [field value...] |      给集合key中的field键赋值为value，返回值为成功个数       |
|             hget key field             |                  查询集合key中field对应的值                  |
| hmset key field value [field value...] |                     和hset类似，批量添加                     |
|           hexsists key field           |                查看key中，给定的field是否存在                |
|               hkeys key                |            查看key中所有的field，返回所有的field             |
|               hvals key                |                     查看key中所有的value                     |
|      hincrby key field increment       |            为哈希表 key 中的域 field 的值加上增量            |
|         hsetnx key field value         | 将哈希表 key 中的域 field 的值设置为 value ，当且仅当域 field 不存在 |

#### 4.4.2 数据结构

Hash类型对应的数据结构是两种：ziplist（压缩列表），hashtable（哈希表）。当field-value长度较短且个数较少时，使用ziplist，否则使用hashtable。

### 4.5 Redis有序集合(Zset)

Redis有序集合zset与普通集合set非常相似，是一个没有重复元素的字符串集合。

不同之处是有序集合的每个成员都关联了一个**评分（score）**,这个评分（score）被用来按照***默认从最低分到最高分的方式排序***集合中的成员。集合的成员是唯一的，但是评分可以是重复了 。

因为元素是有序的, 所以你也可以很快的根据评分（score）或者次序（position）来获取一个范围的元素。

访问有序集合的中间元素也是非常快的,因此你能够使用有序集合作为一个没有重复成员的智能列表。

#### 4.5.1 常用命令

|                   命令                    |                             描述                             |
| :---------------------------------------: | :----------------------------------------------------------: |
|   zadd key score member [score member]    | 将一个或多个 member 元素及其 score 值加入到有序集 key 当中。 |
|    zrange key start stop [withscores]     | 返回有序集 key 中，下标在<start><stop>之间的元素,<br />带WITHSCORES，可以让分数一起和值返回到结果集 |
|  zrangebyscore key min max [withscores]   | 返回有序集 key 中，所有 score 值介于 min 和 max 之间(包括等于<br /> min 或 max )的成员。有序集成员按 score 值递增(从小到大)次序排列。 |
| zrevrangebyscore key max min [withscores] |                  同上，结果为从大到小排列。                  |
|       zincrby key increment member        |    为元素的score加上增量increment ，返回值为修改后的score    |
|         zrem key member [member]          |        删除集合key中的元素member。返回成功删除的个数         |
|            zcount key min max             |            统计集合key中，在分数区间内的元素个数             |
|              zrank key value              |      返回集合key中，value的排名，从0开始，0是分数最低的      |

#### 4.5.2 数据结构

SortedSet(zset)是Redis提供的一个非常特别的数据结构，一方面它等价于Java的数据结构Map<String, Double>，可以给每一个元素value赋予一个权重score，另一方面它又类似于TreeSet，内部的元素会按照权重score进行排序，可以得到每个元素的名次，还可以通过score的范围来获取元素的列表。

zset底层使用了两个数据结构

（1）hash，hash的作用就是关联元素value和权重score，保障元素value的唯一性，可以通过元素value找到相应的score值。

（2）跳跃表，跳跃表的目的在于给元素value排序，根据score的范围获取元素列表。

##### 4.5.2.1 跳跃表

其实就是使用二分查找的链表。

> 详情可以参考：
>
> [跳跃表详解 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/260075092)
>
> [(58条消息) 跳跃表的原理及实现_跳跃表原理_强迫症Kobe的博客-CSDN博客](https://blog.csdn.net/qpzkobe/article/details/80056807)

有序集合在生活中比较常见，例如根据成绩对学生排名，根据得分对玩家排名等。对于有序集合的底层实现，可以用数组、平衡树、链表等。数组不便元素的插入、删除；平衡树或红黑树虽然效率高但结构复杂；链表查询需要遍历所有效率低。Redis采用的是跳跃表。跳跃表效率堪比红黑树，实现远比红黑树简单。

对比有序链表和跳跃表，从链表中查询出51

（1） 有序链表                         

要查找值为51的元素，需要从第一个元素开始依次查找、比较才能找到。共需要6次比较。

（2） 跳跃表

 ![image](E:\笔记\java_Redis\image\redis-4.5.2.1.png)

从第2层开始，1节点比51节点小，向后比较。

21节点比51节点小，继续向后比较，后面就是NULL了，所以从21节点向下到第1层

在第1层，41节点比51节点小，继续向后，61节点比51节点大，所以从41向下

在第0层，51节点为要查找的节点，节点被找到，共查找4次。

从此可以看出跳跃表比有序链表效率要高

## 五、Redis发布和订阅

### 5.1 定义

Redis 发布订阅 (pub/sub) 是一种消息通信模式：发送者 (pub) 发送消息，订阅者 (sub) 接收消息。

Redis 客户端可以订阅任意数量的频道。

下图展示了频道 channel1 ， 以及订阅这个频道的三个客户端 —— client2 、 client5 和 client1 之间的关系：

![image](https://www.runoob.com/wp-content/uploads/2014/11/pubsub1.png)

当有新消息通过 PUBLISH 命令发送给频道 channel1 时， 这个消息就会被发送给订阅它的三个客户端：

![image](https://www.runoob.com/wp-content/uploads/2014/11/pubsub2.png)

### 5.2 示例

开启两个redis-cli客户端

<img src="E:\笔记\java_Redis\image\redis-5.2.1.png" alt="image" style="zoom:70%;" /><img src="E:\笔记\java_Redis\image\redis-5.2.2.png" alt="image" style="zoom:80%;" />

让一个客户端订阅频道channel1；

![image](E:\笔记\java_Redis\image\redis-5.2.3.png)

打开另一个客户端，给channel1发布消息hello，得到的返回值是订阅者的数量

![image](E:\笔记\java_Redis\image\redis-5.2.4.png)

打开第一个客户端可以看到发送的消息

![image](E:\笔记\java_Redis\image\redis-5.2.5.png)

> 注：发布的消息没有持久化，正在订阅的客户端收不到hello，只能收到订阅后发布的消息

## 六、新数据类型

### 6.1 bitmap

#### 6.1.1 简介

现代计算机用二进制（位） 作为信息的基础单位， 1个字节等于8位， 例如“abc”字符串是由3个字节组成， 但实际在计算机存储时将其用二进制表示， “abc”分别对应的ASCII码分别是97、 98、 99， 对应的二进制分别是01100001、 01100010和01100011，如下图

![image](E:\笔记\java_Redis\image\redis-6.1.1_1.png)

合理地使用操作位能够有效地提高内存使用率和开发效率。

​    Redis提供了Bitmaps这个“数据类型”可以实现对位的操作：

（1）  Bitmaps本身不是一种数据类型， 实际上它就是字符串（key-value） ， 但是它可以对字符串的位进行操作。

（2）  Bitmaps单独提供了一套命令， 所以在Redis中使用Bitmaps和使用字符串的方法不太相同。 可以把Bitmaps想象成一个以位为单位的数组， 数组的每个单元只能存储0和1， 数组的下标在Bitmaps中叫做偏移量。

![image](E:\笔记\java_Redis\image\redis-6.1.1_2.png)

#### 6.1.2 常用命令

参考[Redis Setbit 命令_对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit)。](https://www.redis.net.cn/order/3550.html)

|                 命令                  |                             描述                             |
| :-----------------------------------: | :----------------------------------------------------------: |
|        setbit key offset value        | Setbit 命令用于对 key 所储存的字符串值，设置或清除指定偏移量上的位(bit),<br />将指定位上的值设置为0或者1,返回值是指定偏移量原来储存的位。 |
|           getbit key offset           | Getbit 命令用于对 key 所储存的字符串值，获取指定偏移量上的位(bit)。（从第0位开始算） |
|   bitcount key star end [bit\|byte]   | 统计字符串被设置为1的bit数. 返回值为被设置为 1 的位的数量。  |
| BITOP operation destkey key [key ...] | 对一个或多个保存二进制位的字符串 key 进行位元操作，<br />并将结果保存到 destkey 上.支持 AND 、 OR 、 NOT 、 XOR ,<br />返回值为保存到 destkey 的字符串的长度，和输入 key 中最长的字符串长度相等。 |

#### 6.1.3 示例

setbit举例：

每个独立用户是否访问过网站存放在Bitmaps中， 将访问的用户记做1， 没有访问的用户记做0， 用户的id作为偏移量。

设置键的第offset个位的值（从0算起） ， 假设现在有20个用户，userid=1， 6， 11， 15， 19的用户对网站进行了访问， 那么当前Bitmaps初始化结果如图

![image](E:\笔记\java_Redis\image\redis-6.1.3_1.png)

bittest:user:20221102 表示2022-11-02这一天访问网站的用户的Bitmaps 对于的key。

![image](E:\笔记\java_Redis\image\redis-6.1.3_2.png)

> 注：
>
> 很多应用的用户id以一个指定数字（例如10000） 开头， 直接将用户id和Bitmaps的偏移量对应势必会造成一定的浪费， 通常的做法是每次做setbit操作时将用户id减去这个指定数字。
>
> 在第一次初始化Bitmaps时， 假如偏移量非常大， 那么整个初始化过程执行会比较慢， 可能会造成Redis的阻塞

---

getbit举例：

获取id=19以及id=5的用户是否在2022-11-02这天访问过， 返回0说明没有访问过，返回1说明访问过：

![image](E:\笔记\java_Redis\image\redis-6.1.3_3.png)

> 因为用户5根本不存在，所以也是返回0

bitcount举例：

计算2022-11-02这天的独立访问用户数量

![image](E:\笔记\java_Redis\image\redis-6.1.3_4.png)

bitop举例：

计算2022-11-02和04两天共同的访问用户数量

![image](E:\笔记\java_Redis\image\redis-6.1.3_5.png)

#### 6.1.4 bitmap和set的对比

假设网站有1亿用户， 每天独立访问的用户有5千万， 如果每天用集合类型和Bitmaps分别存储活跃用户可以得到表

|          |                    | set和Bitmaps存储一天活跃用户对比 |                        |
| :------: | :----------------: | :------------------------------: | :--------------------: |
| 数据类型 | 每个用户id占用空间 |         需要存储的用户量         |       全部内存量       |
|   集合   |        64位        |             50000000             | 64位*50000000 = 400MB  |
|  bitmap  |        1位         |            100000000             | 1位*100000000 = 12.5MB |

很明显， 这种情况下使用Bitmaps能节省很多的内存空间， 尤其是随着时间推移节省的内存还是非常可观的

|          |        | set和Bitmaps存储独立用户空间对比 |       |
| :------: | :----: | :------------------------------: | :---: |
| 数据类型 |  一天  |               一月               | 一年  |
|   集合   | 400MB  |               12GB               | 144GB |
|  bitmap  | 12.5MB |              375MB               | 4.5GB |

但Bitmaps并不是万金油， 假如该网站每天的独立访问用户很少， 例如只有10万（大量的僵尸用户） ， 那么两者的对比如下表所示， 很显然， 这时候使用Bitmaps就不太合适了， 因为基本上大部分位都是0。

|          |                    | set和Bitmaps存储一天活跃用户对比（独立用户比较少） |                        |
| :------: | :----------------: | :------------------------------------------------: | :--------------------: |
| 数据类型 | 每个userid占用空间 |                  需要存储的用户量                  |       全部内存量       |
|   集合   |        64位        |                       100000                       |  64位*100000 = 800KB   |
|  bitmap  |        1位         |                     100000000                      | 1位*100000000 = 12.5MB |

### 6.2  HyperLogLog

Redis HyperLogLog 是用来做基数统计的算法，HyperLogLog 的优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。

在 Redis 里面，每个 HyperLogLog 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基 数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。

但是，因为 HyperLogLog 只会根据输入元素来计算基数，而不会储存输入元素本身，所以 HyperLogLog 不能像集合那样，返回输入的各个元素。

用于海量数据基数统计的场景，比如百万级网页 UV 计数等；

---

#### 6.2.1 什么是基数?

比如数据集 {1, 3, 5, 7, 5, 7, 8}， 那么这个数据集的基数集为 {1, 3, 5 ,7, 8}, 基数(不重复元素)为5（有五个不重复的元素）。 基数估计就是在误差可接受的范围内，快速计算基数。

#### 6.2.2 常用命令

|                   命令                    |                             描述                             |
| :---------------------------------------: | :----------------------------------------------------------: |
|      pfadd key element [element ...]      | Pfadd 命令将所有元素参数添加到 HyperLogLog 数据结构中。<br />返回值，整型，如果至少有个元素被添加返回 1， 否则返回 0。 |
|           PFCOUNT key [key ...]           | Pfcount 命令返回给定 HyperLogLog 的基数估算值。<br />返回值，整数，返回给定 HyperLogLog 的基数值，如果多个 HyperLogLog 则返回基数估值之和。 |
| PFMERGE destkey sourcekey [sourcekey ...] | Pgmerge 命令将多个 HyperLogLog 合并为一个 HyperLogLog ，合并后的 HyperLogLog 的基数估算值是通过对所有给定 HyperLogLog 进行并集计算得出的。返回ＯＫ． |

示例：

![image](E:\笔记\java_Redis\image\redis-6.2.2.png#pic_center)

### 6.3  Geospatial

Redis 3.2 中增加了对GEO类型的支持。GEO，Geographic，地理信息的缩写。该类型，就是元素的2维坐标，在地图上就是经纬度。redis基于该类型，提供了经纬度设置，查询，范围查询，距离查询，经纬度Hash等常见操作。



#### 6.3.1 常用命令

|                             命令                             |                             描述                             |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
|            geoadd key longtitude latitude member             | 将指定的地理空间位置（纬度、经度、名称）添加到指定的`key`中。这些数据将会存储到`sorted set`<br />返回值是添加到sorted set元素的数目，但不包括已更新score的元素。 |
|                GEOPOS key member [member ...]                | 从`key`里返回所有给定位置元素的位置（经度和纬度）。返回值是经纬度坐标，当不存在时，返回空 |
|              GEODIST key member1 member2 [unit]              | 返回两个给定位置之间的距离。计算出的距离会以双精度浮点数的形式被返回。如果两个位置之间的其中一个不存在， 那么命令返回空值。 |
| GEORADIUS key longitude latitude radius m\|km\|ft\|mi [WITHCOORD] [WITHDIST] [WITHHASH] [COUNT count] | 以给定的经纬度为中心， 返回键包含的位置元素当中， 与中心的距离不超过给定最大距离的所有位置元素。其实就是查找指定坐标为中心半径内的位置元素。 |

> geoadd 
>
> 该命令以采用标准格式的参数x,y,所以经度必须在纬度之前。这些坐标的限制是可以被编入索引的，区域面积可以很接近极点但是不能索引。区域限制如下：
>
> - 有效的经度从-180度到180度。
> - 有效的纬度从-85.05112878度到85.05112878度。
>
> 当坐标位置超出上述指定范围时，该命令将会返回一个错误。
>
> 已经添加的数据，是无法再次往里面添加的。

> geodist
>
> 返回两个给定位置之间的距离。
>
> 如果两个位置之间的其中一个不存在， 那么命令返回空值。
>
> 指定单位的参数 unit 必须是以下单位的其中一个：
>
> - **m** 表示单位为米。
> - **km** 表示单位为千米。
> - **mi** 表示单位为英里。
> - **ft** 表示单位为英尺。
>
> 如果用户没有显式地指定单位参数， 那么 `GEODIST` 默认使用米作为单位。
>
> `GEODIST` 命令在计算距离时会假设地球为完美的球形， 在极限情况下， 这一假设最大会造成 0.5% 的误差。

> georadius
>
> 在给定以下可选项时， 命令会返回额外的信息：
>
> - `WITHDIST`: 在返回位置元素的同时， 将位置元素与中心之间的距离也一并返回。 距离的单位和用户给定的范围单位保持一致。
> - `WITHCOORD`: 将位置元素的经度和维度也一并返回。
> - `WITHHASH`: 以 52 位有符号整数的形式， 返回位置元素经过原始 geohash 编码的有序集合分值。 这个选项主要用于底层应用或者调试， 实际中的作用并不大。
>
> 命令默认返回未排序的位置元素。 通过以下两个参数， 用户可以指定被返回位置元素的排序方式：
>
> - `ASC`: 根据中心的位置， 按照从近到远的方式返回位置元素。
> - `DESC`: 根据中心的位置， 按照从远到近的方式返回位置元素。

#### 6.3.2 示例

![image](E:\笔记\java_Redis\image\redis-6.3.2.png#pic_center)

## 七、Jedis操作redis

### 7.1 Jedis需要添加的依赖

```xml
<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>3.2.0</version>
</dependency>
```

### 7.2 连接Redis注意事项

> 禁用Linux的防火墙：Linux(CentOS7)里执行命令
>
> **systemctl stop/disable firewalld.service**  
>
> redis.conf中注释掉bind 127.0.0.1 ,然后 protected-mode no

**Java中使用Jedis来执行操作redis**

**测试相关操作：**

```java
@Test
    public void testjedis(){

        System.out.println("*************key操作***************");
        Jedis jedis = new Jedis("192.168.169.129", 6379);
        String auth = jedis.auth("991102");
        jedis.set("k1", "value1");
        String k1 = jedis.get("k1");
        System.out.println(k1);

        //get keys *
        System.out.println("************keys***************");
        Set<String> keys = jedis.keys("*");
        System.out.println(keys);
//        for (String key:keys){
//            System.out.println(key);
//        }

        //同时设置多个值
        System.out.println("*************mset*****************");
        jedis.mset("k2","value2","k3","value3");
        List<String> mget = jedis.mget("k2", "k3");
        System.out.println(mget);

        //list操作
        System.out.println("*************list***********");
        jedis.lpush("k4","value4","value4-2","value4-3");
        String k41 = jedis.lpop("k4");
        List<String> k4 = jedis.lrange("k4", 0, -1);
        System.out.println(k4);
        System.out.println(k41);
        jedis.lrem("k4",3,"value4-2");
        jedis.lrem("k4",3,"value4");

        //set操作
        System.out.println("************set*************");
        jedis.sadd("kset1","lucy","jack");
        Set<String> kset1 = jedis.smembers("kset1");
        System.out.println(kset1);
        jedis.srem("kset1","jack");
        System.out.println(jedis.smembers("kset1"));


        //hash操作
        System.out.println("***********hash***********");
        jedis.hset("hkey1","name","cheng");
        HashMap<String, String> map = new HashMap<>();
        map.put("age","20");
        map.put("email","cheng@163.com");
        String hkey1 = jedis.hget("hkey1", "name");
        //利用map添加多个hash：field-value
        jedis.hmset("hkey2",map);
        List<String> hkey2 = jedis.hmget("hkey2","age","email");
        System.out.println(hkey2);
        System.out.println(hkey1);

        //zset操作
        System.out.println("*******zset************");
        //添加单独有序集合
        jedis.zadd("zkey1",100d,"shanghai");
        jedis.zadd("zkey1",200d,"beijing");
        HashMap<String, Double> zmap = new HashMap<>();
        zmap.put("tianjin",150d);
        zmap.put("linfen",50d);
        //利用map类型添加多个值
        jedis.zadd("zkey1",zmap);
        Set<String> zset1 = jedis.zrange("zkey1", 0, -1);
        System.out.println(zset1);
        jedis.close();
    }
```

**运行结果：**

```java
*************key操作***************
value1
************keys***************
[hkey2, bittest:user:06-04, hkey1, newkey, kset1, another-program, zset1, zkey1, k1, bittest:user:20221102, k2, k3, bittest:user:20221106, program, china:city, topn]
*************mset*****************
[value2, value3]
*************list***********
[value4-2, value4]
value4-3
************set*************
[jack, lucy]
[lucy]
***********hash***********
[20, cheng@163.com]
cheng
*******zset************
[linfen, shanghai, tianjin, beijing]

进程已结束,退出代码0
```

## 八、利用jedis实现简单的手机验证码

要求：

1、输入手机号，点击发送后随机生成6位数字码，2分钟有效

2、输入验证码，点击验证，返回成功或失败

3、每个手机号每天只能输入3次

```java
import redis.clients.jedis.Jedis;

import java.util.Random;

/**
 * 1、输入手机号，点击发送后随机生成6位数字码，2分钟有效
 * 2、输入验证码，点击验证，返回成功或失败
 * 3、每个手机号每天只能输入3次
 */
public class phoneVerify {
    public static void main(String[] args) {
        String phone = "18834171358";
        //获取随机验证码
        String code = getRandCode();
        //将验证码保存在redis中，并设置过期时间，还有电话访问次数
        saveCode(phone,code);
        //验证是否一致
        boolean res = verify(phone, code);
        System.out.println(res);

    }

    //验证号码是否一致
    public static boolean verify(String phone,String code){
        String CodeVerify = "Verify" + phone + ":code";
        Jedis jedis = new Jedis("192.168.169.129", 6379);
        jedis.auth("991102");
        if (code.equals(jedis.get(CodeVerify))){
            System.out.println("成功");
            jedis.close();
            return true;
        }else {
            System.out.println("失败");
            jedis.close();
            return false;
        }
    }
    //将验证码保存至redis，并设置过期时间，同时保存手机号发送次数
    public static void saveCode(String phone,String code){
        Jedis jedis = new Jedis("192.168.169.129", 6379);
        jedis.auth("991102");
        //设置验证码的保存key值
        String CodeVerify = "Verify" + phone + ":code";
        String phoneVerify = "Verify" + phone + "count";


        //手机号第一次申请验证码
        if (jedis.get(phoneVerify)==null){
            //保存这个手机号申请验证的次数
            jedis.setex(phoneVerify,24*60*60,"1");
        }else if (Integer.parseInt(jedis.get(phoneVerify))<=2){
            //手机号申请两次以内
            jedis.incr(phoneVerify);
        }else{
            System.out.println("今日申请次数已经超过3次！");
            jedis.close();
            return;
        }

        jedis.setex(CodeVerify,120,code);
    }

    //编写一个功能获取随机验证码
    public static String getRandCode(){
        Random random = new Random();
        String code = "";
        for (int i=0;i<6;i++){
            int num = random.nextInt(10);
            code += num;
        }
        return code;
    }

}
```

运行四次结果如下：

![image](E:\笔记\java_Redis\image\redis-8.1.1.png)

![image](E:\笔记\java_Redis\image\redis-8.1.1.png)

![image](E:\笔记\java_Redis\image\redis-8.1.1.png)

![image](E:\笔记\java_Redis\image\redis-8.1.2.png)

**redis:**

![image](E:\笔记\java_Redis\image\redis-8.1.3.png)

## 九、spring boot整合redis

### 1、首先要在pom.xml中添加相关依赖

```XML
<!-- redis -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
        </dependency>

        <!-- spring2.X集成redis所需common-pool2-->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-pool2</artifactId>
        </dependency>
```

这里使用的3.0.0以下的spring boot版本，因为3.0.0以上的spring boot要求JDK 17以上版本

```XML
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.10</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
```

### 2、要在application.properties中配置redis配置

```properties
#Redis服务器地址
spring.redis.host=192.168.169.129
#Redis服务器连接端口
spring.redis.port=6379
spring.redis.password=991102
#Redis数据库索引（默认为0）
spring.redis.database= 0
#连接超时时间（毫秒）
spring.redis.timeout=1800000
#连接池最大连接数（使用负值表示没有限制）
spring.redis.lettuce.pool.max-active=20
#最大阻塞等待时间(负数表示没限制)
spring.redis.lettuce.pool.max-wait=-1
#连接池中的最大空闲连接
spring.redis.lettuce.pool.max-idle=5
#连接池中的最小空闲连接
spring.redis.lettuce.pool.min-idle=0
```

### 3、添加redis配置类

创建包`com.cheng.redis_springboot.config` ，并在包中创建RedisConfig.java类

```java

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.cache.CacheManager;
import org.springframework.cache.annotation.CachingConfigurerSupport;
import org.springframework.cache.annotation.EnableCaching;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.cache.RedisCacheConfiguration;
import org.springframework.data.redis.cache.RedisCacheManager;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;
import org.springframework.data.redis.serializer.RedisSerializationContext;
import org.springframework.data.redis.serializer.RedisSerializer;
import org.springframework.data.redis.serializer.StringRedisSerializer;

import java.time.Duration;

@EnableCaching
@Configuration
public class RedisConfig extends CachingConfigurerSupport {

    @Bean
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory factory) {
        RedisTemplate<String, Object> template = new RedisTemplate<>();
        RedisSerializer<String> redisSerializer = new StringRedisSerializer();
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
        template.setConnectionFactory(factory);
//key序列化方式
        template.setKeySerializer(redisSerializer);
//value序列化
        template.setValueSerializer(jackson2JsonRedisSerializer);
//value hashmap序列化
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
        return template;
    }

    @Bean
    public CacheManager cacheManager(RedisConnectionFactory factory) {
        RedisSerializer<String> redisSerializer = new StringRedisSerializer();
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
//解决查询缓存转换异常的问题
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);
// 配置序列化（解决乱码的问题）,过期时间600秒
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                .entryTtl(Duration.ofSeconds(600))
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(redisSerializer))
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(jackson2JsonRedisSerializer))
                .disableCachingNullValues();
        RedisCacheManager cacheManager = RedisCacheManager.builder(factory)
                .cacheDefaults(config)
                .build();
        return cacheManager;
    }
}
```

### 4、创建测试类RedisTestController.java

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequestMapping("/redisTest")
public class RedisTestController {

    @Autowired
    private RedisTemplate redisTemplate;

    @GetMapping
    public String testRedis(){

        //完成redis键值的设置
        redisTemplate.opsForValue().set("name","lucy");

        //从redis中获取值
        String name = (String)redisTemplate.opsForValue().get("name");
        return name;
    }
}
```

结果如图所示：

![image](E:\笔记\java_Redis\image\redis-9.1.png)

## 十、Redis事务

### 10.1 Redis事务定义

Redis事务是一个单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。

Redis事务的主要作用就是串联多个命令防止别的命令插队。

### 10.2 Redis事务执行指令

[EXEC命令]([EXEC — Redis 命令参考 (redisfans.com)](http://doc.redisfans.com/transaction/exec.html))   [Multi命令]([MULTI — Redis 命令参考 (redisfans.com)](http://doc.redisfans.com/transaction/multi.html))    [Discard命令]([DISCARD — Redis 命令参考 (redisfans.com)](http://doc.redisfans.com/transaction/discard.html)) 

从输入Multi命令开始，输入的命令都会依次进入命令队列中，但不会执行，直到输入Exec后，Redis会将之前的命令队列中的命令依次执行。

在组队的过程中可以通过discard命令来放弃组队。

![image](E:\笔记\java_Redis\image\redis-10.1.1.png)

**操作示例：**

提交成功，以及取消入队操作。当取消组队后，这个事务中添加的所有命令操作都将被取消。

![image](E:\笔记\java_Redis\image\redis-10.1.2.png)

### 10.3 事务过程中的错误处理

1. **当错误发生在组队过程中，那么在执行的时候，队列中所有的命令都会被取消**

![image](E:\笔记\java_Redis\image\redis-10.3.1.png)

举例如下所示：

![image](E:\笔记\java_Redis\image\redis-10.3.2.png)

当开启事务后，在组队过程中，set k4语法错误，出现报错，因此在exec执行队列中命令的时候，所有的命令都会被取消。事务中的命令操作无效。

2. **如果是在执行阶段某个命令报出了错误，则只有报错的命令不会被执行，而其他的命令都会执行，不会发生回滚。**

![image](E:\笔记\java_Redis\image\redis-10.3.3.png)

举例如下所示：

在事务中，创建键值k3 值为v3，第二个命令为将k3的值增1，但是这个命令适用于int类型的操作。当输入命令exec开始执行阶段的时候，可以看到成功保存了k3的值，只要incr命令错误，并且其他命令不受它的影响。

![image](E:\笔记\java_Redis\image\redis-10.3.4.png)

### 10.4 为什么要用事务

有很多场景需要用到事务，例如，有很多人拥有同一个账户，当多个人用同一个账户购物的时候，余额的更新需要事务。

### 10.5 事务冲突

案例：

一个请求想给金额减8000
一个请求想给金额减5000
一个请求想给金额减1000

![image](E:\笔记\java_Redis\image\redis-10.5.1.png)

当多个人都想对余额进行修改的时候，因为都访问的是初始的金额数据，因此可能会导致最总的数据不正确。

**解决事务冲突可以添加锁来解决:例如`悲观锁`和`乐观锁`。**

### 10.6 悲观锁

![image](E:\笔记\java_Redis\image\redis-10.6.1.png)

**悲观锁(Pessimistic Lock)**, 顾名思义，就是很悲观，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会block（阻塞）直到它拿到锁。**传统的关系型数据库里边就用到了很多这种锁机制**，比如**行锁**，**表锁**等，**读锁**，**写锁**等，都是在做操作之前先上锁。

### 10.7 乐观锁

![image](E:\笔记\java_Redis\image\redis-10.7.1.png)

**乐观锁(Optimistic Lock),** 顾名思义，就是很乐观，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。**乐观锁适用于多读的应用类型，这样可以提高吞吐量**。Redis就是利用这种check-and-set机制实现事务的。

### 10.8 锁机制示例

[Watch命令]([WATCH — Redis 命令参考 (redisfans.com)](http://doc.redisfans.com/transaction/watch.html))   [Unwatch命令]([UNWATCH — Redis 命令参考 (redisfans.com)](http://doc.redisfans.com/transaction/unwatch.html))  

乐观锁在redis中的使用

> 使用watch key1 [key2]命令来监视一个或多个key，
>
> 使用unwatch取消 WATCH 命令对所有 key 的监视。
>
> 如果在执行 WATCH 命令之后，EXEC 命令或DISCARD 命令先被执行了的话，那么就不需要再执行UNWATCH 了。
>
> 因为 [*EXEC*](http://doc.redisfans.com/transaction/exec.html#exec) 命令会执行事务，因此 [*WATCH*](http://doc.redisfans.com/transaction/watch.html#watch) 命令的效果已经产生了；而 [*DISCARD*](http://doc.redisfans.com/transaction/discard.html#discard) 命令在取消事务的同时也会取消所有对 key 的监视，因此这两个命令执行之后，就没有必要执行 [UNWATCH](http://doc.redisfans.com/transaction/unwatch.html#unwatch) 了。

在执行multi之前，先执行watch key1 [key2],可以监视一个(或多个) key ，如果在**事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。**

![image](E:\笔记\java_Redis\image\redis-10.8.1.png)

初始时创建一个happykey值为100并保存到redis中，这时两个连接都通过watch happykey命令监视着happykey这个键，因此当两个连接都启动事务的时候，左侧连接1对happykey执行加10操作，但还未执行，此时右侧连接2对happykey执行加20的操作，仍未执行。此时左侧连接1先进行exec执行操作，成功将happykey改变为110，并修改了happykey的版本号，此时连接2试图执行队列中的命令，但是由于存在乐观锁，检查到happykey的版本号与数据库中的不一致，因此，此次执行返回nil，事务被打断。

### 10.9 事务三种特征

|        特性        |                             描述                             |
| :----------------: | :----------------------------------------------------------: |
|   单独的隔离操作   | 事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。 |
| 没有隔离级别的概念 | 队列中的命令没有提交之前都不会实际被执行，因为事务提交前任何指令都不会被实际执行 |
|    不保证原子性    | 事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚 |

## 十一、Redis事务示例-秒杀案例

## 十二、Redis持久化——RDB

[参考官网信息]([REDIS persistence -- Redis中国用户组（CRUG）](http://www.redis.cn/topics/persistence.html))

Redis 提供了不同级别的持久化方式:

- RDB持久化方式能够在指定的时间间隔能对你的数据进行快照存储.
- AOF持久化方式记录每次对服务器写的操作,当服务器重启的时候会重新执行这些命令来恢复原始的数据,AOF命令以redis协议追加保存每次写的操作到文件末尾.Redis还能对AOF文件进行后台重写,使得AOF文件的体积不至于过大.
- 如果你只希望你的数据在服务器运行的时候存在,你也可以不使用任何持久化方式.
- 你也可以同时开启两种持久化方式, 在这种情况下, 当redis重启的时候会优先载入AOF文件来恢复原始的数据,因为在通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整.

### 12.1 定义

- RDB持久化方式能够在指定的时间间隔能对你的数据进行快照存储。也就是说在指定的时间间隔内将内存中的数据集快照写入磁盘， 也就是Snapshotting快照，进行数据恢复时是将快照文件直接读到内存里

- > 快照：
  >
  > 在默认情况下， Redis 将数据库快照保存在名字为 dump.rdb的二进制文件中。你可以对 Redis 进行设置， 让它在“ N 秒内数据集至少有 M 个改动”这一条件被满足时， 自动保存一次数据集。你也可以通过调用 SAVE或者 BGSAVE ， 手动让 Redis 进行数据集保存操作。
  >
  > 比如说， 以下设置会让 Redis 在满足“ 60 秒内有至少有 1000 个键被改动”这一条件时， 自动保存一次数据集:
  >
  > save 60 1000
  >
  > 这种持久化方式被称为快照snapshotting

### 12.2 RDB持久化流程（工作方式\如何进行持久化）

当 Redis 需要保存 dump.rdb 文件时， 服务器执行以下操作:

- Redis 调用forks. 同时拥有父进程和子进程。
- 子进程将数据集写入到一个临时 RDB 文件中。
- 当子进程完成对新 RDB 文件的写入时，Redis 用新 RDB 文件替换原来的 RDB 文件，并删除旧的 RDB 文件。

这种工作方式使得 Redis 可以从写时复制（copy-on-write）机制中获益。

通俗来讲：Redis会单独创建（fork）一个子进程来进行持久化，子进程先将数据写入到 一个临时RDB文件中，待临时文件写入完成时，再用这个临时文件替换旧的RDB文件。 整个过程中，主进程是不进行任何IO操作的，这就确保了极高的性能。如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。

### 12.3 写时复制机制

[写时复制]([Linux 写时复制机制原理 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/366707663))

在 Linux 系统中，调用 `fork ` 系统调用创建子进程时，并不会把父进程所有占用的内存页复制一份，而是与父进程共用相同的内存页，而当子进程或者父进程对内存页进行修改时才会进行复制 —— 这就是著名的 `写时复制` 机制。

![image](E:\笔记\java_Redis\image\redis-15.3.1.png)

Linux会fork（可以理解为克隆）一个子进程，子进程只复制主进程的页表，因此耗时很短，其次，当进行读数据时，内存中设定为只读模式，当主进程进行写操作时，也就是修改数据时，主进程会进行数据拷贝，将内存中的要修改的数据单独拷贝一份进行数据操作，操作完成后更新数据的指针。当再进行读操作时，也会将之前更新的数据添加到read-only区域。

`写时复制` 的原理大概如下：

- 创建子进程时，将父进程的 `虚拟内存` 与 `物理内存` 映射关系复制到子进程中，并将内存设置为只读（设置为只读是为了当对内存进行写操作时触发 `缺页异常`）。
- 当子进程或者父进程对内存数据进行修改时，便会触发 `写时复制` 机制：将原来的内存页复制一份新的，并重新设置其内存映射关系，将父子进程的内存读写权限设置为可读写。

![image](https://pic4.zhimg.com/80/v2-87ed5a2084718934efb43d8448ee0137_720w.webp)

当创建子进程时，父子进程指向相同的 `物理内存`，而不是将父进程所占用的 `物理内存` 复制一份。这样做的好处有两个：

- 加速创建子进程的速度。
- 减少进程对物理内存的使用。

### 12.4 RDB备份与数据恢复

先通过config get dir 查询rdb文件的目录 

将*.rdb的文件拷贝到另一个安全的地方

RDB进行Redis数据恢复

只要持久化文件存在，那么当重启redis的时候，服务器会载入rdb或aof文件进行数据恢复。

- 关闭Redis

- 先把备份的文件拷贝到工作目录下 cp dump2.rdb dump.rdb

-  启动Redis, 备份数据会直接加载

### 12.5 RDB配置位置

RDB持久化方式将快照数据保存至dump.rdb文件中，在redis.conf中配置文件可以配置名称，默认为dump.rdb

![image](E:\笔记\java_Redis\image\redis-12.5.1.png)

也可以通过配置文件修改rdb文件的保存路径。默认为启动Redis服务时所在的目录下

![image](E:\笔记\java_Redis\image\redis-12.5.2.png)

> - **stop-writes-on-bgsave-error:**当Redis无法写入磁盘时，直接关掉Redis的写操作。默认为yes
>
> ![image](E:\笔记\java_Redis\image\redis-12.5.3.png)
>
> - **rdbcompression**:对于存储到磁盘中的快照，可以设置是否进行压缩存储。如果是的话，redis会采用LZF算法进行压缩。如果你不想消耗CPU来进行压缩的话，可以设置为关闭此功能。默认yes
>
>   ![](E:\笔记\java_Redis\image\redis-12.5.4.png)
>
> - **rdbchecksum**：检查完整性。在存储快照后，还可以让redis使用CRC64算法来进行数据校验，
>
>   但是这样做会增加大约10%的性能消耗，如果希望获取到最大的性能提升，可以关闭此功能，默认为yes.
>
>   ![image](E:\笔记\java_Redis\image\redis-12.5.6.png)

### 12.6 如何调用RDB进行持久化备份；保持策略；

- 当满足一定条件时，自动保存一次数据集
- 通过调用 SAVE或者 BGSAVE ， 手动让 Redis 进行数据集保存操作。

通过对 Redis 进行设置， 让它在“ N 秒内数据集至少有 M 个改动”这一条件被满足时， 自动保存一次数据集。

![image](E:\笔记\java_Redis\image\redis-12.6.1.png)

> 比如说， 以下设置会让 Redis 在满足“ 60 秒内有至少有 1000 个键被改动”这一条件时， 自动保存一次数据集:
>
> save 60 1000
>
> 这种持久化方式被称为快照snapshotting

|    命令    |                             描述                             |
| :--------: | :----------------------------------------------------------: |
|  **SAVE**  | 用于创建当前数据库的备份,该命令将在 redis 安装目录中创建dump.rdb文件 |
| **BGSAVE** | 创建 redis 备份文件也可以使用命令 **BGSAVE**，该命令在后台执行。 |

获取 redis 安装目录可以使用 **CONFIG** 命令

```she
redis 127.0.0.1:6379> CONFIG GET dir
1) "dir"
2) "/usr/local/redis/bin"
```

 **CONFIG GET dir** 输出的 redis 安装目录为 /usr/local/redis/bin。

### 12.7  RDB优点

- RDB是一个非常紧凑的文件,它保存了某个时间点得数据集,非常适用于数据集的备份,比如你可以在每个小时报保存一下过去24小时内的数据,同时每天保存过去30天的数据,这样即使出了问题你也可以根据需求恢复到不同版本的数据集.**内容紧凑，节省磁盘空间**
- RDB是一个紧凑的单一文件,很方便传送到另一个远端数据中心或者亚马逊的S3（可能加密），非常适用于灾难恢复.**适合大规模的数据恢复**
- RDB在保存RDB文件时父进程唯一需要做的就是fork出一个子进程,接下来的工作全部由子进程来做，父进程不需要再做其他IO操作，所以**RDB持久化方式可以最大化redis的性能.**
- 与AOF相比,在恢复大的数据集的时候，RDB方式会更快一些.**恢复速度快**

### 12.8  RDB缺点

- Fork的时候，内存中的数据被克隆了一份，大致2倍的膨胀性需要考虑

- 如果你希望在redis意外停止工作（例如电源中断）的情况下丢失的数据最少的话，那么RDB不适合你.虽然你可以配置不同的save时间点(例如每隔5分钟并且对数据集有100个写的操作),是Redis要完整的保存整个数据集是一个比较繁重的工作,你通常会每隔5分钟或者更久做一次完整的保存,万一在Redis意外宕机,你可能会丢失几分钟的数据.**可能会丢失redis宕机前的最后一次快照**
- RDB 需要经常fork子进程来保存数据集到硬盘上,**当数据集比较大的时候,fork的过程是非常耗时的,**可能会导致Redis在一些毫秒级内不能响应客户端的请求.如果数据集巨大并且CPU性能不是很好的情况下,这种情况会持续1秒,AOF也需要fork,但是你可以调节重写日志文件的频率来提高数据集的耐久度.**数据庞大的时候会影响性能**

### 12.9 如何关闭RDB

- 通过命令行输入` config set save ""`禁用RDB
- 通过在配置文件中注释掉save指令

## 十三、Redis持久化——AOF

### 13.1 定义

- AOF持久化方式记录每次对服务器**写的操作**,当服务器重启的时候会重新执行这些命令来恢复原始的数据,AOF命令以redis协议追加保存每次写的操作到文件末尾.Redis还能对AOF文件进行后台重写,使得AOF文件的体积不至于过大.
- 以**日志**的形式来记录每个写操作（增量保存），将Redis执行过的所有写指令记录下来(**读操作不记录**)， **只许追加文件但不可以改写文件**，redis启动之初会读取该文件重新构建数据，换言之，redis 重启的话就根据日志文件的内容将写指令从前到后执行一次以完成数据的恢复工作

### 13.2 为什么要用AOF

快照功能并不是非常耐久（durable）： 如果 Redis 因为某些原因而造成故障停机， 那么服务器将丢失最近写入、且仍未保存到快照中的那些数据。 从 1.1 版本开始， Redis 增加了一种完全耐久的持久化方式： AOF 持久化。

可以在配置文件中打开AOF方式:

![image](E:\笔记\java_Redis\image\redis-13.2.2.png)

从现在开始， 每当 Redis 执行一个改变数据集的命令时（比如 SET）， 这个命令就会被追加到 AOF 文件的末尾。这样的话， 当 Redis 重新启时， 程序就可以通过重新执行 AOF 文件中的命令来达到重建数据集的目的。

> AOF默认不开启
>
> AOF文件的保存路径，同RDB的路径一致。

### 13.3 AOF持久化流程

（1）客户端的请求写命令会被append追加到AOF缓冲区内；

（2）AOF缓冲区根据AOF持久化策略[always,everysec,no]将操作同步到磁盘的AOF文件中；

（3）AOF文件大小超过重写策略或手动重写时，会对AOF文件重写，压缩AOF文件容量；

（4）Redis服务重启时，会重新加载AOF文件中的写操作达到数据恢复的目的；

![iamge](E:\笔记\java_Redis\image\redis-13.2.1.png)

### 13.4 AOF和RDB同时打开，Redis如何选择

可以同时开启两种持久化方式, 在这种情况下, 当redis重启的时候会优先载入AOF文件来恢复原始的数据,因为在通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整.

> RDB和AOF直接的相互作用
>
> 在版本号大于等于 2.4 的 Redis 中， BGSAVE 执行的过程中， 不可以执行 BGREWRITEAOF 。 反过来说， 在 BGREWRITEAOF 执行的过程中， 也不可以执行 BGSAVE。这可以防止两个 Redis 后台进程同时对磁盘进行大量的 I/O 操作。
>
> 如果 BGSAVE 正在执行， 并且用户显示地调用 BGREWRITEAOF 命令， 那么服务器将向用户回复一个 OK 状态， 并告知用户， BGREWRITEAOF 已经被预定执行： 一旦 BGSAVE 执行完毕， BGREWRITEAOF 就会正式开始。 当 Redis 启动时， 如果 RDB 持久化和 AOF 持久化都被打开了， 那么程序会优先使用 AOF 文件来恢复数据集， 因为 AOF 文件所保存的数据通常是最完整的。

### 13.5 AOF 启动、修复、恢复

- AOF的备份机制和性能虽然和RDB不同, 但是备份和恢复的操作同RDB一样，都是拷贝备份文件，需要恢复时再拷贝到Redis工作目录下，启动系统即加载。
- 正常恢复
  1. 修改默认的appendonly no，改为yes
  2. 将有数据的aof文件复制一份保存到对应目录(查看目录：config get dir)
  3. 恢复：重启redis然后重新加载
- 如果AOF文件受损，或者AOF文件异常时进行恢复
  1. 修改默认的appendonly no，改为yes
  2. 如果AOF文件受损，修复AOF文件
  3. 备份AOF文件
  4. 恢复数据：重启redis,载入AOF文件。

### 13.6 AOF文件修复

服务器可能在程序正在对 AOF 文件进行写入时停机， 如果停机造成了 AOF 文件出错（corrupt）， 那么 Redis 在重启时会拒绝载入这个 AOF 文件， 从而确保数据的一致性不会被破坏。当发生这种情况时， 可以用以下方法来修复出错的 AOF 文件：

- 为现有的 AOF 文件创建一个备份。

- 使用 Redis 附带的 redis-check-aof 程序，对原来的 AOF 文件进行修复:

  $ redis-check-aof –fix

- （可选）使用 diff -u 对比修复后的 AOF 文件和原始 AOF 文件的备份，查看两个文件之间的不同之处。

- 重启 Redis 服务器，等待服务器载入修复后的 AOF 文件，并进行数据恢复。

### 13.7 AOF同步频率设置

可以配置 Redis 多久才将数据 fsync 到磁盘一次。有三种方式：

- `appendfsync always`每次有新命令追加到 AOF 文件时就执行一次 fsync(日志同步写入) ：非常慢，也非常安全，性能较差但是数据完整性较好
- `appendfsync everysec`每秒 fsync 一次：足够快（和使用 RDB 持久化差不多），并且在故障时只会丢失 1 秒钟的数据。
- `appendfsync no`从不 fsync ：将同步时机交给操作系统来处理。更快，也更不安全的选择。
- 推荐（并且也是默认）的措施为每秒 fsync 一次， 这种 fsync 策略可以兼顾速度和安全性。

### 13.8 AOF 日志重写压缩

- AOF采用文件追加方式，随着写入命令不断增加，AOF文件会越来越大。为处理这种情况，新增了重写机制, 当AOF文件的大小超过所设定的阈值时，Redis就会启动AOF文件的内容压缩， 只保留可以恢复数据的最小指令集。

  也就是说可以在不打断服务客户端的情况下， 对 AOF 文件进行重建（rebuild）。执行 BGREWRITEAOF 命令， Redis 将生成一个新的 AOF 文件， 这个文件包含重建**当前数据集所需的最少命令**。Redis 2.2 需要自己手动执行 BGREWRITEAOF 命令； Redis 2.4 则可以自动触发 AOF 重写， 具体信息请查看 2.4 的示例配置文件。

- > 因为 AOF 的运作方式是不断地将命令追加到文件的末尾， 所以随着写入命令的不断增加， AOF 文件的体积也会变得越来越大。举个例子， 如果你对一个计数器调用了 100 次 INCR ， 那么仅仅是为了保存这个计数器的当前值， AOF 文件就需要使用 100 条记录（entry）。然而在实际上， 只使用一条 SET 命令已经足以保存计数器的当前值了， 其余 99 条记录实际上都是多余的。

#### **重写流程**

AOF 重写和 RDB 创建快照一样，都巧妙地利用了写时复制机制:

- BGREWRITEAOF 触发重写，判断是否当前有bgsave或bgrewriteaof在运行，如果有，则等待该命令结束后再继续执行。
- Redis 执行 fork() ，现在同时拥有父进程和子进程。
- 子进程开始将新 AOF 文件的内容写入到临时文件。
- 对于所有新执行的写入命令，父进程一边将它们累积到一个内存缓存中，一边将这些改动追加到现有 AOF 文件的末尾,这样样即使在重写的中途发生停机，现有的 AOF 文件也还是安全的。
- 当子进程完成重写工作时，它给父进程发送一个信号，父进程在接收到信号之后，将内存缓存中的所有数据追加到新 AOF 文件的末尾。
- 搞定！现在 Redis 原子地用新文件替换旧文件，之后所有命令都会直接追加到新 AOF 文件的末尾。

> redis4.0版本后的重写，是指上就是把rdb 的快照，以二级制的形式附在新的aof头部，作为已有的历史数据，替换掉原来的流水账操作。

![image](E:\笔记\java_Redis\image\redis-13.8.1.png)

#### AOF重写和AOF追加操作时阻塞情况

同时在执行bgrewriteaof操作和主进程写aof文件的操作，两者都会操作磁盘，而bgrewriteaof往往会涉及大量磁盘操作，这样就会造成主进程在写aof文件的时候出现阻塞的情形，现在no-appendfsync-on-rewrite（在重写的时候不进行追加AOF）参数出场了。如果 no-appendfsync-on-rewrite=yes ,不写入aof文件只写入缓存，用户请求不会阻塞，但是在这段时间如果宕机会丢失这段时间的缓存数据。（降低数据安全性，提高性能）

如果 no-appendfsync-on-rewrite=no, 还是会把数据往磁盘里刷，但是遇到重写操作，可能会发生阻塞。（数据安全，但是性能降低）

#### 什么时候触发重写

Redis会记录上次重写时的AOF大小，默认配置是当AOF文件大小是上次rewrite后大小的一倍且文件大于64M时触发

重写虽然可以节约大量磁盘空间，减少恢复时间。但是每次重写还是有一定的负担的，因此设定Redis要满足一定条件才会进行重写。 

![image](E:\笔记\java_Redis\image\redis-13.8.2.png)

auto-aof-rewrite-percentage：设置重写的基准值，文件达到100%时开始重写（文件是原来重写后文件的2倍时触发）

auto-aof-rewrite-min-size：设置重写的基准值，最小文件64MB。达到这个值开始重写。

例如：文件达到70MB开始重写，降到50MB，下次什么时候开始重写？100MB

系统载入时或者**上次重写完毕时**，Redis会记录此时AOF大小，设为base_size,

如果Redis的AOF当前大小>= base_size +base_size*100% (默认)且当前大小>=64mb(默认)的情况下，Redis会对AOF进行重写。

### 13.9 优势

- 使用AOF 会让你的Redis更加耐久: 你可以使用不同的fsync策略：无fsync,每秒fsync,每次写的时候fsync.使用默认的每秒fsync策略,Redis的性能依然很好(fsync是由后台线程进行处理的,主线程会尽力处理客户端请求),一旦出现故障，你最多丢失1秒的数据.**备份机制更稳健，丢失数据概率更低。**
- AOF文件是一个只进行追加的日志文件,所以不需要写入seek,即使由于某些原因(磁盘空间已满，写的过程中宕机等等)未执行完整的写入命令,你也也可使用redis-check-aof工具修复这些问题.**具有强壮的稳健性和可靠性**
- Redis 可以在 AOF 文件体积变得过大时，自动地在后台对 AOF 进行重写： 重写后的新 AOF 文件包含了恢复当前数据集所需的最小命令集合。 整个重写操作是绝对安全的，因为 Redis 在创建新 AOF 文件的过程中，会继续将命令追加到现有的 AOF 文件里面，即使重写过程中发生停机，现有的 AOF 文件也不会丢失。 而一旦新 AOF 文件创建完毕，Redis 就会从旧 AOF 文件切换到新 AOF 文件，并开始对新 AOF 文件进行追加操作。**良好的重写机制，解决了AOF文件过大的问题**
- AOF 文件有序地保存了对数据库执行的所有写入操作， 这些写入操作以 Redis 协议的格式保存， 因此 AOF 文件的内容非常容易被人读懂， 对文件进行分析（parse）也很轻松。 导出（export） AOF 文件也非常简单： 举个例子， 如果你不小心执行了 FLUSHALL 命令， 但只要 AOF 文件未被重写， 那么只要停止服务器， 移除 AOF 文件末尾的 FLUSHALL 命令， 并重启 Redis ， 就可以将数据集恢复到 FLUSHALL 执行之前的状态。**可读的日志文本，通过操作AOF稳健，可以处理误操作**

### 13.10 劣势

- 对于相同的数据集来说，AOF 文件的体积通常要大于 RDB 文件的体积。**比起RDB占用更多的磁盘空间。**
- 根据所使用的 fsync 策略，AOF 的速度可能会慢于 RDB 。 在一般情况下， 每秒 fsync 的性能依然非常高， 而关闭 fsync 可以让 AOF 的速度和 RDB 一样快， 即使在高负荷之下也是如此。 不过在处理巨大的写入载入时，RDB 可以提供更有保证的最大延迟时间（latency）。**恢复备份速度要慢**
- 每次读写都同步的话，有一定的性能压力
- 存在个别Bug

### 13.11 如何选择

![image](E:\笔记\java_Redis\image\redis-13.10.1.png)

官方推荐两个都启用。

如果对数据不敏感，可以选单独用RDB。

不建议单独用 AOF，因为可能会出现Bug。

如果只是做纯内存缓存，可以都不用

## 十四、Redis主从复制

### 定义

主机数据更新后根据配置和策略， 自动同步到备机的master/slaver机制，**Master以写为主，Slave以读为主**

### 作用

- 读写分离，性能扩展

-  容灾快速恢复

![image](E:\笔记\java_Redis\image\redis-14.1.1.png)

### 过程

拷贝多个redis.conf文件为每个服务器生成一个配置文件

内容配置：

- include(写绝对路径)

- 开启daemonize yes

- Pid文件名字pidfile

- 指定端口port

- Log文件名字

- dump.rdb名字dbfilename

- Appendonly 关掉或者换名字

### 示例

新建三个重命名的redis.conf配置文件，并填写以下内容

`redis6379.conf`

![image](E:\笔记\java_Redis\image\redis-14.1.2.png)

`redis6380.conf`

![image](E:\笔记\java_Redis\image\redis-14.1.3.png)

`redis6381.conf`

![image](E:\笔记\java_Redis\image\redis-14.1.4.png)

**启动这三台redis服务器**

![iamge](E:\笔记\java_Redis\image\redis-14.1.5.png)

查看服务是否打开，并且打印主从复制的相关信息

![image](E:\笔记\java_Redis\image\redis-14.1.6.1.png)

![image](E:\笔记\java_Redis\image\redis-14.1.7.png)

![](E:\笔记\java_Redis\image\redis-14.1.8.png)

### 配置一主二从

**配从库不配主库**

`SLAVEOF <ip><port>` 命令可以将当前服务器转变为指定服务器的从属服务器(slave server)。

令6379为主服务器，6380和6381为它的从属服务器,只需要在8380和6381执行改命令即可。

![image](E:\笔记\java_Redis\image\redis-14.1.10.png)

![image](E:\笔记\java_Redis\image\redis-14.1.11.png)

>  :notes:注意！！当主服务器设置密码的时候，需要在从机中添加配置`masterauth <password>`，这样才能确保主机在线，从机连接到主机。
>
> ![image](E:\笔记\java_Redis\image\redis-14.1.9.png)

在主机上可以完成写入操作和读取操作，但是从机只能读，不能写

![iameg](E:\笔记\java_Redis\image\redis-14.5.1.png)

>  主机挂掉，重启就行，一切如初
>
> 从机重启需重设：slaveof 127.0.0.1 6379   可以将配置增加到文件中。永久生效。

### 常用的设计模式：

#### 一主二仆

切入点问题？slave1、slave2是从头开始复制还是从切入点开始复制?比如从k4进来，那之前的k1,k2,k3是否也可以复制？

从机是否可以写？set可否？ 

主机shutdown后情况如何？从机是上位还是原地待命？

主机又回来了后，主机新增记录，从机还能否顺利复制？ 

其中一台从机down后情况如何？依照原有它能跟上大部队吗？

![image](E:\笔记\java_Redis\image\redis-14.6.1.png)

#### 薪火相传

上一个Slave可以是下一个slave的Master，Slave同样可以接收其他 slaves的连接和同步请求，那么该slave作为了链条中下一个的master, 可以有效减轻master的写压力,去中心化降低风险。

用 slaveof <ip><port>

中途变更转向:会清除之前的数据，重新建立拷贝最新的

风险是一旦某个slave宕机，后面的slave都没法备份

主机挂了，从机还是从机，无法写数据了

![image](E:\笔记\java_Redis\image\redis-14.6.2.png)

#### 反客为主

当一个master宕机后，后面的slave可以立刻升为master，其后面的slave不用做任何修改。

用 slaveof no one  将从机变为主机

![image](E:\笔记\java_Redis\image\redis-14.6.3.png)

### 复制原理

- Slave启动成功连接到master后会发送一个sync命令

- Master接到命令启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令， 在后台进程执行完毕之后，master将传送整个数据文件到slave,以完成一次完全同步

- 全量复制：而slave服务在接收到数据库文件数据后，将其存盘并加载到内存中。

- 增量复制：Master继续将新的所有收集到的修改命令依次传给slave,完成同步

- 但是只要是重新连接master,一次完全同步（全量复制)将被自动执行

![image](E:\笔记\java_Redis\image\redis-14.7.1.png)

### 哨兵模式

**反客为主的自动版**，能够后台监控主机是否故障，如果故障了根据投票数自动将从库转换为主库

![image](E:\笔记\java_Redis\image\redis-14.8.1.png)



## 十五、使用场景

![image](E:\笔记\java_Redis\image\redis-15.1.1.png)

### **一般在哪些场景使用到了redis？**

- **缓存**：

  - 穿透、击穿、雪崩

  - 双写一致、持久化
  - 数据过期、淘汰策略

- **分布式锁**
  - setnx、redisson 以及底层实现原理
- 计数器
  - incr
- 保存token
  - string
- 消息队列
  - list
- 延迟队列
  - zset

### 缓存穿透三兄弟

> :rabbit: **缓存穿透**：缓存穿透是指查询一个一定不存在的数据，由于缓存是不命中时需要从数据库查询，查不到数据则不写入缓存，这将导致这个不存在的数据每次请求都要到数据库去查询，进而给数据库带来压力。
>
> 造成缓存穿透的主要原因就是：查询某个Key对应的数据，Redis缓存中没有相应的数据，则直接到数据库中查询。. 数据库中也不存在要查询的数据，则数据库会返回空，而Redis也不会缓存这个空结果。. 这就造成每次通过这样的Key去查询数据都会直接到数据库中查询，Redis不会缓存空结果
>
> ![image](E:\笔记\java_Redis\image\redis-15.1.2.png)
>
> :ice_cream: 解决方案一：缓存空数据，查询返回的数据为空，仍然把这个结果存储在缓存中。
>
> 优点：简单
>
> 缺点：消耗内存，可能会发生不一致的情况
>
>  :ice_cream:解决方案二：添加布隆过滤器
>
> ![image](E:\笔记\java_Redis\image\redis-15.1.3.png)
>
> 当进行缓存预热的时候，把一些热点数据也添加到布隆过滤器中，每次查询的时候，先用布隆过滤器进行筛选，如果布隆过滤器中存在，再查询redis，如果布隆过滤器中不存在直接返回。
>
> 布隆过滤器的作用：用于检索一个元素是否存在于一个集合中
>
> 布隆过滤器底层是使用一个bitmap(位图)来实现的，数组中的每个单元只存储0或1.
>
> 布隆过滤器存储数据时，先使用多个hash函数获取key对应的hash值，根据hash值将数组对应位置改为1.
>
> 查询数据时，使用相同的hash函数获取hash值，判断是否所有的位置都为1，如果是，则存在这个值，如果有一个不为1，则不存在该键值。
>
> 可能存在误判的情况：可以设置误判率，一般设置为百分之5以内。
>
> > redisson或guava中有布隆过滤器的实现。
>
> ![image](E:\笔记\java_Redis\image\redis-15.1.4.png)
>
> 优点：内存占用少，没有多余的key
>
> 缺点：实现复杂，存在误判。
>
> 
>
>
> :rabbit:**缓存击穿：**缓存击穿是指某个一个热点数据的Key设置了过期时间，当key过期之后，恰好有大量并发请求对其进行访问，导致大量并发全部打在数据库上，导致数据库压力剧增
>
> ![image](E:\笔记\java_Redis\image\redis-15.1.5.png)
>
> :ice_cream: 解决方案一：使用互斥锁
>
> ![image](E:\笔记\java_Redis\image\redis-15.1.6.png)
>
> 当缓存没有命中时，添加一个互斥锁，然后该线程查询数据库并将数据写入缓存中，然后释放锁，在此期间，其他线程无法获取锁，因此效率较低。
>
> 优点：强一致性
>
> 缺点：性能较差
> 
>
> :ice_cream:方案二：逻辑过期
>
> ![image](E:\笔记\java_Redis\image\redis-15.1.7.png)
>
> 在一个线程中查询缓存，如果发现key已经过期，则添加一个互斥锁并开启一个新的线程进行缓存数据重建，重建成功后释放锁。此时之前的线程之间返回**过期**数据。当有新的线程请求这个数据的时候，直接返回新的数据。如果在数据重建期间有新的线程请求这个数据，则直接返回过期数据。
>
> 优点：高可用、性能较好
>
> 缺点：无法保证数据绝对一致
>
> :rabbit:缓存雪崩：在同一个时段内，有大量的key同时失效或者redis服务宕机，导致大量的请求到达数据库，造成较大的数据库压力。
>
> ![image](E:\笔记\java_Redis\image\redis-15.1.8.png)
>
> 如果是由于大量的key采用了相同的过期时间导致的雪崩
>
> :ice_cream: 解决方案一：给不同的key的TTL（过期时间）添加一个随机值
>
> 如果是由于redis服务宕机导致的雪崩
>
> :ice_cream:解决方案二：可以搭建高可用的redis集群，例如哨兵模式或者集群模式。
>
> :ice_cream:解决方案三：在设计缓存业务的时候可以添加降级限流策略（设置限流规则）
>
> :ice_cream:解决方案四：添加多级缓存，但是可能会使得业务结构复杂。



**redis作为缓存，MySQL的数据如何于redis进行同步呢？**

就是问如何保证的数据库和缓存的双写一致性，根据应用场景的要求不同，可以分为两个类型，一致性要求高的方案和允许延迟一致的方案。

### 双写一致性（读写一致性）

双写一致性：当修改了数据库的数据也要同时修改缓存中的数据，缓存和数据库的数据要保持一致。

根据业务背景不同方案不同：

- 一致性要求高：采用redisson提供的读写锁

  > 共享锁：读锁readLock,加锁之后，其他线程可以共享读操作
  >
  > 排他锁：独占锁writeLock,加锁之后，阻塞其他线程**读写操作**

- 允许延迟一致，采用异步通知

  > 异步方案：
  >
  > 使用MQ中间件，更新数据后，通知缓存删除
  >
  > 使用Canal中间件，不需要修改业务代码，伪装为Mysql的一个从节点，Canal通过读取binlog数据更新缓存。

![image](E:\笔记\java_Redis\image\redis-15.1.2.png)

- 对于读操作：缓存如果命中，则直接返回结果。如果缓存中不存在，则查询数据库，并将结果存储缓存中，设置超时时间。
- 写操作(更新修改)：使用**延迟双删除**
  - 删除缓存-》修改数据库-》延迟一段时间之后-》删除缓存

:question:先删除缓存还是先删除数据库？

先删除哪个都会有问题

**1.先删除缓存，再修改数据库。可能导致数据不一致**

正常情况下，多个线程进行工作，一个线程A进行更新操作，首先会删除缓存，然后修改数据库中的值，这时线程B进行查询操作，发现缓存中没有值，因此到数据库中取到新值，然后存储到缓存中。

:rabbit: 可能出现问题：多个线程交替工作的时候，一个线程A进行更新操作，首先删除缓存中的数据，这个时候线程B进行查询操作，发现缓存中没有值，因此到数据库中取出旧值(因为A还没有更新数据库),然后将旧值存储再缓存中。然后线程A继续进行，更新修改数据库中的值，然后结束线程。这个时候就导致了数据库和缓存中的数据不一致。

**2.先操作数据库，再删除缓存，可能会造成数据不一致**

正常情况下，多个线程工作，一个线程A进行更新操作，首先会更新数据库中的值，然后删除缓存中的数据。线程B进行查询操作，发现缓存中没有数据，因此会查询数据库得到修改后的新值，然后将新值写入缓存中。

:rabbit: **异常情况：多个线程工作，线程A成功跟新数据库中的数据，但是在删除缓存的时候出错没有删除成功，那么其他请求再查询缓存的时候每次都只能取到错误的数据。**

还有一种异常情况就是，如果线程B先进行查询操作，发现缓存中的数据过期，则会查询数据库中的数据，得到旧值，这时缓存A更新数据库中的数据为新值，然后删除掉缓存中的数据，此时，线程B继续执行，将查询到的旧值更新到缓存中，造成了数据不一致。

:ice_cream:解决方案1：延迟双删

>  删除缓存-》修改数据库-》延迟一段时间之后-》删除缓存

为什么要删除两次缓存？

为了降低脏数据的出现，如情况１说明的情况，先删除缓存再更新数据库会导致数据不一致。所有要删除脏数据

:rabbit: **如果Mysql是采用的主从分离的形式，那么主库和从库之间同步时也会有时间差，也会导致查询数据拿到的是脏数据。**例如：

![image](E:\笔记\java_Redis\image\redis-15.2.1.png)

此时有两个请求，请求A（更新操作）和请求B（查询操作）

1. 请求A更新操作，删除缓存
2. 请求主库进行更新操作，主库与从库进行数据同步
3. 请求B查询操作，发现缓存中没有数据
4. 到从库中进行查询操作
5. 此时数据同步还没有完成，查询到的是旧数据

:ice_cream:解决方案1.2：要求如果是对缓存进行填充数据的查询数据库操作，则强制要求指向主库进行查询

![image](E:\笔记\java_Redis\image\redis-15.2.2.png)

:ice_cream:解决方案二：分布式锁

线程对数据进行更新和查询的时候，都添加互斥锁。缺点：效率低下。优点：保证了强一致性

:ice_cream:解决方案2.2:读数据共享锁，写数据排他锁

共享锁：读锁readLock,加锁之后，其他线程可以共享读操作

排他锁：独占锁writeLock,加锁之后，阻塞其他线程**读写操作**

​	![image](E:\笔记\java_Redis\image\redis-15.2.3.png)

优点：保证了强一致性

缺点：性能低。

:ice_cream:解决方案三：使用MQ异步通知保证数据一致性。

![image](E:\笔记\java_Redis\image\redis-15.2.4.png)

当进行更新操作之后，发布一个更新消息给MQ，redis订阅MQ中的更新消息，执行更新缓存的操作。

利用消息队列进行删除操作的补偿，如果更新完数据库后，在对redis进行删除时出错，此时将redis中的key作为消息体发送到消息队列，系统接收到消息再次对缓存中的数据进行删除。

缺点：业务耦合严重。

:ice_cream:解决方案3.2：使用Canal异步通知保证数据一致性

![image](E:\笔记\java_Redis\image\redis-15.2.5.png)

binlog记录数据库中的DDL（数据定义语言）语句和DML（数据操作语言）语句，但不包括数据查询语句。当数据发生了变化，可以从缓存服务中获取变化后的数据，然后更新到缓存中。

优点：耦合度较低。

---

### 持久化

redis如何进行持久化操作？

rdb或者aof的执行原理？

![image](E:\笔记\java_Redis\image\redis-15.3.1.png)

---

### 过期策略

:question:假如redis中的key过期后，会立即删除吗？

Redis对数据设置有效时间，数据过期后，就需要将数据从内存中删除。可以按照不同的规则进行删除，这种删除规则就被称为数据的删除策略（过期策略）。

- 惰性删除:数据到达过期时间，不做处理。等下次访问该数据时，如果未过期，返回数据 ；发现已过期，删除，返回不存在。
  - 优点：节约CPU性能，发现必须删除的时候才删除 
  - 缺点：内存压力很大，出现长期占用内存的数据 

- 定期删除:周期性轮询redis库中的时效性数据，采用随机抽取的策略（在数据库中取出一定数量的随机key进行检查，并删除其中过期的key），利用过期数据占比的方式控制删除频度 



## 十六、场景问题

- 集群
  - 主从、哨兵、集群
  - 事务
  - redis为什么快

## 相关问题

