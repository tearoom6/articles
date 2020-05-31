# OBS (macOS) Virtual Camera で遊ぶ

References に挙げた記事を見て、遊んでみました。

## [OBS (Open Broadcaster Software) Studio](https://obsproject.com/) をインストール

自分の場合、以下のコマンドで最新版が入りましたが、一応最新版かどうか確認しておいてください。

```sh
brew cask install obs
```

## [OBS (macOS) Virtual Camera](https://github.com/johnboiles/obs-mac-virtualcam) をインストール

最近 `.pkg` 形式で、正式版がリリースされたということらしいですが、誰かが早速 brew cask 化してくれているので、これもそれで入れます。

```sh
brew cask install obs-virtualcam
```

## [Zoom.us](https://zoom.us/) をインストール

こちらもインストール後、最新版でなければ、最新版に上げておきます。

```sh
brew cask install zoomus
```

なお v4.6.9 で、おそらくセキュリティの関係上 (リリースノートには一時的にとは書いてありますが) virtual camera が選択できなくなったのですが、以下のコマンドを実行することで最新版でも選択できるようになるそうです。
macOS 10.15 (Catalina) での Device Abstraction Layer (DAL) plugins をアプリが許可しているかどうかというのと関係しているそうですが、セキュリティ的に保証できないので、あくまで私的な実験にお使いください。

```sh
sudo codesign -f -s - /Applications/zoom.us.app
```

もちろん zoom をインストールしなおせば、元に戻ります。

** 追記 (2020-05-31) **

Zoom.us v5.0.4 にて再度 virtual camera が選択可能になりました。これで Zoom でも堂々と OBS 経由の配信ができますね。

- [New updates for May 24, 2020 – Zoom Help Center](https://support.zoom.us/hc/en-us/articles/360043996131-New-updates-for-May-24-2020)

## [Snap Camera](https://snapcamera.snapchat.com/) をインストール

こちらは残念ながら brew cask や App Store に登録されていないっぽいので、手動で [公式ページ](https://snapcamera.snapchat.com/download/) からダウンロードしてインストールします。

## 操作手順

Snap Camera -> OBS Studio (-> OBS Virtual Camera) -> Zoom.us (or Google Meet on Chrome) の経路で映像が流れるようにします。

つまり Snap Camera と OBS Virtual Camera の仮想カメラ2段構えです。

以下の手順で操作します。

1. Snap Camera を起動, Emoji Head を選ぶ!

   ![obs_virtial_camera_trial_01.png](https://files.tearoom6.biz/9593cbe9-73b1-41e8-9ac8-212fc5d0bfe1.png)

2. OBS Studio を起動, Source に Device として "Snap Camera" を選択した "Video Capture" を追加

   ![obs_virtial_camera_trial_02.png](https://files.tearoom6.biz/f7988904-ad9b-4d42-9363-f300fa36d497.png)

   ![obs_virtial_camera_trial_03.png](https://files.tearoom6.biz/2f2cce48-9843-4c8d-868d-59739113b6b8.png)

3. OBS Studio のメニューで [Tools] > [Start Virtual Camera] を選択

   ![obs_virtial_camera_trial_04.png](https://files.tearoom6.biz/d079f07b-c3c4-46c6-b4ce-ca050fd4c377.png)

4. Zoom.us を起動, [Settings] > [Video] > [Camera] に "OBS Virtial Camera Device" を選択 (文字とか入れてる場合は "Mirror my video" を外さないと逆さ文字になりますよ)

   ![obs_virtial_camera_trial_05.png](https://files.tearoom6.biz/c8d00880-76ad-48ed-8db1-e2dd65979a0b.png)

以上です。

> References

- [johnboiles/obs-mac-virtualcam](https://github.com/johnboiles/obs-mac-virtualcam)
- [ライブ配信アプリOBS Studioに仮想カメラを作り出し、ZoomやGoogle Meetなどに映像を直接配信できるOBSプラグイン「OBS (macOS) Virtual Camera」がリリース。 | AAPL Ch.](https://applech2.com/archives/20200520-obs-studio-for-mac-virtual-camera-plugin.html)
- [New updates for macOS – Zoom Help Center](https://support.zoom.us/hc/en-us/articles/201361963-New-updates-for-macOS)
- [Compatibility · johnboiles/obs-mac-virtualcam Wiki](https://github.com/johnboiles/obs-mac-virtualcam/wiki/Compatibility)
