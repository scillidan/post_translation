# [介绍PPResume PWA](https://blog.ppresume.com/posts/introducing-ppresume-pwa)

## 介绍

我很高兴地宣布，[PPResume](https://ppresume.com/?utm_source=ppresume&utm_medium=blog&utm_campaign=introducing-ppresume-pwa)现已成为[PWA](https://en.wikipedia.org/wiki/Progressive_web_app)。

这是几周辛勤工作的成果。从现在开始，您可以将PPResume作为独立应用程序安装在设备上，享受更好的性能、更低的网络延迟和带宽成本，并在弱网或离线网络条件下获得出色的用户体验。

让我们先简单看看：

![PPResume PWA安装提示](https://scillidan.github.io/image_post/introducing-ppresume-pwa_01.webp)  
![PPResume PWA macOS应用搜索](https://scillidan.github.io/image_post/introducing-ppresume-pwa_02.webp)  
![PPResume PWA macOS应用切换](https://scillidan.github.io/image_post/introducing-ppresume-pwa_03.webp)  
![PPResume PWA macOS窗口](https://scillidan.github.io/image_post/introducing-ppresume-pwa_04.webp)  
![PPResume PWA iOS](https://scillidan.github.io/image_post/introducing-ppresume-pwa_05.webp)

## 什么是PWA？

> 一台iPod、一部手机、一台互联网移动通信器……这不是三种不同的设备！我们称之为iPhone！
> 
> — 史蒂夫·乔布斯，MacWorld 2007

[iPhone的诞生](https://www.youtube.com/watch?v=VQKMoT-6XSg)开启了移动计算的新纪元。当2007年iPhone 1发布时，它配备了3.5英寸的显示屏，并有Safari的移动版本用于上网。而在当时，网络是向iPhone提供应用的唯一方式。正如你所见，那时大多数网站都是为桌面设备设计的——只有少部分[wap](https://en.wikipedia.org/wiki/Wireless_Application_Protocol)网站是为手机设计的——所以您必须用手指捏合以放大或缩小，从而浏览手机上的网站。

![iPhone的诞生，MacWorld 2007](https://scillidan.github.io/image_post/introducing-ppresume-pwa_06.webp)  

不必多说，iPhone是一个革命性的移动设备，它是如此成功，以至于移动优先设计随后成为一种趋势，因为越来越多的流量来自移动设备。

在2007年的[WWDC](https://www.youtube.com/watch?v=ubm2dYzoDW8)上，史蒂夫·乔布斯宣布iPhone将“运行以Web 2.0互联网标准创建的应用程序”。不需要SDK，也没有App Store。所有应用程序都只是为移动设备设计的网站，通过Safari浏览器引擎提供。

然而，当时的网络标准太弱，无法提供良好和一致的用户体验——[HTML5](https://www.w3.org/TR/2011/WD-html5-20110405/)在2008年1月首次向公众发布，并在2014年10月获得W3C推荐状态，历时6年！人们对网络应用感到不满，开发者也感到沮丧，苹果不得不改变其战略。在2007年10月，史蒂夫·乔布斯宣布将于次年发布SDK。2008年7月，苹果发布了[iPhone OS 2.0](https://en.wikipedia.org/wiki/IPhone_OS_2)，并推出了[App Store](https://en.wikipedia.org/wiki/App_Store_(Apple))。从此，绝大多数iOS应用程序转向了App Store。

App Store被证明是一个巨大的成功。它提供了一个中心位置，用户可以在其中查找、下载和安装应用。它还为开发者提供了一种安全可靠的方式，将其应用分发给用户。App Store生态系统如此成功，以至于被其他平台如Google Play、Windows Store和Amazon Appstore等采纳。大多数用户打开手机时的第一件事就是找到合适的应用并打开它，或者从App Store安装尚未安装的应用——App Store已经成为手机体验中必不可少的一部分。

App Store生态系统继续蓬勃发展，但网络也没有停止进化。随着[HTML5](https://www.w3.org/TR/2011/WD-html5-20110405/)、[ECMAScript 6.0](https://262.ecma-international.org/6.0/)、[Node.js](https://nodejs.org/)和许多其他强大的[web API](https://developer.mozilla.org/en-US/docs/Web/API)s的出现，网络变得比以往更强大和更具能力。

2010年，[Ethan Marcotte](https://ethanmarcotte.com/)在[A List Apart](https://alistapart.com/)上发表了一篇文章，介绍了[响应式网页设计](https://alistapart.com/article/responsive-web-design/)的概念。响应式网页设计是一种设计方法，旨在创建能够适应绝大多数设备，从桌面计算机到移动设备的网站。响应式网页设计的核心思想是使用[CSS媒体查询](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_media_queries)根据设备的特点（如屏幕大小、方向和分辨率等）应用不同的样式。

![内容像水一样](https://scillidan.github.io/image_post/introducing-ppresume-pwa_07.webp)  

2015年，设计师[Frances Berriman](https://fberriman.com/)和她的丈夫[Alex Russell](https://infrequently.org/)，一位Google Chrome工程师，在Alex的博客中创造了“渐进式网络应用”的术语：[渐进式网络应用：在不失去我们的灵魂的情况下逃脱标签](https://infrequently.org/2015/06/progressive-apps-escaping-tabs-without-losing-our-soul/)。

> 昨晚晚餐时，Frances和我列举了这一新类应用的属性：
> 
> - 响应式：适应任何形态
> - 独立于网络连接：通过服务工作者渐进增强，实现离线工作
> - 类应用的交互：采用外壳 + 内容的应用模型，创造出应用般的导航与互动
> - 新鲜：通过服务工作者更新过程，透明地保持最新
> - 安全：通过TLS提供（服务工作者的要求），以防止窃听
> - 可发现：通过W3C清单和服务工作者注册范围被识别为“应用”，使搜索引擎能够找到它们
> - 可重新参与：能够访问操作系统的重新参与UI；例如，推送通知
> - 可安装：通过浏览器提供的提示安装到主屏幕，让用户可以“保留”他们认为最有用的应用，而不必麻烦地使用应用商店
> - 可链接：意味着它们是零摩擦、零安装，易于分享。URL的社交力量是重要的。
> 
> — Alex Russell

在我看来，PWA本质上仍然是**网络**，然而，它旨在通过对网站的**渐进式**增强提供一种**应用**般的用户体验。

那么问题来了：为什么选择PWA？

## 为什么选择PWA

自从互联网的发明以来，有两种主要方式向用户提供服务：

- 本地应用，这些应用是针对特定平台（如Windows、macOS、Linux、iOS、Android等）专门设计的。
- 网络应用，这些应用旨在通过任何设备上的网络浏览器访问。

从用户的角度来看，本地应用通常提供更好的用户体验、最佳性能和利用设备所有功能的能力。然而，用户需要花时间下载和安装应用。

另一方面，网络应用可以通过网络浏览器访问，无需下载和安装任何东西，并且用户始终会自动获得最新的网络应用。然而，用户体验通常不如本地应用，特别是在移动设备上，而且网络应用无法充分利用设备的所有功能，因为它们在浏览器中运行并依赖于浏览器实现来连接到设备特定的功能。

从开发者的角度来看，为多个平台开发和维护本地应用成本非常高，而且应用商店的审核过程可能漫长且充满风险，而对于网络应用，只涉及一个可以在多个平台上运行的代码库，发布更新也非常简单，因为中间没有应用商店。

最后但并非最不重要的是，网络应用本质上是开放和免费的，基于开放标准，可以在任何设备上使用，只要有浏览器，最重要的是，没有[恩威并施的独裁者](https://en.wikipedia.org/wiki/Benevolent_dictatorship)决定哪些网络应用可以根据某些应用商店指南交付给公众。自由是无价的，不是吗？

考虑到本地应用和网络应用的优缺点，为什么还要关注PWA呢？**简言之，PWA兼具两者的优点。**

PWA本质上仍然是一种网络应用，可以通过任何网络浏览器访问；它是可发现的，可以被搜索引擎索引并获得免费流量；它建立在URL之上，默认是可链接的，这意味着它是零摩擦、零安装，非常易于分享。通过渐进式增强，它可以获取许多仅适用于本地应用的功能，例如：

- **可安装**：PWA可以通过浏览器提示或应用商店安装为独立应用。
- **可发现**：PWA在安装后在设备的桌面、停靠或主屏幕上有自己的图标。
- **独立窗口**：PWA在安装后有自己的窗口，因此您不会在浏览器的标签中迷失。
- **更好的性能**：PWA可以利用服务工作者缓存资源和数据，因此您不会一次又一次从远程服务器下载相同的资源和数据，这极大地减少了网络延迟和带宽成本，从而提供更流畅的用户体验。
- **离线支持**：PWA在部分情况下可以离线工作，只要访问过某个页面，在离线时您仍然可以以只读模式继续使用它。
- **推送通知**：PWA可以向用户发送推送通知，重新吸引用户并将他们带回应用。
- **本地集成**：PWA可以利用设备的其他本地特性，如相机、麦克风、GPS等，通过集成[web APIs](https://web.dev/learn/pwa/capabilities#empowering_your_pwa)，查看[今天PWA能做什么](https://whatpwacando.today/)。

总的来说，PWA是一种“投入小，回报大”的努力，通过渐进增强为网络提供应用般的体验。渐进增强意味着您不必一次性进行所有增强，而是可以从基本功能开始，逐步增强更多高级功能。首先使您的网站响应式，然后通过创建清单文件使其可安装，再应用服务工作者使其能够离线工作，然后集成其他高级功能。灵活性、快速迭代，这就是网络开发的精神，这正是创建PWA所需的一切。

## 展示时间

好的，长话短说，现在是展示时间。

[(视频) 介绍PPResume PWA](https://www.youtube.com/watch?v=hoqP0vUMDy4)

## 未来改进

PPResume PWA还远未完美，仍有很多改进的空间。以下是我仍在努力工作的功能和错误修复列表：

- 功能增强
	- 提供丰富的安装UI：[问题](https://github.com/ppresume/community/issues/46)
	- 自定义离线页面：[问题](https://github.com/ppresume/community/issues/49)
	- 尊重系统深色模式，[问题](https://github.com/ppresume/community/issues/52)
	- 向各大商店提交应用，[问题](https://github.com/ppresume/community/issues/50)
- 错误修复：
	- 修复iOS上的黑色启动画面。[问题](https://github.com/ppresume/community/issues/54)
- 文档：
	- PPResume PWA的文档。[问题](https://github.com/ppresume/community/issues/56)
- 测试：
	- 检查PPResume PWA在不同设备上的兼容性。[问题](https://github.com/ppresume/community/issues/57)

还有许多工程细节值得深入探讨，例如：

- 如何创建好看的网络应用清单
- 如何利用服务工作者缓存资源和数据
- 如何处理缓存CDN资产时的CORS问题
- 使用哪个库实现Next.js PWA

不过，我会将这些主题留到未来的博客文章中，否则这篇文章会太长而影响阅读体验。

请继续关注！

[GPT-4o mini]
