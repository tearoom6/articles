# Mysterious technologies vol.01

Learn hints to solve problems from breakthroughs of other fields.

日常の生活の中で、ごく身近に存在している、よく考えたらどうやってるんだろうっていう謎技術。

そんな謎技術を支えるテクノロジーから、現在の仕事における課題を解くためのヒントを探るシリーズ、第一回目です。


## 高層ビル建設 *-High building constructions-*

今や、東京のあちこちで見られるタワーマンションや、高層オフィスビル。
でも、ふと気になった。いったいどうやってこんな高いもの建ててるんだ?

このような高層ビルの建設中には、てっぺんの方に、たいてい何台かの大きなクレーンが乗っていることが多い。
どうやらこのクレーンが鍵になっているようだ。


### タワークレーン

クレーンの仕事について、改めて説明する必要はないだろう。
なにか重たい部材をクレーンのケーブルの先に引っ掛けて持ち上げる。

高層ビルの建設においても、クレーンによって部材を下から持ち上げて、組み立てていることは想像できる。
でも、そこでさらに2つの疑問が発生する。

- どうやってこんな大きなクレーンがビルの先っぽに乗っかってるの?
- ビルが完成した後、どうやってクレーンを撤去しているの?

それぞれについて調べてみると、なるほどー、と唸らせる、シンプルな工夫が行われていた。


### タワークレーンを上に上げていく方法

タワークレーンを高いところに持っていくための、一般的なやり方は、クライミングクレーン (climbing crane) というもの。
これは、クレーン自らが上に登っていく、というものである。

一般的なタワークレーンは、以下のように、柱部分のマスト (mast) と、腕部分のジブ (jib) に分かれている。

![mystech_01_crane_01.png](https://files.tearoom6.biz/ded92e6d-3eff-42fa-bb87-a3c9deb0eb96.png)

高層ビルのてっぺんにクレーンが乗ることになる、**フロアクライミング (floor climbing)** と呼ばれる方式では、以下の図のように、

1. クレーンの途中の部分と建物を固定して、マストを引き上げ (1-3) 、底の部分を持ち上げる
2. 今度は底を固定して、ジブ部分を引き上げる (4-6)

の2段階の工程によって、最終的に、クレーンを数フロア分引き上げる。

![mystech_01_crane_02.png](https://files.tearoom6.biz/b5a6958e-8cc9-4126-9686-33a6ee6a9546.png)

もしオフィスの窓から見える範囲で、高層ビルの工事なんかしていたら、きっとこの過程が確認できるはずだ。
もっとも上に登っていく速度が遅すぎて、タイムラプスとかで撮影しないとよく分かんないだろうけど。

このやり方では、マストが通った跡が、そのまま穴になって残るんだけど、それをどうするのかは調べた範囲では見つからなかった。
塞ぐこともできるんだろうけど、おそらく業務用のエレベータなどにして再利用することもあるんじゃないかななどと予想。

ちなみに、他にも方式があって、それぞれに長所・短所がある。

**マストクライミング (mast climbing)**

建物の外部でクライミングをする方式。
マストを自分で継ぎ足しながら、伸ばして、ジブ部分を徐々に上げていく。

当然、高さに制限が出てきてしまうのと、クレーンのための敷地が必要になるのがデメリット。
一方、後で述べるように、撤去の工程が比較的容易なのがメリット。

下の動画で、作業過程がよく分かる。

<iframe width="560" height="315" src="https://www.youtube.com/embed/qityGWgQaB0" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

**マストコラムクライミング (mast column climbing)**

建物の内部でマストクライミング、つまり、マストの継ぎ足しを行いつつ、マスト部分をそのまま建物の鉄骨構造の一部に組み込む。
フロアクライミングでは建物に穴が空いたままになるのを防ぐことができる。

下の動画なんかが、たぶんそれ。

<iframe width="560" height="315" src="https://www.youtube.com/embed/yWoxCnIxgys" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


### タワークレーンの撤去方法

マストクライミングの方は単純で、上に書いたのと逆の手順を行う。 (逆クライミング)
つまり、マストを1段ずつ自分で外しながら、ジブを徐々に下げていく。

フロアクライミングの方は、もうちょっと工程が多くて、

1. より小さなクレーンの部材を解体される方のクレーンで引き上げる
2. 引き上げた部材を使って、より小さなクレーンを組み立てていく
3. 一回り小さなクレーンが完成
4. その小さな方のクレーンを使って、大きい方のクレーンを解体
5. 解体したパーツを下に下ろしていく
6. 大きな方のクレーンの解体完了

という手順によって、クレーンのサイズを一回り小さくする。

![mystech_01_crane_03.png](https://files.tearoom6.biz/79e99369-be1c-4caf-97ea-04c71414e4e3.png)

これを1サイクルとして、徐々にクレーンのサイズを小さくしていき、最終的に各パーツが 80kg 未満くらいのサイズのクレーンにする。これらをエレベータやゴンドラに積み込んで下ろせば、撤去完了。

最後に東京スカイツリー建設時の、タワークレーンの建設・撤去の様子を収めた、貴重な動画を紹介して終わりにする。

これなんか見てると、途中で、フロアクライミングから、マストクライミングに切り替えているのがよく分かる。

<iframe width="560" height="315" src="https://www.youtube.com/embed/ACImkWdehXk" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


### まとめ

以上、タワークレーンの技術からは、以下の考え方を学ぶことができた。

- 大きな仕事は、小さく分けて考える

一体どうやったらできるのか検討もつかない大きな仕事も、小さなタスクに細かく分けて考えることで、案外実現できたりする。
一気に解決しようとするのではなく、 `これをやるにはこれができれば OK` と細かくタスクを breakdown していけば、正解に至る道が開けたりするものだ。

**他の "謎テク" シリーズの記事は [こちら](https://tearoom6.hateblo.jp/archive/category/%E8%AC%8E%E6%8A%80%E8%A1%93)**

> References

- [タワークレーンクライミング方法・組立解体の手順を写真画像で謎解く](https://tobisyoku.net/tower.htm)
- [タワークレーン - Wikipedia](https://ja.wikipedia.org/wiki/%E3%82%BF%E3%83%AF%E3%83%BC%E3%82%AF%E3%83%AC%E3%83%BC%E3%83%B3)
- [IHI運搬機械株式会社](http://www.iuk.co.jp/howto/t_crane.html)
- [解決！タワークレーンの謎｜タワークレーン特設サイト：大林組](https://www.obayashi.co.jp/towercrane/)
- [July 2013：特集「建設の素朴な疑問」| KAJIMAダイジェスト | 鹿島建設株式会社](https://www.kajima.co.jp/news/digest/jul_2013/feature/question1/index-j.html)
- [JCA](http://www.cranenet.or.jp/tisiki/kuraimingu.html)
- [取扱製品一覧｜大型クレーンのアウトソーシング｜産業リーシング株式会社](http://www.sangyo-leasing.co.jp/products/)
- [東京スカイツリー建設プロジェクト｜株式会社大林組 : GO! GO! TOKYO SKY TREE by OBAYSHI](http://www.skytree-obayashi.com/)
