# Security sub-team kick-off

### Initial Agenda:
* Introductions
* Goals
* Development process overview
* Next Steps

### Attendees:
* Chris Bailey - IBM, Kitura and Swift.org stuff
* Cătălin Stan - Interested in Swift/server as of Swift 3. Background is Linux systems programming and Obj-C. Starting withing on a fast-cgi framework, then moved on to also include http
* Johannes Weiss - Apple iCloud/Swift Server. 
* Gregor Milos - Apple iCloud/Swift Server. Building low level tools/frameworks
* Alfredo Delli Bovi - contributions to Swift core. Background in Node.js/PHP
* Tyler Stromberg - mostly working on client apps - BE stuff with Ruby, Elixir, Go
* Luke Hiesterman - Apple in the security engineering team on security APIs
* Piers Mainwaring - web & mobile dev in Swift. JavaScript/Ruby/Python on the BE. Worked on some home-rolled Swift server frameworks
* Gelareh Taban - IBM, working on security for the Swift@IBM group

### Minutes:
_Q) 我们的预期时间表是什么:_  
* 我们真的需要在Swift 4中发布时提供 1.0版本. 在此之前，Previews 应该/将可用.
* 我们不受Swift 4.0限制，因为我们可以作以 packages 的形式发布 .

_Q) 安全在服务器端到底有多重要? 不能在前面放一个反向代理（reverse proxy）吗?_  
* 如果你把代理放在同一个容器（container）中，那么HTTPS就不应该是一个显示拦截（show stopper）…  
* 对于出站（outbound）来说更重要, 因为其需要以https的形式请求其他服务（service）
* 然而，代码的可移植性是必须的. 我们想要 API layer 能兼容各种平台 - 包括 iOS clients 和 Linux/macOS servers. 不必在所有的平台下都保持相同的实现, 但是API应该是（如果可能的话）.
* 虽然有挑战 - 我们需要以不显著增加开销的方式这样做，, 我们不希望你在iOS客户端上使用Linux/服务器的包.

_Q)APIs 应该是怎样层级的? 应该是适用于所有使用场景还是仅仅是针对服务器来实现_  
例如, 如果我想发起一个 HTTP request, 我不想使用 socket, 就像使用一个便捷的 API
* 我们真的希望从最底层开始, 构建一个适用于多种使用场景的 API,可以在各种场景案例下灵活扩展.  
* 一旦这么做了, 我们可以基于底层 API 为更多的使用场景构建一层或者多层服务.

_Q) 使用 swift vs 复用已经存在的技术?_  
* 想想一下使用现有的组件, 我们不必维护复杂的代码, 处理CVEs（CVE - Common Vulnerabilities and Exposures）, 等等. 没必要去重复造轮子.

_Q) 如果我们复用已有的技术, 应该从哪层开始设计 API 呢? 应该从 common crypto 上构建一层?_  
* 起始点应该是使用swift构建一层 “translation layer”, 然后再看是否需要在此之上再构建一个抽象层.  
* 由下往上构建比由上往下构建要简单.

_Q) 我们应当使用哪些技术栈的东西?_  
* 如果我们想使 iOS clients 和 servers 保持一致可用, 那我们至少需要使用 CommonCrypto/SecureTransport/Keychain , 这在 macOS 中也是一样的.  
* 在其他的平台, OpenSSL 或者 LibreSSL 可能是可行的.  
关键是使 Swift API 保持一致, 从而使用这些 API的 applications 和 packages 可以移植、兼容.  
* 不过，对于 IO 处理或许会有些问题,以及 certificate 处理 和 配置.  
* 在 BlueSSLService 中已经尝试这样做了, 因此其已经走过的路对我们有很大的参考价值，防止我们实践时碰到类似的问题和障碍.

### Next Steps/Actions:
* Everyone to review minutes
* Chris to post minutes to work group Git Repo and send a link to the mailing list
* Next meeting in 2ish weeks - Gelareh to walk through BlueSSLService and issues/pitfalls found during its development.
