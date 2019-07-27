# 理解OAuth 2.0

> 原文出处：http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html
> 作者： [阮一峰](http://www.ruanyifeng.com/)

[OAuth](http://en.wikipedia.org/wiki/OAuth)是一个关于授权（authorization）的开放网络标准，在全世界得到广泛应用，目前的版本是2.0版。

本文对OAuth 2.0的设计思路和运行流程，做一个简明通俗的解释，主要参考材料为[RFC 6749](https://tools.ietf.org/html/rfc6749)。

![OAuth Logo](image/oauth2.png)

##  一、应用场景及解释

[OAuth 2.0](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html) 是目前最流行的授权机制，用来授权第三方应用，获取用户数据。

这个标准比较抽象，使用了很多术语，初学者不容易理解。其实说起来并不复杂，下面我就通过一个简单的类比，帮助大家轻松理解，OAuth 2.0 到底是什么。

### 1.  云冲印

有一个"云冲印"的网站，可以将用户储存在Google的照片，冲印出来。用户为了使用该服务，必须让"云冲印"读取自己储存在Google上的照片。

![](image/oauth2-1.png)

问题是只有得到用户的授权，Google才会同意"云冲印"读取这些照片。那么，"云冲印"怎样获得用户的授权呢？

传统方法是，用户将自己的Google用户名和密码，告诉"云冲印"，后者就可以读取用户的照片了。这样的做法有以下几个严重的缺点。

> （1）"云冲印"为了后续的服务，会保存用户的密码，这样很不安全。
>
> （2）Google不得不部署密码登录，而我们知道，单纯的密码登录并不安全。
>
> （3）"云冲印"拥有了获取用户储存在Google所有资料的权力，用户没法限制"云冲印"获得授权的范围和有效期。
>
> （4）用户只有修改密码，才能收回赋予"云冲印"的权力。但是这样做，会使得其他所有获得用户授权的第三方应用程序全部失效。
>
> （5）只要有一个第三方应用程序被破解，就会导致用户密码泄漏，以及所有被密码保护的数据泄漏。

### 2. 快递员

我住在一个大型的居民小区。小区有门禁系统。进入的时候需要输入密码。我经常网购和外卖，每天都有快递员来送货。我必须找到一个办法，让快递员通过门禁系统，进入小区。如果我把自己的密码，告诉快递员，他就拥有了与我同样的权限，这样好像不太合适。万一我想取消他进入小区的权力，也很麻烦，我自己的密码也得跟着改了，还得通知其他的快递员。

有没有一种办法，让快递员能够自由进入小区，又不必知道小区居民的密码，而且他的唯一权限就是送货，其他需要密码的场合，他都没有权限？

####  授权机制的设计

于是，我设计了一套授权机制。

>（1）门禁系统的密码输入器下面，增加一个按钮，叫做"获取授权"。快递员需要首先按这个按钮，去申请授权。
>
>（2）他按下按钮以后，屋主（也就是我）的手机就会跳出对话框：有人正在要求授权。系统还会显示该快递员的姓名、工号和所属的快递公司。
>
>我确认请求属实，就点击按钮，告诉门禁系统，我同意给予他进入小区的授权。
>
>（3）门禁系统得到我的确认以后，向快递员显示一个进入小区的令牌（access token）。令牌就是类似密码的一串数字，只在短期内（比如七天）有效。
>
>（4）快递员向门禁系统输入令牌，进入小区。
>
>有人可能会问，为什么不是远程为快递员开门，而要为他单独生成一个令牌？这是因为快递员可能每天都会来送货，第二天他还可以复用这个令牌。另外，有的小区有多重门禁，快递员可以使用同一个令牌通过它们。
>
>

#### 互联网场景

我们把上面的例子搬到互联网，就是 OAuth 的设计了。

> 首先，居民小区就是储存用户数据的网络服务。比如，微信储存了我的好友信息，获取这些信息，就必须经过微信的"门禁系统"。
>
> 其次，快递员（或者说快递公司）就是第三方应用，想要穿过门禁系统，进入小区。
>
> 最后，我就是用户本人，同意授权第三方应用进入小区，获取我的数据。
>
> **简单说，OAuth 就是一种授权机制。数据的所有者告诉系统，同意授权第三方应用进入系统，获取这些数据。系统从而产生一个短期的进入令牌（token），用来代替密码，供第三方应用使用。**



#### 令牌与密码

令牌（token）与密码（password）的作用是一样的，都可以进入系统，但是有三点差异。

>（1）令牌是短期的，到期会自动失效，用户自己无法修改。密码一般长期有效，用户不修改，就不会发生变化。
>
>（2）令牌可以被数据所有者撤销，会立即失效。以上例而言，屋主可以随时取消快递员的令牌。密码一般不允许被他人撤销。
>
>（3）令牌有权限范围（scope），比如只能进小区的二号门。对于网络服务来说，只读令牌就比读写令牌更安全。密码一般是完整权限。
>
>上面这些设计，保证了令牌既可以让第三方应用获得权限，同时又随时可控，不会危及系统安全。这就是 OAuth 2.0 的优点。
>
>注意，只要知道了令牌，就能进入系统。系统一般不会再次确认身份，所以**令牌必须保密，泄漏令牌与泄漏密码的后果是一样的。** 这也是为什么令牌的有效期，一般都设置得很短的原因。
>
>OAuth 2.0 对于如何颁发令牌的细节，规定得非常详细。具体来说，一共分成四种授权类型（authorization grant），即四种颁发令牌的方式，适用于不同的互联网场景。

OAuth就是为了解决上面这些问题而诞生的。

##  二、OAuth 2.0 的四种方式

### RFC 6749

OAuth 2.0 的标准是 [RFC 6749](https://tools.ietf.org/html/rfc6749) 文件。该文件先解释了 OAuth 是什么。

> OAuth 引入了一个授权层，用来分离两种不同的角色：客户端和资源所有者。......资源所有者同意以后，资源服务器可以向客户端颁发令牌。客户端通过令牌，去请求数据。

这段话的意思就是，**OAuth 的核心就是向第三方应用颁发令牌。**然后，RFC 6749 接着写道：

> （由于互联网有多种场景，）本标准定义了获得令牌的四种授权方式（authorization grant ）。

也就是说，**OAuth 2.0 规定了四种获得令牌的流程。你可以选择最适合自己的那一种，向第三方应用颁发令牌。**下面就是这四种授权方式。

> - 授权码（authorization-code）
> - 隐藏式（implicit）
> - 密码式（password）：
> - 客户端凭证（client credentials）

注意，不管哪一种授权方式，第三方应用申请令牌之前，都必须先到系统备案，说明自己的身份，然后会拿到两个身份识别码：客户端 ID（client ID）和客户端密钥（client secret）。这是为了防止令牌被滥用，没有备案过的第三方应用，是不会拿到令牌的。

在详细讲解OAuth 2.0之前，需要了解几个专用名词。它们对读懂后面的讲解，尤其是几张图，至关重要。

> （1） Third-party application：第三方应用程序，本文中又称"客户端"（client），即上一节例子中的"云冲印"。
>
> （2）HTTP service：HTTP服务提供商，本文中简称"服务提供商"，即上一节例子中的Google。
>
> （3）Resource Owner：资源所有者，本文中又称"用户"（user）。
>
> （4）User Agent：用户代理，本文中就是指浏览器。
>
> （5）Authorization server：认证服务器，即服务提供商专门用来处理认证的服务器。
>
> （6）Resource server：资源服务器，即服务提供商存放用户生成的资源的服务器。它与认证服务器，可以是同一台服务器，也可以是不同的服务器。

知道了上面这些名词，就不难理解，OAuth的作用就是让"客户端"安全可控地获取"用户"的授权，与"服务商提供商"进行互动。

###  OAuth的思路

OAuth在"客户端"与"服务提供商"之间，设置了一个授权层（authorization layer）。"客户端"不能直接登录"服务提供商"，只能登录授权层，以此将用户与客户端区分开来。"客户端"登录授权层所用的令牌（token），与用户的密码不同。用户可以在登录的时候，指定授权层令牌的权限范围和有效期。

"客户端"登录授权层以后，"服务提供商"根据令牌的权限范围和有效期，向"客户端"开放用户储存的资料。

###  运行流程

OAuth 2.0的运行流程如下图，摘自RFC 6749。

![OAuth运行流程](image/oauth2-2.png)

> （A）用户打开客户端以后，客户端要求用户给予授权。
>
> （B）用户同意给予客户端授权。
>
> （C）客户端使用上一步获得的授权，向认证服务器申请令牌。
>
> （D）认证服务器对客户端进行认证以后，确认无误，同意发放令牌。
>
> （E）客户端使用令牌，向资源服务器申请获取资源。
>
> （F）资源服务器确认令牌无误，同意向客户端开放资源。

不难看出来，上面六个步骤之中，B是关键，即用户怎样才能给于客户端授权。有了这个授权以后，客户端就可以获取令牌，进而凭令牌获取资源。


###  授权码模式

**授权码（authorization code）方式，指的是第三方应用先申请一个授权码，然后再用该码获取令牌。**

这种方式是最常用的流程，安全性也最高，它适用于那些有后端的 Web 应用。授权码通过前端传送，令牌则是储存在后端，而且所有与资源服务器的通信都在后端完成。这样的前后端分离，可以避免令牌泄漏。

授权码模式（authorization code）是功能最完整、流程最严密的授权模式。它的特点就是通过客户端的后台服务器，与"服务提供商"的认证服务器进行互动。

![授权码模式](image/oauth2-3.png)

它的步骤如下：

> （A）用户访问客户端，后者将前者导向认证服务器。
>
> （B）用户选择是否给予客户端授权。
>
> （C）假设用户给予授权，认证服务器将用户导向客户端事先指定的"重定向URI"（redirection URI），同时附上一个授权码。
>
> （D）客户端收到授权码，附上早先的"重定向URI"，向认证服务器申请令牌。这一步是在客户端的后台的服务器上完成的，对用户不可见。
>
> （E）认证服务器核对了授权码和重定向URI，确认无误后，向客户端发送访问令牌（access token）和更新令牌（refresh token）。

下面是上面这些步骤所需要的参数。

A步骤中，客户端申请认证的URI，包含以下参数：

- response_type：表示授权类型，必选项，此处的值固定为"code"
- client_id：表示客户端的ID，必选项
- redirect_uri：表示重定向URI，可选项
- scope：表示申请的权限范围，可选项
- state：表示客户端的当前状态，可以指定任意值，认证服务器会原封不动地返回这个值。

下面是一个例子。

> ```
> GET /authorize?response_type=code&client_id=s6BhdRkqt3&state=xyz
>         &redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb HTTP/1.1
> Host: server.example.com
> ```

C步骤中，服务器回应客户端的URI，包含以下参数：

- code：表示授权码，必选项。该码的有效期应该很短，通常设为10分钟，客户端只能使用该码一次，否则会被授权服务器拒绝。该码与客户端ID和重定向URI，是一一对应关系。
- state：如果客户端的请求中包含这个参数，认证服务器的回应也必须一模一样包含这个参数。

下面是一个例子。

> ```
> HTTP/1.1 302 Found
> Location: https://client.example.com/cb?code=SplxlOBeZQQYbYS6WxSbIA
>           &state=xyz
> ```

D步骤中，客户端向认证服务器申请令牌的HTTP请求，包含以下参数：

- grant_type：表示使用的授权模式，必选项，此处的值固定为"authorization_code"。
- code：表示上一步获得的授权码，必选项。
- redirect_uri：表示重定向URI，必选项，且必须与A步骤中的该参数值保持一致。
- client_id：表示客户端ID，必选项。

下面是一个例子。

> ```
> POST /token HTTP/1.1
> Host: server.example.com
> Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
> Content-Type: application/x-www-form-urlencoded
> 
> grant_type=authorization_code&code=SplxlOBeZQQYbYS6WxSbIA
> &redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb
> ```

E步骤中，认证服务器发送的HTTP回复，包含以下参数：

- access_token：表示访问令牌，必选项。
- token_type：表示令牌类型，该值大小写不敏感，必选项，可以是bearer类型或mac类型。
- expires_in：表示过期时间，单位为秒。如果省略该参数，必须其他方式设置过期时间。
- refresh_token：表示更新令牌，用来获取下一次的访问令牌，可选项。
- scope：表示权限范围，如果与客户端申请的范围一致，此项可省略。

下面是一个例子。

> ```
>      HTTP/1.1 200 OK
>      Content-Type: application/json;charset=UTF-8
>      Cache-Control: no-store
>      Pragma: no-cache
> 
>      {
>        "access_token":"2YotnFZFEjr1zCsicMWpAA",
>        "token_type":"example",
>        "expires_in":3600,
>        "refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",
>        "example_parameter":"example_value"
>      }
> ```

从上面代码可以看到，相关参数使用JSON格式发送（Content-Type: application/json）。此外，HTTP头信息中明确指定不得缓存。



第一步，A 网站提供一个链接，用户点击后就会跳转到 B 网站，授权用户数据给 A 网站使用。下面就是 A 网站跳转 B 网站的一个示意链接。

> ```javascript
> https://b.com/oauth/authorize?
>   response_type=code&
>   client_id=CLIENT_ID&
>   redirect_uri=CALLBACK_URL&
>   scope=read
> ```

上面 URL 中，`response_type`参数表示要求返回授权码（`code`），`client_id`参数让 B 知道是谁在请求，`redirect_uri`参数是 B 接受或拒绝请求后的跳转网址，`scope`参数表示要求的授权范围（这里是只读）。

![img](image/oauth2-7.jpg)

第二步，用户跳转后，B 网站会要求用户登录，然后询问是否同意给予 A 网站授权。用户表示同意，这时 B 网站就会跳回`redirect_uri`参数指定的网址。跳转时，会传回一个授权码，就像下面这样。

> ```javascript
> https://a.com/callback?code=AUTHORIZATION_CODE
> ```

上面 URL 中，`code`参数就是授权码。

![img](image/oauth2-8.jpg)

第三步，A 网站拿到授权码以后，就可以在后端，向 B 网站请求令牌。

> ```javascript
> https://b.com/oauth/token?
>  client_id=CLIENT_ID&
>  client_secret=CLIENT_SECRET&
>  grant_type=authorization_code&
>  code=AUTHORIZATION_CODE&
>  redirect_uri=CALLBACK_URL
> ```

上面 URL 中，`client_id`参数和`client_secret`参数用来让 B 确认 A 的身份（`client_secret`参数是保密的，因此只能在后端发请求），`grant_type`参数的值是`AUTHORIZATION_CODE`，表示采用的授权方式是授权码，`code`参数是上一步拿到的授权码，`redirect_uri`参数是令牌颁发后的回调网址。

![img](image/oauth2-9.jpg)

第四步，B 网站收到请求以后，就会颁发令牌。具体做法是向`redirect_uri`指定的网址，发送一段 JSON 数据。

> ```javascript
> {    
>   "access_token":"ACCESS_TOKEN",
>   "token_type":"bearer",
>   "expires_in":2592000,
>   "refresh_token":"REFRESH_TOKEN",
>   "scope":"read",
>   "uid":100101,
>   "info":{...}
> }
> ```

上面 JSON 数据中，`access_token`字段就是令牌，A 网站在后端拿到了。

![img](image/oauth2-10.jpg)



## 隐藏模式

有些 Web 应用是纯前端应用，没有后端。这时就不能用上面的方式了，必须将令牌储存在前端。**RFC 6749 就规定了第二种方式，允许直接向前端颁发令牌。这种方式没有授权码这个中间步骤，所以称为（授权码）"隐藏式"（implicit）。**

隐藏模式（implicit grant type）不通过第三方应用程序的服务器，直接在浏览器中向认证服务器申请令牌，跳过了"授权码"这个步骤，因此得名。所有步骤在浏览器中完成，令牌对访问者是可见的，且客户端不需要认证。

![简化模式](image/oauth2-4.png)

它的步骤如下：

> （A）客户端将用户导向认证服务器。
>
> （B）用户决定是否给于客户端授权。
>
> （C）假设用户给予授权，认证服务器将用户导向客户端指定的"重定向URI"，并在URI的Hash部分包含了访问令牌。
>
> （D）浏览器向资源服务器发出请求，其中不包括上一步收到的Hash值。
>
> （E）资源服务器返回一个网页，其中包含的代码可以获取Hash值中的令牌。
>
> （F）浏览器执行上一步获得的脚本，提取出令牌。
>
> （G）浏览器将令牌发给客户端。

下面是上面这些步骤所需要的参数。

A步骤中，客户端发出的HTTP请求，包含以下参数：

- response_type：表示授权类型，此处的值固定为"token"，必选项。
- client_id：表示客户端的ID，必选项。
- redirect_uri：表示重定向的URI，可选项。
- scope：表示权限范围，可选项。
- state：表示客户端的当前状态，可以指定任意值，认证服务器会原封不动地返回这个值。

下面是一个例子。

> ```
>     GET /authorize?response_type=token&client_id=s6BhdRkqt3&state=xyz
>         &redirect_uri=https%3A%2F%2Fclient%2Eexample%2Ecom%2Fcb HTTP/1.1
>     Host: server.example.com
> ```

C步骤中，认证服务器回应客户端的URI，包含以下参数：

- access_token：表示访问令牌，必选项。
- token_type：表示令牌类型，该值大小写不敏感，必选项。
- expires_in：表示过期时间，单位为秒。如果省略该参数，必须其他方式设置过期时间。
- scope：表示权限范围，如果与客户端申请的范围一致，此项可省略。
- state：如果客户端的请求中包含这个参数，认证服务器的回应也必须一模一样包含这个参数。

下面是一个例子。

> ```
>      HTTP/1.1 302 Found
>      Location: http://example.com/cb#access_token=2YotnFZFEjr1zCsicMWpAA
>                &state=xyz&token_type=example&expires_in=3600
> ```

在上面的例子中，认证服务器用HTTP头信息的Location栏，指定浏览器重定向的网址。注意，在这个网址的Hash部分包含了令牌。

根据上面的D步骤，下一步浏览器会访问Location指定的网址，但是Hash部分不会发送。接下来的E步骤，服务提供商的资源服务器发送过来的代码，会提取出Hash中的令牌。



第一步，A 网站提供一个链接，要求用户跳转到 B 网站，授权用户数据给 A 网站使用。

> ```javascript
> https://b.com/oauth/authorize?
>   response_type=token&
>   client_id=CLIENT_ID&
>   redirect_uri=CALLBACK_URL&
>   scope=read
> ```

上面 URL 中，`response_type`参数为`token`，表示要求直接返回令牌。

第二步，用户跳转到 B 网站，登录后同意给予 A 网站授权。这时，B 网站就会跳回`redirect_uri`参数指定的跳转网址，并且把令牌作为 URL 参数，传给 A 网站。

> ```javascript
> https://a.com/callback#token=ACCESS_TOKEN
> ```

上面 URL 中，`token`参数就是令牌，A 网站因此直接在前端拿到令牌。

注意，令牌的位置是 URL 锚点（fragment），而不是查询字符串（querystring），这是因为 OAuth 2.0 允许跳转网址是 HTTP 协议，因此存在"中间人攻击"的风险，而浏览器跳转时，锚点不会发到服务器，就减少了泄漏令牌的风险。

![img](image/oauth2-11.jpg)

这种方式把令牌直接传给前端，是很不安全的。因此，只能用于一些安全要求不高的场景，并且令牌的有效期必须非常短，通常就是会话期间（session）有效，浏览器关掉，令牌就失效了。



### 密码模式

**如果你高度信任某个应用，RFC 6749 也允许用户把用户名和密码，直接告诉该应用。该应用就使用你的密码，申请令牌，这种方式称为"密码式"（password）。**

密码模式（Resource Owner Password Credentials Grant）中，用户向客户端提供自己的用户名和密码。客户端使用这些信息，向"服务商提供商"索要授权。

在这种模式中，用户必须把自己的密码给客户端，但是客户端不得储存密码。这通常用在用户对客户端高度信任的情况下，比如客户端是操作系统的一部分，或者由一个著名公司出品。而认证服务器只有在其他授权模式无法执行的情况下，才能考虑使用这种模式。

![密码模式](image/oauth2-5.png)

它的步骤如下：

> （A）用户向客户端提供用户名和密码。
>
> （B）客户端将用户名和密码发给认证服务器，向后者请求令牌。
>
> （C）认证服务器确认无误后，向客户端提供访问令牌。

B步骤中，客户端发出的HTTP请求，包含以下参数：

- grant_type：表示授权类型，此处的值固定为"password"，必选项。
- username：表示用户名，必选项。
- password：表示用户的密码，必选项。
- scope：表示权限范围，可选项。

下面是一个例子。

> ```
>      POST /token HTTP/1.1
>      Host: server.example.com
>      Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
>      Content-Type: application/x-www-form-urlencoded
> 
>      grant_type=password&username=johndoe&password=A3ddj3w
> ```

C步骤中，认证服务器向客户端发送访问令牌，下面是一个例子。

> ```
>      HTTP/1.1 200 OK
>      Content-Type: application/json;charset=UTF-8
>      Cache-Control: no-store
>      Pragma: no-cache
> 
>      {
>        "access_token":"2YotnFZFEjr1zCsicMWpAA",
>        "token_type":"example",
>        "expires_in":3600,
>        "refresh_token":"tGzv3JOkF0XG5Qx2TlKWIA",
>        "example_parameter":"example_value"
>      }
> ```

上面代码中，各个参数的含义参见《授权码模式》一节。

整个过程中，客户端不得保存用户的密码。



第一步，A 网站要求用户提供 B 网站的用户名和密码。拿到以后，A 就直接向 B 请求令牌。

> ```javascript
> https://oauth.b.com/token?
>   grant_type=password&
>   username=USERNAME&
>   password=PASSWORD&
>   client_id=CLIENT_ID
> ```

上面 URL 中，`grant_type`参数是授权方式，这里的`password`表示"密码式"，`username`和`password`是 B 的用户名和密码。

第二步，B 网站验证身份通过后，直接给出令牌。注意，这时不需要跳转，而是把令牌放在 JSON 数据里面，作为 HTTP 回应，A 因此拿到令牌。

这种方式需要用户给出自己的用户名/密码，显然风险很大，因此只适用于其他授权方式都无法采用的情况，而且必须是用户高度信任的应用。



## 凭证模式

**最后一种方式是凭证式（client credentials），适用于没有前端的命令行应用，即在命令行下请求令牌。**

凭证模式（Client Credentials Grant）指客户端以自己的名义，而不是以用户的名义，向"服务提供商"进行认证。严格地说，客户端模式并不属于OAuth框架所要解决的问题。在这种模式中，用户直接向客户端注册，客户端以自己的名义要求"服务提供商"提供服务，其实不存在授权问题。

![客户端模式](image/oauth2-6.png)

它的步骤如下：

> （A）客户端向认证服务器进行身份认证，并要求一个访问令牌。
>
> （B）认证服务器确认无误后，向客户端提供访问令牌。

A步骤中，客户端发出的HTTP请求，包含以下参数：

- granttype：表示授权类型，此处的值固定为"clientcredentials"，必选项。
- scope：表示权限范围，可选项。

> ```
>      POST /token HTTP/1.1
>      Host: server.example.com
>      Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
>      Content-Type: application/x-www-form-urlencoded
> 
>      grant_type=client_credentials
> ```

认证服务器必须以某种方式，验证客户端身份。

B步骤中，认证服务器向客户端发送访问令牌，下面是一个例子。

> ```
>      HTTP/1.1 200 OK
>      Content-Type: application/json;charset=UTF-8
>      Cache-Control: no-store
>      Pragma: no-cache
> 
>      {
>        "access_token":"2YotnFZFEjr1zCsicMWpAA",
>        "token_type":"example",
>        "expires_in":3600,
>        "example_parameter":"example_value"
>      }
> ```

上面代码中，各个参数的含义参见《授权码模式》一节。



第一步，A 应用在命令行向 B 发出请求。

> ```javascript
> https://oauth.b.com/token?
>   grant_type=client_credentials&
>   client_id=CLIENT_ID&
>   client_secret=CLIENT_SECRET
> ```

上面 URL 中，`grant_type`参数等于`client_credentials`表示采用凭证式，`client_id`和`client_secret`用来让 B 确认 A 的身份。

第二步，B 网站验证通过以后，直接返回令牌。

这种方式给出的令牌，是针对第三方应用的，而不是针对用户的，即有可能多个用户共享同一个令牌。



### 令牌的使用

A 网站拿到令牌以后，就可以向 B 网站的 API 请求数据了。

此时，每个发到 API 的请求，都必须带有令牌。具体做法是在请求的头信息，加上一个`Authorization`字段，令牌就放在这个字段里面。

> ```bash
> curl -H "Authorization: Bearer ACCESS_TOKEN" \
> "https://api.b.com"
> ```

上面命令中，`ACCESS_TOKEN`就是拿到的令牌。



### 更新令牌

如果用户访问的时候，客户端的"访问令牌"已经过期，则需要使用"更新令牌"申请一个新的访问令牌。

客户端发出更新令牌的HTTP请求，包含以下参数：

- granttype：表示使用的授权模式，此处的值固定为"refreshtoken"，必选项。
- refresh_token：表示早前收到的更新令牌，必选项。
- scope：表示申请的授权范围，不可以超出上一次申请的范围，如果省略该参数，则表示与上一次一致。

下面是一个例子。

> ```
>      POST /token HTTP/1.1
>      Host: server.example.com
>      Authorization: Basic czZCaGRSa3F0MzpnWDFmQmF0M2JW
>      Content-Type: application/x-www-form-urlencoded
> 
>      grant_type=refresh_token&refresh_token=tGzv3JOkF0XG5Qx2TlKWIA
> ```

令牌的有效期到了，如果让用户重新走一遍上面的流程，再申请一个新的令牌，很可能体验不好，而且也没有必要。OAuth 2.0 允许用户自动更新令牌。

具体方法是，B 网站颁发令牌的时候，一次性颁发两个令牌，一个用于获取数据，另一个用于获取新的令牌（refresh token 字段）。令牌到期前，用户使用 refresh token 发一个请求，去更新令牌。

> ```javascript
> https://b.com/oauth/token?
>   grant_type=refresh_token&
>   client_id=CLIENT_ID&
>   client_secret=CLIENT_SECRET&
>   refresh_token=REFRESH_TOKEN
> ```

上面 URL 中，`grant_type`参数为`refresh_token`表示要求更新令牌，`client_id`参数和`client_secret`参数用于确认身份，`refresh_token`参数就是用于更新令牌的令牌。

B 网站验证通过以后，就会颁发新的令牌。



##  三、GitHub OAuth 第三方登录示例教程

很多网站登录时，允许使用第三方网站的身份，这称为"第三方登录"。

很多网站登录时，允许使用第三方网站的身份，这称为"第三方登录"。

![img](image/oauth2-12.jpg)

下面就以 GitHub 为例，写一个最简单的应用，演示第三方登录。

### 一、第三方登录的原理

所谓第三方登录，实质就是 OAuth 授权。用户想要登录 A 网站，A 网站让用户提供第三方网站的数据，证明自己的身份。获取第三方网站的身份数据，就需要 OAuth 授权。

举例来说，A 网站允许 GitHub 登录，背后就是下面的流程。

> 1. A 网站让用户跳转到 GitHub。
> 2. GitHub 要求用户登录，然后询问"A 网站要求获得 xx 权限，你是否同意？"
> 3. 用户同意，GitHub 就会重定向回 A 网站，同时发回一个授权码。
> 4. A 网站使用授权码，向 GitHub 请求令牌。
> 5. GitHub 返回令牌.
> 6. A 网站使用令牌，向 GitHub 请求用户数据。

下面就是这个流程的代码实现。

### 二、应用登记

一个应用要求 OAuth 授权，必须先到对方网站登记，让对方知道是谁在请求。

所以，你要先去 GitHub 登记一下。当然，我已经登记过了，你使用我的登记信息也可以，但为了完整走一遍流程，还是建议大家自己登记。这是免费的。

访问这个[网址](https://github.com/settings/applications/new)，填写登记表。

![img](image/oauth2-13.jpg)

应用的名称随便填，主页 URL 填写`http://localhost:8080`，跳转网址填写 `http://localhost:8080/oauth/redirect`。

提交表单以后，GitHub 应该会返回客户端 ID（client ID）和客户端密钥（client secret），这就是应用的身份识别码。

### 三、示例仓库

我写了一个[代码仓库](https://github.com/ruanyf/node-oauth-demo)，请将它克隆到本地。

> ```bash
> $ git clone git@github.com:ruanyf/node-oauth-demo.git
> $ cd node-oauth-demo
> ```

两个配置项要改一下，写入上一步的身份识别码。

> - [`index.js`](https://github.com/ruanyf/node-oauth-demo/blob/master/index.js#L3)：改掉变量`clientID` and `clientSecret`
> - [`public/index.html`](https://github.com/ruanyf/node-oauth-demo/blob/master/public/index.html#L16)：改掉变量`client_id`

然后，安装依赖。

> ```bash
> $ npm install
> ```

启动服务。

> ```bash
> $ node index.js
> ```

浏览器访问`http://localhost:8080`，就可以看到这个示例了。

### 四、浏览器跳转 GitHub

示例的首页很简单，就是一个链接，让用户跳转到 GitHub。

![img](image/oauth2-14.jpg)

跳转的 URL 如下。

> ```markup
> https://github.com/login/oauth/authorize?
>   client_id=7e015d8ce32370079895&
>   redirect_uri=http://localhost:8080/oauth/redirect
> ```

这个 URL 指向 GitHub 的 OAuth 授权网址，带有两个参数：`client_id`告诉 GitHub 谁在请求，`redirect_uri`是稍后跳转回来的网址。

用户点击到了 GitHub，GitHub 会要求用户登录，确保是本人在操作。

### 五、授权码

登录后，GitHub 询问用户，该应用正在请求数据，你是否同意授权。

![](image/oauth2-15.png)

用户同意授权， GitHub 就会跳转到`redirect_uri`指定的跳转网址，并且带上授权码，跳转回来的 URL 就是下面的样子。

> ```markup
> http://localhost:8080/oauth/redirect?
>   code=859310e7cecc9196f4af
> ```

后端收到这个请求以后，就拿到了授权码（`code`参数）。

### 六、后端实现

示例的[后端](https://github.com/ruanyf/node-oauth-demo/blob/master/index.js)采用 Koa 框架编写，具体语法请看[教程](http://www.ruanyifeng.com/blog/2017/08/koa.html)。

这里的关键是针对`/oauth/redirect`的请求，编写一个[路由](https://github.com/ruanyf/node-oauth-demo/blob/master/index.js#L16)，完成 OAuth 认证。

> ```javascript
> const oauth = async ctx => {
>   // ...
> };
> 
> app.use(route.get('/oauth/redirect', oauth));
> ```

上面代码中，`oauth`函数就是路由的处理函数。下面的代码都写在这个函数里面。

路由函数的第一件事，是从 URL 取出授权码。

> ```javascript
> const requestToken = ctx.request.query.code;
> ```

### 七、令牌

后端使用这个授权码，向 GitHub 请求令牌。

> ```javascript
> const tokenResponse = await axios({
>   method: 'post',
>   url: 'https://github.com/login/oauth/access_token?' +
>     `client_id=${clientID}&` +
>     `client_secret=${clientSecret}&` +
>     `code=${requestToken}`,
>   headers: {
>     accept: 'application/json'
>   }
> });
> ```

上面代码中，GitHub 的令牌接口`https://github.com/login/oauth/access_token`需要提供三个参数。

> - `client_id`：客户端的 ID
> - `client_secret`：客户端的密钥
> - `code`：授权码

作为回应，GitHub 会返回一段 JSON 数据，里面包含了令牌`accessToken`。

> ```javascript
> const accessToken = tokenResponse.data.access_token;
> ```

### 八、API 数据

有了令牌以后，就可以向 API 请求数据了。

> ```javascript
> const result = await axios({
>   method: 'get',
>   url: `https://api.github.com/user`,
>   headers: {
>     accept: 'application/json',
>     Authorization: `token ${accessToken}`
>   }
> });
> ```

上面代码中，GitHub API 的地址是`https://api.github.com/user`，请求的时候必须在 HTTP 头信息里面带上令牌`Authorization: token 361507da`。

然后，就可以拿到用户数据，得到用户的身份。

> ```javascript
> const name = result.data.name;
> ctx.response.redirect(`/welcome.html?name=${name}`);
> ```