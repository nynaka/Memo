Kubernetes 環境設定・ツールのインストール
===

## ホスト OS の設定 その 1:

* SWAP の無効化

    `/etc/fstab` を参照すると下記の例のようにファイルシステムのタイプにあたるパラメータ (左から3番目) が `swap` の行があると思いますので、この行を削除、または、先頭に `#` を挿入してコメントに変更し、OS を再起動して、SWAP を無効化します。

    ```text
    /swap.img       none    swap    sw      0       0
    ↓
    #/swap.img       none    swap    sw      0       0
    ```

* IP アドレスの固定

    Control Plane の IP アドレスが変更されると `kubeadm reset` からやり直すことになります。  
    面倒ごとを避けるために、最初から IP アドレスは固定しておくことがおすすめです。


## [Docker](https://docs.docker.com/engine/install/ubuntu/)

* Ubuntu Linux

    1. 古いバージョンの削除 (インストールしていた場合)

        ```bash
        sudo apt-get remove docker docker-engine docker.io containerd runc
        ```

    2. APT リポジトリの設定

        * 関連ツールのインストール

            ```bash
            sudo apt-get update
            sudo apt-get -y install \
                ca-certificates \
                curl \
                gnupg \
                lsb-release
            ```

        * GPG key のインストール

            ```bash
            sudo mkdir -p /etc/apt/keyrings
            curl -fsSL https://download.docker.com/linux/ubuntu/gpg | \
                sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
            ```

        * リポジトリの追加

            ```bash
            echo \
                "deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
                "$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
                sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
            ```

        * Docker のインストール

            ```bash
            sudo apt-get update
            sudo apt-get install -y \
                docker-ce docker-ce-cli \
                containerd.io docker-buildx-plugin \
                docker-compose-plugin
            ```

        * ユーザ権限の設定

            ```bash
            sudo usermod -aG docker ubuntu
            ```

        * 自動起動設定

            ```bash
            sudo systemctl enable docker
            sudo systemctl start docker
            ```


## ホスト OS の設定 その 2: bridge 関連設定

たぶん Docker をインストール時に L2 関連のカーネルモジュールがインストールされるのだと思います。  
Docker インストール後に下記の設定ができるようになります。

```bash
echo "net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-arptables = 1" | \
    sudo tee /etc/sysctl.d/99-bridge-nf-call-iptables
sudo sysctl -p /etc/sysctl.d/99-bridge-nf-call-iptables 
```


## [kubeadm、kubelet、kubectlのインストール](https://kubernetes.io/ja/docs/setup/production-environment/tools/kubeadm/install-kubeadm/#kubeadm-kubelet-kubectl%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB)

* Debian / Ubuntu

    1. Kubernetes の apt リポジトリ利用のためのパッケージ追加

        ```bash
        sudo apt-get update
        sudo apt-get install -y apt-transport-https ca-certificates curl
        ```

    2. リポジトリ公開鍵のダウンロード

        ```bash
        curl -fsSL https://dl.k8s.io/apt/doc/apt-key.gpg | \
            sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg
        ```

    3. Kubernetes の apt リポジトリの追加

        ```bash
        echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | \
            sudo tee /etc/apt/sources.list.d/kubernetes.list
        ```

    4. kubelet、kubeadm、kubectlをインストール

        ```bash
        sudo apt-get update
        sudo apt-get install -y kubelet kubeadm kubectl
        sudo apt-mark hold kubelet kubeadm kubectl
        ```


## その他

### Docker Registry

インターネットで公開されているコンテナイメージではなく、自分でカスタマイズしたコンテナイメージを利用する場合は、ローカルネットワーク上に Docker Registry のようなサービスを作成しておくと便利です。

> [!NOTE]
> * Docker Registry に登録したコンテナイメージを、ブラウザの削除ボタンから削除できません (ボタンは押下できるのだけれど。。。)

* docker-compose.yml

    ```yaml
    version: "3"
    services:

        frontend:
            #image: konradkleine/docker-registry-frontend:v2
            image:  ekazakov/docker-registry-frontend
            container_name: frontend
            hostname: frontend
            environment:
                ENV_DOCKER_REGISTRY_HOST: backend
                ENV_DOCKER_REGISTRY_PORT: 5000
            ports:
                - 80:80

        backend:
            image: registry:2
            container_name: backend
            hostname: backend
            restart: always
            ports:
                - "5000:5000"
            #volumes:
            #    - ./var.lib.registry:/var/lib/registry
            environment:
                REGISTRY_STORAGE_DELETE_ENABLED: 'True'
    ```

* 起動コマンド

    ```bash
    docker compose up -d --build
    ```
