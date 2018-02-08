
Terminalのタブを複数開いて、各タブでssh接続した状況にするのを一発でやる方法です。

それぞれのサーバで、同一コマンドを実行させたいのであれば別のやり方がありますが、今回の方法は各サーバで別々の作業をしたいときなどに便利です。

まず、Terminalで新規タブを開いて、そのタブでコマンドを実行するApple Scriptを用意します。Terminal以外でもカスタマイズしたらできそうです。

```newtab.scpt
#!/usr/bin/osascript
on run argv
   # open new tab in terminal
   tell application "System Events"
      tell process "Terminal" to keystroke "t" using command down
   end tell

   delay 1

   # execute command in new tab
   tell application "Terminal"
      activate
      repeat with execCmd in argv
         do script with command execCmd in selected tab of the front window
      end repeat
   end tell
end run
```

続いて、一発で複数実行するためにスクリプトを書きます。
server001 server002 server003 といったように連番になっている場合は、以下のようにするといいかと思います。
サーバ名が連番ではない場合は、forループは使わず、愚直に一つずつ書くといいでしょう。

```Bash:sshonce.sh
#!/bin/bash
for no in `seq 1 3`; do
   number=`printf "%03d" ${no}`
   osascript newtab.scpt "ssh server"${number}
done
```

そして実行すると、Terminal のタブが一つずつ開いて、それぞれのサーバにssh接続します。実行中は触らないように注意です！

```Bash
$ ./sshonce.sh
```

