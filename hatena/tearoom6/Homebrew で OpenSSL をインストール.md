自分の環境 (macOS Catalina Version 10.15.4) では、以下のように `openssl` command を打つと、 LibreSSL が使われていることがわかります。

```sh
$ openssl version
LibreSSL 2.8.3
```

現在、 OpenSSL の主要な fork として、本家を含めて、以下の 3 つが存在します。

- [OpenSSL](https://www.openssl.org/)
- [LibreSSL](https://www.libressl.org/)
- [BoringSSL](https://boringssl.googlesource.com/boringssl)

これ何かといえば、 2014-04 に公表され、世間を揺るがせた、有名な OpenSSL の Heartbleed 脆弱性が発端になって、誕生したもの。
(僕の中では 2010 年代に発覚した 2 大脆弱性といえば、これと、 2018-01 に公表された CPU 脆弱性の Spectre / Meltdown です)

この事件を機に、コードベースが古く、膨大になっていた OpenSSL からの脱却を図るべく、 OpenSSL を利用していた OpenBSD チームが独自 fork である LibreSSL を作成しました。
また、 Google も LibreSSL に続いて、主に社内利用を目的に BoringSSL という fork を作成しました。

オープンソースではこの手のことはよくあることです。

fork じゃないけど

- [npm](https://www.npmjs.com/) に対しての [Yarn](https://yarnpkg.com/) とか
- [Node.js](https://nodejs.org/en/) に対しての [Deno](https://deno.land/) とか

どれも、なんだかんだ本家も譲らずに、改善を重ねて、よい (?) ライバル関係になったりするものです。

話がそれたけど、 LibreSSL は基本的には OpenSSL との互換性を保っているものの、やはり fork した時点で、完全に互換性を保つのは難しいものです。
なので、 `openssl` と打ったのに、内実 LibreSSL なのはどうなんだと思う気持ちもあったりする。

そこで、 macOS で本家 OpenSSL を Homebrew で入れることにします。

なお、僕の環境では、なぜか既に入っていたのですが (依存関係かな?)、念の為 `brew reinstall` します。
入っていない場合は `brew install` してください。
(実際としては `openssl@1.1` が入るようです)

```sh
$ brew update
$ brew reinstall openssl
```

ただ、このままだと macOS の標準コマンドが使われるため、 Homebrew で入れたほうが使われるようにします。
ネットを調べると、以下のように `brew link` を使えばいいよ、と出てきたのですが、実行してみると、以下のように怒られる。

```sh
$ brew link --force openssl
Warning: Refusing to link macOS provided/shadowed software: openssl@1.1
If you need to have openssl@1.1 first in your PATH run:
  echo 'export PATH="/usr/local/opt/openssl@1.1/bin:$PATH"' >> ~/.zshrc

For compilers to find openssl@1.1 you may need to set:
  export LDFLAGS="-L/usr/local/opt/openssl@1.1/lib"
  export CPPFLAGS="-I/usr/local/opt/openssl@1.1/include"

For pkg-config to find openssl@1.1 you may need to set:
  export PKG_CONFIG_PATH="/usr/local/opt/openssl@1.1/lib/pkgconfig"
```

指示に従って (ただ、ちょっとアレンジして)、素直に、 `PATH` を上書きし、 Homebrew で入れたほうが使われるようにします。

```sh
$ echo 'export PATH=$(brew --prefix openssl)/bin:$PATH' >> ~/.zshrc
$ source ~/.zshrc
$ openssl version
OpenSSL 1.1.1g  21 Apr 2020
```

無事に本家 OpenSSL が使われるようになりました。

> References

- [LibreSSL の現在（2018年2月時点） - Qiita](https://qiita.com/kinichiro/items/c06692d81c175516e3cf)
- [LibreSSL の意義 - Qiita](https://qiita.com/kinichiro/items/3108e950b056963c33ad)
- [Heartbleed - Wikipedia](https://en.wikipedia.org/wiki/Heartbleed)
- [Spectre (security vulnerability) - Wikipedia](https://en.wikipedia.org/wiki/Spectre_(security_vulnerability))
- [Meltdown (security vulnerability) - Wikipedia](https://en.wikipedia.org/wiki/Meltdown_(security_vulnerability))
- [macos - How to install latest version of openssl Mac OS X El Capitan - Stack Overflow](https://stackoverflow.com/questions/35129977/)
- [macos - Update OpenSSL on OS X with Homebrew - Stack Overflow](https://stackoverflow.com/questions/15185661/)
- [macos - How do you re-install a package with Homebrew (Mac)? - Super User](https://superuser.com/questions/324980/)
- [Since upgrading to OSX Mojave, am receiving warnings about commonmarker-0.17.9/0.17.7.1 · Issue #5155 · Homebrew/brew](https://github.com/Homebrew/brew/issues/5155)
