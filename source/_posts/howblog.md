---
title: Hexo + Next + GitHubPage搭建个人blog
date: 2019-03-24
tags: [Next, Hexo]
categories: Blog搭建
---
这周末，折腾了很长时间，终于在GitHub上基于[Hexo](https://hexo.io)搭建了个人网页，用的是比较热门的[NexT](https://theme-next.org/)主题。
<!-- more -->
# 参考
## 官方文档
* [Hexo的文档](https://hexo.io/docs/)
* [Hexo主题之——NexT的说明文档](http://theme-next.iissnan.com/)
(这个说明文档有些过时，很多地方不符合当前Next版本:V7.0.1)

## 个人博客教程
* [Hexo之Next主题优化整理_Michael翔](http://michael728.github.io/2015/11/30/hexo-next-optimize/)
* [【Hexo搭建独立博客全纪录】_baoyuzhang](https://baoyuzhang.github.io/2017/05/12/%E3%80%90Hexo%E6%90%AD%E5%BB%BA%E7%8B%AC%E7%AB%8B%E5%8D%9A%E5%AE%A2%E5%85%A8%E7%BA%AA%E5%BD%95%E3%80%91%EF%BC%88%E4%B8%89%EF%BC%89%E4%BD%BF%E7%94%A8Hexo%E6%90%AD%E5%BB%BA%E5%8D%9A%E5%AE%A2/)
* [Github Pages + Hexo 博客搭建，Next主题个性化修改](https://www.lixint.me/hexo-blog.html)
(这个教程挺好，最近有更新，里面有关Next主题的修改比较符合当前的版本)


# Hexo和NexT主题的安装
## Hexo
### Hexo是啥？
Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用[Markdown](https://daringfireball.net/projects/markdown/)（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。
### 安装
在安装前一定给这hexo单独一个空间，比如，我放在 `D:\workspace\githubpage\hexo`中。
Hexo的安装具体可以参看[Hexo的文档](https://hexo.io/docs/)
首先需要安装
* [Node.js](https://nodejs.org/)
* [Git](https://git-scm.com/)

这里主要说一下在安装过程中出现的问题。
在安装过程中，由于连接国外网站，速度相当感人，所以在这里用淘宝源替换,并安装hexo
```shell
# 添加淘宝源
npm install -g cnpm --registry=https://registry.npm.taobao.org
```
之后用法是和npm一样的，只不过把`npm install`用`cnpm install`代替
## NexT主题
Next 主题我也不多说了，那些参考文档里说的都很详细。
我主要说说我在用的过程中遇到的问题。
### 把NexT主题clone到哪里？
我们把hexo安装好后，进去hexo的文件夹，可以看到有一个`themes`文件夹，next就是放在这个文件夹下的，其实[next仓库](https://github.com/theme-next/hexo-theme-next)的readme文件已经写得很清楚了(捂脸):
```
$ cd hexo
$ git clone https://github.com/theme-next/hexo-theme-next themes/next
```
### 怎么切换主题风格
在`\themes\next`中找到`_config.yml`文件找到`Schemes Settings`，想使用哪种风格，把其他三种注释掉即可。
```
# ---------------------------------------------------------------
# Scheme Settings
# ---------------------------------------------------------------

# Schemes
#scheme: Muse
#scheme: Mist
#scheme: Pisces
scheme: Gemini
```
### 怎么把侧边栏的头像设置成圆形？
找到`/themes/next/source/css/_common/components/sidebar/sidebar-author.styl`，修改如下：
```
.site-author-image {
  display: block;
  margin: 0 auto;
  padding: $site-author-image-padding;
  max-width: $site-author-image-width;
  height: $site-author-image-height;
  border: $site-author-image-border-width solid $site-author-image-border-color;
  opacity: hexo-config('avatar.opacity') is a 'unit' ? hexo-config('avatar.opacity') : 1;
  #加入下面5行，把头像设置成圆形（这行别复制进去）
  border-radius: 50%
  webkit-transition: 1.4s all;
  moz-transition: 1.4s all;
  ms-transition: 1.4s all;
  transition: 1.4s all;
}
```
### 在底部和文章页面加上访问量
打开主题配置文件`/themes/next/_config.yml`,搜索`busuanzi_count`,将对应的`false`改为`true`,`total`项为站点总统计，显示在站点首页底部，分为`total_visitors`：总访问人数，`total_views`：总访问量。`post_views`为文章访问量，显示在文章页面的标题下方。
```
busuanzi_count:
  enable: true
  total_visitors: true
  total_visitors_icon: user
  total_views: true
  total_views_icon: eye
  post_views: true
  post_views_icon: eye
```
**注意**：不蒜子之前更新过一次，Next主题如果设置后不生效请检查一下`themes\next\layout\_third-party\analyticsbusuanzi-counter.swig`文件中的链接是否是https://busuanzi.ibruce.info/busuanzi/2.3/busuanzi.pure.mini.js

### 添加README.md文件
hexo部署时候，会把source下的*.md文件全部解析成网页文件，所以使用hexo部署到仓库后，是没有`README.md`文件的。有两种方法可以解决这个问题：
1. `\hexo\source`根目录下建一个`README.MDOWN`文件，GitHub是可以将其识别为markdown文件的，hexo却不会将其解析。
2. `\hexo\source`根目录下建一个`README.md`文件，然后修改站点配置文件`_config.yml`，将`skip_render`参数的值设置为:
```
skip_render: README.md
```
显然，第二种方法是比较**优雅**的做法

### 将标签图标改掉
在文章下方，标签前的图标是个#号，想改成那种标签的图标，怎么改呢？
> 修改`/themes/next/layout/_macro/post.swig`文件，找到`rel="tag">#`，将`#`换成`<i class="fa fa-tag"></i>`

我查的答案中，只给了上面这一句，我照着操作后，就成功了，但是，为什么需要这样子改呢？为什么改了一行代码，#就变成了一个图标呢？这激起了我的好奇心，于是就开动搜索引擎，一顿操作。没搜到确切的答案，但找到了一些相关资料，可以推断出来为什么要这样改。
资料如下（来源：[知乎-Hexo博客之主题美化](https://zhuanlan.zhihu.com/p/28360099)）

> ### 侧边栏图标设置
 NexT网站已经告诉我们如何为博客添加社交链接并设定链接图标，但是我们国内一些常用的社交网站如知乎、豆瓣等在 Font Awesome 图标库中没有对应的图标，结果就是我们添加的这些社交链接图标都是默认的。在主题配置文件中，设定链接图标对应的字段是 social_icons， 其键值格式是 匹配键名: 图标名称， 图标名称 就是对应的 Font Awesome 图标的名字（不必带 fa- 前缀）。实现方法：到 Font Awesome 图标库中找寻找喜爱的图标，复制图标的名字替换下面代码箭头处的默认图标名字即可。如下所示：
 ```
 social:
  GitHub: https://github.com/username
  知乎: https://www.zhihu.com/people/feng-di-54-18/activities

 social_icons:
  enable: true
  icons_only: false
  transition: false
  # Icon Mappings.
  # KeyMapsToSocialItemKey: NameOfTheIconFromFontAwesome
  GitHub: github  <---
  知乎: heart-o   <---
 ```
 哦~原来`fa`的就是`Font Awesome`的意思啊！NexT是可以使用[Font Awesome](https://fontawesome.com/icons)图标库中的图标的。
知道了这一点，我们就不难知道，为什么仅仅将`#`换成`<i class="fa fa-tag"></i>`就可以了。


### 设置“阅读原文”按钮
最简单的做法，在想分割的地方，加入一行
```
<!-- more -->
```

# 将写好的东西部署到GitHub Page上
经过上面的的步骤，在本地`\hexo`文件夹下，使用`hexo s`命令后，可以看到：
```
INFO  Start processing
INFO  Hexo is running at http://localhost:4000 . Press Ctrl+C to stop.
```
在浏览器中输入`http://localhost:4000`就可以看到页面了。
在这里，如果4000端口被占用，可以用`hexo server -p 端口号`手动指定端口。
可以看到自己blog的样子了，怎么将其部署到GitHub页面上呢？

## 创建GitHub Page页面
具体可以参看[GitHub Pages](https://pages.github.com/)
创建一个repo，名字一定要命名为：`你的github用户名.github.io`
然后本地的git设置，怎么将本地仓库和远程仓库连接起来，怎么实现SSH推送，这些，我都不讲了，在其他教程里很清楚，而且，我懒啊~
好吧，在这里推荐一个[Git教程](https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000)

## 打开hexo配置文件`_config.yml`
最后一行找到 Deployment,有关配置如下，repo后面填上你刚刚创建的仓库名，比方我的，就是下面这个，branch后面填上master，也就是默认的主分支。
```
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: https://github.com/gtbaby/gtbaby.github.io.git
  branch: master
```
在hexo文件夹下，执行命令`hexo clean && hexo g -d`,稍作等待即可。

插一个我的疑问：
在hexo的[文档](https://hexo.io/docs/)中，关于部署的部分，这样说到：
>### How does it work exactly?
>Your repository will have a master branch when you first made it. Keep working on this branch to create your site. When you deploy Hexo will create, or update, a new branch on the remote site (called published in the config above). Deployment won’t create a new branch locally, nor will it mess with your existing source code in the master branch locally or on the remote. You still need to keep pushing commits to the master branch manually to the remote server to keep your site backed up. In addition, if you are using a CNAME file to customize your Github Pages domain name, you need to put the CNAME file under source_dir so that Hexo can push it to the published branch.

他说这个部署分支应该和文章版本控制的分支是不同的，部署分支用`published`。但是，问题就出在这里！GitHub Pages文档中明确说到，在GitHub页面上只显示master分支的内容。推送到`publish`分支是没有用的！
我照着这个说明弄了半天，还是没成功，把publish分支设置为默认分支也没有用。

# Hexo 常用命令总结

## 常用命令
具体命令可参考[hexo commands](https://hexo.io/docs/commands)
 命令 | 缩写 |含义
 ---  | --- | ---
 `hexo new` | `hexo n` | 新建文章
 `hexo generate` | `hexo g` | 生成静态页面至public目录
 `hexo server` | `hexo s` | 开启预览访问端口（默认端口4000，'ctrl + c'关闭server）
 `hexo deploy` | `hexo d` | 部署到GitHub
 `hexo help` | - | 查看帮助
 `hexo version` | - | 查看Hexo版本
 `hexo clearn` | - | 清理缓存文件 (db.json)和生成的文件夹 (public)

## 组合命令
```
hexo s -g #生成并本地预览
hexo d -g #生成并上传
```

# 结语
我的搭建和优化过程大抵如此，很多细节其他教程写的很清楚了，我就不写了（其实是懒）。
后面遇到新问题，会时不时更新。
其实，重要的不是外观，而是这里面的文章质量，费半天劲去美化，没多大必要。建个站点，就是为了将学到的知识输出，向他人分享知识的同时，巩固自己已学的知识。
一起学习进步吧！笔芯~