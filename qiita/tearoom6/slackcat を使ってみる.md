
## slackcat

- slackcat を使えばシェルからSlackへ投稿できる

<http://slackcat.chat/>

## インストール

- MacOSX の場合は、Homebrew を使って簡単にインストールできる

```Bash
$ brew update
$ brew install slackcat
```

- Linux にもインストール可能 (公式ページ参照)

## 設定

<https://github.com/vektorlab/slackcat/blob/master/docs/configuration-guide.md>

- 以下のコマンドを叩けば、ブラウザが開いてSlackとの連携画面が表示される

```Bash
$ slackcat --configure
```

- 許可すると、トークンが発行されるので、`~/.slackcat` に保存する

```Bash
$ echo "xoxp-xxxx-xxxx-xxxx-xxxx" > ~/.slackcat
```

- Slackをマルチチームで使っている場合には、次のように設定するといい

```:~/.slackcat
team1 = xoxp--xxxx-xxxx-xxxx-xxxx
team2 = xoxp--xxxx-xxxx-xxxx-yyyy
default_team = team1
default_channel = develop
```

## 使い方

- 以下のような感じで使える

```
# ストリームモードで使用開始
$ slackcat --channel team2:general --stream --plain

# ファイルをスラックに投稿
$ slackcat --channel general --filename test.jpg ~/Images/test.jpg

# コマンド出力をプレーンテキストとして投稿
$ echo test | slackcat --tee --stream --plain

# コマンド出力をスニペットとして投稿
$ date | slackcat --tee
```


