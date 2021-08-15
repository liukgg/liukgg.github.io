---
title: hexo在matery主题下集成utteranc评论插件
date: 2021-08-15 21:15:06
tags: Hexo-Matery
categories: Reference
---

hexo在matery主题下集成utteranc评论插件
--------
本博客使用的是hexo搭建，主题选择的是matery。（感兴趣的可参考第一篇博客。）

matery主题的介绍和使用详见：https://github.com/blinkfox/hexo-theme-matery.

我选择该主题的主要原因是比较喜欢其界面风格、整体感觉比较优雅，且功能较为完整。
在此感谢该主题作者的分享 :)

为了能够和各位更好地交流，希望引入评论插件。找了一圈以后，想要用 utteranc 这个工具：
https://utteranc.es       (https://github.com/utterance/utterances)

然而在网上找了下集成的方法，发现基本上只介绍了hexo下的fluid主题和next，对于matery没有找到介绍。

所以这里针对matery主题如何集成 utteranc插件做一下简单分享。

# 前置准备：准备一个代码仓库并配置好 utteranc
该部分内容，网上有详细教程，本文不再赘述。（直接google或者百度搜索 hexo 集成 utteranc）

可以参考：https://www.jianshu.com/p/785d727810b3

在此，也对该文章的作者表示感谢 :)

# 在自己的博客中集成 utteranc
> 注意以下前置条件：
> 1. 已经有一个博客系统，基于hexo搭建，使用matery主题。
> 2. 完成前述步骤：有一个公开的github仓库，用来支持评论。（因为utteranc的评论实际上是会记录到github仓库的issue中。）
> 3. 完成前述步骤：配置好 utteranc，并复制其提供的一段js代码，类似下面这样：

```javascript
<script src="https://utteranc.es/client.js"
        repo="liukgg/comment-utterances"
        issue-term="pathname"
        theme="github-light"
        crossorigin="anonymous"
        async>
</script>
```

<br/>

接下来，正式开始在hexo搭建的博客、且选择 matery 主题的情况下，集成utteranc 以实现评论功能。

### 主要改动：修改 matery 主题中的相关文件
改动文件目录基本都在这个下面： themes/hexo-theme-matery/

##### 改动第1个文件： themes/hexo-theme-matery/_config.yml
加入以下配置：

```yaml
# utteranc config, default disabled
# utteranc 评论模块的配置，默认为不激活
utteranc:
  enable: true
```
该配置的目的主要是和 matery 原来的引入评论插件保持一致，保持灵活性。

##### 改动第2个文件: themes/hexo-theme-matery/layout/_partial/post-detail.ejs
  加入以下配置(在165行左右，参考其他评论插件的代码位置)：
  
```javascript
    <% if (theme.utteranc && theme.utteranc.enable) { %>
    <%- partial('_partial/utteranc') %>
    <% } %>
```

该处改动的目的是为了在所有博客文章中统一在底部加入一个评论模块，这个改动很关键、所放位置也很重要。
post-detail.ejs 是所有博客文章渲染的模板，因此需要在此处统一改动。

##### 改动第3个文件：新增一个文件 themes/hexo-theme-matery/layout/_partial/utteranc.ejs
该文件内容如下：

```javascript
<div class="card" data-aos="fade-up">
    <div id="utteranc-container" class="card-content">
        <script src="https://utteranc.es/client.js"
                repo="liukgg/comment-utterances"
                issue-term="pathname"
                theme="github-light"
                crossorigin="anonymous"
                async>
        </script>
    </div>
</div>
```
该文件就是评论模块的真实内容，核心内容是在前述步骤中复制的 utteranc 配置。
此外，外面加了2个div模块，主要是为了在页面展示的时候加入一个方块、和文章风格保持基本一致。
（对于这个div里的样式具体是如何生效的，没有做过多探究，基本上是参考另一个评论插件 gitalk 的代码。）

##### 小结
以上，就已经集成好了utteranc评论。重新启动 hexo server，刷新页面后，就能在文章下放看到评论模块了。

效果可以参考：http://luckliu.xyz/2021/08/01/hello-world/

所有的留言都会出现在前述配置的github 仓库中，可以参考：https://github.com/liukgg/comment-utterances/issues

### 其他
如果博客中还有单独的一个留言板模块，那么还需要改动一下这个文件： themes/hexo-theme-matery/layout/contact.ejs

加入以下代码(136行附近)：

```javascript
<% if (theme.utteranc && theme.utteranc.enable) { %>
    <%- partial('_partial/utteranc') %>
<% } %>
```

重启服务 hexo server，刷新页面以后，就能看到留言板也能留言评论了。

效果可以参考：http://luckliu.xyz/contact

如有问题，欢迎在我以上的博客留言板中留言交流。
