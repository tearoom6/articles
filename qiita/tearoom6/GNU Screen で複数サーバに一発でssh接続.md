
以前、[Terminalで複数サーバに一発でssh接続](http://qiita.com/tearoom6/items/e3c8b27bc4de9985e471) という投稿をしましたが、今回はそのGNU Screen版です。

## やりたいこと

- コマンド1つで、Screenを起動し、画面を4分割し、さらにそれぞれの画面で別々のサーバに接続した状態にさせます。

## 手順

以下のようなScreen用のスクリプトを記述します。

- `^M`という記載がありますが、これは特殊文字(CR)で、Vim上から`<C-v><C-m>`(Control-vの後にControl-m)と入力すれば、打込めます。(通常のコピペでは無理?)
- `^M`を省略したとしても、下記のスクリプトは動きます。ただし、最後のコマンドの部分は入力どまりで実行はされないので、手動で実行する必要があります。

```~/.screenrc_init
# もともとの起動ファイルを読み込む
source ~/.screenrc

# 4つのウィンドウをタイトル付きで立ち上げる
screen -t local 0
screen -t server1 1
screen -t server2 2
screen -t server3 3
# 画面を4分割し、それぞれにウィンドウを配置
split -v
split
focus
select 1
focus
split
select 2
focus
select 3
focus
select 0
# 各ウィンドウで別々のコマンドを実行
stuff 'date^M'
focus
stuff 'ssh server001^M'
focus
stuff 'ssh server002^M'
focus
stuff 'ssh server003^M'
```

- そして、シェル上で以下のように実行すれば、画面4分割されたScreenが起動するかと思います。

```Bash
$ screen -c ~/.screenrc_init
```

