## 1 gRPC

### 1.1 什么是 RPC

RPC 代指远程过程调用（Remote Procedure Call），它的调用包含了传输协议和编码（对象序列）协议等等，允许运行于一台计算机的程序调用另一台计算机的子程序，而开发人员无需额外地为这个交互作用编程，因此我们也常常称 RPC 调用，就像在进行本地函数调用一样方便。

### 1.2 什么是 gRPC

gRPC 是一个高性能、开源和通用的 RPC 框架，面向移动和基于 HTTP/2 设计。目前提供 C、Java 和 Go 语言等等版本，分别是：grpc、grpc-java、grpc-go，其中 C 版本支持 C、C++、Node.js、Python、Ruby、Objective-C、PHP 和 C# 支持。

gRPC 基于 HTTP/2 标准设计，带来诸如双向流、流控、头部压缩、单 TCP 连接上的多复用请求等特性。这些特性使得其在移动设备上表现更好，在一定的情况下更节省空间占用。

gRPC 的接口描述语言（Interface description language，缩写 IDL）使用的是 Protobuf，都是由 Google 开源的。

### 1.3 gRPC 调用模型

接下来我们一起看看 gRPC 的一个最简的调用模型，便于在脑海中形成一个基本调用流转，如下官方图：

![image](README.assets/grpc_concept_diagram.jpg)

1. 客户端（gRPC Stub）在程序中调用某方法，发起 RPC 调用。
2. 对请求信息使用 Protobuf 进行对象序列化压缩（IDL）。
3. 服务端（gRPC Server）接收到请求后，解码请求体，进行业务逻辑处理并返回。
4. 对响应结果使用 Protobuf 进行对象序列化压缩（IDL）。
5. 客户端接受到服务端响应，解码请求体。回调被调用的 A 方法，唤醒正在等待响应（阻塞）的客户端调用并返回响应结果。

## 2 Protobuf

### 2.1 什么是 Protobuf

Protocol Buffers（Protobuf）是一种与语言、平台无关，可扩展的序列化结构化数据的数据描述语言，我们常常称其为 IDL，常用于通信协议，数据存储等等，相较于 JSON、XML，它更小、更快，因此也更受开发人员的青眯。

### 2.2 基本语法

```protobuf
syntax = "proto3";

package helloworld;

service Greeter {
    rpc SayHello (HelloRequest) returns (HelloReply) {}
}

message HelloRequest {
    string name = 1;
}

message HelloReply {
    string message = 1;
}
```

1. 第一行（非空的非注释行）声明使用 `proto3` 语法。如果不声明，将默认使用 `proto2` 语法。同时建议无论是用 v2 还是 v3 版本，都应当进行显式声明。而在版本上，目前主流推荐使用 v3 版本。
2. 定义名为 `Greeter` 的 RPC 服务（Service），其包含 RPC 方法 `SayHello`，入参为 `HelloRequest` 消息体（message），出参为 `HelloReply` 消息体。
3. 定义 `HelloRequest`、`HelloReply` 消息体，每一个消息体的字段包含三个属性：类型、字段名称、字段编号。在消息体的定义上，除类型以外均不可重复。

在编写完.proto 文件后，我们一般会进行编译和生成对应语言的 proto 文件操作，这个时候 Protobuf 的编译器会根据选择的语言不同、调用的插件情况，生成相应语言的 Service Interface Code 和 Stubs。

### 2.3 基本数据类型

在生成了对应语言的 proto 文件后，需要注意的是 protobuf 所生成出来的数据类型并非与原始的类型完全一致，因此你需要有一个基本的了解，下面是我列举了的一些常见的类型映射，如下表：

| .proto Type | C++ Type | Java Type  | Go Type | PHP Type       |
| ----------- | -------- | ---------- | ------- | -------------- |
| double      | double   | double     | float64 | float          |
| float       | float    | float      | float32 | float          |
| int32       | int32    | int        | int32   | integer        |
| int64       | int64    | long       | int64   | integer/string |
| uint32      | uint32   | int        | uint32  | integer        |
| uint64      | uint64   | long       | uint64  | integer/string |
| sint32      | int32    | int        | int32   | integer        |
| sint64      | int64    | long       | int64   | integer/string |
| fixed32     | uint32   | int        | uint32  | integer        |
| fixed64     | uint64   | long       | uint64  | integer/string |
| sfixed32    | int32    | int        | int32   | integer        |
| sfixed64    | int64    | long       | int64   | integer/string |
| bool        | bool     | boolean    | bool    | boolean        |
| string      | string   | String     | string  | string         |
| bytes       | string   | ByteString | []byte  | string         |

## 3 思考 gRPC

### 3.1 gRPC 与 RESTful API 对比

| 特性       | gRPC                   | RESTful API          |
| ---------- | ---------------------- | -------------------- |
| 规范       | 必须.proto             | 可选 OpenAPI         |
| 协议       | HTTP/2                 | 任意版本的 HTTP 协议 |
| 有效载荷   | Protobuf（小、二进制） | JSON（大、易读）     |
| 浏览器支持 | 否（需要 grpc-web）    | 是                   |
| 流传输     | 客户端、服务端、双向   | 客户端、服务端       |
| 代码生成   | 是                     | OpenAPI+ 第三方工具  |

### 3.2 gRPC 优势

#### 3.2.1 性能

gRPC 使用的 IDL 是 Protobuf，Protobuf 在客户端和服务端上都能快速地进行序列化，并且序列化后的结果较小，能够有效地节省传输占用的数据大小。另外众多周知，gRPC 是基于 HTTP/2 协议进行设计的，有非常显著的优势。

另外常常会有人问，为什么是 Protobuf，为什么 gRPC 不用 JSON、XML 这类 IDL 呢，我想主要有如下原因：

- 在定义上更简单，更明了。
- 数据描述文件只需原来的 1/10 至 1/3。
- 解析速度是原来的 20 倍至 100 倍。
- 减少了二义性。
- 生成了更易使用的数据访问类。
- 序列化和反序列化速度快。
- 开发者本身在传输过程中并不需要过多的关注其内容。

#### 3.2.2 代码生成

在代码生成上，我们只需要一个 proto 文件就能够定义 gRPC 服务和消息体的约定，并且 gRPC 及其生态圈提供了大量的工具从 proto 文件中生成服务基类、消息体、客户端等等代码，也就是客户端和服务端共用一个 proto 文件就可以了，保证了 IDL 的一致性且减少了重复工作。

#### 3.2.3 流传输

gRPC 通过 HTTP/2 对流传输提供了大量的支持：

1. Unary RPC：一元 RPC。
2. Server-side streaming RPC：服务端流式 RPC。
3. Client-side streaming RPC：客户端流式 RPC。
4. Bidirectional streaming RPC：双向流式 RPC。

#### 3.2.4 超时和取消

gRPC 允许客户端设置截止时间，若超出截止时间那么本次 RPC 请求将会被取消，与此同时服务端也会接收到取消动作的事件，因此客户端和服务端都可以在达到截止时间后进行取消事件的相关联动处理。

并且根据 Go 语言的上下文（context）的特性，截止时间的传递是可以一层层传递下去的，也就是我们可以通过一层层 gRPC 调用来进行上下文的传播截止日期和取消事件，有助于我们处理一些上下游的连锁问题等等场景。但是同时也会带来隐患，如果没有适当处理，第一层的上下文取消，可以把最后的调用也给取消掉，这在某些场景下可能是有问题的（需要根据实际业务场景判别）。

### 3.3 gRPC 缺点

#### 3.3.1 可读性

默认情况下 gRPC 使用 Protobuf 作为其 IDL，Protobuf 序列化后本质上是二进制格式的数据，并不可读，因此其可读性差，没法像 HTTP/1.1 那样直接目视调试，除非进行其它的特殊操作调整格式支持。

#### 3.3.2 浏览器支持

目前来讲，我们无法直接通过浏览器来调用我们的 gRPC 服务，这意味着单从调试上来讲就没那么便捷了，更别提在其它的应用场景上了。

那官方有没有其余的工具协助呢，有的，gRPC-Web 提供了一个 JavaScript 库，使浏览器客户端可以访问 gRPC 服务，但它也是有限的 gRPC 支持（对流传输的支持比较弱）。gRPC-Web 由两部分组成：一个支持浏览器的 JavaScript 客户端，以及服务器上的一个 gRPC-Web 代理。调用流程为：gRPC-Web 客户端调用代理，代理将根据 gRPC 请求转发到 gRPC 服务。

但总归是需要额外的组件进行支持的，因此对浏览器的支持是有限的。

#### 3.3.3 外部组件支持

gRPC 是基于 HTTP/2 设计的，HTTP/2 标准在 2015 年 5 月以 RFC 7540 正式发表，虽然已经过去了好几年，HTTP/3 也已经有了踪影，但目前为止各大外部组件对 gRPC 这类基于 HTTP/2 设计的组件支持仍然不够完美，甚至有少数暂时就完全不支持。与此同时，即使外部组件支持了，但其在社区上的相关资料也比较少，需要开发人员花费部分精力进行识别和研究，这是一个需要顾及的点。

本图书由[ 煎鱼 ](https://github.com/eddycjy)©2020 版权所有