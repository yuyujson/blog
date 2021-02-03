---
title: git免密
urlname: fnyksc
date: '2021-01-20 12:02:31 +0800'
categories:
  - util
permalink: git-nonpassword
tags: []
---

使用 git credential.helper 插件配置 http 请求用户名密码:

```bash
git config --global credential.helper store -file=.git_credentails
echo 'http://用户名:密码@icode.baoshiyun.com' > ~/.git-credentials
```

配置后 git 请求 `github` 时会自动从 `~/.git-credentials`  这个文件中读取, 不需要再次输入用户名密码
