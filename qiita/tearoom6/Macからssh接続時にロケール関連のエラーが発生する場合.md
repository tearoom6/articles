
## 現象

- MacOSX Terminal から ssh で繋いだ時に、以下のようなロケール関連のエラーが出ることがある

```
can't set the locale; make sure $LC_* and $LANG are correct
```


## 解決方法

- 結論から言えば、Terminal のメニューから、[Preferences...]を開き、[Profiles]-[Advanced]タブの [Set locale environment variables on startup] をオフにすれば良い


## 原因

1. Terminal の上記オプションがオンになっていると、Terminal 起動時に、`LC_CTYPE="UTF-8"`がセットされる
2. Mac 側の /etc/ssh/ssh_config で `SendEnv LANG LC_*` になっていると、ssh接続時にこの環境変数 LC_CTYPE がサーバ側に送られる
3. サーバ側の /etc/ssh/sshd_config で `AcceptEnv LANG LC_*` になっていると、クライアント側から送られた LC_CTYPE が接続先でも受け入れられ、有効になる
4. "UTF-8"なんてロケールないよ！って怒られる

このため、原因1 の元を断てば、エラーは起こらなくなります。

