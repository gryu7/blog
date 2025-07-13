---
title: "zipdetailsコマンドで Can't call method \"name\" のエラー終了"
date: 2025-07-13T17:27:16+09:00
lastmod: 2025-07-13T17:27:16+09:00
categories: 
  - "2025/07"

draft: false

slug: "20250713"
thumbnail: ""
tags:
  - "linux"
  - "rhel10"
summary: "`zipdetails`というコマンド自体はじめて知ったが、dnfのDeveloper Toolsに含まれているデフォルトのツールだった。<br> zipアーカイブをいじっているとうまくいかなかったのでメモ。"
---

zipアーカイブを調査する必要があり、調べていたところ、`zipdetails`というコマンドがあることを発見。  
便利そうなので使ってみたのだが、エラー終了してしまうので、対応をメモ。

## 結論

### エラー内容
```txt
Can't call method "name" on an undefined value at /usr/bin/zipdetails line 7099.
```


### 解消方法
`perl-encoding`をインストールしてあげると解消。
```bash
sudo dnf install perl-encoding
```



## zipdetails コマンド
現在利用しているWSL2上の RHEL10 の環境では、dnfのgroupでインストールした`Development Tools`内の`perl-IO-Compress-2.212-512.el10.noarch`に`zipdetails`というコマンドが含まれている。
```bash
[gryu7: 20250713]$ cat /etc/redhat-release 
Red Hat Enterprise Linux release 10.0 (Coughlan)
[gryu7: 20250713]$ rpm -qf $(which zipdetails)
perl-IO-Compress-2.212-512.el10.noarch
[gryu7: 20250713]$ 
```

zipアーカイブの調査で利用できそうだったので、使ってみることにしたが、うまく動かないファイルに遭遇。  
というか、だいたい動かない。Windowsから持ってきたファイルはうまく動いた。
```bash
[gryu7: 20250713]$ echo aaa > test
[gryu7: 20250713]$ zip test.zip test
  adding: test (stored 0%)
[gryu7: 20250713]$ zipdetails test.zip

0000 LOCAL HEADER #1       04034B50 (67324752)
0004 Extract Zip Spec      0A (10) '1.0'
0005 Extract OS            00 (0) 'MS-DOS'
0006 General Purpose Flag  0000 (0)
0008 Compression Method    0000 (0) 'Stored'
000A Modification Time     5AED8D57 (1525517655) 'Mon Jul 14 02:42:46 2025'
000E CRC                   77F85D95 (2012765589)
0012 Compressed Size       00000004 (4)
0016 Uncompressed Size     00000004 (4)
001A Filename Length       0004 (4)
001C Extra Length          001C (28)
Can't call method "name" on an undefined value at /usr/bin/zipdetails line 7099.
[gryu7: 20250713]$ zipinfo test.zip
Archive:  test.zip
Zip file size: 162 bytes, number of entries: 1
-rw-r--r--  3.0 unx        4 tx stor 25-Jul-13 17:42 test
1 file, 4 bytes uncompressed, 4 bytes compressed:  0.0%
[gryu7: 20250713]$ file test.zip 
test.zip: Zip archive data, at least v1.0 to extract, compression method=store
[gryu7: 20250713]$ 
```


## 原因
原因を見ていこうと調べてみると、`/usr/bin/zipdetails`自体perlで読めるが、とりあえず既存の情報をみていくと、同一RPMパッケージ内の`/usr/share/doc/perl-IO-Compress/README`に[GitのURL](https://github.com/pmqs/IO-Compress)があり、そこからIssueを確認。  
zipdetailsは別リポジトリと非同期だが定期的にコードを同期しているようなので、[zipdetailsのリポジトリ](https://github.com/pmqs/zipdetails)も確認。

Issue発見。  
https://github.com/pmqs/zipdetails/issues/22

これをみて、`perl-encoding`のRPMパッケージをインストールすれば良さそうとわかって、インストールしたら上手く動いた。
```bash
[gryu7: 20250713]$ zipdetails test.zip 

0000 LOCAL HEADER #1       04034B50 (67324752)
0004 Extract Zip Spec      0A (10) '1.0'
0005 Extract OS            00 (0) 'MS-DOS'
0006 General Purpose Flag  0000 (0)
0008 Compression Method    0000 (0) 'Stored'
000A Modification Time     5AED8D57 (1525517655) 'Mon Jul 14 02:42:46 2025'
000E CRC                   77F85D95 (2012765589)
0012 Compressed Size       00000004 (4)
0016 Uncompressed Size     00000004 (4)
001A Filename Length       0004 (4)
001C Extra Length          001C (28)
001E Filename              'test'
0022 Extra ID #1           5455 (21589) 'Extended Timestamp [UT]'
0024   Length              0009 (9)
0026   Flags               03 (3) 'Modification Access'
0027   Modification Time   68737185 (1752396165) 'Sun Jul 13 17:42:45 2025'
002B   Access Time         68737185 (1752396165) 'Sun Jul 13 17:42:45 2025'
002F Extra ID #2           7875 (30837) 'Unix Extra type 3 [ux]'
0031   Length              000B (11)
0033   Version             01 (1)
0034   UID Size            04 (4)
0035   UID                 000003E9 (1001)
0039   GID Size            04 (4)
003A   GID                 000003E9 (1001)
003E PAYLOAD               aaa.

0042 CENTRAL HEADER #1     02014B50 (33639248)
0046 Created Zip Spec      1E (30) '3.0'
0047 Created OS            03 (3) 'Unix'
0048 Extract Zip Spec      0A (10) '1.0'
0049 Extract OS            00 (0) 'MS-DOS'
004A General Purpose Flag  0000 (0)
004C Compression Method    0000 (0) 'Stored'
004E Modification Time     5AED8D57 (1525517655) 'Mon Jul 14 02:42:46 2025'
0052 CRC                   77F85D95 (2012765589)
0056 Compressed Size       00000004 (4)
005A Uncompressed Size     00000004 (4)
005E Filename Length       0004 (4)
0060 Extra Length          0018 (24)
0062 Comment Length        0000 (0)
0064 Disk Start            0000 (0)
0066 Int File Attributes   0001 (1)
     [Bit 0]               1 'Text Data'
0068 Ext File Attributes   81A40000 (2175008768)
     [Bits 16-24]          01A4 (420) 'Unix attrib: rw-r--r--'
     [Bits 28-31]          08 (8) 'Regular File'
006C Local Header Offset   00000000 (0)
0070 Filename              'test'
0074 Extra ID #1           5455 (21589) 'Extended Timestamp [UT]'
0076   Length              0005 (5)
0078   Flags               03 (3) 'Modification Access'
0079   Modification Time   68737185 (1752396165) 'Sun Jul 13 17:42:45 2025'
007D Extra ID #2           7875 (30837) 'Unix Extra type 3 [ux]'
007F   Length              000B (11)
0081   Version             01 (1)
0082   UID Size            04 (4)
0083   UID                 000003E9 (1001)
0087   GID Size            04 (4)
0088   GID                 000003E9 (1001)

008C END CENTRAL HEADER    06054B50 (101010256)
0090 Number of this disk   0000 (0)
0092 Central Dir Disk no   0000 (0)
0094 Entries in this disk  0001 (1)
0096 Total Entries         0001 (1)
0098 Size of Central Dir   0000004A (74)
009C Offset to Central Dir 00000042 (66)
00A0 Comment Length        0000 (0)
#
# Done
[gryu7: 20250713]$ 
```


## 根本的に何が悪かったのか
先のIssueを見ると、`perl-libs`のRPMパッケージ内に含まれている`encoding.pm`では関数が不足していて、`perl-encoding`のパッケージをインストールする必要があるとのことだが、一方で、全てのバージョンのPerlにおいて`encoding.pm`は標準であるとのこと。

RPMの中身やPerl本体を見ていくと、以下がわかる。
- Perl本体
  - https://github.com/Perl/perl5/tree/v5.40.2
  - これをみると `cpan/Encode/encoding.pm` が存在していて、必要な関数も存在していそう
- RPMのbuild設定
  - [ソースコード閲覧ノススメ](https://www.intellilink.co.jp/column/oss/2014/120900.aspx) を参考にして、パッケージのbuildの設定をみていく
  - Sourceになっていたのが、GitHubではなくCPANだったので、一応`encoding.pm`があるか確認
    - ```bash
      [gryu7: 20250713]$ wget https://www.cpan.org/src/5.0/perl-5.40.2.tar.xz
      --2025-07-13 22:41:48--  https://www.cpan.org/src/5.0/perl-5.40.2.tar.xz
      Resolving www.cpan.org (www.cpan.org)... 146.75.113.55, 2a04:4e42:15::311
      Connecting to www.cpan.org (www.cpan.org)|146.75.113.55|:443... connected.
      HTTP request sent, awaiting response... 200 OK
      Length: 13923524 (13M) [application/x-xz]
      Saving to: ‘perl-5.40.2.tar.xz’

      perl-5.40.2.tar.xz                                 100%[=============================================================================================================>]  13.28M  --.-KB/s    in 0.1s    

      2025-07-13 22:41:48 (95.2 MB/s) - ‘perl-5.40.2.tar.xz’ saved [13923524/13923524]

      [gryu7: 20250713]$ tar tvf perl-5.40.2.tar.xz | grep encoding.pm
      -r--r--r-- steve/steve      22950 2025-03-30 19:35 perl-5.40.2/cpan/Encode/encoding.pm
      -r--r--r-- steve/steve       1220 2025-03-30 19:35 perl-5.40.2/ext/PerlIO-encoding/encoding.pm
      [gryu7: 20250713]$ 
      ```
  - 存在していたので、SPECファイルとそのGitをみていくと、これにたどり着いた
    - https://src.fedoraproject.org/rpms/perl/c/c61591b4f13bffcc3140b1458553d7a8ae7419b6
  - SPECファイル側のコメントによれば、以下とのこと
    - ```txt
      # All dual_life files/directories are deleted here instead of %%exclude in
      # %%files. So that debuginfo does not find unpacked binaries and blindly
      # symlinks to them at random packages.
      ```


## 所感
- RPMパッケージでのインストール と [ソースからのインストール](https://www.cpan.org/src/#:~:text=How%20to%20install%20from%20source)でPerlの挙動が変わるのが超気持ち悪い
  - そもそも、RPM以外のパッケージマネージャで管理されたPerlとの間でも挙動変わりそうな雰囲気がある
- RPMのbuildの設定など、色々調べられて勉強にはなった
  - ここ良かったです
    - [第10回「ソースコード閲覧ノススメ」 | 株式会社NTTデータ先端技術](https://www.intellilink.co.jp/column/oss/2014/120900.aspx)
  - 実際の環境で動いているソースをどこから取得しているかなど、こういうところから調査をすることも出てくる・・・かもしれない
- まったく意識していなかったが、zipというファイルの書庫化もおもしろかった
  - Javaで作られたzipを良い感じに修正して、もう1回Javaで読み込ませたかったっていうのが元々でしたが、超寄り道をした
