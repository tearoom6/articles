この記事は [UMITRON Advent Calendar 2021](https://adventar.org/calendars/6652) **6日目** の記事です。

朝起きるまでは断固として **6日目** です🌅

---

お魚食べてますか?

改めまして tearoom6 です。
現在は UMITRON 株式会社で、養殖の海洋生物 🐟 の持つ潜在的可能性を IT を使って引き出すことに取り組んでいます。

さて、日々開発を行っていると、どうしても優先度付けで後回しになるタスクはあるかと思います。ユーザに直接価値を提供しないけど、細かいところで不便を感じる、あったら便利なんだけどなぁ、、という類のものですね。ほんとにすぐできるものはさっと作っちゃうほうが良いかと思いますけど、実際にほんとにすぐできちゃうものは実はそんなに多くないかと思います。

そういう場合、まずは既に世の中にあるもので使えそうなものがないか探しますが、ぱっと探して、良いものが無かった場合は、自分で作るか、と思いつつ、塩漬けになりがちです。

ただ、こういう開発支援的なタスクは、制約が強くなくて、比較的自由にできるので、技術的ノウハウが溜まりやすいと感じているのもあって、できるなら並行して進めたい気持ちがあります。 (何よりもエンジニアは何でも自動化したがるのが習性ですし・・・)

そこで、表題の "Advent calendar driven development" です。
Advent calendar で宣言して、その日 (**6日目** です) までに必ずやるぞとプレッシャーを与えることで、その手のタスクをこなそうとするものです。

## 本題 - 実現したい機能

前置き長くなりましたが、以下、本題です。

私の関わっているプロダクトでは、画面を見ていて UTC で記録している時刻が、現地のタイムゾーンでは何時になるのだろう、というのを知りたい場面に出くわすことが多くありました。

もちろん画面上で表示するよう実装しちゃってもいいのですが、もし、ブラウザの拡張機能で実現することができるなら、再利用性もあり、アプリケーションコードもシンプルに保てるので、実現したい機能に対して親和性が高いのではと思いました。

まず、既存の拡張機能を探すところから始めたのですが、公開されている Chrome の拡張機能だと、以下の 2 系統が多く、目的のものがなかなか見つかりませんでした。

- 使用しているブラウザの timezone の設定を手軽に変更する
- 現在時刻を複数 timezone の時刻で表示 (世界時計的な)

一番近かったのが [Utime - Chrome Web Store](https://chrome.google.com/webstore/detail/utime/kpcibgnngaaabebmcabmkocdokepdaki) という拡張機能でした。

[billdami/utime: A Google Chrome extension that converts UNIX timestamps to dates (and vice versa)](https://github.com/billdami/utime)

この拡張機能は、テキストを選択した状態で context menu を押すと、そのテキストが示す日時を、事前に設定したタイムゾーンでの日時に変換した上で、 Notification で表示する、という機能などを提供していました。

![chrome_extension_utime_notification.png](https://files.tearoom6.biz/f04e0507-6ca3-4288-a490-f2f14a444d7d.png)

しかし、実現したいのは、選択したときに変換対象のタイムゾーンを指定したい、もしくは、複数タイムゾーンでの時刻を表示したい、、という感じだったので、惜しかったのですが、ちょっと違う感じでした。なので、チャンスがあれば自作しようかなと思っていました。

## 前提条件

実装前の状況は、以下のような感じです。

- Chrome の拡張機能での開発経験はあり
- どのような API を使えば、どのような機能が実現できる、というような知識・調査を行わない状態で、とりあえず実現したい機能だけある状態で開始
- [Content scripts](https://developer.chrome.com/docs/extensions/mv3/content_scripts/) を使えば、割とどんなことでもできるイメージはあったが、この機能はどんなウェブサイトでも使えるようにしたかったため [Context Menus](https://developer.chrome.com/docs/extensions/reference/contextMenus/) を使うことにした
- 当初のイメージでは、以下のどちらかで実現できれば非常に嬉しいなという期待を持っていたが、他の拡張機能を使っている感覚からすると、そういう機能は実現できないのかなという気もしていた
  - 対象の時刻文字列を選択した状態でコンテキストメニューを開けば、コンテキストメニュー内に、各タイムゾーンでの時刻が表示される
  - 対象の時刻文字列を選択した状態でコンテキストメニューを選択すれば、拡張機能の popup が表示され、その中で各タイムゾーンでの時刻が表示される

なお、今回は、普段遣いの Chrome のみを対象にすることにしました。

私は作ったことはないですが、他の主要モダンブラウザについても、拡張機能の仕組みは用意されています。

- [Extensions - Microsoft Edge Developer](https://developer.microsoft.com/en-us/microsoft-edge/extensions/)
- [Browser Extensions - Mozilla | MDN](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions)
- [Safari Web Extensions | Apple Developer Documentation](https://developer.apple.com/documentation/safariservices/safari_web_extensions)

ただ、 API 標準化は現時点では道半ばで、互換性のないところは相当あると思われます。 (つまり、なかなか同一コードでは書くのは難しそうという感じがします)

- [About the WebExtensions API | Firefox Extension Workshop](https://extensionworkshop.com/documentation/develop/about-the-webextensions-api/)
- [Chrome incompatibilities - Mozilla | MDN](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Chrome_incompatibilities)
- [Porting a Google Chrome extension | Firefox Extension Workshop](https://extensionworkshop.com/documentation/develop/porting-a-google-chrome-extension/)

## 実装方針

全体としては、以下の方針を持っていました。

- TypeScript で実装しておく
- 画面が必要なら React を用いる (結果的には画面は作っていない)

また、時間のない中作るので、とりあえず荒削りでも OK ということにしました。

また、 MV3 (Manifest V3 for Chrome Extensions) と呼ばれる Chrome Extensions API の新しいバージョンが使えるので、チャレンジングな要素として、そちらで実装することにしました。

MV2 と MV3 は、例えば以下のような違いがあります。

- Browser Action API と Page Action API が Action API に統合された
- background pages と呼ばれていた headless で動く部分は service workers で実装しなければならない
- Remotely hosted code が実行できなくなった

[Chrome Extension Tutorial: Migrating to Manifest V3 from V2](https://blog.shahednasser.com/chrome-extension-tutorial-migrating-to-manifest-v3-from-v2/)

機能的に向上したというよりは、セキュリティやパフォーマンス面に配慮したアップデートが中心な気がします。ただ、今 MV2 で動いている拡張機能は 2022 年中の MV3 へのアップデートが求められているので、拡張機能開発者は対応する必要があります。

[Manifest V2 support timeline - Chrome Developers](https://developer.chrome.com/docs/extensions/mv3/mv2-sunset/)

※ 余談ですが、一昔前は Chrome Web Store への拡張機能の公開は、割とハードルが低かったのですが、今はレビューも厳格で、結構厳しくなっている気がしました。まぁ、拡張機能って結構セキュリティ的には厳格にしないと危ないよね、っていう気はしてましたが。。

## 実装結果と知見

こちらがとりあえず作ったコードです。

https://github.com/tearoom6/TimezoneTraveler

![chrome_extension_timezone_traveler_usage.png](https://files.tearoom6.biz/d7852806-38da-44a2-bd47-c6be8001a163.png)

結果的に出来上がったものは、 "対象の時刻文字列を選択した状態でコンテキストメニューを選択すれば、再度コンテキストメニューを開いた際に、各タイムゾーンでの時刻が表示される" というもので、これだと、目的の各タイムゾーンでの日時を表示するのに 2 クリック (+ カーソルの移動 = これはショートカットキーを設定すれば、回避できるかも) を要するので、使い勝手としてはかなり悪く、さらなる改良は必要そうです。

ブラウザの拡張機能は、セキュリティ的な要件から制約が強いので、どうしても実際に作りながら、実現できそうなラインを探っていく感じになっちゃいますね。。
それも数をこなせば、大体勘所は掴めてくるんでしょうけど・・・。

実装上の苦労したポイントは以下のとおりです。

### service worker に起因する問題

service worker は必要なときにだけロードされて、動作するという特性を持つため、いくつかのルールを守って実装する必要があります。

- event listeners の登録は top level で行う (非同期 callback の中などで行わない)
- global variable の状態は、すぐにリセットされてしまう可能性があるため、必要なものは storage に永続化を行う
- `setTimeout`, `setInterval` の代わりに [Alarms API](https://developer.chrome.com/docs/extensions/reference/alarms/) を top level で用いる

### `chrome.contextMenus`

こちらも割とクセのある API でした。

- context menu item は親子関係にしてネストすることができるが、親にできるのは `ItemType` が `normal` のもののみ
  - `checkbox`, `radio`, `separator` は親にすることができない
- 親になった menu item へのクリックは `OnClick` event で捕捉できない
- 同一 `id` の menu item は登録できない
- 存在しない `parentId` を指定した menu item は登録できない
- テキスト選択時の context menu に item を表示したい場合は、全ての対象 item の `contexts` に `all` or `selection` を含める必要がある
  - 親 item で指定していたからといって、子 item で指定していなければ、子 item の方は表示されない

### `browserAction.openPopup()`

当初の野望にあった、コンテキストメニュー押下で、拡張機能の popup を表示、という機能が実現できなかった点です。

Mozilla の出している WebExtensions API のドキュメントを見てみると `browser.browserAction.openPopup()` というのが実はあるので、 Chrome でも popup をプログラムから開けるのではないかという期待が生まれたのですが、どうも Security の問題から Chrome では `chrome.browserAction.openPopup()` を一般公開はしていないようです。詳細は把握していないですが、 Security 周りはかなり泥臭い実装が行われているのかもしれないです。

https://chromium.googlesource.com/chromium/src/+/0ab916c3ef2fd0674d179e2c29b3e6b937231138/chrome/common/extensions/api/_api_features.json#167

ただし、拡張機能の画面を popup ではなく、新規タブとして開くことは (Permission が許せば) 可能なようなので、それを使えば、もっと使い勝手を良くすることができるかもしれないです。

[Question How to programmatically open a chrome extension popup window from background.html](https://www.titanwolf.org/Network/q/bc51ed2d-e9db-4698-b476-2f8e34519620/y)

> References

- [TypeScriptで作るイマドキChrome拡張機能開発入門 - Qiita](https://qiita.com/markey/items/ea9ed18a1a243b39e06e)

---

(追記)
レビュー通って Chrome Web Store からインストール可能になりました。
もしご興味持った方がいれば改善にご協力お願いします🙏

https://chrome.google.com/webstore/detail/timezone-traveler/gndkkoonfiibdihdaklkhfiikfkbhdik

---

以上、6日目が誕生日の tearoom6 の記事でした 🎉
