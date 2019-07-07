Vim のバージョンアップ (再インストール) 後に Vim を開くと、以下のようなエラーが出るようになりました。

```
hint: Waiting for your editor to close the file... Warning: Cannot find word list "en.utf-8.spl" or "en.ascii.spl"
Warning: Cannot find word list "en.utf-8.spl" or "en.ascii.spl"
Warning: Cannot find word list "en.utf-8.spl" or "en.ascii.spl"
Error detected while processing /Users/tomohiro_murota/.vimrc:
line  187:
E185: Cannot find color scheme 'darkblue'
```

これは Vim のプラグイン管理に使っている dein.vim によるキャッシュが原因でした。

以下のようにキャッシュクリアを行ってあげると、エラーは解消されます。

```vim
:call dein#recache_runtimepath()
```

> References

- [VimでCannot find color schemeが出る原因と対処法](https://rcmdnk.com/blog/2017/07/18/computer-vim/)
- [dein.vim/dein.txt at master · Shougo/dein.vim](https://github.com/Shougo/dein.vim/blob/master/doc/dein.txt)
