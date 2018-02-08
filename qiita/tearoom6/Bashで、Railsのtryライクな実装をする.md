
## ActiveSupportのtryメソッド

- ActiveSupportの[try](http://guides.rubyonrails.org/active_support_core_extensions.html#try)メソッドといえば、非常にかゆいところのnilチェックを不要にしてくれる、とてもイカすやつだ

```
@account = Account.find(445)
account_name = @account.try(:name)
```

```
@account = Account.find(445)
if @account.present?
  account_name = @account.name
end
```


## Bashで Let's try

- そんな `try` のようなものをBashでも実装してみた
- Bashの場合はオブジェクト指向ではないので、メソッドではなく、関数という形で、こんな感じ
- コマンドや関数、エイリアス等が存在しているかチェックして、存在している場合のみ実行する
- `~/.bash_profile` などに定義しておく

※ 書いておいてアレですが、将来的にBashに`try`句が実装される可能性を嫌って、私個人は`try_exec`として定義して使っています

```Bash:~/.bash_profile
function try() {
   if type $1 2>/dev/null 1>/dev/null; then
      eval $@
   fi
}
```

- どのように使うのかといえば、インストールされているかどうか、ifで確認してから実行する場面があると思う
- 例えば、[rbenv](https://github.com/rbenv/rbenv) の初期化処理で、rbenvが存在している場合のみ実行するのに、以下のようにするところ、

```Bash:~/.bash_profile
export PATH="$HOME/.rbenv/bin:$PATH"
if which rbenv > /dev/null; then eval "$(rbenv init -)"; fi
```

- こんな感じ、ちょっと綺麗に見えません...?

```Bash:~/.bash_profile
export PATH="$HOME/.rbenv/bin:$PATH"
eval "$(try rbenv init -)"
```

- パイプもok

```Bash
$ try fortune | try cowthink
```

- 完全にオレオレ流なので、流儀的にどうなのかは怪しいけど、使用は自己責任でオネシャス

