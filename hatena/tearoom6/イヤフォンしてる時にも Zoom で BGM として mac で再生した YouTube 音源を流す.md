通常だと Zoom や Google Meet で拾う音源は、 mac の備え付けマイク (Internal Microphone) のものだけになってしまいます。
そのため、イヤフォンをしてる状態だと、スピーカーから出た音声を再度マイクで拾って、みんなに聞こえるようにするとかいうのができないですが (スピーカーの場合でも音質も落ちるというのがありますし)、マイクで拾った音声と PC で流れている音声をミキシングして、流すようにします。

なお LadioCast を使わなくても、 macOS 付属のアプリ "Audio MIDI Setup" でも同様のことはできるみたいですが、おそらく LadioCast を使ったほうが簡単です！

## [Soundflower](https://github.com/mattingalls/Soundflower) をインストール

macOS 上で仮想オーディオデバイスを作るためのソフトウェアです。

オープンソースであることもあって、 GitHub を見ると、かなりたくさんの系統が有るようですが、現在人気なものは以下の 2 つになるようです。

- https://github.com/mattingalls/Soundflower
- https://github.com/mLupine/SoundflowerBed

とりあえず https://github.com/mattingalls/Soundflower の方をインストールします。

```sh
brew cask install soundflower
```

## [LadioCast](https://apps.apple.com/jp/app/ladiocast/id411213048) をインストールする

macOS 上で動作する音声ミキサーツールです。

[mas](https://github.com/mas-cli/mas) で App Store からインストールします。

```sh
mas install 411213048
```

## 原理

ややこしいので、 Sound Effects の話は横に置いておきます。

macOS のデフォルト設定では、 macOS 内で流れる出力音声 (Output) は、

- イヤフォン/ヘッドフォン未接続状態だと "Internal Speakers" (内蔵スピーカー)
- イヤフォン/ヘッドフォン接続状態だと "Headphones" (イヤフォン/ヘッドフォン)

に流れてしまいます。

また、 macOS が拾う入力音声 (Input) は、 "Internal Microphone" (内蔵マイク) のものになります。 (多分、外付けマイクを繋いだ場合は、それが選ばれる)

通常だと、音声の流れは、以下のようになっています。

```
(入力方向) マイクの音声             ---> macOS 内のアプリが受け取る音声

(出力方向) macOS 内のアプリが流す音声 ---> 内蔵スピーカー/イヤフォン/ヘッドフォン
```

これを、仮想オーディオデバイスを用いて、以下のように流れを変えてやります。

```
(入力方向) マイクの音声             ---> "Soundflower (64ch)" ---> macOS 内のアプリが受け取る音声
                                            ↑
(出力方向) macOS 内のアプリが流す音声 ---> "Soundflower (2ch)" ---> 内蔵スピーカー/イヤフォン/ヘッドフォン
```

## 操作手順

以下の手順で操作します。

1. macOS の Preferences から [Sound] > [Output] で device として "Soundflower (2ch)" を選びます

   ![audio_mixing_mac_preferences.png](https://files.tearoom6.biz/f107ba16-a0ce-43e3-972f-feac1fba3e60.png)

2. LadioCast を起動, 以下のような設定にします

   - Input1: "Soundflower (2ch)", Main / Aux 1 を選択
   - Input2: "Built-in Microphone", Main を選択
   - Main Output: "Soundflower (64ch)" を選択
   - Aux Output 1: "Built-in Output" を選択
   - その他は全て "N/A" を選択
   - 音量の調整はご自由に (変化の仕方が割合急なので、結構うまく調整しないと、音声が流れてこないなと勘違いしてしまいます)
   - 1, 2 の手順で、設定ミスると、ループして、エコーがかかったりするので、注意

   ![audio_mixing_ladiocast.png](https://files.tearoom6.biz/700e6c57-d653-4af4-936f-f8bb3427f3d0.png)

3. Zoom.us を起動, [Settings] > [Audio] > [Microphone] に "Soundflower (64ch)" を選択

   ![audio_mixing_zoom.png](https://files.tearoom6.biz/fc6a7dfa-9b8f-4cb0-8945-6122b0df10a7.png)

以上です!

設定切り替え忘れて、真面目な社内会議の時に実は BGM 流れてたみたいなことになると恥ずかしいので、そこだけお気をつけて...ｗ (みんな言わずにクスクスしてるみたいな)

> References

- [【Mac OS Catalinaでも使える】SoundflowerとLadioCastの使い方と設定 | 脳無](https://irilyuu.com/mac-ios/mac/soundflower-ladiocast/)
- [Macの音とマイクの声をミックスして動画を収録するTIPS - kun432's blog](https://kun432.hatenablog.com/entry/tips_for_screencast_with_sound_and_voice_on_mac)
- [Macのサウンドを自由にルーティング/配信する　Soundflower / LadioCast の使い方](https://sleepfreaks-dtm.com/dtm-materials/mac-sound/)
- [ボイスチャットの音声ルーティングメモ｜はるねずみ｜note](https://note.com/field_mouse/n/nc8b5bd38ee5b)
- [Uninstalling Soundflower – shinywhitebox help](https://support.shinywhitebox.com/hc/en-us/articles/202751790-Uninstalling-Soundflower)
- [Using SoundFlower with Google Hangouts - Mike Lynch](http://mikelynchgames.com/fun-n-games/using-soundflower-with-google-hangouts/)
- [Audio MIDI Setup User Guide for Mac - Apple Support](https://support.apple.com/en-in/guide/audio-midi-setup/welcome/mac)
