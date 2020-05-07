## What's that?

macOS Catalina のバージョンを 10.15.4 (19E287) に上げたタイミングで、表題の "Legacy System Extension" っていうダイアログが出現した。

Apple 公式のお知らせに出ている通り、 system extensions (システム機能拡張) の一種ではあるが、古い手法である kernel extensions (KEXT: カーネル機能拡張) のサポートを Catalina を最後に打ち切るそうで、そのことの啓蒙のためにこのようなダイアログを出すことになったらしい。

つまり、一部の App (主に driver?) が kernel extensions を使っている場合に、このダイアログをユーザに表示することで、それらの App の開発元にアップグレードを促す施策のようだ。


## How should we do?

ダイアログの文言の中に、 App 開発元の情報が含まれているので、これを元に調べてみる。

自分の場合は、前に使った [j5create JUE120 USB™ 2.0 Ethernet Adapter](https://en.j5create.com/products/jue120) という製品を利用するためにインストールした [ドライバ](https://en.j5create.com/pages/drivers) が原因だったようだ。 (今見たら macOS Catalina support と書いてあるので、既にアップデートされているのかな?)

この製品に組み込まれている ASIX Electronics の AX88772 っていう IC chip のドライバが対象になってしまっているらしい。
このことは以下のコマンドでも確認できる。

```sh
# List only third party kernel extensions.
$ kextstat | grep -v com.apple
Index Refs Address            Size       Wired      Name (Version) UUID <Linked Against>
  161    3 0xffffff7f83d86000 0xef000    0xef000    org.virtualbox.kext.VBoxDrv (6.0.2) 23F2E3C8-C38A-3339-9A6E-08F8AD6CA635 <8 6 5 3 1>
  164    0 0xffffff7f83e75000 0x8000     0x8000     org.virtualbox.kext.VBoxUSB (6.0.2) E73ADEDC-4A15-36FB-B741-2373408F514B <163 161 59 8 6 5 3 1>
  166    0 0xffffff7f83e7d000 0x5000     0x5000     org.virtualbox.kext.VBoxNetFlt (6.0.2) 016CC2E0-4FB7-3DBA-A725-D353DF9D7DC9 <161 8 6 5 3 1>
  167    0 0xffffff7f83e82000 0x6000     0x6000     org.virtualbox.kext.VBoxNetAdp (6.0.2) 5044F02A-8CCC-36A9-A1B5-E31D27F69B62 <161 6 5 1>
  181    0 0xffffff7f83e8b000 0xa000     0xa000     com.asix.driver.ax88772 (1.6.0) 4094EF2F-D3D7-346E-B162-5D0EA479B99F <59 18 8 6 5 3 1>
```
```sh
# Print a list of kernel extensions which include `asix` text.
$ kextfind -bundle-id -substring  'asix' -print
/System/Library/Extensions/AppleUSBEthernet.kext
/Library/Extensions/AX88179_178A.kext
/Library/Extensions/AX88772.kext
```

見つかった KEXT は、以下のようなコマンドによって "無効化" することができるらしい。 (危険なコマンドであるようなので、実行には注意してください)

```sh
$ sudo kextunload /Library/Extensions/AX88772.kext
```

コマンド実行後は、以下のように ASIX の KEXT が出てこなくなる。

```sh
$ kextstat | grep -v com.apple
Index Refs Address            Size       Wired      Name (Version) UUID <Linked Against>
  161    3 0xffffff7f83d86000 0xef000    0xef000    org.virtualbox.kext.VBoxDrv (6.0.2) 23F2E3C8-C38A-3339-9A6E-08F8AD6CA635 <8 6 5 3 1>
  164    0 0xffffff7f83e75000 0x8000     0x8000     org.virtualbox.kext.VBoxUSB (6.0.2) E73ADEDC-4A15-36FB-B741-2373408F514B <163 161 59 8 6 5 3 1>
  166    0 0xffffff7f83e7d000 0x5000     0x5000     org.virtualbox.kext.VBoxNetFlt (6.0.2) 016CC2E0-4FB7-3DBA-A725-D353DF9D7DC9 <161 8 6 5 3 1>
  167    0 0xffffff7f83e82000 0x6000     0x6000     org.virtualbox.kext.VBoxNetAdp (6.0.2) 5044F02A-8CCC-36A9-A1B5-E31D27F69B62 <161 6 5 1>
```

ただ、上記コマンドの実行は危険なので、ドライバにアンインストーラが付いている場合は、それを実行することが推奨される。
(JUE120 のドライバもちゃんとアンインストーラが付いているようです)

また、もちろん、単にドライバをアンロードするだけだと、その機能を使えなくなってしまうので、引き続き使いたい場合で、アップデートされたドライバがあるなら、それをインストールする必要がある。

> References

- [About legacy system extensions - Apple Support](https://support.apple.com/en-us/HT210999)
- [レガシーのシステム機能拡張について - Apple サポート](https://support.apple.com/ja-jp/HT210999)
- [Deprecated Kernel Extensions and System Extension Alternatives - Support - Apple Developer](https://developer.apple.com/support/kernel-extensions/)
- [Why do I get the Legacy System Extension notification… | Endpoint Protector](https://www.endpointprotector.com/support/endpoint-protector/why-do-i-get-the-legacy-system-extension-notification-on-macos-10.15.4-catalina-beta-versions)
- [Apple Begins Warning Users That 'Legacy System Extensions' Won't Work With a Future Version of macOS | MacRumors Forums](https://forums.macrumors.com/threads/2227960/)
- [USB Ethernet Adapter not working after macOS Catalina 10.15 Update? We can help! – Plugable](https://plugable.com/2019/10/04/usb-ethernet-adapter-not-working-after-macos-catalina-10-15-update-we-can-help/)
- [USB Ethernet Controller - USB 2.0 to Gigabit Ethernet | ASIX](https://www.asix.com.tw/products.php?op=ProductList&PLine=71&PSeries=100)
- [ASCII.jp：不要なドライバーをスッキリ整理、KEXT関連コマンドを知る (1/2)](https://ascii.jp/elem/000/000/544/544198/)
- [kextstat Man Page - macOS - SS64.com](https://ss64.com/osx/kextstat.html)
- [kextfind Man Page - macOS - SS64.com](https://ss64.com/osx/kextfind.html)
- [kextunload Man Page - macOS - SS64.com](https://ss64.com/osx/kextunload.html)
