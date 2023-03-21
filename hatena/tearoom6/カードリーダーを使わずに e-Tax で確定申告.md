(2023-03 追記)

今年の確定申告は macOS (Intel CPU) の Safari 16.3 (と Android Pixel 5a) で完結できました。
OS の言語設定も、日本語に変更しなくてもいけました。

公共のシステムは、 Windows PC で、ブラウザは IE で、言語設定を日本語に変えて... っていうのが定番だったんですが、ここ 2-3 年で急速に改善されている感がありますね！

---

表題のとおりです。時期遅れは否めない。

今年は住宅ローン減税の恩恵を受けるため、何が何でも確定申告しなければならなかったんですが、ずぼらにずぼらを重ねた結果、コロナで延びた申告期限である 4/15 に滑り込みセーフという危ない橋を渡りました。 Twitter 見てたら、他にもそういう人多そうではあったけど。

激混み税務署には行きたくなかったので (コロナで延ばした意味ない...)、コロナの国民給付金のときにも便利だった、スマホがマイカードのカードリーダー代わりに使えるというのがいけるらしいというので、 e-Tax で確定申告してみることにしました。 (カードリーダー購入したら今までも e-Tax できたんですが、間違いなくこれでしか使わないカードリーダーをわざわざ買うのかといえば。。 ID/PW 方式もあるみたいだけど、ちゃんと調べてない)

**結論からいえば、できた**んですが、謎技術のオンパレードだし、罠多すぎという感じでした。

(備忘録として書いてますが、もう終わった後にこれ書いても参考にならんかも。来年には多くが改善されてそうだし)

## 前提

私の場合、妻が昔始めた趣味のネットショップをやってる背景で、個人事業主登録していて、青色申告しないといけないので、そこのハードルがありました。
(青色申告するほどの事業でもないんですが......)

https://ninnyhammer.buyshop.jp/

**いわゆる事業所得(青色・白色)の申告をする場合は、スマホだけでは e-Tax では完結しません。**

**青色・白色申告しない場合で、雑所得や各種控除等を含めて所得税の申請するだけであれば、 PC も要らなくて、スマホだけで申告完結するそうです。**
(出産などに伴う医療費控除や、初年度の住宅ローン控除を受ける場合は、会社員も確定申告必要ですが、ボリュームゾーン的にそういうケースを最優先しているんですかね)
(こちらは Android, iPhone ともに対応していて、 "マイナポータルAP" アプリだけで完結するとのこと)

**以下で書くのは、事業所得の申告を e-Tax で行う場合の話です。**

なお、面倒な場合は、税務署で書面で申告しても良いのですが、その場合は、今年から青色申告の控除額が 55 万円になってしまいました。引き続き 65 万円の控除を受けるためには、 e-Tax を行う必要があるので、ご注意を。税務署は 17 時に対面は閉まっちゃうけど、 e-Tax の場合は、申告期間の 23:59 まで受け付けてくれるというメリットもあります。

なお **MoneyForward や freee のスマホアプリからも有料プランユーザなら電子申告できるらしい**です。

## 準備するもの

- マイナンバーカード
- マイナンバーカード関連の暗証番号
- Windows PC (Bluetooth 搭載の Windows 8.1, Windows 10 対応です。家にあったのが古い Windows 8.1 PC だったけど、なんとかできた) (macOS 未対応です)
- Android スマホ (PC と接続するカードリーダーとして利用する機能は iPhone 未対応です)
- 利用者識別番号 (電子申告・納税開始届出によって取得します。これも Web 上でできます。住所変更 (所轄税務署の変更) は Web でできなかったけど...)

## 事前準備

- Windows PC に [利用者クライアントソフト (Ver 3.3)](https://www.jpki.go.jp/download/win.html) (`JPKIAppli03-03.exe`) をインストールする
- Android スマホに [JPKI利用者ソフト](https://play.google.com/store/apps/details?id=jp.go.jpki.mobile.utility) をインストールする
- Android スマホに [マイナポータルAP](https://play.google.com/store/apps/details?id=jp.go.cas.mpa) もインストールしておく
- Windows PC と Android スマホを Bluetooth でペアリングする

## 手順

手順 7 から先はオプションです。住宅ローン控除を利用する場合は、添付書類を提出する必要があるため、この手順が必要になります。

1. Windows PC で Internet Explorer 11 (!?) を起動し [国税庁 確定申告書等作成コーナー](https://www.keisan.nta.go.jp/kyoutu/ky/sm/top#bsctrl) を開く
2. 申告書等の作成を開始し、 "e‐Taxで提出 マイナンバーカード方式" を選択する
3. 事前準備セットアップのチェックが入るので、まだ済んでいない場合は "事前準備セットアップファイル" (`jizen_setup.exe`) をダウンロードし、インストールしておく
4. "マイナンバーカードの読み取り" 画面が出てくるので、 Android スマホで "JPKI利用者ソフト" を起動し、 "PC接続" > "PC接続の開始" を実行した上で、マイナンバーカードをスマホに押し当てた状態にし、 PC ブラウザ上の "マイナンバーカードの読み取り" ボタンを押して、画面の指示にしたがう (以下、マイナンバーカード認証を求められたときには同様に行う)
5. "青色申告決算書・収支内訳書" の作成、 "所得税の確定申告書" の作成を順次行う
6. 申告データにマイナンバーカードで電子署名した上で、 e-Tax で送信する (この過程でも何度かスマホを使ったマイナンバー読み取りが求められる)
7. [e-Taxソフト(WEB版)](https://clientweb.e-tax.nta.go.jp/UF_WEB/WP000/FCSE00001/SE00S010SCR.do) を開く
8. 申告の際に案内された添付書類 (PDF) をアップロードして提出する

> References

- [各ソフト・コーナー | 【e-Tax】国税電子申告・納税システム(イータックス)](https://www.e-tax.nta.go.jp/software/software.htm)
- [【確定申告書等作成コーナー】-作成コーナートップ](https://www.keisan.nta.go.jp/kyoutu/ky/sm/top#bsctrl)
- [ご利用の流れ | 【e-Tax】国税電子申告・納税システム(イータックス)](https://www.e-tax.nta.go.jp/start/index.htm)
- [e-Taxの開始(変更等)届出書作成・提出コーナーについて | 【e-Tax】国税電子申告・納税システム(イータックス)](https://www.e-tax.nta.go.jp/todokedesho/index.htm)
- [e-Taxソフト(WEB版)について | 【e-Tax】国税電子申告・納税システム(イータックス)](https://www.e-tax.nta.go.jp/e-taxsoftweb/e-taxsoftweb.htm)
- [e-Taxソフト(SP版)について | 【e-Tax】国税電子申告・納税システム(イータックス)](https://www.e-tax.nta.go.jp/e-taxsoftsp/e-taxsoftsp.htm)
- [【確定申告書等作成コーナー】-事前準備セットアップファイルをダウンロードしてセットアップする方法](https://www.keisan.nta.go.jp/h28yokuaru/cat1/cat12/cat121/cid130.html)
- [【確定申告書等作成コーナー】-マイナンバーカード読取対応のスマートフォンをICカードリーダライタとして利用する方法](https://www.keisan.nta.go.jp/r2yokuaru/cat1/cat12/cat121/cid960.html)
- [【確定申告書等作成コーナー】-所得税のイメージデータの送信方法について](https://www.keisan.nta.go.jp/r2yokuaru/cat1/cat16/cat163/cid945.html)
- [一度送信した申告等データに対して追加で送信する方法(追加送信方式)｜e-Tax](https://www.e-tax.nta.go.jp/toiawase/qa/e-taxweb/34_02.htm)
- [国税庁からのお知らせ ＜スマートフォンでの申告が更に便利に＞：令和2年分 確定申告特集](https://www.nta.go.jp/taxes/shiraberu/shinkoku/tokushu/info-smartphone.htm)
- [スマートフォンｄｅマイナンバーカードアクセス！！スタートアップガイド](https://www2.jpki.go.jp/download/pdf/startupguide.pdf)
- [ICカードリーダライタのご用意 | 公的個人認証サービス ポータルサイト](https://www.jpki.go.jp/prepare/reader_writer.html)
- [利用者クライアントソフトのダウンロード | 公的個人認証サービス ポータルサイト](https://www.jpki.go.jp/download/)
- [スマホで確定申告！対象者ややり方、できる条件ってなに？ | スモビバ！](https://sumoviva.jp/article/1002130)
- [個人事業主がスマホで電子申告する方法 - スマホアプリで確定申告 | 自営百科](https://jiei.com/news/etax-way-app2021) (この記事がよくまとまっていて分かりやすい)
- [もう年末調整の住宅ローン控除で悩まない！必要書類と記入例](https://www.chibabank.co.jp/blog/housing-loan-deducation-yearend-tax-adjustment.html)
