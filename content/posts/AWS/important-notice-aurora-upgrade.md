+++
title = "AuroraのUpgradeでの注意点"
date = 2018-05-03
tags = ["AWS", "AuroraMySQL"]
draft = false
+++

Auroraのアップグレードには何種類かあり、それぞれでダウンタイム有無やパッチ適用時間が異なる。
アップグレードの種類と気にしなければいけないことをまとめた。

<!--more-->


---
## アップグレード適用種類

- 通常: 20～30s
- ZDP: 0s

---
## ZDP

- writerにしか適用されない
- reader適用外

---
## ZDP適用条件

- 実行時間が長いクエリが進行中である
- 実行時間が長いトランザクションが開いている
- バイナリログ記録が有効になっている
- バイナリログのレプリケーションが実行中である
- パラメータの変更が保留中である
- 一時テーブルが使用中である
- テーブルロックが使用中である
- オープン SSL 接続がある

---
## ZDPかの判定

> ZDPが発動するか事前に確認するコマンドや表示項目はない。
> AWSサポートに聞いてもわからない。
> 利用者がZDP発動条件に当てはまるのかどうかを確認しないといけない。
> 実際にアップグレードを実施してみないとZDPアップグレードなのか、通常の再起動アップグレードなのかわからない。

---

## アップグレード有無チェック

```bash
$ aws rds describe-pending-maintenance-actions
{
    "PendingMaintenanceActions": []
}
```

```bash
$ aws rds describe-pending-maintenance-actions
{
    "PendingMaintenanceActions": [
        {
            "ResourceIdentifier": "arn:aws:rds:ap-northeast-1:xxxxxxxxxxx:cluster:cluster-auth-prd",
            "PendingMaintenanceActionDetails": [
                {
                    "Description": "Aurora 2.02.3 release",
                    "Action": "system-update"
                }
            ]
        }
    ]
}
```

---

## 参考サイト

- [Amazon Aurora のアップグレード](https://qiita.com/tonishy/items/542f7dd10cc43fd299ab)
- [Amazon RDSのメンテナンスについて調べてみた | 本日も乙](http://blog.jicoman.info/2017/01/rds_maintenance/)
- [Auroraメンテナンス、アップグレード作業で得た教訓](https://qiita.com/tkyamada112/items/cd5882aadcb2095cecd6)
