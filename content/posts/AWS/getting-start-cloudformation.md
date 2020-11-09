+++
title = "aws-cliで始めるCloudFormation"
date = 2019-03-05
tags = ["AWS", "aws-cli", "CloudFormation"]
draft = false
+++

#### 目的

- CloudFormationで使われる用語を理解する
- CloudFormationの構成を理解する
- 出来るだけaws-cliのコマンドを実行していく

<!--more-->

#### 前提条件

- aws-cliがインストール済み
- AWSのaccess key発行済
- access keyを環境変数に設定済み
- CFn実行にS3にテンプレートが保存される動作は割愛する
- stack-nameが違っていても気にしない。ブログ記述時の試行錯誤によるもの。

#### テンプレート

CloudFormation(以後、CFn)では、リソースを記述するファイルを`CFnテンプレート`を呼ぶ。  
CFnテンプレートの記法は、`JSON` or `YAML`。

```yaml:example.yaml
AWSTemplateFormatVersion: 2010-09-09
Description: Create coco's vpc
Resources:
  CocoFirstVPC:
    Type: 'AWS::EC2::VPC'
    Properties: 
      CidrBlock: 10.10.0.0/16
```


#### スタック

関連リソースを管理するための単一ユニットを`スタック`という。  
スタックを作成、更新、削除することでリソースのコレクションを作成、更新、削除する。
スタック内の全てのリソースはCFnテンプレートで定義される。
スタック作成時にパラメータを指定することで、インスタンスタイプなどをスタック作成時に指定できる。

ひとまず、ここまでで一度CFnをaws-cliから実行してみる。
```bash
# stack作成
(*'-') <<< aws cloudformation create-stack --stack-name coco-vpc --template-body file://./example.yaml
{
    "StackId": "arn:aws:cloudformation:ap-northeast-1:012345678901:stack/coco-vpc/4a6f6260-3f09-11e9-a59f-066cec78e1f8"
}
```
</details>

<details>
<summary>stackの情報取得</summary>
```bash
(*'-') <<< aws cloudformation describe-stacks
{
    "Stacks": [
        {
            "StackId": "arn:aws:cloudformation:ap-northeast-1:012345678901:stack/coco-vpc/4a6f6260-3f09-11e9-a59f-066cec78e1f8",
            "StackName": "coco-vpc",
            "Description": "Create coco's vpc",
            "CreationTime": "2019-03-05T05:41:14.771Z",
            "RollbackConfiguration": {},
            "StackStatus": "CREATE_COMPLETE",
            "DisableRollback": false,
            "NotificationARNs": [],
            "Tags": [],
            "DriftInformation": {
                "StackDriftStatus": "NOT_CHECKED"
            }
        }
    ]
}
```
</details>

<details>
<summary>stack一覧</summary>
```bash
(*'-') <<< aws cloudformation list-stacks
{
    "StackSummaries": [
        {
            "StackId": "arn:aws:cloudformation:ap-northeast-1:012345678901:stack/coco-vpc/4a6f6260-3f09-11e9-a59f-066cec78e1f8",
            "StackName": "coco-vpc",
            "TemplateDescription": "Create coco's vpc",
            "CreationTime": "2019-03-05T05:41:14.771Z",
            "LastUpdatedTime": "2019-03-05T06:52:30.668Z",
            "DeletionTime": "2019-03-05T06:56:27.631Z",
            "StackStatus": "DELETE_COMPLETE",
            "DriftInformation": {
                "StackDriftStatus": "NOT_CHECKED"
            }
        }
    ]
}
```
</details>

<details>
<summary>作成したVPCの情報取得</summary>
```bash
(*'-') <<< aws ec2 describe-vpcs
{
    "Vpcs": [
        {
            "CidrBlock": "10.10.0.0/16",
            "DhcpOptionsId": "dopt-6d45040a",
            "State": "available",
            "VpcId": "vpc-0ff5de1320b84a7a0",
            "OwnerId": "012345678901",
            "InstanceTenancy": "default",
            "CidrBlockAssociationSet": [
                {
                    "AssociationId": "vpc-cidr-assoc-0cb726d4c403e6aa6",
                    "CidrBlock": "10.10.0.0/16",
                    "CidrBlockState": {
                        "State": "associated"
                    }
                }
            ],
            "IsDefault": false,
            "Tags": [
                {
                    "Key": "aws:cloudformation:stack-name",
                    "Value": "coco-vpc"
                },
                {
                    "Key": "aws:cloudformation:stack-id",
                    "Value": "arn:aws:cloudformation:ap-northeast-1:012345678901:stack/coco-vpc/4a6f6260-3f09-11e9-a59f-066cec78e1f8"
                },
                {
                    "Key": "aws:cloudformation:logical-id",
                    "Value": "CocoFirstVPC"
                }
            ]
        }
    ]
}
```
</details>

```bash
# stack削除
(*'-') <<< aws cloudformation delete-stack --stack-name coco-vpc
```

CFnテンプレートから簡単なスタック作成/削除ができた。


#### 変更セット
スタックで実行中のリソースに変更を加える場合、スタックを更新する。
リソースに変更を加える前に変更案の概要となるのが`changeset(変更セット)`。

ここでは、前述で作成したVPCに対して、`tag:Name`としてcoco-vpcを与える。

CFnテンプレートにtagを追加する記述を追加
```yaml
AWSTemplateFormatVersion: 2010-09-09
Description: Create coco's vpc
Resources:
  CocoFirstVPC:
    Type: 'AWS::EC2::VPC'
    Properties: 
      CidrBlock: 10.10.0.0/16
# START changeset target
      Tags:
        -
          Key: Name
          Value: coco-vpc
# END changeset target
```

変更したCFnテンプレートを用いて、changesetを作成。その後、changesetの確認と適用を行う。

```bash
# changeset作成
(*'-') <<< aws cloudformation create-change-set --stack-name coco-vpc --change-set-name first-changeset --template-body file://./example.yaml
{
    "Id": "arn:aws:cloudformation:ap-northeast-1:012345678901:changeSet/first-changeset/8c6d9520-df98-4fae-b00c-28c0267703fc",
    "StackId": "arn:aws:cloudformation:ap-northeast-1:012345678901:stack/coco-vpc/4a6f6260-3f09-11e9-a59f-066cec78e1f8"
}
```

<details>
<summary>changeset一覧</summary>
```bash
(*'-') <<< aws cloudformation list-change-sets --stack-name coco-vpc
{
    "Summaries": [
        {
            "StackId": "arn:aws:cloudformation:ap-northeast-1:012345678901:stack/coco-vpc/4a6f6260-3f09-11e9-a59f-066cec78e1f8",
            "StackName": "coco-vpc",
            "ChangeSetId": "arn:aws:cloudformation:ap-northeast-1:012345678901:changeSet/first-changeset/8c6d9520-df98-4fae-b00c-28c0267703fc",
            "ChangeSetName": "first-changeset",
            "ExecutionStatus": "AVAILABLE",
            "Status": "CREATE_COMPLETE",
            "CreationTime": "2019-03-05T06:45:17.395Z"
        }
    ]
}
```
</details>

<details>
<summary>changeset確認</summary>
```bash
(*'-') <<< aws cloudformation describe-change-set --stack-name coco-vpc --change-set-name first-changeset
{
    "Changes": [
        {
            "Type": "Resource",
            "ResourceChange": {
                "Action": "Modify",
                "LogicalResourceId": "CocoFirstVPC",
                "PhysicalResourceId": "vpc-0ff5de1320b84a7a0",
                "ResourceType": "AWS::EC2::VPC",
                "Replacement": "False",
                "Scope": [
                    "Tags"
                ],
                "Details": [
                    {
                        "Target": {
                            "Attribute": "Tags",
                            "RequiresRecreation": "Never"
                        },
                        "Evaluation": "Static",
                        "ChangeSource": "DirectModification"
                    }
                ]
            }
        }
    ],
    "ChangeSetName": "first-changeset",
    "ChangeSetId": "arn:aws:cloudformation:ap-northeast-1:012345678901:changeSet/first-changeset/8c6d9520-df98-4fae-b00c-28c0267703fc",
    "StackId": "arn:aws:cloudformation:ap-northeast-1:012345678901:stack/coco-vpc/4a6f6260-3f09-11e9-a59f-066cec78e1f8",
    "StackName": "coco-vpc",
    "Description": null,
    "Parameters": null,
    "CreationTime": "2019-03-05T06:45:17.395Z",
    "ExecutionStatus": "AVAILABLE",
    "Status": "CREATE_COMPLETE",
    "StatusReason": null,
    "NotificationARNs": [
        "arn:aws:sns:ap-northeast-1:012345678901:slack"
    ],
    "RollbackConfiguration": {
        "RollbackTriggers": []
    },
    "Capabilities": [],
    "Tags": null
}
```
</details>

```bash
# changeset適用
(*'-') <<< aws cloudformation execute-change-set --stack-name coco-vpc --change-set-name first-changeset
```

以上が、CFnの用語説明とaws-cliからのstack操作/changeset操作の基本的な使い方になる。

### まとめ

Terraformとの比較

| CloudFormation  | Terraform                         |
|-----------------|-----------------------------------|
| CFnテンプレート | tfファイル                        |
| stack           | module(?)                         |
| parameter       | var                               |
| changeset作成   | terraform plan --out <file>のfile |
| changeset適用   | terraform apply <file>            |


