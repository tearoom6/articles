
[Terminal](http://qiita.com/tearoom6/items/e3c8b27bc4de9985e471), [Screen](http://qiita.com/tearoom6/items/e01c04eb0d0a814594a3) とやったので、惰性で tmux もやってみました。

tmux の場合は、[Tmuxinator](https://github.com/tmuxinator/tmuxinator) というツールが非常に便利なので、これを使うといいでしょう。使い方については他の方も解説されているので、割愛します。

## やりたいこと

- コマンド1つで、tmux を起動し、画面を 4 panes に分割し、さらにそれぞれの画面で別々のサーバに接続した状態にさせます。
- ついでに、別 window でデプロイ用サーバにも接続しておきます。

## 手順

- まずはインストール

```
$ gem install tmuxinator
```

- 以下のように $EDITOR を定義しておきます

```
$ export EDITOR='vim'
```

- プロジェクト作成
- mux コマンドは、tmuxinator コマンドの alias
- `~/.tmuxinator/PROJECT.yml` が作られ、vimが開きます

```
$ mux new [PROJECT]
```

- 以下のように入力

```YAML:~/.tmuxinator/multi_ssh.yml
name: multi_ssh
root: ~/
windows:
  - multi_ssh:
      layout: tiled
      panes:
        - local:
          - date
        - server001:
          - ssh server001
          - cd /var/log
        - server002:
          - ssh server002
          - cd /var/log
        - server003:
          - ssh server003
          - cd /var/log
  - deploy:
    - ssh server_deploy
    - cd ~/deploy
```

- 起動すると、tmux が開き、上記で入力したコマンドが分割した画面で次々実行されていきます

```
# Start tmux session.
$ mux start PROJECT
```

## 補足

### `layout` について

標準の layout として、以下の5つが選べます。

- even-horizontal
   - 均等に縦縞上に並びます
- even-vertical
   - 均等に横縞上に並びます
- main-horizontal
   - メインのpaneが上に表示され、残りが下に縦縞上に並びます
- main-vertical
   - メインのpaneが左に表示され、残りが右に横縞上に並びます
- tiled
   - 格子状

他にもサイズをカスタム指定することもできます。

### `mux local`

- 特定のディレクトリ (プロジェクトルートなど) ごとに Tmuxinator 環境を用意する場合、ディレクトリに `./.tmuxinator.yml` を作成しておきます
- 以下のコマンドで起動できます

```
# `./.tmuxinator.yml` を読み取り、起動
$ mux local
```

