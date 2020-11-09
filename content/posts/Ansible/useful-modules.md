+++
title = "Ansibleでよく使うモジュール"
date = 2018-09-30
tags = ["Ansible"]
draft = false
+++

<!--more-->

## blockinfile : 複数行を扱えるfileモジュール

[blockinfile - Insert/update/remove a text block surrounded by marker lines](https://docs.ansible.com/ansible/latest/modules/blockinfile_module.html?highlight=blockinfile#blockinfile-insert-update-remove-a-text-block-surrounded-by-marker-lines)

```yaml
- hosts: all
  roles:
    - yaegashi.blockinfile
  tasks:
    - name: 特定の場所に追加
      blockinfile:
        path: ./test.txt
        insertafter: '^# xxxx'
        marker: "<!-- {mark} ANSIBLE MANAGED BLOCK -->"
        block: |
          1行目
          2行目
```

---

## unarchive

[unarchive - Unpacks an archive after (optionally) copying it from the local machine.
](https://docs.ansible.com/ansible/latest/modules/unarchive_module.html?highlight=unarchive)

```yaml
- name: install peco
  become: yes
  unarchive:
    remote_src: yes
    src: "{{ peco_bin_url }}"
    dest: /usr/local/bin/
    mode: 0755
    owner: root
    group: root
    extra_opts:
      - --strip-components=1
      - peco_linux_amd64/peco
```
