+++
title = "AWSでのシェルスクリプト集"
date = 2018-05-02
tags = ["AWS", "ShellScript"]
draft = false
+++

便利なシェルスクリプト@AWS

<!--more-->

```shell
function ssh-peco() {
  local user="ec2-user"
  local host=$(aws ec2 describe-instances \
                 --region ap-northeast-1 \
                 --output json \
                 --filters "Name=instance-state-code,Values=16" | \
               jq -r '.Reservations[].Instances[] | [.Tags[] | select(.Key == "Name").Value][] + "\t" + .InstanceType + "\t" + .PrivateIpAddress + "\t" + .State.Name' |\
               awk '{printf "%-30s %-15s %-15s %-10s\n",$1,$2,$3,$4}' |\
               sort |\
               peco | awk '{print $3}')
  ssh -i ~/.ssh/key/auth-dev.pem \
      -o StrictHostKeyChecking=no \
      -o UserKnownHostsFile=/dev/null \
      "$user@$host"
}
```
