---
title: "lsコマンドで隠しファイルを表示する"
date: 2024-11-11T00:11:02+09:00
lastmod: 2024-11-11T00:11:02+09:00
categories: 
  - "2024/11"

draft: false

slug: ""
thumbnail: ""
tags:
  - "linux"
summary: "lsコマンドで隠しファイルを表示する場合、`-a`オプションを利用していたが、`-A`オプションというものがあることを知った。"
---

## 概要
lsコマンドで隠しファイルを表示する場合、`-a`オプションを利用していたが、`-A`オプションというものがあることを知った。

```txt
 -a, --all
        do not ignore entries starting with .
 -A, --almost-all
        do not list implied . and ..
```

> 引用: https://man7.org/linux/man-pages/man1/ls.1.html

`ls -la`だと出力される`.` , `..`ディレクトリが非表示になるので、少し便利。

## 出力サンプル
```bash
[user@host hoge]$ ls -l
total 0
-rw-r--r--. 1 user user 0 Nov 10 06:34 README.md
[user@host hoge]$ ls -la
total 4
drwxr-xr-x.  3 user user   35 Nov 10 06:34 .
drwx------. 25 user user 4096 Nov 10 06:34 ..
drwxr-xr-x.  7 user user  119 Nov 10 06:34 .git
-rw-r--r--.  1 user user    0 Nov 10 06:34 README.md
[user@host hoge]$ ls -lA
total 0
drwxr-xr-x. 7 user user 119 Nov 10 06:34 .git
-rw-r--r--. 1 user user   0 Nov 10 06:34 README.md
[user@host hoge]$
```
