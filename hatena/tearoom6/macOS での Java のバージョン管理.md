ちょっと真の本題で書いたことを調べようと思って、いろいろ調べた結果の備忘録です。
でも、真の本題の方は、いまいち解決していないという。。

## Java のバージョンについて

Java については、まず、言語そのものと、その動作環境である JVM があり、それらについて、公式の仕様の部分と、公式 (Oracle) やその他さまざまなところが出している実装の部分があります。

Oracle が出している Java の仕様は Java SE (Standard Edition) と呼ばれます。
アプリケーション実装に踏み込んだ仕様である Java EE (Enterprise Edition) (現在は Jakarta EE として移管) や組み込み用途での Java ME (Micro Edition) というのがあった影響で、 Java SE と名付けられていますが、完全に歴史的経緯ですね。

Oracle による Java 実装は、通称 [Oracle JDK](https://www.oracle.com/technetwork/java/javase/downloads/index.html) と呼ばれています。

また、準公式的な位置づけのオープンソース Java 実装として [OpenJDK](https://openjdk.java.net/) というのもあります。Oracle 買収前の Sun Microsystems が始めたもののようです。

そして、昨今 Java を取り巻く環境が大きく変わってきています。
他のプログラミング言語の進歩に取り残される形になった Java が変化に対応しようとする中での動きなのでしょう。

まず、短いスパンで、細かい機能修正を加えたバージョンアップが行われるようになりました。
2017-09 リリースの Java 9 以降、現在のところ、半年に1回、メジャーバージョンが更新されています。

そして Java 11 以降ですが、 Oracle のライセンス形態が変わり、公式版のいわゆる Oracle JDK 自体は利用が有償化されるようになりました。
それに変わって、今まで Oracle JDK に含まれていた機能で OpenJDK に入っていなかった部分も OpenJDK に移管され、オープンソース化されました。
OpenJDK 自体は無償で利用できますし、この流れを受けて、 OpenJDK を独自にビルドしたディストリビューションも各社・コミュニティが公開するようになりました。

つまり、今まで Java といえば、だいたい Java 6-8 あたりの Oracle JDK だけ使っていればよかったのが、各ベンダ別のさまざまなバージョンの "Java" が普及するようになっています。

> References

- [Oracle JDK Releases for Java 11 and Later | Oracle Java Platform Group, Product Management Blog](https://blogs.oracle.com/java-platform-group/oracle-jdk-releases-for-java-11-and-later)
- [「Java8からJava11」で何が起きたのか、どう環境構築すればいいのか - Qiita](https://qiita.com/to-lz1/items/898421e5050cae90ec20)
- [JDK、Oracle JDK、OpenJDK、Java SEってなに？ - Qiita](https://qiita.com/nowokay/items/c1de127354cd1b0ddc5e)
- [オープンソースのOracle JDKとOpenJDKを比較](https://www.ossnews.jp/compare/Oracle_JDK/OpenJDK)


## macOS での Java のバージョン管理

本題です。主には SDKMAN! を使った方法と `/usr/libexec/java_home` を使った方法があります。

### SDKMAN! を使った方法

[SDKMAN!](https://sdkman.io/) は、 JVM 系のバージョン管理ツールで、 Java だけではなくて、 JVM 系の言語 (Groovy, Scala など) やミドルウェアのバージョン管理までやってくれます。
もともとは GVM (Groovy enVironment Manager) だったらしい。

各バージョンの検索・インストールから、各 scope でのバージョン切り替えまでをやってくれます。詳細はここでは書きません。

内部的に `JAVA_HOME` を切り替えているようです。

### `/usr/libexec/java_home` を使った方法

こちらの方法では、インストーラや Homebrew を使ってインストールした Java のバージョンの切り替えを行えます。

例えば Homebrew で以下のように AdoptOpenJDK 最新版をインストールします。
macOS Catalina では、以下のように `--no-quarantine` option が必要です。

```sh
$ brew cask install adoptopenjdk --no-quarantine
```

インストールした Java の一覧は、以下のコマンドで確認できます。 `/Library/Java/JavaVirtualMachines/` 配下にあるものだけが表示されるようです。
ちなみに、このディレクトリはシステムベースのもので、 SDKMAN! はユーザベースでモジュールを管理することから、 SDKMAN! でインストールした Java はこのディレクトリには配置されず、したがってこのコマンドの結果には出てきません。
SDKMAN! のドキュメントにも、 `/usr/libexec/java_home` を使った方法との互換性はないと書かれています。

```sh
$ /usr/libexec/java_home -V
```

特定のバージョンの Java のパスを得たい場合は、小文字の `-v` オプションを利用する。

```sh
$ /usr/libexec/java_home -v 13.0
```

オプション無しで実行すれば、デフォルト (たぶん最新バージョン) の Java のパスが取得されます。

```sh
$ /usr/libexec/java_home
```

特定のバージョンに `JAVA_HOME` を切り替える場合は、以下のようにします。

```sh
$ export JAVA_HOME=`/usr/libexec/java_home -v 13.0`
```

一時的にデフォルト以外の Java バージョンを使って、実行することも可能です。

```sh
$ /usr/libexec/java_home -v 13.0 --exec java -version
```

お察しのとおりですが、 `/usr/libexec/java_home` は Java のバージョンを見るだけで、そのビルドを発行しているベンダまでは見てくれないので、複数ベンダの Java を入れた場合はうまく対処してくれないみたいです。
`/Library/Java/JavaVirtualMachines/<JAVA>/Contents/Info.plist` に書いてある `JVMVersion` のバージョンで比較をしているようですね。

ベンダも含めて管理したい場合は、 [jEnv](http://www.jenv.be/) を使うのが良さそうです。

> References

- [sdkmanでJavaのインストール - mike-neckのブログ](https://mike-neck.hatenadiary.com/entry/2017/01/15/002502)
- [java_home and JAVA_HOME on macOS - Notes for Geeks - Medium](https://medium.com/notes-for-geeks/java-home-and-java-home-on-macos-f246cab643bd)
- [Working with multiple Java versions in MacOS - Bruno Frascino - Medium](https://medium.com/@brunofrascino/working-with-multiple-java-versions-in-macos-9a9c4f15615a)
- [HomebrewでインストールできるJDKまとめ（2019年11月時点） - Qiita](https://qiita.com/gishi_yama/items/ee3526e7e7a922148333)
- [macos - Why doesn't /usr/libexec/java_home recognize JDK 1.8? - Ask Different](https://apple.stackexchange.com/questions/74280/why-doesnt-usr-libexec-java-home-recognize-jdk-1-8)
- [Unsigned error when using this cask on Catalina 10.15.1 · Issue #267 · AdoptOpenJDK/homebrew-openjdk](https://github.com/AdoptOpenJDK/homebrew-openjdk/issues/267)
- [macos - How can I change Mac OS's default Java VM returned from /usr/libexec/java_home - Stack Overflow](https://stackoverflow.com/questions/17885494/)
- [FAQ · sdkman/sdkman-cli Wiki](https://github.com/sdkman/sdkman-cli/wiki/FAQ#on-mac-usrlibexecjava_home-does-not-detect-alternatives-installed-by-sdkman-what-can-i-do)
- [Installed JDK in Mac, But can show it with system command `/usr/libexec/java_home` · Issue #638 · sdkman/sdkman-cli](https://github.com/sdkman/sdkman-cli/issues/638)


## Homebrew で `Java 1.8+ is required to install this formula` みたいなエラーが出る場合

真の本題です。

SDKMAN! で入れた Java しか無い状態で、 Homebrew で bundletool をインストールしようとしたところ、以下のエラーが発生しました。

```sh
$ brew install bundletool
bundletool: Java 1.8+ is required to install this formula.
Install AdoptOpenJDK with Homebrew Cask:
  brew cask install adoptopenjdk
Error: An unsatisfied requirement failed this build.
```

[bundletool の formula](https://formulae.brew.sh/formula/bundletool) を見たところ、 `depends_on :java => "1.8+"` の指定があり、これを満たせていないようです。
SDKMAN! で入れた場合にも `JAVA_HOME` は入っているし、[Homebrew のソースコード](https://github.com/Homebrew/brew/blob/2.2.3/Library/Homebrew/requirements/java_requirement.rb) をちらっと見るには、 `JAVA_HOME` に適切な Java が入っていれば問題なさそうなんですが...
あまりじっくり見ていないので、何がダメなのかちょっと良くわかりませんでした。誰か分かる人教えてください><

とりあえず言われるがままに、 Homebrew で AdoptOpenJDK をインストールして、特に `JAVA_HOME` の切り替えなどはせずに再度 `brew install bundletool` を実行すると、うまくインストールできました。

```sh
$ brew cask install adoptopenjdk --no-quarantine
```

Homebrew でのインストール時に必要なだけで、後は SDKMAN! でインストールした Java でも動作しそうなので、すぐにアンインストールしても問題なさそうです。

```sh
$ brew cask uninstall adoptopenjdk
```

> References

- [When you come across this issue “Java 1.8 - michael khan - Medium](https://medium.com/@michaelkhan3/when-you-come-across-this-issue-java-1-8-8f88d27f6a23)
