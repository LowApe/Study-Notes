# 目录

# Redis 简介
Redis 是一个速度非常快的**非关系数据库**,它可以存储键(key)与 5 种不同类型的值(value)之间的映射,可以将**存储在内存的键值**对数据持久化到硬盘,可以使用**复制特性来扩展读性能**,还可以使用**客户端分片来扩展写性能**

> 通过对数据进行分片,用户可以将数据存储到多台机器里面,也可以从多台机器里面读取数据,这种方法在解决某些问题时可以获得线性级别的性能提升

**Redis与其他数据库和软件的对比**

![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g398iabxpnj30xn0kj166.jpg)

**附加特性**

Redis 拥有两种不同形式的持久化方法,它们都可以用小而紧凑的格式将存储在内存中的数据写入硬盘:
1. 时间点转储,在指定时间段内有指定数量的写操作执行
2. 所有修改了数据库的命令都写入一个只追加文件里面,用户可以根据数据的重要程度,将只追加写入设置为从不同步(sync)、每秒同步一次或者和每写入一个命令就同步一次


尽管 Redis 性能很好,但受限与 Redis 的内存存储设计,使用一台 Redis 服务器可能没有办法处理所有请求。为了 Redis 提供故障转移支持, Redis 实现**主从复制特性**:执行复制的从服务器会连接上主服务器,接收主服务器发送的整个初始副本(copy);之后主服务器执行写命令,都会被发送给所有连接着的从服务器去执行,从而实时更新从服务器的数据集

# Redis 数据结构简介

5 种不同数据结构类型:
- STRING
- LIST
- SET
- HASH
- ZSET (有序集合)

![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g39914bpr5j30wz0dhk12.jpg)

## Redis 中的字符串

字符串拥有一些和其他键值存储相似的命令,比如 GET(获取值)、SET(设置值)和 DEL (删除值)
![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g399b5of60j30wv04xn0b.jpg)
> redis-cli 这个客户端来介绍 Redis 命令

```
<!-- 进入 redis-cli -->
redis-cli

set hello world   //将键 hello 的值设置为 world
get hello          //获取键 hello 的值
del hello           //删除这个键值对
nil         //键值删除得到 nil,Python 将这个 nil 转换成 None

```

## Redis 中的列表
Redis 对链表(Linked-list)结构的支持使得它在键值存储的世界独树一帜。一个列表结构可以**有序存储多个字符串**,Redis 列表可执行的操作和很多编程语言里面的列表操作非常相似:LPUSH 命令和 RPUSH 命令分别用于将元素**推入列表的左端和右端**;LPOP 命令和 RPOP 命令分别用于从列表的**左端和右端弹出元素**;LINDEX 命令用于获取列表在给定位置上的元素;LRANGE 命令用于获取列表在给定范围上的所有元素。

![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g3abcvwfw6j30wv05x0vr.jpg)

```
<!-- RPUSH LRANGGE LINDEX LPOP 使用示例 -->
rpush list-key item  
rpush list-key item2
rpush list-key item   // 前三个向列表插入元素
lrange list-key 0 -1  // 使用 0 索引范围,-1 为范围的结束索引,可以取出列表所有的元素
lindex list-key 1     // 使用 LINDEX 可以从元素取出单个元素
lpop list-key         // 从列表弹出一个元素,被弹出的元素将不再存在于列表

```
Redis 列表还拥有从列表里面移除元素的命令,将元素插入列表中间的命令、将列表修剪至指定长度等等

## Redis 中的集合
Redis 的集合和列表都可以存储多个字符串,它们之间的不同在于,列表可以存储多个**相同的字符串**,而集合则通过使用**散列表**保证自己存储的每个字符串都是各不相同的(这些散列表只有键,但没有与键相关联的值)。本书表示集合的方法和表示列表的方法基本相同。

因为 Redis 的集合使用**无序方式存储元素**,所以用户不能像使用列表那样,将元素推入集合的某一端,或者从集合的某一端弹出元素。而使用 SADD 命令将元素添加到集合,或者使用 SREM 命令从集合里面移除元素。还可以通过 SISMEMBER 命令快速地检查一个元素是否已经**存在于集合中**,或者使用 SMEMBERS 命令获取集合包含的所有元素(如果集合包含非常多元素非常多,那么 SMEMBERS 会很慢)

![](http://ww1.sinaimg.cn/mw690/006rAlqhgy1g3acztwgw5j30wp061ad4.jpg)

```
<!-- SADD SMEMBERS Sismenber 和 SREM 的使用示例 -->
sadd set-key item
sadd set-key item2
sadd set-key item3    
sadd set-key item    // 尝试将一个元素添加到集合的时候,命令返回 1 表示这个元素被成功地添加到了集合里面
smembers set-key    // 获取集合包含的所有元素将得到一个由元素组成的序列,Python 客户端会将这个序列转换成 Python 集合

sismember set-key item4    //检查一个元素是否存在于集合中,Python 客户端会返回一个布尔值来表示检查结果

srem set-key item2     //在使用命令移除集合中的元素时
```

## Redis 中的散列

Redis 的散列可以**存储多个键值对之间的映射**。和字符串一样,散列存储的值既可以是**字符串**又可以是**数字值**,并且用户同样可以对散列存储的数字值执行**自增操作或者自减操作**。散列在很多方面就像是一个微缩版的 Redis,不少字符串命令都有相应的散列版本。下面对散列执行插入元素、获取元素和移除元素等操作

![](http://ww1.sinaimg.cn/mw690/006rAlqhly1g3adj36n3aj30wx062act.jpg)

```
<!-- 散列命令 -->
hset hash-key sub-key1 value1 // 存储散列表 hash-key ,键值对为 sub-key1 value1
hset hash-key sub-key2 value2
hset hash-key sub-key1 value1 //无法插入因为已经存在
hgetall hash-key   // 获取散列包含的所有键值对时,python 客户端会把整个散列表转换成一个 Python 字典

hdel hash-key sub-key1      // 删除整个键值对

```

## Redis 中的有序集合
有序序列和散列一样,都用于存储键值对;有序集合的键被称为**成员(member)**,每个成员都是各不相同；而有序集合的值则被称为**分值(score),分值必须为浮点数**有序集合是 Redis 里面唯一一个既可以根据成员访问元素,又可以根据分值以及分值的排列顺序访问元素结构

![](http://ww1.sinaimg.cn/large/006rAlqhly1g3g82w37uwj30g4031js5.jpg)

```
<!-- 有序集合命令 -->
zadd zset-key 728 member1
zadd zset-key 982 member0 //尝试向有序序列添加元素
zrange zset-key  0 -1 withscores // 获取有序集合包含的所有元素,多个元素按照分值大小进行排序,并且 python 客户端会将元素的分值转换成浮点数
zrangebyscore zset-key 0 800 withscores //用户根据分值范围获取一部分元素
zrem zset-key member1 //移除有序集合
```

# 你好 Redis

最近几年，越来越多的网站提供对网页链接、文章或者问题进行投票的功能，这些网站根据文章的发布时间和文章获得的投票数量计算一个评分，然后按照这个评分来决定如何排序和展示文章。

## 对文章进行投票

为了产生一个能够随着时间流逝而不断减少的评分，程序需要更具文章的发布时间和当前时间来计算：文章得到的支持票数量乘以一个常量，然后加上文章的发布时间，得到结果是文章的评分

![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1g8bqeh7h34j30bk0ao0uw.jpg)

- 使用**散列**存储文章信息的例子
	- key 为信息
	- value 对应的值
- 投票文章使用**有序集合**存储文章
	- key 为 文章ID
	- value 对应文章的发布时间(第二个有序是非文章的分值)

![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1g8bqapmuovj30q006utby.jpg)



> ⚠️注意:防止用户对同一篇文章进行多次投票,网站需要为每片文章记录一个**已投票的用户名单**，程序将为文章创建一个**集合(集合不能重复)**，并使用这个集合来存储已投票用户的ID

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhly1g8bqui1u4rj30b8086q4j.jpg)

- 当用户给一片文章投票的时候数据结构发生的变化
	1. 对应文章 ID 的数值改变
	2. 对应文章的已投用户添加该用户 ID

![image.png](http://ww1.sinaimg.cn/mw690/006rAlqhly1g8bqveb1gmj30qu0h0jz7.jpg)

```python
//准备好常量
ONE_WEEK_IN_SECORES = 7* 86400
VOTE_SCORE= 432

def article_vote(){
    cutoff = time.time() - ONE_WEEK_IN_SECORES
    if conn.zscore('time',article) < cutoff:
    	return;
    
    article_id = article.partition(':')[-1];
    if conn.sadd('voted:' + article_id,user):
    	conn.zincrby('score',article,VOTE_SCORE)
        conn.hincrby(article,'votes',1)
}

```



> Redis 事务：SADD、ZINCRBY 和 HINCRBY 这三个命令放在一个事务里面执行

## 发布并获取文章

发布一篇文章首先创建新的**文章 ID** ，这些工作可以通过对一个计数器(counter)执行 INCR 命令来完成。接着程序需要使用 SADD 将文章**发布者的 ID **添加到记录文章已投票用户集合，并使用 EXPIRE 命令为这个集合设置一个**过期时间** ，让 Redis 在文章发布期满一周之后自动删除这个集合，之后，程序会使用 HMSET 命令来存储文章相关信息，并执行两个 ZADD 命令，将文章的**初始评分(initial)**和**发布时间**分别添加两个相应的**有序集合**里面

```python
def post_article(conn,user,title,link):
	//生成一个新的文章 ID
	article_id = str(conn.incr('article:'))
    
    voted = 'voted:' + article_Id;
	//文章用户添加到已投票集合 key AID value user
	conn.sadd(voted,user);
	conn.expire(voted,ONE_WEEK_IN_SECONDS)
        
    now = time.time()
    article = 'article:' + article_id
    //	将文章信息存放到散列
    conn.hmset(article,{
        'titile':title,
        'link':link,
        'poster':user,
        'time':now,
        'votes':1,
    })
    
    //将文章添加到根据发布时间排序的有序集合和根据评分排序的有序集合里面
    conn.zadd('score',article,now+VOTE_SCORE)
    conn.zadd('time',article,now)
   	return article_id;
```

下面考虑就是如何取出**评分最高**的文章以及如何取出**最新发布**的文章。为了实现这两个功能，使用 ZRERANGE 命令取出多个文章 ID，然后在对每个文章 ID 执行一次 HGETALL 命令取出文章的详细信息。因为有序集合会根据成员的分值从小到大地排列元素，所以使用 ZRERANGE 命令，从分值从大到小的排序顺序取出文章 ID 才是正确的做法

```python
ARTICLE_PER_PAGE = 25

def get_articles(conn,page,order = 'score'):
    start = (page-1) * ARTICLE_PER_PAGE
    end = start + ARTICLE_PER_PAGE-1
    //获取多个文章 ID
    ids = conn.zrerange(order,start,end)
    articles = []
    for id in ids:
        article_data = conn.hgetall(id)
        article_data['id'] = id
        articles.append(article_data)
return articles
```

虽然限制可以展示最新发布的文章和评分最高的文章了，但它还不具备目前很多投票网站都支持的**群组功能**：这个功能可以让用户只看见与特定话题有关的文章、与“Java”有关的文章或者“学习编程“文章等等。

## 对文章进行分组

群组功能由两个部分组成,一个部分负责记录文章属于哪个群组，另一部分负责取出里面的文章。为了记录各个群组保存了哪些文章。需要为每个群组创建一个集合,并将同属一个群组的文章 ID 都记录到集合里面

```python
def add_remove_groups(conn,article_id,to_add=[],to_remove=[]):
    article = 'article' + article_id
    for group in to_add:
        conn.sadd('group:'+group,article)
    for group in to_remove:
        conn.sadd('group:'+group,article)
```

为了根据评分对群组文章进行**排序和分页**，网站需要将同一个群组里面的所有文章都按照评分有序地存储到一个有序集合里面。Redis 的 ZINTERSTORE 命令可以接受多个集合和多个有序集作为输入，找出所有同事存在于集合和有序集合的成员，并以几种不同的方式**合并**这些成员的分值。

下图包含了少量文章的群组集合和一个包含大量文章及评分的有序集合执行 ZINTRESTORE 命令的过程，注意观察哪些同时出现在集合和有序集合里面的文章时怎么被添加到结果有序集合

![image.png](http://ww1.sinaimg.cn/large/006rAlqhly1g8ciy1va6xj30te0c0qb8.jpg)

通过对存储群组文章的集合和存储文章评分的有序集合执行 ZINTERSORE 命令时，程序可以得到按照文章评分排序的群组文章；如果群组文章非常多，那么执行 ZINTERSTORE 命令就会比较花时间，为了尽量减少 Redis 的工作量，程序会将这个命令的**计算结果缓存 60 秒**。另外，我们还重用了已有的 get_articles() 函数来**分页并获取群组文章**

```python
def get_group_articles(conn,group,page,order='score:'):
    //为每个群组的每种排序顺都创建一个键
    key = order + group	
    //检查是否有已缓存的排序结果，没有就排序
    if not conn.exists(key):
        conn.zinterstore(key,
                        ['group:'+group,order],
                        aggregate='max',)
        conn.expire(key,60)
   //根据之前定义的 get_articles()函数进行分页并获取文章数据
   return get_articles(conn,page,key)
```

> 我们构建的文章投票网站允许一篇文章同时属于多个群组(比如一篇文章可以同事属于“编程”和"算法"两个群组),所以对于一篇同时属于多个群组的文章来说，**更新文章的评分**意味着程序需要对文章所属的全部群组执行**自增操作**。这种情况，如果一篇文章同时属于很多群组，那么更新文章评分这一操作可能会变得相当耗时,因此，我们在 get_group_articles() 函数里面对 ZINTERSTORE 命令的**执行结果进行缓存处理**，以此来尽量减少 ZINTERSTORE 命令的执行次数。开发者在灵活性或限制条件之间的取舍将改变程序存储和更新数据的方式



# 小结

- Redis 与其他数据库相同之处和不同之处
- Redis 是一个可以用来解决问题的工具，它既拥有其他数据库不具备的数据结构，又拥有**内存存储**(Redis 速度非常快)、**远程**(使得Redis 可以与多个客户端和服务器进行连接)、**持久化**(使得服务器可以在重启之后仍然保持重启之前的数据)等多个特性