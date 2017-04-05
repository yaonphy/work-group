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

_Q) What level should the APIs be at? Should they be multi-purpose or implement specific (server) use cases_  
ie, if I want to make a HTTP request, I don’t want to use a socket, I want to use a convenience API
* We really want so start at the lowest level, building a multi-purpose API, to provide flexibility of use case.  
* Once we have that, we can build up one or more layers of more use case based APIs.

_Q) Should be build in Swift vs reuse existing tech?_  
* Ideally want to reuse existing components, so we don’t have to maintain complex code, deal with CVEs, etc. There’s no desire to re-invent the wheel.

_Q) If we’re reusing tech, how low a level do we start the API at? Should it be just a layer on, say, common crypto?_  
* The starting point is probably just to build a common “translation layer” in Swift, and then see if we want to build abstraction layers on top of that.  
* It’s easier to build up than it is to build down.

_Q) Which tech would we use?_  
* If we want to be consistent on iOS clients and servers, then we need to use CommonCrypto/SecureTransport/Keychain at least there, and it would therefore make sense to on macOS as well.  
* On the other platforms, OpenSSL or LibreSSL probably makes sense.  
The key is making the Swift API consistent, so applications and packages that use the API can be ported.  
* There may be some issues around IO handling, certificate handling and config though.  
* There’s already an attempt to do this kind of thing in BlueSSLService, so a walk through of that might be informative of potential issues/roadblocks/dragons as we build the new approach.

### Next Steps/Actions:
* Everyone to review minutes
* Chris to post minutes to work group Git Repo and send a link to the mailing list
* Next meeting in 2ish weeks - Gelareh to walk through BlueSSLService and issues/pitfalls found during its development.
