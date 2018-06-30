ブラウザの "Hard Reload" という機能のお話。

Chrome の場合は、以下のいずれかの手順で "Hard Reload" が可能です。

- Developer Tools を開いた上で、更新ボタン長押し
- shift を押しながら更新ボタンクリック
- ショートカットコマンド: command + shift + r (macOS)

![chrome_refresh.png](https://files.tearoom6.biz/e04a11c9-070f-469c-8a0e-20917ab2b076.png)

この "Hard Reload" が通常の "Soft Reload" と何が違うのでしょう?

まず、 "Soft Reload" の場合は、以下の HTTP Caching の仕組み (HTTP Caching の仕組みはもっとパターンが有り、複雑ですが、ざっとしたイメージとして) に従います。

1. HTTP の仕組みで、サーバ側がレスポンスヘッダ中で `cache-control` や `expires` などを使ってレスポンスをキャッシュするよう指示することがあります
2. この場合、ブラウザはレスポンスボディをキャッシュします
3. ブラウザの更新ボタンが押されると、再度リクエストを投げますが、リクエストヘッダ中の "cache-control" や "if-modified-since" などでキャッシュがあることを伝えます
4. すると、サーバ側は 304 Not Modified を返して、前回のキャッシュを使用するよう指示することがあります (レスポンスボディは返りません)

"Hard Reload" の場合は、上の 3 のところで、キャッシュがあることを伝えずにリクエストするので、サーバは新規のリクエストとして扱い、 200 OK とともにレスポンスボディを返します。

通常、 304 Not Modified を返すパターンは、静的リソース等でサーバ側から返される内容は変化ない場合が多いのですが、開発環境等で頻繁に静的ファイルを書き換えている場合などで "Hard Reload" が役に立つことがあります。

他のブラウザにも同様の機能があるようなので、他をお使いの場合はチェックしてみてください。

> References

- [Google Chrome: hard reload vs. normal reload - gHacks Tech News](https://www.ghacks.net/2018/01/24/google-chrome-hard-reload-vs-normal-reload/)
- [HTTP caching - HTTP | MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Caching)
- [Tech tip: How to do hard refresh in Chrome, Firefox and IE?](https://www.getfilecloud.com/blog/2015/03/tech-tip-how-to-do-hard-refresh-in-browsers/#.WwQldVOFMkp)
