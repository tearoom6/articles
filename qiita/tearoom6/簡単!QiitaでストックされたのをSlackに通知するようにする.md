
> **【修正】**
> すみません、Redisの接続数が増え続けるバグがあったため、修正いたしました。
> お手数ですが、再度デプロイをやり直していただけると幸いですm(__)m


Qiitaに投稿を初めて1週間。ストック数が増えるたびにちょっと嬉しくなるもんですね。ストック数が更新のモチベーションになるというか。

そんなわけで、Qiitaにストックされた際にSlackに通知されるようにしてみました。


## 対象読者

- 1つ1つのストックに喜びを感じる私みたいなQiitaビギナー
   - 1つ1つ通知するようにしているので、たくさんストックされる人気Qiiterさんには向いてないです
- とりあえず簡単にやってみたい人


## まずは手っ取り早く通知できるようにしよう!

- ['Deploy to Heroku' Buttonを追加してみた](http://qiita.com/tearoom6/items/9174e23d7d061659b177) でご紹介した方法を用います
- Herokuを使うので、ユーザ登録・カード登録は済ませておいてください
- Free帯のアドオンを使ってるうちは料金は発生しないんですが、カード登録しないと、アドオン追加できないのです

### 手順

1. Slack の Hubot integration を追加して、APIトークンを入手してください
   https://slack.com/apps/A0F7XDU93-hubot

2. Qiita のパーソナルAPIトークンを追加してください
   https://qiita.com/settings/tokens/new

3. 以下のページの'Deploy to Heroku'ボタンを押してください
   https://github.com/tearoom6/qiitan

4. Heroku Scheduler dashboard でタスクを追加してください
   https://dashboard.heroku.com/apps/{YOUR_APP_NAME}/resources.
   - [Heroku Scheduler] のリンクをクリック
   - 以下のタスクを Daily で追加します
   - 時刻は、1日の開始時刻 (下記の HUBOT_HEROKU_WAKEUP_TIME に該当) にセットしてください
      - ただし、UTCなので、日本時刻で入れないように。0:00AM UTC なら、こちらの 9:00AM です

      ```
      $ curl -I http://{YOUR_APP_NAME}.herokuapp.com/heroku/keepalive
      ```

### Herokuアプリ起動時の設定値

* App Name
   - お好きなものを。すでに取られている名前は使えません。
   - ここで設定した名前を上記手順の`{YOUR_APP_NAME}`に当てはめます
* Runtime Selection
   - Japanは提供されていません。United States でよいかと。
* Config Variables
   * HUBOT_HEROKU_KEEPALIVE_URL
      - `https://{YOUR_APP_NAME}.herokuapp.com/` の `{YOUR_APP_NAME}` を上で設定した App Name に置き換えてください
   * HUBOT_HEROKU_WAKEUP_TIME
      - 夜型の人は遅めの時刻に
   * HUBOT_HEROKU_SLEEP_TIME
      - 夜型の人は遅めの時刻に
      - `SLEEP_TIME - WAKEUP_TIME`が18時間以内になるようにしてください
   * HUBOT_HEROKU_KEEPALIVE_INTERVAL
      - デフォルト値で大丈夫です
   * CRON_TIME
      - ジョブ実行頻度の設定(後述)です
   * QIITA_ACCESS_TOKEN
      - 手順2 で取得したAPIトークン
   * HUBOT_SLACK_TOKEN
      - 手順1 で取得したAPIトークン
   * SLACK_CHANNEL_NAME
      - 通知を流したいSlack channel名を#付きで
   * LANG
      - デフォルト値で大丈夫です
   * TZ
      - デフォルト値で大丈夫です

### Hubotを動かし続けるための対応

- 無料版のHerokuサーバは放っておくと勝手に止まってしまうため、[hubot-heroku-keepalive](https://github.com/hubot-scripts/hubot-heroku-keepalive) を取り込んで動かし続けるようにしています
- また、無料では1日のうちに動かし続けられる時間が18時間までに限られているため、こちらも上記プラグインのページで説明にある通り、Heroku Scheduler で1日の初めにURLを叩いて、サーバを起こしてあげるようにしています


## 実装について

### botアプリ

- [Hubot](https://hubot.github.com/) を使用しています
- cron でデフォルトで15分に一度動かして、新規ストックをチェックしています

### Qiita API v2

- 本当なら [Qiita Webhook](https://qiita.com/api/webhook/docs) を使えると、効率がいいんですが、この機能は Qiita:Team に限られているのと、新規ストックをトリガにするものがないため、今回は [Web API v2](https://qiita.com/api/v2/docs) を用います
- また、APIを使った時に、item(投稿)一覧を取得した時にストック数を取れたらよかったんですが、取れないため、itemごとにストックしたユーザ(stocker)一覧を取り直しています
- これを全投稿に対して15分毎に実行しているので、`1 + 投稿数` 回のAPIアクセスが15分毎に最低でも発生します。投稿数・投稿あたりストック数が100を超えると、APIアクセスが分割されるため、さらにアクセス数が多くなります。
- 以下のような利用制限が設けられているため、投稿数の多いQiiterさんはご注意を！

> ```
> 認証している状態ではユーザごとに1時間に1000回まで、認証していない状態ではIPアドレスごとに1時間に60回までリクエストを受け付けます。
> ```

### カスタマイズ

- 上記の利用制限の問題等があるため、cronのアクセス頻度の調整はしてもいいかと思われます
   - 設定値の`CRON_TIME`を変えてください
   - 通常のcronと違って、[node-cron](https://github.com/ncb000gt/node-cron) は秒単位の調整ができます

### 今後の改善点 (TODO)

- 今回は正攻法で API を使いましたが、ストック数の取得が目的ならスクレイピングでもいいかも
- ある程度溜まってからまとめて通知も可能に
- フォロアー追加の通知、など

