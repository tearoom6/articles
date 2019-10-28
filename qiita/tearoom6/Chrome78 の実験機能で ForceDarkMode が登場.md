# Chrome78 の実験機能で ForceDarkMode が登場

Chrome は以前から Experimental features を設定で試してみることができるようになっています。
将来的に実装されるかもしれない機能をひと足早く試してみることができます。

Chrome 78 では、ちょっと面白い機能がこれらに追加されました。


## Force Dark Mode for Web Contents

Chrome のアドレスバーで `chrome://flags/#enable-force-dark` を入力すると、設定項目が出てきます。
設定変更したら、 Chrome を再起動して、設定反映させます。

Chrome 73 で macOS の Dark mode 設定と連動するように、 Chrome の Dark mode が導入されましたが、これはあくまで Web contents 以外の部分に関しての見た目の変更でした。

続いて Chrome 76 では、 CSS の media query `prefers-color-scheme` に対応するようになりました。これによって、ユーザが Dark mode 設定をしているかどうかを Web site 側も判定できるようになり、それに合わせて、サイトの外観をマニュアル実装で変更できる状態になっていました。

そして、今回 Chrome 78 で Experiments として登場した機能は、 Web site 側でマニュアル実装していなくても、 Dark mode にしたときに、自動でサイトの外観を変更してしまおうという、結構アグレッシブなものです。

以下のように、結構いくつかの方式を選ぶことができます。

![chrome78_force_dark_mode_options.png](https://files.tearoom6.biz/cc750a0e-b6bc-4a49-ba1f-482437bfe180.png)

試しに、 `Enabled` に設定して、 google.com を開いてみました。う、うん...

![chrome78_force_dark_mode_google_screenshot.png](https://files.tearoom6.biz/5a2d7bbc-a24e-4e57-bd92-ac1c87855fd9.png)

きっと実際の機能としてリリースされる場合には、いい感じになっていることでしょう。


## Password Leak Detection

Chrome のアドレスバーで `chrome://flags/#password-leak-detection` を入力すると、設定項目が出てきます。
設定変更したら、 Chrome を再起動して、設定反映させます。

この機能を `Enabled` にすると、ユーザ名やパスワードの "Data breach" によって流出した、秘匿情報のデータベースとの照合が安全に行われ、もし一致した場合には、警告が表示されるとのことです。

> References

- [How to Enable Chrome 78's Hidden Dark Mode and Secure Password Features](https://lifehacker.com/1839357304)
- [How to Get Access to Experimental Features in Chrome (and on Chromebooks)](https://www.howtogeek.com/179070/how-to-get-access-to-experimental-features-on-your-chromebook-or-just-in-chrome/)
- [CSSだけでWebページをダークモード対応。ついにChromeにも対応した prefers-color-scheme が超便利｜平田 / U-NEXT｜note](https://note.mu/psephopaiktes/n/n878424784a1b)
- [How to enable password leak detection in Google Chrome • Pureinfotech](https://pureinfotech.com/enable-password-leak-detection-google-chrome/)
- [Chrome OS 78 bringing Password Leak Detection, similar to “Have I been pwned?” – About Chromebooks](https://www.aboutchromebooks.com/news/chrome-os-78-bringing-password-leak-detection-similar-to-have-i-been-pwned/)
- [What is a data breach?](https://us.norton.com/internetsecurity-privacy-data-breaches-what-you-need-to-know.html)
