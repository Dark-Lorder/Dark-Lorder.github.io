---
title: Writing Article
date: 2022-07-31 00:00:00
categories:
tags:
---
写作
<!-- more -->

## 
### 写作

你可以执行下列命令来创建一篇新文章或者新的页面。
``` bash
$ hexo new [layout] <title>
```
- 您可以在命令中指定文章的布局（layout），默认为 post，可以通过修改 _config.yml 中的 default_layout 参数来指定默认布局。

### 布局

Hexo 有三种默认布局：post、page 和 draft。在创建这三种不同类型的文件时，它们将会被保存到不同的路径；而您自定义的其他布局和 post 相同，都将储存到 source/_posts 文件夹。

|  布局   | 路径    |
|:---:|:----|
|  post   | source/_posts    |
|  page   | source    |
|  draft   | source/_drafts    |



### 草稿

刚刚提到了 Hexo 的一种特殊布局：draft，这种布局在建立时会被保存到 source/_drafts 文件夹，您可通过 publish 命令将草稿移动到 source/_posts 文件夹，该命令的使用方式与 new 十分类似，您也可在命令中指定 layout 来指定布局。
``` bash
$ hexo publish [layout] <title>
```
- 草稿默认不会显示在页面中，您可在执行时加上 --draft 参数，或是把 render_drafts 参数设为 true 来预览草稿。

### Front-matter

Front-matter 是文件最上方以 ------ 分隔的区域，用于指定个别文件的变量，举例来说：
``` bash
---
title: Hello World
date: 2022-07-24 00:00:00
---
```
- 以下是预先定义的参数，您可在模板中使用这些参数值并加以利用。

| 参数              | 描述                                                         | 默认值                                                       |
| :---------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| `layout`          | 布局                                                         | [`config.default_layout`](https://hexo.io/zh-cn/docs/configuration#文章) |
| `title`           | 标题                                                         | 文章的文件名                                                 |
| `date`            | 建立日期                                                     | 文件建立日期                                                 |
| `updated`         | 更新日期                                                     | 文件更新日期                                                 |
| `comments`        | 开启文章的评论功能                                           | true                                                         |
| `tags`            | 标签（不适用于分页）                                         |                                                              |
| `categories`      | 分类（不适用于分页）                                         |                                                              |
| `permalink`       | 覆盖文章网址                                                 |                                                              |
| `excerpt`         | Page excerpt in plain text. Use [this plugin](https://hexo.io/docs/tag-plugins#Post-Excerpt) to format the text |                                                              |
| `disableNunjucks` | Disable rendering of Nunjucks tag `{{ }}`/`{% %}` and [tag plugins](https://hexo.io/docs/tag-plugins) when enabled |                                                              |
| `lang`            | Set the language to override [auto-detection](https://hexo.io/docs/internationalization#Path) | Inherited from `_config.yml`                                 |

### 删除
```
删除文件夹source/_posts下目标文章markdown文件
删除.deploy_git文件夹
执行hexo cl 后，再执行 hexo g ，hexo s 即可。
```


### 分类

创建 分类 页并添加 tpye 属性
``` bash
$ hexo new page categories
```

source/categories/index.md 添加 categories 属性
``` md
---
title: categories
date: 2022-07-24 00:00:00
type: "categories"
---
```

给文章添加 categories 属性
``` md
---
title: 写作
date: 2022-07-31 00:00:00
categories:
- Hello World
---
```

- 打开需要添加分类的文章，为其添加categories属性。
- 下方的categories: Hello World 表示添加这篇文章到 Hello World 这个分类。
- 注意：hexo一篇文章只能属于一个分类，也就是说如果在 "- Hello World" 下方添加 "-xxx"，hexo不会产生两个分类，
- 而是把分类嵌套（即该文章属于 "- Hello World" 下的 "-xxx" 分类）。

### 标签

创建 标签 页并添加 tpye 属性
``` bash
$ hexo new page tags
```

source/tags/index.md 添加 tags 属性
``` md
---
title: tags
date: 2022-07-24 00:00:00
type: "tags"
---
```

给文章添加 tags 属性
``` md
---
title: 写作
date: 2022-07-25 00:00:00
categories:
- Hello World
tags:
- Hello 
- World
---
```

- 打开需要添加标签的文章，为其添加tags属性。下方的tags:下方的- Hello - World 就是这篇文章的标签了
- 至此，成功给文章添加分类，点击首页的 "标签" 可以看到该标签下的所有文章。
- 当然，只有添加了tags: xxx的文章才会被收录到首页的 "标签" 中。