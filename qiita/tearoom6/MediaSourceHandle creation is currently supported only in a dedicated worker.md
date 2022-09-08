少し古いバージョン (v7.10 未満) の Video.js を使っているサイトを、 M105 の Chrome / Edge で見た時 (HLS streaming 開始時?) に、以下のエラーが発生する場合があるようです。

```
DOMException: Failed to read the 'handle' property from 'MediaSource': MediaSourceHandle creation is currently supported only in a dedicated worker.
```

(2022-09-08 現在の状況をまとめます。Android/iOS 版でも同様に起きるようですが、以下の内容は Desktop 版の状況についてです)

M105 stable がリリースされた、昨月末以降チラホラ報告があるみたいです。

[Chrome Releases: Stable Channel Update for Desktop](https://chromereleases.googleblog.com/2022/08/stable-channel-update-for-desktop_30.html)

これは Chromium 105 に含まれている "Media Source Extensions in Workers" ([MediaSource in Worker · Issue #175 · w3c/media-source](https://github.com/w3c/media-source/issues/175)) 機能のリリースによって起きているそうです。

[Chromium Blog: Chrome 105 Beta: Custom Highlighting, Fetch Upload Streaming, and More](https://blog.chromium.org/2022/08/chrome-105-beta-custom-highlighting.html)

commit 的にはこちら。

https://chromium.googlesource.com/chromium/src/+/09e5fb1a68416f65ef2255c846b81a46f6aa8e52

以下のチケットで、この issue について、話し合われています。

[1358542 - [User Feedback - Stable] User reports of video playback not working as expected on M105 stable - chromium](https://bugs.chromium.org/p/chromium/issues/detail?id=1358542)

いったん、該当の機能 (`MediaSourceInWorkers`, `MediaSourceInWorkersUsingHandle`) は `stable` -> `experimental` status に戻されています。つまり、いったん revert された形になっています。

https://chromium.googlesource.com/chromium/src/+/d5cef2e293c12d33ca25955897d090018a3fe7d3

Chrome M105 では、このパッチが既に当てられた 105.0.5195.102 がリリースされているため、最新版にアップデートすれば、エラーは解消します。 M106 beta にも既にこの修正が取り込まれているとのこと。 (M107 以降は、現時点では未定です)

Edge の最新版 105.0.1343.27 にはまだ含まれていないです。

Video.js 側でも issue のチケットが上げられていますが、 Video.js v7.10 (2020-10-15 リリース) 以上では起きないようです。

[Failed to read the 'handle' property from 'MediaSource': MediaSourceHandle creation is currently supported only in a dedicated worker. · Issue #7905 · videojs/video.js](https://github.com/videojs/video.js/issues/7905)

またしばらくしたら状況をウォッチしておきたいと思います。
記載内容に誤りがあれば、ご指摘お願いします！

---

ウミトロンでは、私たちと一緒に「持続可能な水産養殖を地球に実装する」仲間を積極的に募集しています。
このミッションのもと、テクノロジーを用いて水産養殖の課題に挑む航海に共に出てみませんか?

https://umitron.com/ja/career.html
