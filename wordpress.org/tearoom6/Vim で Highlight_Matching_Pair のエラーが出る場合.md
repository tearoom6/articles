Vim のバージョンアップデート後に、例えば、以下のようなコード

```ruby
def test(a)
  puts "Test: #{a}"
end
```

を書いたとして、 `{`, `}` のところにカーソルを合わせたときに、以下のようなエラーが発生しました。

```
Error detected while processing function <SNR>57_Highlight_Matching_Pair:
line   97:
E475: Invalid argument: 0
```

このエラーの原因は、結果的には、 Vim のバージョンアップによって API が変わっているのに、インストール済みの Vim のプラグインのバージョンを上げていないので、発生していたものでした。
(どのプラグインが原因までかは調べていません)

プラグインの方が修正されていない場合はアウトですが、プラグインの方も更新してくれているかもしれません。ということで、プラグインをアップデートしてみましょう。

私の場合、 Vim のプラグイン管理に [dein.vim](https://github.com/Shougo/dein.vim) を使っているので、以下のコマンドを叩いて、プラグインのアップデートを行い、 Vim を開き直すことで解消しました。

```vim
:call dein#update()
```

> References

- [dein.vim/dein.txt](https://github.com/Shougo/dein.vim/blob/af70c00/doc/dein.txt)
