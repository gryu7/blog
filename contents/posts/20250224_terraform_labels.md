---
title: "Google Cloudのterraformが管理するlabels"
date: 2025-02-24T16:21:22+09:00
lastmod: 2025-02-24T16:21:22+09:00
categories: 
  - "2025/02"

draft: false

thumbnail: ""
tags:
  - "terraform"
  - "Google Cloud"
summary: ""
---

Google Cloudでterraformを利用しており、利用しているproviderを4.x系からバージョンアップしたところ、5.x系からlabel管理機能が大きく変わっていたので覚書。  
ざっくりは、 labels(リソース個別) ⊆ terraform_labels(リソース個別+provider単位) ⊆ effective_labels(リソース個別+provider単位+Webコンソール) という感じで、terraform上でのlabelの扱いが細分化された。

<!--more-->

## Google Cloudのterraform provider
Google Cloudをterraformで操作する際のproviderが2023/10に5.x系、2024/8に6.x系がリリースされており、4.x系だと取り扱えないGoogle Cloudのサービスも多くなってきたので、重い腰をあげてバージョンアップをしていたところ、5.x系からlabelの管理方法が大きく変わっていた。  
Google Cloud上ではlabelのままだが、terraform上の扱いのみ変更されている。


## provider単位で設定するdefault_labels
provider単位で __default_labels__ が設定できるようになって、同一平面のすべてのterraformで管理しているリソースにlabelの適用が可能になった。
```txt
provider "google" {
  default_labels = {
    my_global_key = "one"
    my_default_key = "two"
  }
}
```

従来通り、リソース単位で __labels__ を設定することも可能で、各labelのkey-valueのkeyが同一である場合、リソース単位の __labels__ が優先される。  
この __default_labels__ と __labels__ を足し合わせたものがtfstate上では、 __terraform_labels__ として表現されるため、`labels(リソース個別に設定) ⊆ terraform_labels(リソース個別のlabels+provider単位のdefault_labels)`という和になる。


## 実際のGoogle Cloud上に設定されているeffective_labels
さらに、terraform以外のクライアント、例えば、Google Cloudコンソール(Webコンソール)からlabelを追加した場合、それがterraformに影響を与えないようになった。

terraform上から付け加えたlabel以外を含めて、実機のlabelの全量が、tfstate上では __effective_labels__ として表現され、`terraform_labels(リソース個別のlabels+provider単位のdefault_labels) ⊆ effective_labels(リソース個別のlabels+provider単位のdefault_labels+Webコンソールなどterraform外で付与したlabel)`という和になる。


## サンプル
Google Cloud向けの[terraform privider Upgrade Guide](https://registry.terraform.io/providers/hashicorp/google/latest/docs/guides/version_5_upgrade#provider-level-labels-rework)に具体例のサンプルが掲載されている。

```txt
provider "google" {
  default_labels = {
    my_global_key = "one"
    my_default_key = "two"
  }
}

resource "google_compute_address" "my_address" {
  name     = "my-address"

  labels = {
    my_key = "three"
    # overrides provider-wide setting
    my_default_key = "four"
  }
}
```

上記のように、providerから __default_labels__ で *my_global_key = "one"*, *my_default_key = "two"* を設定し、リソース個別に *my_key = "three"*, *my_default_key = "four"* とすると、以下のようなterraform planの結果が得られる。  
ここでは、 __labels__ と __default_labels__ で同一のkeyを使用している *my_default_key* のvalueは __labels__ のvalueが優先された状態で、それらの和が __terraform_labels__ へ表現されている。  
また、terraform上から新規リソース作成をしており、Webコンソールなどterraform外からのツールは利用していないため、 __effective_labels__ は __terraform_labels__ と同じ値となっている。

```bash
# google_compute_address.my_address will be created
  + resource "google_compute_address" "my_address" {
      + address            = (known after apply)
      + address_type       = "EXTERNAL"
      + creation_timestamp = (known after apply)
      + effective_labels   = {
          + "my_default_key" = "four"
          + "my_global_key"  = "one"
          + "my_key"         = "three"
        }
      + id                 = (known after apply)
      + label_fingerprint  = (known after apply)
      + labels             = {
          + "my_default_key" = "four"
          + "my_key"         = "three"
        }
      + name               = "my-address"
      + network_tier       = (known after apply)
      + prefix_length      = (known after apply)
      + project            = "my-project"
      + purpose            = (known after apply)
      + region             = (known after apply)
      + self_link          = (known after apply)
      + subnetwork         = (known after apply)
      + terraform_labels   = {
          + "my_default_key" = "four"
          + "my_global_key"  = "one"
          + "my_key"         = "three"
        }
      + users              = (known after apply)
    }

Plan: 1 to add, 0 to change, 0 to destroy.
```

上記のリソースを作成後、Webコンソールなどterraform外からlabelを追加したのち、`terraform apply -refresh-only`を行うと、Google Cloud実機のlabelが __effective_labels__ に取り込まれる。

### 実機確認
サンプルのリソースを作成後に、terraform外からlabel追加を行い、それがtfstate内の __effective_labels__ への取り込まれることを確認。

サンプルリソースを作成
```bash
[user@host hoge]$ gcloud compute addresses describe --region asia-northeast1 my-address --format json | jq .labels
{
  "my_default_key": "four",
  "my_global_key": "one",
  "my_key": "three"
}
```

terraform外からlabel追加(サンプルの __google_compute_address__ はWebコンソールからlabel修正できなかったので、gcloudコマンドでlabel追加)
```bash
[user@host hoge]$ gcloud beta compute addresses update --region asia-northeast1 my-address --update-labels=my_cli=five
Updating labels of address [my-address]...done.                                                                                             
[user@host hoge]$ gcloud compute addresses describe --region asia-northeast1 my-address --format json | jq .labels
{
  "my_cli": "five",
  "my_default_key": "four",
  "my_global_key": "one",
  "my_key": "three"
}
```

terraformから確認すると、planでは __No changes.__ だが、-refresh-onlyにより追加したlabelがtfstateの __effective_labels__ へ取り込まれる
```bash
[user@host hoge]$ terraform plan
google_compute_address.my_address: Refreshing state...

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration and found no differences, so no changes are needed.
[user@host hoge]$ 
[user@host hoge]$ terraform apply -refresh-only
google_compute_address.my_address: Refreshing state...

Note: Objects have changed outside of Terraform

Terraform detected the following changes made outside of Terraform since the last "terraform apply" which may have affected this plan:

  # google_compute_address.my_address has changed
  ~ resource "google_compute_address" "my_address" {
      ~ effective_labels   = {
          + "my_cli"         = "five"
            # (3 unchanged elements hidden)
        }
...omitted...
        name               = "my-address"
        # (11 unchanged attributes hidden)
    }


This is a refresh-only plan, so Terraform will not take any actions to undo these. If you were expecting these changes then you can apply
this plan to record the updated values in the Terraform state without changing any remote objects.

Would you like to update the Terraform state to reflect these detected changes?
  Terraform will write these changes to the state without modifying any real infrastructure.
  There is no undo. Only 'yes' will be accepted to confirm.

  Enter a value: yes


Apply complete! Resources: 0 added, 0 changed, 0 destroyed.
[user@host hoge]$ 
[user@host hoge]$ terraform state show 'google_compute_address.my_address'
# google_compute_address.my_address:
resource "google_compute_address" "my_address" {
...omitted...
    effective_labels   = {
        "my_cli"         = "five"
        "my_default_key" = "four"
        "my_global_key"  = "one"
        "my_key"         = "three"
    }
...omitted...
    labels             = {
        "my_default_key" = "four"
        "my_key"         = "three"
    }
    name               = "my-address"
...omitted...
    terraform_labels   = {
        "my_default_key" = "four"
        "my_global_key"  = "one"
        "my_key"         = "three"
    }
...omitted...
}
```



## 参考
- [Terraform provider for Google Cloud 5.0.0 Upgrade Guide](https://registry.terraform.io/providers/hashicorp/google/latest/docs/guides/version_5_upgrade#provider-level-labels-rework)
- [Terraform Google Provider 5.0.0 の新機能と変更点について](https://zenn.dev/cloud_ace/articles/terraform-google-provider-5)
