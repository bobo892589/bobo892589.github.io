---
title: 基于GoogleProtobuf的TCPSocket通信（nodeJS Server、iOS Client）
date: 2017-07-31 20:22:11
tags:
---
本文这是一个在socket中使用protobuf的demo的记录，算是比较简单的尝试，考虑到测试一下跨平台，所以这里是使用nodeJS来做服务端，iOS做客户端，
## Protobuf
源码地址：https://github.com/google/protobuf   
Google的一种数据交换的格式，开源的，具有空间开销小、解析速度快、兼容性好等优点，非常适合于对性能要求高的一些场景中。特别是对于即时通讯，就效率和成本而言，二进制协议明显优于http这样的文本协议。
## TCP Socket


## NodeJS服务器端实现


### NodeJS的Protobuf模块
