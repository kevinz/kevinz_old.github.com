---
layout: post
title: "Blog with jekyll+octtpress+emacs on github"
date: 2012-07-05 7:59
comments: true
categories: [tool]
description: "使用jekyll octopress emacs写blog"
keywords: "jekyll, octopress, emacs, github"

---
最近学习的动力大爆发，学了不少东西，应该记录并分享出来，把自己挂在
github上的静态页面换成了强大的jekyll+octopress，捣鼓下开始写东西了，用markdown写
blog的感觉应该很棒。

接下来是开发环境，没错，是按写代码的方式写blog，我用的是emacs +
mardown-mode,非常爽的组合。因为我是用archlinux，用yaourt怎么
一个方便了得。

{% codeblock install ibus-el lang:bash %}
$ yaourt -S ibus-el
{% endcodeblock %}

{% codeblock ibus-mode configuration %}
(require 'ibus)
(add-hook 'after-init-hook 'ibus-mode-on)
(global-set-key (kbd "C-\\") 'ibus-toggle)
{% endcodeblock %}

{% codeblock Enable markdown mode %}
(autoload 'markdown-mode "markdown-mode.el"
   "Major mode for editing Markdown files" t)
(setq auto-mode-alist
   (cons '("\\.text"  . markdown-mode) auto-mode-alist))

(setq auto-mode-alist
      (cons '("\\.md"  . markdown-mode) auto-mode-alist))
{% endcodeblock %}      


