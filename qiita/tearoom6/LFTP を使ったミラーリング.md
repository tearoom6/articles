
## LFTP

- [LFTP](http://lftp.yar.ru) は、FTPによる一括処理やスクリプト処理を容易に行えるFTPクライアントツール


### インストール

- MacOSX には、Homebrew でインストールできる

```
$ brew install lftp
```


### 使い方

- まず以下のようにして接続

```
$ lftp -u <username>,<password> <url>
```

- もしくは、-fオプションでスクリプトを実行も可能

```
$ lftp -f <script_file>
```

### コマンド

- LFTP コンソールでは、通常のFTPコマンドのほか、独自コマンドが使える
- コマンド一覧を表示

```
> help
```

- コマンドのヘルプを表示

```
> help <command>
```

### ミラーリング

- mirror コマンドを用いてミラーリングが可能

```
# アップロード例
mirror --dry-run -R -v -p -L <src_dir> <dst_dir>
mirror -R -v -p -L <src_dir> <dst_dir>

# ダウンロード例
mirror --dry-run -v -p -L <src_dir> <dst_dir>
mirror -v -p -L <src_dir> <dst_dir>
```

`-v` 詳細表示
`-p` パーミッションを設定しない
`-L` シンボリックリンクをファイルとしてコピー

