
Athena 遅ればせながら使ってみました。

Hive とかと似たプロダクトみたいなので、 DB 代わりに使うのはちょっと違うんでしょうけど、ログの検索やその他ビッグデータの解析用途には威力を発揮してくれそう。 (そういう意味だと、今回 JRuby からつないだけど、 Jython で繋いだほうがよいのかもしれない)

Athena は現在のところ Web の管理画面と JDBC しか公式のインタフェースが公開されておらず、プログラムから呼び出すには JDBC を経由するしか無さそうです。 Ruby から使いたい場合、 Ruby から Java を呼び出すか、 JRuby を使うかが簡単に思いつく手段ですが、後者のほうが面白そうなので、そちらを選択しました。 (JRuby 使ったので、がっつりな CRuby のアプリケーションにはどのみち取り込めないでしょう。。きっと公式の Ruby SDK からもじきに使えるようになりそうだけど！)

データは、そこかしこで使われている日本郵政のデータ [住所の郵便番号（ローマ字・zip形式）](http://www.post.japanpost.jp/zipcode/dl/roman-zip.html) を使用させて頂きました。 (そもそも、元々今回のモチベーションが Athena を使うことではなく、別で作っている [ITS健保の施設予約スクリプト](https://github.com/tearoom6/kenpo_api) で、郵便番号から住所の順引きをしたいということから始まっていまして...そちらでは結局使わないつもりですが...)

## JRuby から JDBC で Athena に接続

早速核心部分から。

```ruby
require 'java'
require 'jbundler'

access_key = '' # Himitsu Dayo!
access_secret = '' # Himitsu Dayo!
region = 'us-west-2'
athena_jdbc_url = "jdbc:awsathena://athena.#{region}.amazonaws.com:443"

com.amazonaws.athena.jdbc.AthenaDriver

props = java.util.Properties.new
props.set_property('user', access_key)
props.set_property('password', access_secret)
props.set_property('s3_staging_dir', 's3://my-postal-search-jp/my_postal_search_jp/query_results/')

connection = java.sql.DriverManager.get_connection(athena_jdbc_url, props)
statement = connection.create_statement

query = "SELECT * FROM postalsearchjp.jp_postal_codes LIMIT 1"
rs = statement.execute_query(query)

JpPostalCode = Struct.new(:postal_code, :prefecture, :city, :street, :prefecture_kana, :city_kana, :street_kana)
records = []
while (rs.next) do
  record = JpPostalCode.new
  record.postal_code = rs.get_string('postal_code')
  record.prefecture = rs.get_string('prefecture')
  record.city = rs.get_string('city')
  record.street = rs.get_string('street')
  record.prefecture_kana = rs.get_string('prefecture_kana')
  record.city_kana = rs.get_string('city_kana')
  record.street_kana = rs.get_string('street_kana')
  records << record
end
```

JRuby からの JDBC 使用は [公式ドキュメント](https://github.com/jruby/jruby/wiki/JDBC) 参照。
Athene での JDBC 利用も [公式ドキュメント](http://docs.aws.amazon.com/athena/latest/ug/connect-with-jdbc.html) 参照、なのですが、簡単にまとめると、以下のような点になります。

- Driver には [JDBC 4.1-compatible driver](https://s3.amazonaws.com/athena-downloads/drivers/AthenaJDBC41-1.0.0.jar) を用いる (`com.amazonaws.athena.jdbc.AthenaDriver`)
- JDBC URL として、 `jdbc:awsathena://athena.REGION.amazonaws.com:443` を用いる
- Driver の options として `s3_staging_dir` に検索結果や履歴等を保存するための S3 Bucket の Path を指定する
- 認証系のクラス (`com.amazonaws.auth.*`) が使えるように `aws-java-sdk-core` モジュールをロードしておく
- 認証用に user には AWS の aws_access_key_id を、 password には aws_secret_access_key を指定する

## JBundler

AthenaDriver と AWS Java SDK の取り込みに直接 jar ファイルを配置してもいいのですが、今回は [JBundler](https://github.com/mkristian/jbundler) を用いて取り込みました。
AthenaDriver の方は Maven Central Repository にはなかったので、 Atlassian の Repository から取得しています。

以下のように Jarfile を書きます。

```ruby:Jarfile
source 'http://central.maven.org/maven2/'
source 'https://maven.atlassian.com/3rdparty/'

jar 'com.amazonaws.athena.jdbc:AthenaJDBC41', '1.0.0-atlassian-hosted'
jar 'com.amazonaws:aws-java-sdk-core', '~> 1.11'
```

これでインストール。 (Jarfile.lock が作られます)

```sh
# Install
$ jruby -S jbundle install
```

`.jbundler/classpath.rb ` も作られており、以下の require だけでインストールした全ての jar を取り込めます。

```ruby
require 'jbundler'
```

## できたもの

できたものはこちらにおいてあります。

[PostalSearchJp](https://github.com/tearoom6/postal_search_jp)

README にも書きましたが、使い方はいたって簡単。 Web console は一切使いません。(もしかしたら、使い始めの Get Started 押すとことチュートリアルの完了だけはやらないといけないかも)

環境を Java 8 / JRuby 9 にして、

```sh
$ git clone https://github.com/tearoom6/postal_search_jp
$ bundle install
$ jruby -S jbundle install
```

```ruby
PostalSearchJp.configure(
  aws_access_key_id:     '', # Himitsu Dayo!
  aws_secret_access_key: '', # Himitsu Dayo!
  aws_region:            'us-east-1',
  s3_bucket:             'my-postal-search-jp', # 全世界でユニークなものに変えてね
)

# これで、日本郵政のファイルのダウンロード・文字コード等の変換・ S3 へのアップロード・ Athena でのスキーマ定義、全部やります
PostalSearchJp.setup

# 順引き・逆引きに対応
PostalSearchJp.search_by_address('井の頭')
# => [#<struct PostalSearchJp::JpPostalCode postal_code="\"1810001\"", prefecture="\"東京都\"", city="\"三鷹市\"", street="\"井の頭\"", prefecture_kana="\"TOKYO TO\"", city_kana="\"MITAKA SHI\"", street_kana="\"INOKASHIRA\"">]
PostalSearchJp.find_by_postal_code('1810001')
# => #<struct PostalSearchJp::JpPostalCode postal_code="1810001", prefecture="東京都", city="三鷹市", street="井の頭", prefecture_kana="TOKYO TO", city_kana="MITAKA SHI", street_kana="INOKASHIRA">
```

PreparedStatement 使ってないなど、セキュリティ的なことは何も考えていないので、インタフェースを外部使用できる状態にするのは注意して下さい。
