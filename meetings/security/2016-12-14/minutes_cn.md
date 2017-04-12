# Swift Server APIs: Security stream meeting 1

### Initial Agenda:
* Current discussion status roundup
* Review of BlueSSLService, a layer on top of OpenSSL and macOS Secure Transport libraries:  BlueSSLService Review
* Next Steps

### Attendees:
* Chris Bailey
* David Sperling
* Gelareh Taban
* Alfredo Delli Bovi
* Bill Abt
* Alex Blewitt

# Minutes:

_Security APIs 的范围:_  
* 所有期望能够实现的模块就是整个API的集合 , 包含 secure transport (SSL/TLS), crypto, 和 keychain/certificate management, 等等
* 目标也是提供一套适用于客户端和服务端 APIs, 尽管所有的 API 并不适用于每个使用场景
* 使用哪种 crypto 取决于具体的场景 (以及需求是否超出了 macO以及SOpenSSL/LibreSSL库所提供的).

_回顾一下 BlueSSLService 所采用的方式:_  
* BlueSSLService 包含了 "secure transport"这一层的API. 还有一个 BlueCryptor library 来提供 crypto 服务.
* 它通过让现有的 Socket 库遵从某个 Protocol 来提供一种服务. 当前对该 Protocol的实现是 BlueSocket, 如果需要其它 socket 的实现，通过简单的修改即可.
  * socket 库需要实现 6 个Protocol方法.
  * SSL service 的一个方法让你重新实现 connection verification process，从而可以使用一个 specialized connection verification. 
* 最初，在所有平台（包括MacOS）使用 OpenSSL 来实现BlueSSLService , 但是在 MacOS上，最终转变为使用 SecureTransport.
  * OpenSSL 和 SecureTransport之间的 APIs 差异非常大, 需要花不少时间和精力来对付 SecureTransport 的文档，哈哈.
  * OpenSSL 提供了更复杂的 configuration. macOS / SecureTransport仅限于使用pkcs＃12, 虽然有内部的API处理来处理其他的 (如PEM)，但是并没有暴漏出来.
  * 我们是否需要支持 PEM, 等等? 可能 - 这在 Linux/server 中的使用更为普遍.
    * 这也许是为了可用性，简洁性, 只暴漏用户需要的. 增加对服务端使用的情况的考虑的话, 这也许需要改变.
  * 对于 cipher suites, SecureTransport 使用 hex IANA 值来选择 cipher suites，而 OpenSSL 让你使用 text abbreviations (相对于 IANA values), AND/OR 逻辑判断也是一样.
* 最终的结果是，相比在macOS平台使用SecureTransport， 在Linux平台，通过使用OpenSSL能够实现更多的功能
* 还有一点， BlueSSLService 并不能支持 iOS平台，由于SecureTransport 的一些APIs在iOS端没有暴漏出来.
  * 但是，如果用户的需求能够通过其它方式来满足的话，也没有啥问题 (比如，用户的需求可以通过 URLSession等来满足 )

_使用 OpenSSL vs LibreSSL:_  
* 使用OpenSSL，因为它提供了FIPS认证，而LibreSSL则没有
  * 有很多对 OpenSSL 的forks, 但这些并不都是支持 FIPS 认证的. 还有，好些fork都只是原库的一个子集.
  * 直接使用操作系统的库可以很容易实现 FIPS认证，因为macOS和iOS中已经有这样的实现案例了，并且有一个适用于Linux的OpenSSL的certified version .

_回顾一下 BlueCryptor 的 Crypto:_


* BlueCryptor尝试为Crypto做同样的事情，在iOS和macOS上使用CommonCrypto，在Linux上使用OpenSSL.
* 在CommonCrypto中，缺少的功能是RSA. 就像BlueSSLService中其它格式的文件一样, RSA在内部的库中有接口，但是没有公开出来. 在OpenSSL中，RSA是可用的，并且API与CommonCrypto API相似，可以在BlueCryptor中很容易的公开出来.

_Keychain Services:_
* keychain services目前还没有任何尝试的案例. Keychain services 在macOS/iOS的应用非常棒, 但是在 Linux 上没有相应的实现
* Linux只是依赖文件系统中存在的证书等
* 很多Linux服务器，特别是基于Java的服务器, 使用一个CMS（证书管理系统）来管理证书和密钥.  这与Apple平台上的Keychain API提供的服务类似.
* 需要进一步的工作来了解我们是否需要实现, 或期望用户自己处理证书管理（在各个操作系统上）.

_使用BlueSSLService和BlueCryptor作为原型实现:_  
* 我们应该使用BlueSSLService和BlueCryptor作为原型实现，并将其用作 API 设计的参考?
* 这似乎是一个合理的方式

### Next Steps:
* 与Luke Hiesterman讨论SecureTransport和CommonCrypto中的实现但没有暴露出来的功能
* 对普通证书/钥匙串管理方面（如果有的话）进行更多的调查
* 开始为安全库构建一个 outline proposal，用作Swift Evolution “pitch”
