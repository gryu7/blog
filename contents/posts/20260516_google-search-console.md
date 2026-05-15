---
title: "GitHub Pages でホストするサイトが Google 検索にヒットしない"
date: 2026-05-16T05:47:04+09:00
lastmod: 2026-05-16T05:47:04+09:00
categories:
  - "2026/05"

draft: false

thumbnail: ""
tags:
  - "GitHub Pages"
  - "netlify"
summary: "この GitHub Pages でホストしているブログを Google 検索にヒットさせたかったが、サイトマップの登録が Google Search Console で取得させることがなかなかできなかった。最終的に netlify でホストするようにした。"
---

随分前から、どうせ技術ブログ書くなら Google 検索でヒットするようにしたいと思っていたが、なかなかできずにモチベーションが下がっていたが、ようやくできた。(と思う。)

## 結論
GitHub Pages の仕様が変更になって、遅くとも2025年初から Google Search Console へのサイトマップ登録で GitHub Pages 上のサイトマップが取得できなくなっていた模様。  
参考: [Uploading a Sitemap from GitHub Pages to Google Search Console · community · Discussion #149884](https://github.com/orgs/community/discussions/149884)

そこで、ホスティングサービスを [netlify](https://www.netlify.com/) に変更し、サイトマップ登録を試したところ、 Google Search Console での登録作業後1分ほどで成功した。


## 発生していたこと
* 基本的な Google 検索にヒットさせるために行う、サイトマップの登録に関する情報はすぐに出てくるので割愛
  * 参考: [GitHubのリポジトリがGoogle検索に引っかからないのでやったこと #GithubPages - Qiita](https://qiita.com/NightOwl/items/a7f972ba70f1ad037d65#google-search-console%E3%81%AE%E8%A8%AD%E5%AE%9A)
  * Google 検索にヒットさせたい URL の一覧をサイトマップという形式で Google Search Console へ登録してやることで、検索にでてくるようになるよ、という感じ
    * 正確には、個別の URL 、本サイトならポストごとの URL を手動登録することもできるが、なかなかめんどくさい
* このサイトマップ登録をしたところ、 Google Search Console にてサイトマップを取得できないというエラーが発生してしまう

{{< img
  src="20260516_sitemap-fetch-error.png"
  alt=""
  title=""
  caption="取得できませんでした というエラーになっていた"
  attr=""
  link=""
  attrlink=""
>}}

この 取得できませんでした というエラーだが、時間経過で修正されるという情報も多く存在している。  
そのため、たまに Google Search Console を開いて、サイトマップの登録状況を確認するということを繰り返していたが、今回、ホスティングサービスを変更したところ、1分程度で 成功しました というステータスになった。

{{< img
  src="20260516_sitemap-fetch-success.png"
  alt=""
  title=""
  caption="netlify に変更したらすぐに成功した・・・・・・"
  attr=""
  link=""
  attrlink=""
>}}


## 対応
解決策として、前述の [GitHub Discussion](https://github.com/orgs/community/discussions/149884) にあるように、ホスティングサービスを変更した。  
とりあえず、 Discussion 内で挙げられており、無料で使えそうな [netlify](https://www.netlify.com/) で対応したので、手順メモ。

### 手順メモ
1. アカウントを作成してログイン
2. project に登録して公開するファイルや Git リポジトリの選択画面になるので、 GitHub へのアクセスを許可
3. 画面の指示に従って、リポジトリ選択・ビルド設定を実施
   * 私の hugo 環境では、 GitHub 上の hugo.toml の設定を一部オーバーライドしたかったので、ビルド設定で環境変数の設定が必要になった
     * HUGO_VERSION と HUGO_BASEURL を環境変数として追加することで、ひとまずはうまくビルドもできている
4. サイドバーの Projects → Project configuration → General project settings から Project name を変更し、 サブドメイン を好きなものに変更する
5. サイドバーの Projects → Deploys から Trigger Deploy ボタンでビルド・デプロイが実行される


## 感想
* 過去には GitHub Pages のサイトマップも取得できていたようで、古い情報やそもそも Google 検索にヒットする GitHub Pages が多数存在しているので、わかりづらかった
* Google Search Console のヘルプにも、時間で解決すると書かれており、これにもかなり惑わされていた
  * いや、もしかすると、これから GitHub Pages のほうも検索に引っかかるのかも・・・望み薄
* とりあえずで netlify を選んでみたが、 [Cloudflare Pages](https://www.cloudflare.com/ja-jp/developer-platform/products/pages/) のほうが良いかも？
  * netlify は無料枠のビルド・デプロイ回数が少ないかもしれない
    * これ、早く決めないと、またドメイン・URL変わっちゃうんだよな。。。。。。