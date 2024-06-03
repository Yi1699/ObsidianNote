#### HTTP协议概述

WEB浏览器与WEB服务器之间的一问一答的交互过程必须遵循一定的规则，这个规则就是HTTP协议。

HTTP是hypertext transfer protocol（超文本传输协议）的简写，它是TCP/IP协议之上的一个应用层协议，用于定义WEB浏览器与WEB服务器之间交换数据的过程以及数据本身的格式。

- HTTP协议到底约束了什么:
1. 约束了浏览器以何种格式向服务端发生数据:
2. 约束了服务器应该以何种格式来接受客户端发送的数据:
3. 约束了服务器应该以何种格式来反馈数据给浏览器;
4. 约束了浏览器应该以何种格式来接收服务器反馈的数据.


- HTTP1.0规范:
	若请求的有N个资源,得建立N次连接,发送N次请求,接收N次响应,关闭N次连接.
	每次请求的之间都要建立单独的连接,请求,响应,响应完关闭该次连接:
	缺点:每请求一个资源都要单独的建立新的连接,请求完并关闭连接.

- HTTP1.1规范:
	能在一次连接之间,多次请求,多次响应,响应完之后再关闭连接.
	- HTTP 1.1的特点
	在一个TCP连接上可以传送多个HTTP请求和响应
	多个请求和响应过程可以重叠进行
	增加了更多的请求头和响应头

- HTTP协议的版本
	HTTP/1.0、HTTP/1.1、HTTPS2.0


### 请求信息
##### 一个完整的由客户端发给服务器的HTTP请求中包含以下内容:

- 请求行
	包含了请求方法、请求资源路径、HTTP协议版本
	GET /MJServer/resources/images/1.jpg HTTP/1.1
- 多个请求头
	包含了对客户端的环境描述、客户端请求的主机地址等信息
	Accept：浏览器可接受的MIME类型（Tomcat安装目录/conf/web.xml中查找）注意: MIME: 表示文件内容的类型.
	Accept-Charset：告知服务器，客户端支持哪种字符集
	Accept-Encoding：浏览器能够进行解码的数据编码方式
	Accept-Language：浏览器支持的语言。
	Referer：当前页面由哪个页面访问过来的。
	Content-Type：通知服务器，请求正文的MIME类型。
	Content-Length：请求正文的长度
	If-Modified-Since:通知服务器，缓存的文件的最后修改时间。
	User-Agent:通知服务器，浏览器类型.
	Connection:表示是否需要持久连接。如果服务器看到这里的值为“Keep -Alive”，或者看到请求使用的是HTTP 1.1（HTTP 1.1默认进行持久连接)
	Cookie:这是最重要的请求头信息之一（会话有关)


### 响应信息

客户端向服务器发送请求，服务器应当做出响应，即返回数据给客户端:
- 状态行
	包含了HTTP协议版本、状态码、状态英文名称
	HTTP/1.1 200 OK
	多个响应头
	包含了对服务器的描述、对返回数据的描述
	Location：制定转发的地址。需与302/307响应码一同使用
	Server：告知客户端服务器使用的容器类型
	Content-Encoding：告知客户端服务器发送的数据所采用的压缩格式
	Content-Length：告知客户端正文的长度
	Content-Type：告知客户端正文的MIME类型
	Conent-Type:text/html;charset=UTF-8
	Refresh：定期刷新。还可以刷新到其他资源
	Refresh:3;URL=otherurl
	3秒后刷新到otherurl这个页面
	Content-Disposition：指示客户端以下载的方式保存文件。
	Content-Disposition：attachment;filename=2.jpg
	Expires：网页的有效时间。单位是毫秒(等于-1时表示页面立即过期)
	Cache-Control：no-cache
	Pragma：no-cache
	控制客户端不要缓存
	Set-Cookie:SS=Q0=5Lb_nQ; path=/search服务器端发送的Cookie（会话有关）
- 请求实体
	服务器返回给客户端的具体数据，比如文件数据.


### 常见状态响应码

状态码	英文名称	中文描述
200	OK	请求成功
400	Bad Request	客户端请求的语法错误,服务器无法解析
404	Not Found	服务器无法根据客户端的请求找到资源
500	Internal Server Error	服务器内部错误,无法完成请求
GET请求和POST请求
### 一、简单说明
在HTTP/1.1协议中，定义了8种发送http请求的方法
GET、POST、OPTIONS、HEAD、PUT、DELETE、TRACE、CONNECT、PATCH
根据HTTP协议不同的方法对资源有不同的操作方式
- PUT ：增
- DELETE ：删
- POST：改
- GET：查

`提示：`最常用的是`GET和POST`（实际上GET和POST都能办到增删改查）
### 二、GET请求

请求的数据全部在浏览器的地址栏(很不安全)
```
http://localhost:8080/webapp/?username=coder&email=111#
```
请求信息全部会存储在请求行中
注意：由于浏览器和服务器对URL长度有限制，因此在URL后面附带的参数是有限制的，通常不能超过2KB
参数连接
资源?参数名=参数值值&参数名=参数值&…
### 三、POST请求

请求的数据不会出现在浏览器的地址栏中(比较安全)
请求信息会全部存储到请求实体中
GET和POST请求的区别:

#### 四、两者区别

GET的请求数据在地址栏中,而POST不会
POST比GET更安全
POST请求的参数存放于请求实体中, 而GET存放在请求行中.
GET请求的数据不能超过2K, 而POST没有上限
文件上传时,必须使用POST方式
GET可以缓存, 而POST没有缓存
#### 五、如何选择GET和POST

如果要传递大量数据，比如文件上传，只能用POST请求
GET的安全性比POST要差些，如果包含机密\敏感信息，建议用POST
如果仅仅是索取数据（数据查询），建议使用GET
如果是增加、修改、删除数据，建议使用POST
HTTP无状态连接

- 一次会话
打开一个浏览器,访问某一个站点,在该网址内部查看信息,点击超链接等相关操作,最后关闭浏览器的整个过程,称之为一次会话.

- HTTP协议
它是无状态连接,服务器不知道上一次是哪个客户端请求了自己.

- 无状态连接带来的问题:
在一次会话中,多个请求之间无法共享数据,无法跟踪用户的会话信息.
在一次会话中共享数据即会话跟踪技术.

- 解决方案

>使用参数的传递机制
>在每一个请求之间使用参数来传递需要共享的数据.http://localhost/param/list?username=gzy
>可以解决问题,但请求需要共享的数据全部暴露在URL中(请求行),不安全.想要解决这个问题,把共享的数据存放到请求头中. 期待Cookie技术

----
### 断点续传
#### 断点续传的用途

有时用户上传/下载文件需要历时数小时，万一线路中断，不具备断点续传的 HTTP/FTP 服务器或下载软件就只能从头重传，比较好的 HTTP/FTP 服务器或下载软件具有断点续传能力，允许用户从上传/下载断线的地方继续传送，这样大大减少了用户的烦恼。

常见的支持断点续传的上传/下载软件：QQ 旋风、迅雷、快车、电驴、酷6、土豆、优酷、百度视频、新浪视频、腾讯视频、百度云等。

在 Linux/Unix 系统下，常用支持断点续传的 FTP 客户端软件是 lftp。

#### Range & Content-Range

HTTP1.1 协议（RFC2616）开始支持获取文件的部分内容，这为并行下载以及断点续传提供了技术支持。它通过在 Header 里两个参数实现的，客户端发请求时对应的是 Range ，服务器端响应时对应的是 Content-Range。

**Range**

用于请求头中，指定第一个字节的位置和最后一个字节的位置，一般格式：

> Range:(unit=first byte pos)-[last byte pos]

Range 头部的格式有以下几种情况：

> Range: bytes=0-499 表示第 0-499 字节范围的内容   
> Range: bytes=500-999 表示第 500-999 字节范围的内容   
> Range: bytes=-500 表示最后 500 字节的内容   
> Range: bytes=500- 表示从第 500 字节开始到文件结束部分的内容   
> Range: bytes=0-0,-1 表示第一个和最后一个字节   
> Range: bytes=500-600,601-999 同时指定几个范围

**Content-Range**

用于响应头中，在发出带 Range 的请求后，服务器会在 Content-Range 头部返回当前接受的范围和文件总大小。一般格式：

> Content-Range: bytes (unit first byte pos) - [last byte pos]/[entity legth]

例如：

> Content-Range: bytes 0-499/22400

0－499 是指当前发送的数据的范围，而 22400 则是文件的总大小。

而在响应完成后，返回的响应头内容也不同：

> HTTP/1.1 200 Ok（不使用断点续传方式）   
> HTTP/1.1 206 Partial Content（使用断点续传方式）

#### 增强校验

在实际场景中，会出现一种情况，即在终端发起续传请求时，URL 对应的文件内容在服务器端已经发生变化，此时续传的数据肯定是错误的。如何解决这个问题了？显然此时需要有一个标识文件唯一性的方法。

在 RFC2616 中也有相应的定义，比如实现 Last-Modified 来标识文件的最后修改时间，这样即可判断出续传文件时是否已经发生过改动。同时 FC2616 中还定义有一个 ETag 的头，可以使用 ETag 头来放置文件的唯一标识。

#### Last-Modified

If-Modified-Since，和 Last-Modified 一样都是用于记录页面最后修改时间的 HTTP 头信息，只是 Last-Modified 是由服务器往客户端发送的 HTTP 头，而 If-Modified-Since 则是由客户端往服务器发送的头，可以看到，再次请求本地存在的 cache 页面时，客户端会通过 If-Modified-Since 头将先前服务器端发过来的 Last-Modified 最后修改时间戳发送回去，这是为了让服务器端进行验证，通过这个时间戳判断客户端的页面是否是最新的，如果不是最新的，则返回新的内容，如果是最新的，则返回 304 告诉客户端其本地 cache 的页面是最新的，于是客户端就可以直接从本地加载页面了，这样在网络上传输的数据就会大大减少，同时也减轻了服务器的负担。

#### Etag

Etag（Entity Tags）主要为了解决 Last-Modified 无法解决的一些问题。

1. 一些文件也许会周期性的更改，但是内容并不改变（仅改变修改时间），这时候我们并不希望客户端认为这个文件被修改了，而重新 GET。
2. 某些文件修改非常频繁，例如：在秒以下的时间内进行修改（1s 内修改了 N 次），If-Modified-Since 能检查到的粒度是 s 级的，这种修改无法判断（或者说 UNIX 记录 MTIME 只能精确到秒）。
3. 某些服务器不能精确的得到文件的最后修改时间。

为此，HTTP/1.1 引入了 Etag。Etag 仅仅是一个和文件相关的标记，可以是一个版本标记，例如：v1.0.0；或者说 “627-4d648041f6b80” 这么一串看起来很神秘的编码。但是 HTTP/1.1 标准并没有规定 Etag 的内容是什么或者说要怎么实现，唯一规定的是 Etag 需要放在 “” 内。

#### If-Range

用于判断实体是否发生改变，如果实体未改变，服务器发送客户端丢失的部分，否则发送整个实体。一般格式：

> If-Range: Etag | HTTP-Date

也就是说，If-Range 可以使用 Etag 或者 Last-Modified 返回的值。当没有 ETage 却有 Last-modified 时，可以把 Last-modified 作为 If-Range 字段的值。

例如：

> If-Range: “627-4d648041f6b80”   
> If-Range: Fri, 22 Feb 2013 03:45:02 GMT

If-Range 必须与 Range 配套使用。如果请求报文中没有 Range，那么 If-Range 就会被忽略。如果服务器不支持 If-Range，那么 Range 也会被忽略。

如果请求报文中的 Etag 与服务器目标内容的 Etag 相等，即没有发生变化，那么应答报文的状态码为 206。如果服务器目标内容发生了变化，那么应答报文的状态码为 200。

用于校验的其他 HTTP 头信息：If-Match/If-None-Match、If-Modified-Since/If-Unmodified-Since。

#### 工作原理

Etag 由服务器端生成，客户端通过 If-Range 条件判断请求来验证资源是否修改。请求一个文件的流程如下：

第一次请求：

1. 客户端发起 HTTP GET 请求一个文件。
2. 服务器处理请求，返回文件内容以及相应的 Header，其中包括 Etag（例如：627-4d648041f6b80）（假设服务器支持 Etag 生成并已开启了 Etag）状态码为 200。

第二次请求（断点续传）：

1. 客户端发起 HTTP GET 请求一个文件，同时发送 If-Range（该头的内容就是第一次请求时服务器返回的 Etag：627-4d648041f6b80）。
2. 服务器判断接收到的 Etag 和计算出来的 Etag 是否匹配，如果匹配，那么响应的状态码为 206；否则，状态码为 200。