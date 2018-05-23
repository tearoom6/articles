軽く周回遅れの情報ですが・・・

macOS High Sierra 10.13.2 で対応した [Meltdown](https://en.wikipedia.org/wiki/Meltdown_(security_vulnerability)) / [Spectre](https://en.wikipedia.org/wiki/Spectre_(security_vulnerability)) の修正がパフォーマンスを悪くするのが嫌で、アップデートを避けていたんですが、ついに観念して 2018-03-29 にリリースされた macOS High Sierra 10.13.4 にアップデートしようとしたところ、 "Installation could not be completed." のような表示とともに、 Shutdown もしくは Restart しか選べないダイアログが表示されるという事案が発生しました。

ここで、どちらを選択しても次に起動する際に同じ画面が表示されるという無限ループ。

とりあえずアップデートを避けるため、再起動時に option を押し続けて、表示されるダイアログで通常の起動ディスクを選んで通常起動しました。
その後、 [10.13.4 Combo Update](https://support.apple.com/kb/DL1959) をダウンロードし、これをインストールすることで解決しました。

このように幾つかのバージョンを飛び越えてアップデートする場合は Combo Update (日本語だと 統合アップデート) を使うべきなようです。
というか、通常は現バージョンから判断して必要な場合は Combo Update が選ばれると思うのですが (違ったらすみません)、今回はそうでなかった模様です。
私の場合、アップデート前は 10.13.1 でした。

アップデート前にはかならずバックアップ取りましょうというお話でした。

> References

- [Download macOS High Sierra 10.13.4 Update](https://support.apple.com/kb/DL1962)
- [install - Installation of macOS High Sierra Update 10.13.4 failed; how do I recover without losing data? - Ask Different](https://apple.stackexchange.com/questions/321244/)
- [About the security content of macOS High Sierra 10.13.2, Security Update 2017-002 Sierra, and Security Update 2017-005 El Capitan - Apple Support](https://support.apple.com/en-us/HT208331)
- [macOSのアップデート失敗！インストールを完了できなかった時の対処法](http://mitchie-m.com/blog/mac/macos-update-install-unfinished/)
- [Macのcombo(統合)アップデートと通常のアップデートの違い | MacRuby](https://macruby.info/mac/mac-combo-update-difference.html)
