Docker BuildKit は次世代の Docker のビルドツール。

詳細は他の記事に譲るとして、 macOS の Docker Desktop 最新版 (2.3.0.4) で BuildKit を有効にする方法をメモします。

Docker Desktop の [Preferences] > [Docker Engine] の configuration に以下の内容を記載 (追記) します。

```json
{
  "experimental": true,
  "features": {
    "buildkit": true
  }
}
```

![docker_desktop_configuration_buildkit.png](https://files.tearoom6.biz/bdfd83cb-3d5e-4e97-a07d-1697c1b41dab.png)

この内容は `~/.docker/daemon.json` に保存されます。

この設定を行っておけば、 CLI の `docker` command で実行する場合も含めて、自動で BuildKit が使われるようになります。 (`DOCKER_BUILDKIT=1` を渡す必要がない)

```sh
docker build .
```

Docker Compose を用いる場合は、引き続き `COMPOSE_DOCKER_CLI_BUILD=1` の方は指定する必要があります。

```sh
COMPOSE_DOCKER_CLI_BUILD=1 docker-compose up -d
```

> References

- [dockerd | Docker Documentation](https://docs.docker.com/engine/reference/commandline/dockerd/#daemon-configuration-file)
- [Faster builds in Docker Compose 1.25.1 thanks to BuildKit Support - Docker Blog](https://www.docker.com/blog/faster-builds-in-compose-thanks-to-buildkit-support/)
- [BuildKitによりDockerとDocker Composeで外部キャッシュを使った効率的なビルドをする方法 - Qiita](https://qiita.com/tatsurou313/items/ad86da1bb9e8e570b6fa)
