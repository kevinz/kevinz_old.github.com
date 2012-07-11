---
layout: post
title: "Fix octopress git deploy issuse"
date: 2012-07-12 07:05
comments: true
categories: [tool]
description: "解决octopress的git自动deploy问题"
keywords: "octopress, git, _deploy, rake"
---
已经习惯了在折腾中学习，把遇到的问题看成是学习的机会，有了这种心态，就
不怕麻烦了。昨天晚上为了解决octopress不能`rake deploy`的问题，搞了几个
小时，搞完了对git的分支的理解得到了提升。

解决完了再回头看，其实我的问题很sb，master branch本来只应该包括_deploy
目录里的内容，但我因为误操作把上层目录都变成了master branch，这样就会
在github上自己的blog项目内，看到很多本来不属于blog的文件。当然使用
`rake deploy`也会出现问题。之前瞎折腾了很久，浪费好多时间，结果还是读
源码解决了问题，Rakefile里`setup_github_page`和`push`任务看看，就知道
咋回事了。
<!-- more -->
{% codeblock my solution lang:bash %}
$rm -rf _deploy
$mkdir _deploy
$cd _deploy
$git init
$echo "test" > index.html
$git branch -m master
$git commit -m "octopress init"
$git remote add origin git@github.com:username/username.github.com.git
$git push origin master
{% endcodeblock %}

