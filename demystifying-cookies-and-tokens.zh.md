# [揭开Cookies和Token的神秘面纱](https://tommihovi.com/2024/05/demystifying-cookies-and-tokens/)

最近我开始深入了解Token和Cookies的世界。其中有一个客户希望防止Token和Cookies被盗，这促使我深入挖掘更多信息。我敢打赌你们一定听说过这些术语，并且很可能每天都在使用它们。但是Token和Cookies有很多变体，并且我们需要理解它们的一些背景，以便看到更大的图景。以下是我到目前为止的所学，让我们开始吧！

## 设定目标

在这篇博客中，我们将理解什么是Web Cookies和Tokens，以及它们的用途。我们还将看看Web Cookies和Tokens的最常见类型。本文不会讨论如何防止Token或Cookie的滥用，这将是另一个主题。

## 什么是Cookies？

让我们首先尝试理解Web Cookies是什么，以及我们如何使用它们。本文中的一般术语Cookies指的是Web Cookie。

### Cookies的解释

**Cookies是小文本文件**，由你访问的网站（Web服务器）创建，并存储在你的设备上。我们都遇到过它们。实际上，你访问的几乎每个网站都会用一个Cookie横幅“问候”你。Cookie的目的取决于Cookie的类型以及网站希望Cookie做的事情。例如，一个在线商店可能会使用Cookies来跟踪用户在其网站上的活动，包括用户最后访问的页面、选择的语言、查看过的商品以及放入购物车的商品。当用户离开网站后再次返回时，网站可以读取Cookie，并例如在用户离开时继续他的位置。然而，并不是所有的Cookies都有相同的目的。

以下图片高层次地解释了Cookies是如何工作的。

![](https://scillidan.github.io/image_post/demystifying-cookies-and-tokens_01.webp)  
_图片1. Cookies如何形成的高层次定义_  
_来源: Cookieyes。检查图片左下角的源URL_

但是，为什么我们需要Cookies呢？**我们需要Cookies，因为HTTP是一种无状态协议。** HTTP不会记住客户端的会话信息，因此客户端负责存储这些信息——使用Cookie。对于某些网站，无状态行为可能是可以接受的，也许网站在用户会话期间没有需要保留的元素或用户操作。但对于大多数互动性网站，Cookies是必要且必不可少的，并且作为网站访问者，我们也希望网站以某种方式行为。多亏了Cookies，这一切变为可能。当客户端从Web服务器请求网站时，Web服务器会以**‘200 OK’**响应，并使用**Set-Cookie**头发送Cookie。Cookie包含会话ID，通常还包含其他属性。Web服务器还会跟踪服务器端给客户端发出的会话ID。

Cookies主要用于：

1. HTTP会话管理
2. 个性化
3. 跟踪

**Cookies存储在你的浏览器中。** 更确切地说，它们存储在浏览器的临时目录中。Cookies的具体存储位置和方法取决于浏览器和操作系统。你最常会通过进入设置的方式来管理Cookies，但不同操作系统和浏览器的文件夹路径在互联网上是可以找到的。例如，Microsoft Edge将Cookies存储在以下路径：

- **C:\\Users\\\[用户名\]\\AppData\\Local\\Microsoft\\Edge\\User Data\\Default\\Network**

![](https://scillidan.github.io/image_post/demystifying-cookies-and-tokens_02.webp)  
_图片2. 当浏览到文件路径时，应期望看到这样的文件_

通常，你可以使用浏览器界面管理、查看或删除Cookies。

![](https://scillidan.github.io/image_post/demystifying-cookies-and-tokens_03.webp)  
_图片3. Microsoft Edge浏览器设置中的Cookies_

#### Cookie属性

Cookies有一些属性（有时也称为标志），这些属性非常重要，因为它们定义了Cookie的工作方式。以下是Cookie可能具有的一些属性列表。

- _会话ID_是用于识别和匹配客户端与Web服务器之间会话的唯一随机字符串
- _Expires_定义了Cookie设置到期的日期
- _Domain_指定Cookie有效的域或多个域
- _Path_指定Cookie有效使用的资源或路径
- _HttpOnly_启用时将防止客户端API（例如JavaScript）访问Cookie。这可以减轻跨站脚本攻击（XSS）的威胁。
- _Secure_启用时将要求仅使用HTTPS发送Cookie，而不允许未加密的连接（如HTTP），这使得Cookie在一定程度上减少被盗。
- _Session_定义Cookie是一个临时Cookie，当浏览器关闭时将过期
- _SameSite_可以有值 **strict**、**lax**或**none**
	- Strict = 浏览器仅向与原始域相同的目标域发送Cookie
	- Lax = 浏览器即使是不同于原始域的目标域也会发送Cookie，但仅限于安全请求（如GET），而不包括第三方Cookies。
	- None = 允许第三方Cookies（跨站Cookies）

你可以通过右键单击并选择“_检查_” > “_应用_” > “_存储_” > “_Cookies_”来查看你正在浏览网站的Cookies。当你选择一行时，可以在页面底部查看值（_见图片5._）

![](https://scillidan.github.io/image_post/demystifying-cookies-and-tokens_04.webp)  
_图片4. 网站“检查”将让你查看该特定网站的Cookies以及用户接受的Cookie同意_

![](https://scillidan.github.io/image_post/demystifying-cookies-and-tokens_05.webp)  
_图片5. Cookie同意也很明显_

![](https://scillidan.github.io/image_post/demystifying-cookies-and-tokens_06.webp)  
_图片6. 打开Google Chrome浏览器后自动打开www.google.com网站并检查活动Cookies的结果！_

### Cookies的类型

以下是你在操作Cookies时可能会遇到的一些Cookie术语。Cookies可以用多种方式大致划分为类别。有第一方Cookies和第三方Cookies。有必要Cookies（严格必要）和非必要Cookies（非必要）。我们将在这里使用后者的定义来分类我们的Cookies。分类其实并不重要，只是人类更容易将东西放入心理类别中。另外，有些Cookies只是其他Cookies的变体。例如，你可能会听到“安全Cookie”，但实际上那可能只是一个启用了_secure_-属性的会话或第一方Cookies。即便如此，会话Cookies和第一方Cookies可能非常相似。尽管我的观点是术语相当模糊，并且不同网站使用不同的Cookie术语可以互换使用，然而我们仍然要至少尝试给出每种术语某种定义。

#### 必要Cookies

必要Cookies是网站正常工作所必需的，并且通常使用户的浏览体验更方便。

- 会话Cookies
- 第一方Cookies
- 身份验证Cookies
- 用户中心安全Cookies
- 用户输入Cookies

##### 会话Cookies _(亦称为非持久Cookies、内存Cookies、临时Cookies)_

**会话Cookies是临时的Cookie文件**，在用户关闭浏览器或会话变得不活跃时被删除（如果用户注销会话将结束）。它们是单个会话Cookies。当用户重新启动浏览器并返回创建该Cookie的网站时，网站将无法识别用户，因为没有Cookie可供网站在用户的浏览器中读取。用户将需要重新登录（如果该网站需要登录）。在登录后，将生成一个新的会话Cookie，用于存储用户的浏览信息，直到用户离开网站并关闭浏览器。由于会话Cookie拥有自己的唯一ID，它还可以用于跟踪网站访客数量。如果你计划进行一次度假旅行，并且一天多次访问一家旅行社的网站，那么Cookie的会话ID将向网站表明你只是一个唯一的访客。

##### 第一方Cookies _(亦称为持久Cookies、永久Cookies、存储Cookies)_

**第一方Cookies将保持** 在浏览器中，**直到用户删除它们或浏览器基于持久Cookie文件中的持续时间删除它们**。这种类型的Cookie可以被称为其他术语，如持久Cookie、永久Cookie或存储Cookie，从而在单个会话中具有持久性。大多数第一方Cookies将在网站在该期间未访问时在1-2年内过期。当Cookie过期时，浏览器将自动删除它。第一方Cookie最常用的功能是让用户保持登录到网站，从而避免每次访问网站时重新输入凭据的繁琐。

##### 身份验证Cookies

**身份验证Cookies** 是会话Cookies的一种变体。它们**会在成功登录后识别用户**，并在用户在网站上浏览时携带该认证信息，用户浏览的内容经过授权。例如，当用户认证到在线银行时，他们被授权查看自己的银行账户余额。如果他们转到另一个页面以查看他们的贷款，身份验证Cookies会确保用户不必为该页面提供新的身份验证。

##### 用户中心安全Cookies

**用户中心安全Cookies** 可以跟踪身份验证错误，并通过记录在登录页面尝试输入错误登录凭证的次数来**检测可能的身份验证滥用**。

##### 用户输入Cookies

**用户输入Cookies** 是第一方会话Cookies，**用于跟踪用户自己在网站上输入的操作和项**，例如填写表单或点击一些按钮（如添加到购物车）。

#### 非必要Cookies

非必要Cookies对用户的浏览体验或网站的正常功能并不是必需的。

- 分析和自定义Cookies
- 广告Cookies
- 第三方Cookies
- 超级Cookies

##### 分析和自定义Cookies

分析和自定义Cookies跟踪用户活动，以便网站管理员更好地理解他们的网站被如何使用。网站管理员可以使用的某些分析信息来提升网站的例子是网站上的某些信息可能非常“冷”，这意味着用户很少打开该页面（不感兴趣）或无法找到它。

##### 广告Cookies

广告Cookies仅用于根据用户的浏览历史自定义用户在网页上看到的广告，以使广告“更相关”。这就是为什么我们开始看到与我们以前搜索的内容相关的广告。

##### 第三方Cookies _(亦称为跟踪Cookies)_

**第三方Cookies是“Cookies国度”的坏家伙**。这些是我们不喜欢的Cookies，也是Cookies声誉如此糟糕的原因。第三方Cookies来自用户正在访问的不同域，因此它们没有提供会话Cookies或第一方Cookies的任何好处。**这些Cookies的唯一目的就是跟踪你**。

##### 超级Cookies

超级Cookies不算真正的Cookies，并且在某些情况下并不存储在用户的设备上。正常的Cookie可以存储最多4 KB的数据，而超级Cookies可以存储100 KB。超级Cookies与跟踪Cookies的功能相似，但它们的控制方式与其他Cookies不同。在某些情况下，发现超级Cookies存储在浏览器缓存中，而某些形式的超级Cookies则注入到ISP仅控制的UIDH（唯一标识符头）中。由于超级Cookies的性质，它们更难识别，且在UIDH超级Cookies中，你无法清除它们。

___

## 什么是Tokens？

Tokens是自包含和紧凑的JSON对象，用于信息交换。典型的Token是JSON Web Token（JWT），用于客户端（应用程序）与其他服务（如Web服务器）之间。具体的细节取决于确切的身份验证流程。在本文中，我们将使用术语JWT（发音为‘JOT’），因为它在专业文献中更方便且更广泛使用。

### Tokens的解释

与存储用户在Web会话中的短期或长期活动信息的Cookies不同，Tokens在软件之间传递信息。存储在Token中的信息取决于Token的类型。例如，ID Token携带经过身份验证用户的信息。访问Token通常包含有关安全主体及其被授权访问的内容的信息。**这就是Tokens和Cookies交集的地方**：某些Tokens会存储在客户端，或者在称为本地存储的地方，或者在Cookie中。出于安全原因，最好是存储在HttpOnly Cookie中。我不是Web开发人员，但我了解到开发者社区倾向于使用Cookies，因为本地存储被视为较不安全的选项。此外，值得一提的是，并非所有Tokens都是同一协议的一部分。

我将在此提及并简短解释的协议是**OAuth 2.0**和**OpenID Connect**（OIDC），因为它们是由**Microsoft身份平台**使用的。

**OAuth**是开放授权的缩写，它是一种标准，旨在处理应用程序或网站代表用户访问资源的授权流程。简单来说，OAuth是一种主要用于Web平台的授权协议。OAuth使用访问Tokens，通常，但不限于，JSON Web Tokens。Microsoft身份平台中的OAuth协议使用格式为JWT的访问Tokens（持有者Tokens）。OAuth 2.0授权流程有多种不同类型。

**OpenID Connect**扩展了OAuth 2.0授权协议，使其也作为一种身份验证协议，使用ID Token。OpenID Connect登录流程用于获取ID Token，该Token被发送到验证用户身份的应用程序。

![](https://scillidan.github.io/image_post/demystifying-cookies-and-tokens_07.webp)  
_OpenID Connect登录流程。来源: Microsoft Learn_

![](https://scillidan.github.io/image_post/demystifying-cookies-and-tokens_08.webp)  
_登录流程后，通过OAuth授权流程获取访问Token。来源: Microsoft Learn_

### Tokens解决的问题

让我们描述一个传统的身份验证和授权场景，其中Tokens不存在。用户首先登录，Web服务器通过检查其数据库中的凭据来对用户进行身份验证。一旦凭据匹配，服务器就会发出唯一的会话标识符，并将其发送到客户端。用户的客户端在设备上存储会话ID。对于客户端对服务器的每个后续请求，会话ID会通过Cookie或HTTP请求头发送。服务器需要再次从其数据库中查找会话ID，以识别用户并检查授权级别。

上述场景的问题在于，服务器需要对每个来自客户端的请求与数据库进行往返查找，这使得应用程序使用起来更慢。

而使用Tokens的场景通过发出一个JWT来解决了后续数据库查找的问题，该JWT包含用户信息和验证Token内容真实性的签名。JWT被发送到客户端，客户端再次存储它。对于客户端向服务器的每个请求，客户端包括JWT。服务器只需验证Token的签名以确保其真实性，并在该检查通过后，从Token中提取身份和授权详细信息，而无需数据库查找。

### Tokens的类型

以下是一些常见的Token类型：

- ID Token
- 访问Token
- 刷新Token
- 持有者Token

接下来，我将简单介绍这些Tokens和术语。

##### ID Token

ID Tokens是通过OpenID Connect（OIDC）登录流程获得的，这是一种去中心化的身份验证开放标准。ID Tokens要求采用JSON Web Token格式。它们由授权服务器（Microsoft Entra ID）发放给客户端应用程序。**ID Token包含有关用户**或安全主体的声明，也可以包含有关多因素身份验证状态的声明等信息。客户端应用程序可以使用ID Token中的信息和声明来验证用户是否如他们所称。默认情况下，Microsoft身份平台的ID Tokens有效期为一个小时。**ID Tokens不用于调用受保护的API。**

Microsoft有两个版本的ID Tokens，这两个版本具有不同的端点。版本之间的区别在于它们可以包含的信息和声明。

- v1.0 **https://login.microsoftonline.com/common/oauth2/authorize**
- v2.0 **https://login.microsoftonline.com/common/oauth2/v2.0/authorize**

![](https://scillidan.github.io/image_post/demystifying-cookies-and-tokens_09.webp)

_用户正在登录到Web应用程序，该应用程序会将用户重定向到Entra ID以完成身份验证。一旦验证成功，Entra ID将发出ID Token并将其发送给客户端（浏览器）。浏览器不会尝试理解ID Token，而只是将其发送到Web服务器。_

##### 访问Token

**访问Token** 是由授权服务器签名的短期（JWT）Token，**使客户端能够安全地调用受保护的Web API**。访问Tokens不需要以JSON Web Tokens格式，但在Microsoft身份平台中采用JWT格式。访问Tokens包含在客户端到服务器的每个HTTP请求中。通常通过HTTP请求中的授权头使用“Bearer”模式进行。这么做可以避免服务器在每个请求中对用户进行身份验证。客户端要么将访问Token存储在Cookie中，要么存储在本地存储中。与ID Tokens一样，访问Tokens也有两个版本，这两个版本都有不同的端点。访问Tokens旨在由服务器端消耗。客户端不需要并且不应使用访问Token读取其内容。

##### 刷新Token

**刷新Token通常与访问Token一起获得。** 它用于在以前的访问Tokens过期时获取新的访问Tokens，并且也会获取新的刷新Token。在Microsoft身份平台中，刷新Tokens是加密的，其他人无法读取。客户端将访问Token存储在Cookie中，或者存储在本地存储中。刷新Token的默认生命周期为90天，单页应用程序为例，其默认生命周期为24小时。

##### 持有者Token

持有者Token是指可以被任何拥有Token的人使用的Token。Token持有者不需要证明自己拥有加密密钥。可以把这想象成一张酒店房间的钥匙卡。如果你丢失了钥匙卡，其他人得知它的所在，即可使用它进入酒店房间，而不需要证明他们是房间的合法主人。

## JSON Web Token (JWT)

JSON Web Tokens是一种标准化的对象，用于安全地在两方之间发送数据。为了确保JWT的内容在传输过程中没有被篡改或修改，JWT使用加密密钥进行签名。这一点需要重复强调：**JWT是签名的，而不是加密的**。签名验证数据的发送者，而加密通过将数据从明文转换为密文来防止未授权的接收者读取。如果在传输过程中签名的Token内容被更改，公钥（与用于签署Token的私钥配对）将无法再验证签名。因此，强烈建议使用HTTPS等协议与JWT一起使用，以保护Token内容在传输过程中的机密性。

互联网工程任务组（IETF）在[RFC 7519](https://datatracker.ietf.org/doc/html/rfc7519)中描述了JWT标准。

JSON Web Token由三个部分组成：

- 头部
- 有效载荷
- 签名

#### 头部

头部通常由两个部分组成，描述**Token的类型**和**签名算法**。

```
{
	"alg": "HS256",
	"typ": "JWT"
}
```

#### 有效载荷

有效载荷是包含所有实际信息的部分。例如**声明**。在RFC 7519的第4章中，注册声明被定义如下：

- **“iss”** = 发行人声明。发行人声明标识发放JWT的主体
- **“sub”** = 主体声明。主体声明标识JWT的主题
- **“aud”** = 受众声明。受众声明标识JWT的预期接收者
- **“exp”** = 过期时间声明。过期声明标识JWT自该时间之后不可被接受处理
- **“nbf”** = 不可在此之前接受声明。NFB声明标识JWT在此之前不可被接受处理
- **“iat”** = 签发声明。签发声明标识JWT签发的时间
- **“jti”** = JWT ID 声明。JWT ID声明为JWT提供了唯一标识符

该标准不强制使用注册声明，因此它们是可选的。其他类型的声明有**公共声明名称**和 **私人声明名称**。

```
{
	"session": "ch72gsb320000udocl363eofy",
	"client_id": "YzEzMGdoMHJnOHBiOG1ibDhyNTA=",
	"response_type": "code",
	"scope": "introscpect_tokens, revoke_tokens",
	"iss": "bjhIRjM1cXpaa21zdWtISnp6ejlMbk44bTlNZjk3dXE=",
	"sub": "YzEzMGdoMHJnOHBiOG1ibDhyNTA=",
	"name": "John Doe",
	"aud": "https://login.microsoftonline.com/{tid}/oauth2/v2.0/authorize",
	"jti": "1516239022",
	"exp": "2024-05-17T07:09:48.000+0545"
}
```

#### 签名

头部和有效载荷都被**Base64Url**编码。为了签署Token，使用编码后的头部和编码后的有效载荷以及加密密钥和签名算法来完成签名。

```
HMACSHA256(
	base64UrlEncode(header) + "." +
	base64UrlEncode(payload),
	secret)
```

以下截图显示了左侧的已签名JWT，以及右侧的头部、有效载荷和签名。

![](https://scillidan.github.io/image_post/demystifying-cookies-and-tokens_10.webp)  
_你可以在已签名JWT中看到所有三个部分，其中不同部分用句点(.)分开_

### 示例Token

查看JWT的最简单方法是访问Microsoft Graph Explorer [https://developer.microsoft.com/en-us/graph/graph-explorer](https://developer.microsoft.com/en-us/graph/graph-explorer)并登录。登录后，我们运行“**获取我的个人资料**”查询并打开**访问Token**选项卡，复制Token。

![](https://scillidan.github.io/image_post/demystifying-cookies-and-tokens_11.webp)  
_部分访问Token出于安全卫生原因已被屏蔽_

然后，我们进入 [https://jwt.ms](https://jwt.ms)，并粘贴Token。

![](https://scillidan.github.io/image_post/demystifying-cookies-and-tokens_12.webp)  
__部分访问Token出于安全卫生原因已被屏蔽__

在我们粘贴的Token下面是解码的Token内容，以及“声明”选项卡，解释了声明缩写，如“oid”、“uti”等。从下面的Token中，我们可以推断出以下内容：

- **“typ”: “JWT”** 表明Token的类型是JSON Web Token
- **“alg”: “RS256”** 意味着该Token的签名使用RSA签名和SHA-256算法
- **“aud”: “00000003-0000-0000-c000-000000000000”** 表示Token的接收者（_受众_）是**Microsoft Graph**
- **“iss”: “https://sts.windows.net/f77b793c-9409-XXXX-XXXX-b793feaadb48/”** 表示该Token的发行者是Azure Active Directory v1端点
- **“appid”: “de8bc8b5-d9f9-48b1-a8ad-b748da725064”** 意味着使用该Token的客户端是**Graph Explorer**，因为它使用该Token调用Microsoft Graph API
- **“scp”: “openid profile User.Read email”** 列出了应用程序（Graph Explorer）请求的API作用域以及同意的内容和Microsoft Graph已公开的内容
- **“ver”: “1.0”** 表示Token版本

![](https://scillidan.github.io/image_post/demystifying-cookies-and-tokens_13.webp)

```
**小贴士！** 我注意到我获取的所有Token的发行者都是**sts.windows.net**。我原以为它应该是**login.microsoftonline.com**。快速研究指向**accessTokenAcceptedVersion**参数，该参数似乎控制Token发行端点。稍后需要更多测试来确认，但目前似乎是这样工作的：  
值 **null** 默认情况下为版本1.0  
值 **1** 同样也是版本1.0（sts.windows.net）  
值 **2** 表示版本2.0（login.microsoftonline.com）
```

![](https://scillidan.github.io/image_post/demystifying-cookies-and-tokens_14.webp)  
_应用程序注册清单配置_

### JWT的安全隐患

像任何技术解决方案一样，JSON Web Token也有其缺点。

##### 大小限制和存储

某些复杂应用程序需要在Token中存储大量信息。当Tokens存储在Cookies中时，这可能会增加每个请求的开销，甚至超过Cookie的允许大小（4 KB）。当发生这种情况时，常见的变通方法是将JWT存储在**本地存储**中，这具有其自身的安全问题，主要是本地存储中的数据可供该页面内的任何脚本访问。这可能导致跨站脚本（XSS）攻击能够访问Token。

一般来说，Cookies是用于服务器读取的，而本地存储只能由浏览器读取。这意味着Cookie的数据量受到比本地存储更小的限制。当Token存储在本地存储中时，浏览器使用JavaScript进行访问。这是为什么存储在本地存储中的Tokens容易受到跨站脚本攻击的根本原因，因为攻击者可以将恶意JavaScript注入网页并窃取访问Token。本地存储没有像Cookies那样的安全属性来保护Tokens。这不是说，如果Token存储在未妥善保护的Cookie中，那么我们也会面临XSS攻击的风险。但是Cookies具有机制来保护Tokens免受这些类型的攻击，这就是为什么OWASP社区也推荐使用Cookies存储Tokens的原因。

##### 使Token失效

如前所述，Tokens是自包含的。没有中央权威来跟踪和管理Tokens。在尝试使Token失效时，这成为一个挑战。默认情况下，访问Token的生命周期为一个（1）小时，Token中的过期细节包含在Token中。

##### 未加密

请记住，Tokens是签名的，但未加密的。因此，如果未充分通过HTTPS保护Token的内容，Token的内容可能会暴露给未授权方。

## 总结

我们讨论了什么是Cookies，以及浏览器和网站如何使用Cookies来增强浏览体验。我们列举了一些Cookies的变化和它们的独特特征。我们还讨论了什么是Tokens以及在Microsoft身份平台中用于获取Tokens的协议，以及每种类型的Token在何种情况下相关。有了这些关于Cookies和Tokens的基础知识，我们就能继续在后续的博客文章中探讨如何妥协Cookies或Tokens并滥用它们。稍后，我们还将讨论如何防止Cookie被窃取和Token被盗！

保持安全！

[GPT-4o mini]
