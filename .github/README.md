# Blog on GitHub Pages
自分向け :memo:

* ページを作成する
  * 環境
    * hugo v0.128.0
  * 以下の手順で実施
    * templateからmdファイルの生成

     ```bash
      $ hugo new content posts/hoge.md
      Content "path/posts/hoge.md" created
      ```

    * 内容の記述
      * mdファイルの修正
    * post前の確認

      ```bash
      # 確認用にlocalhost向けのbuildとweb serverの起動
      $ hugo server --buildDrafts
      # ドラフト記事の公開
      $ sed -i -e '1,15s/^draft:.*/draft: false/' contents/posts/hoge.md
      # build(github actionsでも実行するため、不要な感じもする)
      $ hugo
      ```

* ページの修正
  * contents/posts配下の各mdファイルを修正
    * このとき、`lastmod`行を削除すると最終更新日付も更新される
  * それ以外は作成と同様にlocalhostで確認したりbuildしたり

* 公開ディレクトリ配下の設定
  * `hugo.toml`の`publishDir`として指定してbuildすることで`docs`ディレクトリ配下のファイルが自動生成される
    * ただし、`docs/css/custom.css`のみ直接作成している
