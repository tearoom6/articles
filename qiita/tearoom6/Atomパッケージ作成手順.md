# Atomパッケージ作成手順

## 作成手順

[Cmd]-[Shift]-[p] でコマンドパレットを開き、 "Package Generator: Generate Package" を選ぶことで作成できます。
こうして作成した開発中のパッケージは、自動で自分の Atom Editor にインストールさせた状態となります。

## 開発手順

- 昔は Atom Editor は全体として CoffeeScript / CSON を使うようになっていましたが、今は JavaScript / JSON で開発を行うようになっています。
- コードを実行しても自動的には反映してくれません。動作確認する場合 [Ctrl]+[Alt]+[Cmd]+[l] でウィンドウのリロードを行うようにしてください。
- [Ctrl]+[Alt]+[Cmd]+[p] で Jasmine で実装したテストコードを実行可能です。
- ファイル1行目に `'use babel'` を入れておけば、 Babel による変換が効くので、最新の仕様も使えます。
- `atom` は自動的に import された状態で使えます。
   - ドキュメントは https://atom.io/docs/api/v1.23.3/AtomEnvironment を参照

## 公開手順

以下のコマンド一つで公開できます。 [version-level] には major / minor / patch のいずれかを指定します。
コマンドを叩くと、 [version-level] で指定したレベルで package.json のバージョンをインクリメントし、 現在のブランチで git commit / tag / push を行い、同時に Atom のパッケージとしてアップロードしてくれるようです。

```sh
apm publish [version-level]
```
