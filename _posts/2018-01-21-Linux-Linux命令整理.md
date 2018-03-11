---
layout:     post
title:      Linux命令整理
date:  2018-01-21
category:   Linux
tags:   [Linux]
---
- `tar -cvf - /ect | tar -xvf - `
   - 等价于将/etc目录复制到当前目录，首先将/etc打包到标准输出(-),通过管道后面的tar命令从标准输入里面读取之前的tar包，进行解打包到当前目录
