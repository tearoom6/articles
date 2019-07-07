# 事象

HTTPS の Web サイトを切り替えのため、一時的に HTTPS でのアクセスを遮断。

いったん HTTP でのアクセスを確認した後、 HTTPS でアクセスできるか確かめようとしたところ、 Chrome から見たときに HTTP でアクセスしようとしても、すぐに HTTPS にリダイレクト(?)される事象が発生。

なお、サーバ側では HTTP から HTTPS へのリダイレクトは設定していない。


# 原因

原因は HTTP Strict Transport Security (HSTS) という仕様によるものだった。

一度でも HTTPS でアクセスして、その時の Response header に `Strict-Transport-Security` を以下のように付けて返した場合、対象サイトへのアクセスは以降、必ず HTTPS で行われるようになる。 (HTTP で行えなくなる)

```
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

ユーザの預かり知らぬうちに不意に HTTP に切り替わっていて、重要な情報が盗み見されることを防ぐのに有効な仕様で、もはや時代は HTTPS 必須化なので、あるべき仕様と言えます。


# TIPS

少なくとも Chrome の場合は、 HSTS の対象となったドメインを内部的に記録している。 (仕様的にはそうでないと実現できない)

もし手動でこの情報を編集したい場合は、以下のページから行う。

chrome://net-internals/#hsts

[Delete domain security policies] を実行することで、この制約を解除して、 HTTP でアクセス可能になる。(それでも消えない場合は、ブラウザ閲覧履歴を消す)

なお、 HSTS preload list という RFC 6797 には定義されていない仕様もあり、このリストに入ったサイトは、上記の [Delete domain security policies] でも制約を削除できない。
HSTS preload list は予め、サイトのドメインを登録フォームから登録しておくことで、チェックを通ったものは、 Chrome (および、その他のモダンブラウザ) にハードコードで HSTS の対象として追加され、数カ月後にリリースされたバージョンから反映されるようになる、というものである。

> References

- [RFC 6797 - HTTP Strict Transport Security (HSTS)](https://tools.ietf.org/html/rfc6797)
- [Strict-Transport-Security - HTTP | MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Strict-Transport-Security)
- [httpでアクセスしたいのにhttpsへリダイレクトされて悩んだ（HSTSまたはStrict-Transport-Securityの沼） - Qiita](https://qiita.com/tmasu/items/14632bf45cf05b90d0c9)
- [Chrome: how to stop redirect from http:// to https:// - Super User](https://superuser.com/questions/565409/)
- [HSTS Preload List Submission](https://hstspreload.org/)
- [Can I use... Support tables for HTML5, CSS3, etc](https://caniuse.com/#feat=stricttransportsecurity)
