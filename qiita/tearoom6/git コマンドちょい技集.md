
Git は毎日使うものだから、皆さん日々の研鑽に余念がないツールの一つでしょう。GUIツール等で見るのもいいですが、どのような環境にいっても、シェル上でさっと git コマンドが使えるとかっこいいかもしれないですね(!?)

今回はちょっとした便利な使い方をご紹介します。新たな発見があれば嬉しいです！


## 別ブランチを最新化

- ブランチを切り替えなくても、特定のローカルブランチを最新にする方法です
- 実際にはローカルブランチは必要な時に最新化しておけばいいですが、なんとなくmasterは定期的に最新化しておきたい気がします
- そんな時は以下のようにすればいいです
- もちろん、ローカルmasterに独自のコミットがあって、merge時にconflictする場合は、以下のコマンドは失敗しますが

```
$ git fetch origin master:master
```

- このコマンドは、リモートブランチをローカルに持ってくるのにも使えます
- 通常は下のようにすると思いますが・・・

```
$ git checkout -b branchA origin/branchA
```

- 以下も同意になります
- まぁ上のコマンドでいいですが。。。

```
$ git fetch origin branchA:branchA
$ git branch --set-upstream-to=origin/branchA branchA
```


## 特定のファイルを覗き見

- Gitコマンドをきちんと使えば、わざわざブランチを切り替えなくても、別ブランチやある時点のコミットのファイルをさっと確認することができます
- たとえば、origin/branchA の path/to/file ファイルの内容を確認したい時は次のようにします

```
$ git show origin/branchA:path/to/file
```

- また、過去の commitA (hash:1705c36756) の時点での path/to/file ファイルの内容を確認したい時は次のようにします

```
$ git show 15c7670356:path/to/file
```

- git blame の場合は次のようにします

```
$ git blame origin/branchA -- path/to/file
$ git blame 15c7670356 -- path/to/file
```

- 現在のファイルシステム上の特定ブランチ、特定コミットのファイルを持ってくることも簡単です
- ブランチ切り替えてファイルをコピーして戻してから貼り付け・・・なんてしないようにしましょう

```
$ git checkout origin/branchA -- path/to/file
$ git checkout 15c7670356 -- path/to/file
```


## 差分を見る

### git diff の場合

- origin/branchA と branchA の差分を見たい場合は通常次のようにします
- しかし、これだと origin/branchA の最新を branchA に取り込んでいない場合、origin/branchA に加えられた変更も、差分として見えてしまいますよね
- branchA に origin/branchA の最新を取り込めばいいんですが、branchA はまだ中途半端な作業中で、自分的にマージしたくない場合もよくあります

```
$ git diff origin/branchA..branchA
```

- そんな時は.を1個足しましょう

```
$ git diff origin/branchA...branchA
```

- これは、"origin/branchA と branchA の共通祖先" と branchA との差分を表示します
- つまり、"origin/branchA" で独自に加えられたコミットによる分は表示しないです

### git log の場合

- git log の場合も .. と ... が使えますが、git diff の場合とは少し違います
- つまり、次の例は、origin/branchA には含まれず、branchA にのみ含まれるコミット一覧が表示されます

```
$ git log origin/branchA..branchA
```

- それに対して、次の例は、origin/branchA と branchA のどちらか片方にのみ存在するコミットを表示します
- .. の場合は左右のどちらに書くかが重要だけど、... の場合は左右どちらに書いても関係ないという感じですね

```
$ git log origin/branchA...branchA
```

- ただ、オプション --right-only / --left-only を指定することで、左右いずれかのブランチにのみ存在するコミットを表示するように指定することも可能です

```
$ git log --right-only origin/branchA...branchA
```


## 最後に

何か間違いや補足があればご指摘お願いします！


