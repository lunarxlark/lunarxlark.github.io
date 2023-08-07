---
title: Raspberry Pi 4 Model B でKubernetes cluster構築した
tags: ["kubernetes", "raspberry-pi"]
date: 2023-08-07
published: true
---

完成形です。いぇい。
![完成形](/images/rpi_cluster.jpg)

---
## 材料

| 材料 | 個数 |
|---|-----|
| [Raspberry Pi 4 Model B 8GB](https://www.switch-science.com/products/6370) | 4 |
| [PoE HAT(E)](https://www.switch-science.com/products/8726) | 4 |
| [PoE対応 スイッチングハブ](https://www.amazon.co.jp/gp/product/B0763TGBTS/ref=ppx_yo_dt_b_asin_title_o08_s00?ie=UTF8&psc=1) | 1 |
| [無線ルーター](https://www.amazon.co.jp/gp/product/B0B4S2V99D/ref=ppx_yo_dt_b_asin_title_o07_s00?ie=UTF8&psc=1) | 1 |
| MicroSD 128GB | 4 |
| LAN (0.15m)| 4 |
| LAN (0.3m)| 1 |
| [ケース](https://www.amazon.co.jp/gp/product/B07TJ15YL1/ref=ppx_yo_dt_b_asin_title_o07_s01?ie=UTF8&th=1) | 1 |
| [追加スペーサー](https://www.amazon.co.jp/gp/product/B08FT76RFX/ref=ppx_yo_dt_b_asin_title_o03_s00?ie=UTF8&psc=1) | 50個入り |


---
## 組み立ての感想

見た目をスッキリさせたかったので、ラズパイの電源は PoE HATによりハブからLANで取ってます。  
ただ、これだとLANを引っこ抜いてノードを切り離すことが出来ないので失敗したかなって思ってます。  
LAN引っこ抜いてノードネットワーク障害ごっこしたいのに、電源も落ちちゃう。  
cf. [PoE (Power Over Ethernet)](https://ja.wikipedia.org/wiki/Power_over_Ethernet)。  

また、ケースでもPoE HATで失敗しました。  
PoE HATを付けると高さが高くなります。(値段も)  
買ったケースのファンとPoE HATのIsolation Transformerがぶつかります。  
なので、ケース付属スペーサーだけでは足りないのでスペーサーを買い足しました。  
(そして、そのスペーサーが付属スペーサーのネジ径と合わない...。付属のはM2だったっぽい...。)  
付属スペーサーは高さ20mm。このケース(+ファン) + PoE HAT の場合、30mmでぴったり合います。  
Amazonで激安のPoE HATはさらに高くなるので、ご注意を。


---
## k8s cluster 構築

### 0. 別PCでの準備

1. 無線ルーターを中継機として既存LANに参加させます。
2. Raspberry Pi Imagerを使ってmicroSDカード4枚にRaspberry Pi OS(64-bit) Liteを書き込みます。
    - Imagerでホスト名やssh有効化、user/passwd設定等出来るのでここでやっても可。

中継機が192.168.8.1なので、192.168.8.xで4台に固定IPを割り当てていきます。
- 192.168.8.101 ... rpi1 コントロールプレーン
- 192.168.8.102 ... rpi2 ワーカーノード
- 192.168.8.103 ... rpi3 ワーカーノード
- 192.168.8.104 ... rpi4 ワーカーノード


---
### 1. ホスト名設定

```
$ sudo vi /etc/hostname
rpix   # <- xには識別番号
```

```
$ sudo vi /etc/hosts
...
127.0.0.1 rpix

192.168.8.101 rpi1 # <- あとでIPアドレスを固定
192.168.8.102 rpi2
192.168.8.103 rpi3
192.168.8.104 rpi4
```

```
$ reboot
```


---
### 2. インターネット接続 & timezone

```
$ sudo raspi-config
```

`1 System Options` > `S1 Wireless LAN`
1. `JP` を選択
2. SSIDを入力
3. アクセスポイントのパスワードを入力
4. `5 localisation options` > `L2 Timezone` > `Asia` > `Tokyo`を選択

> WLANで一度国を選択すると次から国の選択が出てきませんでした。  
> 後述するファイルに国コード `contry` を `JP` に修正することで対応出来ます。  


---
### 3. Tailscale

[Install with one command](https://tailscale.com/download/linux)からインストールします。

```
$ curl -fsSL https://tailscale.com/install.sh | sh
...
Installation complete! log in to start using Tailscale by running:

sudo tailscale up
```

```
$ sudo tailscale up --ssh # <-- tailscale sshを有効に。

to Authenticate, visit:

    https://login.tailscale.com/a/xxxxxxxx
```

このURLを入れるのが一番しんどかったです。  
(上記URLへアクセスし認証を通せばいいので、写真撮ってOCRからiPhoneでアクセスしました。fが£になるのが辛かった。)  
tailscaleのadminコンソールに表示されるホスト名は、クライアントPC側でホスト名を変更したら更新されます。  
ラズパイに設定するホスト名でtailscaleでも管理したいので特に触らない。

admin consoleから接続した端末に対して `disable key expiry` 。  
これ以降はtailscale ssh で接続して作業します。  
(weztermがpaneのsyncronizeしない/必要に感じてないそうなので、tmuxでやっていきます。)


---
### 4. リポジトリ&パッケージ更新

```
sudo apt update
sudo apt upgrade -y
```


---
### 5. IPアドレス固定

```
$ sudo vi /etc/dhcpcd.conf
...
# Example static IP configuration:
interface eth0
static ip_address=192.168.8.10x/24
static routers=192.168.8.1
static domain_name_servers=192.168.8.1 8.8.8.8

...
```


---
### 6. kubeadmの必要要件を満たす

kubeadmのインストールを [始める前に](https://kubernetes.io/ja/docs/setup/production-environment/tools/kubeadm/install-kubeadm/)に記載があるので設定する

#### 6.1. swap無効化

```
$ sudo swapon   # 現状確認
NAME      TYPE SIZE USED PRIO
/var/swap file 100M.  0B.  -2

$ sudo dphys-swapfile swapoff
$ sudo systemctl stop dphys-swapfile
$ sudo systemctl disable dphys-swapfile

# 設定確認
$ sudo swapon
$ #設定なし
```

#### 6.2. cgroupのmemory有効化

```
$ cat /proc/cgroups   # 現状確認
#subsys_name    hierarchy       num_cgroups     enabled
cpuset  0       80      1
cpu     0       80      1
cpuacct 0       80      1
blkio   0       80      1
memory  0       80      0   <-- 無効になってる
devices 0       80      1
freezer 0       80      1
net_cls 0       80      1
perf_event      0       80      1
net_prio        0       80      1
pids    0       80      1
# 有効化(末尾にcgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memoryを追加)
$ sudo sed -i "s/$/ cgroup_enable=cpuset cgroup_memory=1 cgroup_enable=memory/" /boot/cmdline.txt
$ sudo reboot
```


---
### 7. containerd/runc/CNI pluginsのインストール + α

コンテナランタイムとしてcontainerdを使用します。  
docker engineを使用しても引き続きk8sは動くようですが、  
必要最小限の構成にしたいのでcontainerdを入れます。  

[containerdの使用を開始する](https://github.com/containerd/containerd/blob/main/docs/getting-started.md)に従います。  
2023/8/6時点での各ツールの最新バージョンは下記です。

| tools | latest |
|-------|--------|
| contaienrd | 1.7.3 |
| runc | 1.1.8 |
| CNI plugins | 1.3.0 |

#### 7.1. containerd

執筆時点のaptだとv1.4.13が降ってきて、cgroup API v1を使おうとしますが、 他ツールではcgroup API v2を使うバージョンが降ってきます。  
結果、cgroup APIのバージョンが違うじゃねーかってkubeletに怒られるのでここではcontainerdのバージョンを指定できる方法でインストールします。  

```
$ cd /tmp
$ curl -LO https://github.com/containerd/containerd/releases/download/v1.7.3/containerd-1.7.3-linux-arm64.tar.gz
$ sudo tar Cxzvf /usr/local containerd-1.7.3-linux-arm64.tar.gz
$ # systemctl unitファイルのダウンロード
$ sudo mkdir -p /usr/local/lib/systemd/system
$ sudo curl https://raw.githubusercontent.com/containerd/containerd/main/containerd.service -o /usr/local/lib/systemd/system/containerd.service
$ sudo systemctl daemon-reload
$ sudo systemctl enable --now containerd
```

#### 7.2. runc

```
$ cd /tmp
$ curl -LO https://github.com/opencontainers/runc/releases/download/v1.1.8/runc.arm64
$ sudo install -m 755 runc.arm64 /usr/local/sbin/runc
```

#### 7.3. CNI plugins

```
$ sudo mkdir -p /opt/cni/bin
$ curl -LO https://github.com/containernetworking/plugins/releases/download/v1.3.0/cni-plugins-linux-arm-v1.3.0.tgz
$ sudo tar Cxzvf //cni/bin cni-plugins-linux-arm-v1.3.0.tgz
```

#### 7.4. IPv4フォワーディングを有効化し、iptablesからブリッジされたトラフィックを見えるようにする

[こちらの設定](https://kubernetes.io/ja/docs/setup/production-environment/container-runtimes/#%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB%E3%81%A8%E8%A8%AD%E5%AE%9A%E3%81%AE%E5%BF%85%E9%A0%88%E8%A6%81%E4%BB%B6)に従います。  
(わかってないので後で調べる。)  

```
$ cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

$ sudo modprobe overlay
$ sudo modprobe br_netfilter
$ lsmod | grep br_netfilter
$ lsmod | grep overlay
```

```
# この構成に必要なカーネルパラメーター、再起動しても値は永続します
$ cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF

# 再起動せずにカーネルパラメーターを適用
$ sudo sysctl --system

# `br_netfilter`と`overlay`モジュールが読み込まれていることを確認
$ sudo sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward

# 各カーネルパラメータがて1に設定されていることを確認
sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
```

#### 7.5. cgroupドライバーにsystemdを設定

初期設定ファイルを出力してsystemdを指定する。
(公式に沿うならファイル開いて場所を確認した方がいい。)

```
$ sudo mkdir -p /etc/containerd
$ containerd config default | sed "s/SystemdCgroup = false/SystemdCgroup = true/" | sudo tee /etc/containerd/config.toml
$ sudo systemctl restart containerd
```


---
### 8. `kubeadm` / `kubelet` / `kubectl` のインストール

[公式の手順](https://kubernetes.io/ja/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#kubeadm-kubelet-kubectl%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB)に沿う

```
# 1. `apt`のパッケージ一覧を更新し、Kubernetesの`apt`リポジトリを利用するのに必要なパッケージをインストールします:
$ sudo apt-get install -y apt-transport-https ca-certificates curl

# 2. Google Cloudの公開鍵をダウンロードします:
$ curl -fsSL https://dl.k8s.io/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /usr/share/keyrings/kubernetes-archive-keyring.gpg

# 3. Kubernetesの`apt`リポジトリを追加します:
$ echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

# 4. `apt`のパッケージ一覧を更新し、kubelet、kubeadm、kubectlをインストールします。そしてバージョンを固定します:
$ sudo apt-get update
$ sudo apt-get install -y kubelet kubeadm kubectl
$ sudo apt-mark hold kubelet kubeadm kubectl
```

#### 8.1. kubeletで使用されるcgroupドライバーにsystemdを指定

```
$ sudo mkdir /var/lib/kubelet
$ echo "KUBELET_EXTRA_ARGS=--cgroup-driver=systemd" | sudo tee /var/lib/kubelet/kubeadm-flags.env
$ sudo systemctl daemon-reload
$ sudo systemctl restart kubelet
```


#### 8.2. k8sクラスタ作成 (コントロールプレーンノードでのみ)

```
$ sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --control-plane-endpoint=rpi1 --apiserver-cert-extra-sans=rpi1

...

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a Pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  /ja/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join <control-plane-host>:<control-plane-port> --token <token> --discovery-token-ca-cert-hash sha256:<hash>


You can now join any number of control-plane nodes by copying certificate authorities
and service account keys on each node and then running the following as root:

  kubeadm join <control-plane-host>:<control-plane-port> \
        --token <token> \
        --discovery-token-ca-cert-hash sha256:<hash>\
        --control-plane

Then you can join any number of worker nodes by running the following on each as root:

  kubeadm join <control-plane-host>:<control-plane-port> \
        --token <token> \
        --discovery-token-ca-cert-hash sha256:<hash>
```


#### 8.3. k8sクラスタへの参加

コントロールプレーンに参加するコマンドとワーカーノードに参加するコマンドが表示されます。  
残り3台はワーカーノードに参加させたいので、後ろのコマンドを3台に実施します。  

```
$ kubeadm join <control-plane-host>:<control-plane-port> \
    --token <token> \
    --discovery-token-ca-cert-hash sha256:<hash>
```

> もし、上記joinコマンドを忘れてしまった場合、下記で確認出来ます。  
> `kubeadm token create --print-join-command`


#### 8.4. kubectlを通常ユーザで使えるように

通常ユーザでもkubectlを実行できる様に `kubeadm init` の結果に表示されているコマンドを実行します。

```
$ mkdir -p $HOME/.kube
$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```


### 9. pod間通信用のネットワークアドオン `flannel` インストール

[こちらの手順](https://github.com/flannel-io/flannel#deploying-flannel-with-kubectl)を実施する。  

```
$ sudo kubectl apply -f https://github.com/flannel-io/flannel/releases/latest/download/kube-flannel.yml
```


### 10. 動作確認

上記を実行後にしばらくして、ワーカーノードが `Ready` になっていればOK。

```
$ kubectl get nodes
NAME   STATUS   ROLES           AGE   VERSION
rpi1   Ready    control-plane   23h   v1.27.4
rpi2   Ready    <none>          22h   v1.27.4
rpi3   Ready    <none>          23h   v1.27.4
rpi4   Ready    <none>          23h   v1.27.4
```

おわり。  
そこそこ面倒だったので、Nixで一発で入れられるようにしたいです。  
([ここ](https://nixos.wiki/wiki/Kubernetes)にほぼ書いてありそうだけど、k8sのお勉強の方を優先しなきゃ...)  
