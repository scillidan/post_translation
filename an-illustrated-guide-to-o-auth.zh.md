# [OAuth的图解指南](https://www.ducktyped.org/p/an-illustrated-guide-to-oauth)

![访问令牌图像](https://scillidan.github.io/image_post/an-illustrated-guide-to-o-auth_01.webp)

OAuth首次于2007年推出。它是在Twitter创建的，因为Twitter希望为第三方应用程序提供一种让用户能够代表他们发布推文的方法。花点时间想想，今天如果要设计这样的东西，你会怎么做？一种方法就是要求用户提供他们的用户名和密码。因此，你创建了一个非官方的Twitter客户端，向用户展示一个登录屏幕，上面写着“使用Twitter登录”。用户确实这么做了，但他们实际上是将他们的数据发送给你，这个第三方服务，它为他们登录Twitter。

![坏方法 - 将密码交给第三方](https://scillidan.github.io/image_post/an-illustrated-guide-to-o-auth_02.webp)

这因为很多原因都是不好的。即使你信任一个第三方应用，但如果他们没有正确存储你的密码，导致某人窃取了它呢？你绝不应该将密码交给这样的第三方网站。

你可能在想，另一种方法是什么，API密钥呢？因为你正在访问Twitter的API来为用户发布数据，而对于一个_ API_，你使用_ API密钥_。但API密钥是通用的。你需要的是一个特定于用户的API密钥。

![好方法 - OAuth令牌](https://scillidan.github.io/image_post/an-illustrated-guide-to-o-auth_03.webp)

为了解决这些问题，OAuth应运而生。你将看到它是如何解决所有这些问题的，但OAuth的核心是**访问令牌**，它有点像特定用户的API密钥。应用程序获取一个访问令牌，然后可以使用它在用户的名义下采取行动或访问用户的数据。

![一些YNAB分类的图示](https://scillidan.github.io/image_post/an-illustrated-guide-to-o-auth_04.webp)

## OAuth的工作原理

OAuth可以以许多不同的方式使用，这也是它难以理解的原因之一。在这篇文章中，我们将查看一个典型的OAuth流程。

我将使用的例子是YNAB。如果你没用过，YNAB就像是Mint的付费版本。你将它连接到一个银行账户，然后它从该账户中提取所有交易，以非常漂亮的图表显示给你。你可以对你的支出进行分类，然后它可以告诉你，例如，嘿，你在杂货上的花费太多了。它帮助你管理你的财务。所以，我想使用YNAB，并希望将其连接到Chase Bank，但我不想给它我Chase的密码。所以我将使用OAuth。

首先，让我们看看流程，然后理解发生了什么。我们实际将查看两次OAuth流程，因为我认为你至少需要观看一次OAuth流程两次才能理解发生了什么。

## OAuth流程，第一次

所以，首先，我在YNAB，我想连接Chase作为来源。OAuth流程如下：

1. YNAB将我重定向到Chase。
2. 在Chase，我用我的用户名和密码登录。
3. Chase向我显示一个屏幕，说明“YNAB想连接到Chase。选择你希望给予YNAB访问权的账户。”它将显示我所有账户的列表。假设我只选择我的支票账户，以便给予YNAB该账户的读取权限，然后点击确认。
4. 从Chase，我被重定向回YNAB，现在神奇的是，YNAB与Chase连接了。

![流程图示](https://scillidan.github.io/image_post/an-illustrated-guide-to-o-auth_05.webp)

这是从用户的角度来看。但那时发生了什么？在后台发生了什么魔法，使得YNAB以某种方式可以访问我在Chase的数据？

## 最终目标是得到一个访问令牌

记住，_OAuth的最终目标是让YNAB得到一个访问令牌_，以便它可以访问我的Chase数据。在我经历这个流程时，YNAB以某种方式得到了一个访问令牌。我将通过告诉你_它_是如何得到访问令牌的来揭露这个惊喜，然后详细讲述发生了什么。

## 关于安全的简要说明

Chase是如何给YNAB访问令牌的？当你从Chase被重定向回YNAB时，Chase本可以直接在URL中添加访问令牌。它可以将你重定向回一个这样的URL：

```
https://www.ynab.com/redirect?access_token=123
```

然后YNAB就能够获取访问令牌。

**坏主意！！**

访问令牌应该是机密的，但URL可能会出现在浏览器的历史记录或某些服务器日志中，这样任何人都可以轻易看到你的访问令牌。

所以Chase_从技术上_可以将你重定向回YNAB，_将访问令牌_放在_URL_中，然后YNAB将拥有访问令牌。OAuth流程结束了。但我们不会这样做，因为**将访问令牌发送在URL中是不安全的。**

当你被重定向回YNAB时，Chase向YNAB发送一个_授权代码_在URL中。

授权代码不是访问令牌！Chase发送给YNAB一个授权代码，YNAB_用这个授权代码换取访问令牌_。它通过向Chase发起一次后台请求，一个后台的HTTPS POST请求来实现，这意味着没有人可以看到访问令牌。

![](https://scillidan.github.io/image_post/an-illustrated-guide-to-o-auth_06.webp)

然后YNAB就得到了访问令牌。OAuth流程结束。OAuth成功。

## OAuth的两个部分

![](https://scillidan.github.io/image_post/an-illustrated-guide-to-o-auth_07.webp)

让我们谈谈我们刚刚看到的内容。从高层次来看，OAuth流程有两个部分。第一部分是**用户同意流程**，即你作为用户登录并选择授予访问权限的内容。这是OAuth中的一个关键部分，因为在OAuth中，我们始终希望用户积极参与并控制。

另一部分是**授权代码流程**。这是YNAB实际获取这个**访问令牌**的流程。

让我们详细谈谈具体的工作原理。我们还需要讨论一些术语，因为OAuth有非常具体的术语。

- 我们说**资源拥有者**代替用户。
- 我们说**OAuth客户端**或**OAuth应用**代替应用。
- 你登录的服务器称为**授权服务器**。你从中获取用户数据的服务器称为**资源服务器**（这可以与授权服务器相同）。
- 在授权服务器上，当用户选择允许的内容时，这些被称为**范围**。

![](https://scillidan.github.io/image_post/an-illustrated-guide-to-o-auth_08.webp)

我会尽量使用这些术语，因为如果你要阅读更多关于OAuth的文档，你需要熟悉这些术语。

所以，让我们再次从高层次看一下，使用新术语。

## OAuth流程，第二次

你有OAuth客户端。一个OAuth客户端希望访问一个资源服务器上的数据，而这些数据属于资源拥有者。

![](https://scillidan.github.io/image_post/an-illustrated-guide-to-o-auth_09.webp)

为此，OAuth客户端重定向到授权服务器。用户登录，用户同意**范围**（这个令牌被允许访问的内容），然后用户被重定向回OAuth客户端，URL中带有授权代码。

![](https://scillidan.github.io/image_post/an-illustrated-guide-to-o-auth_10.webp)

在后台，OAuth客户端将授权代码和客户端秘钥（我们稍后会谈到客户端秘钥）发送到授权服务器，授权服务器将响应访问令牌。

![](https://scillidan.github.io/image_post/an-illustrated-guide-to-o-auth_11.webp)

这就是完全相同的流程，但使用了我们刚刚讨论的新术语。现在让我们谈谈具体细节。我们已经看到了这个流程从用户的角度看是什么样子，让我们看看从开发者的角度是什么样子。

## 注册一个新应用

![](https://scillidan.github.io/image_post/an-illustrated-guide-to-o-auth_12.webp)

要使用OAuth，你首先需要注册一个新应用。例如，GitHub提供OAuth。如果你想为GitHub创建一个新应用，你必须先注册它。不同的服务在应用注册中要求不同类型的数据，但每个服务至少需要：

1. 一个应用名称，因为当用户访问GitHub时，例如，GitHub需要能够说“Amazon Web Services正在请求对您的代码库和代码片段的读取权限。”
2. 一个重定向URI。我们稍后会谈到重定向URI。

GitHub将响应：

1. 一个客户端ID。这是一个公共ID，你将用它来发出请求。
2. 一个客户端秘钥。你将用这个来验证你的请求。

太棒了，你已经注册了你的OAuth应用。假设你的应用是YNAB，并且你的一个用户想要连接到Chase。所以你开始一个新的OAuth流程...你的第一个OAuth流程！

第一步：你将重定向他们到Chase的授权服务器的OAuth端点，在URL中传递这些参数：

![](https://scillidan.github.io/image_post/an-illustrated-guide-to-o-auth_13.webp)

1. 客户端ID，我们刚才讨论过的。
2. 重定向URI。一旦用户在Chase结束，这将是Chase将重定向他们返回的地址。这将是一个YNAB的URL，因为你是YNAB应用。
3. 响应类型，通常为“code”，因为我们通常想要获得授权代码，而不是访问令牌，后者不太安全。
4. 范围。那么我们请求的范围是什么？即我们希望访问什么用户数据？

这些信息足以让授权服务器验证请求并向用户显示一条消息，如“YNAB正在请求对您账户的读取权限”。

授权服务器如何验证请求？如果客户端ID无效，请求将立即无效。如果客户端ID有效，授权服务器需要检查重定向URI。基本上，由于客户端ID是公开的，任何人都可以获取YNAB的客户端ID，并创建自己的OAuth流程以访问Chase，但然后将用户返回到，比如说，evildude.com。但这就是为什么在你注册应用时，你需要告诉Chase有效的重定向URI是什么。在那时，你会告诉Chase，只有YNAB.com的URI是有效的，从而防止这种evildude.com的场景。

如果所有内容都有效，授权服务器可以使用客户端ID来获取应用名称，也许还有应用图标，然后显示用户同意屏幕。

用户将点击他们想要给予YNAB访问权限的账户，然后点击确认。

Chase将重定向他们回到你所给出的重定向URI，例如ynab.com/oauth-callback?authorization\_code=xyz。

> 旁注：你可能会想知道，URI和URL之间的区别是什么？因为我在这两者之间有点混用。URL是我们熟知和喜爱的任何网站URL。URI则更一般。URL是URI的一种类型，但还有许多其他类型的URI。

![](https://scillidan.github.io/image_post/an-illustrated-guide-to-o-auth_14.webp)

> 我之所以说重定向URI而不是重定向URL，是因为移动应用不会有URL。它们只会有URI，这可能是他们创建的一种协议，看起来像`myapp://foobar`。所以如果你只做网页工作，每当你看到URI时，你可以将其解读为URL。而如果你在做移动工作，你可以理解URI，也知道是的，你的用例也得到支持。

所以用户被重定向回ynab.com/oauth-callback?authorization\_code=xyz，现在你的应用有了一个授权代码。你将该授权代码发送到Chase授权服务器，连同你的客户端秘钥。为什么要包括客户端秘钥？因为同样，授权代码在URL中。所以任何人都可以看到它，任何人都可以尝试用它交换访问令牌。这就是为什么我们需要发送客户端秘钥，以便Chase的服务器可以说：“哦，是的，我记得我为这个客户端ID生成了这个代码，并且客户端秘钥匹配。这是一个有效请求。”

然后它返回访问令牌。注意，在OAuth流程的每一步中，他们都考虑到有人可能会利用这个流程，并添加了安全措施。这是一个大原因，为什么它如此复杂。

_*我可靠地被一个安全专家朋友告知，OAuth设计者在这方面经过了很多艰辛教训，这也是它如此复杂的另一个原因：因为它需要反复修补。_

另一个大原因是因为我们希望用户参与。这使得它复杂，因为所有用户相关的内容都必须在前端，而那是不安全的，因为任何人都可以看到它。而所有安全的东西都必须在后端。

我一直在说前端和后端，但在OAuth文档中，他们使用前通道和后通道这两个术语。让我们谈谈原因。

![](https://scillidan.github.io/image_post/an-illustrated-guide-to-o-auth_15.webp)

## 前通道和后通道

因此，OAuth不使用前端和后端这两个术语，它使用前通道和后通道。前通道指的是GET请求，任何人都可以在URL中看到参数，后通道则指的是POST请求，那些数据是加密的（作为POST主体的一部分）。OAuth为什么不使用前端或后端，因为你可以使用JavaScript进行POST请求！所以，理论上，你可以直接在前端用JavaScript将授权代码交换为访问令牌，方法是发起一个POST的fetch请求。

现在，这里有一个大问题，即你还需要客户端秘钥来发出该请求。当然，一旦秘钥在前端并可以被JavaScript访问，它就不再是秘密了。任何人都可以访问它。因此，如果后端不是一个选项，你可以使用一种不同的方式称为**PKCE**，拼写为P-K-C-E，发音为“pixie”（说真的）。它的安全性不如在后端使用客户端秘钥，但如果后端不是一个选项，你仍然可以使用PKCE。所以请知道，如果你有一个没有后端的应用，你仍然可以做OAuth。

> 我可能会在未来的帖子中覆盖PKCE，因为现在推荐标准流程，因为它帮助防止授权代码被拦截。

移动应用也是同样的问题。除非你有一个带有后端组件的移动应用，例如，某个地方的后端服务器，如果你把客户端秘钥放在移动应用中，任何人都可以获取它，因为有很多工具用于从移动应用中提取字符串。因此，而不是在你的应用中包含客户端秘钥，你应该再次使用PKCE来获取访问令牌。

所以，另外两个值得知道的术语是：**前通道**和**后通道**。

![](https://scillidan.github.io/image_post/an-illustrated-guide-to-o-auth_16.webp)

到这时，你已经看到了OAuth流程从用户的角度和开发者的角度，以及你已经了解到使其安全的组成部分。

## 一些最后的想法

最后我想提到的一点是，OAuth可能看起来有许多不同的方式。我在上面涵盖了主要推荐的OAuth流程，但有些人可能会通过在重定向中传回一个访问令牌而不是授权令牌来进行OAuth（这样做称为“隐式流程”）。有些人可能会使用PKCE进行操作。甚至还有一种方法可以在没有用户同意部分的情况下进行OAuth，但这实际上并不推荐。

我们没有涵盖OAuth的另一个部分是，令牌会过期，你需要刷新它们。这通过刷新流程来完成。此外，OAuth完全是关于授权，但一些工作流使用OAuth进行登录，例如，当你使用“Google登录”功能时。这使用的是OpenID Connect，或OIDC，这是OAuth之上的一层，它还返回用户数据，而不仅仅是访问令牌。我在这里提到这一点，因为当你在网上查找OAuth时，你会看到许多不同的流程，而你可能会困惑为什么它们都不同。原因是，OAuth并不像HTTP那样简单，OAuth可以有很多不同的外观。

现在你准备好去进行自己的OAuth操作了。祝你好运！

![](https://scillidan.github.io/image_post/an-illustrated-guide-to-o-auth_17.webp)

[GPT-4o mini]
