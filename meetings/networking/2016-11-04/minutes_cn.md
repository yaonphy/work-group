# Networking sub-team kick-off

### Initial Agenda:
* Introductions
* Goals
* Development process overview
* Next Steps

### Attendees:
* Chris Bailey - IBM, Kitura and Swift.org stuff
* William Dillon (hpux735) - Swift-ARM, embedded systems, mac, linux, etc
* Alfredo Delli Bovi - contributions to Swift core. Background in Node.js/PHP
* Steve Algernon - Apple, CF, CFNetwork, NSURLSession
* Paulo Faria - Zewo
* Thiago Holanda - Zewo, iOS Developer and Python.
* Morgan Lieberthal - Mac / backend web developer
* John Lin - iOS dev, Ruby, Python & Erlang
* Illia Tykhonkov - iOS developer
* Matt DeFoor - Condrey Corporation
* Piers Mainwaring - Backend web dev
* Tyler Stromberg – iOS/Mac dev; Ruby, Elixir, Go
* Lasse Jansen - iOS/Mac developer
* Bill Abt - IBM,  Swift@IBM - Kitura, BlueSocket, BlueSSLService
* Johannes Weiss - Apple, iCloud
* Gregor Milos - Apple, iCloud
* Scott Lessans - Full Stack/Robotics; nourish.ai
* Ben Cohen - Swift standard lib @Apple

### Minutes:

_Q) 我们不需要在POSIX上封装一层吗？ 或者是直接使用POSIX层？ 开发者如何以最有效的方式使用APIs? 他们真的需要底层APIs吗？
* 开发者真的只是想使用专注于业务层的高级API。
* 我们采取创建高层级APIs的方式来帮助我们了解底层到底需要怎样的功能?
* 一个使用底层 APIs 的理由是：POSIX C API 和 Swift之间无法兼容的阻碍. 这在 fcntl(http://pubs.opengroup.org/onlinepubs/009695399/basedefs/fcntl.h.html http://baike.baidu.com/link?url=LRYz4VgUUi3JkfXmRWzgFB01XDqkllbrV7JQW105lDlhaqBlgpHuBqDbxEMy1f98DZQcYDZo_B0eKNGERocOsq) 函数的可变参数特性上更为明显, 在相似的struct之间以二进制进行转换时也非常普遍，如 sockaddr。

* 一些容易忽略的底层东西:
  * Errno(错误码) and error(错误) 处理 - 可能是标准库的事情 (POSIX lib)
  * fcntl 和 ioctl 的Swift兼容实现
  * Basic address processing: resolution, conversion, equality（基本地址的处理：解析，转换，比较？？？）


_Q)高层与底层APIs的价值是什么？ 
比如: 创建一个API和嵌入式系统连接。 嵌入式系统使用UDP多播实现了自有的一种机制。 这需要手动加入多播组, 进而发送和接收组多播数据包.
试图用Vapor的网络代码尝试这样做, 但由于Vapor代码中的使用 assumptions 而存在困难, 比如用于管理网络地址结构体的私有方法. 为了实现所需要的，代码必须被fork下来然后修改.


* 我们应该在比较高的层级来暴漏sockets - I/O + config + policy.
* 但是，如果用户需要的功能不能通过高层级API满足，他们将/需要降低到较低级别的API
* 需要降低到特定平台的C代码，还是“Swift”层API？
* 降低到 swift 层是更可取和更安全的.


_Q) 怎么处理个别平台需要一些额外功能的问题?_
用户是否必须在该平台使用C代码, 或者我们应该提供一种方式来实现针对每个平台的extensions
* Swift 提供了 extensions…
* 平台兼容的检测 “#if os(...)”


_Q) 如果我们有高层级的抽象,我们必须assume concurrency model是什么吗？
如果是这样, 将会阻碍其他并发模型的选择, 我们是否需要提供一个较低级别的架构（也可以让使用者选择自己的模型）?_
* 我们不能假设并发模型将会是何种形式 - 因为针对该议题的讨论还没有开始.

* 我们可以提供多层级的API:
  * 底层 - 对于并发没有任何 assumptions. 使用自己的文件描述符（File descriptor） and 使用自己的轮询(polling). 自己选择何种blocking形式的I/O,尽量提供多样的选择?
  * 开始引入异步模型 - 基于默认的 Dispatch
    - 基于默认实现的 Protocol?
  * 高层级别的抽象 - 基于你想要做的事情.

* 位于底层的API可能是更具体的 (由于他们是由标准（RFC）设定的) - 虽然我们必须决定如何让他们更 “Swifty”.


**Note:** 我们需要确保我们使用适当的术语来避免混淆:
* Async vs. Sync (API level)
* Blocking vs. Non-blocking IO (Concurrency model level)

Also, a nice, clear definition of what being more “Swifty” means is probably needed - at least one we can agree on.
