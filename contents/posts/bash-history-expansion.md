---
title: "コマンドラインでよく使う引数 !$"
date: 2025-01-20T08:35:47+09:00
lastmod: 2025-01-20T08:35:47+09:00
categories: 
  - "2025/01"

draft: false

slug: "20250120"
thumbnail: ""
tags:
  - "linux"
  - "bash"
summary: "コマンドライン上からよく`!$`を利用して、直前に実行したコマンドの最後の引数を利用することがよくあったが、この機能が何かを調べたメモ。"
---

基本的にLinux、なかでも、RHEL系のOSをコマンドラインから扱う仕事が多いのですが、その中で良く`!$`という引数を利用してコマンド実行することが多かった。  
この`!$`は直前に実行したコマンドの最後の引数を再利用するくらいの曖昧な理解しかなかったが、何によって実行されているかを調べたのでメモ。

## 結論
* `!$`はbashの[History Expansion](https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html#History-Interaction)という機能により提供される
* [History Expansion](https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html#History-Interaction)では、実行したコマンド履歴中の語句の再利用等が行える
* `!$`は`!!:$`の省略形で、直前に実行しコマンドの最後の引数という意味になる


## 調べたメモ
### History Expansionとは
[History Expansion](https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html#History-Interaction)はbashの機能の一部で、コマンド履歴(historyで見られるやつ)から語句を再利用したり修正して利用することができる。

以下の3段階で機能を提供しており、`:`を区切り文字としてそれぞれの機能を使うことができる(2., 3.は省略可能)。
1. [Event Designator](https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html#Event-Designators)(コマンド履歴からコマンド行を指定)
2. [Word Designator](https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html#Word-Designators)(指定したコマンド行から語句を指定)
3. [Modifier](https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html#Modifiers)(指定した語句の修正)

<details><summary class="pseudo_h3">各機能の詳細</summary>
Event Designator, Word Designator, Modifierの機能を抜粋で紹介。

### Event Designator(コマンド履歴からコマンドを指定)
実行したコマンド履歴からどのコマンド行(event)を参照するかを指定。
- `!n`: n行目のコマンド行を参照
- `!-n`: n行前に実行したコマンド行を参照
- `!!`: 直前に実行したコマンド行を参照、`!-1`と同義
- `!string`: string文字列から始まる直近実行したコマンド行を参照
- `!?string?`: string文字列を含む直近実行したコマンド行を参照


### Word Designator(指定したコマンド行から語句を指定)
参照しているコマンド行内からどの語句を指定するかを指定。
- `0`: コマンド行の先頭の語句。主にコマンドそのもの
- `n`: コマンド行のn番目の語句
- `^`: コマンド行の最初の引数、1と同義
- `$`: コマンド行の最後の引数
- `x-y`: コマンド行のx番目からy番目の語句の範囲
- `*`: コマンド行の先頭の語句を除いた全ての引数。`1-$`と同義


### Modifier(指定した語句の修正)
参照している語句の修正等。
- `h`: パス名の末尾を削除して選択
- `t`: パス名の末尾のみを選択
- `p`: コマンドを表示するが、実行はしないようにする
- `s/old/new/`: 語句内の最初の`old`を`new`に置換する
- `gs/old/new/`: 語句内のすべての`old`を`new`に置換する

</details>


## 利用例
### よく使うやつ
- リダイレクトで作成したファイルをVSCodeで開く
  - ```bash
    [student@workstation ~]$ echo test > file.txt
    [student@workstation ~]$ code !$
    code file.txt
    [student@workstation ~]$
    ```
- 実行しているコマンドのファイルを参照
  - ```bash
    [student@workstation ~]$ which kubectl
    /usr/local/bin/kubectl
    [student@workstation ~]$ file $(!!)
    file $(which kubectl)
    /usr/local/bin/kubectl: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, Go BuildID=fB0W48bQW-P7sS0ClLxz/5k7yPdjl4oztAuiiFBSu/RyFxH58-Bf6icTsZY_Nx/2Yy2aWoRMNxfify2hrdM, BuildID[sha1]=6eb151949a7188ec1511b2d9de35395791e592d8, with debug_info, not stripped
    [student@workstation ~]$
    ```



## 感想
Modifierはsedに渡したほうが便利そう。  
結構機能が豊富なので、色々使い道はあるかな。


## 参考
- [Bash Reference Manual](https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html#History-Interaction)
- [[bash] シェルの履歴機能を使いこなす](https://mseeeen.msen.jp/bash-history-expansion/)
