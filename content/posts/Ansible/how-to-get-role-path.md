+++
title = "ansibleでの各roleのパス取得方法"
date = 2018-10-15
tags = ["Ansible"]
draft = false
+++

playbookをrole毎に分けた際、roleの中でfilesのpath指定をするために実行中の/path/to/role/{current}を知りたくなった。

ref: [Accessiing information about other hosts with magic variables](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#accessing-information-about-other-hosts-with-magic-variables)
<!--more-->

```yaml
# roles/anyenv/tasks/main.yml
- name: test
  debug:
    var: role_path
```

```bash
# 実行結果
TASK [anyenv : test] ***************************************
ok: [192.168.33.11] => {
    "role_path": "/vagrant/roles/anyenv"
}
```
