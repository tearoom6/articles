最近、プライベートで使っている Slack の workspace で、以下のようなメッセージが表示されるようになりました。

![slack_free_plan_notification_hidden_messages.png](https://files.tearoom6.biz/34245b7b-359b-48d1-be8f-d75fa603a7de.png)

どうやら、 2022-09-01 から Slack のフリープランの内容が変わって、 **90 日以前に投稿されたメッセージについては非表示になる** ようです。

現在は、メッセージ数 10,000 件、ストレージ容量 5 GB 以内であるならば、投稿日時に関わらず、閲覧可能となっているので、私のようにプライベートで使っている場合だと、ちょっとしたメモアプリ、ウェブクリップのように使用できていたのですが、フリープランではこれができなくなってしまいました。

なお Slack のお知らせ上では、よりポジティブに、 **過去 90 日間のメッセージ履歴とファイルストレージを (メッセージ数、ストレージ容量の制限なく) 無制限に利用できる** ようになると書かれています。まぁ、チームでのコミュニケーションに活用する、という本来の用途をフリープランで体験するためには、あるべき形になるのかなというのは、同じ開発者としてよく分かりますが。

兎にも角にも、私のような使い方をしていた場合には、過去分のメッセージを退避させなくてはいけません。

この発表があったのが 2022-07-18 となっていますから、1ヶ月そこらの猶予しかないわけですが、この手のタスクは「期限までにやらなくてはいけない」類のものなので、後回しにしても何もいいことはありません。少しでも余裕があるうちにやってしまいましょう。

`Slack フリープラン 保存期間` などで検索すれば、ソリューションがたくさん出てくるので、詳しくはそれらを参照、としてもいいのですが、せっかく対処したので、ここでも記録を残しておきます。

----------

いくつかの、レベルの違う対処法が考えられます。

## 1. プロプランに変更する

時間がない人は、お金に物を言わせましょう。
今回の発表で、同時にプロプランの値上げも発表されていますが、新しい価格では、月額料金で 1 ユーザーあたり 1,050 円となっています。 (年間契約ではもうちょっと安くなります)

ただ、プライベートで開発に使うものって、基本無料だから使っている、的なものは多いとは思います。これも同じ開発者として、タダ乗りはあまり薦められないですが、この費用負担に耐えれるかどうかは、個々人の価値観と判断に依るかなとは思います。

なお、上記のフリープランの制限を再掲すると、 **90 日以前に投稿されたメッセージについては非表示になる** であり、削除されるとは書いていないのがポイントです。実際の Slack の FAQ にも `有料プランへアップグレードすれば、いつでもワークスペースのメッセージとファイルの全履歴にアクセスできるようになります。` と書いてあります。

万が一、メッセージの退避を期限内に終えられなくても、一時的にプロプランに変更することで、引き続き過去のメッセージの参照ができるようになる救済策が用意されていると考えられるでしょう。良心的ですね。

## 2. Slack 公式のデータエクスポート機能を用いる

Slack には、公式のデータエクスポート機能が用意されています。

workspace owner の権限で https://{workspace_name}.slack.com/services/export にアクセスすることで、この機能を利用できます。

ただし、少し制限があって、フリープラン、および、プロプランでは、 public channel のデータエクスポートしか行うことができません。つまり private channel と direct message (DM) のデータエクスポートができないことになります。 (ビジネスプランでは、これらのチャンネルのデータエクスポートも可能)

Slack の仕様として、 private channel を public にすることはできない (逆は可能) ので、ちゃんと抜け穴も塞がれています。

これを許容できるなら最も手っ取り早い対処法かなと思います。

なお、データエクスポートを実際に行ってみると、 `{workspace_name} Slack export MMM DD YYYY - MMM DD YYYY.zip` (e.g. `tearoom6 Slack export Nov 16 2015 - Aug 22 2022.zip`) のようなファイル名で zip file がダウンロードでき、それを展開すると、以下のようなディレクトリ構成となっていました。 Slack のデータ構造の一端を覗ける感じがして、興味深いですね。

- `channels.json` (チャンネル一覧: public channel のみ)
- `users.json` (ユーザ一覧)
- `integration_logs.json` (インテグレーションの追加・更新・削除ログ)
- `{channel_name}/` (チャンネル名ごとのディレクトリ)
   - `{YYYY-MM-DD}.json` (日付ごとのメッセージ一覧)
   - `{YYYY-MM-DD}.json`
   - ...
- `{channel_name}/` (チャンネル名ごとのディレクトリ)
   - `{YYYY-MM-DD}.json` (日付ごとのメッセージ一覧)
   - `{YYYY-MM-DD}.json`
   - ...
- `{channel_name}/` (チャンネル名ごとのディレクトリ)
   - `{YYYY-MM-DD}.json` (日付ごとのメッセージ一覧)
   - `{YYYY-MM-DD}.json`
   - ...
- ...

## 3. Slack API でメッセージ取得

やはり private channel や DM のメッセージも保存したい！ ということになると、メッセージ数が多い場合には、一個一個手動でコピーするのは修行になるかと思うので、 Slack API を使うしか無いかなと思います。

[Slack API でメッセージ一覧取得 - My tech diary](https://tearoom6.hateblo.jp/entry/2020/05/25/091422) に記載したやり方を流用してやってみます。 2 年前なので、画面などがちょいちょい変わっていますが、記憶を思い出しながら、 Slack App の作成から token を取得までの流れをやっておきます。

こうして得た user token と、 Slack 上でチャンネルの右クリックメニューから実行できる "Copy link" で取得できる、各チャンネルの URL に含まれている channel ID を用いて、以下のようなスクリプトを作成し、実行することで、とりあえず private channel なども含めたメッセージ一覧を抜き出すことができます。

```ruby
require 'json'
require 'net/http'

BASE_URL = 'https://slack.com/api/'
USER_TOKEN = 'xoxp-...'  # REPLACE IT!!
CHANNEL_ID = 'G0HEFHV61' # REPLACE IT!! You can get it by "Copy link" operation to the channel.

# Get conversation history.
cursor = nil
while true do
  uri = URI("#{BASE_URL}/conversations.history")
  params = {
    channel: CHANNEL_ID,
    cursor: cursor,
    limit: 100,
  }
  uri.query = URI.encode_www_form(params)
  req = Net::HTTP::Get.new(uri)
  req[:Authorization] = "Bearer #{USER_TOKEN}"

  res = Net::HTTP.start(uri.host, uri.port, use_ssl: true) do |http|
    http.request(req)
  end
  res_body = JSON.parse(res.body)

  res_body['messages'].each do |message|
    puts message['text']
    puts '--------------------------------'
    puts ''
  end

  break unless res_body['has_more']
  cursor = res_body['response_metadata']['next_cursor']
end
```

チャンネルを一覧指定できるようにしたり、投稿ユーザ名や投稿日時も出力するようにするなど、改良してみても良さそうですね。

## 4. 世の人が作ってくれているツールを使う

Slack API を使う形式の派生系と言えますが、世の人が使いやすい形で公開してくれているツールもいくつかあるみたいですので、それらを使ってみるのもいいかと思います。

> References

- [Slack 初の料金改定とフリープランの内容変更のお知らせ | Slack](https://slack.com/intl/ja-jp/blog/news/pricing-and-plan-updates)
- [プロプランの料金改定とフリープランの最新情報 | Slack](https://slack.com/intl/ja-jp/help/articles/7050776459923)
- [ワークスペースのデータをエクスポートする | Slack](https://slack.com/intl/ja-jp/help/articles/201658943)
- [無料版Slackメッセージ保存・退避ツール！フリープランの1万→90日の変更への対策法 | AutoWorker〜Google Apps Script(GAS)とSikuliで始める業務改善入門](https://auto-worker.com/blog/?p=6184)
