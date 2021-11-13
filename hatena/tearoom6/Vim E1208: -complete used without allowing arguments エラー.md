Vim のバージョンを 8.2.3550 に上げた後、 Vim を開くと、以下のようなエラーが発生しました。

```
hint: Waiting for your editor to close the file... Error detected while processing /Users/t-murota/dotfiles/.vim/dein/.cache/.vimrc/.dein/plugin/fugitive.vim:
line  410: E1208: -complete used without allowing arguments
line  411: E1208: -complete used without allowing arguments
```

これは Vim 本体の patch v8.2.3141 にて加わった Vim の API 変更の影響を、プラグインである fugitive.vim が受けたことによるエラーメッセージになります。

この問題は fugitive.vim v3.4 にて修正されています。

私の場合、 Vim のプラグイン管理に [dein.vim](https://github.com/Shougo/dein.vim) を使っているので、以下のコマンドを叩いて、プラグインのアップデートを行い、 Vim を開き直すことで解消しました。

```vim
:call dein#update()
```

> References

- [patch 8.2.3141: no error when using :complete for :command without -n… · vim/vim@de69a73](https://github.com/vim/vim/commit/de69a7353e9bec552e15dbe3706a9f4e88080fce)
- [E1208: Error detected while processing /usr/share/vim/vimfiles/plugin/fugitive.vim · Issue #1791 · tpope/vim-fugitive](https://github.com/tpope/vim-fugitive/issues/1791)
- [Release v3.4: fugitive.vim 3.4 · tpope/vim-fugitive](https://github.com/tpope/vim-fugitive/releases/tag/v3.4)
