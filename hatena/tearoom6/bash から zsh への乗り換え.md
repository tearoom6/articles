完全に個人的なメモ。

macOS Catalina から、標準シェルが zsh に変更された。
特に bash で困ってないけど、 shell 開くたびに以下のようなメッセージが出てくるのも耐え難いので、今更だけど乗り換えることにした。

```
The default interactive shell is now zsh.
To update your account to use zsh, please run `chsh -s /bin/zsh`.
For more details, please visit https://support.apple.com/kb/HT208050.
```


操作面はさておいて、乗り換えで大切なのは dotfiles の扱い。
現時点で `.bash_profile` と `.bashrc` を以下のように設定しているので、これらを円滑に移行する。

- [dotfiles/.bash_profile](https://github.com/tearoom6/dotfiles/blob/9b8e88dd966c32456cf2d1e6f547d366f7d60515/.bash_profile)
- [dotfiles/.bashrc](https://github.com/tearoom6/dotfiles/blob/9b8e88dd966c32456cf2d1e6f547d366f7d60515/.bashrc)

一応、 zsh の設定系の framework として、以下のようなものがあるらしいけど、細かいカスタマイズこそが shell の醍醐味じゃろってことで使わない。
(とかいいつつ、特に .bashrc とかメンテしてなかったけど...)

- [Oh My Zsh](https://github.com/ohmyzsh/ohmyzsh)
- [Prezto](https://github.com/sorin-ionescu/prezto)

framework よりは控えめな plugin manager 系のツールもいろいろあるようなので、不便を感じたらこちらは使ってもよいかも。
(結構カオス状態だな...)

- [Antigen](https://github.com/zsh-users/antigen)
- [zgen](https://github.com/tarjoilija/zgen)
- [Zinit](https://github.com/zdharma/zinit)
- [zplug](https://github.com/zplug/zplug)


とりあえず、今回はカスタムでやってみるのです。

`.bash_profile` と `.bashrc` の内容を `.zshrc` に全てまとめる。
また、 zsh の便利機能的なものもちょいちょい追加。

history のオプションに関して、 `INC_APPEND_HISTORY` と `SHARE_HISTORY` の違いが少しわかりにくかったので、メモ。
デフォルトでは、 HISTFILE へは、 shell が exit するタイミングで書き込まれるんだけど、 `INC_APPEND_HISTORY` を有効にすると、コマンドが発行される度 (実行前に) HISTFILE に書き込まれるようになる。ただ、これだけだと、他の window で開いている shell からは、この HISTFILE に書き込まれた履歴は使われない。(メモリ上にある、各 shell ごとに管理されている履歴が使われる)
`SHARE_HISTORY` を有効にすると、 `INC_APPEND_HISTORY` の効果にプラスして、他の window で開いている shell からも即座に、 HISTFILE に書かれた履歴が使われるようになる。別にそこまでは望まないかなっていう場合は `INC_APPEND_HISTORY` を使えばいい。
なお、 `INC_APPEND_HISTORY_TIME` というオプションを有効にすると、コマンド開始時刻だけでなく、実行時間も書き込まれる。 (つまりコマンド終了時に書き込まれる)
`INC_APPEND_HISTORY`, `INC_APPEND_HISTORY_TIME`, `SHARE_HISTORY` は互いに排他的なオプションとなっている。

prompt での Git のステータス表示に関しては、 `vcs_info` というモジュールを使ってみた。
push や stash のステータス表示を標準ではサポートしていないようで、以前に比べてそのあたりは不満はあるけど、カスタマイズはできるらしいし、まずは標準のまま使ってみる。

補完系に bash では [scop/bash-completion](https://github.com/scop/bash-completion) を使っていた。
zsh は標準でもある程度のコマンド群をサポートしてくれているみたいだけど、追加で主要なものを追加しておく。

```sh
curl -L https://raw.githubusercontent.com/docker/cli/master/contrib/completion/zsh/_docker > ~/.zsh.d/completions/_docker
curl -L https://raw.githubusercontent.com/docker/compose/master/contrib/completion/zsh/_docker-compose > ~/.zsh.d/completions/_docker-compose
```

また 3rd-party 製のものが [zsh-users/zsh-completions](https://github.com/zsh-users/zsh-completions) に公開されているので、これも使用する。

```sh
git clone git://github.com/zsh-users/zsh-completions.git ~/.zsh.d/completions/zsh-completions
mv ~/.zsh.d/completions/zsh-completions/src/* ~/.zsh.d/completions/
rm -rf ~/.zsh.d/completions/zsh-completions
```

最終的にこんな感じになった。たぶん、使いながらちょいちょい変更することにはなるだろう。

- [dotfiles/.zshrc](https://github.com/tearoom6/dotfiles/blob/8e9434b7308aeb0b44c7879497952f3ae51f0de2/.zshrc)

最後に以下のコマンドでバッツん切り替える。

```sh
chsh -s /bin/zsh
```

> References

- [zsh: 15 Parameters](http://zsh.sourceforge.net/Doc/Release/Parameters.html)
- [zsh: 16 Options](http://zsh.sourceforge.net/Doc/Release/Options.html)
- [Use zsh as the default shell on your Mac - Apple Support](https://support.apple.com/en-us/HT208050)
- [zshのhistoryを便利に使うためのTips - Qiita](https://qiita.com/syui/items/c1a1567b2b76051f50c4)
- [最強のシェル zsh - Qiita](https://qiita.com/yasuoy/items/8d100af5ef903438fc90)
- [初心者向け：Zshの導入 - Qiita](https://qiita.com/iwaseasahi/items/a2b00b65ebd06785b443)
- [zshのオプション設定の分かりにくいところまとめ - Qiita](https://qiita.com/mollifier/items/02eb36b5a58d119c2f1c)
- [Zsh 入門者のための超速設定ガイド - Qiita](https://qiita.com/uasi/items/c4288dd835a65eb9d709)
- [.zshrcの設定例（設定内容の説明コメント付き） - Qiita](https://qiita.com/d-dai/items/d7f329b7d82e2165dab3)
- [めんどくせーからzshrcそのまま晒す - Qiita](https://qiita.com/kotashiratsuka/items/ae37757aa1d31d1d4f4d)
- [zshrcを見直してまとめる - Qiita](https://qiita.com/Kakuni/items/a8025e075926272f491d)
- [Shellライフをそこそこ快適に！僕のこだわりzsh設定](https://suin.io/568)
- [【連載】漢のzsh | マイナビニュース](https://news.mynavi.jp/series/zsh)
- [bashとzshの違い。bashからの乗り換えで気をつけるべき16の事柄](https://kanasys.com/tech/803)
- [zshで究極のオペレーションを：連載｜gihyo.jp … 技術評論社](https://gihyo.jp/dev/serial/01/zsh-book)
- [dotfiles/zshrc at master · joshtronic/dotfiles](https://github.com/joshtronic/dotfiles/blob/master/zshrc)
- [Preztoによるzshプラグインの導入: 黄昏てなんかいられない](http://kronus9.sblo.jp/article/135674974.html)
- [Zshプラグインをzgenでシンプルに管理する - longkey1's blog](https://blog.longkey1.net/2018/03/24/zgen/)
- [シェルスクリプトでの関数の書き方について](https://rcmdnk.com/blog/2018/01/29/computer-bash-zsh/)
- [zshでプロンプトをカラー表示する - Qiita](https://qiita.com/mollifier/items/40d57e1da1b325903659)
- [shell - zsh config - to export or not to export? - Super User](https://superuser.com/questions/598810/)
- [zshのターミナルにリポジトリの情報を表示してみる · けんごのお屋敷](http://tkengo.github.io/blog/2013/05/12/zsh-vcs-info/)
- [zsh の vcs_info に独自の処理を追加して stash 数とか push していない件数とか何でも表示する - Qiita](https://qiita.com/mollifier/items/8d5a627d773758dd8078)
- [zshのメニュー補完で候補をインタラクティブに絞り込む - Qiita](https://qiita.com/ToruIwashita/items/5cfa382e9ae2bd0502be)
- [今更ながらzshのREPORTTIME便利 - キモブロ](http://kimoto.hatenablog.com/entry/2012/08/14/112500)
- [command line - CTRL-a and CTRL-e map incorrectly in tmux - Ask Ubuntu](https://askubuntu.com/questions/1155199/)
- [よく実行するコマンドにキーバインドを割り当てると捗る話 - Qiita](https://qiita.com/yuku_t/items/e58cbecf13407446bd50)
