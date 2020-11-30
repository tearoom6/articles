今日は小ネタです。 Docker じゃなくて Dock の方です。
以前から気にはなっていたんですが、ちょっと macOS の Dock の挙動について調べてみたので、記事にしてみます。

以下の挙動は、全て macOS Big Sur (11.0.1) で確認しました。

結果、思っていた以上に挙動不審で、闇を抱えていました・・・ (たぶん、歴史的経緯とか、それなりの理由とか、あるんだとは思うけど)

## Dock は画面上部には持ってこれない

これは個人的にはすごくやって欲しいんですけど、仕様的に Dock を画面上部に持ってくることはできないようです。
理由としてはおそらく Menu bar と被るから。それなら Menu bar も移動できるようにしてくれよ、と思ったりしますが、そういうわけにもいかないんでしょう。

System Preferences の設定項目の中では、これは [Dock & Menu Bar] の中にあります。

こちら、 macOS Catalina までは、単に [Dock] というメニュー項目でした。さすが Big Sur はメジャーバージョン上げているだけあって、 UI/UX も刷新していますね。

Dock 全般のその他の設定も可能になっています。

![macos_system_preferences_dock_and_menu_bar.png](https://files.tearoom6.biz/57b481fa-03a1-4afa-b35d-cd065d2ce29c.png)

## デュアルディスプレイにしたとき Dock の出現パターンは、ディスプレイの配置に影響される

まず、ここでの記述内容は、後述する System Preferences の [Mission Control] メニューの [Display have separate Spaces] にチェックが入っている前提での話になります。

![macos_system_preferences_mission_control.png](https://files.tearoom6.biz/85f2cf76-01d5-4264-bdb6-bf82539db49e.png)

この状態で、モニタを MacBook につなぎ、ミラーリングではなく、デュアルディスプレイとして使います。

そして System Preferences の [Displays] を開けば、 "メインディスプレイ" の方には、 [Arrangement] というタブが表示されるはずなので、それを選んで、ディスプレイの配置を調整します。

このとき、まず下のように、ディスプレイを左右に並べて配置した場合

![macos_system_preferences_displays_arrangement_001.png](https://files.tearoom6.biz/745d0cc8-e8e6-49ca-b186-1aba21727198.png)

1. Dock のポジションが Left の場合
   - `1` のディスプレイの左端にのみ Dock が出現
2. Dock のポジションが Right の場合
   - `2` のディスプレイの右端にのみ Dock が出現
3. Dock のポジションが Bottom の場合
   - `1`, `2` 両方のディスプレイの下端に Dock が出現

まぁ分からなくはない仕様ですね。

続いて、 `1`, `2` のディスプレイを左右逆に、左右に並べて配置した場合

![macos_system_preferences_displays_arrangement_002.png](https://files.tearoom6.biz/cf4cccd8-cd10-4d5b-bd41-9edeb8f3d4dd.png)

1. Dock のポジションが Left の場合
   - `2` のディスプレイの左端にのみ Dock が出現
2. Dock のポジションが Right の場合
   - `1` のディスプレイの右端にのみ Dock が出現
3. Dock のポジションが Bottom の場合
   - `1`, `2` 両方のディスプレイの下端に Dock が出現

パターンが分かってきた気がします。

その調子で、以下のように上下に並べて配置した場合も確かめてみたとき、異変が起こります。

![macos_system_preferences_displays_arrangement_003.png](https://files.tearoom6.biz/0305585f-11cc-417a-988b-2035a86b3e94.png)

1. Dock のポジションが Left の場合
   - メインディスプレイ (この場合 `1`) の左端にのみ Dock が出現
2. Dock のポジションが Right の場合
   - メインディスプレイ (この場合 `1`) の右端にのみ Dock が出現
3. Dock のポジションが Bottom の場合
   - `1` のディスプレイの下端にのみ Dock が出現... のように見せかけて
   - 赤い楕円で囲ったエリアの下端でカーソルをゴリゴリ押し当てていると、 `1` のディスプレイの下端に Dock が出現

お分かりいただけたでしょうか。説明してもわかりにくと思うので、ぜひやってみてください。

Dock のポジション、および、ディスプレイの配置方向において、上下と左右というのは同列の扱いではなく、明らかに非対称な動作を伴うものなのだということがわかります。そのことも考えて、使いやすいようディスプレイを配置する必要があるようです。

(ディスプレイの配置、上下派で、 Dock のポジション、左端派の自分にとっては、嬉しくない仕様です...)

なお、一番最後のケースの説明で "メインディスプレイ" という言葉を使っていますが、実はこのメインディスプレイは切り替えることができます。

同じ [Arrangement] の設定画面で、ディスプレイの上部に白い枠がありますが、これをドラッグドロップでもう一つのディスプレイに持っていけば、そちらがメインディスプレイに切り替わります。

![macos_system_preferences_displays_arrangement_004.png](https://files.tearoom6.biz/e81d61b6-d1f3-4a46-b123-4c3e18ff3f27.png)

この状態では、メインディスプレイは `2` の方ですから、 Dock のポジションを Left または Right に持ってきた場合 Dock が出現するのは `2` のディスプレイの方になります。

ただし、メインディスプレイの変更による影響は Dock だけに留まらないため、その点はご注意ください。

## (だいたいは) メインディスプレイの方にだけ Dock が出現できるように設定可能

先ほどチラッと書いたんですが System Preferences の [Mission Control] メニューに [Display have separate Spaces] という項目があります。

こちらのチェックを外すと、下のように `Requires log out` とのメッセージが出るので、その通り、ログアウトして、ログインし直すと、この設定変更が有効になります。

![macos_system_preferences_mission_control_displays_have_separate_spaces.png](https://files.tearoom6.biz/ce8eeee3-3cab-46c7-8dfa-743a0533cdb4.png)

これをすると、どのような変化があるのかというと、以下のようになります。

ディスプレイの配置が左右の場合:

1. Dock のポジションが Left の場合
   - 左側ディスプレイの左端にのみ Dock が出現
2. Dock のポジションが Right の場合
   - 右側のディスプレイの右端にのみ Dock が出現
3. Dock のポジションが Bottom の場合
   - メインディスプレイの下端にのみ Dock が出現

ディスプレイの配置が上下の場合:

1. Dock のポジションが Left の場合
   - メインディスプレイの左端にのみ Dock が出現
2. Dock のポジションが Right の場合
   - メインディスプレイの右端にのみ Dock が出現
3. Dock のポジションが Bottom の場合
   - メインディスプレイの下端にのみ Dock が出現
   - さっきみたいに、境目部分にゴリゴリカーソルを当てても、 Dock はサブディスプレイの方には出現しない

つまりは、いずれのケースでも、どちらか片方の画面にしか Dock が出現しないようになるんですね。そして、多くのケースでは、それはメインディスプレイです。

ただし、この設定は Dock 以外にも影響があり、 Menu bar もメインディスプレイの方にしか出現しないようになります。個人的にはそれじゃない感が強い。


以上、謎多き Dock の挙動でした。

え、トリプルディスプレイの場合?? 以上ですー。

> References

- [Pin the dock in macOS to the top of the screen - Super User](https://superuser.com/questions/1274452/)
- [【Mac】上下にデュアルディスプレイにした際に、Dockを上に持っていく方法 | leisureed.net](https://www.leisureed.net/mac_dock_dualdisplay/)
- [Mac でサブディスプレイに Dock を移動させないようにする方法 - Corredor](https://neos21.hatenablog.com/entry/2017/12/12/080000)
- [マルチディスプレイ環境でDockの位置… - Apple コミュニティ](https://discussionsjapan.apple.com/thread/10184241)
- [マルチディスプレイ時のDockについて - Apple コミュニティ](https://discussionsjapan.apple.com/thread/10170346)
- [外部モニタにメニューバーとドックを表示… - Apple コミュニティ](https://discussionsjapan.apple.com/thread/10082404)
