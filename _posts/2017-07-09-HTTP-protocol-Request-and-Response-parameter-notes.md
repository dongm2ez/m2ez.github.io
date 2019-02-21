---
layout: post
title: HTTP 协议 Request 和 Response 参数笔记
---

### Request

**请求方法**

一般的网页应用中只会用到 GET 和 POST 方法，而 RESTful 接口中会用到其他方法。

GET : 请求获取 Request-URI 所标识的资源。
POST : 在 Request-URI 所标识的资源后附加新的数据。
HEAD : 请求获取由 Request-URI 所标识的资源的响应消息报头。
PUT : 请求服务器更新一个资源，并用 Request-URI 作为其标识。
PATCH : 请求服务器更新资源的一小部分，并返回更新部分。
DELETE : 请求服务器删除 Request-URI 所标识的资源。
OPTIONS : 请求查询服务器性能，或者查询与资源相关的选项或需求。（跨域请求中会首先发送此请求，获取服务器的允许信息）
    

**请求头**


| 协议头名 | 说明 | 示例 | 状态 |
| :-: | :-: | :-: | :-: |
| Accept | 可接受的响应内容类型`（Content-Types）` | `Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8` | 固定 |
| Accept-Charset | 可接受字符集 | `Accept-Charset: utf-8` | 固定 |
| Accept-Encoding | 可接受响应内容的编码方式 | `Accept-Encoding: gzip, deflate, br` | 固定 |
| Accept-Language | 可接受响应的内容语言列表 | `Accpet-Language: zh-CN,zh;q=0.8,en;q=0.6` | 固定 |
| Accept-Datatime | 可接受的按照时间来表示的响应内容 | `Accpet-Datatiem: Sun, 09 Jul 2017 02:56:09 GMT` | 临时 |
| Authorization | 用于HTTP协议中需要认证的资源的认证凭证 | `Authorization: Basic {token}` | 固定 |
| Cache-Control | 用来指定当前的请求/响应中，是否使用缓存机制。 | `Cache-Control: no-cache` | 固定 |
| Connection | 客户端想要优先使用的连接类型 | `Connection: keep-alive` `Connection: Upgrade` | 固定 |
| Cookie | 由之前服务器 `Set-Cookie` 下发的 HTTP Cookie | `Cookie: _ga=GA1.2.413108871.1480302912`  | 固定：标准 |
| Content-Length | 请求体长度 | `Content-Length: 345` | 固定 |
| Content-Type | 请求体的 MIME 类型（用于 POST 和 PUT 请求） | `Content-Type: application/x-www-form-urlencoded`  | 固定 |
| Date | 发送该请求的日期时间 | `Sun, 09 Jul 2017 02:56:09 GMT` | 固定 |
| Expect | 要服务器做出的特定行为 | `Expect: 100-continue` | 固定 |
| Host | 表示请求服务器的域名及端口号，80端口可省略 | `Host: www.m2ez.com` `127.0.0.1:9000` | 固定 |
| Origin | 发起一个针对跨域资源共享的请求，该请求要求服务器在响应中加入一个 `Access-Control-Allow-Origin` 的消息头，表示访问控制所允许的来源） | `Origin: http://www.m2ez.com` | 固定: 标准 |
| Pragma | 与具体实现相关，可能在请求/响应的任何阶段产出 | `Pragma: no-cache` | 固定 |
| Referer | 表示浏览所访问的前一个页面，`Referer`其实是`Referrer`这个单词，但`RFC`制作标准时给拼错了，后来也就将错就错使用`Referer`了 | `Referer:https://www.google.com/` | 固定 |
| User-Agent | 浏览器身份标识 | `User-Agent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/59.0.3071.115 Safari/537.36` | 固定 |
| If-Modified-Since | 允许在对应资源未响应的情况下返回304 | `If-Modified-Since:Sun, 09 Jul 2017 02:22:53 GMT` | 固定 |


### Response

**响应状态码**

状态码由三位数字组成，首位定义了响应的类别。

* 1XX : 表示请求已经接收，继续处理。

        100 （继续）请求者应该继续请求，服务器返回此代码表示已收到请求第一部分，正在等待其余部分。
        
        101 （切换协议）请求者已要求服务器切换协议，服务器已确认并准备切换。

* 2XX : 表示请求已经成功处理请求。

        200 （成功） 服务器已成功除了请求，通常表示服务器已提供了请求的网页。
        
        201 （已创建）请求成功且服务器创建了新资源。
        
        202 （已接受）服务器已接受请求，但尚未处理。
        
        203 （非授权信息）服务器已成功处理了请求，但返回的信息可能来自另一个来源。
        
        204 （无内容）服务器成功处理了请求，但没返回任何内容。
        
        205 （重置内容）服务器成功处理了请求，但没有返回任何内容。
        
        206 （部分内容）服务器成功处理了部分 GET 请求。
        
* 3XX : 表示要完成请求，需要进一步操作。通常这些状态码是用来重定向。

        300 (多种选择) 针对请求，服务器可执行多种操作。 服务器可根据请求者 (user agent) 选择一项操作，或提供操作列表供请求者选择。

        301 (永久移动) 请求的网页已永久移动到新位置。 服务器返回此响应(对 GET 或 HEAD 请求的响应)时，会自动将请求者转到新位置。

        302 (临时移动) 服务器目前从不同位置的网页响应请求，但请求者应继续使用原有位置来进行以后的请求。

        303 (查看其他位置) 请求者应当对不同的位置使用单独的 GET 请求来检索响应时，服务器返回此代码。

        304 (未修改) 自从上次请求后，请求的网页未修改过。 服务器返回此响应时，不会返回网页内容。

        305 (使用代理) 请求者只能使用代理访问请求的网页。 如果服务器返回此响应，还表示请求者应使用代理。

        307 (临时重定向) 服务器目前从不同位置的网页响应请求，但请求者应继续使用原有位置来进行以后的请求。
        
* 4XX : （客户端错误）表示请求出错，妨碍服务器的处理。

        400 (错误请求) 服务器不理解请求的语法。

        401 (未授权) 请求要求身份验证。 对于需要登录的网页，服务器可能返回此响应。
    
        403 (禁止) 服务器拒绝请求。
        
        404 (未找到) 服务器找不到请求的网页。
        
        405 (方法禁用) 禁用请求中指定的方法。
        
        406 (不接受) 无法使用请求的内容特性响应请求的网页。
        
        407 (需要代理授权) 此状态代码与 401(未授权)类似，但指定请求者应当授权使用代理。
        
        408 (请求超时) 服务器等候请求时发生超时。
        
        409 (冲突) 服务器在完成请求时发生冲突。 服务器必须在响应中包含有关冲突的信息。
        
        410 (已删除) 如果请求的资源已永久删除，服务器就会返回此响应。
        
        411 (需要有效长度) 服务器不接受不含有效内容长度标头字段的请求。
        
        412 (未满足前提条件) 服务器未满足请求者在请求中设置的其中一个前提条件。
        
        413 (请求实体过大) 服务器无法处理请求，因为请求实体过大，超出服务器的处理能力。
        
        414 (请求的 URI 过长) 请求的 URI(通常为网址)过长，服务器无法处理。
        
        415 (不支持的媒体类型) 请求的格式不受请求页面的支持。
        
        416 (请求范围不符合要求) 如果页面无法提供请求的范围，则服务器会返回此状态代码。
        
        417 (未满足期望值) 服务器未满足"期望"请求标头字段的要求。
        
* 5XX : （服务器错误）这些状态代码表示服务器尝试处理请求时发生内部错误

        500 (服务器内部错误) 服务器遇到错误，无法完成请求。

        501 (尚未实施) 服务器不具备完成请求的功能。 例如，服务器无法识别请求方法时可能会返回此代码。

        502 (错误网关) 服务器作为网关或代理，从上游服务器收到无效响应。

        503 (服务不可用) 服务器目前无法使用(由于超载或停机维护)。 通常，这只是暂时状态。

        504 (网关超时) 服务器作为网关或代理，但是没有及时从上游服务器收到请求。

        505 (HTTP 版本不受支持) 服务器不支持请求中所用的 HTTP 协议版本。

**响应头**

| 协议头名 | 说明 | 示例 | 状态 |
| :-: | :-: | :-: | :-: |
| Access-Control-Allow-Origin | 指定哪些网站可以 `跨域源资源共享` | `Access-Control-Allow-Origin: *` | 固定 |
| Age | 响应对象在代理缓存中存在的时间，以秒为单位 | `Age: 0` | 固定 |
| Accept-Ranges | 服务器所支持的内容范围 | `Accept-Ranges: bytes` | 固定 |
| Allow | 对资源允许的请求方法 | `Allow: GET, HEAD` | 固定 |
| Cache-Control | 服务器通知客户端缓存内容时间 | `Cache-Control:max-age=600` | 固定 |
| Connection | 连接类型 | `Connection:keep-alive`| 固定 |
| Content-Disposition |	对已知MIME类型资源的描述，浏览器可以根据这个响应头决定是对返回资源的动作，如：将其下载或是打开 | `Content-Disposition: attachment; filename="fname.ext"` | 固定 |
| Content-Encoding | 响应资源所使用的编码类型 | `Content-Encoding: gzip` | 固定 |
| Content-Language | 响就内容所使用的语言 | `Content-Language: zh-cn` | 固定 |
| Content-Length | 响应消息体的长度 | `Content-Length: 348` | 固定 |
| Content-Location | 所返回的数据的一个候选位置 | `Content-Location: /index.htm` | 固定  |
| Server |	服务器的名称 | `Server:nginx/1.11.3` | 固定 |
| Set-Cookie | 设置 HTTP cookie | `Set-Cookie: XSRF-TOKEN=XX;Max-Age=7200;` | 固定: 标准 |
| Content-Type |	当前内容的MIME类型 | `Content-Type: text/html; charset=utf-8` | 固定 |
| Date | 此条消息被发送时的日期和时间 | `Date:Sun, 09 Jul 2017 03:21:11 GMT` | 固定 |
| ETag | 	对于某个资源的某个特定版本的一个标识符，通常是一个 消息散列 | `ETag: "XXXX"` | 固定 |
| Expires | 指定一个日期/时间，超过该时间则认为此回应已经过期 | `Expires: Sun, 09 Jul 2017 03:21:11 GMT` | 固定: 标准 |
| Last-Modified | 请求的对象的最后修改日期 | `Last-Modified: Sun, 09 Jul 2017 03:21:11 GMT` | 固定 |
| Location | 用于在进行重定向 | `Location: http://www.m2ez.com` | 固定 |
| Refresh | 用于重定向跳转，可设定时间 | `Refresh: 5; url=http://www.m2ez.com` | 固定 |
| Vary | 告知下游的代理服务器，应当如何对以后的请求协议头进行匹配，以决定是否可使用已缓存的响应内容而不是重新从原服务器请求新的内容 | `Vary: Accept-Encoding` | 固定 |
| Via |	告知代理服务器的客户端，当前响应是通过什么途径发送的 | `1.1 varnish` | 固定 |

### 非标准头

基本以 X 开头的请求头都是非标准请求头，由开发者定义，但是一些非标准请求头实际上已经是事实标准。

