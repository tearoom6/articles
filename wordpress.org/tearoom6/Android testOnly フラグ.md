# 事象

Android のビルド後の APK を実機にインストールしようとすると、次のようなエラーが発生した。

```sh
$ adb install -d app-debug.apk
adb: failed to install app-debug.apk: Failure [INSTALL_FAILED_TEST_ONLY: installPackageLI]
```


# 原因

これは testOnly フラグによるもの。
API level 4 の頃から testOnly フラグは、 AndroidManifest.xml の `application` 要素の `android:testOnly` 属性で設定できる。

testOnly フラグが立っているかどうかは、コード上は以下のように確かめられる。

```kotlin
val applicationInfo = packageManager.getPackageInfo("com.example.app", 0).applicationInfo
val isTestOnly = (applicationInfo.flags and ApplicationInfo.FLAG_TEST_ONLY) != 0
```

ところで、今回は AndroidManifest.xml で特に `android:testOnly` を設定したりはしていないのだが、このエラーが出てきた。

調べたところ、マニフェストファイルで指定する以外に、このフラグが立つ条件があるらしい。
公式の情報も間違っているんじゃないかな、と思う記載があったりして、正確な情報がいまいちわからないんだが、僕が調べた感じだと、以下の場合は testOnly な APK になるようだ。

- Android Studio 3.0 以降の [Run] or [Debug] で直接インストールした際に生成された APK
- `compileSdkVersion` が正式リリース版でない場合 (developer preview SDK、つまり数字ではなくてアルファベットで表される場合 : 今回の場合は `'android-P'`)


# 対応

このエラーを回避するには、2つの方法がある。

1つ目は `adb install` に `-t` オプション (`INSTALL_ALLOW_TEST` フラグ) を付けて実行すること。

```sh
$ adb install -d -t app-debug.apk
```

2つ目は `gradle.properties` に testOnly フラグを無効化するオプションを付けること。

```groovy
android.injected.testOnly=false
```

> References

- [Disable 'testOnly' mode for Android Studio 3.0](https://gist.github.com/xujiaao/5fd127a72979cdc3c70dcc1324786f87)
- [The CommonsBlog — Android Studio 3.0 and FLAG_TEST_ONLY](https://commonsware.com/blog/2017/10/31/android-studio-3p0-flag-test-only.html)
- [android - Google Play error: cannot upload a test-only APK - Stack Overflow](https://stackoverflow.com/questions/43690118/)
- [Android Debug Bridge (adb)  |  Android Developers](https://developer.android.com/studio/command-line/adb#-t-option)
- [Build and run your app  |  Android Developers](https://developer.android.com/studio/run/)
- [Build your app from the command line  |  Android Developers](https://developer.android.com/studio/build/building-cmdline)
- [androidStudio(3.0)でビルドしたapkがインストールできなくなる - Qiita](https://qiita.com/taohaipai/items/cf05d47675781d58a92a)
