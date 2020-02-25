---
title: 一个关系数据库LevelSQL的设计和实现
date: 2020-02-25 20:52:12
tags: SQL, Database, MySQL, LevelSQL
---

前些天过年疫情在家时为了学习和复习数据库知识，实现了一个兼容MySQL的关系数据库LevelSQL，存储层可以基于磁盘文件或者LevelDB或者对象存储OSS或者Redis等中.

# 主要特性有:
* 一个查询计算引擎(SQL和执行计划等)
* 从存储引擎中分离出来的索引（目前索引只支持B+树）
* 统一的存储层接口以及基于磁盘文件或者LevelDB或者OSS或者Redis等的具体实现
* 支持MySQL网络协议(可以使用现有MySQL客户端和驱动使用)

# 代码

* [https://github.com/zoowii/levelsql](https://github.com/zoowii/levelsql)

# 本文主要讲下大概设计和结构

为了支持未来支持分布式架构，所以一开始就把查询引擎和存储引擎剥离，但是没有像TiDB那样直接用LSM同时做存储和索引，而是做成存储引擎和索引也分开来，通过固定接口使用和传递数据。计算引擎，索引，存储引擎分离的优势是未来存储引擎做成分布式存储，查询引擎使用无状态计算引擎，索引层做到基于存储引擎之上的应用层本身无状态或者伪无状态，则可以比较容易实现分布式数据库，通过简单给计算层或者存储层增加机器扩容

在这个考虑上，代码架构上主要组件如下


# 主要组件
- 存储引擎
    - 存储引擎接口
    - 存储引擎实现
- 索引
- 查询引擎 
    - SQL解析
    - 执行计划生成
    - 执行计划优化
    - 计划执行器
-  MySQL网络协议

# 请求处理流程

以insert插入语句和select查询语句为例进行说明

## select语句
server端通过MySQL网络协议接收到SQL语句后，先SQL解析器把SQL语句转换成抽象语法树AST，然后通过visitor模式把SQL AST转换成初始的执行计划树，执行计划树是一个由若干执行计划算子构成的树状结构，比如

```
 select sum(employee.age)
 from employee join country on employee.country_id = country.id 
 where age > 18
 order by age asc 
 limit 10
```

构造出来的逻辑执行计划树大致如下(火山模型下resultSet.next轻轻从上往下,查询结果数据流从下往上返回)

```
                                   aggregate(sum(age))
                                           |
                                      projection(从输入中选择employee.age列)
                                           |
                                         limit
                                           |
                                        order by (employee.age asc)
                                           |
                                        group by (无)
                                           |
                                         filter(age > 18)
                                           |
                                          join
                                     |              |
              scan table employee/index             scan table country/index
```

逻辑执行计划生成后可以根据需要进行优化，比如使用索引或者联合索引，部分不需要的操作可以不做，不需要的列可以不输出给上层，提前做一些表达式计算等

执行计划生成并优化后，交给计划执行器进行执行，LevelSQL中的计划执行器的实现方式是一个执行计划树启动后，把各算子都作为单独任务交给一个线程池执行，各算子从自己的任务队列中循环检查有没有收到新的FetchTask任务，有收到就执行自身逻辑，向children算子发送子任务，把子任务和处理结果作为自身的任务的一次结果.执行器在顶层算子输出sourceEnd结果或者error或者超时时结束本次计划的执行并把结果输出.

FetchTask是一个可能有三种结果(部分数据, 执行结束sourceEnd, 错误)的future类.

## insert语句
收到insert SQL语句后，会找到对应表的所有索引，并分别向聚集索引和二级索引（包括联合索引）分别构造key和数据插入索引中，索引收到插入请求后调用存储引擎的接口把数据写入存储层


# 存储引擎接口和实现

存储引擎的接口比较简单，因为目前还没加入事务的实现，所以存储引擎接口目前设计为一个类Key-Value的API接口供上层调用.

和普通KV接口区别是Key不是单个字符串或者字节数组，而是一个包含(命名空间 namespace: String, 序号 seq: Int)的元祖，value是普通的字节数组.

```
/** namespace是区分不同命名空间的区分词，比如不同索引，不同表有不同的namespace
 * seq是一个顺序增加的整数，一般每次只增加1(或者比较少的数量不要求必须递增1)
 */
data class StoreKey(val namespace: String, val seq: Long)

interface IStore {
     fun put(key: StoreKey, value: ByteArray) // 需要在新增数据调用时key.seq逐渐增加
     fun get(key: StoreKey): ByteArray?
}
```

之所以要求 key.seq顺序增加是为了索引层增加新页时key.seq可以比较简单的用新页的编号等提供，存储引擎实现时新增数据有更大序号也更容易实现(最简单的比如磁盘文件则可以简单的追加数据)

具体到实现这个接口的方式上，比如磁盘文件mmap到内存后，各namespace有不同文件，每个文件根据seq转换成所在块的偏移量直接写入即可. 如果用LevelDB/Redis实现，则可以把key整体序列化成字节数组或者字符串去作为键值插入.

# 查询引擎的设计

执行计划目前使用的火山模型，本文中select语句的执行流程中已经有一个很简单的说明，具体实现另外文章详细再写.

# SQL解析
因为SQL语法的简单性，所以没用lex/yacc等工具生成parser，直接基于一边遍历+lookahead的方式手写了个SQL Parser。具体可以看代码中com.zoowii.levelsql.sql.parser包下的代码

# MySQL网络协议

MySQL网络协议是一个基于TCP上的应用层协议，客户端和服务端发送的命令和返回都是组织成packet的方式发送, packet中有序号和command type等信息。简单介绍下TCP连接建立后的mysql会话建立阶段(connection phase)和会话建立后的命令阶段(command phase).

```

1. TCP连接建立，开始mysql会话建立截断
2. mysql server给client发送一个handshake packet
3. mysql client根据收到的handshake packet中信息构造包含authentication等信息的handshake response packet给server端
4. mysql server接受到handshake response packet后检查包中的信息，验证成功就发送err packet给客户端并结束连接，否则发送ok packet给客户端然后进入命令阶段
5. mysql client在收到ok packet进入命令阶段后，发送命令给mysql server，比如init db packet, query packet等
6. 以mysql client发送query packet为例，server端从query packet中解析得到SQL语句，并执行，得到结果后发送若干个packets给客户端完成这次查询(之所以返回多个包是因为返回结果的列数，headers的内容, 结束headers部分，返回结果的行数，各行数据，结束eof packet等都分别是一个单独的packet)

```