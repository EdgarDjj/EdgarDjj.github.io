# 面试题-JavaWeb和网络


# 面试题-JavaWeb和网络

## Java Web

64. Jsp 和 Servlet 有什么区别？
JSP的本质是Servlet，JVM只能识别Java类，不能识别jsp的代码，Web容器将jsp代码编译成JVM能够识别的类。
Jsp更擅长 页面表现，而Servlet更擅长逻辑控制。
Servlet中没有内置对象，Jsp中的内置对象都是必须通过HttpServletRequest对象，HTTPServletResponse对象以及HttpServlet对象得到。
Jsp是Servlet的一种简化，使用Jsp只需要完成程序员需要输出到客户端的内容，Jsp中的Java脚本如何嵌入到一个类中，由Jsp容器完成，而Servlet则是个完成的Java类，这个类的Service方法用于生成对客户端的响应。

65. Jsp 有哪些内置对象？作用分别是什么？
JSP有9个内置对象：
* response：封装服务端对客户端的响应
* request：封装客户端的请求
* pageContext：通过该对象可以获取其它对象
* session：封装用户会话对象
* application：封装服务器运行环境的对象
* out：输出服务器响应的输出流独享
* config：web应用的配置对象
* page：JSP页面本身
* exception：封装页面抛出异常的对象

66. 说一下Jsp的4种作用域？
JSP中的四种作用域包括page、request、session和application，具体来说：
* page代表与一个页面相关的对象和属性。
* request代表与Web客户机发出的一个请求相关的对象和属性。一个请求可能跨越多个页面，涉及多个Web组件；需要在页面显示的临时数据可以置于此作用域。
* session代表与某个用户与服务器建立的一次会话相关的对象和属性。跟某个用户相关的数据应该放在用户自己的session中。
* application代表与整个Web应用程序相关的对象和属性，它实质上是跨越整个Web应用程序，包括多个页面、请求和会话的一个全局作用域。
67. session 和 cookie的区别？
由于http是无状态协议，所以需要一种机制来对用户的身份状态进行保存。
session 和 cookie 都是用来跟踪浏览器用户身份的会话方式。
session 存在于服务端，而cookie存在于客户端。
cookie一般用于保存用户信息，例如在某些登录的时候，浏览器自动将你的账号进行填写，相当于给了用户一点甜头，这也是cookie的名称由来。同时是在进行会话的时候，若是使用Token进行安全的验证时候，Token存在于cookie中，访问网站的某一个资源的时候，只需要根据Token值来查找用户即可。登录一次网站后访问网站其它页面的时候就无需进行登录了。
session的主要作用是通过服务端记录用户的状态，同时进行用户的认证和权限访问。

68. 说一下session的工作原理？
session在服务端的存储结构类似于key-value键值对，里面存储的事sessionId
，用户向服务发送请求的时候会带上这个sessionId，这是服务端拿着客户端发送过来的sessionId进行校对，实现安全认证。由于sessionId存储于cookie中，所以客户端在使用sessionId用于认证的时候会出现CSRF（Cross-site request forgery）的危险。

69. 如果客户端禁用了cookie能实现session吗？
失去了cookie相当于失去了sessionId，服务端无法进行身份的校验
但可以通过以下几种方式来使用session
* 将sessionId通过url，隐藏表单传递sessionId
* 用文件、数据库、缓存等显示保存sessionId，在跨页过程中手动调用

70. SpringMVC 和 Struts的区别？
* 拦截机制的不同
Struts2是类级别的拦截，每次请求就会创建一个Action，和Spring整合时Struts2的ActionBean注入作用域是原型模式prototype，然后通过setter，getter吧request数据注入到属性。Struts2中，一个Action对应一个request，response上下文，在接收参数时，可以通过属性接收，这说明属性参数是让多个方法共享的。Struts2中Action的一个方法可以对应一个url，而其类属性却被所有方法共享，这也就无法用注解或其他方式标识其所属方法了，只能设计为多例。

SpringMVC是方法级别的拦截，一个方法对应一个Request上下文，所以方法直接基本上是独立的，独享request，response数据。而每个方法同时又何一个url对应，参数的传递是直接注入到方法中的，是方法所独有的。处理结果通过ModeMap返回给框架。在Spring整合时，SpringMVC的Controller Bean默认单例模式Singleton，所以默认对所有的请求，只会创建一个Controller，有应为没有共享的属性，所以是线程安全的，如果要改变默认的作用域，需要添加@Scope注解修改。
Struts2有自己的拦截Interceptor机制，SpringMVC这是用的是独立的Aop方式，这样导致Struts2的配置文件量还是比SpringMVC大。
* 底层框架的不同
Struts2采用Filter（StrutsPrepareAndExecuteFilter）实现，SpringMVC（DispatcherServlet）则采用Servlet实现。Filter在容器启动之后即初始化；服务停止以后坠毁，晚于Servlet。Servlet在是在调用时初始化，先于Filter调用，服务停止后销毁。
* 性能方面
Struts2是类级别的拦截，每次请求对应实例一个新的Action，需要加载所有的属性值注入，SpringMVC实现了零配置，由于SpringMVC基于方法的拦截，有加载一次单例模式bean注入。所以，SpringMVC开发效率和性能高于Struts2。
* 配置方面　
spring MVC和Spring是无缝的。从这个项目的管理和安全上也比Struts2高。

71. 如何避免sql注入？
使用PreparedStatement
使用正则表达式过滤传入的参数
字符串过滤
服务层进行判断
orm层注意#{}和${}的使用区别

72. 什么是XSS攻击，如何避免？
XSS攻击又称CSS,全称Cross Site Script  （跨站脚本攻击），其原理是攻击者向有XSS漏洞的网站中输入恶意的 HTML 代码，当用户浏览该网站时，这段 HTML 代码会自动执行，从而达到攻击的目的。XSS 攻击类似于 SQL 注入攻击，SQL注入攻击中以SQL语句作为用户输入，从而达到查询/修改/删除数据的目的，而在xss攻击中，通过插入恶意脚本，实现对用户游览器的控制，获取用户的一些信息。 XSS是 Web 程序中常见的漏洞，XSS 属于被动式且用于客户端的攻击方式。
XSS防范的总体思路是：对输入(和URL参数)进行过滤，对输出进行编码。

73. 什么是 CSRF 攻击，如何避免？
CSRF（Cross-site request forgery）也被称为 one-click attack或者 session riding，中文全称是叫跨站请求伪造。一般来说，攻击者通过伪造用户的浏览器的请求，向访问一个用户自己曾经认证访问过的网站发送出去，使目标网站接收并误以为是用户的真实操作而去执行命令。常用于盗取账号、转账、发送虚假消息等。攻击者利用网站对请求的验证漏洞而实现这样的攻击行为，网站能够确认请求来源于用户的浏览器，却不能验证请求是否源于用户的真实意愿下的操作行为。
如何避免：
1. 验证HTTP Refer字段
HTTP Refer字段记录了该HTTP请求的来源地址。
2. 使用验证码
3. 在请求地址中添加token验证
4. 在HTTP头中自定义属性并验证
将TOKEN以参数的形式置于HTTP请求中。通过 XMLHttpRequest 这个类，可以一次性给所有该类请求加上 csrftoken 这个 HTTP 头属性，并把 token 值放入其中。这样解决了上种方法在请求中加入 token 的不便，同时，通过 XMLHttpRequest 请求的地址不会被记录到浏览器的地址栏，也不用担心 token 会透过 Referer 泄露到其他网站中去。

## 网络

79. http 响应码 301 和 302 代表的是什么？有什么区别？
301 redirect：代表永久性重定向（Permanently Moved）
302 redirect：代表暂时性转移（Temporaily Moved）

80. forward 和 redirect 区别？
直接转发（forward）：
客户端和浏览器只发出一次请求，Servlet、HTML、JSP等其它信息资源，由第二个信息资源响应该请求，在请求对象request中，保存的对象对于每个信息资源时共享的。

重定向（redirect）：
实际是两次HTTP请求，服务器在响应第一次的时候，让浏览器再向另一个URL发出请求，从而达到转发的目的。

81. 简述TCP和UDP
TCP是面向连接的（即发送数据前，需要先建立连接，数据传输结束后需要释放连接），TCP是无连接的（无需建立连接即可发送数据）
TCP是字节流传输，UDP是以数据报文段方式
TCP头部有20-60字节，UDP头部为8字节
TCP提供可靠的服务，UDP尽最大努力进行交付
TCP通过校验和，重传控制，序号标识，滑动窗口、确认应答实现可靠传输。如丢包时的重发控制，还可以对次序乱掉的分包进行顺序控制
UDP每个数据包都是独立的，没有固定顺序
每一条TCP连接只能是点到点的;UDP支持一对一，一对多，多对一和多对多的交互通信。

82. TCP为什么要三次握手，两次不行吗，为什么？
通过三次握手，TCP能确定双方互相的发送数据能力、接收数据能力正常，两次无法保证能进行全部确定。
为了实现数据的可靠传输，TCP协议的通信双方，都必须维护一个序列号，以标识发送出去的数据包，哪些是被对方收到的，三次握手的过程即使通信双方相互告知序列的起始值，并确认对象已经收到序列号起始值的必经步骤。
如果只是两次握手，至多只有连接发起方的起始序列号能喂确认，另一方得不到确认。

83. 说一下TCP粘包是怎么产生的？


84. OSI的七层模型
应用层：网路服务与用户的最终一个借口
表示层：数据的表示、安全、压缩
会话层：建立、管理、终止会话
传输层：定义传输数据的协议端口号，以及流量控制和差错校验
网络层：进行逻辑地址寻址，实现不同网络之间的路径选择
数据链路层：建立逻辑连接，进行硬件地址寻址、差错校验等功能
物理层：建立、维护、断开物理连接

85. GET和 POST请求
* GET在浏览器回退时是无害的，而POST会再次提交请求。
* GET产生的URL地址可以被Bookmark，而POST不可以。
* GET请求会被浏览器主动cache，而POST不会，除非手动设置。
* GET请求只能进行url编码，而POST支持多种编码方式。
* GET请求参数会被完整保留在浏览器历史记录里，而POST中的参数不会被保留。
* GET请求在URL中传送的参数是有长度限制的，而POST么有。
* 对参数的数据类型，GET只接受ASCII字符，而POST没有限制。
* GET比POST更不安全，因为参数直接暴露在URL上，所以不能用来传递敏感信息。
* GET参数通过URL传递，POST放在Request body中。
86. 如何实现跨域
方式一：图片ping或script标签跨域
图片ping常用于跟踪用户点击页面或动态广告曝光次数。 script标签可以得到从其他来源数据，这也是JSONP依赖的根据。 
方式二：JSONP跨域
JSONP（JSON with Padding）是数据格式JSON的一种“使用模式”，可以让网页从别的网域要数据。根据 XmlHttpRequest 对象受到同源策略的影响，而利用 <script>元素的这个开放策略，网页可以得到从其他来源动态产生的JSON数据，而这种使用模式就是所谓的 JSONP。用JSONP抓到的数据并不是JSON，而是任意的JavaScript，用 JavaScript解释器运行而不是用JSON解析器解析。所有，通过Chrome查看所有JSONP发送的Get请求都是js类型，而非XHR。 
方式三：CORS
Cross-Origin Resource Sharing（CORS）跨域资源共享是一份浏览器技术的规范，提供了 Web 服务从不同域传来沙盒脚本的方法，以避开浏览器的同源策略，确保安全的跨域数据传输。现代浏览器使用CORS在API容器如XMLHttpRequest来减少HTTP请求的风险来源。与 JSONP 不同，CORS 除了 GET 要求方法以外也支持其他的 HTTP 要求。服务器一般需要增加如下响应头的一种或几种：
	

Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Allow-Headers: X-PINGOTHER, Content-Type
Access-Control-Max-Age: 86400

跨域请求默认不会携带Cookie信息，如果需要携带，请配置下述参数：

"Access-Control-Allow-Credentials": true
// Ajax设置
"withCredentials": true

方式四：window.name+iframe
window.name通过在iframe（一般动态创建i）中加载跨域HTML文件来起作用。然后，HTML文件将传递给请求者的字符串内容赋值给window.name。然后，请求者可以检索window.name值作为响应。
* iframe标签的跨域能力；
* window.name属性值在文档刷新后依旧存在的能力（且最大允许2M左右）。
每个iframe都有包裹它的window，而这个window是top window的子窗口。contentWindow属性返回<iframe>元素的Window对象。你可以使用这个Window对象来访问iframe的文档及其内部DOM。


方式五：window.postMessage()
HTML5新特性，可以用来向其他所有的 window 对象发送消息。需要注意的是我们必须要保证所有的脚本执行完才发送 MessageEvent，如果在函数执行的过程中调用了它，就会让后面的函数超时无法执行。


方式六：修改document.domain跨子域
前提条件：这两个域名必须属于同一个基础域名!而且所用的协议，端口都要一致，否则无法利用document.domain进行跨域，所以只能跨子域
在根域范围内，允许把domain属性的值设置为它的上一级域。例如，在”aaa.xxx.com”域内，可以把domain设置为 “xxx.com” 但不能设置为 “xxx.org” 或者”com”。
现在存在两个域名aaa.xxx.com和bbb.xxx.com。在aaa下嵌入bbb的页面，由于其document.name不一致，无法在aaa下操作bbb的js。可以在aaa和bbb下通过js将document.name = 'xxx.com';设置一致，来达到互相访问的作用。

方式七：WebSocket
WebSocket protocol 是HTML5一种新的协议。它实现了浏览器与服务器全双工通信，同时允许跨域通讯，是server push技术的一种很棒的实现。相关文章，请查看：WebSocket、WebSocket-SockJS
需要注意：WebSocket对象不支持DOM 2级事件侦听器，必须使用DOM 0级语法分别定义各个事件。

方式八：代理
同源策略是针对浏览器端进行的限制，可以通过服务器端来解决该问题
DomainA客户端（浏览器） ==> DomainA服务器 ==> DomainB服务器 ==> DomainA客户端（浏览器）
87. 说一下JSONP实现原理？
jsonp 即 json + padding，动态创建script标签，利用script标签的src可以获取任何域下的js脚本，通过这个特性(也可以说漏洞)，服务器端不在返回json格式，而是返回一段调用某个函数的js代码，在src中进行了调用，这样实现了跨域。

