# Ruby open-uri の open の戻り値

## Ruby open-uri ライブラリの open メソッド

open メソッドは引数に URL を取り、通常呼び出し or ブロック呼び出しのいずれかのスタイルを取ります。

```ruby
require 'open-uri'

file = open('https://hogehoge.com/image.jpg')
# 何らかの処理
file.close
```

```ruby
require 'open-uri'

open('https://hogehoge.com/image.jpg') do |file|
  # 何らかの処理
end
```

## open メソッド戻り値の型

ここで戻り値 (上記の場合 `file`) は、実は開こうとしたファイルのサイズによって、戻り値の型が以下のように変わるようです。

- 10 kb (10,240 bytes) より大きい場合、 [Tempfile](https://docs.ruby-lang.org/ja/latest/class/Tempfile.html)
- 10 kb (10,240 bytes) 以下の場合、 [StringIO](https://docs.ruby-lang.org/ja/latest/class/StringIO.html)

これは結構罠で、例えば Tempfile であることを期待して `file.path` なんて呼んでしまうと、以下のような例外が出てしまうケースがあります。

```
undefined method `path' for #<StringIO:0x00005583b538edc8>
```

## 対策

なら、どうするかということで、 10kb のしきい値を決めている `OpenURI::Buffer` の定数 `StringMax` を無理やり書き換えるなんで荒業もあるようですが、いい方法とは思えません。
もし常に Tempfile であることを期待したいのであれば、素直に手動で Tempfile を作るしか無いのかなぁという、現在までの結論です。

```ruby
temp_file = Tempfile.new('test_')
temp_file.binmode
open('https://hogehoge.com/image.jpg') do |file|
  # read は Tempfile でも StringIO でも duck typing でいけます。
  # 本来ならこれを handling すれば問題ないのですが、あくまで Tempfile であることを期待したい場合は...
  temp_file.write(file.read)
end
temp_file.rewind

# 何らかの処理

temp_file.close
```

> References

- [library open-uri (Ruby 2.5.0)](https://docs.ruby-lang.org/ja/latest/library/open=2duri.html)
- [Rubyのopen_uriでファイルオブジェクトがTempfileになる瞬間を追った - Qiita](https://qiita.com/Pujyoooo/items/96c680a156405b8cd442)
- [ruby - Why does OpenURI treat files under 10kb in size as StringIO? - Stack Overflow](https://stackoverflow.com/questions/10496874/)
- [OpenURI's open, Tempfile and StringIO - WinstonYW](http://winstonyw.com/2013/10/02/openuris_open_tempfile_and_stringio/)
- [Ruby - Reading A Remote Zip File - WinstonYW](http://winstonyw.com/2013/10/01/ruby_reading_a_remote_zip_file/)
