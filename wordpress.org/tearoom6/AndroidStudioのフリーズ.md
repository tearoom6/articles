AndroidX のパッケージが使いたくて Android Studio Canary を使っている時の話。

しばらく使っているうちは何の問題もなかったんですが、突然にどんな操作を行ってもフリーズするようになりました。
Memory Indicator で見ても、メモリにはまだまだ余裕がある状態で、全く原因がわかりませんでした。

解決しようと、 Android Studio をインストールし直してみたり、マシンをリスタートしてみたり、いろいろとやったので、どれが原因だったのが断言はできないのですが、おそらく一番効いたと思うのが、 Gradle のキャッシュ削除です。

まず Android Studio の Launch 後の画面のプロジェクト一覧から、対象プロジェクトを削除した上で、 Android Studio を Quit します。

その上で、Android のプロジェクト直下のディレクトリで、以下のコマンドを実行します。

```sh
$ ./gradlew cleanBuildCache
```

そして、再度 Open an existing Android Studio project でプロジェクトを開き直すことでフリーズが起こらなくなりました。

もし同様の事態が起きたら、試してみてください。

他にも Gradle はデーモンを動かしていたり、いろいろなオプションがあることが分かって、勉強になりました。

```sh
$ ./gradlew --build-cache
$ ./gradlew --no-build-cache

$ ./gradlew --no-parallel
$ ./gradlew --parallel

$ ./gradlew --no-daemon
$ ./gradlew --daemon

$ ./gradlew --status
$ ./gradlew --stop
```

> 追記 (2018-06-15)

それにしてもやたら重いなと思っていたら、 Android Studio でメモリリークが起きていたようですね。。いや、それでなくても重いんですけどね。

通常版の 3.1.3 (June 2018) および Canary 版の Android Studio 3.2 Canary 17 (June 7, 2018) にて修正されています。

```
Memory leaks caused Android Studio to become slow and unresponsive after you had been using the Layout Editor. This update includes fixes for most of these issues. We intend to release another update soon to address additional memory leaks.
```

追加の修正が予定されているとのこと。

> References

- [android - How to clear gradle cache? - Stack Overflow](https://stackoverflow.com/questions/23025433/)
- [Android Studio release notes  |  Android Developers](https://developer.android.com/studio/releases/)
- [Android Studio Release Updates](https://androidstudio.googleblog.com/)
- [Android Studio Release Updates: Android Studio 3.2 Canary 17 available](https://androidstudio.googleblog.com/2018/06/android-studio-32-canary-17-available.html)
- [Updates on recent Android Studio memory leak issues : androiddev](https://www.reddit.com/r/androiddev/comments/8pcb2s/updates_on_recent_android_studio_memory_leak/)
