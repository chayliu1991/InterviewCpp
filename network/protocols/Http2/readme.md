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









