---
title: "Keycloak の Client Secret を特定の文字列にしたい"
date: 2025-09-30T23:13:24+09:00
lastmod: 2025-09-30T23:13:24+09:00
categories:
  - "2025/09"

draft: false

thumbnail: ""
tags:
  - "Keycloak"
summary: ""
---
Keycloak の Client を Webコンソールからいじっていて、 Client Secret を誤って Regenerate してしまったので、以前の文字列に戻したい。  
Webコンソール での Client Secret は入力が非活性なフィールドで、 再生成(Regenerate)しかできないようになっており、戻すことができない。
<!--more-->

## 結論
Keycloak の Admin CLI や REST API を利用することで、強制的に特定の文字列に変更することができた。  
これによって、変更前の文字列が分かれば、復旧することも可能。


## 環境
* Keycloak 26.3.1
  * Client type: OpenID Connect の Client を作成して、 Client authentication を On にすることで Client Secret が生成される

{{< img
  src="20250930_client_secret.png"
  alt=""
  title=""
  caption="Client Secret は非活性なフィールドの文字列でWebコンソールから任意の文字列が設定できない"
  attr=""
  link=""
  attrlink=""
>}}


## 詳細
### 事前に利用するシェル変数を設定
* KC_URL=<Keycloak を公開しているURL http://localhost:8080/ など>
* USER=<`KC_BOOTSTRAP_ADMIN_USERNAME` などで設定した 管理者ユーザ名>
* PASSWORD=<`KC_BOOTSTRAP_ADMIN_PASSWORD` などで設定した 管理者パスワード>
* REALM=<Client が作成されている realm>
* CLIENT_UUID=<Client を作成した際に生成される UUID>
  * Client の操作を行う際のエンドポイントとして利用される


### Admin CLI の場合
1. Admin CLI のインストール
https://www.keycloak.org/getting-started/getting-started-zip に記載されている、 Keycloak を取得し、展開する。  
展開したディレクトリ内の `bin` をパスに通しておくとベター。

```console
export PATH=$(pwd)/keycloak-26.3.1/bin:$PATH
```

2. 認証取得
```console
kcadm.sh config credentials --server ${KC_URL} --realm master --user ${USER} --password ${PASSWORD}
```

3. Client Secret 更新
```console
kcadm.sh update clients/${CLIENT_UUID} -s "secret=<Client Secret に設定したい文字列>" -r ${REALM}
```

4. Client Secret 確認
```console
kcadm.sh get clients/${CLIENT_UUID}/client-secret -r ${REALM}
```

結果が以下の形式で出力される

```
{
  "type" : "secret",
  "value" : "<設定された Client Secret>"
}
```


### REST API の場合
1. TOKEN取得
```console
TOKEN=$(curl -s \                                                                                                 
  -d "client_id=admin-cli" \
  -d "username=$USER" \
  -d "password=$PASSWORD" \
  -d "grant_type=password" \
  "$KC_URL/realms/master/protocol/openid-connect/token" | jq -r .access_token)
```

2. Client Secret 更新
```console
curl -X PUT \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"secret": "<Client Secret に設定したい文字列>"}' \
  "$KC_URL/admin/realms/$REALM/clients/$CLIENT_UUID"
```

3. Client Secret 確認
```console
curl -s \
  -H "Authorization: Bearer $TOKEN" \
  "$KC_URL/admin/realms/$REALM/clients/$CLIENT_UUID/client-secret" | jq
```

結果が以下の形式で出力される

```
{
  "type" : "secret",
  "value" : "<設定された Client Secret>"
}
```


### 事後確認
Webコンソール から確認してみると、 Client Secret の文字数が変更されており、適切に更新されたことがわかる(もちろん表示することも可能)

{{< img
  src="20250930_updated_client_secret.png"
  alt=""
  title=""
  caption="更新後の Client Secret"
  attr=""
  link=""
  attrlink=""
>}}



## 所感・まとめ
* オペミスで Regenerate してしまった場合の救済措置くらいで利用可能
* Keycloak の他の設定値と異なり、 Client Secret は、 GET で取得したJSONの形式、と、 PUT/POST で送信するJSONの形式が異なるので、注意が必要
  * 例えば、Admin CLIであれば、 `kcadm.sh update clients/${CLIENT_UUID}/client-secret -r ${REALM} -s value=<Client Secret に設定したい文字列>` はエラーになる


## 参考
* Keycloak の Admin CLI
  * [Server Administration Guide](https://www.keycloak.org/docs/26.3.1/server_admin/index.html#updating-the-current-secret-for-a-specific-client)
