---
layout:     post
title:      WKWebView从入门到趟坑
date:       2016-12-17 01:33:07 +0800
author:     Zhang yinglong
tags: 	    iOS
---

### UIWebView 之痛

开发App的过程中，常常会遇到在App内部加载网页，通常用UIWebView加载。而这个自iOS2.0开始使用的Web容器一直是开发的心病：加载速度慢，占用内存多，优化困难。如果加载网页多，还可能因为过量占用内存而给系统kill掉。各种优化的方法效果也不那么明显，常见的优化缓存方式：
1、尽量使用 GET 请求，iOS 系统 SDK 会自动帮你做缓存。你需要的仅仅是设置下内存缓存大小、磁盘缓存大小、以及缓存路径。只要设置了这两行代码，基本就可满足80%的缓存需求。
![设置缓存.jpg](/assets/images/2016/1200910-d1fd440debe66659.png)

2、Web资源离线加载，热更新资源，完成另外20%的缓存需求（Hybrid框架的Web部分）。
可是无数开发者尝试自己做一个“简陋而脆弱的”系统来实现网络缓存的功能，效果往往是事倍功半 。

### 初识 WKWebView

UIWebView从 iOS2 就有，iOS8 以后，苹果推出了新框架 WebKit，提供了替换 UIWebView 的组件 WKWebView。各种 UIWebView 的性能问题没有了，速度更快了，占用内存少了，体验更好了，下面列举一些其它的优势:
1、在性能、稳定性、功能方面有很大提升（加载速度，内存的提升谁用谁知道）
2、更多的支持 HTML5 的特性
3、官方宣称的高达60fps的滚动刷新率以及内置手势
4、Safari 相同的 JavaScript 引擎
5、将 UIWebViewDelegate 与 UIWebView 拆分成了14类与3个协议，包含该更细节功能的实现。
相比之下，WKWebView 复杂得多，一些常用API如下：

### 容器相关
![WKWebView.jpg](/assets/images/2016/1200910-6ce3c8147f12ea51.jpg)

#### JavaScript 配置相关

![JavaScript API.jpg](/assets/images/2016/1200910-f0a0db0317f46a0f.jpg)

#### 存储相关（只支持iOS9以上）

![存储类型.jpg](/assets/images/2016/1200910-4aba55b318eec700.jpg)

![存储 API.jpg](/assets/images/2016/1200910-d398babae43c7e96.jpg)

#### 页面加载相关

![WKWebView loadRequest.jpg](/assets/images/2016/1200910-f048d8c67973b10b.jpg)

#### 代理相关

![WKNavigationDelegate.jpg](/assets/images/2016/1200910-45435434b221468f.jpg)

看完API以后，要掌握 WKWebView 并不难，难的是如何处理iOS版本碎片化兼容问题。

### 性能对比测试

都说提高多么多么大的性能，实测告诉你 WKWebView 的性能有多好，下面用实际项目做个对比测试：
UIWebView 首次加载 www.58.com 首页，耗时 0.0154584ms，内存消耗 24.1 MB
![UIWebView耗时.jpg](/assets/images/2016/1200910-dd80f1a39d0da790.jpg)
![UIWebView内存消耗.jpg](/assets/images/2016/1200910-309fd27cd1b9afe5.jpg)

WKWebView 首次加载 www.58.com 首页，耗时 0.013875ms，内存消耗仅 6.4 MB
![WKWebView耗时.jpg](/assets/images/2016/1200910-b72e02c112781ba3.jpg)
![WKWebView内存消耗.jpg](/assets/images/2016/1200910-0c14aca01c270408.jpg)

**结论：加载耗时差别不大，WKWebView 的内存优化减少了几乎4倍，更重要的是，无论 WKWebView 跳转多少 Web 页面都没有内存泄漏了。WKWebView 使用和 Safari 相同的 Nitro JS 引擎性能，对HTML5性能也提升了4倍。**

### WKWebView 之坑

新技术的出现必然会或多或少的瑕疵，WKWebView 也不例外。

#### 1、关于缓存
在 WKWebsiteDataStore 出现之前（iOS 9 中），WKWebView 是没有缓存，也无从清理。WKWebView 是基于 WebKit 框架的，它会忽视先前使用的网络存储 NSURLCache, NSHTTPCookieStorage, NSCredentialStorage等，它也有自己的存储空间用来存储cookie和cache，其他的网络类如NSURLConnection 是无法访问到的。 同时WKWebView发起的资源请求也是不经过NSURLProtocol的，导致无法拦截或自定义新请求。
体验过 WKWebView 的一定会遇到修改了H5页面，APP打开时却没有即时更新的问题（实在是缓存得太好了），iOS 8的时候只能增加时间戳的方式解决这个问题（调试下使用，生产环境就只能要求前端修改Cache-Controll了），如下：
![时间戳更新.jpg](/assets/images/2016/1200910-5bbb8dc02e1c1164.jpg)

iOS 9以后终于可以使用 WKWebsiteDataStore 来清理缓存。后来Google一下，又发现iOS 8可以通过清理 Library 目录下的 Cookies 目录来清除缓存，于是
![清除WKWebView缓存.jpg](/assets/images/2016/1200910-155f9838a43aa4db.jpg)

缓存清理的坑趟过了，喜大普奔。

#### 2、关于 Cookie

在使用 UIWebVIew 的时候我们并不关注 Cookie，因为在调用登录接口的时候无论是AFNetworking，还是其他，登录成功之后都会自动保存在
[NSHTTPCookieStorage sharedHTTPCookieStorage].cookies 中，以后再使用也会自动去获取（这里有个 UIWebView 的坑：访问的链接越多，如不处理Cookie，它会加载越来越多的无效 Cookie 导致内容急剧增大）。但 WKWebView 的存储体系与 UIWebVIew 完全不一样，只能手动给它添加 Cookie，如下：
![WKWebView设置cookie.jpg](/assets/images/2016/1200910-bb6b6890fe1774f6.jpg)

但即便如此，Cookie 还是会偶现丢失的问题，最终只好采用每次 Web 开始加载之时判断 Cookie 是否存在，否则手动添加重新加载，如下：
![WKWebView设置cookie.jpg](/assets/images/2016/1200910-0226b4e39b5cc281.jpg)

Cookie 获取的坑趟过了，再次喜大普奔。

#### 3、关于跨域

WebKit框架对跨域进行了安全性检查限制，不允许跨域，比如从一个 HTTP 页对 HTTPS 发起请求是无效的（有一个界面要跳到支付宝页面去支付，死活没反应）。而系统的 Safari ，iOS 10出现的 SFSafariViewController 都是支持跨域的，因此解决办法如下：
![跨域跳转.jpg](/assets/images/2016/1200910-1039aa39914f2e79.jpg)

对于自身域名，还是建议全站 HTTPS 化吧（大势所趋）。

#### 4、关于 JavaScript 交互

UIWebView 使用的 JavaScriptCore 框架，交互时为 JavaScript 运行的上下文环境 JSContext 注入对象 Bridge；WKWebView 使用的 WebKit 框架，交互时为 webkit.messageHandlers 注入对象，如下：
![JavaScript注入.jpg](/assets/images/2016/1200910-05ad39acba4c05ec.jpg)

前端H5需要做判断两种不同注入方式带来的不同调用方式：
![js调用.jpg](/assets/images/2016/1200910-ffe0251ccd26d0bf.jpg)

#### 5、关于 NSURLProtocol 拦截

WKWebView 基于 WebKit 框架，与 UIWebView 机制不同：加载过程中所有的请求都不经过 NSURLProtocol，换句话说就是 WKWebView 无法拦截响应数据 鉴于之前大部分 Hybrid 框架的离线预加载机制都依赖于拦截功能，这意味着废掉很多程序猿们辛辛苦苦设计实现的 Hybrid 框架（内功尽失，感觉身体被掏空），再加上 WKWebView 自身的坑不少，因此很多团队都不会轻易替换掉 UIWebView。拥抱变化吧，WKWebView 迟早会取代 UIWebView 成为最佳 Web 容器（iOS 9带来的 SFSafariViewController 更是武功全废，啥都干不了，只能干瞪眼）。

那么问题来了，如何设计新的 Hybrid 框架呢？此处出门左转，点击文章开头进入公众号历史文章，查看《通用Web&Native交互协议设计方案》。

#### 6、关于 POST 请求

简书 http://www.jianshu.com/p/403853b63537 中有关于这个坑的具体描述，笔者这里就不再做研究，这里只说明怎么趟过的坑：使用通用的 Web&Native 交互协议，为 Web 提供 Native POST 请求的接口+回调 CallBack 即可，参见 **关于 JavaScript 交互**。

#### 7、关于本地 HTML 加载

当使用 loadRequest 来读取本地 Documents 目录的 HTML 文件时，WKWebView 是无法读取成功的，只能通iOS 9的新接口加载
![](/assets/images/2016/1200910-f5961819811ccd89.png)

但是在iOS9以下的版本是没提供这个便利的方法的，解决办法：先将本地 HTML 文件的数据 copy 到 tmp 目录中，然后再使用 loadRequest 来加载。但是如果在 HTML 中加入了其他资源文件，例如 js，css，image 等也必须一同 copy 到 tmp 中，这个是非常蛋疼的事情了。然而还有更蛋疼的事：iOS 8下还必须 copy 到 tmp 的 www 目录下 WKWebView 才能读取（Word天，心中千万只草泥马狂奔而过）。参见 http://stackoverflow.com/questions/24882834/wkwebview-not-loading-local-files-under-ios-8

#### 8、关于捏合手势

很多人都喜欢使用 UIWebView 的捏合手势来进行放大和缩小，观看 Web 内容，但 WKWebView 在手机上不支持，却支持其他iOS设备（草泥马再次狂奔而过）。
![](/assets/images/2016/1200910-eacdb13fe3477d0c.jpeg)

### 写在最后

当时选择 WKWebView 就是为了提高性能，但是没有想到遇到这么多坑，真是我待 WKWebView 如初恋，WKWebView 虐我千百遍，兴许还有许多未知的坑，欢迎大家留言补充。谢谢支持！