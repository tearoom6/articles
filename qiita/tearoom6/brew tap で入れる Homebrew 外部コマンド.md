
## brew tap

brew tap は、公式以外の formula を追加することのできる Homebrew のサブコマンドです。

```
$ brew tap <userName>/<repository>
```

とすると、GitHub 公開リポジトリ (`https://github.com/<userName>/homebrew-<repository>`) が参照され、`/usr/local/Library/Taps`以下に取り込まれます。

```
$ brew tap <url>
```

とすると、GitHub 以外のリポジトリからも取り込むことが可能です。(HTTP/HTTPS である必要もありません)

```
$ brew tap
```

と打てば、既に取り込んだ taps の一覧を表示することができます。


## External Commands

[External Commands](https://github.com/Homebrew/homebrew/blob/master/share/doc/homebrew/External-Commands.md) にあるように、Homebrew は外部コマンドを取り込むことができます。取り込んだ外部コマンドは、`brew commands`を実行した時に"External commands"として表示されます。

```
$ brew commands
Built-in commands
...

External commands
...
```

brew tap で取り込むものが多いです。
有名どころを以下にいくつか記載します。

## [Homebrew Services](https://github.com/homebrew/homebrew-services)

- `launchctl`と連携させるための拡張です。

### インストール

```
$ brew tap homebrew/services
```

### 使い方

```
# mysql を Homebrew でインストールしたとします
$ brew install mysql

# すると、Homebrew のコマンドとして mysql の状態管理をできます
$ brew services start mysql
$ brew services stop mysql
$ brew services restart mysql

# brew service で扱えるサービス一覧を表示します
$ brew services list
```

## [Brew Bundle](https://github.com/Homebrew/homebrew-bundle)

- Ruby の Bundler のように、インストールしたいパッケージ一覧をファイルに記載するようにする拡張です。
- 以前は Brewdler と呼ばれていたみたいです。

### インストール

```
$ brew tap Homebrew/bundle
```

### 使い方

現在のディレクトリに `Brewfile` というファイルを用意して、インストールしたいパッケージを記載します。
(以下の例は GitHub の README からそのまま借用しています)

```Brewfile
cask_args appdir: '/Applications'
tap 'caskroom/cask'
tap 'telemachus/brew', 'https://telemachus@bitbucket.org/telemachus/brew.git'
brew 'emacs', args: ['with-cocoa', 'with-gnutls']
brew 'redis', restart_service: true
brew 'mongodb'
brew 'sphinx'
brew 'imagemagick'
brew 'mysql'
cask 'google-chrome'
cask 'java' unless system '/usr/libexec/java_home --failfast'
cask 'firefox', args: { appdir: '/Applications' }
```

そして、以下のコマンドで一括インストールします。

```
$ brew bundle
```

他にも次のような使い方ができます。

```
# 現在 Homebrew でインストールしているパッケージを Brewfile に全て書き出す
$ brew bundle dump
# Brewfile に書いていないパッケージを全てアンインストール
$ brew bundle cleanup
# Brewfile に記載のあるものでインストール・アップグレードすべきものを表示
$ brew bundle check
```


## [Homebrew-Cask](https://github.com/caskroom/homebrew-cask)

- バイナリ形式で配布されているMacアプリケーションをインストールするための拡張です。

### インストール

```
$ brew tap caskroom/cask
$ brew install brew-cask
```

### 使い方

`/opt/homebrew-cask/Caskroom`にインストールされて、`/Applications`にショートカットが置かれるようです。

```
# パッケージを検索
brew cask search google
# パッケージの情報表示
brew cask info google-chrome
# パッケージの homepage を表示
brew cask home google-chrome
# インストール
brew cask install google-chrome
# アンインストール
brew cask uninstall google-chrome
# インストール済みのものを表示
brew cask list
# 不要なダウンロード済みファイルを削除
brew cask cleanup
```



