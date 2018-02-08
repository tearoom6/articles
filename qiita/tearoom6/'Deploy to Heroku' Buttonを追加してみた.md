
## 'Deploy to Heroku' Button とは

<https://devcenter.heroku.com/articles/heroku-button>

- GitHub リポジトリの README ファイルに設置するボタンで、これを押すだけで Heroku のサーバに該当リポジトリのコードをデプロイできます。

![Deploy to Heroku](https://www.herokucdn.com/deploy/button.svg)

- 仕組み的には、デフォルトでは、リンク先 (つまり Heroku 側) がHTTPヘッダの `referer` を参照 (そこにはつまり GitHub リポジトリのURLが入ってるんですが)、そこにある app.json を見に行ってアプリのメタ情報を確認、ユーザに必要な情報を入力してもらい、Gitコードを展開という流れです。

### app.json

<https://devcenter.heroku.com/articles/app-json-schema>

- `app.json` には、アプリのメタ情報を記載します。
- 特に以下の2つは重要です。
   - `env` : 環境変数。ここで書いておくと、アプリ作成時に入力欄が出てくる
      - `value` に値を設定しておくと、フォームにデフォルト値として表示される
   - `addons` : アドオン。ここで書いておくと、デプロイ時に自動でアドオンが追加される

```JSON:app.json
{
  "name": "アプリ名",
  "description": "アプリの説明",
  "keywords": [
    "XXX",
    "YYY",
    "ZZZ"
  ],
  "repository": "https://github.com/xxx/xxx",
  "logo": "https://raw.githubusercontent.com/xxx/xxx/master/logo.png",
  "env": {
    "HUBOT_HEROKU_KEEPALIVE_INTERVAL": {
      "description": "hubot-heroku-keepalive interval.",
      "value": ""
    },
    "HUBOT_HEROKU_KEEPALIVE_URL": {
      "description": "hubot-heroku-keepalive URL.",
      "value": ""
    }
  },
  "addons": [
    "heroku-redis:hobby-dev",
    "scheduler:standard"
  ]
}
```

### README

- README への追加は簡単！下記のようなコードを追加するだけです。

```Markdown:README.md
[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy)
```

### template パラメータ

- 例えば、private な GitHub repository の場合、`referer` 情報は渡されず、上記の仕組みが使えない・・・
- そんな場合は、以下のようにボタンのリンク先に `template` パラメータを渡し、明示的にリポジトリのURLを指定してあげます。
- なお、`template` パラメータ以外にも、上述の環境変数のデフォルト値などもパラメータとして渡してあげることができるみたいです。環境別のデプロイボタンを作ってあげるとかができそうですね。

```Markdown:README.md
[![Deploy](https://www.herokucdn.com/deploy/button.svg)](https://heroku.com/deploy?template=https://github.com/xxx/xxx)
```

### ボタン画像

- ボタン画像は、なんだっていい (テキストリンクであってもいい) はずですが、Heroku がオフィシャルに使える画像を用意してくれてます
- 以下のいずれかを使いましょう
   - https://www.herokucdn.com/deploy/button.png
   - https://www.herokucdn.com/deploy/button.svg


## 所感

- Heroku へのデプロイって癖があってちょっと面倒ですが、このボタンさえあれば簡単にできて、便利ですね！！


