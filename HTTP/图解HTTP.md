## HTTP报文首部

### 请求报文

在请求中，HTTP报文由`方法`、`URI`、`HTTP版本`、`HTTP首部字段`等部分构成

![请求报文](https://raw.githubusercontent.com/aboutcroon/Notes/main/HTTP/assets/request%20message.png)



### 响应报文

在响应中，HTTP报文由`HTTP版本`、`状态码（数字和原因短语）`、`HTTP首部字段`3部分构成。

![响应报文](https://raw.githubusercontent.com/aboutcroon/Notes/main/HTTP/assets/response%20message.png)



### 4种HTTP首部字段类型

HTTP/1.1 规范定义了如下 47 种首部字段。

HTTP首部字段根据实际用途被分为以下4种类型。

#### 通用首部字段（General Header Fields）：

请求报文和响应报文两方都会使用的首部。

![通用首部字段](https://raw.githubusercontent.com/aboutcroon/Notes/main/HTTP/assets/universal%20header.png)

#### 请求首部字段（Request Header Fields）：

从客户端向服务器端发送请求报文时使用的首部。补充了请求的附加内容、客户端信息、响应内容相关优先级等信息。

![请求首部字段](https://raw.githubusercontent.com/aboutcroon/Notes/main/HTTP/assets/request%20header.png)

#### 响应首部字段（Response Header Fields）：

从服务器端向客户端返回响应报文时使用的首部。补充了响应的附加内容，也会要求客户端附加额外的内容信息。

![响应首部字段](https://raw.githubusercontent.com/aboutcroon/Notes/main/HTTP/assets/response%20header.png)

#### 实体首部字段（Entity Header Fields）：

针对请求报文和响应报文的实体部分使用的首部。补充了资源内容更新时间等与实体有关的信息。

![实体首部字段](https://raw.githubusercontent.com/aboutcroon/Notes/main/HTTP/assets/response%20entity%20header.png)

### 非HTTP/1.1首部字段

在HTTP协议通信交互中使用到的首部字段，不限于RFC2616中定义的47种首部字段。还有`Cookie`、`Set-Cookie`和`Content-Disposition`等在其他RFC中定义的首部字段，它们的使用频率也很高。这些非正式的首部字段统一归纳在RFC4229 HTTP Header Field Registrations中。

HTTP首部字段将定义成缓存代理和非缓存代理的行为，分成2种类型。

#### 端到端首部

分在此类别中的首部会转发给请求/响应对应的最终接收目标，且必须保存在由缓存生成的响应中，另外规定它必须被转发。

#### 逐跳首部

分在此类别中的首部只对单次转发有效，会因通过缓存或代理而不再转发。HTTP/1.1和之后版本中，如果要使用hop-by-hop首部，需提供Connection首部字段。

下面列举了HTTP/1.1中的逐跳首部字段。除这8个首部字段之外，其他所有字段都属于端到端首部。

- Connection
- Keep-Alive
- Proxy-Authenticate
- Proxy-Authorization
- Trailer
- TE
- Transfer-Encoding
- Upgrade



## HTTPS

HTTP主要有这些不足，例举如下：

- 通信使用明文（不加密），内容可能会被窃听

- 不验证通信方的身份，因此有可能遭遇伪装

- 无法证明报文的完整性，所以有可能已遭篡改



HTTP + 通信加密 + 证书 + 完整性保护 = HTTPS

HTTPS 并非是应用层的一种新协议。只是HTTP通信接口部分用SSL（Secure Socket Layer）和TLS（Transport Layer Security）协议代替而已。

通常，HTTP 直接和 TCP 通信。当使用 SSL 时，则演变成先和 SSL 通信，再由 SSL 和 TCP 通信了。简言之，所谓 HTTPS，其实就是身披 SSL 协议这层外壳的 HTTP。

![HTTPS](https://raw.githubusercontent.com/aboutcroon/Notes/main/HTTP/assets/https.png)

在采用 SSL 后，HTTP 就拥有了 HTTPS 的加密、证书和完整性保护这些功能。

