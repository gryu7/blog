---
title: "AWS CloudFormation で not authorized to perform with an explicat deny"
date: 2024-12-25T01:58:14+09:00
lastmod: 2024-12-25T01:58:14+09:00
categories: 
  - "2024/12"

draft: false

slug: "20241225"
thumbnail: ""
tags:
  - "AWS"
summary: "IAMでは強権が割り当てられているはずだが、CloudFormationでリソース作成に失敗した。"
---

ここ3ヶ月程度でついにAWSを触る機会が出てきたが、詰まったことがあるのでメモ :memo:

## エラー
CloudFormationを利用したリソース作成を行おうとしたところ、一部リソースの作成においてエラーを検出。

```txt
Resource handler returned message: "User: arn:aws:iam::hoge is not authorized to perform: logs:CreateLogDelivery with an explicit deny in an identity-based policy"
```

ただ、とりあえず初期段階での環境構築だったので、作業しているIAMへは`AdministratorAccess`が付与されているはず。。。


## 原因
社内のセキュリティーポリシーにより、AWSへアクセス可能な接続元IPアドレスを制限していたが、CloudFormationでは、作業している端末のIPアドレスではない、CloudFormationサービスのIPアドレスがAWSアカウントにアクセスする場合があり、アクセスできなかったことが原因でした。  

> `aws:SourceIp` AWS 全体の条件を使用しないでください。CloudFormation はリクエストの送信元 IP アドレスではなく、独自の IP アドレスを使用してリソースをプロビジョニングします。例えば、スタックを作成する際に、CloudFormation は、Amazon EC2 インスタンスを起動したり Amazon S3 バケットを作成したりするために、`CreateStack` コールや **create-stack** コマンドを実行した IP アドレスではなく、その IP アドレスからリクエストを行います。  
> <small>引用: [AWS Identity and Access Management で CloudFormation アクセスを制御する](https://docs.aws.amazon.com/ja_jp/AWSCloudFormation/latest/UserGuide/control-access-with-iam.html#using-iam-conditions)</small>

まさに上記の`aws:SourceIp`で接続元IPを絞っており、絞っていることは認識していたのですが、自身のIAMのperformが拒否されているというエラーからだと分かりづらかった・・・


## 対応
今回のように、AWSサービスがAWSへリクエストを実行したかを確認するため、`aws:ViaAWSService`を条件に追加。  
具体的には接続元を制限していたIAMポリシーを以下のようにすることで回避できました。

```json
{
    "Version": "2012-10-17",
    "Statement": {
        "Effect": "Deny",
        "Action": "*",
        "Resource": "*",
        "Condition": {
            "NotIpAddress": {
                "aws:SourceIp": [
                    "接続許可IPリスト",
                ]
            },
            "Bool": {
                "aws:ViaAWSService": "false"
            }
        }
    }
}
```
