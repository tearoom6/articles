TLS 関連の Chrome の TIPS を 2 つほど紹介します。 (macOS で確認しています。 Windows など他 OS では挙動未確認)

## オレオレ証明書を一時的に信用する

正式な認証局の署名が入っていない、オレオレ証明書を使ったページに HTTPS でアクセスした場合、ブラウザがアクセスを拒否してきます。

証明書の期限切れなどの場合には、 Chrome では `Advanced` menu に `Proceed to xxx (unsafe)` link が出現するので、これを押せば進めます。

![chrome_tls_error_page_001.png](https://files.tearoom6.biz/fa7365e1-75ba-49a2-b310-bf763fdc4d9f.png)

しかし、不正な証明書などの場合で、これが出てこない場合があります。

![chrome_tls_error_page_002.png](https://files.tearoom6.biz/bb818339-8059-4914-bba3-85fab300d442.png)

(https://badssl.com/ っていうサイトで、いろいろな証明書の不具合のサンプルが置かれているので、それぞれどのような表示になるのか確かめてみるといいと思います)

このような場合 Chrome で対象のページを開いて、フォーカスがあたっている状態で、 `thisisunsafe` (現在最新の Chrome 83 の場合) とタイプしてください。
すると、あら不思議、ブロックが解除されて、ページに進めるようになります。

また、逆に、この bypass keyword による効果を解除したい場合は、ブラウザ URL 欄の "Not Secure" リンクをタップして出てくるダイアログ中の `Re-enable warnings` リンクをクリックすることで、この一時的な信用状態をリセットできます。

![chrome_site_information_dialog_001.png](https://files.tearoom6.biz/d27e71bf-c920-45e6-a189-602b38f10588.png)

なお、 masOS の場合だと Keychain Access にオレオレ証明書を登録する方法のほうが、よりよいのではないかと思います。その方法の解説は [開発環境の自己証明書をブラウザに信頼させる | スターフィールド株式会社](https://sterfield.co.jp/programmer/%E9%96%8B%E7%99%BA%E7%92%B0%E5%A2%83%E3%81%AE%E8%87%AA%E5%B7%B1%E8%A8%BC%E6%98%8E%E6%9B%B8%E3%82%92%E3%83%96%E3%83%A9%E3%82%A6%E3%82%B6%E3%81%AB%E4%BF%A1%E9%A0%BC%E3%81%95%E3%81%9B%E3%82%8B/) に譲ります。

ただし、その場合は、絶対に信用できない人にオレオレ証明書をバラまかないように注意してくださいねｗ

> References

- [ssl - Getting Chrome to accept self-signed localhost certificate - Stack Overflow](https://stackoverflow.com/questions/7580508/)
- [security - Does using 'badidea' or 'thisisunsafe' to bypass a Chrome certificate/HSTS error only apply for the current site? - Stack Overflow](https://stackoverflow.com/questions/35274659/)


## 現在アクセスしているページの証明書をローカルに保存する

Chrome で現在アクセスしている URL の証明書をローカルにダウンロードしたい場合、ブラウザ URL 欄の鍵マークをタップして出てくるダイアログで、 `Certificate` を選択します。

![chrome_site_information_dialog_002.png](https://files.tearoom6.biz/0b842f9c-dcb9-46d9-a1de-7e552b3a7aee.png)

すると、以下のようなダイアログが開くので、ここの証明書画像を macOS の場合 Finder (デスクトップとかでもいいです) に drag & drop してください。

![chrome_tls_certificate_dialog.png](https://files.tearoom6.biz/246f47fc-aa00-4c68-9dc1-41b794770202.png)

すると、あら不思議、証明書ファイルが、該当のディレクトリにダウンロードされます。

ちょっと操作方法がマニアックで他にも方法ありそうな気がするので、あったらぜひ教えてほしいです...!

> References

- [How to save a remote server SSL certificate locally as a file - Super User](https://superuser.com/questions/97201/)
