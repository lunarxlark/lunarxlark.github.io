---
title : aws-cliの結果をJMESPathで取得する方法
tags : ["JMESPath", "aws-cli"]
date : 2018-06-10
published : true
---

+ tagやownerでfilterして、作成日でソートした結果のうちImageIdだけ取り出す

<!--more-->

```bash
$ aws ec2 describe-images \
  --filters \
    Name=owner-id,Values=1234512345 \
    Name=tag:Branch,Values=master \
    Name=tag:Role,Values=ap \
  --query "reverse(sort_by(Images, &CreationDate)[].ImageId)"
# option:query解説
# mapのImagesをCreationDateでsortして、ImageIdを抽出
# 結果を逆順にする
```

---

+ 取り出すモノを複数指定

```bash
$ aws ec2 describe-images \
  --filters \
    Name=owner-id,Values=1234512345 \
    Name=tag:Branch,Values=master \
    Name=tag:Role,Values=ap \
  --query "reverse(sort_by(Images, &CreationDate)[].[ImageId, Name])"
# option:query解説
# mapのImagesをCreationDateでsortして、ImageIdを抽出
# 結果を逆順にする
```

---

+ 上記の結果の内、最新1行のみ取得

```bash
$ aws ec2 describe-images \
  --filters \
    Name=owner-id,Values=1234512345 \
    Name=tag:Branch,Values=master \
    Name=tag:Role,Values=ap \
  --query "reverse(sort_by(Images, &CreationDate)[].ImageId)[1]"
```

---

+ double-quoteを外したい

```bash
$ aws ec2 describe-images \
  --filters \
    Name=owner-id,Values=1234512345 \
    Name=tag:Branch,Values=master \
    Name=tag:Role,Values=ap \
  --query "reverse(sort_by(Images, &CreationDate)[].ImageId)[1]" \
  --output text
```

---

+ ラベルをつける

```bash
$ aws ec2 describe-images \
  --filters \
    Name=owner-id,Values=1234512345 \
    Name=tag:Branch,Values=master \
    Name=tag:Role,Values=ap \
  --query "reverse(sort_by(Images, &CreationDate)[].{imageid:ImageId, snapshotid: BlockDeviceMappings[0].Ebs.SnapshotId})[1]"
```

---

+ ラベルをつけて、特定ラベルの値だけ取得

```bash
$ aws ec2 describe-images \
  --filters \
    Name=owner-id,Values=1234512345 \
    Name=tag:Branch,Values=master \
    Name=tag:Role,Values=ap \
  --query "reverse(sort_by(Images, &CreationDate)[].{imageid:ImageId, snapshotid: BlockDeviceMappings[0].Ebs.SnapshotId})[1] | snapshotid" \
  --output text
> {
>   "imageid": "ami-xxxxxxxxxxxxxxxx",
>   "snapshotid": "snap-xxxxxxxxxxxxx"
> }
```

---

+ 元ネタ

```json
{
    "Images": [
        {
            "Hypervisor": "xen",
            "State": "available",
            "RootDeviceType": "ebs",
            "RootDeviceName": "/dev/xvda",
            "ImageType": "machine",
            "BlockDeviceMappings": [
                {
                    "Ebs": {
                        "VolumeSize": 8,
                        "SnapshotId": "snap-xxxxxxxxxx",
                        "VolumeType": "gp2",
                        "DeleteOnTermination": true,
                        "Encrypted": false
                    },
                    "DeviceName": "/dev/xvda"
                }
            ],
            "OwnerId": "1234512345",
            "ImageId": "ami-xxxxxxxxxxxxxxxx",
            "CreationDate": "2018-10-04T01:17:47.000Z",
            "Architecture": "x86_64",
            "VirtualizationType": "hvm",
            "ImageLocation": "xxxxxxxxxx/Pckr-ap-auth-2018-10-04T01-14-43Z",
            "Public": false,
            "Name": "Pckr-ap-auth-2018-10-04T01-14-43Z",
            "SriovNetSupport": "simple",
            "EnaSupport": true,
            "Tags": [
                {
                    "Key": "Made",
                    "Value": "packer"
                },
                {
                    "Key": "CommitId",
                    "Value": "61b0e7"
                },
                {
                    "Key": "Base_AMI_Name",
                    "Value": "amzn2-ami-hvm-2.0.20180810-x86_64-gp2"
                },
                {
                    "Key": "Branch",
                    "Value": "master"
                },
                {
                    "Key": "Role",
                    "Value": "ap"
                },
                {
                    "Key": "OS_Version",
                    "Value": "Amazon Linux 2 LTS"
                },
                {
                    "Key": "Base_AMI",
                    "Value": "ami-xxxxxxxxxxxxx"
                },
                    "Key": "Name",
                    "Value": "packer-ap"
                }
            ],
            "Description": "ami-ap-auth"
        },
        {
            "Hypervisor": "xen",
            ...
}
```
