
いつもQiitaにお世話になっているので、自分も恩返しで投稿を始めることにします！小さなことでもいいので、できるだけ毎日投稿を続けていこうと思う。

さて、初投稿は記念日的な投稿として、次のようなことをやってみたという記録をば。

- 妻の誕生日に合わせて、LINE でメッセージを送る
- 携帯を触ってないのに、俺からメッセージが送られるサプライズをしたい
- ディナーを食べ終わった 19:30 くらいに自動で送られるようにしたい

## 検討1: REST APIs

- まずは正攻法で、LINE の [REST APIs](https://developers.line.me/restful-api/overview) を使うことを検討
- 結論から言えば、これは個人向けに開放されていないので断念

### 断念理由

- 詳細を言うと、LINE REST APIs を使うには、Access Tokenの発行が不可欠なんですが、これには [Channel Registration](https://developers.line.me/web-login/channel-registration) (LINEへのいわゆるアプリ登録) が必要で、[LINE Partner Program](https://developers.line.me/requestform/input) へ登録していないと、 `CHANNEL_CREATOR` の権限が得られないみたいなんですよね。
- なお、どうせ LINE Partner Program へ登録するなら、LINE公式アカウントを取って、より高機能な [Business Connect](https://developers.line.me/businessconnect/overview) を用いたほうがよさそう
   - REST APIs では、[Link Message](https://developers.line.me/restful-api/link-messages) しか送れないが、Business Connect なら、[Microsoft りんな](http://rinna.jp/rinna/) とかみたいに ChatBot みたいなメッセージのやり取りが可能になるはず
   - 今作りたいものはそんな大掛かりなものじゃない・・・と思いつつ、LINEさん懐が狭いなと感じたり・・・

## 検討2: LINE@

- 次に、個人向けにも公開されている、ライト版LINE公式アカウントともいえる [LINE@](http://at.line.me/jp/) を使うことを検討
- [この記事](http://gaiax-socialmedialab.jp/line/379)でも紹介されている通り、無料で結構できるので、個人で使う分には何の問題もなさそう！
   - すません、LINEさん、やっぱ太っ腹でした！！
- スマホアプリのほか、Web の [LINE@ Manager](https://admin-official.line.me/) でも管理できて、送信メッセージの予約ができるので、今回の要件にもぴったし！

![line_manager.png](https://qiita-image-store.s3.amazonaws.com/0/54750/a5e5e675-6365-6530-e849-19a4ba123d5a.png "line_manager.png")

### 断念理由

- だが、あくまで個人アカウントからではなく、LINE@アカウントから送ることになるので、それじゃまず妻に個人のLINE@アカウントのフレンドになってもらわないといけないじゃないか!?
- サプライズでこっそりやりたいのだよ、もっと事前にフレンドになってもらうならともかく、直前では怪しすぎる・・・
- でも、LINE@、これはこれで面白そうなので、今度機会があったら何かに使ってみよう

## 検討3: MacOSXのLINEアプリ

- 前置きが長くなったけど、かくなる上は、Mac に入っている LINE アプリを使うことに
- まず次のような AppleScript を書く
   - [goo.gl](https://goo.gl/) の短縮アドレスは、お手紙画像を Web 上に置いたアドレスを短縮したもの

```:send_line_message.scpt
#!/usr/bin/osascript
on run argv
   tell application "LINE"
      activate
      delay 1
      tell applications "System Events"
         keystroke "HappyBirthday!!!"
         keystroke return
         keystroke "http://goo.gl/xxxxxx"
         keystroke return
      end tell
   end tell
end run
```

- 次に [atコマンド](http://yamaqblog.tokyo/?p=16006) を使って、指定時間に上記スクリプトが実行されるようにする

```bash
# Mac で atコマンドを使うには、あらかじめ有効にしておく
sudo launchctl load -w /System/Library/LaunchDaemons/com.apple.atrun.plist

# 時刻を指定してコマンド実行を予約
echo "osascript send_line_message.scpt" | at -t 201601301930

# 確認
atq
```

- Mac がスリープ状態にならないよう、System Preference で以下の点を確認する
   - Screen Saver が開始しないようにする
   - Energy Saver で Sleep にならないように設定しておくことも忘れない
   - 地球環境のためにこれらは後で元に戻しておいてね
- ノートのフタを閉めても大丈夫なようにしたいなら、[InsomniaX](http://www.macupdate.com/app/mac/22211/insomniax) とかを入れておけばokなはず
- 電源はコンセントに接続して切れることのないように
- そして、あとは LINE アプリを起動し、妻とのトーク画面を開いた状態にしておくだけ・・・

### 結果は・・・？

- こうして、準備万端で妻と外出したわけだが、時が来ても妻のスマホが光らない・・・あれ？
- さすがにおかしいので、しれっーと手動で送ることに・・・妻は喜んでくれたが、エンジニアとしてはかなり無念・・・T T
- 家に帰って確かめてみたら、ネットとの接続が断たれていたのが原因だったorz
   - ということで、ネットとの接続も確認した上で外出を、というのも条件に加えておく、というオチ付きで
