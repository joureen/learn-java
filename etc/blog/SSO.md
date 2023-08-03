# SSO

## 什么是SSO

单点登录: Single Sign On (简称**SSO**)

我们最初学习的时候，一般都是单机系统，也就是说所有的功能全部写在同一个系统上。

而后来，我们为了提高系统性能和可维护性，并且可以合理利用资源、降低耦合性，我们学习会**分布式**，把单个系统拆分为多个模块。

![image-20230714134349032](https://gitee.com/sky-dog/note/raw/master/img/202307141343147.png)

这时候单点登录就出现了作用，我们这里以阿里的天猫和淘宝为例：

我们可以很明显的知道这是两个系统，但是当你登录了天猫的时候，淘宝也会自动登录

<img src="https://gitee.com/sky-dog/note/raw/master/img/202307141345548.png" alt="image-20230714134504492" style="zoom:67%;" />

简单来说，单点登录就是**在多个系统中，用户只需一次登录，各个系统即可感知到该用户已经登录**。这就是单点登录的意义。

而单点登录的应用，我们在很多系统中都能见到。

## SSO原理

在SSO体系中，主要包括三部分：

* User (多个，一般指浏览器等客户端应用)
* Web服务 (多个，一般指你开发的web接口服务等)
* SSO 认证中心 (1个，专门来解决单点登录问题)

而SSO的实现基本核心原则如下：

* 所有的登录都在 SSO 认证中心进行
* SSO 认证中心通过一些方法来告诉 Web 应用当前访问用户究竟是不是已通过认证的用户
* SSO 认证中心和所有的 Web 应用建立一种信任关系， SSO 认证中心对用户身份正确性的判断会通过某种方法告之 Web 应用，而且判断结果必须被 Web 应用信任。

### 登录

上面介绍我们知道，在SSO中有一个独立的认证中心，只有认证中心能接受用户的用户名密码等安全信息，其他系统不提供登录入口，只接受认证中心的间接授权。

那其他的系统如何访问受保护的资源？

这里就是通过认证中心间接授权通过token令牌来实现，当SSO验证了用户信息的正确性后，就会创建授权token，在接下来的跳转过程中，授权令牌作为参数发送给各个子系统，子系统拿到令牌，即得到了授权，可以借此创建局部会话，局部会话登录方式与单系统的登录方式相同。

![image-20230714135639832](https://gitee.com/sky-dog/note/raw/master/img/202307141356931.png)

大部分的SSO系统大致这样的一个流程: 

这里我们举个例子，比如我们连接福大的`校园网`之后，我们访问某个网站，这时候我们校园网会给你重定向到一个`福大统一认证中心`

(这个就可以理解为是SSO的服务)，当我们在认证中心登录帐号之后，用户与认证中心会建立全局绘画（一份生成token，写道cookie中，保留在浏览器）。然后认证在想会**重定向**回你原本访问的服务，重定向的地址大概可能会是这个样子:  `www.bilibili.com?token=balabalabalbalbablablabla`，你访问的系统就会去找sso中心去验证token是否正确。

(上述例子不一定准确，仅供理解参考)

### 注销

既然有登陆那么就自然有注销，单点登录也要单点注销，在一个子系统中注销，所有子系统的会话都将被销毁。原理图如下：

![image-20230714140509685](https://gitee.com/sky-dog/note/raw/master/img/202307141405751.png)

SSO认证中心一直监听全局会话的状态，一旦全局会话销毁，监听器将通知所有注册系统执行注销操作

同样的我们也来分析一下具体的流程：

* 用户向系统1发起注销请求
* 系统1根据用户与系统1建立的session拿到token，向sso提出注销请求
* sso验证token有效，销毁全局session，同时拿到所有使用这个token的系统地址
* sso向所有的系统发起注销请求
* 各个系统注销，销毁局部绘画
* sso重定向到登录界面

## CAS原理

CAS，CAS全称为Central Authentication Service即**中央认证服务**，是一个企业多语言单点登录的解决方案，并努力去成为一个身份验证和授权需求的综合平台。

CAS系统主要分成两部分:

* Cas Server
  * 负责完成对用户的认证工作，需要独立部署，Cas Server会处理用户名/密码等凭证(Credentials)
* Cas Client
  * 负责处理对客户端受保护资源的访问请求(就是用户对需要登录才能访问的请求)，需要对请求方进行身份认证时，重定向到Cas Server进行认证。(原则上，Cas Client 不应该再接收任何有关用户名的账号或密码等Credentials)

### CAS协议

CAS协议是一个简单而强大的基于票据的协议，它涉及一个或多个客户端和一台服务器。即在CAS中，通过**TGT**(Ticket Granting Ticket)来获取 **ST**(Service Ticket)，通过ST来访问具体服务。

其中主要的关键概念：

* TGT（Ticket Granting Ticket）是存储在TGCcookie中的代表用户的SSO会话。
* 该ST（Service Ticket），作为参数在GET方法的URL中，代表由CAS服务器授予访问CASified应用程序（包含CAS客户端的应用程序）具体用户的权限。

#### 基本协议模式

结合官方的流程图，我们可以知道，CAS中的单点登录流程：

![image-20230714141522403](https://gitee.com/sky-dog/note/raw/master/img/202307141415490.png)

我们可以发现这里的流程与上面SSO执行的基本是一致，只是CAS协议流程图中，更加清楚的指定了我们在访问过程当中的各种情况，在SSO中令牌也是我们在CAS中的ST(Service Ticket)。

首先用户访问受保护的资源，权限没有认证，所以会把请求的URL以参数跳转到CAS认证中心，CAS认证中心发现没有SSO session，所以弹出登录页面，输入用户信息，提交到CAS认证中心进行信息的认证，如果信息正确，CAS认证中心就会创建一个SSO session和CASTGC cookie，这个CASTGC cookie包含了TGT，而用户session则以TGT为key创建，同时服务端会分发一个ST返回给用户。用户拿到了ST后，访问带参数ST的资源地址，同时应用将ST发送给CAS认证中心，CAS认证中心对ST进行校验，同时判断相应的cookie（包含TGT）是否正确（通过先前设定的key），判断ST是否是有效的，结果会返回一个包含成功信息的XML给应用。应用在建立相应的session和cookie跳转到浏览器，用户再通过浏览器带cookie去应用访问受保护的资源地址，cookie和后端session验证成功便可以成功访问到信息。

第二次访问应用时，浏览器就会携带相应的cookie信息，后台session验证用户是否登录，与一般单系统应用登录模式一样。

当我们访问其他的应用，与前面的步骤也是基本相同，首先用户访问受保护的资源，跳转回浏览器，浏览器含有先前登录的CASTGC cookie，CASTGC cookie包含了TGT并发送到CAS认证中心，CAS认证中心校验TGT是否有效，如果有效分发浏览器一个带ST参数的资源地址URL，应用程序拿到ST后，再发送给CAS认证中心，如果认证了ST有效后，结果会返回一个包含成功信息的XML给应用。同样的步骤，应用在建立相应的session和cookie跳转到浏览器，用户再通过浏览器带cookie去应用访问受保护的资源地址，验证session成功便可以成功访问到信息。