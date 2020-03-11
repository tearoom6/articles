ソースコード変更の Docker コンテナへの反映について、ちょうどいい機会があったので基礎も含めて整理します。
(タイミングよく投稿されていた) 以下の Qiita の記事に書かれていることが全てだけど、補足してまとめておきます。

> References

- [Docker Composeにおける各種ファイルの変更時の反映 - Qiita](https://qiita.com/subretu/items/5857628534b53f29f5a3)
- [Docker overview | Docker Documentation](https://docs.docker.com/engine/docker-overview/)
- [Docker Image VS Container: What is the difference?](https://phoenixnap.com/kb/docker-image-vs-container)
- [docker-compose stop | Docker Documentation](https://docs.docker.com/compose/reference/stop/)
- [docker-compose down | Docker Documentation](https://docs.docker.com/compose/reference/down/)
- [docker container stop | Docker Documentation](https://docs.docker.com/engine/reference/commandline/container_stop/)
- [docker container rm | Docker Documentation](https://docs.docker.com/engine/reference/commandline/container_rm/)
- [How to restart a single container with docker-compose - Stack Overflow](https://stackoverflow.com/questions/31466428/)

## image と container

抽象化された概念の理解は、以下のような感じでいいと思っています。

- image: 実行環境の構成を保存した情報
- container: image を実際に動かした実行環境

その上で、以下の点を理解できていれば、今回の話的には大丈夫です。

- container を動かすには、ベースとなる image が必ず必要なこと
- container を破棄しても image が残っている場合があること
- リポジトリ内のソースコードは、 image 内に含まれている場合と container 上に含まれている場合があること

なお image は実際には layer に分かれて保存されますので、そこが分かっていると image size の削減や差分 build の高速化を図ったりできます。

## image の状態管理

image は実際に動いているわけではないので、状態としては存在するか否かの2択です。
そのため、操作としては、簡単に言えば、作成する、破棄するの2つとなります。

デフォルトのオプション設定で、それぞれの操作を行うコマンドは以下のとおりです。

作成する:

- `docker image build`
- `docker-compose build`

破棄する:

- `docker image rm`
- `docker-compose rm`

## container の状態管理

container は実際に動いているので、状態としては、存在しない <=> 存在する <=> 動いている、の3択です。
よって、それぞれの状態遷移を起こすための操作が行えます。

デフォルトのオプション設定で、それぞれの操作を行うコマンドは以下のとおりです。

作成する:

- `docker container create`
- `docker-compose create` (Deprecated)

起動する:

- `docker container start`
- `docker-compose start`

停止する:

- `docker container stop`
- `docker-compose stop`

破棄する:

- `docker container rm`
- `docker-compose rm`

作成・起動を一気にやる:

- `docker-compose up`

停止・破棄を一気にやる:

- `docker-compose down`

## host と Docker container の関係

host というのは、 Docker container を動かす実行環境のことですね。
Docker for Mac を使っている場合なら macOS のことです。

Docker container は host とは切り離された状態で (無関係に) 動きますので、一度 container を立ち上げてしまえば、 host 上のファイルの変更は container 上には反映されません。

ただし、 host 上のファイルシステムの一部を volume として、 Docker container に mount することができるので、その場合は、 host 上のファイルの変更が container にそのまま反映されます。

## Case Study

### Dockerfile を変更した場合

Dockerfile 内の記載内容は、 image を作るためのものなので、 Dockerfile を書き換えた場合は **image から作り直す必要があります**。
もちろん、その image を使って動いている **container も作り直す必要があります**。

### docker-compose.yaml を変更した場合

docker-compose.yaml の記載内容のうち `build` section 以外は、出来上がった image を使って、その設定内容で container, volume, network を作成するためのものなので、これを書き換えた場合は **image を作り直す必要はありません**。ただ、 container は作成時にこの設定ファイルの内容を取り込みますので、 `docker-compose restart` (`stop` -> `start`) だけではダメで、 **container の再作成を行う必要があります**。

また、 docker-compose.yaml の中の `build` section の設定内容を書き換えた場合は、 **image の再作成が必要になります**。

### その他のファイルを変更した場合

これは本当にケースバイケースなのですが、大きく分けて以下のようなケースが考えられます。

#### そのファイルが image に含まれている場合

Dockerfile 内で `ADD` や `COPY` によって、対象ファイルを image に取り込んでいる場合は、**基本的には image (および container) の再作成が必要になります**。

ただし、そのファイルの変更が動的に動作に反映される場合、 container 上でそのファイルを書き換える (host の volume を mount している場合は host 上で書き換えてもいい) ことで、一時的に、その container での動作に反映することも可能です。その場合、 host 上で永続化していない container に対する変更内容は、 container を再作成することで元に戻ってしまうことに注意が必要です。

例としては、例えば Gemfile は普通 image にも含めると思いますが、 container 動作中に Gemfile を書き換えて `bundle install` することで、新たな gem を container にインストールすることができます。しかし、この container を再作成すると、このインストールした状態もリセットされてしまうため、再度 Gemfile の書き換え (host の volume を mount していない場合) や `bundle install` が必要になるわけです。

さらに、その gem のインストールに、 image のビルド中にだけ使って、最終 image には含めていない native library が必要だったりすると、そのインストールも必要だったりして、結局は image の再作成からやり直したほうが効率的だったりします。

#### そのファイルが image に含まれていない場合

image に含めていないファイルを container が利用するということは、基本的には host などから volume を mount したり、 network 経由等で取得したりという構成になっているかと思います。この場合は、変更反映のための **image の再作成は不要です**。

container については、そのファイル内容の変更が動的に動作に反映されるかどうか (live reload が有効かどうか) によって、 container の再起動や、設定反映のためのコマンド発行が必要かどうかが変わります。この部分は、アプリケーションの実装次第になるため、随時調べるか、とりあえず restart or 再作成 してみる、というのがよいかと思われます。
