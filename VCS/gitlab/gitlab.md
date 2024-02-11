GitLab
===

## 環境構築

```yaml
version: '3.6'
services:
  gitlab:
    image: 'gitlab/gitlab-ce:latest'
    #image: gitlab/gitlab-ee:latest
    container_name: gitlab
    restart: always
    hostname: gitlab
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        # Add any other gitlab.rb configuration here, each on its own line
        external_url 'https://gitlab.local'
    ports:
      - '80:80'
      - '443:443'
      - '22:22'
    volumes:
      - './volume/config:/etc/gitlab'
      - './volume/logs:/var/log/gitlab'
      - './volume/data:/var/opt/gitlab'
    shm_size: '256m'
```
* [Install GitLab using Docker](https://docs.gitlab.com/ee/install/docker.html)
* [docker-composeでGitLab](https://qiita.com/kujiraza/items/f87d2a9fb42ff227d3e6)


## 初回起動

1. STATUS が `health: starting` ⇒ `healthy` になるまで待つ (15分くらい？)

    ```text
    CONTAINER ID   IMAGE                     COMMAND             CREATED         STATUS                         
    dc18d732b698   gitlab/gitlab-ce:latest   "/assets/wrapper"   4 minutes ago   Up 9 seconds (health: starting)
    ```
    ↓
    ```text
    CONTAINER ID   IMAGE                     COMMAND             CREATED         STATUS                         
    dc18d732b698   gitlab/gitlab-ce:latest   "/assets/wrapper"   19 minutes ago   Up 15 minutes (healthy)
    ```

2. 初回ログイン

    デフォルトユーザ名は `root` で、デフォルトパスワードは下記コマンドで取得できます。

    ```bash
    docker exec gitlab cat /etc/gitlab/initial_root_password
    ```

    ![](./image/00_gitlab_login.png)

