Rails での秘密情報の管理機能がバージョンごとに大きく違っている件について、まとめておきます。

### Rails 4.1

- `config/secrets.yml`

秘密情報を格納する設定ファイルとして `config/secrets.yml` が追加されました。

このファイルの設定値は `Rails.application.secrets` で取得できます。

この設定ファイルの設定値は、同一ファイル内で、環境別に分けられており、本番環境での設定値は VCS へコミットされても問題ないよう、通常、環境変数を参照するようにします。

```yaml
development:
  secret_key_base: a4f6336a67817352a26e098403561530cdecdea6f9031deac218a8369b13fb39ba5e9663d59a3a379d88b5609d56e83457dc6b04ef11ce90d7e3cbcd98814199

test:
  secret_key_base: 7c0558816ee4552593e89a69184771b4b254bfc64af4fcd3e0299b2877c98ac67ba625b305fb9886c0bd8b50e9881423eebfcc4180bffd5c42f05503f779a7c2

production:
  secret_key_base: <%= ENV["SECRET_KEY_BASE"] %>
```

デフォルトでは、上記の通り `secret_key_base` が格納されています。この設定値の役割については、以下の参照リンクを参考にしてください。

> References

- [Ruby on Rails 4.1 Release Notes — Ruby on Rails Guides](https://guides.rubyonrails.org/4_1_release_notes.html)
- [Understanding the secret_key_base in Ruby on Rails - Michael J Coyne - Medium](https://medium.com/@michaeljcoyne/understanding-the-secret-key-base-in-ruby-on-rails-ce2f6f9968a1)


### Rails 5.1

- Encrypted secrets ([Pull Request #28038](https://github.com/rails/rails/pull/28038))

秘密情報を、暗号化した状態で保存するファイルとして、新たに `config/secrets.yml.enc` が導入されました。
このファイルに暗号化された状態で保存されている設定値は、環境変数 `ENV["RAILS_MASTER_KEY"]` もしくは `config/secrets.yml.key` file に書かれた暗号化鍵によって、復号されて、 `Rails.application.secrets` で取得できます。

この機能を有効にするには、以下の設定が必要になります。 (環境別に切り分け可能)
この設定値を `false` にすると、従来どおり `config/secrets.yml` からの読み込みが行われます。

```ruby
config.read_encrypted_secrets = true
```

以下のコマンドを実行すれば、 `config/secrets.yml.enc` と `config/secrets.yml.key` が生成されます。

```sh
rails secrets:setup
```

`config/secrets.yml.enc` を、暗号化されていない状態で編集したい場合、以下を実行します。

```sh
rails secrets:edit
```

エディタを指定したい場合、以下のようにします。

```sh
EDITOR=vi rails secrets:edit
```

明示的に暗号化鍵を指定したい場合、以下のようにします。

```sh
RAILS_MASTER_KEY=1015d9018711afd106dbf752c15bf5b0 EDITOR=vi rails secrets:edit
```

> References

- [Ruby on Rails 5.1 Release Notes — Ruby on Rails Guides](https://guides.rubyonrails.org/5_1_release_notes.html)
- [Rails 5.1 Encrypted secretsの使い方 - chulip.org](https://chulip.org/entry/2017/05/16/191321)


### Rails 5.2

- Credentials ([Pull Request #30067](https://github.com/rails/rails/pull/30067))

秘密情報格納用の設定ファイルとして、新たに `config/credentials.yml.enc` が作られます。
このファイルに暗号化された状態で保存されている設定値は、環境変数 `ENV["RAILS_MASTER_KEY"]` もしくは `config/master.key` file に書かれた暗号化鍵によって、復号されて、 `Rails.application.credentials` で取得できます。

この秘密情報の設定ファイルを、暗号化されていない状態で編集したい場合、以下を実行します。

```sh
rails credentials:edit
```

エディタを指定したい場合、以下のようにします。

```sh
EDITOR=vim rails credentials:edit
```

以下の設定を有効にすることで、いずれかの暗号化鍵の存在チェックを確実に行うことができます。 (主に本番環境を想定)

```ruby
config.require_master_key = true
```

なお、引き続き Encrypted secrets も使えます。

> References

- [Ruby on Rails 5.2 Release Notes — Ruby on Rails Guides](https://guides.rubyonrails.org/5_2_release_notes.html)
- [Rails5.2からsecrets.yml*が廃止されcredentials.yml.encに統合されるよ - Qiita](https://qiita.com/daichirata/items/da40e205d273ae69fcfc)


### Rails 6.0

- Add support for multi environment credentials. ([Pull Request #33521](https://github.com/rails/rails/pull/33521))

環境別の credentials file (`config/credentials/<environment>.yml.enc`) が存在する場合、その環境では、そちらのファイルから設定値が読み取られるようになります。
その環境では `config/credentials.yml.enc` が用いられないことに注意が必要です。

環境別の credentials file の設定値は、環境変数 `ENV["RAILS_MASTER_KEY"]` もしくは `config/credentials/<environment>.key` file に書かれた暗号化鍵によって、暗号化・復号されます。

復号された設定値は、引き続き `Rails.application.credentials` で取得できます。

環境別の credentials file は、以下のように `--environment` option を付けることで、暗号化されていない状態で編集できます。

```sh
rails credentials:edit --environment staging
```

なお、環境別の設定ファイルと暗号化鍵ファイルのパスは、設定値 `config.credentials.content_path` と `config.credentials.key_path` によって変更できます。

> References

- [Ruby on Rails 6.0 Release Notes — Ruby on Rails Guides](https://edgeguides.rubyonrails.org/6_0_release_notes.html)
- [Rails 6 adds support for Multi Environment credentials | BigBinary Blog](https://blog.bigbinary.com/2019/07/03/rails-6-adds-support-for-multi-environment-credentials.html)
