# HTTP/1.1 发展中遇到的问题  

## HTTP/1.1 发明以来发生了哪些变化？  

- 从几 KB 大小的消息，到几 MB 大小的消息  
- 每个页面小于 10 个资源，到每页面 100 多个资源  
- 从文本为主的内容，到富媒体（如图片、声音、视频）为主的内容  
- 对页面内容实时性高要求的应用越来越多  

## HTTP/1.1 的高延迟问题  

高延迟带来页面加载速度的降低  

- 随着带宽的增加，延迟并没有显著下降
- 并发连接有限
- 同一连接同时只能在完成一个 HTTP 事务（请求/响应）才能处理下一个事务  

## 高延迟 VS 高带宽  

单连接上的串行请求，无状态导致的高传输量（低网络效率）  

![](./img/low_latency.png)

## 无状态特性带来的巨大 HTTP 头部  

重复传输的体积巨大的 HTTP 头部  

## HTTP/1.1 为了解决性能问题做过的努力  

- Spriting 合并多张小图为一张大图供浏览器 JS 切割使用  
  - 不能区别对待  
- Inlining 内联，将图片嵌入到 CSS 或者 HTML 文件中，减少网络请求次数  
- Concatenation 拼接，将多个体积较小的 JavaScript 使用 webpack 等工具打包成 1个体积更大的 JavaScript 文件  
  - 1 个文件的改动导致用户重新下载多个文件  
- Sharding 分片，将同一页面的资源分散到不同域名下，提升连接上限  

## HTTP/1.1 不支持服务器推送消息  

# HTTP/2 有哪些特性  

## 解决 HTTP/1 性能问题的 HTTP/2  

![](./img/http2_layer.png)

- SPDY（2012-2016）
- HTTP2（RFC7540，2015.5）
  - 在应用层上修改，基于并充分挖掘 TCP 协议性能
  - 客户端向 server 发送 request 这种基本模型不会变
  - 老的 scheme 不会变，没有 http2://
  - 使用 http/1.x 的客户端和服务器可以无缝的通过代理方式转接到 http/2 上
  - 不识别 http/2 的代理服务器可以将请求降级到 http/1.x

## 多路复用带来的提升  

![](./img/http2_multiplexing.png)

## HTTP/2 主要特性  

- 传输数据量的大幅减少
  - 以二进制方式传输
  - 标头压缩

- 多路复用及相关功能
  - 消息优先级
- 服务器消息推送
  - 并行推送

# h2c：如何从 http://升级到 HTTP/2 协议？  

## HTTP/2 是不是必须基于 TLS/SSL 协议？  

- IETF 标准不要求必须基于TLS/SSL协议
- 浏览器要求必须基于TLS/SSL协议
- 在 TLS 层 ALPN (Application Layer Protocol Negotiation)扩展做协商，只认HTTP/1.x 的代理服务器不会干扰 HTTP/2
- shema：http://和 https:// 默认基于 80 和 443 端口
- h2：基于 TLS 协议运行的 HTTP/2 被称为 h2
- h2c：直接在 TCP 协议之上运行的 HTTP/2 被称为 h2c

## h2 与 h2c  

![](./img/protocol_negotiation.png)

## H2C：不使用 TLS 协议进行协议升级  

## H2C：客户端发送的 Magic 帧  

Preface（ASCII 编码，12字节）  

- 何时发送？
  - 接收到服务器发送来的 101 Switching Protocols
  - TLS 握手成功后
- Preface 内容
  - 0x505249202a20485454502f322e300d0a0d0a534d0d0a0d0a
  - PRI * HTTP/2.0\r\n\r\nSM\r\n\r\n
- 发送完毕后，应紧跟 SETTING 帧  

## 统一的连接过程  

![](./img/start_http2.png)

# h2：如何从 https://升级到 HTTP/2 协议？  

## 在 HTTP/2 应用层协议之下的 TLS 层  

![](./img/http2_tls.png)

## TLS 通讯过程  

![](./img/tls_communication_process.png)

## Next Protocol Negotiation (NPN)  

SPDY 使用的由客户端选择协议的 NPN 扩展  

![](./img/npn.png)

## Application-Layer Protocol Negotiation Extension  

![](./img/alpne.png)

# Stream、Message、Frame 间的关系  

## HTTP/2 核心概念  

- 连接 Connection：1个 TCP 连接，包含一个或者多个 Stream
- 数据流 Stream：一个双向通讯数据流，包含 1 条或者多条 Message
- 消息 Message：对应 HTTP/1 中的请求或者响应，包含一条或者多条 Frame
- 数据帧 Frame：最小单位，以二进制压缩格式存放 HTTP/1 中的内容

![](./img/http2_conception.png)

## Stream、Message、Frame 间的关系  

![](./img/stream_frame_message.png)

## 消息的组成：HEADERS 帧与 DATA 帧  

![](./img/headers_data.png)

## 传输中无序，接收时组装  

![](./img/http2_connection.png)

## 消息与帧  

![](./img/message_frame.png)

# 帧格式：Stream 流 ID 的作用  

## 9 字节标准帧头部  

![](./img/http2_std_header.png)

## Stream ID 的作用  

### 实现多路复用的关键  

- 接收端的实现可据此并发组装消息  
- 同一 Stream 内的 frame 必须是有序的（无法并发）  
- SETTINGS_MAX_CONCURRENT_STREAMS 控制着并发 Stream 数  

![](./img/http2_connection.png)

### 推送依赖性请求的关键  

- 由客户端建立的流必须是奇数  
- 由服务器建立的流必须是偶数  

![](./img/http2_conception2.png)

### 流状态管理的约束性规定  

- 新建立的流 ID 必须大于曾经建立过的状态为 opened 或者 reserved 的流 ID  
- 在新建立的流上发送帧时，意味着将更小 ID 且为 idle 状态的流置为 closed 状态  
- Stream ID 不能复用，长连接耗尽 ID 应创建新连接  

### 应用层流控仅影响数据帧  

Stream ID 为 0 的流仅用于传输控制帧  

### 在HTTP/1 升级到 h2c 中，以 ID 为 1 流返回响应，之后流进入half-closed (local)状态  
# 帧格式：帧类型及设置帧的子类型  

## 帧长度  

![](./img/frame_length.png)

- 0 至 2^14 (16,384) -1，所有实现必须可以支持 16KB 以下的帧  
- 2^14 (16,384) 至 2^24-1 (16,777,215)，传递 16KB 到 16MB 的帧时，必须接收端首先公布自己可以处理此大小，通过 SETTINGS_MAX_FRAME_SIZE 帧（Identifier=5）告知    

## 帧类型 Type  

![](./img/frame_type.png)

帧类型：

![](./img/frame_type2.png)

### Setting 设置帧格式（type=0x4）  

- 设置帧并不是“协商”，而是发送方向接收方通知其特性、能力  
- 一个设置帧可同时设置多个对象  

![](./img/frame_setting_type.png)

- Identifier：设置对象  
- Value：设置值  

设置类型：

- SETTINGS_HEADER_TABLE_SIZE (0x1)：通知对端索引表的最大尺寸（单位字节，初始 4096 字节）
- SETTINGS_ENABLE_PUSH (0x2)：Value设置为 0 时可禁用服务器推送功能，1 表示启用推送功能
- SETTINGS_MAX_CONCURRENT_STREAMS (0x3)：告诉接收端允许的最大并发流数量
- SETTINGS_INITIAL_WINDOW_SIZE (0x4)：声明发送端的窗口大小，用于Stream级别流控，初始值2^16-1 (65,535)字节
- SETTINGS_MAX_FRAME_SIZE (0x5)：设置帧的最大大小，初始值 2^14 (16,384)字节
- SETTINGS_MAX_HEADER_LIST_SIZE (0x6)：知会对端头部索引表的最大尺寸，单位字节，基于未压缩前的头部

# HPACK 如何减少 HTTP 头部的大小？  

## HPACK 头部压缩  

三种压缩方式

- 静态字典
- 动态字典
- 压缩算法：Huffman 编码（最高压缩比 8:5）  

## 静态字典  

- 含有 value 的表项
- 不含有 value 的表项  

![](./img/static_dict.png)

## HPACK 压缩示意  

同一个索引空间的 HEADER 表：  

![](./img/hpack.png)

索引表用法示意：

![](./img/index_table.png)

# HPACK 中如何使用 Huffman 树编码  

## Huffman 编码  

原理：出现概率较大的符号采用较短的编码，概率较小的符号采用较长的编码  

- 静态 Huffman 编码  
- 动态 Huffman 编码  

# HPACK 中 HEADER 的编码格式  

## HEADER 帧的格式  

![](./img/http2_header.png)

## CONTINUATION 持续帧(type=0x9)  

跟在 HEADER 帧或者 PUSH_PROMISE 帧之后，补充完整的 HTTP 头部  

![](./img/http2_continuation.png)

## 同一地址空间下的静态表与动态表  

- 静态表：61 项
- 动态表：先入先出的淘汰策略
  - 动态表大小由 SETTINGS_HEADER_TABLE_SIZE 设置帧定义
  - 允许重复项
  - 初始为空  

![](./img/index_address_space.png)

## 字面编码  

### 组成

- header name 和 value 以索引方式编码
- header name 以索引方式编码，而 header value 以字面形式编码
- header name 和 value 都以字面形式编码  

### 可控制是否进入动态表

- 进入动态表，供后续传输优化使用
- 不进入动态表
- 不进入动态表，并约定该头部永远不进入动态表  

## HEADER 二进制编码格式  

### 名称与值都在索引表中（包括静态表与动态表）  

编码方式：首位传 1，其余 7 位传索引号  

![](./img/header_index.png)

例如，method: GET在静态索引表中序号为 2，其表示应为 1000 0010，HEX 表示为 82  

### 名称在索引表中，值需要编码传递，同时新增至动态表中  

前 2 位传 01

- 名称举例，if-none-match 在静态索引表中序号为 41，表示为 01101001，HEX 表示为 69  
- 值举例，"5cb816f5-19d8” ，value长度 15 个字节，采用 Huffman 编码(H 为1)  ：10001100   

![](./img/header_index2.png)

### 名称、值都需要编码传递，同时新增至动态表中  

前 2 位传 01  

![](./img/header_index3.png)

### 名称在索引表中，值需要编码传递，且不更新至动态表中  

前 4 位传 0000  

![](./img/header_index4.png)

### 名称、值都需要编码传递，且不更新至动态表中  

前 4 位传 0000  

![](./img/header_index5.png)

### 名称在索引表中，值需要编码传递，且永远不更新至动态表中  

前 4 位传 0001  

![](./img/header_index6.png)

### 名称、值都需要编码传递，且永远不更新至动态表中  

前 4 位传 0001  

![](./img/header_index7.png)

## 动态表大小的两种控制方式  

### 在 HEADER 帧中直接修改  

![](./img/max_size.png)

### 在 SETTING 帧中修改  

SETTINGS_MAX_HEADER_LIST_SIZE (0x6)  

# 服务器端的主动消息推送  















































































