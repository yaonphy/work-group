# HTTP sub-team kick-off

### Initial Agenda:
* Introductions
* Goals
* Development process overview
* Next Steps

### Attendees:
* Chris Bailey
* Alex Blewitt
* Ryan Collins
* Ben Cohen
* Sam Liu
* Shmuel Kallner
* Tyler Stromberg
* Robert F. Dickerson
* Logan Wright
* Tanner Nelson
* David Sperling
* Kuan Huang
* Johannes Weiss

### Minutes:

_我们将如何将这些东西连接起来，也就是说，HTTP站点是如何在网络上连接的?_
* 至少从现在起一直到 Swift 5， Concurrency 将会是基于 Dispatch. 从SWIFT 5和开始可能有一个新的Concurrency模型. 那时候我们需要转换到新模型。
* 假设转换到新模型时 Dispatch 出现了同样的问题, 所以我们需要共享问题/解决方案
* HTTP API应该与网络/并发功能分开设计呢，还是组合起来.

_我们应该采用怎样的模型呢? request/response 应该是 value types 还是 reference types?_  
* Swift努力open的道路上遇到了一些模型(model)问题, 以及对 HTTP models的分歧. 请求Class是 Reference 还是 value types.
* 有一些从 openswift / swiftx 开发中吸取的经验教训 - 应该在下次会议上进行review.


_HTTP 1.x与HTTP / 2支持_  
* HTTP2将是不可避免的，所以我们应该将它包含在第一个版本中，这样我们就不会冒着破坏API的风险。

_使用案例:_  
* 仅在服务器上的HTTP？ 在客户端，实现 device -> device 怎么样?
* Foundation 对 libcurl 有依赖，我们想打破其依赖.


_Reference vs. value-types_
* (Logan) Vapor 采用 reference types已经得到了一定的实践验证，而不是采用 value types
* (Shmuel) Kitura 在 NSData -> Data 转换时, 导致性能下降了约20％. 中间件采用 reference types时表现的挺好 
* (Ben C) 我们应该将semantics 与 performance 分开. 
* 一般来说，我们将要传送 request/response. 问题是传递过程中 request/response 被callee更改时，caller 怎么处理.
* Middleware 在request中增加了功能 - callee 传递给了 caller, 在整个传递链条中一直这样进行。


_编码问题（Encoding issues）:_
* Header 使用 ASCII vs. content-body 使用其它 encodings
* 在最底层, 我们将仅仅提供 bytes - the framework 可以处理这样的内容吗?

_使用C vs 纯Swift 来解析:_
* 由Dave Sperling完成的Node.js c-http解析器的Swift实现（port）
   * https://github.com/smithmicro/HTTPParser
   * 尽可能接近C版本 - 一行一行的对照实现.
   * 对 UnsafePointer 的使用已经到位
   * 引用计数（reference counting）仍然花费了大量时间 - 如果有Xcode Instruments的专家来审查代码库，我们将受益匪浅.
   * Vanilla c-http解析器达到了700K请求/秒，而对于Swift的实现（port）则为500K，所以c-http要快40％. 
* 使用纯Swift有可能实现更深层次的内联（inline http://baike.baidu.com/link?url=G3uWgGKuXm54K9E_sD5z1iIyBAnQBvRqjbQcgYYZAjIoODTmvIOiRRkIA9OHm8Qbvyredx5nBXqbm6mHCohlj_)，因此可能会最终提供更好的性能
* Helge Heß 已经做了类似的C vs Swift实现比较:
   * https://github.com/helje5/http-c-vs-swift/
   * c-http解析器几乎完全使用宏来内联所有内容.
   * 尝试使用非官方的“@inline（__ always）”进行复制,提高了一点性能. 但是非常的不稳定,比如，Swift 3 编译器经常 crash等等.
   * 发现类似的结果：c-http解析器比Swift的实现快得多。
* 使用Swift实现可以从swift社区中吸引来大量的贡献者/维护者， (vs. c的实现，就是已经存在的为其贡献的社区).
   * 但是如果我们复用C的实现，那么Swift社区根本就不需要维护它。
* 对HTTP2的解析是否影响我们的选择?


