
今年、確定申告を e-Tax で完結させたんですが、住宅ローン関連の書類を PDF でアップロードしなければならなかったので、その際に役立った TIPS です。

元の書類は紙なので、スキャナーが手元になくて、コンビニ行くのも面倒な場合 (そもそも住宅ローン関連の書類は厳重にホッチキス留めされていて、スキャンしにくいし)、スマホで写真撮影して、それを PDF にしたくなりますね。

知らなかったんですが、 ImageMagick を使えば、簡単に複数画像を結合して 1 つの PDF ファイルにできました。

```sh
convert image0.jpg image1.jpg image2.jpg output.pdf
```

複雑なオプションなども要らなくて、いや、楽でした。

> References

- [Create a single pdf from multiple text, images or pdf files - Ask Ubuntu](https://askubuntu.com/questions/303849/)
