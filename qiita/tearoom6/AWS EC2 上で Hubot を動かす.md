
以前ご紹介した [簡単!QiitaでストックされたのをSlackに通知するようにする](http://qiita.com/tearoom6/items/1028b05ce049e6fb8e24) は、Hubot を Heroku 上で動かしたものでしたが、今回はこれを AWS EC2 上で動かせるようにしてみました。
もともと EC2 サーバを動かしている人は、こちらの方がいいかもしれません！

## 前提

- EC2 インスタンスを起動して、ssh 接続するところまで実施済みとします。

## 手順

- 環境変数を簡単に設定できるように、[direnv](https://github.com/direnv/direnv) をインストールします

```Bash
$ git clone https://github.com/direnv/direnv
$ cd direnv
$ sudo make install
$ eval "$(direnv hook bash)"
```

- [nvm](https://github.com/creationix/nvm) を使って、Node.js をインストールします

```Bash
$ git clone https://github.com/creationix/nvm.git ~/.nvm && cd ~/.nvm && git checkout `git describe --abbrev=0 --tags`
$ . ~/.nvm/nvm.sh
$ nvm install v5.6.0
$ nvm use v5.6.0
```

- Redis をインストールして、バックグラウンドで起動します

```Bash
$ wget http://download.redis.io/releases/redis-3.0.7.tar.gz
$ tar xzf redis-3.0.7.tar.gz
$ rm redis-3.0.7.tar.gz
$ cd redis-3.0.7
$ make
$ sudo make install
$ redis-server &
```

- [Qiitan](https://github.com/tearoom6/qiitan) のソースコードをダウンロードします
- 以下の2つのファイルを作成して、秘密の設定値を記載します
   - .hubot_slack_token : SlackのHubot連携のトークンを記載
   - .qiita_access_token : Qiitaのパーソナルアクセストークンを記載
- 詳しくは、[以前の記事](http://qiita.com/tearoom6/items/1028b05ce049e6fb8e24)を参照してください

```Bash
$ git clone https://github.com/tearoom6/qiitan.git
$ cd qiitan/
$ vi .hubot_slack_token
$ vi .qiita_access_token
```

- Hubot を起動します
- Node のプロセスをデーモンとして動かすには、[forever](https://github.com/foreverjs/forever) というツールを使うと便利らしいので、それをインストールして以下のように起動します

```Bash
$ npm install forever -g
$ forever start -c bash bin/hubot -a slack
```

- プロセスがちゃんと起動していること、ログが出ていることを確認します

```Bash
$ forever list
$ forever logs 0
```

以上です。
