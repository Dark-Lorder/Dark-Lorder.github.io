---
title: iPhone今天这个大漏洞，让人打开App就被盗刷一万五。。。
date: 2023-07-24 21:18:14
categories:
tags:
---
微信零钱每日限额一万，微笑
<!-- more -->

众所周知，苹果一直在宣传自己的安全、隐私、可控。

而且在过去，它也足够的安全。

但世界上也没有绝对坚固的墙，大家也别因为这样，就彻底放松警惕。

因为今天，它就翻车了。。。

<center><img src="https://img.darklorder.com/img/202307311242237.gif"  alt="文件名" style="zoom:100%;" /></center>

事情是这样的，一个网友的丈母娘被 App Store 应用商店里的 “骗子 App” 骗走了一万五千元。

直到目前，他们被盗刷的钱都没能追回。

而我在复盘完事情的完整经过之后发现，其中两个重要环节，都是苹果出现了问题。

但凡苹果在这两个环节中的任何一个环节有安全措施，也不至于让骗子得逞。

先跟大家聊聊大概发生了什么事吧，大家可以先看看原博主 airycanon 描述的事情经过~

上下滑动可以查看完整长图

[图片](https://img.darklorder.com/img/202307311319978.jpg)

简单来说，就是 airycanon 的丈母娘从 AppStore 应用商店里下载了一个菜谱 App。

但是这个 App 本身有问题，打开 App 之后，首先会弹出一个 Apple 账号登录界面。

<img src="https://img.darklorder.com/img/202307311248469.png"/>

用过 iPhone 的小伙伴都知道，不少 App 都支持使用苹果账号一键注册登录，就像微信小程序一键登录一样。

**所以这一步看起来是合理的。**

但其实，真正的苹果一键登录界面，是这个样子的：

大家可以上下对比看看

<img src="https://img.darklorder.com/img/202307311248753.png"/>

菜谱 App 弹出的那个“假登录界面”，其实也是有用的，**它在后台已经完成了一次苹果账号登录。**

至于有什么用途，我们放到后面再说。

登录弹窗之后，这个 App 又弹出一个仿照系统界面设计的密码输入界面。

因为长得和安装应用时候的密码验证界面很像，手机用的不那么灵光的老人还是很容易中招的。

App"L"eID。。。

<img src="https://img.darklorder.com/img/202307311249332.png"/>


自此，骗子就搞到了受害者 iPhone 的账号和密码。

假如用户给苹果账号绑定了支付方式 —— 就比如 airycanon 的丈母娘绑定了微信免密支付。。。

那骗子就可以开始盗号刷刷刷了。

u1s1，这个骗子绝对是个惯犯。**他为了避免受害人收到短信交易提醒，盗刷之前甚至还利用查找 iPhone 远程把受害者的手机给重置恢复出厂设置了。。。**

这要是没用 iCloud 备份相册的人，不得气疯了。。。

**真·砂仁猪心。**

OK，事情大概就是这么一回事，讲道理，看完之后我整个人就是一个大写的懵。

首先，我倒是理解这种骗人 App 能堂而皇之在苹果官方 App Store 上架。。。

**因为骗过苹果商店审核的操作在业内根本不算什么秘密。**

马甲包、同 ID 双版本、幸运按钮。。。黑产总有办法。

<img src="https://img.darklorder.com/img/202307311250825.png"/>


比如我也在 App Store 里下载了几个菜谱大全，他们倒是没有盗我的密码，但是点开以后也都不是菜谱。

**而是一个个关不掉的强制弹窗，“ 帮 ” 我开各种彩铃包、权益合约包。。。**

<img src="https://img.darklorder.com/img/202307311250766.png"/>

难得遇到了一个正经菜谱 App，结果刚看了两个菜，就要收我 28。。。

**不对，是每周 28 块钱。。。**

我估计正经厨房类 App 的产品经理们都得看傻了：同行们黑心钱都这么好赚的吗？

“ 非强制消费”

<img src="https://img.darklorder.com/img/202307311250241.png"/>

但是，就算 App 能通过钓鱼的方式骗到受害者苹果账户的密码，但是苹果本身是有 “ 两步验证 ” 机制的呀？

在登录新设备或者浏览器的时候，除了输入密码，苹果还会要求输入一个短信验证码。

而且 airycanon 也在帖子里说明了，他丈母娘的 Apple ID 已经开启了两步验证。

但是他们自始至终没有收到苹果的验证码。

<img src="https://img.darklorder.com/img/202307311251965.png"/>

这时候他发现，**丈母娘账号的两步验证设置里，多出来了一个从来没见过的境外号码。**

怪不得自己手机上没有验证码了，**因为接收验证码的手机已经变成骗子的手机了。。。**

考虑到设置两步验证是一个挺复杂的操作、即使手把手跟丈母娘说都不一定能设置成功。

那这个号码只能是骗子偷偷添加进来的了。。。

<img src="https://img.darklorder.com/img/202307311251618.png"/>

这就很离谱了好吧，因为虽然“菜谱骗子”们骗过了苹果的 App 审核，**但它们最多也只是诈骗，是在玩弄社会工程学，而不是病毒。**

**理论上来说，它们根本没有办法绕过苹果最根本的安全措施，在不弹任何验证码的情况下往苹果的双证验证系统里加入****能收验证码的新手机号****。。。**

这一点是我和编辑部小伙伴们都感觉非常诧异的，也是今天关注到这件事的网友们讨论最激烈的。

不过在一段讨论之后，研究苹果开发的 BugOS 技术组提到了一种可能的思路：

<img src="https://img.darklorder.com/img/202307311252605.png"/>

上面截图里的内容大家看不懂没关系，简单来说，**苹果浏览器框架的安全策略出了问题。**

事情大概是这么回事：我们都知道，不管是 iPhone，还是安卓手机，系统里都会有一个预装的默认网页浏览器对不对。

<img src="https://img.darklorder.com/img/202307311252308.png"/>

比如 iPhone 上就是 Safiri，安卓这边则是各家的自带浏览器。

但这其实只是表面上的浏览器。

但其实，**再往系统底层找，还有一个“没有图标”的浏览器框架：WebView。**

<img src="https://img.darklorder.com/img/202307311252723.png"/>

这个 “ 浏览器框架 ” 不能被普通用户在手机里直接点开，它存在的意义，是供其他 App 调用的。

我们举个例子，就比方说美团、滴滴他们经常在 App 里搞领券的活动，对于这种临时的活动页面一般就是写个网页。

<img src="https://img.darklorder.com/img/202307311253933.png"/>

这些 “ App 内的网页 ”，实际上都不是 App 本身在渲染，而是美团和滴滴拉起了系统里的 WebView 组件来进行渲染的。

这个组件其实帮了开发者很大的忙，假如没有 WebView 浏览器框架的话，包括美团和滴滴在内的所有 App 开发者，都得往 App 里再额外集成一个独立的浏览器内核。

本身现在的 App 们就已经很占存储空间了，要是一人再背一个 Chrome。。。

画面太美了，我不敢想！

另外，作为网络世界的窗口，浏览器漏洞本身也是很多黑客行为的突破口。

系统本身提供一个全局自动更新的浏览器框架，也可以避免一些 App 不更新内置的浏览器内核，导致黑客趁虚而入。

**这次的问题，恰恰就出在这个 “为了不出问题” 而设计的系统级浏览器框架上。**

不知道大家有没有体验过系统浏览器的 “**便捷单点登录**” 功能。

就比方说，假如你在 Windows 电脑上使用自带的 Edge 浏览器打开微软账户官网，Edge 浏览器不会让你输入微软账户的账号密码。

而是会自动读取当前电脑里登录的微软账户，然后帮你在浏览器网站里完成登录。

<img src="https://img.darklorder.com/img/202307311253625.png"/>

假如你在登录了谷歌账号的安卓手机上使用谷歌 Chrome 浏览器，它也会自动帮你完成登录操作。

<img src="https://img.darklorder.com/img/202307311254951.png"/>

苹果也是如此，假如你在 Safari 浏览器里打开 Apple ID 官网，并点击登录。

浏览器也不会让你输入密码，而是直接弹出来一个登陆操作的确认框。

<img src="https://img.darklorder.com/img/202307311254573.gif"/>

假如你点了 “ 继续 ”，得益于高性能的苹果 A16 处理器，系统会光速弹出 Face ID 验证。

一个眨眼的功夫，就登录成功了。

<img src="https://img.darklorder.com/img/202307311255122.png"/>

诶，等会儿。。。

**这个登陆框，怎么有点儿眼熟啊？？？**

**为什么 “ 菜谱大全 ” 会请求登录 Apple ID 官网啊？？？**

<img src="https://img.darklorder.com/img/202307311255195.png"/>

说真的，假如没有对比的话，菜谱大全的操作很容易会被大家当成是普通的 “ 一键注册账号 ” ——

包括发帖的 airycanon 也没反应过来，以为是丈母娘没有选苹果的隐私邮箱登录选项才暴露了 Apple ID，让黑客掌握了信息。

真正的一键注册环节会要求选择是否隐藏邮件地址

<img src="https://img.darklorder.com/img/202307311256034.png"/>

实际上，当这个确认窗验证完毕之后，骗子都已经准备好往账号里加料了。。。

<img src="https://img.darklorder.com/img/202307311256164.png"/>

“ 菜谱大全 ” 之所以能够一键登录，恰恰就是利用了 WebView 这个系统级浏览器框架的 “ 便捷登录 ” 特性。

表面上，是一个菜谱 App，**而在它的内部，隐藏了一个正在访问 Apple ID 官网、并准备篡改用户接收验证短信手机号的浏览器界面。**

我后来看 BugOS 技术组又发了一个微博，他们已经用自己写的代码还原完整个攻击过程了。

<img src="https://img.darklorder.com/img/202307311256161.png"/>

按照苹果 Apple ID 官网目前的安全逻辑，**只有一开始的账号登录环节需要下发验证码做双重验证。**

而这最开始的一步，骗子已经通过 WebKit 的便捷登录绕过去了。

**已经处于登录状态的情况下，只要输固定的账号密码，就可以直接添加新的验证手机了。**

<img src="https://img.darklorder.com/img/202307311257506.gif"/>

现在相信大家已经彻底搞明白背后是怎么一回事儿了，这时候我们再重新回看故事的全貌：

“ 菜谱大全 ” 先是在表层界面的下面，隐藏了一个 WebView 浏览器组件，然后利用它系统级的 “便捷登录” 能力，进入了 Apple ID 官网。

接着，它给用户弹出了一个密码输入框，用来搞定添加验证手机的最后一步障碍。

<img src="https://img.darklorder.com/img/202307311258021.png"/>

拿到密码之后，App 就会偷偷跑起添加新验证手机的自动脚本，这时候受害者的苹果账号就已经不属于自己了。

什么时候发起攻击，全看黑客心情了。

OK，复盘完毕，这么一看好像还是受害者太傻，平白无故把密码交出来了对不对 —— 假如受害者打死不填密码，黑客也没招。

我们不应该骂苹果对不对？

emmmm，在下这个结论之前，我想先带大家看一看苹果的老对手 —— 谷歌是怎么做的。

<img src="https://img.darklorder.com/img/202307311258470.png"/>

和苹果 Apple ID 一样，只要已经处于登录状态了，谷歌这边的账号系统要想添加新的验证手机，也只是输一下固定密码的事。

**但是和苹果不同，谷歌根本不允许系统的 WebView 组件使用 “便捷登录” 技术。**

我在自己的安卓手机上做了个小测试，分别使用谷歌 Chrome 浏览器和 Via 浏览器（ 一款直接调用系统 WebView 框架的极简浏览器 App ）访问谷歌账号官网。

Chrome 浏览器因为已经获取了系统里的账号登录状态，因此直接就登录了。

Via 浏览器则没有这个能力，需要一步步重新输入账号、密码、验证码。

<img src="https://img.darklorder.com/img/202307311259604.png"/>

换句话说，假如有骗子想在安卓手机上做一个同样套路的事，第一步就卡住了。。。

但是在苹果系统里，不管是调用 WebView 的 Via，还是真正的自带浏览器 Safari，**都能调用便捷登录。**

再搭配上 App Store 的审核漏洞，一锅好菜就出炉了。。。

<img src="https://img.darklorder.com/img/202307311259869.png"/>

严格来说，这对于 iOS 系统来说也算是一个漏洞 —— 它不是代码漏洞，而是一个逻辑漏洞。

骗子利用苹果开放的便捷登录能力，伪装了自己一波，再利用一点点社工技巧，就把钱骗到手了。

由于系统逻辑漏洞造成的问题，正确的解决方式应该是着手准备 OTA 补丁，同时帮着受骗的用户追回损失。

不过苹果给这个受害者带来的感知，并不是很好。。。

<img src="https://img.darklorder.com/img/202307311259025.png"/>

可能现在时间还比较短，希望苹果后续可以帮这个受害者妥善解决。

**不瞒大家说，本来我今天是没打算写这篇文章的。**

因为真要细究的话，安卓这边虽然把 WebView 的洞补上了，但是其他的漏洞更多、骗人的方式根本数不过来。

苹果生态总体来说都还是更安全的、让人用着更放心的，但是大家不要因为它 “安全” 的标签就变得麻木、不重视安全了。

就像大家都说沃尔沃安全，但你不能因为这点就不握着方向盘了。。。

相信很多给家长买 iPhone 的小伙伴，都是希望家长尽可能不被骗。

但我觉得，我们还是要告诉他们即便是 iPhone，即便是 App Store，也不能保证绝对安全。

不随便输密码、不给所有 App 用一模一样的密码是最后的底线。

**千万记得叮嘱他们，免密支付能不开就不开。** **如果开了的话，免密支付的卡里面也不要放太多的钱。**

**参考资料**
[iPhone今天这个大漏洞，让人打开App就被盗刷一万五。。。](https://mp.weixin.qq.com/s/G55w5UakMUcuhWyUPaHFYQ)
