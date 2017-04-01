Swift Server APIs: HTTP stream meeting 2


Time: Wednesday March 15th			
6pm CET 			
5pm UK 
1pm EST 
10am PST


WebEx and Agenda/Minutes doc:

WebEx:   		 https://ibm2.webex.com/ibm2-en/j.php?MTID=m59d2772a43f192bf64ae3bdbd5c77faf

password: swift-http

Agenda:    https://docs.google.com/document/d/1GTm7mJH2vCAK-Ts4M3t8C3boJ18XqiFnCRuyp55eKJE/edit?usp=sharing

If you'd like to add anything to the agenda ahead of time, please update the agenda document in Google Docs.

Initial Agenda:
Review of draft HTTP API Proposal 
https://docs.google.com/document/d/1ErHA8RXLiTGvOPHNY0KtbWCvwR1IUz_2ffi7CxCUo_4/edit?usp=sharing

Discussion of key points:
Use of URLRequest/Response
Value Types vs. Reference Types

Attendees:
Chris Bailey
Ben Cohen
Carl Brown
Stevey Brown
Tony Parker
Gregor Milos
Shmuel Kallner
Alex Blewitt
Allen
Chuck Goddard
George
Johannes Weiss
Ludovic Dewailly
Marcin Kliks
Nasser Ebrahim
Steve Algernon
Michael Papp
Thomas Catterall


Minutes:

Use of value types vs. reference types for request and response
    一般的共识是赞成使用value types for request and response.
        参数被传递并不保留 mutated 时应该使用 reference
        应当注意的是，仍然有 NSURLRequest/Response (reference types)原因是 subclass 的存在, 而不是有意要使用 reference types.
        使用 value types 可以帮助避免 closures (like middlewares) 中的 reference cycles，否则需要 weak self 来避免.
        也可以帮助减少额外的 ARC 开销（overhead）
如果 struct/value types 内部嵌套有 references，ARC 开销（overhead）并不会减少 - 并且body中的数据实际上是一个流（stream），必须是 reference type
    在 Foundation中, request and response 仅仅是 headers, body 里的数据是一个 side stream，所以他们必须被分开
    这是将URLSession（与downloadTask一样）转换为文件（file）或流（stream）（对于响应responses也是如此） 
    可以像序列（sequence）一样，它是一个值类型（value type），您可以得到一个对应的 reference type的迭代器 
        Johannes 的经验是，创建一个 framework ，这个framework 将 request/response 同 body stream分离.
        Request and response 是 value types
        处理程序（Handler）提供一个BodyProcessor，当数据块可用时被调用
        当客户端使用分块传输编码时，这一点尤其重要，目前一些框架并未尝试这样做
将request and response headers (metadata) 和 body 分离的普遍设计，可以通过处理程序（handler）API或闭（closure）包单独处理来实现
将request and response headers 和 body stream 分离后，我们如何处理 HTTP trailers?
    Trailers 在 body stream 中，在 headers 传输完成后传过来, 但是会加到 headers
    这些都没有很好地指定，并且可能需要由处理 stream 的任何handledler处理

将 Foundation 的基本 types 用于 request and response
    现在，Foundation types 与一般的 request and response types 不匹配，因为他们是针对某个特定的使用场景来设计的，这是可以接受的。
    我们应该弄清楚Swift的“正确答案”是什么（不考虑Objective-C的兼容性），然后看看它如何适应现有的类型
        绝对可以扩展或修改现有类型
        但是我们需要一个迁移计划，以便不会影响现有的用户
Next Steps:
    定义request and response types所需的内容
        我们如何处理各种头(header)解析规则?
        数据是否应该全部被及时（eagerly）或者延迟处理（lazily）
    定义将 body stream 和 header metadata 分开处理的实现方式


