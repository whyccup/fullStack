## 遇到问题怎么解决问题，事件绑定，打桩，debug，网络错误代码分析

1. 打印
1. 断点
1. 性能分析
1. chrom Performance
1. 青花瓷抓包
1. 单元测试
1. 测试框架 jest

## 从输入url到页面展示的详细过程

1. 输入网址
    1. 当输入地址时，浏览器会根据关键字，从历史记录，书签等地方，找到可能对应的url，并给予用户提示。
        1. 对于 Chrome 浏览器，它甚至会把页面从缓存中展示出来。
        1. 缓存放在 用户的appdata中
    1. 用户发起请求
2. 浏览器查找域名的IP地址
    1. 请求发起后，浏览器首先解析这个域名，先查看本地的hosts文件，若其中有该域名的对应规则，则直接使用hosts中的ip地址。
    1. 若没在host文件中找到，则会发出一个DNS请求到本地DNS服务器。本地DNS服务器一般都是你的网络接入服务商提供。
    3. 当输入的url的DNS请求到达本地DNS之后，服务会首先查询它的缓存记录，如果缓存中有此条记录，就直接返回结果（递归方式查询）。若没有，本地DNS还要向根DNS进行查询。
    4. 根DNS没有记录具体的域名和IP地址的对应关系，而是告诉本地DNS, 去域服务器上继续查询，并给出域服务器的地址。这种过程是迭代的过程。
    5. 向域服务器请求后，（.com域，.cn域等），告诉本地DNS，输入的url（域名）的解析服务器的地址。
    6. 最后本地DNS向域名的解析服务器发出请求，对方返回和域名对应的IP地址，本地DNS不仅要把IP地址返回给用户电脑，还要将域名（url）和 IP 之间的关系缓存，以加快下次访问。
        1. DNS
            1. DNS 因特网上作为域名和IP地址相互映射的一个分布式数据库，能够使用户更方便的访问互联网，而不用去记住能够被机器直接读取的IP数串。
                1. DNS 类似通讯录，备注是域名，电话是IP。
        1. DNS 查询方式 递归（一查到底，返回正确IP） 迭代 （查到下一层DNS，就返回，让客户端继续查）
        1. DNS 负载均衡
            1. 当用户足够多时，每次请求的资源都位于同一台机器上会很容易崩坏。
            1. 负载均衡技术，在DNS服务器中为同一个域名配置多个ip地址，在应答时，将ip地址按顺序返回，将客户端引导到不同的机器上去，从而达到负载均衡的目的。
                1. 或根据 负载量 用地理位置 等。
3. 浏览器向 web 服务器发送一个 HTTP 请求
    1. 有IP之后，浏览器会以一个随机端口，向向对方服务器（nginx等）的80端口发起TCP的连接请求。
    1. 此请求通过路由设备到达服务器端后，进入到网卡，进入到其内核的TCP/IP协议栈（用于识别该连接请求，解封包，一层层剥开），还有可能经过防火墙的过滤，最终到达web程序，最终建立起 TCP（卡车）/IP（马路） 的连接。
    1. 建立连接后，发起一个http（货物）请求。一般会包含3个部分：
        1. 请求方法、URL、协议、版本
        1. 请求头
        1. 请求正文
    1. 三次握手
        1. 客户端给服务端发起 SYN=1，seq=j（j为数字）数据包，进入 SYN_SENT 状态，等待服务端确认。
        1. 服务端收到包后，发回 SYN=1，ACK=1，ack=j+1，seq=K，进入 SYN_RCVD 状态。
        1. 客户端收到包后，检查 ack 是否为 j+1, ACK 是否为 1，若正确则发回 ACK=1 ack=k+1 的包。服务器检查 ack 是否为 k+1，若正确则连接建立成功，客户端和服务器都进入ESTABLISHED，完成三次握手，可以开始传输数据了。
            1. 为什么需要 3次握手？ 为了防止已失效的连接请求报文段突然又传送到了服务端。
    1. 四次挥手
        1. 客户端发送 FIN 包，告知服务器要关闭连接。
        1. 服务器收到后发送一个 ACK 包，给客户端，告诉你我收到了。
        1. 服务器发送一个FIN包，告知客户端要关闭连接。
        1. 客户端收到后发送一个 ACK 包，给服务器，告诉你我收到了。双方关闭连接
            1 为什么需要 4次挥手？ 关闭连接时，当收到对方的FIN报文时，仅仅表示对方不再发送数据了但是还能接收数据，己方也未必全部数据都发送给对方了，所以己方可以立即close，也可以发送一些数据给对方后，再发送FIN报文给对方来表示同意现在关闭连接，因此，己方ACK和FIN一般都会分开发送。
4. 服务器的永久重定向响应
    1. 服务器给浏览器响应一个301永久重定向响应，此时浏览器会访问 www.com , 而非 .com 。
    1. 为什么不直接发送用户想要的网页内容呢？
        1. 搜索引擎认为 www.com .com 为两个网址，会降低搜索排名，但它能识别301，则会只记录 www.com ,从而提升排名。
        1. 若一个页面有好几个名字时，它可能会在缓存里出现好几次。
    1. 301 和 302
        1. 都表示重定向，这个地址都可以从响应的 location 中获取。
        1. 301 表示访问的地址已经被永久移除了。302 表示我希望你跳，但是你仍然可以访问该地址。 SEO302好于301。
    1. 重定向原因：
        1. 网站调整。
        1. 网页被转移。
        1. 网页扩展名变更。
    1. 为什么要做重定向
        1. 有以上原因，若不做重定向，则用户收藏、搜索引擎中旧地址，就只能让访问客户得到一个404页面。
        1. 注册了多个域名，也需要通过重定向让访问这些域名的用户自动跳转到主站点。
5. 浏览器跟踪重定向地址————从响应的 location 中获取了url，则发起另一个http请求。
6. 服务器处理请求
    1. 后端从固定的端口接收到TCP报文开始，对TCP连接进行处理，对HTTP协议进行解析，并按照报文格式进一步封装成 HTTP request 对象，供上层使用。
    1. 大一点的网站会将请求到反向代理服务器中，此时不是直接通过HTTP协议访问某些网站应用服务器，而是先请求到 nginx ， nginx 再请求其他分布式应用服务器，顺便做个负载均衡。
        1. 分布式好处是，其中一个服务器挂了也不会影响用户使用。
7. 服务器返回一个 HTTP 响应
    1. HTTP响应也由3个部分构成，分别是：
        1. 状态行
            1. 由协议版本、数字形式的状态代码、及相应的状态描述，各元素之间以空格分隔。
        1. 响应头
            1. 由关键字/值对组成，每行一对，关键字和值用英文冒号":"分隔
        1. 响应正文
            1. 包含着我们需要的一些具体信息，比如cookie，html,image，后端返回的请求数据等等。
8. 浏览器显示 HTML
    1. webkit的渲染过程
        1. 解析html以构建dom树 -> 构建render树 -> 布局render树 -> 绘制render树
            1. 浏览器在解析html文件时，会”自上而下“加载，并在加载过程中进行解析渲染。
                1. 在解析过程中，如果遇到请求外部资源时，如图片、外链的CSS、iconfont等，请求过程是异步的，并不会影响html文档进行加载。
            1. 解析过程中，浏览器首先会解析HTML文件构建DOM树，然后解析CSS文件构建渲染树，等到渲染树构建完成后，浏览器开始布局渲染树并将其绘制到屏幕上。
                1. reflow(回流) 修改、计算其位置和大小等时。
                1. repain(重绘) 修改背景、颜色,字体等时。
        1. 当文档加载过程中遇到js文件，html文档会挂起渲染（加载解析渲染同步）的线程，不仅要等待文档中js文件加载完毕，还要等待解析执行完毕，才可以恢复html文档的渲染线程。
            1. 为JS有可能会修改DOM，最为经典的document.write，这意味着，在JS执行完成前，后续所有资源的下载可能是没有必要的，这是js阻塞后续资源下载的根本原因。
            1. 所以我们平时的代码中，js是放在html文档末尾的。
            1. js的解析
                1. 由浏览器中的JS解析引擎完成的，比如谷歌的是V8。
                1. JS是单线程运行，所有的任务都需要排队，同步任务(synchronous)。
                1. 但是又存在某些任务比较耗时，所以需要回调机制，可以先执行排在后面的任务，异步任务(asynchronous)。
                1. JS的执行机制就可以看做是一个主线程加上一个任务队列(task queue)。
                    1. 所有的同步任务在主线程上执行，形成一个执行栈。
                    1. 异步任务有了运行结果就会在任务队列中放置一个事件。
                    1. 脚本运行时先依次运行执行栈，然后会从任务队列里提取事件，运行任务队列中的任务，这个过程是不断重复的，所以又叫做事件循环(Event loop)。
9. 浏览器发送请求获取嵌入在 HTML 中的资源（如图片、音频、视频、CSS、JS等等）
    1. 类似前8步，但不像动态页面，静态文件会允许浏览器对其进行缓存。有的文件可能会不需要与服务器通讯，而从缓存中直接读取，或者可以放到CDN中。


## HTTP状态码

1. 1** 信息，服务器收到请求，需要请求者继续执行操作
    1. 100 continue 继续。客户端应继续其请求。
    1. 101 switching Protocols 切换协议。服务器根据客户端的请求切换协议。只能切换到更高级的协议，例如，切换到HTTP的新版本协议
1. 2** 成功，操作被成功接收并处理
    1. 200 OK 请求成功。一般用于GET与POST请求
    1. 201 Create 已创建。成功请求并创建了新的资源
    1. 202 Accepted 已接收。已经接收请求，但未处理完成
    1. 203 Non-auth 非授权信息。请求成功，单返回的meta信息不在原始的服务器，而是一个副本。
    1. 204 No content 无内容。服务器成功处理，但未返回内容。在未更新网页的情况下，可确保浏览器继续显示当前文档。
    1. 206 Partial content 成功执行了一个范围请求
1. 3** 重定向桩体码，表示服务器要求客户端重定向
    1. 301 Moved Permanently 永久重定向，响应报文的location首部应该有该资源的新URL
    1. 302 Found 临时性重定向，响应报文的Location首部给出的URL用来临时定位资源
    1. 303 See Other 请求的资源存在着另一个URI，客户端应使用GET方法定向获取请求的资源
    1. 304 Not Modified 服务器内容没有更新，可以直接读取浏览器缓存
    1. 307 Temporary Redirect 临时重定向。与302 Found含义一样。302禁止POST变换为GET，但实际使用时并不一定，307则更多浏览器可能会遵循这一标准，但也依赖于浏览器具体实现
1. 4**：客户端错误状态码，表示客户端的请求有非法内容。
    1. 400 Bad Request 表示客户端请求有语法错误，不能被服务器所理解
    1. 401 Unauthonzed 表示请求未经授权，该状态代码必须与 WWW-Authenticate 报头域一起使用
    1. 403 Forbidden 表示服务器收到请求，但是拒绝提供服务，通常会在响应正文中给出不提供服务的原因
    1. 404 Not Found 请求的资源不存在，例如，输入了错误的URL
    1. 405 客户端请求中的方法被禁止
    1. 406 服务器无法根据客户端请求的内容特性完成请求
    1. 407 请求要求代理人的身份证类型，与401类似，但请求者应当使用代理进行授权
    1. 408 服务器等待客户端发送的请求时间过长，超时
    1. 409 服务完成客户端的put的请求时可能返回此代码，服务器处理请求时发生了冲突
    1. 410 客户端请求的资源已经不存在，不同于404，如果资源以前有限制被永久删除了可使用410代码，网站设计人员可通过301代码指定资源的新位置。
    1. 411 服务器无法处理客户端发送的不带 content-length 的请求信息
    1. 412 客户端请求信息的先决条件错误
    1. 413 由于请求的实体过大，服务器无法处理，因此拒绝请求。为防止客户端连续请求，服务器可能会关闭连接。如果只是服务器暂时无法处理，则会包含一个Retry-After的响应信息
    1. 414 请求的URL过长，服务器无法处理
    1. 415 服务器无法处理请求附带的媒体格式
    1. 416 客户端请求的范围无效
    1. 417 服务器无法满足expect的请求头信息
1. 5**：服务器错误状态码，表示服务器未能正常处理客户端的请求而出现意外错误。
    1. 500 Internel Server Error 表示服务器发生不可预期的错误，导致无法完成客户端的请求。
    1. 501 服务器不支持请求的功能，无法完成请求
    1. 502 充当网管或代理的服务器，从远端服务器接收到了一个无效的请求
    1. 503 Service Unavailable 表示服务器当前不能够处理客户端的请求，在一段时间之后，服务器可能会恢复正常。
    1. 504 充当网管或代理的服务器，未及时从远端服务器获取请求。
    1. 505 服务器不支持请求的Http协议的版本，无法完成处理。
    

## 通用首部字段（请求响应都可能会有的）

1. Cache-Control 控制缓存的行为
    1. 缓存请求指令
        1. no-cache 强制再次验证
        1. no-store 不缓存请求或响应的任何内容
        1. max-age=[秒] 响应的最大age值
        1. max-stale(=[秒])接受已过期的响应
        1. min-fresh=[秒] 期望在指定时间内的响应仍然有效
        1. no-transform 代理不可更改媒体类型
        1. only-if-cached 从换从获取资源
        1. cahce-extension 新指令标记（token）
    1. 缓存响应指令
        1. public 可向任意方提供响应的缓存。
        1. private 仅向特定用户返回响应
        1. no-cache 缓存前必须要先确认其有效性
        1. no-store 不缓存请求或响应的任何内容
        1. no-transform 代理不可更改媒体类型
        1. must-revalidate 可缓存但必须再向源服务器进行确认
        1. proxy-revalidate 要求中间缓存服务器对缓存的响应有效性再进行确认
        1. max-age=[秒] 响应的最大age值
        1. s-maxage=[秒] 公共缓存服务器响应的最大age值
        1. cache-extension 新指令标记
1. Connection 连接的管理
    1. 控制不再转发给代理的首部字段名
    1. 管理持久连接
        1. http 1.1 默认是tcp长连接，客户端会连续发送请求，当服务器明确想断开连接时，指定其为close
        1. 1.0 默认都是tcp短连接，想要转变为tcp长连接则需要指定为Keep-Alive
            ```
            Keep-Alive:timeout=10,max=500
            Connection:Keep-Alive
            ```
1. Date 创建报文的日期时间
1. Pragma 报文指令
1. Traiker 报文末端的首部一览
1. Transfer-Encoding 指定报文主体的传输编码方式
1. Upgrade 升级为其他协议
1. Via 代理服务器相关信息
1. warning 错误通知

## 请求头

1. Accept： 客户端可识别的内容类型列表。
1. Accept-Charset: 请求内容的编码集，utf-8
1. Accept-Encoding: 请求内容编码方式。
1. Accept-Language： 请求内容语言。
1. Authorization web认证信息
1. Expect 期待服务器的特定行为
1. From 用户的电子邮箱地址
1. Host 请求资源所在的服务器
1. If-Match 比较实体标记
1. If-Modified-Since 比较资源的更新时间
1. If-None-Match 比较实体标记 （与 If-Match 相反）
1. If-Range 资源未更新时发送实体Byte的范围请求
1. if-Unmodified-Since 比较资源的更新时间（与If-Modified-Since相反）
1. Max-Forwards 最大传输逐跳数
1. Proxy-Authorization 代理服务器要求客户端的认证信息
1. Range 实体的字节范围请求
1. Referer 对请求中URL的原始获取方
1. TE 传输编码的优先级
1. User-Agent： 产生请求的浏览器类型。

## 响应头

1. 由关键字/值对组成，每行一对，关键字和值用英文冒号":"分隔，典型的响应头有：
1. Accept-Ranges 是否接受字节范围请求
1. Age 推算资源创建经过时间
1. Etag 资源的匹配信息
1. Proxy-Authenticate 代理服务器对客户端的认证信息
1. Retry-After 对再次发起请求的时机要求
1. Vary 代理服务器缓存的管理信息
1. Allow 服务器支持哪些请求方法 （如GET、POST等）
1. Content-Encoding 文档的编码方法。表示只有在解码之后才可以得到Content-Tyoe头指定的的内容类型。
1. Content-Language 文档的自然语言。
1. Content-length 表示内容长度，只有当浏览器使用持久HTTP连接时才需要这个数据。
1. content-Location 替代对应资源的URL
1. content-MD5 内容的摘要
1. content-Range 位置范围
1. Content-Type 表示后面的文档属于什么MIME类型。默认text/plain。
1. Date 当前GMT时间。
1. Expires 应该在什么时候认为文档已经过期，从而不再缓存它？
1. Last-Modified 文档的最后改动时间。
    1. 客户可以动过if-Modified-Since请求提供一个日期，该请求将被视为一个条件GET，只有改动时间迟于指定时间的文档才会返回，否则返回一个304状态。
1. Location 表示客户应当到哪里去提取文档。
1. Refresh 表示浏览器硬钢在多少时间之后刷新文档。
1. Server 服务器名字。
1. Set-cookie 设置和页面管理的Cookie。
1. WWW-Authenticate 客户应该在 Authorization 头中提供什么类型的授权信息？在包含401的应答中这个头是必需的。

## tcp长连接和tcp短连接的区别

1. tcp长连接多个http请求可以复用同一个Tcp连接，节省建立、断开tcp连接的消耗
    1. 例如你打开一个网页，通常默认就建立了一个tcp长连接，此时还要请求css，js，图片等，就可以复用这个tcp长连接。 
1. 短连接每请求一次就建立断开一次，耗费资源多
1. 长连接也不是永远连接，在Keep-Alive中有定义超时时间，过了时间自然会被断掉

## 长轮询和短轮询

1. 短轮询
    1. 不停请求服务器，服务器立即返回数据
1. 长轮询 
    1. 客户端请求一次服务器，服务器发现数据没有变化将请求挂起，等数据变化了，再返回数据，否则一直等到超时时间为止。
1. 长短都会消耗资源，但长会少一些。

# tcp长短连接和长短轮询的区别

1. 决定的方式
    1. TCP连接是否为长连接，是通过设置HTTP的Connection Header来决定的，而且是需要客户端和服务器都设置才有效。
    1. 轮询方式是否为长轮询，是根据服务端的处理方式来决定的，与客户端没有关系。
1. 实现的方式
    1. 连接的长短是通过协议来规定和实现的。
    1. 而轮询的长短，是服务器通过编程的方式手动挂起请求来实现的。


## link 和 js 是否会阻塞 DOM 解析和渲染吗？

1. css
    1. CSS 不会阻塞 DOM 的解析。当页面解析到 link rel="stylesheet" 或者 style 标签时，会发起请求下载 css，但是解析不会被阻塞，依旧继续向下解析。
        1. 浏览器是解析DOM生成DOM Tree，结合CSS生成的CSS Tree，最终组成render tree，再渲染页面。
        1. CSS完全无法影响DOM Tree，因而无需阻塞DOM解析。
    1. CSS 阻塞页面渲染，若不阻塞渲染，用户体验差；渲染有代价，所以浏览器会尽量减少渲染的次数。
1.  js
    1. js 阻塞 DOM 的解析，延后了渲染。
    1. 若 js 文件太大，可以加上 defer 或者 async 属性，此时脚本下载的过程中是不会阻塞 DOM 解析的。
    1. 浏览器遇到 `<script>` 标签时，会触发页面渲染
        1. 浏览器不知道脚本的内容，因而碰到脚本时，只好先渲染页面，确保脚本能获取到最新的DOM元素信息，尽管脚本可能不需要这些信息。
        1. 若 link 在 js 上部的话，js 会等待 css 下载完毕之后再取消阻塞解析。所以 js 放在 link 下面会更好。
1. 结论
    1. CSS 不会阻塞 DOM 的解析，但会阻塞 DOM 渲染。
    1. JS 阻塞 DOM 解析，但浏览器会"偷看"DOM，预先下载相关资源。
    1. 浏览器遇到 `<script>`且没有defer或async属性的 标签时，会触发页面渲染，因而如果前面CSS资源尚未加载完毕时，浏览器会等待它加载完毕在执行脚本。
