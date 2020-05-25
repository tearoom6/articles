僕は Web でブラウジングしていて、後で読もうと思った技術系の記事などを、 Slack の my channel にとりあえず貼り付けておく癖があるんですが、
全部消化できるはずもなく (むしろ 5 割くらいしか消化できない...)、残りのメッセージが未読のまま溜まっていく状態になってしまっていました。

この中で、まだ読みたい記事と、そうでもない記事があるはずなので、棚卸しのため、いったん全件取得してみることにしました。

## Slack API 概要

Slack の Web API は "HTTP RPC-style methods" で、操作対象となるリソース名と、操作方法を組み合わせて指定する形になっています。

基本的に全てのリクエストは `https://slack.com/api/METHOD_FAMILY.method` という URL に投げる形です。

認証は `Authorization` request header に bearer token をくっつける形。 (もしくは非推奨だけど `token` parameter をくっつける形)
token は "Slack app" を登録した上で OAuth 2.0 で取得します。

2020-05-05 より legacy test token と呼ばれる、いわゆる personal token を発行することはできなくなりました。

Slack の API 一覧は https://api.slack.com/methods で確認できます。

ざっと眺めていて、ぱっと見、僕らから見える名称と、 API のリソース名 (おそらく内部的な名前も一緒でしょう) がだいぶ違っているなという気付き。
chat, groups, conversations などの見慣れない言葉が並んでいて、若干とっつきにくいし、一部は既に deprecated になっています。

## token 取得

"Slack app" は https://api.slack.com/apps で作成できます。

app 作成後、その app メニューの "OAuth & Permissions" のページ (`https://api.slack.com/apps/<APP_ID>/oauth`) に行き、 token を取得します。

App が取得できる token には、 user token と bot token の 2 タイプがあります。
それぞれの token では、それを使ってできる操作の範囲 (scope) が異なっています。それぞれの token でできる操作については、上述の API 一覧から遷移できる API 詳細画面に記載があります。

token 取得手順は、以下のとおりです。

1. まず "Scopes" のところで、必要とする scope を登録します
   - 今回必要となるのは User Token Scopes の以下の scope です
      - `channels:read`
      - `channels:history`
      - `groups:read`
      - `groups:history`
      - `im:read`
      - `im:history`
      - `mpim:read`
      - `mpim:history`
      - `chat:write`

   ![slack_app_token_scopes.png](https://files.tearoom6.biz/ca1274c3-5a29-4ee8-8aa3-11270c2ae2c5.png)

2. "OAuth Tokens & Redirect URLs" のところで "Install App to Workspace" を押します

   ![slack_app_install_app.png](https://files.tearoom6.biz/51d820ab-a9f5-4384-96af-bf4e06bcc52b.png)

3. OAuth の同意画面が出てくるので "Allow" を押します (app 作成時に workspace を選んだはずなので、その workspace が出てきます)

   ![slack_app_install_app_consent_screen.png](https://files.tearoom6.biz/e922bf85-ed27-4813-bf6a-75e84bb95370.png)

4. そしたら、元の画面に戻ってきて、 token が取得可能になっています

   ![slack_app_oauth_tokens.png](https://files.tearoom6.biz/ed652bf4-d292-42d6-bfb8-a958b11eb660.png)

取得した token で、 API をコールできるか、テストしてみます。

```sh
BASE_URL=https://slack.com/api/
USER_TOKEN=xoxp-14606570289-14606808582-1142566515971-a38b3a116996df86ec3c289c3fec4a46
BOT_TOKEN=xoxb-14606570289-1166177545408-1gq5QZjTJ33024CGpmOgTsvn

curl -X POST -H "Authorization: Bearer $USER_TOKEN" "$BASE_URL/auth.test"
#{"ok":true,"url":"https:\/\/tearoom6.slack.com\/","team":"tearoom6","user":"tomohiro","team_id":"T0XXXXXXX","user_id":"U0XXXXXXX"}

curl -X POST -H "Authorization: Bearer $BOT_TOKEN" "$BASE_URL/auth.test"
#{"ok":true,"url":"https:\/\/tearoom6.slack.com\/","team":"tearoom6","user":"tearoom6private","team_id":"T0XXXXXXX","user_id":"U0XXXXXXXXX","bot_id":"B0XXXXXXXXX"}%
```

`Bearer` が小文字だと失敗するみたいなので、ご注意を。

## channel のメッセージ一覧取得

Ruby 2.7.1 で実行。

まず channel 一覧を取得します。
いわゆる my channel (ユーザの個人チャンネル) の情報を得るには bot token ではなく、 user token を利用する必要があります。
(my channel には bot は参加権限がないため、 `not_in_channel` のようなエラーが出る)

```ruby
require 'json'
require 'net/http'

BASE_URL = 'https://slack.com/api/'
USER_TOKEN = 'xoxp-14606570289-14606808582-1142566515971-a38b3a116996df86ec3c289c3fec4a46'

# Get channel list.
uri = URI("#{BASE_URL}/conversations.list")
params = {
  types: [:public_channel, :private_channel, :mpim, :im].join(','),
}
uri.query = URI.encode_www_form(params)
req = Net::HTTP::Get.new(uri)
req[:Authorization] = "Bearer #{USER_TOKEN}"

res = Net::HTTP.start(uri.host, uri.port, use_ssl: true) do |http|
  http.request(req)
end
channels = JSON.parse(res.body)['channels']
```

my channel は `channel['name']` が nil で、 `channel['user']` がその個人の user id の channel として取得できます。
`channel['id']` をメモっておきます。 (今回の場合は `D1KECECUE`)

もしくは Slack の画面上で channel の "Copy link" を行って、リンクを取得することでも channel id を知ることができます。

![slack_channel_copy_link.png](https://files.tearoom6.biz/9b465ed2-02d1-4dd3-93ec-cea376ae7529.png)

続いて、その channel のメッセージ一覧を取得します。

```ruby
require 'json'
require 'net/http'

BASE_URL = 'https://slack.com/api/'
USER_TOKEN = 'xoxp-14606570289-14606808582-1142566515971-a38b3a116996df86ec3c289c3fec4a46'
CHANNEL = 'D1KECECUE'

# Get conversation history.
cursor = nil
while true do
  uri = URI("#{BASE_URL}/conversations.history")
  params = {
    channel: CHANNEL,
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
    next unless message['text'].start_with?('http') or message['text'].start_with?('<http')
    link_url = message['text'].start_with?('<http') ? message['text'].match(/\<([^\|]+)\|?.*\>/)[1] : message['text']
    link_title = message['attachments']&.first ? message['attachments'].first['title'] : link_url
    puts "- [#{link_title}](#{link_url})"
  end

  break unless res_body['has_more']
  cursor = res_body['response_metadata']['next_cursor']
end
```

最後に、抽出したメッセージはもう不要なので、削除します。

```ruby
require 'json'
require 'net/http'

BASE_URL = 'https://slack.com/api/'
USER_TOKEN = 'xoxp-14606570289-14606808582-1142566515971-a38b3a116996df86ec3c289c3fec4a46'
CHANNEL = 'D1KECECUE'

# Get conversation history.
cursor = nil
while true do
  uri = URI("#{BASE_URL}/conversations.history")
  params = {
    channel: CHANNEL,
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
    next unless message['text'].start_with?('http') or message['text'].start_with?('<http')

    # Remove message.
    uri = URI("#{BASE_URL}/chat.delete")
    params = {
      channel: CHANNEL,
      ts: message['ts'],
    }
    req = Net::HTTP::Post.new(uri)
    req.set_form_data(params)
    req[:Authorization] = "Bearer #{USER_TOKEN}"
    req['Content-Type'] = 'application/x-www-form-urlencoded'

    res = Net::HTTP.start(uri.host, uri.port, use_ssl: true) do |http|
      http.request(req)
    end

    sleep 1
  end

  break unless res_body['has_more']
  cursor = res_body['response_metadata']['next_cursor']
end
```

なお、スレッドの内容を取得する場合は、また別のことをしないといけないので、ご注意ください。

> References

- [Using the Slack Web API | Slack](https://api.slack.com/web)
- [API Methods | Slack](https://api.slack.com/methods)
- [Retrieving messages | Slack](https://api.slack.com/messaging/retrieving)
- [Installation & permissions | Slack](https://api.slack.com/authentication)
- [The creation of legacy test tokens is deprecated | Slack](https://api.slack.com/changelog/2020-02-legacy-test-token-creation-to-retire)
- [Legacy tokens | Slack](https://api.slack.com/legacy/custom-integrations/legacy-tokens)
- [Slack Web API に入門してみよう｜sumi｜note](https://note.com/kawa1228/n/nb0811ed44792)
- [Slack API で特定チャンネルの一定期間以上前のメッセージを削除する - michimani.net](https://michimani.net/post/programming-delete-old-slack-messages/)
- [slack api - stack api conversations.history returns error: not_in_channel - Stack Overflow](https://stackoverflow.com/questions/60198159/)
- [String#match (Ruby 2.7.0 リファレンスマニュアル)](https://docs.ruby-lang.org/ja/latest/method/String/i/match.html)
