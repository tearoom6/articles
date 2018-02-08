
Google Drive REST API が version3 にアップグレードされていたので、使用してみた。

## v2 から変わった点

### fields

- 開発者側から見て一番大きいのが、v2 だと取れうる情報を全部返してくれてたのが、v3 だと必要最低限の情報しかデフォルトでは返してくれなくなった点
- Google Drive のメタ情報が膨れた結果、これは全部の情報を返してしまったのではレスポンスが大きくなりすぎちゃったということなんだろう
- では、その他の情報はどう取得するのかというと、リクエストの中でこの情報を欲しいと指定するようにする
- `fields`というリクエストパラメータが追加されていて、これに以下のような感じで返してほしいレスポンスパラメータ名を指定する
   - パラメータは階層構造になっているので、それは`()`を使って表現する

`fields=files(iconLink,id,kind,mimeType,name,viewedByMeTime,webContentLink,webViewLink),nextPageToken`

### パラメータ名変更

- 俺の使った範囲でもいくつかのパラメータの名前が変わったり、なくなったりしている
- ただ、代わりのものがあるので、これは置き換えていくだけで済むから影響は小さい

### その他

- 他にもあるみたいだけど、詳しくは下記の Official ガイドをみて欲しい

## 参考

Drive REST API v3 Reference
<https://developers.google.com/drive/v3/reference/>

Migrate from v2
<https://developers.google.com/drive/v3/web/migration>

Google APIs Explorer
<https://developers.google.com/apis-explorer/#s/>

## 所感

- 俺の場合はもともと使っていた範囲が狭かったので、移行は大した手間じゃなかった
- けど、がっつり使ってた人にとっては、まぁまぁ大変かもしれないなと思う

## Google Drive API TIPS

ついでなので、API使う上でのTIPSも。

### ページング

- Webアプリで言うところのページングと同じで、API経由でも List 系のメソッドに対しては、アイテム数が多い場合、1回のリクエストに対しては部分部分しか返さないようになっている
- offset と limit を指定するようになっているインタフェースはよく見るけど、Drive API はちょっと違う
- 1回で全てを返せなかった場合、レスポンスパラメータに `nextPageToken` というのが含まれていて、この値を次のリクエストの `pageToken` パラメータとして指定することで続きが取れるようになっている
- `nextPageToken` が含まれていなかったらそれで全部取れたということになる
- この仕組みは v2 でも v3 でも変わってません
- なので、一度に全部取るには、次のようにすればいいです

```JavaScript
function listAllFiles(reqParams, callback) {
  var retrievePageOfFiles = function(request, result) {
    request.execute(function(resp) {
      result = result.concat(resp.files);
      var nextPageToken = resp.nextPageToken;
      if (nextPageToken) {
      	// sum up all items if response has multi-pages.
      	var nextParams = reqParams;
      	nextParams.pageToken = nextPageToken;
        request = gapi.client.drive.files.list(nextParams);
        retrievePageOfFiles(request, result);
      } else {
      	// at last, callback would be called.
        callback(result);
      }
    });
  }
  // initial request.
  var initialRequest = gapi.client.drive.files.list(reqParams);
  retrievePageOfFiles(initialRequest, []);
}
```

### Try it!

- 上述した [Reference](https://developers.google.com/drive/v3/reference/) のページには、各メソッド詳細ページの一番最後に "Try it!" という項目があって、ブラウザ上で実際の自分のアカウントを用いて、簡単にAPIのテストができるようになっています！便利！
- なお、何度も呼び出してテストしたい場合は、[Advanced REST Client](https://chrome.google.com/webstore/detail/advanced-rest-client/hgmloofddffdnphfgcellkdfbfbjeloo?utm_source=chrome-ntp-icon) というChrome拡張が便利で、よく使っています
   - 他でも使えるし


## 最後に

このAPIを使ったChrome拡張を以下のURLで公開してます。
<https://chrome.google.com/webstore/detail/quick-drive/aijfbconiilhjgfljolkoiaockgenpgn?utm_source=chrome-ntp-icon>

ソースコードはこちら
<https://github.com/tearoom6/QuickDrive>
