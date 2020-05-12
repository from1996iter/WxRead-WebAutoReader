# WxRead-WebAutoReader

使用 **[微读自动阅读器](https://github.com/DoooReyn/WxRead-WebAutoReader)**，帮助你解放生产力 _（双手）_ ，该干啥干啥去，书币照样拿！

## 更新

- **2020.02.17**

  - 新增标题栏提示，更新章节进度和已读完状态；
  - 新增浏览器通知 _（需要用户授权）_，读完后发送通知给用户，比 alert 更好用 _（由于部分浏览器不支持通知，因此保留 alert）_。

- **2020.02.18**

  - 新增链接拖拽到收藏栏方式添加 **[微读自动阅读器](https://github.com/DoooReyn/WxRead-WebAutoReader)**，见此[链接](https://doooreyn.github.io/wxreader.html)；
  - 现在确认 Safari 浏览器不支持使用书签方法调用，目前没有很好的办法，临时的解决方案是：打开开发者模式，打开控制台，复制代码到控制台直接运行；
  - 目前个人阅读时长：12 小时 28 分钟，因为没有一直开着屏幕，所以不是很长，但是对于上班族来说，1 天 8 小时绝对够用了，如果有条件，不妨试试 24 小时开机。

- **2020.02.22**
  - 修复大概率出现一直刷同一章节的问题: HTMLAnchorElement.click() 有概率失效，因此放弃此方法，使用键盘事件来切换下一章。

- **2020.03.27**
  - 修复在首页使用无效的问题，感谢**Lovely-Wildpointer**提的[issue](https://github.com/DoooReyn/WxRead-WebAutoReader/issues/2)

- **2020.05.07**
  - 修复官方启用 **CPS** 规则后 **WebWorker** 无法使用的问题。

## 做微读自动阅读器的原因

闲的蛋疼！

当然不是！

实际上是因为在微信读书周阅读排行榜里看到了一个 132 小时的 bug 一般的存在，然后向网络求证了一下，发现知乎下有个人问了这个问题，求证之后顺手答了，大家可以在这里看我的回答：[微信读书时长究竟如何计算?](https://www.zhihu.com/question/349487832/answer/1020412380)。

主要还是现行的**挂机方法**太次，所以动手写了这个程序。

为什么我说现行的方法太次？我给你介绍一下它的实现思路：**下载一个安卓模拟器，在模拟器里安装微信读书，通过 ADB 建立模拟器与 Python 脚本之间沟通的桥梁，打开微信读书并选择一本书，运行 Python 脚本，最后由脚本实现微读 UI 自动化，达到模拟机器人读书的目的，也就是挂机了。**

这乍看起来没什么问题吧？然而不是。实际上，首先它涉及到一大堆的概念：

- 安卓模拟器
- ADB
- Python 语言
- Python 依赖

这些步骤对于普通用户来说，实在是过于繁琐和复杂，甚至一不下心就会陷入**我明明按照使用说明做的，为什么就是不行？**的蜜汁困境，我相信只有程序员与极少数的发烧友才可能去折腾这些东西。

而且其中还有很多限制，比如说：ADB 是需要连接调试的，Python 需要安装依赖，模拟器需要设置很长的息屏时间，读完一本书就会暂停而用户根本不知道什么时候会暂停，以及不同 PC 平台部署是有些微差别的，等等。这其中很多事情根本就是开发人员做的事情，你不能把开发的东西丢给终端用户吧！

所以上述问题都可以归结于一点：**由于部署困难导致它的受众范围必然很小**。这是它最失败的地方，也是个人觉得它不会进入大众视野的根本原因，同样也是必然的结果。

## 微读自动阅读器是怎么做的

有鉴于此，我换了另外一个思路来实现，其实有点讨巧，而微信读书又刚好很赏脸地上线了微信读书网页版，并且同样计算有效时长！那么，上面所述的部署困难问题从这里开始就已经被完美地解决或者说规避了，因为从现在开始我们仅仅只需要一个 PC 浏览器了！

接下来的问题也是唯一的问题就是如何模拟用户读书从而达到挂机的目的。首先自然是要分析微信读书网页版的阅读习惯是怎么设计的，这样才好安排程序怎么做，这里就不赘述分析过程，直接贴结果了：**网页版同 APP 版一样，书本分章节，不同的是，APP 版翻页会自动跳转到下一章，网页版每章之间有一个`下一章`的按钮，用户必须点击`下一章`才会跳转；书本阅读完毕后不会出现`下一章`。**

明白了微读网页版的机制，接下来就轮到 JavaScript 出场了，现在我们要用 JavaScript 实现：

1. 获取阅读内容高度和屏幕高度；
2. 每隔几秒钟自动滚动页面；
3. 滚动时判断是否滚到底部；
4. 滚到底部时说明本章已阅读完成，可以跳转下一章了；
5. 找到下一章按钮，模拟用户点击，等待进入下一章，然后回到第 1 步继续阅读；如果找不到按钮，则判定本书阅读完毕，发送通知给用户，用户接收到通知，重新选择一本书。

原理很简单，代码也不复杂，唯一碰到的一个问题就是：刚开始使用 setInterval 做定时器，放到后台后时间长了发现频率不对劲，有的时候快有的时候慢，搜索了一下原因是浏览器的耗电保护机制，解决办法是使用 Web Worker，于是重新使用 Web Worker 实现了一次，测试正常了。详细的实现过程就不说了，大家按照**使用说明**直接拿去用就可以了，现在我们只要 3 个步骤就可以轻轻松松地一边上班一边挂(摸)机(鱼)了：

1. 打开微读网页版、打开任意一本书；
2. 点击 `微信读书自动阅读器` 开启自动阅读；
3. 等待阅读完成通知，回到第一步。

**PS:** 其实，我还连夜学了怎么写 Chrome 扩展，然后只要打开微读网页版，点开任意一本书就会自动开启阅读程序，本来想发到 Chrome 商店的，但是 Chrome 发布扩展竟然要\$，而我实在懒(非)得(常)折(的)腾(穷)了。如果您愿意拿一点点小小的心意资助，请扫下方二维码，鄙人不胜感激。

## 使用说明

**方法一：**

拖动链接到书签栏，[点击此处获取](https://doooreyn.github.io/wxreader.html)。

**方法二：**

1. 打开浏览器 _（目前仅在 Chrome 下测试 OK，其他浏览器还没来得及测试，移动平台无效）_；
2. 复制 [wx_auto_reader.min.js](./wx_auto_reader.min.js) 内容；
3. 在浏览器新建一份书签，名称任意，如：`微信读书自动阅读器`，网址改为`javascript:步骤2的复制内容`；
4. 打开微信读书网页版，打开任意一本书，点击书签栏 `微信读书自动阅读器`，启动阅读程序，挂机就完了；
5. 建议选择一本篇幅较大的书，这样可以达到长时间挂机的目的，不会对你的工作产生干扰。

## 声明

本仓库仅提供代码，一切责任由使用者自行承担。

![Buy me a cup of coffee](https://wu57.cn/images/wechatpay.png)
