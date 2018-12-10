---
layout: redis-go
title: Golang Redis的基本操作
date: 2018-12-10 17:59:10
categories: Golang的第三方库的使用
tags:
  - Golang Reids
---

### 1. Reids 简介
**Redis**是一个开源的、使用C语言编写的、支持网络交互的、可基于内存也可持久化的Key-Value数据库。

## 1.1 特点
* **支持更多数据类型**
和Memcached类似,它支持存储的value类型相对更多, 包括string(字符串),　list(链表), set(集合), zset(sorted set 有序集合)和hash（哈希类型）.

* **支持复杂操作**
这些数据类型都支持push/pop、add/remove及取交集并集和差集及更丰富的操作，而且这些操作都是原子性的。在此基础上，Redis支持各种不同方式的排序。

* **支持主从同步**
与memcached一样，为了保证效率，数据都是缓存在内存中。区别的是Redis会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了master-slave(主从)同步。数据可以从主服务器向任意数量的从服务器上同步，从服务器可以是关联其他从服务器的主服务器。这使得Redis可执行单层树复制。从盘可以有意无意的对数据进行写操作。由于完全实现了发布/订阅机制，使得从数据库在任何地方同步树时，可订阅一个频道并接收主服务器完整的消息发布记录。同步对读取操作的可扩展性和数据冗余很有帮助。


Redis的出现，很大程度补偿了memcached这类key/value存储的不足，在部 分场合可以对关系数据库起到很好的补充作用。它提供了Java，C/C++，C#，PHP，JavaScript，Perl，Object-C，Python，Ruby，Erlang等客户端，使用很方便。Redis的官网地址，非常好记，是 {% link redis.io http://www.redis.io [external] [超链接] %}。目前，Vmware在资助着Redis项目的开发和维护。

### 2. 第三方库 redigo 介绍和使用 

**redigo**是GO语言的一个redis客户端实现。项目位于{% link github.com/gomodule/redigo http://www.github.com/gomodule/redigo  [external] [超链接] %}

## 2.1 安装redigo
**redigo** 没有其他依赖项，可以直接通过`go get`进行安装 
`
go get -v github.com/gomodule/redigo/redis
`

## 2.2 连接Redis
**Conn** 接口是与Redis协作的主要接口，可以使用Dial,DialWithTimeout或者NewConn函数来创建连接，当任务完成时，应用程序必须调用Close函数来完成

```
package main
import (
    "fmt"
    "github.com/gomodule/redigo/redis"
)
func main() {
    c, err := redis.Dial("tcp", "localhost:6379")
    if err != nil {
        fmt.Println("Connect to redis error", err)
        return
    }
    defer c.Close()
    ...

```

## 2.3 命令执行
**Conn**接口中有一个通用方法来执行
`n, err := c.Do("APPEND", "key", "value")`

**Do** 函数会必要时将参数转化为二进制字符串

Go Type			    | Conversion	        
--------------------|------------------
[]byte	            | Sent as is
string              | Sent as is
int, int64	        | strconv.FormatInt(v)
float64	            | strconv.FormatFloat(v, 'g', -1, 64)
bool	            | true -> "1", false -> "0"
nil	                | ""
all other types	    | fmt.Print(v)

**Redis** 命令响应会用以下Go类型表示：
Redis type	        |Go type
--------------------|------------------
error	            | redis.Error
integer	            | int64
simple string	    | string
bulk string	        | []byte or nil if value not present.
array	            | []interface{} or nil if value not present.

可以使用GO的类型断言或者reply辅助函数将返回的interface{}转换为对应类型


## 常用命令
* 读写
`
GET key
SET key value [EX seconds] [PX milliseconds] [NX|XX]`
`

GET取对应键值，如果键值不存在则nil会返回

```
_, err = c.Do("SET", "username", "YunlingZhang")
if err != nil {
    fmt.Println("redis set failed:", err)
}
username, err := redis.String(c.Do("GET", "username"))
if err != nil {
    fmt.Println("redis get failed:", err)
} else {
    fmt.Printf("Got username %v \n", username)
}
```

输出:
Got username YunlingZhang 
SET命令还支持附加参数
1. EX seconds -- 指定过期时间，单位为秒.
2. PX milliseconds -- 指定过期时间，单位为毫秒.
3. NX -- 仅当键值不存在时设置键值.
4. XX -- 仅当键值存在时设置键值. 
   设置键值过期时间为10s
   
```
// 写入值10S后过期
_, err = c.Do("SET", "password", "123456", "EX", "10")
if err != nil {
    fmt.Println("redis set failed:", err)
}
time.Sleep(11 * time.Second)
password, err := redis.String(c.Do("GET", "password"))
if err != nil {
    fmt.Println("redis get failed:", err)
} else {
    fmt.Printf("Got password %v \n", password)
}
```

输出
redis get failed: redigo: nil returned 

批量写入读取
`
MGET key [key ...]
MSET key value [key value ...]
`

批量写入读取对象(Hashtable)
`
HMSET key field value [field value ...]
HMGET key field [field ...]
`

检测值是否存在
`
EXISTS key
`

删除
`
DEL key [key ...]
`

设置过期时间
`
EXPIRE key seconds
`

## 2.4 管道化(Pipelining)
请求/响应服务可以实现持续处理新请求，即使客户端没有准备好读取旧响应。这样客户端可以发送多个命令到服务器而无需等待响应，最后在一次读取多个响应。这就是**管道化(pipelining)**，这个技术在多年就被广泛使用了。距离，很多POP3协议实现已经支持此特性，显著加速了从服务器下载新邮件的过程。 
Redis很早就支持管道化，所以无论你使用任何版本，你都可以使用管道化技术

连接支持使用Send()，Flush()，Receive()方法支持管道化操作

`
Send(commandName string, args ...interface{}) error
Flush() error
Receive() (reply interface{}, err error)
`

Send向连接的输出缓冲中写入命令。Flush将连接的输出缓冲清空并写入服务器端。Recevie按照FIFO顺序依次读取服务器的响应。下例展示了一个简单的管道：
```
c.Send("SET", "foo", "bar")
c.Send("GET", "foo")
c.Flush()
c.Receive() // reply from SET
v, err = c.Receive() // reply from GET
```
Do方法组合了Send,Flush和 Receive方法。Do方法先写入命令，然后清空输出buffer，最后接收全部挂起响应包括Do方发出的命令的结果。如果任何响应中包含一个错误，Do返回错误。如果没有错误，Do方法返回**最后一个响应**。如果Do函数的命令参数为""，那么Do方法会清空out缓冲并接受挂起响应而不发送任何命令。


## 2.5 并发
连接并不支持并发调用写入方法(Send,Flush)或者读取方法（Receive）。但是连接支持并发的读写。 
因为Do方法组合了Send，Flush和Receive，Do不可以与其他方法并发执行。

为了能够完全并发访问Redis，在gorouotine中使用线程安全的连接池来获取和释放连接。

## 2.6 发布和订阅(Pub/Sub)
使用Send，Flush和Receive方法来是实现Pub/Sb订阅者。

```
c.Send("SUBSCRIBE", "example")
c.Flush()
for {
    reply, err := c.Receive()
    if err != nil {
        return err
    }
    // process pushed message
}
```

PubSubConn类型封装了Conn提供了便捷的方法来实现订阅者模式。Subscribe,PSubscribe,Unsubscribe和PUnsubscribe方法发送和清空订阅管理命令。Receive将一个推送消息 
转化为一个在type switch更为方便使用的类型。

```
psc := redis.PubSubConn{c}
psc.Subscribe("example")
for {
    switch v := psc.Receive().(type) {
    case redis.Message:
        fmt.Printf("%s: message: %s\n", v.Channel, v.Data)
    case redis.Subscription:
        fmt.Printf("%s: %s %d\n", v.Channel, v.Kind, v.Count)
    case error:
        return v
    }
}
```

## 2.7 响应辅助函数
Bool,Int,Bytes,String,Strings和Values函数将响应转化为特定类型值。为了允许更方便的封装连接的Do和Receive方法的调用，函数使用第二个参数类型为error类型。如果这个error为non-nil，辅助函数会返回error。如果error为nil，则function转化响应为特定类型。

```
exists, err := redis.Bool(c.Do("EXISTS", "foo"))
if err != nil {
    // handle error return from c.Do or type conversion error.
}
The Scan function converts elements of a array reply to Go types:
var value1 int
var value2 string
reply, err := redis.Values(c.Do("MGET", "key1", "key2"))
if err != nil {
    // handle error
}
 if _, err := redis.Scan(reply, &value1, &value2); err != nil {
    // handle error
}
```

## 2.8 事务支持(Transaction)
MULTI, EXEC,DISCARD和WATCH是构成Redis事务的基础。它们允许在一个步骤中执行一组命令，并提供两点总要保证：
    1. 事务中的全部命令被序列化并且顺序执行。它保证不会在Redis事务的处理过程中处理其他客户端发起的请求。这样保证命令如同一个单独操作被执行。
    2. 事务中的命令全部被执行或者一个都没有被执行，这样一个Redis事务是原子的。Exec命令触发事务中全部命令的执行，这样如果客户端启动事务后失去连接，在调用MULTI命令后的命令都不会被执行，否则如果EXEC命令被调用，全部操作都会被执行。当配置用只允许添加的文件，Redis保证使用连个独立的write(2) 系统调用来将事务写入磁盘。如果Redis服务崩溃或者被系统管理强行关闭可能只有一部分操作被注册。Redis会在重启时检测到这种状态，并且从中退出附带一个错误。使用redis-check-aoft工具它可以修复只允许添加的文件并且移除事务碎片这样可以让服务重启。
    

使用Send和Do方法来实现管道化事务
```
c.Send("MULTI")
c.Send("INCR", "foo")
c.Send("INCR", "bar")
r, err := c.Do("EXEC")
fmt.Println(r) // prints [1, 1]
``` 
