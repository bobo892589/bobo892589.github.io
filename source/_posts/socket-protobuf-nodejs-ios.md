---
title: 基于GoogleProtobuf的TCPSocket通信（nodeJS Server、iOS Client）
date: 2017-07-22 20:22:11
tags:
---
本文这是一个在socket中使用protobuf的一个尝试，算是比较简单的尝试，考虑到测试一下跨平台，所以这里是使用nodeJS来做服务端，iOS做客户端，
## Protobuf
源码地址：https://github.com/google/protobuf   
Google的一种数据交换的格式，开源的，具有空间开销小、解析速度快、兼容性好等优点，非常适合于对性能要求高的一些场景中。特别是对于即时通讯，就效率和成本而言，二进制协议明显优于http这样的文本协议。
## TCP Socket
网络上的两个程序通过一个双向的通信连接实现数据的交换，这个连接的一端称为一个socket。或者理解为：Socket=Ip address+ TCP/UDP + port。   
这里用的是tcp协议，主要还是考虑简单的问题，tcp特性就是可靠，有序，为了做到这些，TCP三次握手，每个包有序号，收到会有确认包，还有重传机制，当然还有更多一些机制来保证传输的可靠有序性。   
然而在网络差的时候，tcp的优势也会边劣势，这也就是为啥qq是以udp协议为主。如果使用udp，那些需要上层实现来考虑数据的可靠，有序，这需要相当大的工作量，但对于业务的发展、服务器的负载、更多网络环境的适应，udp是非常重要的方向，也是以后探索的一个方向。
## NodeJS服务器端实现

### NodeJS中Protobuf的第三方模块
常用主要有三种：   
protobuf.js : https://github.com/dcodeIO/ProtoBuf.js   
Google : https://github.com/google/protobuf/tree/master/js   
protocol-buffers : https://github.com/mafintosh/protocol-buffers   
这里选用的是protobuf.js。安装模块：
```
npm install -g protobufjs
```
### proto文件
.proto 文件是 Protocol Buffers 的结构化数据定义，结构化数据被称为 Message。  
例如定义一个
protobuf.js提供了6种方式使用Protobuf，这里使用json的方式，将


### NodeJS的Protobuf模块
