海外行くことが多い人や、新しい物好きの人には当然の話かもしれないんですが、新しいハードウェアやサービスには滅法コンサバ気味の自分にとっては、ちょっとはまった話です。。

## 状況

まず、国内でプリペイド SIM を購入、会社の海外 (ニュージーランド) 出張に向かいました。
購入した SIM は以下の通り。

[Three.co.uk Trio SIM: ヨーロッパ・オセアニア周遊SIM 4Ｇ・3Ｇデータ通信3ＧＢ](https://www.amazon.co.jp/gp/product/B07FLHCVSP)

また、スマホは、元々 au が出しているキャリア販売端末で、その後、同じく au 系列 MVMO の UQ Mobile の SIM に乗り換えた、以下の Android 9 端末になります。

[Samsung Galaxy S9 SCV38](https://www.au.com/english/mobile/product/smartphone/scv38/)

当然、乗り換え時に、端末の "SIMロック解除" は実施済みです。
また、同じく乗り換え時に APN (Access Point Names) の設定は行っていましたので、今回も同じくそこを設定すればいいんだろ、くらいに考えていました。

[ネットワーク設定方法│格安スマホ/SIMはUQ mobile（モバイル）｜【公式】UQコミュニケーションズ](https://www.uqwimax.jp/mobile/support/guide/apn/)

そして、 Auckland の空港につくやいなや、 SIM card を入れ替え、端末再起動して、 SIM の説明書きにある通り、 APN の設定を行うも、繋がらない。
そもそも、 APN を保存しようとした時に MCC や MNC という項目が未入力だといって怒られる。適当な値を入れて、無理やり保存しても、やっぱり繋がらない。
しかも APN の一覧に、以前入力したはずの UQ Mobile のやつ (uqmobile.jp) も表示されていない。

そして、よく見ると Notification に "Invalid SIM card. (Network locked SIM card inserted.)" みたいな表示が出ていました。

仕方がないので、念の為、持ってきていたポケット Wi-Fi ルータに SIM を差し込むと、特に何の設定も行うことなく、使用可能でした。一安心したものの、腑に落ちない。
エンジニアとしては、これは乗り越えないといけない壁です。出張中に解決しようと固く心に決めました。 (そもそも自分がしょぼいだけなのですが...まぁこういうのを解決していって人は成長するのです)

## 解決

最終的に原因だったのが、 "SIM カードステータスの更新" というのを行わないといけないということでした。

Settings アプリから [About phone] > [Status] > [SIM lock status] を開き、 "Not allowed™ となっている場合は、画面下部の [Update SIM card status] ボタンを押す必要があるとのことでした。

これをした時点で、 Notification に出ていた "Invalid SIM card." の表示は消えます。

その上で SIM カードの説明書にある通り、 Settings アプリで、以下の設定を行うことで、モバイル通信が使えるようになります。

- [Connections] > [Mobile networks] > [Roaming Settings] > [Data roaming] を ON にする
- [Connections] > [Mobile networks] > [Access Point Names] で [Add] > [APN] を押して、以下を入力した上で [Save]
   - Name: 3
   - APN: three.co.uk
   - Authentication type: CHAP
   - その他の項目はデフォルトのまま
- [Connections] > [Data usage] > [Mobile data] が ON になっていることや、機内モードが OFF になっていることを確認

"SIM カードステータスの更新" を行った後だと APN の追加時 MCC, MNC は、なんとデフォルト値が自動で入力されていました。 (234, 20 になっていました)
ちなみに、 UQMobile の方の MCC, MNC はそれぞれ 440, 51 になっていました。

## まとめ

今回、勉強になったこと。

- 少なくとも Android 端末では SIM を入れ替えた場合に "SIM カードステータスの更新" を必要に応じて行わなければならない
- APN のリストは SIM card ごとに別々で、同時には表示されない
- 国別、キャリア別の MCC, MNC という設定が APN にはあるが、通常は SIM を挿した時点で設定不要

以上。

P.S. これ、おそらく au ネットワーク使ってる UQ mobile に乗り換えたから気が付かなかっただけで、他のキャリア網に乗り換えた人なら当然知ってる話なんだろうな。

> References

- [SIMロック解除済みでも現地SIMを認識しない？ - ケータイ Watch](https://k-tai.watch.impress.co.jp/docs/column/minna/1111200.html)
- [au端末は海外SIMを刺して使えるか。 - ヨーグルト・パイン](https://www.slow-bird.com/entry/2018/04/12/070000)
- [11 Proven Solutions To Fix "Invalid SIM Card" Error On Android](https://www.androiddata-recovery.com/blog/fix-invalid-sim-card-android)
- [SIM Unlocking Procedures | Smartphone or mobile phone users | au](https://www.au.com/english/support/contract/simcard/unlock/)
- [MCC MNC list | CellIdFinder](https://cellidfinder.com/mcc-mnc)
