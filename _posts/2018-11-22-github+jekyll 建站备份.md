
---

layout:     post # 使用的布局（不需要改）
title:      github+jekyll建站备份# 标题 
subtitle:   #副标题
date:       2018-11-22  # 时间
author:     zyz  # 作者
header-img: #这篇文章标题背景图片
catalog: true  # 是否归档
tags:        #标签
        - 技术

---

折腾了一天多，终于搭好了自己的博客，网上关于如何搭建博客的资料很详细了。这里做个简单的备份。
使用github + jekyll建立网站。
创建github仓库 命名为 账号名.github.io
在setting中github page页面选择jekyll 主题
挑选自己喜欢的jekyll模板，比如在 [jekyll theme](http://jekyllthemes.org/)中有很多优秀的第三方主题
（以上步骤也可改为在主页上选中某个模板后找到它的github仓库直接fork下来）
clone github仓库到本地
下载ruby
下载gem
gem install bundler
gem install jekyll
使用 ekyll serve -P $POST 进行本地预览
文章放在 _post 这一目录下，要以 yy-mm-dd-文章名.md的形式命名

开始的默认格式：

    ---
    layout:     post                    # 使用的布局（不需要改）
    title:      My First Post               # 标题 
    subtitle:   Hello World, Hello Blog #副标题
    date:       2017-02-06              # 时间
    author:     BY                      # 作者
    header-img: img/post-bg-2015.jpg    #这篇文章标题背景图片
    catalog: true                       # 是否归档
    tags:                               #标签
        - 生活
    ---


tips：
1. 目前选用了[next](https://github.com/Simpleyyt/jekyll-theme-next) 这一主题，需要使用bundle exec jekyll -P 进行浏览，第一次运行可能出现问题，需要运行bundle install 命令，之后耐心等待安装完成
2. 觉得其实hexo渲染的效果比jekyll更加美观流畅



