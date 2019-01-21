## ケース1

`brew upgrade` などをしたときに起こることですが、 Homebrew がローカルに持っているいくつかの Git repository (`/usr/local/Homebrew/Library/Taps/homebrew/homebrew-core` など) で pull が起こるときに、取得していない差分があまりに多いと、それなりの時間がかかることがあります。

`Updating Homebrew...` というログが出ているときがその目印です。
これは必要な処理になるので、辛抱強く待っていましょう。こまめに `brew update` しておけば、上記の repository を更新してくれるので、よいかもしれません。

> References

- [brew/brew.sh](https://github.com/Homebrew/brew/blob/170c5493a4e3628ed77137215a9ed6328e1a17c8/Library/Homebrew/brew.sh)
- [brew/update.sh](https://github.com/Homebrew/brew/blob/170c5493a4e3628ed77137215a9ed6328e1a17c8/Library/Homebrew/cmd/update.sh)

## ケース2 gcc をインストールしようとしている

gcc を直接、ないし、依存しているライブラリをインストールするため間接的にインストールしようとすると、とても時間がかかることがあります。
私の場合は macOS Mojave update 後に起こりました。どうも gcc をソースからコンパイルしようとするため、時間がかかるようです。

gcc をインストールしようとしているログが出ていて、その直後から全然進まない場合は Ctrl-C で一度 stop しちゃってください。
(Homebrew は安全設計なので途中で止めても大丈夫です)

その上で、

```sh
$ xcode-select --install
```

を実行して、まず Xcode Command Line Tools をインストールしてください。その上で再実行すれば、かなり短時間で完了します。

下記のリンクに書いてあることですが、 Xcode Command Line Tools がインストールされている場合、 Homebrew は Homebrew 用語で "bottle" と呼ばれる binary 形式の gcc を使用するようになるので、インストールが高速になるのです。

> References

- [homebrew - brew install gcc too time consuming - Stack Overflow](https://stackoverflow.com/questions/24966404/)
- [homebrew-core/gcc.rb](https://github.com/Homebrew/homebrew-core/blob/6a81aeee836f93d805f679e4f77a1a9589fa35d5/Formula/gcc.rb)
