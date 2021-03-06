---
title: "Software Design 2022 1æå· Terraform"
emoji: "ð°"
type: "idea" # tech: æè¡è¨äº / idea: ã¢ã¤ãã¢
topics: ["Terraform"]
published: true
---

## ã¯ããã«

Software Design 2022å¹´1æå·ã®Terraformã®è¨äºã®åå¿é²

https://www.amazon.co.jp//dp/B09NQSQXLZ

## åå¿é²

- ã¯ã¬ãã³ã·ã£ã«æå ±ãåãæ±ããã¨ãå¤ãã®ã§ãã£ããPUSHããªãããã« [git-secrets](https://github.com/awslabs/git-secrets) ãå°å¥ãã¦ãã
  - è¨­å®ãä½¿ãæ¹ã«ã¤ãã¦ã¯[ãã¡ãã®è¨äº](https://zenn.dev/kkk777/articles/8f55db1e9678f2)ãããã¾ã¨ã¾ã£ã¦ãã
- ãã­ãã¤ãã«ã¤ãã¦
  - main.tfã«æ¸ããããªãã¤ã§ãAWSãAzureãGCPãªã©ã®ã©ã®å¤é¨ã¯ã©ã¦ããµã¼ãã¹ã®APIã¨é£æºããããè¨­å®ããç®æ
    ```terraform
    provider "aws" {
      region  = "ap-northeast-1"
      profile = "hoge-dev"
    }
    ```
  - ãã­ãã¤ãã«ããã¼ã¸ã§ã³ãããããã¼ã¸ã§ã³ãä¸ããã¨æ°ãããµã¼ãã¹ã®ç®¡çãTerraformããã§ããããã«ãªã£ãããã
    - terraform initãå®è¡ããã¨ .terraform/.terraform.lock.hcl ãä½æããã¦ããã­ãã¤ãã®ãã§ãã¯ãµã ãæ¸ãè¾¼ã¾ãã
      - githubã®ç®¡çå¯¾è±¡ã«ããã¦ãããã¨ã§ããã¼ã åã§ãã­ãã¤ãã®ãã¼ã¸ã§ã³ãåºå®ã§ãã
- tfstateãã¡ã¤ã«(.terraform/terraform.tfstate)
  - ã¤ã³ãã©ã®ç°å¢æå ±ã«ã¤ãã¦è¨è¼ããã¦ãããã¡ã¤ã«ã§ãç´æ¥ç·¨éããã¨terraformã³ãã³ããããªã½ã¼ã¹ã®å¤æ´ãã§ããªããªã£ã¦ãã¾ã
    - ãã®ãã¡ã¤ã«ã¯githubã§ `ç®¡çãã` S3ãªã©ã®å¤é¨ãªã½ã¼ã¹ä¸ã§ç®¡çãããã¨
      - githubã§ç®¡çããã¨ããã§jsonãã¡ã¤ã«ãªã®ã§è¯ããã©ããã®å¤æ­ãå°é£
      - terraform applyãããã¨ã«localã®stateãã¡ã¤ã«ã«å¤æ´ãå¥ãã®ã§ãå³åº§ã«mergeããªãã¨ä»ã®ã¡ã³ãã¼ã«å¤æ´ãå±æã§ããªã
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
  - stateãã¡ã¤ã«ã®ç®¡çã§æ¨å¥¨æ¹æ³S3 or Dynamo
    - s3ãä¾¿å©ãªç¹
      - stateãã¡ã¤ã«ã®ä¸­ã«ã¯æ©å¯æå ±ãå«ã¾ãããã¨ãããããS3ã¯æå·æ©è½ãåãã£ã¦ãã
      - ãã¼ã¸ã§ãã³ã°æ©è½ãããã®ã§ãéå»ã®ç¶æã«æ»ãä½æ¥­æã«æ´»ç¨ã§ãã
    - Dynamoãä¾¿å©ãªç¹
      - Dynamo DB lockãç¨ããterraform applyã®æä»å¶å¾¡
        - åæã«terraform applyãã§ããªãããã«ã§ãã
          - Aã¨Bãåæã«applyããã¨stateãã¡ã¤ã«ãè¡çªãã¦äºæãå¯è½æ§ãããï¼éç¨æ¹æ³ã§åé¿ã¯ã§ããï¼
  - terraformã«ããç®¡çå¯¾è±¡ã®ãªã½ã¼ã¹ã¯ä»¥ä¸ã®ã³ãã³ãã§ç¢ºèªã§ãã
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
- tfsecã®ã¨ã©ã¼ã®ç¡è¦ã®ä»æ¹
  ```
  resources hoge {
    hoge = 90 #tfsec:ignroe:AWS006
  }
  ```
