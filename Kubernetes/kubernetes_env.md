Kubernetes 環境
===

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

### Docker Registry

> [!NOTE]
> Docker Registry に登録したコンテナイメージを、ブラウザの削除ボタンから削除できません (ボタンは押下できるのだけれど。。。)

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

### kubectl コマンドの補完

* bash

    ```bash
    kubectl completion bash >> $HOME/.bashrc
    source $HOME/.bashrc
    ```
