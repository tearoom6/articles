
## manifest.json とは

- manifest.json とは、Chrome拡張機能やChromeアプリのメタ情報をChromeや、Chrome Web Storeに伝えるために拡張機能のパッケージに含めておくJSONファイルです
- 拡張機能のバージョン情報を手動で設定するには、manifest.json にバージョン情報を記載します

https://developer.chrome.com/extensions/manifest

## package.json

- Chrome拡張機能は、Webアプリですから、package.jsonも入っているかと思います
- マメな人は、package.jsonの方のバージョン情報も更新しているのではと思います

## 本題

- それぞれのファイルにバージョン情報が入っているなら、一元化してしまえばいいじゃん、というのがきっかけです
   - もちろん、Chrome拡張機能のバージョンと、npmパッケージとしてのバージョンは別々にしたいという場合は、それぞれ管理でいいです

### 手順

package.json にはこれまで通り、バージョン情報を記載します。

```JSON:package.json
  "version": "1.1.5",
```

manifest.json には、以下のように`{{VERSION}}` (何でもいいのですが) というプレイスホルダを記載します。

```JSON:src/manifest.json
   "version": "{{VERSION}}",
```

そして、Gulpでビルド・パッケージングを行う際に、以下のようにパラメータの置換を定義しておけばokです。

```JavaScript:gulpfile.js
var gulp = require('gulp');
var pkg = require('./package.json');

gulp.task('config', function() {
   gulp.src('./src/**/*.json')
      .pipe(replace('{{VERSION}}', pkg.version))
      .pipe(gulp.dest('./dist'));
   console.log('config file copied.');
});
```

