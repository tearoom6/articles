## 概要

今更ですが、いつぞや Git のバージョンを上げた後、 `git pull` したときに、以下の warning が出るようになりました。

```
warning: Pulling without specifying how to reconcile divergent branches is
discouraged. You can squelch this message by running one of the following
commands sometime before your next pull:

  git config pull.rebase false  # merge (the default strategy)
  git config pull.rebase true   # rebase
  git config pull.ff only       # fast-forward only

You can replace "git config" with "git config --global" to set a default
preference for all repositories. You can also pass --rebase, --no-rebase,
or --ff-only on the command line to override the configured default per
invocation.
```

いつかちゃんと対処しようと思いつつ、時間が経ってしまったので、ちゃんと対処します。

このメッセージは、検索すると山ほど情報が出てくる通り Git 2.27.0 で導入されたメッセージのようです。
メッセージにあるように、以下の **3 つの設定のうち、いずれかを実施**すれば、 warning は出なくなります。

- `git config pull.rebase false`
- `git config pull.rebase true`
- `git config pull.ff only`

特に、今までの動作に不満がないという人は `git config --global pull.rebase false` をやっておけば、今までの挙動の通り、 warning だけ出なくなります。

まぁ、なのですが、せっかくの機会なので、振り返りを込めて、もうちょっと理解を深めておきます。

## `git merge` の選択肢の理解

`git pull` というのは、基本的には remote branch の local branch への "マージ" です。

`git merge` の戦略には、以下の 2 つがあります。

- **merge commit を作成** (3-way merge が行われる)
- (必要なら merge 対象のブランチを rebase した上での) **fast-forward** (merge commit を作成しない)

例えば、以下のように *main* branch と *develop* が *commit a* 以降で分岐しているとします。

![git_merge_base.png](https://files.tearoom6.biz/9271b94a-d1a7-4f78-a944-0f811c4e3e8b.png)

merge commit を作って merge する場合は、それぞれの branch の履歴はそのままに、新たに *merge commit m* が作られます。

![git_merge_merge_commit.png](https://files.tearoom6.biz/1744e2c2-f2ed-4522-8ff0-5e85243afed0.png)

fast-forward する場合は、 merge commit を作成しないということなので、 *develop* 分岐後に *main* branch には何もコミットがされていないことが条件になります。
(つまり *main* branch が *commit a* の時点のままなら、そのまま *develop* を fast-forward merge できる)

![git_merge_base_simple.png](https://files.tearoom6.biz/18ff4107-efbf-4bbb-8fc8-68c84d677c2d.png)

そうでない場合は、まず *develop* の分岐元が *main* の HEAD コミットである *commit d* になるように *develop* の付け替え (rebase) を行います。
*develop* 側の全ての commit は、元の commit とは別物の *e'*, *f'*, *g'* になります。
このように、履歴が書き換わるので、コンフリクトにご注意ください。
また、 Git 上はコンフリクトしなくても、コードの意味的・実際の動作上、コンフリクトする場合があるので、テストは必ず行いましょう。 (これは merge commit 作るときも同じですが)

![git_merge_fast_forward_1.png](https://files.tearoom6.biz/8c3480c0-6b14-41cf-bf19-207b8ab2eb5e.png)

その上で fast-forward を行います。これは *main* branch の HEAD を *commit g'* に移動させるだけです。このように履歴の上では一直線になります。

![git_merge_fast_forward_2.png](https://files.tearoom6.biz/297a2b38-fe60-4ab4-a619-c39d9d3fa252.png)

## `git pull` 操作の選択肢の理解

さて、そこで、先程の warning に対処するための 3 つの選択肢を考えたいのですが、その前に、 `git pull` の `--rebase` option について確認します。
何も指定しない場合は、 `--rebase false` と同じで、これがデフォルトの動作です。 `--rebase` を指定した場合は `--rebase true` と同じです。
他にも rebase の挙動を細かく制御するための `--rebase merges`, `--rebase preserve`, `--rebase interactive` がありますが、割愛します。

![git_pull_base.png](https://files.tearoom6.biz/56a47f51-f242-4be9-aa9e-bc5becfda177.png)

通常、 `git pull` (`--rebase` を付けない) を行うと、 `git fetch` を行った上で `git merge` を行う流れになります。
デフォルトの設定では、 rebase せずに fast-forward 可能な場合は fast-forward を行い、そうでない場合は、先程の図のように merge commit を生成しようとします。

![git_pull_merge.png](https://files.tearoom6.biz/aa77cffc-11a3-4a90-b5aa-77e403b97136.png)

それに対して `git pull --rebase` を行うと、 `git fetch` を行った上で、ローカルブランチ (この例では *main* branch) の `git rebase` を行う流れになります。
上述の `git merge` の際の fast-forward の説明では、 `git merge` を行うために *develop* 側の rebase を行ってから fast-forward を実行しましたが、この場合は、 *main* 側の rebase を行っていることに注意してください。(*main* - *develop* の関係が *origin/main* - *main* の関係になっているだけで、全く同じことではあるのですが)

こうすることで、リモートブランチにはなんら影響を与えることがないのです。

![git_pull_rebase_1.png](https://files.tearoom6.biz/b7bb9172-2739-458c-befa-e9ef30849a56.png)

![git_pull_rebase_2.png](https://files.tearoom6.biz/5f351eb3-851f-45f2-8ed0-aac9279e14cd.png)

`git pull` の `--rebase` option の使い所ですが、主に以下の 2 つがあるようです。

- 同一ブランチに対して、複数人が同時並行で開発を行う場合
   - ローカルブランチを rebase した上で push することで、 merge commit が乱立することを防ぐことができます
- Pull Request マージ前に、マージ先のデフォルトブランチを取り込んで、テストなどを行う場合
   - 例えば、 Pull Request の *feature* ブランチにマージ先である *main* ブランチをあらかじめマージしても、 *feature* ブランチには merge commit が作られずに済みます (結果的に履歴が綺麗になります)

また `git pull` の fast-forward 関連の以下の 3 つの option も確認します。 (`git merge` にも全く同じ option があります)
以下の表にまとめます。 (デフォルトは `--ff` です)

| option      | fast-forward 可能な場合 | fast-forward できない場合 |
|-------------|-----------------------|-------------------------|
| `--ff`      | fast-forward で merge する | merge commit を生成する |
| `--no-ff`   | merge commit を生成する | merge commit を生成する |
| `--ff-only` | fast-forward で merge する | merge せず、エラー終了する |

## 先程の 3 つの選択肢をそれぞれ選んだ場合の挙動

ここまでくれば、 `git pull` にオプションを付けずに実行したときの動作について、 Git が以下のいずれかを要求していることが分かります。

### `git config pull.rebase false`

デフォルトの挙動です。`git pull` に `--rebase` option を付けずに実行するのと同じです。
標準動作では rebase せずに fast-forward 可能な場合は fast-forward を行い、そうでない場合は、 merge commit を生成しようとします。

### `git config pull.rebase true`

`git pull --rebase` を行う場合と同じです。
`git fetch` を行った上で、ローカルブランチに対して `rebase` を実施するので、merge commit が作られずに、コミット履歴が一直線になります。

### `git config pull.ff only`

`--ff-only` option を付けた時と同様、 fast-forward 可能な場合のみ、 fast-forward します。
そうでない場合は、 merge/rebase せず、エラー終了します。

## 結局

merge/pull の戦略をどうするかは、結局は、チームごとの方針だと思います。
基本的にはチーム内で揃えておいた方が、 Git の履歴を見るときにやりやすいとは思います。人が増えるとなかなかそれも難しかったりしますが。

merge/pull の戦略は、コードの履歴をどうまとめたいかという話であって、現在のスナップショットには特に影響を及ぼしません。 (Git 運用時の操作で問題が起きやすいとかの影響は出るかもしれませんが)
特に致命的な問題はないので、結論としては、悩んだときはデフォルトの設定にしておきましょう😂

> References

- [https://raw.githubusercontent.com/git/git/master/Documentation/RelNotes/2.27.0.txt](https://raw.githubusercontent.com/git/git/master/Documentation/RelNotes/2.27.0.txt)
- [How to deal with this git warning? "Pulling without specifying how to reconcile divergent branches is discouraged" - Stack Overflow](https://stackoverflow.com/questions/62653114/)
- [Why You Should Use git pull --ff-only | sffc's Tech Blog](https://blog.sffc.xyz/post/185195398930/why-you-should-use-git-pull-ff-only)
- [オプションなし git pull でデフォルトの挙動が未設定だと警告が出る - Qiita](https://qiita.com/toranoko0518/items/f6340a69a60238f7d328)
- [Git 2.27.0 から git pull をすると表示されるようになった "Pulling without specifying how to reconcile divergent branches is discouraged." について - esm アジャイル事業部 開発者ブログ](https://blog.agile.esm.co.jp/entry/git-warns-pulling-without-specifying-how-to-reconcile-divergent-branches)
- [git pull エラー "Pulling without specifying how to reconcile divergent branches is discouraged." | ハックノート](https://hacknote.jp/archives/57795/)
- [ブランチの統合｜サル先生のGit入門【プロジェクト管理ツールBacklog】](https://backlog.com/ja/git-tutorial/stepup/04/)
- [Git - git-merge Documentation](https://git-scm.com/docs/git-merge)
- [Git - git-pull Documentation](https://git-scm.com/docs/git-pull)
- [Git - git-config Documentation](https://git-scm.com/docs/git-config)
- [Git - リベース](https://git-scm.com/book/ja/v2/Git-%E3%81%AE%E3%83%96%E3%83%A9%E3%83%B3%E3%83%81%E6%A9%9F%E8%83%BD-%E3%83%AA%E3%83%99%E3%83%BC%E3%82%B9)
- [fast-forwardマージから理解するgit rebase - Qiita](https://qiita.com/vsanna/items/451b42f886c599a16a55)
- [Git のファストフォワードマージとは - yu8mada](https://yu8mada.com/2018/08/15/what-is-a-fast-forward-merge-in-git/)
- [Git fast-forward な状態にして update branch で master を取り込んだ commit を残さずに済むための git rebase - かもメモ](https://chaika.hatenablog.com/entry/2019/09/24/083000)
- [git pull と git pull –rebase の違いって？図を交えて説明します！ – KRAY Inc.](https://kray.jp/blog/git-pull-rebase/)
- [3-way mergeについて調べた - Qiita](https://qiita.com/myouga/items/fb680e689970c2ec97ba)
- [GithubでのWeb上からのマージの仕方3種とその使いどころ - Qiita](https://qiita.com/ko-he-8/items/94e872f2154829c868df)
- [何故 git rebase は駄目で git pull –rebase はいいのか « LANCARD.LAB｜ランカードコムのスタッフブログ](https://www.lancard.com/blog/2016/11/07/git-rebase-and-pull-rebase/)
