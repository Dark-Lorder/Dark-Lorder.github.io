---
title: Hello World
date: 2022-07-24 00:00:00
categories:
tags:
---
用户手册
<!-- more -->

## 用户手册
### 自动更新主题

``` bash
$ npm update            #  更新所有
$ npm update --save hexo-theme-fluid            # 更新主题
```

### 手动更新主题

修改根目录的package.json文件，将对应插件名称所对应的版本号更改为要更新的版本号。
``` bash
$ npm ls            # 查询当前项目已安装插件版本
$ npm install --save            # 更新
```

### 部署发布
``` bash
$ hexo clean            # 清理缓存 clean
$ hexo generate         # 构建 build
$ hexo server           # 本地启动 server
$ hexo deploy           # 自动部署 deploy
```

### 配置文件

``` yaml
"博客配置" 指的是 根目录下的 _config.yml
"主题配置" 指的是 theme/fluid/_config.yml
```

### 顶部大图

主题配置中，每个页面都有名为 banner_img 的属性，可以使用本地图片的相对路径，也可以为外站链接。
``` yaml
  banner_img: /img/default.png          # 对应存放在 /source/img/default.png
  banner_img: https://xxxx.com/img/default.png
  
# 高度
  鉴于每个人的喜好不同，开放对页面 banner_img 高度的控制。
  主题配置中，每个页面对应的 banner_img_height 属性，有效值为 0 - 100。100 即为全屏，个人建议 70 以上。
# 蒙版透明度
  主题配置中，每个页面对应的 banner_mask_alpha 属性，有效值为 0 - 1.0， 0 是完全透明（无蒙版），1 是完全不透明
```

### 博客标题

``` yaml
navbar:
  blog_title: 博客标题
```

### 导航菜单

``` yaml
navbar:
  menu:
    - { key: "home", link: "/", icon: "iconfont icon-home-fill" }
    - { key: "archive", link: "/archives/", icon: "iconfont icon-archive-fill" }
    - { key: "category", link: "/categories/", icon: "iconfont icon-category-fill" }
    - { key: "tag", link: "/tags/", icon: "iconfont icon-tags-fill" }
    - { key: "links", link: "/links/", icon: "iconfont icon-link-fill" }
    - { key: "about", link: "/about/", icon: "iconfont icon-user-fill" }
```
- key: 用于关联有语言配置，如不存在关联则显示 key 本身的值
- link: 跳转链接
- icon: 图标的 css class，可以省略（即没有图标），主题内置图标详见这里
- name: 强制使用此名称显示（不再按语言配置显示），可省略

另外支持二级菜单（下拉菜单），配置写法如下：
``` yaml
menu:
  - {
      key: '文档',
      icon: 'iconfont icon-books',
      submenu: [
        { key: '主题博客', link: 'https://hexo.fluid-dev.com/' },
        { key: '配置指南', link: 'https://hexo.fluid-dev.com/docs/guide/' },
        { key: '图标用法', link: 'https://hexo.fluid-dev.com/docs/icon/' }
      ]
  }
```

### 全局字体

设置单独的页面，可以直接在 markdown 里通过 style 标签实现：
``` yaml
---
title: example
---

<style>
  /* 设置整个页面的字体 */
  html, body, .markdown-body {
    font-family: KaiTi,"Microsoft YaHei",Georgia, sans, serif;
    font-size: 15px;
  }

  /* 只设置 markdown 字体 */
  .markdown-body {
    font-family: KaiTi,"Microsoft YaHei",Georgia, sans, serif;
    font-size: 15px;
  }
</style>
```

### 强制全局 HTTPS

这种情况可以在主题配置中开启此配置：
``` yaml
force_https: true
```

### 自定义 JS / CSS / HTML

如果你想引入外部的 JS、CSS（比如 IconFont）或 HTML，可以通过以下主题配置，具体见注释：
``` yaml
# 指定自定义 js 文件路径，路径是相对 source 目录
custom_js: /js/custom.js

# 指定自定义 css 文件路径，路径是相对 source 目录
custom_css: /css/custom.css

# 自定义 <head> 节点中的 HTML 内容
custom_head: '<meta name="key" content="value">'

# 自定义底部 HTML 内容（位于 footer 上方），也可用于外部引入 js css 这些操作，注意不要和 post.custom 配置冲突
custom_html: '<link rel="stylesheet" href="//at.alicdn.com/t/font_1067060_qzomjdt8bmp.css">'
```

### Slogan(打字机)

标题文字默认开启了打字机动效，相关配置如下：
``` yaml
fun_features:
  typing: # 为 subtitle 添加打字机效果
    enable: true
    typeSpeed: 70 # 打印速度
    cursorChar: "_" # 游标字符
    loop: false # 是否循环播放效果
```

### 文章摘要

若要手动指定摘要，使用 <!-- more --> MD文档里划分，如：
``` yaml
正文的一部分作为摘要
<!-- more -->
余下的正文
```
或者在 Front-matter 里设置 excerpt 字段，如：
``` yaml
---
title: 这是标题
excerpt: 这是摘要
---
```
- 无论哪种摘要都最多显示 3 行，当屏幕宽度不足时会隐藏部分摘要。

### 文章跳转方式

``` yaml
index:
  post_url_target: _blank
```
- _blank：新标签页打开
- _self：当前标签页打开

### 隐藏文章

如果想把某些文章隐藏起来，不在首页和其他分类里展示，可以在文章开头 Front-matter 中配置 hide: true 属性。
``` yaml
---
title: 文章标题
index_img: /img/example.jpg
date: 2022-07-24 00:00:00
hide: true
---
以下是文章内容
```
- 隐藏会使文章在分类和标签类里都不显示
- 隐藏后依然可以通过文章链接访问

### 文章排序

如果想手动将某些文章固定在首页靠前的位置，在文章开头 Front-matter 中配置 sticky 属性：
``` yaml
---
title: 文章标题
index_img: /img/example.jpg
date: 2022-07-24 00:00:00
sticky: 100
---
以下是文章内容
```
- sticky 数值越大，该文章越靠前，达到类似于置顶的效果，其他未设置的文章依然按默认排序。

### 文章在首页的封面图

对于单篇文章，在文章开头 Front-matter 中配置 index_img 属性。
``` yaml
---
title: 文章标题
tags: [Hexo, Fluid]
index_img: /img/example.jpg
date: 2022-07-24 00:00:00
---
以下是文章内容
```

### 文章页顶部大图

默认显示主题配置中的 post.banner_img，如需要设置单个文章的 Banner，在 Front-matter 中指定 banner_img 属性。
``` yaml
---
title: 文章标题
tags: [Hexo, Fluid]
index_img: /img/example.jpg
banner_img: /img/post_banner.jpg
date: 2022-07-24 00:00:00
---
以下是文章内容
```

### 文章内容本地图片

``` yaml
![](/img/example.jpg)
```

### 日期/字数/阅读时长/阅读数

显示在文章页大标题下的文章信息，除了作者和阅读次数，其他功能都是默认开启的。
``` yaml
post:
  meta:
    author:  # 作者，优先根据 front-matter 里 author 字段，其次是 hexo 配置中 author 值
      enable: false
    date:  # 文章日期，优先根据 front-matter 里 date 字段，其次是 md 文件日期
      enable: true
      format: "dddd, MMMM Do YYYY, h:mm a"  # 格式参照 ISO-8601 日期格式化
    wordcount:  # 字数统计
      enable: true
      format: "{} 字"  # 显示的文本，{}是数字的占位符（必须包含)，下同
    min2read:  # 阅读时间
      enable: true
      format: "{} 分钟"
    views:  # 阅读次数
      enable: false
      source: "leancloud"  # 统计数据来源，可选：leancloud | busuanzi   注意不蒜子会间歇抽风
      format: "{} 次"
```

### 脚注

将脚注写在文末，比如：
``` yaml
正文

## 参考
[^1]: 参考资料1
[^2]: 参考资料2
```

### Tag 插件

在 markdown 中加入如下的代码来使用便签
``` md
{% note success %}
文字 或者 `markdown` 均可
{% endnote %}

使用时 {% note primary %} 和 {% endnote %} 需单独一行，否则会出现问题
```

或者使用 HTML 形式：
``` html
<p class="note note-primary">标签</p>
```
<p class="note note-primary">primary</p>
<p class="note note-secondary">secondary</p>
<p class="note note-success">success</p>
<p class="note note-danger">danger</p>
<p class="note note-warning">warning</p>
<p class="note note-info">info</p>
<p class="note note-light">light</p>

### 行内标签

在 markdown 中加入如下的代码来使用 Label：
``` md
{% label primary @text %}

若使用 {% label primary @text %}，text 不能以 @ 开头
```

或者使用 HTML 形式：
``` html
<span class="label label-primary">Label</span>
```
<span class="label label-primary">primary</span> <span class="label label-default">default</span> <span class="label label-info">info</span> <span class="label label-success">success</span> <span class="label label-warning">warning</span> <span class="label label-danger">danger</span>

### 组图

如果想把多张图片按一定布局组合显示，你可以在 markdown 中按如下格式：
``` md
{% gi total n1-n2-... %}
  ![](url)
  ![](url)
  ![](url)
  ![](url)
  ![](url)
{% endgi %}

total：图片总数量，对应中间包含的图片 url 数量
n1-n2-...：每行的图片数量，可以省略，默认单行最多 3 张图，求和必须相等于 total，否则按默认样式
如下图为 {% gi 5 3-2 %} 示例，代表共 5 张图，第一行 3 张图，第二行 2 张图。
```

{% gi total n1-n2-... %}
![](/img/avatar.png)
![](/img/avatar.png)
![](/img/avatar.png)
![](/img/avatar.png)
![](/img/avatar.png)
{% endgi %}

### 标签页

标签是以词云的形式展示，标签的大小和颜色会根据标签下的文章数量变化，相关配置如下：
``` yaml
tag:
  tagcloud:
    min_font: 15
    max_font: 30
    unit: px  # 字号单位
    start_color: "#BBBBEE"
    end_color: "#337ab7"
```

### 友情链接页

友情链接页用于展示好友的博客入口，默认关闭，开启需要先在 navbar 项中将 links 的注释(#号)删掉：
``` yaml
navbar:
  menu:
    - { key: 'links', link: '/links/', icon: 'iconfont icon-link-fill' }
```
然后找到 links 的配置项，对页面内容进行配置：
``` yaml
links:
  items:
    - {
      title: 'Fluid Docs',
      intro: '主题使用指南',
      link: 'https://hexo.fluid-dev.com/docs/',
      avatar: '/img/favicon.png'
    }
  default_avatar: /img/avatar.png
```
- title: 友链站的标题
- intro: 站点或博主的简介，可省略
- link: 跳转链接
- avatar: 头像图片，可省略
- default_avatar: 成员的默认头像（仅在指定了头像并且加载失败时生效）

友链页也可以使用自定义区域和评论，使用方式类似于文章页，具体见配置项与相关注释。

### 自定义页面

如果想单独生成一个页面，步骤和创建「关于页」类似
先用命令行创建页面：
``` bash
$ hexo new page example
```
创建成功后编辑博客目录下 /source/example/index.md：
```yaml
---
title: example
subtitle: 若不填默认是 title
---

这里写正文，支持 Markdown, HTML
```
正文默认没有 Markdown 样式，如果希望和文章相同的样式，可以加上：
```html
<div class="markdown-body">
正文
</div>
```
页面的参数配置可以在主题配置中统一设置：
```html
page:
  banner_img: /img/default.png
  banner_img_height: 70
  banner_mask_alpha: 0.3
```
也可以直接在 Front-matter 里单独设置：
```yaml
---
title: example
banner_img: /img/default.png
banner_img_height: 60
banner_mask_alpha: 0.5
---

这里可以写正文
```

### Fluid 注入代码

进入博客目录下 scripts 文件夹（如不存在则创建），在里面创建任意名称的 js 文件，在文件中写入如下内容：
``` js
hexo.extend.filter.register('theme_inject', function(injects) {
  injects.header.file('default', 'source/_inject/test1.ejs', { key: 'value' }, -1);
  injects.footer.raw('default', '<script async src="https://xxxxxx" crossorigin="anonymous"></script>');
});
```
- header 和 footer 是注入点的名称，表示代码注入到页面的什么位置；
- file 方法表示注入的是文件，第一个参数下面介绍，第二个参数则是文件的路径，第三个参数是传入文件的参数（可省略），第四个参数是顺序（可省略）；
- raw 方法表示注入的是原生代码，第一个参数下面介绍，第二个参数则是一句原生的 HTML 语句；
- default 表示注入的键名，可以使用任意键名，同一个注入点下的相同键名会使注入的内容覆盖，而不同键名则会让内容依次排列（默认按执行先后顺序，可通过 file 第四个参数指定），这里 default 为主题默认键名，通常会替换掉主题默认的组件；

主题目前提供的注入点如下：

| 注入点名称        | 注入范围                                   | 存在 `default` 键 |
| ----------------- | ------------------------------------------ | ----------------- |
| head              | `<head>` 标签中的结尾                      | 无                |
| header            | `<header>` 标签中所有内容                  | 有                |
| bodyBegin         | `<body>` 标签中的开始                      | 无                |
| bodyEnd           | `<body>` 标签中的结尾                      | 无                |
| footer            | `<footer>` 标签中所有内容                  | 有                |
| postMetaTop       | 文章页 `<header>` 标签中 meta 部分内容     | 有                |
| postMetaBottom    | 文章页底部 meta 部分内容                   | 有                |
| postMarkdownBegin | `<div class="markdown-body">` 标签中的开始 | 无                |
| postMarkdownEnd   | `<div class="markdown-body">` 标签中的结尾 | 无                |
| postLeft          | 文章页左侧边栏                             | 有                |
| postRight         | 文章页右侧边栏                             | 有                |
| postCopyright     | 文章页版权信息                             | 有                |
| postRight         | 文章页右侧边栏                             | 无                |
| postComments      | 文章页评论                                 | 有                |
| pageComments      | 自定义页评论                               | 有                |
| linksComments     | 友链页评论                                 | 有                |




**参考资料**
[Hexo Fluid GitHub](https://github.com/fluid-dev/hexo-theme-fluid)
[Hexo Fluid 用户手册](https://hexo.fluid-dev.com/docs/)
