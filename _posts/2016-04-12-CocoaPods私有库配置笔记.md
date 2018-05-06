---
layout:     post
title:      CocoaPods私有库配置
date:       2016-04-12 23:59:41 +0800
author:     Zhang yinglong
tags:       iOS
---

`CocoaPods`是一个iOS，Mac OS下强大的依赖包工具，不仅用来管理第三方开源代码的项目代码，您也可以通过配置公共组件的私有pods库，来管理整个项目中的公共组件。**通过下面几个步骤，您可以为项目创建私有pods库**。

### 1. 创建私有Spec仓库

为了管理您的私有`pods`库，建议创建一个自己的`Spec`仓库。这个`Spec`仓库包含了所有将要访问到的`pods`库。您不需要从`CocoaPods/Specs`检出分支，只需要保证团队里所有人都有这个`Spec`仓库（不公开的）的访问权限。
（通常`CocoaPods`使用中都会包含公开的和私有库，因此`Spec`仓库也可以直接使用默认公开库的路径地址）

### 2. 添加私有Spec仓库到CocoaPods默认的路径

```
pod repo add <REPO_NAME> <SOURCE_URL>
```

> 注意：如果打算创建本地的`pods`仓库，您还需要推送至`<SOURCE_URL>`源地址
>检测您的`Spec`仓库是否创建成功： 
>
```
cd ~/.cocoapods/repos/<REPO_NAME> // 不同版本的CocoaPods，路径可能不一样
pod repo lint .
```

### 3. 添加私有Spec仓库的Podspec配置文件

确保您的源代码已经打好版本tag，然后执行

```
pod repo push <REPO_NAME> <SPEC_NAME>.podspec
```

这将会执行`pod spec lint`，注意过程中的细节，`Spec`仓库目录格式应该是

```
.
├── Specs
    └── [SPEC_NAME]
        └── [VERSION]
            └── [SPEC_NAME].podspec
```

##### 至此，您的私有`pods`仓库已经可以通过`Podfile`文件使用了，还需要在在`Podfile`中指定私有库的源地址，如下：

```
source '<URL_TO_REPOSITORY>'
```

## 举个例子

### 1. 创建私有代码仓库

创建一个代码仓库，可以托管在Github，或您自己的git服务器

```
$ cd ~/Desktop
$ mkdir EGKit.git
$ cd EGKit.git
$ git init --bare
```

(The rest of this example uses the repo at [https://github.com/zhangyinglong/EGKit.git](https://github.com/zhangyinglong/EGKit.git))

### 2. 添加本地私有库的CocoaPods安装源

使用你的代码仓库地址，作为CocoaPods安装源

```
$ pod repo add EGKit https://github.com/zhangyinglong/EGKit.git
```

进入本地CocoaPods安装源路径（`CocoaPods`版本不同路径可能也不同），检查是否安装成功

```
$ cd ~/.cocoapods/repos/EGKit
$ pod repo lint .
```

### 3. 为私有库添加Podspec配置文件

创建Podspec配置文件

```
cd ~/Desktop
touch EGKit.podspec
```

编辑配置文件内容 `EGKit.podspec` 。

```
#
# Be sure to run `pod lib lint EGKit.podspec' to ensure this is a
# valid spec before submitting.
#
# Any lines starting with a # are optional, but their use is encouraged
# To learn more about a Podspec see http://guides.cocoapods.org/syntax/podspec.html
#

Pod::Spec.new do |s|
  s.name             = "EGKit"
  s.version          = "0.1.0"
  s.summary          = "A demo of EGKit."

# This description is used to generate tags and improve search results.
#   * Think: What does it do? Why did you write it? What is the focus?
#   * Try to keep it short, snappy and to the point.
#   * Write the description between the DESC delimiters below.
#   * Finally, don't worry about the indent, CocoaPods strips it!  
  s.description      = <<-DESC
                        A test demo of EGKit.
                       DESC

  s.homepage         = "https://github.com/zhangyinglong/EGKit"
  # s.screenshots     = "www.example.com/screenshots_1", "www.example.com/screenshots_2"
  s.license          = 'MIT'
  s.author           = { "zhangyinglong" => "zyl04401@gmail.com" }
  s.source           = { :git => "https://github.com/zhangyinglong/EGKit.git", :tag => s.version.to_s }
  # s.social_media_url = 'https://twitter.com/<TWITTER_USERNAME>'

  s.platform     = :ios, '7.0'
  s.requires_arc = true

  s.source_files = 'Pod/Classes/**/*'
  #s.resource_bundles = {
  #  'EGKit' => ['Pod/Assets/*.png']
  #}

  # s.public_header_files = 'Pod/Classes/**/*.h'
  # s.frameworks = 'UIKit', 'MapKit'
  # s.dependency 'AFNetworking', '~> 2.3'
end
```

保存`EGKit.podspec` 配置文件，并添加到本地`pods`库中

```
pod repo push EGKit ~/Desktop/EGKit.podspec
```

假如您的Podspec配置成功，它将会自动添加到本地`pods`库中，目录结构如下：

```
.
├── Specs
    └── EGKit
        └── 0.1.0
            └── EGKit.podspec
```

### 4. 如何删除私有库

```
pod repo remove EGKit
```

参考文献：[http://guides.cocoapods.org/making/private-cocoapods.html](http://guides.cocoapods.org/making/private-cocoapods.html)
