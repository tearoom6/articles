先日、 GitHub のセキュリティアラート (Lodash に関する CVE-2019-10744) が飛んできて、対応を行ったので、メモメモ。
こういう対応って一時的にパパっとやっちゃって、メモ残さないから、後から何やったけなってなって、良くない。

今回、アップデートしたリポジトリは、以下の6つ。

- [tearoom6/tearoom6.github.io: Portfolio Site](https://github.com/tearoom6/tearoom6.github.io)
- [tearoom6/sync-sync: Atom package for managing posts & documents of blog/collaboration services](https://github.com/tearoom6/sync-sync)
- [tearoom6/s3uploader: Files uploader to Amazon S3 by drag & drop.](https://github.com/tearoom6/s3uploader)
- [tearoom6/MemoryTouch: Breakthrough - Unity project](https://github.com/tearoom6/MemoryTouch)
- [tearoom6/QuickDrive2: Chrome extension to provide quick access to your Google Drive files.](https://github.com/tearoom6/QuickDrive2)
- [tearoom6/esa-plus: Chrome Extension to add more features to esa.io.](https://github.com/tearoom6/esa-plus)


## Yarn での依存ライブラリのバージョンアップデート

普通に `yarn upgrade` を実行だと、 `package.json` で縛ったバージョン以上には上げてくれない。
あくまで最新にアップデートしたい場合には、 `--latest` を付けます。

```sh
# 特定のライブラリだけアップデートする場合
yarn upgrade lodash --latest

# 全部アップデートする場合
yarn upgrade  --latest
```


## Gulp v3->v4 の移行について

どうせならと、全ライブラリをアップデートした影響で、 Gulp のバージョンもあがって、後方互換性のない変更にぶち当たりました。

gulp4 では、単体のタスクは `gulp.task(name)` (引数1つ) で、複数タスクからなるパイプラインは `gulp.series` (直列実行) / `gulp.parallel` (並列実行) で表現する、というようになったようです。

これに伴う、主な変更点は、以下のとおりです。

- `run-sequence` の代わりに `gulp.series` / `gulp.parallel` を使う
   - `run-sequence` は Gulp にとって 3rd party library ですが、 Gulp 本体が上述の通り、同様の機能を提供したので、そちらを使うようにします
- gulp3 で、 task の依存関係を示すために、 `gulp.task` の第2引数に依存性のある task の名前の配列を渡していたところには、 `gulp.task(name)` / `gulp.series` / `gulp.parallel` を使う
   - 依存性のあるタスクというのは、要はパイプライン実行するということなので、理にかなってますね
- gulp3 で、 `gulp.watch` の第2引数に実行されるべき task の名前の配列を渡していたところも同様に、 `gulp.task(name)` / `gulp.series` / `gulp.parallel` を使う

これらの変更に伴って、若干、非同期処理終了を告げる `callback` の呼出し方が変わります。
こうやって全体通して見ると、非常に一貫性があって、納得感のある変更となっていました。

なお、 `gulpfile.babel.js` を使って ES6 形式で gulpfile を書いている場合には、 [`@babel/register`](https://github.com/babel/babel/tree/master/packages/babel-register) をインストールだけする必要がありそうです。 (ちょっとハマりました)
この子は今までも `babel-core/register`, `babel-register` と名前を変えている厄介な子みたいです。 (babel 系、パッケージ名変えすぎ!?)

> References

- [yarn upgradeのあれこれ - Qiita](https://qiita.com/teinen_qiita/items/18ca1fb433914e09c9e4)
- [Gulp v4 移行メモ - かもメモ](https://chaika.hatenablog.com/entry/2018/06/02/090000)
- [gulp3→4の変更点に気をつけよう！ - Qiita](https://qiita.com/tatsuo-iriyama/items/08ba4bd621b7fdedcc4e)
- [これからはじめるGulp #3：gulp.watchでファイルの変更を監視しタスクを実行する ｜ DevelopersIO](https://dev.classmethod.jp/client-side/javascript/gulp-solo-adv-cal-03/)
- [Using ES6 with gulp — Mark Goodyear — Front-end developer and designer](https://markgoodyear.com/2015/06/using-es6-with-gulp/)
- [gulp/2-javascript-and-gulpfiles.md at master · gulpjs/gulp](https://github.com/gulpjs/gulp/blob/master/docs/getting-started/2-javascript-and-gulpfiles.md)
- [Babelを6系にアップデートしたらgulpでこける - Qiita](https://qiita.com/sawapi/items/e11d0cfbbacd078b2c3a)
- [Gulp fails to load '@babel/register' · Issue #171 · gulpjs/gulp-cli](https://github.com/gulpjs/gulp-cli/issues/171)
- [Failed to load external module babel-register · Issue #1631 · gulpjs/gulp](https://github.com/gulpjs/gulp/issues/1631)
