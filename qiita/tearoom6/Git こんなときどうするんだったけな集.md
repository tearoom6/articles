イレギュラーなケースについては、毎回調べているような気がするので、気づいたものはまとめておきたい。
(こんなのもあるよというのがあれば、ぜひコメントください！)

## リモートに push しちゃったコミットをなかったコトにしたい (revert ではなく reset したい)

```sh
# HEAD^ のところを戻したいコミットハッシュで指定すれば、どこまでも戻せる
git push -f origin HEAD^:<branch>
```

> References

- [共有レポジトリにPushしてしまったコミットをやり直す - Qiita](https://qiita.com/kentosasa/items/959a1a78ebc8ca70d7f6)

## First commit を reset したい

```sh
git update-ref -d HEAD
```

> References

- [How to revert initial git commit? - Stack Overflow](https://stackoverflow.com/questions/6632191/)

## First commit から rebase したい

```sh
git rebase -i --root
```

> References

- [First commit が git rebase -i できない問題 → git rebase -i --root でできる - 納豆には卵を入れる派です。](http://d.hatena.ne.jp/ken_c_lo/20130421/1366558065)
