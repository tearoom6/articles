# docker login での Sign in について

## 現象

macOS で docker-compose を使って docker pull 中に、次のようなエラーが発生。

```
unauthorized: authentication required
```

ブラウザで https://registry-1.docker.io/v2/library/memcached/manifests/1.4.34-alpine にアクセスしたときにも `authentication required` みたいなエラーが出るので、それと同じですね。

Docker for Mac のメニューを見るとログインできているんだが。。
ネットを検索すると docker login すればいいよ、みたいな情報が出てくる。なるほど、 Docker for Mac のログインとはリンクしているようなしていないような関係なのか。

docker login を試してみる。が、メールアドレスを入れると、認証失敗。

```
$ docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: xxxxxxxxxxxxxxx@gmail.com
Password:
Error response from daemon: Get https://registry-1.docker.io/v2/: unauthorized: incorrect username or password
```

ユーザ名だと、認証成功。

```
$ docker login
Login with your Docker ID to push and pull images from Docker Hub. If you don't have a Docker ID, head over to https://hub.docker.com to create one.
Username: xxxxxxxxxxxxxxx
Password:
Login Succeeded
```

認証成功後は docker pull も通るようになりました。


## Docker registry への Sign in

Docker for Mac のメニューからの Sign in の場合、以下のいずれの組み合わせでも認証成功しました。 ([Docker Cloud](https://cloud.docker.com/) も同様)

- ユーザ名 / パスワード
- メールアドレス / パスワード

ところが、 docker login を使った場合、以下の組み合わせでしか認証通りませんでした。

- ユーザ名 / パスワード

docker login の Reference 見てみると、 [v17.03](https://docs.docker.com/v17.03/engine/reference/commandline/login/) の時点では、 `--email, -e` option というのがある。なるほど、これにメールアドレスを渡せばよかったのか。。

だがしかし、 [v17.06](https://docs.docker.com/v17.06/engine/reference/commandline/login/) を見てみると、 `--email` option が消えてる!!

調べてみると、もともとあった `--email` option は、 email との組み合わせでログインするのではなく、該当ユーザが作成されていなかった場合に、指定した email でユーザを作成しますよ、という機能で v17.06 でその機能自体が削除されたらしい。

つまりは docker login の場合は必ず ユーザ名 / パスワード でログインしてくださいということですね！ひいては Docker for Mac の場合もその組み合わせでログインしたほうが良さそう。

> References

- [docker unauthorized: authentication required - upon push with successful login - Stack Overflow](https://stackoverflow.com/questions/36663742/)
- [Deprecated Engine Features](https://docs.docker.com/engine/deprecated/#-e-and---email-flags-on-docker-login)
