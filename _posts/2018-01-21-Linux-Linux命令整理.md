---
layout:     post
title:      Linux命令整理
date:  2018-01-21
category:   Linux
tags:   [Linux]
---

```bash
tar -cvf - /ect | tar -xvf - 
等价于将/etc目录复制到当前目录，首先将/etc打包到标准输出(-),
通过管道后面的tar命令从标准输入里面读取之前的tar包，进行解打包到当前目录
```
```bash
unix时间戳
自 1970 年 1 月 1 日（00:00:00 GMT）以来的秒数
```
```bash
grep -A1 keyword filename
找出filename中带有keyword的行，输出中除显示该行外，还显示之后的一行(After 1)
grep -B1 keyword filename
找出filename中带有keyword的行，输出中除显示该行外，还显示之前的一行(Before 1)
grep -1 keyword filename
找出filename中带有keyword的行，输出中除显示该行外，还显示之前的一行(After 1)和显示之后的一行(After 1）
```
