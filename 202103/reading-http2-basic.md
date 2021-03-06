# 阅读：HTTP/2基础教程

# Reading: Learning HTTP/2

## 2021 年 3 月 27 日

## Mar 27 2021

---

### Web技术的四大基石

Web（全称 World Wide Web）的四大技术基石是 URI、HTML、HTTP 和 MIME，正是这四大基石支撑了宏伟的 Web 神殿。在这四大基石之中，HTTP 的重要性最为突出。今天有很多移动 Web 应用并没有使用 URI、HTML、MIME，只用了 HTTP，仍然可以称为“Web 应用”。从这个角度看，HTTP 几乎是 Web 的代名词。

### REST与HTTP/1.1

建议对 REST 非常感兴趣的读者去读一下 Fielding 的博士论文中文版《架构风格与基于网络应用软件的架构设计》。REST 理论有力地指导了 HTTP/1.1 的设计，也确保了后续 HTTP 协议沿着正确的方向发展，包括从旧版本到新版本的平滑升级。

### HTTP/0.9

HTTP/0.9 是个相当简单的协议。它只有一个方法（GET），没有首部，其设计目标也无非是获取 HTML（也就是说没有图片，只有文本）。

### HTTP/1.0

1.0 版本为原有的轻量协议新增了大量内容。0.9 版本的规范大概有 1 页，1.0 版本则有 60 页。所以，你可以说它从玩具成长为了工具。它包含了一些我们如今非常熟悉的概念：

- 首部
- 响应码
- 重定向
- 错误
- 条件请求
- 内容编码（压缩）
- 更多的请求方法

### HTTP/1.1

1.0 版本刚刚制定，1.1 版本就接踵而来。截至目前，1.1 版本的协议已经使用了 20 多年了。它修复了之前提到的 1.0 版本的大量问题。因为强制要求客户端提供 Host 首部，所以虚拟主机托管成为可能，也就是在一个 IP 上提供多个 Web 服务。当使用新的连接指令时， Web 服务器也不需要在每个响应之后关闭连接。这对于提升性能和效率而言意义重大，因为浏览器再也不用为每个请求重新发起 TCP 连接了。
添加的变更如下：

- 缓存相关首部的扩展
- OPTIONS 方法
- Upgrade 首部
- Range（范围）请求
- 压缩和传输编码（transfer-encoding）
- 管道化（pipelining）

### 网页请求图解

![客户端](https://cescdf.com/image/kehuduan.jpeg)

![服务端](https://cescdf.com/image/fuwuduan.jpeg)



### 关键性能指标

- 延迟：延迟是指 IP 数据包从一个网络端点到另一个网络端点所花费的时间。与之相关的是往返时延（RTT），它是延迟的时间的两倍。延迟是制约 Web 性能的主要瓶颈，尤其对于HTTP 这样的协议，因为其中包含大量往返于服务器的请求
- 带宽：只要带宽没有饱和，两个网络端点之间的连接会一次处理尽可能多的数据量。依据 Web 页面引用资源的大小和网络连接的传输能力，带宽可能会成为性能的瓶颈
- DNS查询：在客户端能够获取 Web 页面前，它需要通过域名系统（DNS）把主机名称转换成 IP 地址，DNS 相当于互联网上的电话号码簿。获取的 HTML 页面中所引用的各个不同域名也需要转换；幸运的是，一个域名只需转换一次。
- 建立连接时间：在客户端和服务器之间建立连接需要往返数据应答，称为“三次握手”。握手时间一般与客户端和服务器之间的延迟有关。握手过程包括客户端向服务器发起一个 SYN 包，接着服务器返回对应 SYN 的 ACK 响应以及新的 SYN 包，然后客户端返回对应的 ACK。
- TLS协商时间：如果客户端发起 HTTPS 连接，它还需要进行传输层安全协议（TLS）协商；TLS 用来取代安全套接层（SSL）。除了服务器和客户端的计算处理耗时之外，TLS 还会造成额外的往返传输。

目前为止，客户端还没有真正发起 HTTP 请求，却已经用掉了 DNS 查询的往返时间，以及 TCP 和 TLS 的耗时。下面的指标严重依赖于页面内容本身或服务器性能，而不是网络。

- 首字节时间（TTFB）：TTFB 是指客户端从开始定位到 Web 页面，至接收到主体页面响应的第一字节所耗费的时间。它包含了之前提到的各种耗时，还要加上服务器处理时间。对于主体页面上的资源，TTFB 测量的是从浏览器发起请求至收到其第一字节之间的耗时。
- 内容下载时间：等同于被请求资源的最后字节到达时间（TTLB）。
- 开始渲染时间：客户端的屏幕上什么时候开始显示内容？这个指标测量的是用户看到空白页面的时长。
- 文档加载完成时间（又叫页面加载时间）：这是客户端浏览器认为页面加载完毕的时间。

如果我们关注 Web 性能，尤其是要制定新协议来提升效率，就必须把（上面提到的）那些指标牢记于心。后文讨论 HTTP/1.1 面临的问题以及我们想要寻求改变的原因时，还会提到它们。

此外，更多性能瓶颈的出现是因为现代Web的新特点：更多的字节、资源、复杂度、域名以及TCP socket。（为了应对某些方面的增加，客户端会对同一个域名开启多个 socket。这增加了与域名对应的服务器协商建立连接的开销，也加重了设备负担，还有可能导致网络连接过载，引发出错重传和缓存过满，并降低有效带宽。）



### HTTP/1.1的问题

- 队头阻塞：管道化的失效。
- 低效的TCP利用：TCP的避免拥塞机制被设计成即使在最差的网络状况下仍能起作用，并且如果有需求冲突也保证相对公平。这是它取得成功的原因之一。它的成功并不是因为传输数据最快，而是因为它是最可靠的协议之一，涉及的核心概念就是拥塞窗口（congestion window）。拥塞窗口是指，在接收方确认数据包之前，发送方可以发出的 TCP 包的数量。例如，如果拥塞窗口指定为 1，那么发送方发出 1 个数据包之后，只有接收方确认了那个包，才能发送下一个。
- TCP的慢启动：从慢启动到拥塞避免的耗时。因为 h1 并不支持多路复用，所以浏览器一般会针对指定域名开启 6 个并发连接。这意味着拥塞窗口波动也会并行发生 6 次。TCP 协议保证那些连接都能正常工作，但是不能保证它们的性能是最优的。
- 臃肿的首部：460字节。
- 受限的优先级、第三方资源延迟甚至阻塞页面渲染等等问题。

### 一些最佳实践

- DNS查询优化：预获取（dns-prefetch）。
- 优化TCP连接：预连接（preconnect）。
- 重定向：避免或CDN在云端重定向。
- 客户端缓存：没有什么比直接从本地获取缓存更快的方案了。（利用cachecontrol、max-age、expires等首部）。
- 网络边缘的缓存：非隐私数据。
- 条件缓存：Last-Modified-Since。

### 一些针对HTTP/1.1的优化

- 资源合并：多个js或css文件合并到一个，在http/2下非必要，因为请求的传输字节数和时间成本更低。
- 极简化：去除资源文件里无用的代码，在http/2下要保留。
- 域名拆分：资源分布到不同的域名上面，让浏览器利用更多的socket连接。这与http/2的设计思想刚好相反，http/2的设计意图是充分利用单个socket连接，而拆分域名会违背这种意图。
- 禁用cookie的域名：为图片之类的资源建立单独的域名，这些域名不用cookie，以尽可能减少请求尺寸。同上，这在http/2中应该避免，而且提供了首部压缩，cookie的开销会显著降低。
- 精灵图：多张图片拼合为一个，通过css控制展示。这和极简化蕾丝，只不过css实现该效果的代价可能高昂，不推荐在http/2中使用。

### HTTP/2分层

HTTP/2分为两个层：分帧层和数据层（http层）。

- 二进制协议：分帧层传输基于帧的二进制协议。
- 首部压缩：深度压缩。
- 多路复用：请求和响应交织交响。
- 加密传输：加密。

### 判断是否支持HTTP/2的黑魔法：连接前奏

连接前奏包含了一个关于美国国家安全局 PRISM（棱镜）监控计划的笑话，以及HTTP/2的名称。第一条是为了表示协议制定人员很有幽默感，第二条则表明协议已开始就舍弃了小数点，即以后不会出现2.1、2.2这样的更新。

### 帧

HTTP/2是基于帧（Frame）的协议，相比之下h1不是基于帧而是基于文本的。解析文本数据不需要什么算法，但往往速度很慢，你需要不断读入字节直到遇到分隔符为止。

帧的前9个字节意义是固定的：

- Length（3字节）：帧负载的长度，214字节是默认的最大帧大小。
- Type（1字节）：当前帧类型。
- Flags（1字节）：具体帧的标志。
- R（1位）：保留位。
- StreamIdentifier（31位）：每个流的唯一ID。
- FramePayload（可变）：真实的帧内容，长度在Length中设定。

因为规范严格明确，所以解析逻辑也很直白。这样一来，实现和维护都会简单很多。相比依靠分隔符的 h1，h2 还有另一大优势：如果使用 h1 的话，你需要发送完上一个请求或者响应，才能发送下一个；由于 h2 是分帧的，请求和响应可以交错甚至多路复用。多路复用有助于解决类似队头阻塞的问题。

h2有10种帧类型：传输流的核心内容、包含HTTP首部和可选的优先级参数、指定或者更改流的优先级和依赖、允许一端停止流、协商连接级参数、提示客户端服务器要推送些东西、测试连接可用性和往返时延、告诉另一端当前端已结束、协商一端将要接收多少字节、用以扩展HEADER数据块。

### 流

HTTP/2 规范对流（stream）的定义是：“HTTP/2 连接上独立的、双向的帧序列交换。你可以将流看作在连接上的一系列帧，它们构成了单独的 HTTP 请求和响应。

如果客户端想要发出请求，它会开启一个新的流。然后，服务器将在这个流上回复。这与 h1 的请求 / 响应流程类似，重要的区别在于，因为有分帧，所以多个请求和响应可以交错，而不会互相阻塞。流 ID（帧首部的第 6~9 字节）用来标识帧所属的流。

请注意，POST 和 GET 的主要差别之一就是 POST 请求通常包含客户端发出的大量数据。所以GET的客户端的流中仅包含HEADER，而POST包含HEADER+一系列的DATA。

基于流的HTTP/2有如下特点：

- 一切皆是header：h1 把消息分成两部分，请求 / 状态行以及首部。h2 取消了这种区分，并把这些行变成了魔法伪首部。
- 没有分块编码：在基于帧的世界里，谁还需要分块？只有在无法预先知道数据长度的情况下向对方发送数据时，才会用到分块。
- 不再有101响应。

### 基于流的流量控制

h2 的新特性之一是基于流的流量控制。不同于 h1 的世界，只要客户端可以处理，服务端就会尽可能快地发送数据，h2 提供了客户端调整传输速度的能力。（并且，由于在h2 中，一切几乎都是对称的，服务端也可以调整传输的速度。）WINDOW_UPDATE 帧用来指示流量控制信息。

客户端有很多理由使用流量控制。一个很现实的原因可能是，确保某个流不会阻塞其他流。

流的最后一个重要特性是依赖关系（资源树），也就是优先级。**依赖关系**为客户端提供了一种能力，通过指明某些对象对另一些对象有依赖，告知服务器这些对象应该优先传输。**权重**让客户端告诉服务器如何确定具有共同依赖关系的对象的优先级。

### 服务端推送

提升单个对象性能的最佳方式，就是在它被用到之前就放到浏览器的缓存里面。这正是 HTTP/2 的服务端推送的目的。推送使服务器能够主动将对象发给客户端，这可能是因为它知道客户端不久将用到该对象。

### 放弃GZIP而实用HPACK做首部压缩

为什么不直接用 GZIP 做首部压缩，而要使用HPACK ？那样肯定能节省大量工作。不幸的是，CRIME 攻击告诉我们，GZIP 也有泄漏加密信息的风险。CRIME 的原理是这样的，攻击者在请求中添加数据，观察压缩加密后的数据量是否会小于预期。如果变小了，攻击者就知道注入的文本和请求中的其他内容（比如私有的会话 cookie）有重复。在很短的时间内，经过加密的数据内容就可以全部搞清楚。因此，大家放弃了已有的压缩方案，研发出 HPACK。

简而言之，HPACK是不断利用重复的字节，只告诉另一端改变的部分。

### 多路复用

多路复用代替原来的序列和阻塞机制。所有就是请求的都是通过一个 TCP 连接并发完成。因为在多路复用之前所有的传输是基于基础文本的，在多路复用中是基于二进制数据帧的传输、消息、流，所以可以做到乱序的传输。多路复用对同一域名下所有请求都是基于流，所以不存在同域并行的阻塞。多次请求如下图：

![juejin](https://user-gold-cdn.xitu.io/2019/9/5/16cff873bf2ec175?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

[关于多路复用的参考链接](https://juejin.cn/post/6844903935648497678)

### 推送缓存（Push Cash）

推送缓存是HTTP2新增的内容，当Service Worker、Memory Cache、Disk Cache或**prefetch cache(预取缓存)**都没有被命中时，推送缓存才会被使用。它只在会话（Session）中存在，一旦会话结束就被释放，并且缓存时间也很短暂，在Chrome浏览器中只有5分钟左右。

关于缓存，可以看看[掘金这篇文章](https://juejin.cn/post/6947936223126093861)。

有看到多的知识点会做更新。