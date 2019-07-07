# Content-Security-Policy (CSP)

CSP は Cross Site Scripting (XSS) や data injection 攻撃を防ぐための HTTP の仕様です。

CSP を有効にするには、以下のいずれかを実施します。

- HTTP header で `Content-Security-Policy` を返す

   ```http
   Content-Security-Policy: default-src 'self'; img-src *; media-src media1.com media2.com; script-src userscripts.example.com
   ```

- `meta` タグで `Content-Security-Policy` を埋め込む

   ```html
   <meta http-equiv="Content-Security-Policy" content="default-src 'self'; img-src https://*; child-src 'none';">
   ```


## CSP directives

CSP の policy は、上記のように directive に続けて、空白区切りで値リストの記述を `;` 区切りで複数記載していきます。
directive は Content の種類によって、以下の種類があります。

- `default-src`
- `child-src`
- `connect-src`
- `font-src`
- `frame-src`
- `img-src`
- `manifest-src`
- `media-src`
- `object-src`
- `prefetch-src`
- `script-src`
- `style-src`
- `worker-src`

`default-src` は特殊な directive で、対象のコンテンツの directive が定義されていなかった場合に、この値が使われます。


## CSP による制約

CSP が有効な場合、 HTML 本体以外のリソースをロード・実行しようとしたときに制約が生じます。

例えば、 `img-src https://*;` のように設定すれば、 HTTPS 以外の画像の読み込みを制限することができます。


## script-src unsafe-inline

`script-src` に `'unsafe-inline'` を指定することで、 inline scripts と inline event handlers の実行を許可できますが、モダンブラウザではこのオプションは推奨されません。

代わりに nonce もしくは hash を用います。


## script-src nonce

`nonce` は inline scripts と inline event handlers に対して、 whitelist 形式で CSP の許可を与える方法です。

まず実行を許可したい inline script の `nonce` 属性に、 BASE64 エンコードしたテキストを埋め込みます。

```html
<script nonce="2726c7f26c">
  var inline = 1;
</script>
```

上記スクリプトの実行を許可するには、 CSP の script-src の policy に、以下のように `'nonce-BASE64_VALUE'` の値を埋め込みます。

```http
Content-Security-Policy: script-src 'nonce-2726c7f26c'
```

nonce はランダムな値を使えば OK ですから、生成は簡単です。たとえば、 PHP だと以下のように生成できます。

```php
> echo base64_encode(random_bytes(20));
fUz9xyElzZh7x4XwbKkt2OdAhPs=
```


## script-src hash

inline scripts の実行を許可する方法は、他にもあります。
スクリプト本体 (空白・改行を含む / `<script>`タグは除く) 全体のハッシュ値 (sha256 or sha384 or sha512) を求めて、それに対して whitelist 形式で許可するやり方です。

例えば、以下の inline script の実行を許可したいとします。

```html
<script>var inline = 1;</script>
```

まず、このスクリプト本体のハッシュ値 (BASE64 エンコードしたもの) を求めます。今回は Ruby でやってみます。

```ruby
> require 'digest'
> Digest::SHA256.base64digest 'var inline = 1;'
=> "B2yPHKaXnvFWtRChIbabYmUBFZdVfKKXHbWtWidDVF8="
```

そして、 CSP の script-src の policy で以下のように、ハッシュアルゴリズム名に続けて、生成したハッシュを埋め込めば OK です。

```http
Content-Security-Policy: script-src 'sha256-B2yPHKaXnvFWtRChIbabYmUBFZdVfKKXHbWtWidDVF8='
```

> References

- [Content Security Policy (CSP) - HTTP | MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
- [CSP: script-src - HTTP | MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy/script-src)
- [Content-Security-Policy - HTTP | MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy)
- [Content Security Policy (CSP) - Google Chrome](https://developer.chrome.com/extensions/contentSecurityPolicy)
- [CSP Evaluator](https://csp-evaluator.withgoogle.com/)
- [CSP with Kirby - how to best generate a nonce? - Questions - Kirby Forum](https://forum.getkirby.com/t/csp-with-kirby-how-to-best-generate-a-nonce/8010/3)
- [CSP (Content Security Policy) nonce-sourceについて調べてみた — A Day in Serenity (Reloaded) — PHP, FuelPHP, Linux or something](http://blog.a-way-out.net/blog/2014/10/28/csp-nonce-source/)
