---
title: "Software Design 2022 1月号 Terraform"
emoji: "🍰"
type: "idea" # tech: 技術記事 / idea: アイデア
topics: ["Terraform"]
published: true
---

## はじめに

Software Design 2022年1月号のTerraformの記事の備忘録

https://www.amazon.co.jp//dp/B09NQSQXLZ

## 備忘録

- クレデンシャル情報を取り扱うことが多いのでうっかりPUSHしないように [git-secrets](https://github.com/awslabs/git-secrets) を導入しておく
  - 設定や使い方については[こちらの記事](https://zenn.dev/kkk777/articles/8f55db1e9678f2)がよくまとまっていた
- プロバイダについて
  - main.tfに書くこんなやつで、AWSやAzure、GCPなどのどの外部クラウドサービスのAPIと連携するかを設定する箇所
    ```terraform
    provider "aws" {
      region  = "ap-northeast-1"
      profile = "hoge-dev"
    }
    ```
  - プロバイダにもバージョンがあり、バージョンが上がると新しいサービスの管理がTerraformからできるようになったりする
    - terraform initを実行すると .terraform/.terraform.lock.hcl が作成されて、プロバイダのチェックサムが書き込まれる
      - githubの管理対象にいれておくことで、チーム内でプロバイダのバージョンを固定できる
- tfstateファイル(.terraform/terraform.tfstate)
  - インフラの環境情報について記載されているファイルで、直接編集するとterraformコマンドからリソースの変更ができなくなってしまう
    - このファイルはgithubで `管理せず` S3などの外部リソース上で管理すること
      - githubで管理したところでjsonファイルなので良いかどうかの判断が困難
      - terraform applyするごとにlocalのstateファイルに変更が入るので、即座にmergeしないと他のメンバーに変更を共有できない
        ```terraform
        {
            "version": 3,
            "serial": 1,
            "lineage": "a1111l1a-c11c-e111-1ll1-a1l1ll1lllll",
            "backend": {
                "type": "s3",
                "config": {
                    "access_key": null,
                    "acl": null,
                    "assume_role_duration_seconds": null,
                    "assume_role_policy": null,
                    "assume_role_policy_arns": null,
                    "assume_role_tags": null,
                    "assume_role_transitive_tag_keys": null,
                    "bucket": "hoge-dev-tfstate",
                    "dynamodb_endpoint": null,
                    "dynamodb_table": null,
                    "encrypt": true,
                    "endpoint": null,
                    "external_id": null,
                    "force_path_style": null,
                    "iam_endpoint": null,
                    "key": "hoge-dev.tfstate",
                    "kms_key_id": null,
                    "max_retries": null,
                    "profile": "hoge-dev",
                    "region": "ap-northeast-1",
                    "role_arn": null,
                    "secret_key": null,
                    "session_name": null,
                    "shared_credentials_file": null,
                    "skip_credentials_validation": null,
                    "skip_metadata_api_check": null,
                    "skip_region_validation": null,
                    "sse_customer_key": null,
                    "sts_endpoint": null,
                    "token": null,
                    "workspace_key_prefix": null
                },
                "hash": 1234542354
            },
            "modules": [
                {
                    "path": [
                        "root"
                    ],
                    "outputs": {},
                    "resources": {},
                    "depends_on": []
                }
            ]
        }
        ```
  - stateファイルの管理で推奨方法S3 or Dynamo
    - s3が便利な点
      - stateファイルの中には機密情報が含まれることがあるが、S3は暗号機能が備わっている
      - バージョニング機能があるので、過去の状態に戻す作業時に活用できる
    - Dynamoが便利な点
      - Dynamo DB lockを用いたterraform applyの排他制御
        - 同時にterraform applyをできないようにできる
          - AとBが同時にapplyするとstateファイルが衝突して事故る可能性がある（運用方法で回避はできる）
  - terraformによる管理対象のリソースは以下のコマンドで確認できる
    ```terraform
    $ terraform state list
    module.rds.data.aws_vpc.vpc
    module.ssm.aws_ssm_parameter.hoge
    module.rds.module.rds.aws_rds_cluster.default
    module.rds_proxy.module.rds_proxy.aws_iam_role_policy.rds_proxy
    module.s3.module.s3.aws_s3_bucket.hoge
    module.vpc.module.vpc.aws_internet_gateway.igw
    module.vpc.module.vpc.aws_nat_gateway.natgw[0]
    module.vpc.module.vpc.aws_route_table.private_route_table[0]
    module.vpc.module.vpc.aws_route_table.public_route_table
    module.vpc.module.vpc.aws_route_table_association.private_route_table_association[0]
    module.vpc.module.vpc.aws_subnet.private_subnet[0]
    module.vpc.module.vpc.aws_subnet.private_subnet[1]
    module.vpc.module.vpc.aws_subnet.public_subnet[0]
    module.vpc.module.vpc.aws_subnet.public_subnet[1]
    module.vpc.module.vpc.aws_vpc.vpc
    ```
- tfsecのエラーの無視の仕方
  ```
  resources hoge {
    hoge = 90 #tfsec:ignroe:AWS006
  }
  ```
