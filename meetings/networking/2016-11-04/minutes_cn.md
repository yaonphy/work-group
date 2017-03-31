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
* 我们采取创建高级APIs的方式来帮助我们了解底层到底需要怎样的功能?
* 一个使用底层 APIs 的理由是：POSIX C API 和 Swift之间无法兼容的阻碍. 这在 fcntl(http://pubs.opengroup.org/onlinepubs/009695399/basedefs/fcntl.h.html http://baike.baidu.com/link?url=LRYz4VgUUi3JkfXmRWzgFB01XDqkllbrV7JQW105lDlhaqBlgpHuBqDbxEMy1f98DZQcYDZo_B0eKNGERocOsq) 函数的可变参数特性上更为明显, 在相似的struct之间以二进制进行转换时也非常普遍，如 sockaddr。

* 一些容易忽略的底层东西:
  * Errno(错误码) and error(错误) 处理 - 可能是标准库的事情 (POSIX lib)
  * fcntl 和 ioctl 的Swift兼容实现
  * Basic address processing: resolution, conversion, equality（基本地址的处理：解析，转换，比较？？？）


_Q) Whats the value of high level vs low level APIs?_
Example: Building an interface with an embedded system. The embedded system implements a proprietary mechanism using UDP multicast.  This necessitated manually joining multicast groups, and sending and receiving multicast packets.
An attempt was made to try to do it with Vapor's network code, but there were difficulties because of the usage assumptions in the Vapor code, such as private methods for managing internet address structures. To achieve what was required, the code had to be forked and modified.


* We should be able to expose sockets at a fairly high level - I/O plus config plus policy.
* But if the fuction a user needs is not there, they will have/need to “drop down” to lower level APIs.
* Is that dropping down to platform specific C code, or to a “Swift” layer API?
  * Dropping down to a Swift layer more is desirable and safer.


_Q) How do we deal with platforms that have additional capabilities?_
Does the user have to use C code there, or should we provide a way to have per-platform extensions?
* Swift provides extensions…
* Platform capability checking “#if os(...)”


_Q) If we have high level abstractions, do we have to assume what the concurrency model is?
If so, will that prevent other concurrency choices being made, or do we need to provide a lower level construct (as well) that allows you to chose your own model?_
* We can’t assume what the concurrency model is going to be yet - because those discussions are yet to happen.

* We could provide APIs that build up in layers:
  * Lowest level - no assumptions on concurrency. File descriptor and use your own polling.  Is blocking I/O and select() enough on this layer?
  * Start to bring in async model - Dispatch based for the “default”
    - Protocol based with a default implementation?
  * Higher level abstractions - based on what you’re trying to do.

* APIs at the lowest layers are likely to be more concrete (as they are RFC specified) - although we have to decide how to make them “Swifty”.


**Note:** We need to ensure that we use proper terminology in order to avoid confusion:
* Async vs. Sync (API level)
* Blocking vs. Non-blocking IO (Concurrency model level)

Also, a nice, clear definition of what being more “Swifty” means is probably needed - at least one we can agree on.
