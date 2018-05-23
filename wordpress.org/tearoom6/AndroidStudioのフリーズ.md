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

その際、 `.idea/misc.xml`/`.idea/vcs.xml` の2つのファイルが微妙に書き換わったので、そのあたりも関係あるかも。 (関係ないかも)

もし同様の事態が起きたら、試してみてください。

> References

- [android - How to clear gradle cache? - Stack Overflow](https://stackoverflow.com/questions/23025433/)
