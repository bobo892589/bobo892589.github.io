---
title: 基于GoogleProtobuf的TCPSocket通信（nodeJS Server、iOS Client）
date: 2017-07-22 20:22:11
tags: ios socket protobuf nodejs
---
本文这是一个在socket中使用protobuf的一个尝试，算是比较简单的尝试。考虑到测试一下跨平台，所以这里是使用NodeJS实现服务端，ObjectC实现客户端，同时也实现了socket链接和数据分隔，protobuf数据类型判断和解析。
## Protobuf
源码地址：https://github.com/google/protobuf   
Google的一种数据交换的格式，开源的，具有空间开销小、解析速度快、兼容性好等优点，非常适合于对性能要求高的一些场景中。特别是对于即时通讯，就效率和成本而言，二进制协议明显优于http这样的文本协议。
## TCP Socket
网络上的两个程序通过一个双向的通信连接实现数据的交换，这个连接的一端称为一个socket。或者理解为：Socket=Ip address+ TCP/UDP + port。   
这里用的是tcp协议，主要还是考虑简单的问题，tcp特性就是可靠，有序，为了做到这些，TCP三次握手，每个包有序号，收到会有确认包，还有重传机制，当然还有更多一些机制来保证传输的可靠有序性。   
然而在网络差的时候，tcp的优势也会边劣势，这也就是为啥qq是以udp协议为主。如果使用udp，那些需要上层实现来考虑数据的可靠，有序，这需要相当大的工作量，但对于业务的发展、服务器的负载、更多网络环境的适应，udp是非常重要的方向，也是以后探索的一个方向。
## NodeJS服务器端实现
这是[github的demo源码](https://github.com/bobo892589/socket_server_demo)
### NodeJS中Protobuf的第三方模块
常用主要有三种：   
protobuf.js : https://github.com/dcodeIO/ProtoBuf.js   
Google : https://github.com/google/protobuf/tree/master/js   
protocol-buffers : https://github.com/mafintosh/protocol-buffers   
这里选用的是protobuf.js。安装全局模块（可以使用命令行工具pbjs）：
```
npm install -g protobufjs
```
### proto文件
.proto 文件是 Protocol Buffers 的结构化数据定义，结构化数据被称为 Message。  
定义一个msg.proto文件
```
syntax = "proto3";
package msg;

message Msg1 {
    string field1 = 1;
    int32 field2 = 2;
    string field3 = 3;
}

message Msg2 {
    int32 field4 = 1;
    string field5 = 2;
    string field6 = 3;
}
```
protobuf.js提供了6种方式使用Protobuf，这里使用json的方式，将proto文件编译为一个json文件
```
pbjs -t json msg.proto > bundle.json
```
### 编码实现
代码写比较简单，没有实现太多逻辑，特别是socket的管理、断包和粘包的情况并没有处理。   
引入protobuf.js库
```
npm install protobufjs --save
```
加载protobuf的数据定义
```
var protobuf = require("protobufjs");
var jsonDescriptor = require("./bundle.json"); // exemplary for node
var root = protobuf.Root.fromJSON(jsonDescriptor);
var Msg1 = root.lookupType("msg.Msg1");
var Msg2 = root.lookupType("msg.Msg2");
```
创建一个socket server的监听
```
var net = require('net');
var HOST = '127.0.0.1';
var PORT = 6969;
net.createServer(function(sock) {
  // 我们获得一个连接 - 该连接自动关联一个socket对象
  // 为这个socket实例添加一个"data"事件处理函数
  sock.on('data', function(data) {
    // 进行数据解析，看下面代码
    });
  // 为这个socket实例添加一个"close"事件处理函数
  sock.on('close', function(data) {
    // todo
    });
}).listen(PORT, HOST);
```
数据解析一、判断类型和去掉分隔符
```
var type = data[0];
var buffer = data.slice(1, data.length - 3); //丢弃头尾
console.log("type : " + type + '   buffer : ');
console.log(buffer);
if (type == 0x01) {
  // 用Msg1格式来解析数据
} else if (type == 0x02) {
  // 用Msg2格式来解析数据
}
```
数据解析二、用protobuf解析数据
```
var message = Msg1.decode(buffer);
console.log('message : ');
console.log(message);
var object = Msg1.toObject(message, {
  field1: String,
  field2: Number,
  field3: String,
// see ConversionOptions
  });
```
发送数据，编码为protobuf的二进制数据，同时拼接类型头和分隔符
```
// 回发数据
var payload1 = {
  field1: "test msg1_field1",
  field2: 123321,
  field3: "test msg1_field3"
};
var errMsg1 = Msg1.verify(payload1);
if (errMsg1)
  throw Error(errMsg1);
var message1 = Msg1.create(payload1);
var buffer1 = Msg1.encode(message1).finish();
//拼接head，表示类型，01:Msg1  02:Msg2
var contentBuf1 = Buffer.concat([Buffer.from([0x01]), buffer1]);
//拼接结束标志
contentBuf1 = Buffer.concat([contentBuf1, Buffer.from([0x0D, 0x0A])]);
sock.write(contentBuf1);
```

## ObjectC客户端的实现
这是[github的demo源码](https://github.com/bobo892589/SocketClientDemo)
### 安装Protobuf
```
brew install protobuf
```
可以使用`protoc`命令来编译proto文件
### Google protobuf/objectivec库
这是官方的protobuf在ObejctC支持库，引用也非常简单，支持cocoapods:
```
pod 'Protobuf', '~> 3.1.0'
```
### proto文件
直接使用上文的proto文件即可，编译proto文件：
```
protoc --objc_out=./ ./msg.proto
```
会生成`.h`和`.m`文件，添加到项目项目中，注意ARC项目的`.m`文件要加`-fno-objc-arc`编译标记。
### CocoaAsyncSocket库
iOS开发中很强大一个socket库，实现了各种类型的socket，还有各种情况的处理，特别是断包粘包的处理，如果想了解更多这个库，建议可以认真看看一个大神写的这5篇博客：[一](http://www.jianshu.com/p/0a11b2d0f4ae) [二](http://www.jianshu.com/p/22c984eac9b9) [三](http://www.jianshu.com/p/2e16572c9ddc) [四](http://www.jianshu.com/p/fdd3d429bdb3) [五](http://www.jianshu.com/p/19f0fd363f60)   
cocoapods 引入：
```
pod 'CocoaAsyncSocket', '~> 7.5.1'
```
### 源码实现
```
- (IBAction)onConnect:(id)sender {
    NSLog(@"onConnect !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!!");
    self.socket =  [[GCDAsyncSocket alloc] initWithDelegate:self delegateQueue:dispatch_get_main_queue()];
    [self.socket connectToHost:@"127.0.0.1" onPort:6969 error:nil];
}

- (IBAction)onSendBuf:(id)sender {
    NSMutableData *data =  [[NSData dataWithBytes:"\x01" length:1] mutableCopy];//head
    Msg1 *msg1 = [Msg1 new];
    msg1.field1 = @"client-msg1Field1";
    msg1.field2 = 789987;
    msg1.field3 = @"client-msg1Field3";
    [data appendData:[msg1 data]]; //content
    [data appendData:[GCDAsyncSocket CRLFData]]; //flag
    [self.socket writeData:data withTimeout:-1 tag:110];
}

- (IBAction)onSendBuf2:(id)sender {

    NSMutableData *data =  [[NSData dataWithBytes:"\x02" length:1] mutableCopy];//head
    Msg2 *msg2 = [Msg2 new];
    msg2.field4 = 667667;
    msg2.field5 = @"client-msg2Field2";
    msg2.field6 = @"client-msg2Field3";
    [data appendData:[msg2 data]]; //content
    [data appendData:[GCDAsyncSocket CRLFData]]; //flag
    [self.socket writeData:data withTimeout:-1 tag:110];
}

#pragma mark - GCDAsyncSocketDelegate

//连接成功调用
- (void)socket:(GCDAsyncSocket *)sock didConnectToHost:(NSString *)host port:(uint16_t)port {
    NSLog(@"连接成功,host:%@,port:%d",host,port);
    [sock readDataToData:[GCDAsyncSocket CRLFData] withTimeout:-1 tag:110];
}

//断开连接的时候调用
- (void)socketDidDisconnect:(GCDAsyncSocket *)sock withError:(nullable NSError *)err {
    NSLog(@"断开连接,host:%@,port:%d",sock.localHost,sock.localPort);
}

//写的回调
- (void)socket:(GCDAsyncSocket*)sock didWriteDataWithTag:(long)tag {
    NSLog(@"写的回调,tag:%ld",tag);
}


- (void)socket:(GCDAsyncSocket *)sock didReadData:(NSData *)data withTag:(long)tag {
    NSLog(@"收到消息：%@",data);
    if (data.length > 3) {
        const char *bytes = [data bytes];
        unsigned char type = bytes[0];
        NSData *protobuf = [data subdataWithRange:NSMakeRange(1, data.length - 3)];
        if (type == '\x01') {
            Msg1 *msg1 = [[Msg1 alloc] initWithData:protobuf error:nil];
            NSLog(@"Msg1 : field1:%@ | field2:%d | field3:%@", msg1.field1, (int)msg1.field2, msg1.field3);
        } else if (type == '\x02') {
            Msg2 *msg2 = [[Msg2 alloc] initWithData:protobuf error:nil];
            NSLog(@"Msg2 : field4:%d | field5:%@ | field6:%@", (int)msg2.field4, msg2.field5, msg2.field6);
        }
    }

    [sock readDataToData:[GCDAsyncSocket CRLFData] withTimeout:-1 tag:110];
}
```
理解`readDataToData:withTimeout:tag:`方法
```
//读取数据，有数据就会触发代理
- (void)readDataWithTimeout:(NSTimeInterval)timeout tag:(long)tag;
//直到读到这个长度的数据，才会触发代理
- (void)readDataToLength:(NSUInteger)length withTimeout:(NSTimeInterval)timeout tag:(long)tag;
//直到读到data这个边界，才会触发代理
- (void)readDataToData:(NSData *)data withTimeout:(NSTimeInterval)timeout tag:(long)tag;
```
这个框架每次读取数据，必须手动的去调用上述这些read方法，而我们之前的实现思路是，第一次连接成功的代理触发后调用，之后每次收到消息之后，都在去调用一次这个方法，超时为-1，即不超时。这样我们每次收到消息，都会即时触发我们读取消息的代理。   
这里使用读取到指定边界这个方法`readDataToData:withTimeout:tag:`，这样CocoaAsyncSocket就帮我们完成断包粘包的问题，同时也实现使用分隔符来识别数据包的问题。分隔符要跟服务器那边统一`0x0D`和`0x0A`其实它就是一个`\r\n`。   
protobuf的数据编码和解析也非常简单：
```
[msg1 data] //编码为二进制数据
[[Msg2 alloc] initWithData:protobuf error:nil];  //解析数据
```
## 总结一下
Socket对于即时通讯，对于要求即时性高业务是必不可少，而protobuf二进制序列化，虽然相比于json之类的无法肉眼可见可随时编辑，然后它的优点也明显更简洁更快，在一些性能要求的业务会是最好的选择。这里技术探索还是比较简单的，特别是socket，udp是一个非常重要的选择，会在以后更多地去接触和学习。
