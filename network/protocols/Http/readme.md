# HTTP 请求的典型场景  

![](./img/http_request.png)

![](./img/http_request2.png)

HTTP  协议是一种无状态的、应用层的、以请求/应答方式运行的协议，它使用可扩展的语义和自描述消息格式，与基于网络的超文本信息系统灵活的互动。

# HTTP 协议格式  

![](./img/http_format.png)

# OSI概念模型  

OSI（Open System Interconnection Reference Model）

![](./img/osi.png)

# OSI 模型与 TCP/IP 模型对照  

![](./img/tcp_ip.png)

# 报文头部

![](./img/headers.png)

# 评估 Web 架构的关键属性  

HTTP 协议应当在以下属性中取得可接受的均衡：
- 性能 Performance：影响高可用的关键因素
- 可伸缩性 Scalability：支持部署可以互相交互的大量组件
- 简单性 Simplicity：易理解、易实现、易验证
- 可见性 Visiable：对两个组件间的交互进行监视或者仲裁的能力。如缓存、分层设计等
- 可移植性 Portability：在不同的环境下运行的能力
- 可靠性 Reliability：出现部分故障时，对整体影响的程度
- 可修改性 Modifiability：对系统作出修改的难易程度，由可进化性、可定制性、可扩展性、可配置性、可重用性构成

## 架构属性：性能  

- 网络性能 Network Performance
  - Throughput 吞吐量：小于等于带宽 bandwidth
  - Overhead 开销：首次开销，每次开销
- 用户感知到的性能 User-perceived Performance
  - Latency 延迟：发起请求到接收到响应的时间
  - Completion 完成时间：完成一个应用动作所花费的时间
- 网络效率 Network Efficiency
  - 重用缓存、减少交互次数、数据传输距离更近、COD  

## 架构属性：可修改性  

- 可进化性 Evolvability：一个组件独立升级而不影响其他组件
- 可扩展性 Extensibility ：向系统添加功能，而不会影响到系统的其他部分
- 可定制性 Customizability ：临时性、定制性地更改某一要素来提供服务，不对常规客户产生影响
- 可配置性 Configurability ：应用部署后可通过修改配置提供新的功能
- 可重用性 Reusabilit ：组件可以不做修改在其他应用在使用

# REST 架构下的 Web  

![](./img/rest_web.png)

# URI  

URL： Uniform Resource Locator，表示资源的位置，期望提供查找资源的方法  

URN：  Uniform Resource Name，期望为资源提供持久的、位置无关的标识方式，并允许简单地将多个命名空间映射到单个URN命名
空间  

URI ： Uniform Resource Identifier，  用以区分资源，是 URL 和 URN 的超集，用以取代 URL 和 URN 概念    

## Uniform Resource Identifier 统一资源标识符  

Resource 资源，一个资源可以有多个 URI  

Identifier 标识符，将当前资源与其他资源区分开的名称  

Uniform 统一 ：

- 允许不同种类的资源在同一上下文中出现
- 对不同种类的资源标识符可以使用同一种语义进行解读
- 引入新标识符时，不会对已有标识符产生影响
- 允许同一资源标识符在不同的、internet 规模下的上下文中出现 

## URI 的组成  

组成：schema, user information, host, port, path, query, fragment  

![](./img/uri.png)

## 为什么要进行 URI 编码  

- 传递数据中，如果存在用作分隔符的保留字符怎么办？  

- 对可能产生歧义性的数据编码
	- 不在 ASCII 码范围内的字符
	- ASCII 码中不可显示的字符
	- URI 中规定的保留字符
	- 不安全字符（传输环节中可能会被不正确处理），如空格、引号、尖括号等

## URI 百分号编码  

- 百分号编码的方式  
- 非 ASCII 码字符（例如中文）：建议先 UTF8 编码，再 US-ASCII 编码  
- 对 URI 合法字符，编码与不编码是等价的  

# 请求行  

request-line = method SP request-target SP HTTP-version CRLF  

![](./img/request_line.png)

- method 方法：指明操作目的，动词  
- request-target = origin-form / absolute-form / authority-form / asterisk-form  
  - origin-form = absolute-path [ "?" query ]  
    - 向 origin server 发起的请求，path 为空时必须传递 /  
  - absolute-form = absolute-URI  
    - 仅用于向正向代理 proxy 发起请求时  
  - authority-form = authority  
    - 仅用于 CONNECT 方法  
  - asterisk-form = "*“  
    - 仅用于 OPTIONS 方法  

# 常见方法  

- GET：主要的获取信息方法，大量的性能优化都针对该方法，幂等方法
- HEAD：类似 GET 方法，但服务器不发送 BODY，用以获取 HEAD 元数据，幂等方法
- POST：常用于提交 HTML FORM 表单、新增资源等
- PUT：更新资源，带条件时是幂等方法
- DELETE：删除资源，幂等方法
- CONNECT：建立 tunnel 隧道
- OPTIONS：显示服务器对访问资源支持的方法，幂等方法
- TRACE：回显服务器收到的请求，用于定位问题。有安全风险

## 用于文档管理的 WEBDAV 方法(RFC2518)  

- PROPFIND：从 Web 资源中检索以 XML 格式存储的属性。它也被重载，以允许一个检索远程系统的集合结构（也叫目录层次结构）
- PROPPATCH：在单个原子性动作中更改和删除资源的多个属性
- MKCOL：创建集合或者目录
- COPY：将资源从一个 URI 复制到另一个 URI
- MOVE：将资源从一个 URI 移动到另一个 URI
- LOCK：锁定一个资源。WebDAV 支持共享锁和互斥锁。
- UNLOCK：解除资源的锁定

# HTTP 响应行  

status-line = HTTP-version SP status-code SP reason-phrase CRLF  

- status-code = 3DIGIT
- reason-phrase = *( HTAB / SP / VCHAR / obs-text )

![](./img/response_line.png)

# 响应码  

## 1xx  

请求已接收到，需要进一步处理才能完成，HTTP1.0 不支持  

- 100 Continue：上传大文件前使用，由客户端发起请求中携带 Expect: 100-continue 头部触发  
- 101 Switch Protocols：协议升级使用，由客户端发起请求中携带 Upgrade: 头部触发，如升级 websocket 或者 http/2.0  
- 102 Processing：WebDAV 请求可能包含许多涉及文件操作的子请求，需要很长时间才能完成请求。该代码表示服务器已经收到并正在处理请求，但无响应可用。这样可以防止客户端超时，并假设请求丢失  

## 2xx  

成功处理请求  

- 200 OK: 成功返回响应
- 201 Created: 有新资源在服务器端被成功创建
- 202 Accepted: 服务器接收并开始处理请求，但请求未处理完成。这样一个模糊的概念是有意如此设计，可以覆盖更多的场景。例如异步、需要长时间处理的任务
- 203 Non-Authoritative Information：当代理服务器修改了 origin server 的原始响应包体时（例如更换了HTML中的元素值），代理服务器可以通过修改200为203的方式告知客户端这一事实，方便客户端为这一行为作出相应的处理。203响应可以被缓存
- 204 No Content：成功执行了请求且不携带响应包体，并暗示客户端无需更新当前的页面视图
- 205 Reset Content：成功执行了请求且不携带响应包体，同时指明客户端需要更新当前页面视图
- 206 Partial Content：使用 range 协议时返回部分响应内容时的响应码
- 207 Multi-Status：RFC4918 ，在 WEBDAV 协议中以 XML 返回多个资源的状态
- 208 Already Reported：RFC5842 ，为避免相同集合下资源在207响应码下重复上报，使用 208 可以使用父集合的响应码

## 3xx  

重定向使用 Location 指向的资源或者缓存中的资源。在 RFC2068 中规定客户端重定向次数不应超过 5 次，以防止死循环  

- 300 Multiple Choices：资源有多种表述，通过 300 返回给客户端后由其自行选择访问哪一种表述。由于缺乏明确的细节，300 很少使用
- 301 Moved Permanently：资源永久性的重定向到另一个 URI 中
- 302 Found：资源临时的重定向到另一个 URI 中
- 303 See Other：重定向到其他资源，常用于 POST/PUT 等方法的响应中
- 304 Not Modified：当客户端拥有可能过期的缓存时，会携带缓存的标识etag、时间等信息询问服务器缓存是否仍可复用，而304是告诉客户端可以复用缓存
- 307 Temporary Redirect：类似302，但明确重定向后请求方法必须与原请求方法相同，不得改变
- 308 Permanent Redirect：类似301，但明确重定向后请求方法必须与原请求方法相同，不得改变

## 4xx  

客户端出现错误  

- 400 Bad Request：服务器认为客户端出现了错误，但不能明确判断为以下哪种错误时使用此错误码。例如HTTP请求格式错误
- 401 Unauthorized：用户认证信息缺失或者不正确，导致服务器无法处理请求。
- 407 Proxy Authentication Required：对需要经由代理的请求，认证信息未通过代理服务器的验证
- 403 Forbidden：服务器理解请求的含义，但没有权限执行此请求
- 404 Not Found：服务器没有找到对应的资源
- 410 Gone：服务器没有找到对应的资源，且明确的知道该位置永久性找不到该资源
- 405 Method Not Allowed：服务器不支持请求行中的 method 方法
- 406 Not Acceptable：对客户端指定的资源表述不存在（例如对语言或者编码有要求），服务器返回表述列表供客户端选择。
- 408 Request Timeout：服务器接收请求超时
- 409 Conflict：资源冲突，例如上传文件时目标位置已经存在版本更新的资源
- 411 Length Required：如果请求含有包体且未携带 Content-Length 头部，且不属于chunk类请求时，返回 411
- 412 Precondition Failed：复用缓存时传递的 If-Unmodified-Since 或 IfNone-Match 头部不被满足
- 413 Payload Too Large/Request Entity Too Large：请求的包体超出服务器能处理的最大长度
- 414 URI Too Long：请求的 URI 超出服务器能接受的最大长度
- 415 Unsupported Media Type：上传的文件类型不被服务器支持
- 416 Range Not Satisfiable：无法提供 Range 请求中指定的那段包体
- 417 Expectation Failed：对于 Expect 请求头部期待的情况无法满足时的响应码
- 421 Misdirected Request：服务器认为这个请求不该发给它，因为它没有能力处理
- 426 Upgrade Required：服务器拒绝基于当前 HTTP 协议提供服务，通过Upgrade 头部告知客户端必须升级协议才能继续处理
- 428 Precondition Required：用户请求中缺失了条件类头部，例如 If-Match
- 429 Too Many Requests：客户端发送请求的速率过快
- 431 Request Header Fields Too Large：请求的 HEADER 头部大小超过限制
- 451 Unavailable For Legal Reasons：RFC7725 ，由于法律原因资源不可访问

## 5xx  

服务器端出现错误  

- 500 Internal Server Error：服务器内部错误，且不属于以下错误类型
- 501 Not Implemented：服务器不支持实现请求所需要的功能
- 502 Bad Gateway：代理服务器无法获取到合法响应
- 503 Service Unavailable：服务器资源尚未准备好处理当前请求
- 504 Gateway Timeout：代理服务器无法及时的从上游获得响应
- 505 HTTP Version Not Supported：请求使用的 HTTP 协议版本不支持
- 507 Insufficient Storage：服务器没有足够的空间处理请求
- 508 Loop Detected：访问资源时检测到循环
- 511 Network Authentication Required：代理服务器发现客户端需要进行身份验证才能获得网络访问权限

# HTTP 连接的常见流程  

![](./img/http_request3.png)

![](./img/http_request4.png)

# 短连接与长连接  

Connection 头部  

- Keep-Alive：长连接
  - 客户端请求长连接
    - Connection: Keep-Alive
  - 服务器表示支持长连接
	  - Connection: Keep-Alive
  - 客户端复用连接
  - HTTP/1.1 默认支持长连接
	  - Connection: Keep-Alive 无意义  

- Close：短连接  
- 对代理服务器的要求  
  - 不转发 Connection 列出头部，该头部仅与当前连接相关  

Connection 仅针对当前连接有效，user agent 与 origin server 间有层层 proxy 代理  

![](./img/proxy.png)

# Host 头部  

Host = uri-host [ ":" port ]  

- HTTP/1.1 规范要求，不传递 Host 头部则返回 400 错误响应码  
- 为防止陈旧的代理服务器，发向正向代理的请求 request-target 必须以absolute-form 形式出现  
  - request-line = method SP request-target SP HTTP-version CRLF  
  - absolute-form = absolute-URI  
    - absolute-URI = scheme ":" hier-part [ "?" query ]  

## Host 头部与消息的路由  

- 建立 TCP 连接，确定服务器的 IP 地址  
- 接收请求  
- 寻找虚拟主机，匹配 Host 头部与域名  
- 寻找 URI 的处理代码，匹配 URI  
- 执行处理请求的代码，访问资源    
- 生成 HTTP 响应，各中间件基于 PF 架构串行修改响应 
- 发送 HTTP 响应  
- 记录访问日志  

# 客户端与源服务器间存在多个代理  

![](./img/multi_proxy.png)

## 消息的转发  

Max-Forwards 头部  

- 限制 Proxy 代理服务器的最大转发次数，仅对 TRACE/OPTIONS 方法有效  
- Max-Forwards = 1*DIGIT  

Via 头部  

- 指明经过的代理服务器名称及版本  

Cache-Control:no-transform  

- 禁止代理服务器修改响应包体  

## 如何传递 IP 地址？  

- TCP 连接四元组（src ip,src port, dst ip,dst port）
- HTTP 头部 X-Real-IP 用于传递用户 IP
- HTTP 头部 X-Forwarded-For 用于传递 IP
- 网络中存在许多反向代理

![](./img/ip_trans.png)

# 请求的上下文: User-Agent  

指明客户端的类型信息，服务器可以据此对资源的表述做抉择：

```
User-Agent = product *( RWS ( product / comment ) )
	product = token ["/" product-version]
	RWS = 1*( SP / HTAB )
```

请求的上下文: Referer，浏览器对来自某一页面的请求自动添加的头部：

Referer = absolute-URI / partial-URI  

Referer 不会被添加的场景：

- 来源页面采用的协议为表示本地文件的 "file" 或者 "data" URI  
- 当前请求页面采用的是 http 协议，而来源页面采用的是 https 协议  

请求的上下文: From，主要用于网络爬虫，告诉服务器如何通过邮件联系到爬虫的负责人  

From = mailbox  

# 响应的上下文

## 响应的上下文：Server  

指明服务器上所用软件的信息，用于帮助客户端定位问题或者统计数据  

Server = product *( RWS ( product / comment ) )  

- product = token ["/" product-version]  

## 响应的上下文： Allow 与 Accept-Ranges  

Allow：告诉客户端，服务器上该 URI 对应的资源允许哪些方法的执行  

Accept-Ranges：告诉客户端服务器上该资源是否允许 range 请求  

# 内容协商  

每个 URI 指向的资源可以是任何事物，可以有多种不同的表述，例如一份文档可以有不同语言的翻译、不同的媒体格式、可以针对不同的浏览器提供不同的压缩编码等。  

![](./img/content_format.png)

内容协商的两种方式  

- Proactive 主动式内容协商：指由客户端先在请求头部中提出需要的表述形式，而服务器根据这些请求头部提供特定的 representation 表述  
- Reactive 响应式内容协商：   指服务器返回 300 Multiple Choices 或者 406 Not Acceptable，由客户端选择一种表述 URI 使用   

## Proactive 主动式内容协商  

![](./img/Proactive.png)

## Reactive 响应式内容协商  

![](./img/Reactive.png)

## 常见的协商要素  

- 质量因子 q：内容的质量、可接受类型的优先级  
- 媒体资源的 MIME 类型及质量因子  

## 常见的协商要素  

- 字符编码：由于 UTF-8 格式广为使用， Accept-Charset 已被废弃  
- 内容编码：主要指压缩算法  
- 表述语言  

## 国际化与本地化  

internationalization（i18n，i 和 n 间有 18 个字符），指设计软件时，在不同的国家、地区可以不做逻辑实现层面的修改便能够以不同的语言显示   

localization（l10n，l 和 n 间有 10 个字符）， 指内容协商时，根据请求中的语言及区域信息，选择特定的语言作为资源表述  

## 资源表述的元数据头部  

媒体类型、编码  

- content-type: text/html; charset=utf-8  

内容编码  

- content-encoding: gzip  

语言  

- Content-Language: de-DE, en-CA  

# HTTP 包体：承载的消息内容  









