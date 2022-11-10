---
title: "Hello World"
date: 2021-01-20T22:49:51+08:00
lastmod: 2021-01-20T22:49:51+08:00
draft: false
keywords: []
description: ""
tags: ["hugo", "even"]
categories: []
author: "lttzz"

# You can also close(false) or open(true) something for this content.
# P.S. comment can only be closed
comment: false
toc: true
autoCollapseToc: false
postMetaInFooter: false
hiddenFromHomePage: false
# You can also define another contentCopyright. e.g. contentCopyright: "This is another copyright."
contentCopyright: false
reward: false
mathjax: false
mathjaxEnableSingleDollar: false
mathjaxEnableAutoNumber: false

# You unlisted posts you might want not want the header or footer to show
hideHeaderAndFooter: false

# You can enable or disable out-of-date content warning for individual post.
# Comment this out to use the global config.
#enableOutdatedInfoWarning: false

flowchartDiagrams:
  enable: false
  options: ""

sequenceDiagrams: 
  enable: false
  options: ""

---

唔，从hexo换到了hugo，hexo实在是太慢辽。

<!--more-->

# 0x00 hugo-theme-even

[hugo-theme-even](https://github.com/olOwOlo/hugo-theme-even): A super concise theme for Hugo

理由挺简单的，GitHub搜索 `hugo theme` ，简约大方，功能也满足需求。

# 0x01 Deploy to GitHub Page

``` bash
# 生成的静态页面在public下，在此目录下依次输入命令

git init
git add .
git commit -m "xxxx"

# push an existing repository from the command line
git remote add origin git@github.com:lttzz/lttzz.github.io.git
git branch -M main
git push -u origin main
```

``` bash
git clone git@github.com:lttzz/lttzz.github.io.git
cd lttzz.github.io
hugo new site temp
cd temp
mv * ../
git submodule add git@github.com:olOwOlo/hugo-theme-even.git themes/even
```

# 0x02 Reference

+ [从 Jekyll 迁移到 Hugo，Hugo 不完全指南](https://cjting.me/2017/06/04/migrate-to-hugo-from-jekyll/)
+ [Hugo 从入门到会用](https://olowolo.com/post/hugo-quick-start/)
+ [Even document](https://github.com/olOwOlo/hugo-theme-even/blob/master/README.md)
+ [Even Theme preview](https://hugo-theme-even.netlify.app/post/even-preview/)
+ [Shortcodes](https://hugo-theme-even.netlify.app/post/shortcodes/)

