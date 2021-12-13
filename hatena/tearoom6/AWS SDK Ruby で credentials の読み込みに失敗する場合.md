この記事は [🌊 UMITRON 🐟 Advent Calendar 2021](https://adventar.org/calendars/6652) **13日目** の記事です。
Advent Calendar も折り返し地点です 🏃

---

お魚食べてますか? 🐟

現在は UMITRON 株式会社で、水産養殖 🐟🐟🐟 の持つ潜在的可能性を IT を使って引き出すことに取り組んでいます。

当初予定していた内容がちょっと間に合わなかったので、以前遭遇したエラーについての小ネタを書きたいと思います。

---

AWS SDK for Ruby を環境変数 `AWS_CONFIG_FILE`, `AWS_SHARED_CREDENTIALS_FILE` を指定しつつ、使おうとしたときに `Aws::Errors::MissingCredentialsError` のようなエラーが起きました。これらの環境変数で指定している設定ファイルは確実に存在している状態で、 AWS CLI などは正常に使えている状態だったので、原因を突き止めるのに難儀しました。

結論から言えば、これらの設定ファイルの内容にインデントが含まれていたのが原因でした。

設定ファイル中では、名前付きプロファイルを用いており、[名前付きプロファイル - AWS Command Line Interface](https://docs.aws.amazon.com/ja_jp/cli/latest/userguide/cli-configure-profiles.html) に書かれている例を使わせてもらうと、もともと私は以下のように記載していました。

```ini
[default]
    region=us-west-2
    output=json

[profile user1]
    region=us-east-1
    output=text
```

AWS の config, credentials 設定ファイルでは、 INI ファイル形式が使われていますが、名前付きプロファイルの数が多い場合には、上のようにインデントを付けることで、視認性はよくなるため、そのようにしていました。 INI ファイルの公式仕様のような存在を私は知らないのですが、なんとなく慣習的にインデントを付けるのは OK なのかなという理解でいました。実際、 AWS CLI や Boto3 (AWS SDK for Python) では、インデントが含まれた状態でも読み込んでくれていました。

しかし、下のファイルが AWS SDK for Ruby で INI ファイルを読み込む実装になっているのですが、正規表現の部分がインデントに対応していないことがわかります。

https://github.com/aws/aws-sdk-ruby/blob/a8332558e8c8a4889dfce17dd54742172500dffb/gems/aws-sdk-core/lib/aws-sdk-core/ini_parser.rb

改めて INI ファイルについて説明しているネット上の情報を見てみると、確かに、敢えてインデントに言及しているものはあまりなさそうです。
(Wikipedia は `Leading and trailing whitespaces around the outside of the property name are ignored.` と書いてあるので、それだとインデントも許容されそうではあるんですが)

- [INI file - Wikipedia](https://en.wikipedia.org/wiki/INI_file)
- [Format of the .ini File | Microsoft Docs](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/ms717987(v=vs.85))
- [INI - Initiation File Format](https://docs.fileformat.com/system/ini/)

ともかくも AWS SDK for Ruby は対応していないので、それに従う他ありません。以下のように修正することでエラーは解消しました。

```ini
[default]
region=us-west-2
output=json

[profile user1]
region=us-east-1
output=text
```

INI の仕様が決まっていて、インデントを許容すると明示してあるなら、修正の Pull Request を送るのですが、これはむしろこちらの指定ミスの可能性があるので、微妙なところですね。。
とはいえ、踏んでしまうと原因がわかりにくいので、嫌なエラーでした。
