この記事は [🌊 UMITRON 🐟 Advent Calendar 2021](https://adventar.org/calendars/6652) **20日目** の記事です。
Advent Calendar も最終週ですね！

この記事は、以前から気になっていた Docker Compose V2 の経緯・現在地点について、自分なりに調べて、まとめておくものです。
あまり深堀り調査していないので、もし誤りがあれば、ご指摘ください。

---

結構以前だと思うのですが、いつものように `docker-compose` を使っていると、

> Docker Compose is now in the Docker CLI, try `docker compose up`

みたいなメッセージが表示されるようになりました。

この時点では、そうか、これからは `docker compose` と打てばいいのか (後で調べよう...)、くらいの理解でとりあえず使っていました。

ところで、最近、 `docker-compose` を使っても、またこのメッセージ出現しなくなっております。 (後述の `Use Docker Compose V2` のチェックを外したとしても)

この間、私を含む利用者はじわじわと、 Docker Compose の V1 から V2 と呼ばれる実装へ誘導されていたんですが、そもそも Compose V2 とはなんぞ、というところを先に書きます。

## Docker Compose V1 vs V2

もともとの `docker-compose` の実装、つまり Compose V1 (と便宜的に呼びます) は Docker client とは全く別のツールとして Python で書かれていました。

https://github.com/docker/compose/tree/v1.25.2

そのため、従来 Linux において Docker Compose CLI は Docker CLI に追加して、別途インストールする必要がありました。 (現時点では、まだそうなってます)

Docker Compose を Go で書かれている [Docker CLI](https://github.com/docker/cli) に組み込むには、 Compose V1 が Python で書かれていることがネックになっており、実験的なプロジェクトとして、 Go 版の Docker Compose (Compose CLI) の開発が [docker/compose-cli](https://github.com/docker/compose-cli) のリポジトリで進められたようです。

やがて、こちらのリポジトリは Docker Compose "Cloud Integrations" として、 Docker Compose のクラウド連携機能に特化するようになり、 Compose V2 (従来の docker-compose が提供する機能の代替) 部分は、もともとの Compose V1 のリポジトリ (https://github.com/docker/compose) に移管されることになったようです。

https://github.com/docker/compose/tree/v2.2.2

Compose V2 は Go でフルスクラッチで書き直されていることが分かりますね。

Compose V1 と V2 は概ね互換性はあるものの、完全ではないようで、一部のコマンドについては差異があると言及されています。
実装の違いによる細かい動作の違いや、画面上の表示の変化については、自分も使っている中でいくつか感じられます。

(スムーズな移行のために、 `docker-compose` を実行したときに、内部的に `docker compose` (Compose V2) のコマンドに変換して実行してくれる [Compose Switch](https://github.com/docker/compose-switch/) というツールも提供されているようです)

さて、 Compose V2 は Compose V1 の後継ではあるのですが、もちろん機能を引き継いだだけではなく、いくつかの改良・新機能が含まれています。

- profiles 機能 (環境を使い分けできるようになる)
- GPU アクセスをサポート
- Apple Silicon をサポート
- BuildKit をデフォルトで使用
- クラウド連携機能 (上述した https://github.com/docker/compose-cli で開発されている機能)

また、 Compose V2 の開発にあたっては、 Compose V1 の挙動を元にした [Compose Specification](https://github.com/compose-spec/compose-spec) が platform 非依存な標準仕様として策定され、それに準拠する実装として Compose V2 を開発する、という体制が整えられました。

## Docker Compose V2 への移行状況

およそここ1年間、 Docker Compose V2 への移行がどのように進められたかを、 macOS 版 Docker Desktop のみを対象に、漁ってみました。
(リアルタイムで追ってないので、詳しい状況はわからないけど)

- Docker Desktop 3.0.0 (2020-12-10)
   - Go 版の Compose CLI が内包され、 `docker` のサブコマンド `docker compose` で実行できるようになった
- Docker Desktop 3.4.0 (2021-06-09)
   - `docker compose` が Compose V2 beta として提供されるようになった
   - `docker-compose` を呼んだときに Compose V2 を使うかどうかを `docker-compose disable-v2`, `docker-compose enable-v2` によって切り替えられるようになった
- Docker Desktop 4.0.0 (2021-08-31)
   - (大企業向けに有料化)
- Docker Desktop 4.0.1 (2021-09-13)
   - Compose V2 が github.com/docker/compose でホストされるように変更された
- Docker Desktop 4.1.0 (2021-09-30)
   - `docker-compose` がデフォルトで Compose V2 を向くようになった
   - General settings (`Use Docker Compose V2` という項目) で `docker-compose` を呼んだときに Compose V2 を使うかどうか選択できるようになった
- Docker Desktop 4.3.1 (2021-12-11)
   - (今ここ)

ということで、 macOS 版 (たぶん Windows 版も) Docker Desktop をきちんと最新版にアップデートしている場合、明示的に OFF しない限りは、既に Compose V2 への移行が完了しているということですね。 (明示的に `Use Docker Compose V2` のチェックを外せば `docker-compose` コマンドで引き続き Compose V1 を利用することはできますが)

Linux 版の状況もあって Compose V2 は 2021-12-20 現在ではまだ GA (Generally Available) とはなっていませんが、もう GA になるのは時間の問題と思われます。

試しに、今、手元に入っている macOS 版 Docker Desktop 4.3.1 で実験してみます。

`Use Docker Compose V2` のチェックを外した状態では、 `docker-compose` では Compose V1, `docker compose` では Compose V2 がそれぞれ呼ばれます。

```sh
$ docker-compose version
docker-compose version 1.29.2, build 5becea4c
docker-py version: 5.0.0
CPython version: 3.9.0
OpenSSL version: OpenSSL 1.1.1h  22 Sep 2020

$ docker compose version
Docker Compose version v2.2.1
```

`Use Docker Compose V2` のチェックを付けた状態では、いずれも Compose V2 が呼ばれます。

```sh
$ docker-compose version
Docker Compose version v2.2.1

$ docker compose version
Docker Compose version v2.2.1
```

## Compose V2 のクラウド連携機能

ちょっと本筋と離れますが、もともとの "Compose CLI" であった、クラウド連携機能についても、個人的整理をしておきます。

Compose V2 のクラウド連携機能は docker-compose.yml で定義したマルチコンテナ環境を、簡単にクラウド環境にデプロイできるようにするもので、現時点で、以下のプラットフォームがサポートされています。

- Amazon Elastic Container Service (ECS)
- Microsoft Azure Container Instances (ACI)

Kubernetes についても対応が進められています。

こちらの機能については、元々 2020-07-09 に `docker ecs` というサブコマンドを追加する CLI Plugin がベータ版が公開されたものの流れを組むようです。
2020-11 には既に、今の "Compose CLI"、つまり `docker compose` として使えるものを公開したというアナウンスが出されています。

一方、上と同日の 2020-07-09 には AWS Copilot という、これまたコンテナ環境を ECS 環境にデプロイするためのツールが公開されていて、一体どっちを使えってこと?って混乱したのを覚えています。こちらのツールは元々は Amazon ECS CLI (`ecs-cli`) の流れを組むもののようです。

Compose V2 のクラウド連携機能は、当然 `docker-compose.yml` を中心に据えて、そこで定義したコンテナの集合体をクラウドで構築することを目標としているのに対し、 AWS Copilot の方は、個々の Dockerfile (コンテナ) をそれぞれ役割に応じて AWS のサービスにデプロイして、連携して動かすというアプローチの違いがあるようです。

ただ、似たツールなのは間違いないないですね。。
汎用 IaC ツールとして CDK や Terraform もありますし、正直、この手のツールは、今、乱立している感じはありますね。どれが覇権を取るのでしょう。。

さて、調査だけだと退屈なので、最後に簡単に動かしてみます。

コードはこちらに置いてあります。 (🐟漢字クイズです)

https://github.com/tearoom6/docker-ecs-tools-trials

Compose V2 と AWS Copilot を用いて AWS 環境にデプロイするのを試してみました。

まず Compose V2 の方を試してみます。

既存の `docker-compose.yml` がそのまま使えるかというと、全然そんなことはなく、いくつかの制約があるようです。
例えば、以下のようなエラーに遭遇しました。

- published port can't be set to a distinct value than container port: incompatible attribute (`ports` の mapping はコンテナ内外で同じにしなくてはいけない)
- ECS Fargate does not support bind mounts from host: incompatible attribute (Fargate の場合 `volumes` で host からのマウントができない)
- services.build: unsupported attribute (`Dockerfile` を指定したイメージのビルドに対応していない)

image は ECR や DockerHub などから、 public で公開されているものか、カスタムビルドして push しておいたものを引っ張ってくるしかないようです。

今回はまず ECR で repository を作っておいて `Dockerfile` から作った Docker image を push します。

```sh
aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/a9b0d6v6
docker build -t fish_quiz .
docker tag fish_quiz:latest public.ecr.aws/a9b0d6v6/fish_quiz:latest
docker push public.ecr.aws/a9b0d6v6/fish_quiz:latest
```

その上で docker-compose.yml を用意しておけば、以下のようにしてデプロイ可能です。
デプロイの裏側では CloudFormation が動いています。

```sh
# Configure AWS context by answering questions.
docker context create ecs tearoom6
# ? Create a Docker context using: An existing AWS profile
# ? Select AWS Profile default
# Successfully created ecs context "tearoom6"

# Switch context to use AWS resources.
# `--context tearoom6` flag also can be used.
docker context use tearoom6

# Generate and review CFn template.
docker compose convert

# Deploy to AWS env.
docker compose up

# Check created LoadBalancer's DNS name to find endpoint.

# Delete all resources.
docker compose down

# Reset context to default!!
docker context use default
```

AWS Copilot の場合だと、 Dockerfile とソースコードを用意した上で、以下のようにして、とりあえずデプロイ可能です。
こちらも、デプロイの裏側では CloudFormation が動いています。

```sh
# Install copilot-cli.
brew install aws/tap/copilot-cli

# Create project and deploy by answering questions.
copilot init
# Application name: fish-quiz
# Workload type: Load Balanced Web Service
# Service name: front-end
# Dockerfile: ./Dockerfile
# Ok great, we'll set up a Load Balanced Web Service named front-end in application fish-quiz listening on port 80.
# ...
# All right, you're all set for local development.
# Deploy: Yes
# ...
# ✔ Deployed service front-end.
# Recommended follow-up action:
#   - You can access your service at http://fish-Publi-1QFXMQVITL58L-571977271.ap-northeast-1.elb.amazonaws.com over the internet.

# Delete all resources.
copilot app delete
```

> References

- [Compose V2 | Docker Documentation](https://docs.docker.com/compose/cli-command/)
- [Compose command compatibility with docker-compose | Docker Documentation](https://docs.docker.com/compose/cli-command-compatibility/)
- [docker compose | Docker Documentation](https://docs.docker.com/engine/reference/commandline/compose/)
- [Enabling GPU access with Compose | Docker Documentation](https://docs.docker.com/compose/gpu-support/)
- [Using profiles with Compose | Docker Documentation](https://docs.docker.com/compose/profiles/)
- [Docker Desktop for Apple silicon | Docker Documentation](https://docs.docker.com/desktop/mac/apple-silicon/)
- [Overview of docker-compose CLI | Docker Documentation](https://docs.docker.com/compose/reference/)
- [Docker Desktop for Mac release notes | Docker Documentation](https://docs.docker.com/desktop/mac/release-notes/)
- [Docker Desktop for Mac 3.x release notes | Docker Documentation](https://docs.docker.com/desktop/mac/release-notes/3.x/)
- [Docker Compose release notes | Docker Documentation](https://docs.docker.com/compose/release-notes/)
- [[Docker Compose] V2 GA · Issue #256 · docker/roadmap](https://github.com/docker/roadmap/issues/256)
- [[Docker Compose] V1 End of Life Policy · Issue #257 · docker/roadmap](https://github.com/docker/roadmap/issues/257)
- [Let "docker compose" start "docker-compose" and pass all additional arguments. · Issue #2074 · docker/cli](https://github.com/docker/cli/issues/2074)
- [DockerCon Live 2021: A Look Back at What’s New - Docker Blog](https://www.docker.com/blog/dockercon-live-2021-looking-back-at-the-new-stuff/)
- [What’s New In Docker Compose v2? – CloudSavvy IT](https://www.cloudsavvyit.com/12144/whats-new-in-docker-compose-v2/)
- [新しい docker compose](https://zenn.dev/skanehira/articles/2021-06-03-new-docker-compose)
- [docker-compose upじゃなくてdocker compose up - Acme::AnaTofuZ->new;](https://anatofuz.hatenablog.com/entry/2021/04/16/135827)
- [docker-compose v2.0について | Takeshi Yonezu](https://tkyonezu.com/docker-kubernetes/docker-compose-v2-0-0%E3%81%AB%E3%81%A4%E3%81%84%E3%81%A6/)
- [【Docker Desktop 4.1.0 (69386)】docker-compose up でエラーが出た場合の対処法 - Qiita](https://qiita.com/wadakatu/items/025f8f5f43e70820223c)
- [Deploying Docker containers on ECS | Docker Documentation](https://docs.docker.com/cloud/ecs-integration/)
- [ECS integration Compose features | Docker Documentation](https://docs.docker.com/cloud/ecs-compose-features/)
- [ECS integration composefile examples | Docker Documentation](https://docs.docker.com/cloud/ecs-compose-examples/)
- [Docker ComposeによるAmazon ECS対応がGAに！コンテナをローカル環境と同じノリでECS環境で起動できるぞ！！ | DevelopersIO](https://dev.classmethod.jp/articles/docker-compose-for-amazon-ecs-now-available/)
- [Deploy applications on Amazon ECS using Docker Compose | Containers](https://aws.amazon.com/jp/blogs/containers/deploy-applications-on-amazon-ecs-using-docker-compose/)
- [DockerとAWSのコラボによりdocker ecsコマンドが爆誕したので使ってみた | DevelopersIO](https://dev.classmethod.jp/articles/docker-ecs/)
- [AWS and Docker collaborate to simplify the developer experience | Containers](https://aws.amazon.com/jp/blogs/containers/aws-and-docker-collaborate-to-simplify-the-developer-experience/)
- [docker-archive/ecs-plugin: See http://github.com/docker/compose-cli](https://github.com/docker-archive/ecs-plugin)
- [ECSのオペレーションを劇的に簡略化するAWS Copilotが発表されました！ | DevelopersIO](https://dev.classmethod.jp/articles/aws-copilot/)
- [AWS Copilot のご紹介 | Amazon Web Services ブログ](https://aws.amazon.com/jp/blogs/news/introducing-aws-copilot/)
- [Introducing AWS Copilot | Containers](https://aws.amazon.com/jp/blogs/containers/introducing-aws-copilot/)
- [aws/copilot-cli: The AWS Copilot CLI is a tool for developers to build, release and operate production ready containerized applications on AWS App Runner, Amazon ECS, and AWS Fargate.](https://github.com/aws/copilot-cli)
- [aws/amazon-ecs-cli: The Amazon ECS CLI enables users to run their applications on ECS/Fargate using the Docker Compose file format, quickly provision resources, push/pull images in ECR, and monitor running applications on ECS/Fargate.](https://github.com/aws/amazon-ecs-cli)
- [第2回 AWS Fargate かんたんデプロイ選手権 #AWSDevDay / The Easiest Deployment Championship 2020 - Find your winner for AWS Fargate! - Speaker Deck](https://speakerdeck.com/toricls/the-easiest-deployment-championship-2020-find-your-winner-for-aws-fargate)
- [DockerでPHPを実行するコンテナを作る | GRAYCODE](https://gray-code.com/blog/php-on-docker/)
- [Docker ComposeでNginxとphpを連携する](https://zukucode.com/2019/06/docker-compose-nginx-php.html)
- [AWS CLIでECRにログインする時はget-loginではなくget-login-passwordを使おう - Qiita](https://qiita.com/hayao_k/items/3e4c822425b7b72e7fd0)
