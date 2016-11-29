# 1.4开启Dojo之旅
原文地址：https://dojotoolkit.org/documentation/tutorials/1.10/start/index.html

----------
如何学习Dojo？哪里有文档？怎样获得培训和支持？应该用哪个版本的Dojo？为什么需要web服务？怎样避免常见的错误？如何提交问题报告？可通过什么方式来作出贡献？本教程的目的就是回答这些问题。

## 文档
Dojo网站为新手提供了三个主要的文档，它们都是由广大社区群众贡献完成的。

### 教程
[教程集](https://dojotoolkit.org/documentation/?ver=1.10) 提供一系列关于Dojo开发主题的入门。排在前面的几个教程通常是我们优先推荐的。除此之外，其他教程是依照主题分类的。这一系列教程主要是[SitePen](http://sitepen.com/) 上的团队提供在“如何开始学习Dojo？”的回答里。你应该从 Hello Dojo 这篇教程开始，或者如果你以前用过Dojo，只是刚接触1.10，你可以从Modern Dojo教程开始，当然是在你看完本篇之后。

### 参考指南
[参考指南](https://dojotoolkit.org/reference-guide/1.109/)是基于API的文档的深度收集。想参与贡献请看[improve the reference guide through GitHub](https://github.com/dojo/docs/)。

### API浏览器
 API浏览器有用有全部的[Dojo API](https://dojotoolkit.org/api/) 。他们100%来源于源码注释和Javascript源码，主要使用了两个开源项目：[js-doc-parse](https://github.com/SitePen/js-doc-parse) ——用来解析资源树、[API viewer](https://github.com/wkeese/api-viewer) ——用来交付已解析资源树的可读版本。你可以通过 [generate your own custom API viewer](http://www.sitepen.com/blog/2013/01/18/generating-and-viewing-custom-api-docs/) 在你自己的代码中使用这两个项目。
 这些文档之间存在一些交叉引用，未来会作出改进。每一个文档都是基于版本发布的。教程和参考指南涵盖1.6、 1.7、1.8、1.9和1.10，API浏览器则可以追溯到1.3版本。本文末尾可了解到该文档系统的问题和参与改善我们的文档的相关信息。

### 培训和支持
SitePen提供一系列优秀的 [Dojo workshops](http://sitepen.com/workshops) 和 [Dojo and JavaScript support](http://sitepen.com/support)。

### 相关书籍
编写一本优秀的书需要1000-2000个小时（Dojo网站上的系列教程就花了超过1000个小时）。现存的Dojo出版书籍都是在Dojo1.0到1.5之间发行的。我们正在讨论发行一本新的Dojo书籍，一本能够频繁更新并可以购买PDF或印刷版的书。

## 哪个Dojo版本？
我们推荐尽可能使用最新的Dojo版本，但也想旧的发行版提供不间断的支持，因为我们知道升级应用的源代码需要费很大的力气。
Dojo基金会承诺当前会为1.4和更高版本提供新浏览器的支持。我们将定期更新旧版本的Dojo来支持新发布的浏览器。我们也会修复主要的bug和退化，特别是针对最近发布的主流版本。新功能开发创建通常主要局限在当前最新版本。
对于现在来说，新功能开发会基于1.11版本，1.10.x主要进行bug修复，并定期为1.4.x和更新版本提供浏览器支持。
了解Dojo每个发行版的主要特点和新增有助于你作出决定。在版本之间进行升级时，一定要查看每个主要发行版本的发行说明提供的指导。我们尽最大的努力使升级做到向前兼容，但你在把代码向新版本Dojo迁移的时候仍需要在修复bug和引入新特性上花一些力气：

 - 1.10: [release notes](https://dojotoolkit.org/reference-guide/1.10/releasenotes/1.10.html), [announcement](http://dojotoolkit.org/blog/dojo-turns-1-10)
 - 1.9: [release notes](https://dojotoolkit.org/reference-guide/1.10/releasenotes/1.9.html), [announcement](http://dojotoolkit.org/blog/dojo-1-9-released)
 - 1.8: [release notes,](https://dojotoolkit.org/reference-guide/1.10/releasenotes/1.8.html) [announcement](http://dojotoolkit.org/blog/dojo-1-8-released)
 - 1.7: [release notes](https://dojotoolkit.org/reference-guide/1.10/releasenotes/1.7.html), [announcement](http://dojotoolkit.org/blog/dojo-1-7-released)
 - 1.6: [release notes](https://dojotoolkit.org/reference-guide/1.10/releasenotes/1.6.html), [announcement](http://dojotoolkit.org/blog/dojo-1-6-released)
 - 1.5: [release notes](https://dojotoolkit.org/reference-guide/1.10/releasenotes/1.5.html), [announcement](http://www.sitepen.com/blog/2010/07/22/dojo-1-5-ready-to-power-your-web-app/)
 - 1.4: [release notes](https://dojotoolkit.org/reference-guide/1.10/releasenotes/1.4.html), [announcement](http://www.sitepen.com/blog/2009/12/10/dojo-1-4-released/)


## 入门FAQs
这里有一些关于入门的各种常见FAQ和小贴士，先了解一些会比较有用。

### 弃用警告
当你在新Dojo发行版中使用旧版特性时，偶尔会看到弃用的警告。这个警告的意思是该API或特性将在Dojo2.0里移除，并且可能会有一种改进的方法可以考虑。

### 用web服务器
从web服务器上运行你的源码，不要在文件系统里，哪怕web服务器是搭建在你的开发机器上的。就算是在同一台机器上，浏览器对HTTP请求的处理在本地文件系统比web服务器上受限的多。为了保证结果一致，你应该经常用HTTP web服务器(Apache, nginx, Tomcat, IIS, Jetty,等.)来运行Dojo。

### CDN 和 protocol-less URLs
你可能从一个CDN加载Dojo。对于快速使用Dojo这很有用，而且也不要求你有自己的Dojo副本。在我们很多教程里都能看到protocol-less URLs，例如` <script src="//ajax.googleapis.com/ajax/libs/dojo/1.10/dojo/dojo.js" data-dojo-config="async:true"></script>`。你这样不用调整URL就可以在http和https应用里使用Dojo。详见 [Dojo CDN tutorial](https://dojotoolkit.org/documentation/tutorials/1.10/cdn/)。

### 常见错误
之前有人犯过和你一样的错误的概率很高。SitePen 基于他们培训研讨班创建了一个很棒的博客条目，展示在 [common Dojo bugs and error messages](https://www.sitepen.com/blog/2012/10/31/debugging-dojo-common-error-messages)，还有如何解决他们。

## 已知的文档问题
我们已经通过社区贡献充分的改进了我们的文档。即便如此，还有一些已知的问题（以及在预定的时间里数以百计的内容修正）。已知的问题如下：

 - pre-AMD vs. AMD syntax。Dojo1.7我们引入了AMD格式来require和define源码模块。1.7, 1.8, 1.9, 和1.10 提供的一些文档里仍然使用的旧语法。文档只是在部分区域过时，而这些语法也依然支持，所以仍然很有用。
 - 1.7 API viewer。API viewer的1.7版本缺少重要内容。1.8, 1.9, 和1.10已经解决了这个翁提，他们的API也和1.7相似。 如果API viewer页面空白或者缺少细节，请参考1.8, 1.9, 和1.10版本的相同页面。
 - 在1.6之前没有教程和参考指南，在1.3之前没有API viewer。很简单，因为我们在这些版本之后才开始做的，所以就的版本是没有的。
 - 在平板和手机上查看文档。API viewer 和示例主要面向桌面浏览器。请把问题报告给我们，或者参与修复它们。
 - 参考指南的IE示例。我们已经解决了很多关于IE浏览器加载参考指南示例的相关问题。如果发现遗留问题，请按下面提到的方式报告。
 
## 报告问题
 - 对于文档错误，请在每页底部的链接处提交问题报告。2012年我们在加上这个功能之前已经解决了千把个文档问题。修复有时候很快，有时候需要花费几个月，但是我们会全部审阅并且非常感激能够改进文档的建设性反馈。
 - 对于文档说明和后续问题，请 [ 注册dojo-interest mailing list](http://mail.dojotoolkit.org/mailman/listinfo/dojo-interest) ，在那提问题，或者在我们的[freenode](http://www.freenode.net/irc_servers.shtml)的即时聊天频道#dojo 。
 
## 贡献
 Dojo是完全由社区的努力和贡献推动的。想要参与更高的反馈，我们要求您创建一个Dojo基金账户，然后签署我们的网上贡献者许可协议。确保把你的CLA和你的bug追踪账户绑定以简化我们的过程。一旦你在文件上拥有CLA，下面会有所帮助：
 
 - API Viewer。通过GitHub上的  [js-doc-parse](https://github.com/SitePen/js-doc-parse) 和[API viewer](https://github.com/wkeese/api-viewer) 项目。
 - Reference Guide。通过GitHub上的[Dojo docs](https://github.com/dojo/docs/) 项目。
 - 编写新教程。通过[dojo-interest mailing list](http://mail.dojotoolkit.org/mailman/listinfo/dojo-interest) 表达你的兴趣，直接联系[Dylan Schiemann](https://twitter.com/dylans) ，或者直接用我们的 [contact form](http://dojofoundation.org/contact)联系我们。
 - 从DOH到内部的更新测试。详见： [Dojo Automated Testing Improvements: Updating to Intern](https://www.sitepen.com/blog/2014/02/18/dojo-automated-testing-improvements-updating-to-intern/)。
 - 贡献补丁和bug修复。对于有些包比如 [dgrid](http://dgrid.io/)，你应该直接通过GitHub项目。Dojo 1.x使用GitHub，Dojo 2.0也会首选它。
 - 访问Dojo基金会网站来了解关于[参与Dojo](http://dojofoundation.org/about/)。
 
## 入门
你应该从[Hello Dojo tutorial](https://dojotoolkit.org/documentation/tutorials/1.10/hello_dojo/) 入门，如果你有Dojo使用经验，但刚接触1.10，可以从 [Modern Dojo tutorial](https://dojotoolkit.org/documentation/tutorials/1.10/modern_dojo/) 开始。
