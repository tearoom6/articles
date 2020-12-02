タイトル負けしそうな感じしかしない・・・。

私の最新の Homebrew の使い方をまとめておきます。

昔書いた以下の記事も参考にしていただければです。

- [brew tap で入れる Homebrew 外部コマンド - Qiita](https://qiita.com/tearoom6/items/1abf24ca6d872e6579b0)
- [Homebrew の処理ですごく時間がかかる場合 - Qiita](https://qiita.com/tearoom6/items/470e16540e3deaae5efa)

## セットアップ

基本方針として `brew bundle` を使うようにしています。

まず Homebrew を公式ページにある通りにインストールします。

```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

`brew bundle` は明示的にインストールしなくても大丈夫です。

あとは、あらかじめ用意しておいた `Brewfile` のあるディレクトリで `brew bundle` を実行すれば、必要なものを全て入れてくれます。
とても簡単。

```sh
brew bundle
```

参考までに、私の Brewfile を晒しておきます。

<https://github.com/tearoom6/dotfiles/blob/master/Brewfile>

こんな感じで、一時的にインストールしたけど、全ての環境で使うわけではない、という場合は、コメントアウトしておくと、使いたい時だけ使えて便利ですね。

## バージョン管理

`brew bundle` を実行すると `Brewfile.lock.json` が生成されますが、こちらはインストールした際のバージョン情報を記録するものであって、例えば Bundler の `Gemfile.lock` などとは違って、インストールされるバージョンを固定するものではないそうです。

Homebrew 自体が、特定のバージョンをインストールするという機能を備えていないからです。
(代替方法として、バージョンごとの formula を指定できるようにしている場合はありますね)

したがって、運用としては、定期的に最新バージョンにアップデートしておく、ということになるかと思います。

Homebrew インストールしている全てのバージョンを **アグレッシブに** アップデートするには、以下のようにすれば OK です。

```sh
brew update
brew upgrade --cask --greedy
brew bundle
```

> References

- [macos - Upgrade all the casks installed via Homebrew Cask - Stack Overflow](https://stackoverflow.com/questions/31968664/)
