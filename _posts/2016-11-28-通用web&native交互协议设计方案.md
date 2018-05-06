---
layout:     post
title:      通用Web&Native交互协议设计方案
date:       2016-11-28 17:09:03 +0800
author:     Zhang yinglong
tags: 	    iOS
---

随着移动互联网地快速发展，手机设备性能大幅提高，CPU，内存，GPU等性能都大大超过了3-4年前的大部分智能手机；手机OS操作系统开放的功能越来越丰富，如JavaScriptCore，WebKit等；HTML5正式标准的确立和推广，使得手机上浏览Web的用户体验也越来越好，这也就出现了越来越多的Web与Native之间的交互需求。比如：1、Native加载Web时，传递登录信息等；2、Web问卷调查时，需要调用手机摄像头等。

解决Web&Native交互问题的解决方案大致有两种：
1、拦截URL Scheme
在Web端JS代码中使用 location、iFrame 等方式做页面跳转，Native端事先定义好用作实现交互功能的API，在加载跳转链接前拦截处理
![](/assets/images/2016/1200910-697153e7effa48cf.png)
2、注入bridge对象
在Web容器加载之前注入桥接bridge对象，在bridge中定义了交互方法。
若使用JavaScriptCore框架，则是为JavaScript运行的上下文环境JSContext注入对象；若使用WebKit框架，则是为webkit.messageHandlers注入对象。代码如下：
![](/assets/images/2016/1200910-21ae42f3fe799c30.png)
以上两种传统的解决方案都有相同的缺点：接口扩展性不够灵活，只能在APP内部站点使用。

### 设计方案
在Web中存在一个唯一标识URI（Uniform Resource Identifier），URI既可以看成是资源的地址，也可以看成是资源的名称。因此移动端也采用URI来定位一个具体的资源（包括native页面，接口，功能模块，甚至是第三方对接的APP应用）。具体格式如下：

```
scheme://business/path/method?query
```
scheme为自定义协议头，business表示业务线编码，path表示业务线模块编码，method表示API，query表示业务模块所需的参数，参数使用标准的URLEncode编码（为了支持中文）。
Native端使用URLRouter解耦各个业务模块，自然也很容易支持这套URI协议，这样我们就解决了可扩展性。
但是自定义URI协议只能在Native中使用，一旦Web在浏览器中打开，自定义URI就只能望洋兴叹了。为了解决此问题，我们可以引入iOS 9的一个功能：Universal Links（通用链接）。如果你的App支持Universal Links，那就可以访问HTTP/HTTPS链接直接唤起App进入具体页面，不需要其他额外判断；如果未安装App，访问此通用链接时，可以跳转至一个自定义内容网页（页面可以引导用户去下载App）。对用户来说，这是一个非常顺畅的跳转过程。
将之前设计好的自定义URI协议作为一个整体，加入到Universal Links中，

```
https://s.mlinks.cc/AAs8?api=<URI>
```
当用户在浏览器中点击Web链接时，会唤醒App我们就可以通过URLRouter完成交互功能，这也就解决了不支持外部站点的问题。

总结一下，本文提出的通用Web&Native交互协议方案：App内部Web站点使用注入对象，外部站点使用Universal Links，交互方式采用自定义URI，URI根据实际业务情况自行设计（最好支持URLRouter的解耦模块）。同样的我们还可以将这套交互协议扩展至push消息的处理。至此，我们完成了一套通用Web&Native交互协议设计方案。