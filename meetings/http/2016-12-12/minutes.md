# HTTP stream meeting 1


### Initial Agenda:
* Current discussion status roundup
* Value Types vs. Reference Types
* Use of NSHTTPURLRequest/Response
* Experiences of adding HTTP/2 support to Node.js 

### Attendees:
* Chris Bailey
* Alex Blewitt
* Alfredo Delli Bovi 
* Ben Cohen 
* Michael Chiu 

### Minutes:

_C vs Swift （http解析）的实现:_

一些现有的实现:  

| Framework | HTTP 1.x Parser | HTTP/2 Parser | Request Type | Response Type | Notes |
|-----------|-----------------|---------------|--------------|---------------|-------|
| Kitura    | c-http parser   | N/A           | Class        | Class         |       |
| Perfect   | c-http parser(1)| Swift parser  | Class        | Class         |(1) HTTP 1.x parser used to be pure Swift |
| Swifter   | Swift parser    | N/A           | Class        | Enum          |       |
| Vapor     | Swift parser    | N/A           | Class (2)    | Class (2)     |(2) Switched to classes from structs |
| Zewo      | c-http parser   | N/A           | Struct (3)   | Struct (3)    |(3) Class used for stream backed Body fields |
| spartanX  | Swift parser    | N/A           | Struct       | Struct        |       |

Notes:  
(1) Perfect 刚开始使用 Swift 实现了他们的 HTTP 1.x parser , 但是后来转变为使用 c-http parse（用于 Node.js) 来实现
(2) Vapor 刚开始采用 structs 来实现 HTTP request 和 response, 但是后来转变为采用 classes  
(3) Zewo 使用 structs 来实现 HTTP request 和 response, 但是 Body field 是 enum. 当 enum 是 stream时, 由 class 来支持.  

_假设我们将复用现有的 C libraries 来实现 HTTP 1.x and HTTP/2:_  
* 当我们对已经实现的功能重新实现时，必然会出现不少bug, 主要是由于没有足够多的时间来进行大量实践验证 (c.f. Apache Harmony vs existing Java Class Library code)
* 需要更多的付出来维护，修复bug，还要保持新功能的开发
* 从性能的角度来说, 使用 swift 实现的唯一优势是 inlining (across the Swift/C boundary) 以及一些类型转换 (ie, across the C/Swift API boundary).
* 这样我们就可以集中精力开发 Swift API - 我们可以随时查看一些底层的实现.

  **然而...**  
* 我们必须确保底层的C API不会影响Swift API:
  * 就像 C 驱动的内存设计 等等
  * 确保API在non-Swift实现时也是可用的，作为一种 one-off verification，我们可以通过这种方式来减少影响 (eg. that from Vapor or Swifter).
* 我们应当警惕没有经过大量实践验证的 C API. 我们应该看看HTTP/2的 “nghttp2” 库是否属于这样的 C API
* 使用来自于 node.js 的 c-http 解析器会有代码版权许可问题，由于其是 MIT licensed code.
  *阻止在Apache 2项目中使用MIT许可代码没有任何法律问题，并且已经在Swift的其他地方也采用了（例如libkqueue）。 我们应该跟核心小组进行跟进，了解是否有任何问题



_Value types vs Reference Types:_  
* 我们应该使用 term “semantics” 而不是”types“，因为behaviors和types是分开的.
* 我们非常想使用 Foundation type，所以我们应该在短期内关注于这样做是否是对的. 
* Foundation 开始的时候仅仅是 reference, 我们已经开始去添加 value semantics, 但是这个目前没有完全完成，在所有情况下. 我们现在不应该是关注Foundation里哪有某种类型, 我们应该关注于它们最终应该成为怎样的类型(type)
* 我们确实需要和Ben， Tony Parker/Philippe Hausler 聊聊， 来确定 Foundation types的方向.
* 目前我们认为，一般来说， value semantics 是有用的，就像在 middlewares 中使用 `inout`, 但是将reference 和 value semantics 混合时有个问题 (ie, struct 将stream封装为reference type).
  * 构建一个请求是如何被处理的地图/流程图将有助于构建请求与响应之间的交互模型.

_在Node.js中添加HTTP/2支持的经验_  
* 采用 `nghttp2` library后，Node.js已经变得更加完善
* James Snell 对此的演，线上地址为:https://www.youtube.com/watch?v=7uNGKCao8gA&t=1157s&list=PLfMzBWSH11xYaaHMalNKqcEurBH8LstB8&index=19


### Next steps:
* 邀请 Tony Parker and Philippe Hausler 来聊聊 request/response types 在 Foundation 中的方向
* 和 Swift Core Team 聊聊 MIT licensed code 的使用 
* 为 request handling 和 middlewares的使用创建一个流程图 
* 开始为HTTP解析库构建一个大纲提案，作为 Swift Evolution "pitch"
