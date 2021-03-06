## 读书笔记

1.三项 WWW 构建技术：

```
1.把 SGML 作为页面的文本标记语言的 HTML
2.作为文档传递协议的 HTTP
3.指定文档所在地址的 URL
```

2.当年 http 协议的出现主要是为了解决文本传输的难题。由于协议本身非常简单，于是在此基础上设想了很多应用方法并投入了实际使用。现在 http 协议已经超出了 Web 这个框架的局限，被运用到了各种场景里。

3.TCP/IP 的分层：

```
1.应用层。决定了向用户提供应用服务时通信的活动
2.传输层。提供处于网络连接中的两台计算机之间的数据传输
3.网络层。用来处理在网络上流动的数据包
4.链路层。用来处理连接网络的硬件部分
```

4.IP 协议的作用是把各种数据宝传送给对方。而要保证确实传送到对方那里，则需要满足各类条件。其中两个重要的条件是 IP 地址和 MAC 地址。IP 协议位于网络层。

5.TCP 提供可靠的字节流服务，为了方便传输，将大块数据分割成以报文段为单位的数据包进行管理，并且准备可靠的传给对方。TCP 位于传输层。

6.DNS 服务和 HTTP 协议一样位于应用层的协议，它提供域名到 IP 地址之间的解析服务。

7.采用 http 协议时，协议方案就是 http。除此之外，还有 ftp、mailto、telnet、file。

8.URI 用字符串标识某一互联网资源，而 URL 表示资源的地点。可见 URL 是 URI 的子集。常见的 URI 如下所示：

```
// 是 URL
ftp://ftp.is.co.za/rfc/rfc1808.txt

// 不是 URL
tel:+1-816-555-1212
news:comp.infosystems.www.servers.unix
```

9.在两台计算机之间使用 http 协议通信时，在一条通信线路上必定有一端是客户端，另一端则是服务器端。

10.告知服务器意图的 http 方法：

```
1.GET 方法用来请求访问已被 URI 识别的资源。指定资源经服务器端解析后返回响应内容
2.POST 方法用来传输实体的主体，它的主要目的并不是获取响应的主体内容
3.HEAD 方法和 GET 方法一样，只是不返回报文主体部分，用于确认 URI 的有效性及资源更新的日期时间等
4.OPTIONS 方法用来查询针对请求 URI 指定的资源支持的方法
5.TRACE 方法可以查询发送出去的请求是怎样呗加工修改/篡改的。这是因为，请求想要连接到源目标服务器可能会通过代理中转，TRACE 方法就是用来确认连接过程中发生的一系列操作。
6.CONNECT 方法要求在与代理服务器通信时建立隧道，实现用隧道协议进行 TCP 通信。主要使用 SSL 和 TLS 协议把通信内容加密后经网络隧道传输
```

11.持久连接的特点是，只要任意一端没有明确提出断开连接，则保持 TCP 连接状态。持久连接使得多数请求以管线化方式发送成为可能。从前发送请求后需等待并收到响应，才能发送下一个请求。管线化技术出现后，不用等待响应亦可直接发送下一个请求。

12.通常，报文主体等于实体主体。只有当传输中进行编码操作时，实体主体的内容发生变化，才导致它和报文主体产生差异。

13.http 协议中采纳了多部分对象集合，发送的一份报文主体内可含有多类型实体。通常是在图片或文本文件等上传时使用。

14.获取部分内容的范围请求需要指定下载的实体范围，像这样，指定范围发送的请求叫做范围请求。执行范围请求时，会用到首部字段 Range 来指定资源的 byte 范围。

15.当浏览器的默认语言为英语或中文，访问相同 URI 的 WEB 页面时，则会显示对应的英语版或中文版的 WEB 页面。这样的机制称为内容协商。

16.状态码的职责是当客户端向服务器端发送请求时，描述返回的请求结果。实际上经常使用的状态码大概只有 14 种：

```
// 2XX 成功
1.200 OK 表示从客户端发来的请求在服务器端被正常处理了
2.204 No Content 表示服务器接收的请求已成功处理，并且对客户端不需要发送新信息内容
3.206 Partial Content 表示客户端进行了范围请求，而服务器成功执行了这部分的 GET 请求

// 3XX 重定向
4.301 Moved Permanently 表示请求的资源已被分配了新的 URI，以后应使用资源现在所指的 URI
5.302 Found 表示请求的资源已被分配了新的 URI，但是不是永久移动，只是临时性质的
6.303 See Other 表示请求的资源已被分配了新的 URI，并且希望使用 GET 方法获取新资源
7.304 Not Modified 表示客户端发送的条件请求中，服务端资源未改变，可直接使用客户端未过期的缓存
8.307 Temporary Redirect 表示请求的资源已被分配了新的 URI，并且希望仍用以前的方法获取新资源

// 4XX 客户端错误
9.400 Bad Request 表示请求报文中存在语法错误。（浏览器会像200 OK一样对待该状态码）
10.401 Unauthorized 表示该请求需要通过 http 认证
11.403 Forbidden 表示该请求被服务器拒绝了
12.404 Not Found 表示服务器上无法找到请求的资源

// 5XX 服务器错误
13.500 Internal Server Error 表示服务器端在执行请求时发生了错误
14.503 Service Unavailable 表示服务器暂时处理超负载或正在进行停机维护，现在无法处理请求。
```

17.303 和 307 是 http1.1 新加的状态码，它是对 302 的细化。

18.在响应报文内，随状态码一起返回的信息会因为方法的不同而发生改变。比如使用 GET 方法时，对应请求资源的实体会作为响应返回；而使用 HEAD 方法时，对应请求资源的实体主体不随报文首部作为响应返回。

19.每次通过代理服务器转发请求或响应时，会追加写入 Via 首部信息

20.转发请求或响应时，不对报文做任何加工的代理类型被称为透明代理。

21.网关的工作机制和代理十分相似，而网关能使通信线路上的服务器提供非 HTTP 协议服务。

22.HTTP 首部字段根据实际用途被分为以下4种类型：

```
1.通用首部字段
2.请求首部字段
3.响应首部字段
4.实体首部字段
```

23.通过指定首部字段 Cache-Control 指令，就能操作缓存的工作机制。

24.HTTP1.1 会优先处理 max-age 指令，而忽略掉 Expires 首部字段；而 HTTP1.0 则会忽略掉 max-age

25.Pragma 是 HTTP1.1 之前版本的历史遗留字段，仅作为与 HTTP1.0 的向后兼容而定义的。如果所有的中间服务器都能以 HTTP1.1 为基准，那直接采用 Cached-Control 是最为理想的，但是这是不现实的，所以发送的请求会同时含有下面两个首部字段：

```
Cache-Control: no-cache
Pragma: no-cache
```

26.Host 首部字段在 HTTP1.1 规范内是唯一一个必须被包含在请求内的首部字段

27.首部字段 If-Match，属附带条件之一，它会告知服务器匹配资源所用的实体标记值。还可以使用星号（*）指定If-Match的字段值。针对这种情况，服务器将会忽略 ETag 值，只要资源存在就处理请求。

28.通过 TRACE 方法或 OPTIONS 方法，发送包含首部字段 Max-Forwards 的请求时，该字段以十进制整数形式指定可经过的服务器最大数目。

29.接收到附带 Range 首部字段请求的服务器，会在处理请求之后返回状态码为 206 Partial Content 的响应。无法处理该范围请求时，则会返回状态码200 OK的响应及全部资源。

30.响应首部字段是由服务器端向客户端返回响应报文中所使用的字段，用于补充响应的附加信息、服务器信息，以及对客户端的附加要求等信息。

31.首部字段 Accept-Ranges 是用来告知客户端服务器能否处理范围请求，以指定获取服务器端某个部分的资源。

32.首部字段 Age 能告知客户端，源服务器在多久前创建了响应。字段值的单位为秒。

33.当资源更新时，ETag 值也需要更新。生成 ETag 值时，并没有统一的算法规则，而仅仅是由服务器来分配。

34.弱 ETag 值只用于提示资源是否相同。只有资源发生了根本改变，产生差异时才会改变ETag。这时，会在字段值最开始处附加W/。

35.使用首部字段 Location 可以将响应接收方引导至某个与请求 URI 位置不同的资源。

36.实体首部字段是包含在请求报文和响应报文中的实体部分所使用的首部，用于补充内容的更新时间等与实体相关的信息。

37.首部字段Content-Length表明了实体主体部分的大小。对实体主体进行内容编码传输时，不能再使用 Content-Length 首部字段。

38.和首部字段 Location 不同， Content-Location 表示的是报文主体返回资源对应的 URI。

39.首部字段 Content-MD5 是一串由 MD5 算法生成的值，其目的在于检查报文主体在传输过程中是否保持完整，以及确认传输到达。

40.Cookie 的工作机制是用户识别及状态管理。

41.一旦 Cookie 从服务器端发送至客户端，服务器端就不存在可以显式删除 Cookie 的方法。但可通过覆盖已过期的 Cookie，实现对客户端 Cookie 的实质性删除操作。

42.Cookie 的 secure 属性用于限制 Web 页面仅在 HTTPS 安全连接时，才可以发送 Cookie。

43.Cookie 的 httponly 属性是 Cookie 的扩展功能，它使 Javascript 脚本无法获得 Cookie。其主要目的为防止跨站脚本共计对 Cookie 的信息窃取。

44.HTTP 首部字段是可以自行扩展的，所以在 Web 服务器和浏览器的应用上，会出现各种非标准的首部字段：

```
X-Frame-Options
X-XSS-Protection
DNT
P3P
```

45.首部字段 X-Frame-Options 属于 HTTP 响应首部，用于控制网站内容在其它 Web 网站的 Frame 标签内的显示问题，其主要目的是为了防止点击劫持攻击。

46.首部字段 X-XSS-Protection 属于 HTTP 响应首部，它是针对跨站脚本攻击（XSS）的一种对策，用于控制浏览器 XSS 防护机制的开关。

47.首部字段 DNT 属于 HTTP 请求首部，其中 DNT 是 Do Not Track 的简称，意为拒绝个人信息被收集，是表示拒绝被精准广告追踪的一种方法。

48.首部字段 P3P 属于 HTTP 响应首部，通过利用 P3P 技术，可以让 Web 网站上的个人隐私变成一种仅供程序可理解的形式，以达到保护用户隐私的目的。

49.HTTP 主要有这些不足，例举如下：

```
1.通信使用明文，内容可能会被窃听
2.不验证通信方的身份，因此有可能遭遇伪装
3.无法证明报文的完整性，所以有可能已遭篡改
```

50.SSL 不仅提供加密处理，而且还使用了一种被称为证书的手段，可用于确定通信方。SSL 位于表示层。

51.HTTPS 并非是应用层的一种新协议，只是HTTP通信接口部分用 SSL 和 TLS 协议代替而已。

52.SSL 是独立于 HTTP 的协议，所以不光是 HTTP 协议，其它运行在应用层的 SMTP 和 Telnet 等协议均可配合 SSL 协议使用。

53.HTTP 采用混合加密机制，在交换秘钥环节使用公开密钥加密方式，之后的建立通信交换报文阶段则使用共享密钥加密方式。

54.应用层发送数据时会附加一种叫做MAC(Message Authentication Code)的报文摘要，MAC能够查知报文是否遭到篡改，从而保护报文的完整性。

55.BASIC 认证是从HTTP1.0就定义的认证方式。它把Base64加密后的字符串写入首部Authorization后，发送请求。它的缺点是不需要任何附加信息就可对其解码，并且无法实现注销操作。

56.所谓质询响应方式是指，一开始一方会先发送认证要求给另一方，接着使用从另一方那接收到的质询码计算生成响应码，最后将响应码返回给对方进行认证的方式。

57.DIGEST 认证是使用了质询响应的认证方式，它提供了防止密码被窃听的保护机制，但并不存在防止用户伪装的保护机制。

58.SSL 客户端认证是借由HTTPS的客户端证书完成认证的方式。凭借客户端证书认证，服务器可确认访问是否来自已登录的客户端。

59.在多数情况下，SSL 客户端认证不会仅依靠证书完成认证，一般会和基于表单认证组合形成一种双因素认证来使用。

60.一种安全的保存方法是，先利用给密码加盐的方式增加额外信息，再使用散列函数计算出散列值后保存。

61.若想在现有 Web 实现所需的功能，以下这些 HTTP 标准就会成为瓶颈：

```
1.一条连接上只可发送一个请求
2.请求只能从客户端开始。客户端不可以接收除响应外的指令
3.请求/响应首部未经压缩就发送，首部信息越多延迟越大
4.发送冗长的首部，每次互相发送相同的首部造成的浪费较多
5.可任意选择数据压缩格式，非强制压缩发送。
```

62.Comet 是一种通过延迟应答，模拟实现服务端向客户端推送（Server Push）的功能。

63.SPDY 协议具有以下额外功能：

```
1.多路复用流：通过单一的 TCP 连接，可以无限制处理多个 HTTP 请求，所有请求的处理都在一条 TCP 连接上完成，因此TCP的处理效率得到提高
2.赋予请求优先级：可以给请求逐个分配优先级顺序
3.压缩 HTTP 首部：压缩 HTTP 请求和响应的首部
4.推送功能：支持服务器主动向客户端推送数据的功能
5.服务器提示功能：服务器可以主动提示客户端请求所需的资源
```

64.WebSocket 技术主要是为了解决 Ajax 和 Comet 里 XMLHttpRequest 附带的缺陷所引起的问题。它的主要特点是：

```
1.推送功能
2.减少通信量
```

65.为了实现 WebSocket 通信，在 HTTP 连接建立之后，需要完成一次握手的步骤：

```
1.握手请求：需要用到 HTTP 的 Upgrade 首部字段，告知服务器通信协议发生改变
2.握手响应：对于之前的请求，返回状态码101 Switching Protocols 的响应
```

66.防火墙的基本功能就是禁止非指定的协议和端口号的数据包通过，因此如果使用新协议或端口号则必须修改防火墙设置，比如在构建 Web 服务器时，需事先设置防火墙 HTTP（80/tcp）和 HTTPS（443/tcp）的权限。

67.DOM 是用以操作 HTML 文档和 XML 文档的 API。

68.RSS（简易信息聚合，也叫聚合内容）和 Atom 都是发布新闻或博客日志等更新信息文档的格式的总称，两者都用到了 XML。

69.SSH 协议具备协议级别的认证和会话管理等功能，HTTP 协议则没有。

70.主动攻击模式里具有代表性的攻击是 SQL 注入攻击和 OS 命令注入攻击。被动攻击模式中具有代表性的攻击是跨站脚本攻击和跨站点请求伪造。

```
1.SQL 注入是指针对 Web 应用使用的数据库，通过运行非法的 SQL 而产生的攻击
2.OS 命令注入攻击是指通过 Web 应用，执行非法的操作系统命令达到攻击的目的。只要在能调用 Shell 函数的地方就有存在被攻击的风险
3.HTTP 首部注入攻击是指攻击者通过在响应首部字段内插入换行，添加任意响应首部或主题的一种攻击
4.邮件首部注入是指 Web 应用中的邮件发送功能，攻击者通过向邮件首部 To 或 Subject 内任意添加非法内容发起的攻击。利用存在安全漏洞的 Web 网站，可对任意邮件地址发送广告邮件或病毒邮件
5.目录遍历攻击是指对本无意公开的文件目录，通过非法截断其目录路径后，达成访问目的的一种攻击。
6.远程文件包含漏洞是指当部分脚本内容需要从其他文件读入时，攻击者利用指定外部服务器的 URL 充当依赖文件，让脚本读取之后，就可运行任意脚本的一种攻击。
7.可信度高的 Web 网站如果开放重定向功能，则很有可能被攻击者选中并用来作为钓鱼攻击的跳板
```

71.不适合将 Javascript 验证作为安全的防范对策。保留客户端验证只是为了尽早地辨识输入错误，起到提高 UI 体验的作用。

72.彩虹表是由明文密码及与之对应的散列值构成的一张数据库表，是一种通过事先制作庞大的彩虹表，，可在穷举法、字典攻击等实际破解过程中缩短消耗时间的技巧。

73.点击劫持是指利用透明的按钮或链接做成陷阱，覆盖在 Web 页面之上，然后诱使用户在不知情的情况下，点击那个链接访问内容的一种攻击手段，这种行为又称为界面伪装。

74.DoS 攻击是一种让运行中的服务呈停止状态的攻击，有时也叫做服务停止攻击或拒绝服务攻击
