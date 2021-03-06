ここ最近忙しく、久しぶりの投稿になってしまいました。
ちょっと前から書きたかった布教用の記事を書きます。
みんなが IaC すれば、世界がちょっと happy になります。。

## IaC って何？

ここ 2-3 年、僕が注力した分野の一つが IaC です。

IaC は Infrastructure as Code の略です。
つまり、インフラ構成をコードで管理しましょう、っていうことですね。

その対極にあるのは、インフラ構成を CLI や GUI を使って、手動で作成・管理するというものです。
その場合は、しばしば手順書をドキュメントとして残すことになります。

## こんなにあるよ！ IaC のメリット

IaC のメリットとして、パッと思いつくだけでも、以下のようなものがあります。
(何か語呂合わせにしたいけど、思いつかないのでいいや...)

- Easy to Reproduce  (再現容易性)
- Easy to View       (閲覧容易性)
- Easy to Destroy    (破棄容易性)
- Easy to Share      (共有容易性)
- Easy to Transplant (移植容易性)
- Easy to Find       (検索容易性)
- Easy to Review     (レビュー容易性)
- Easy to Rollback   (ロールバック容易性)
- Easy to Refactor   (リファクタ容易性)

GUI でポチポチやった時に、上記のいずれもが、かなり難しいことはすぐ理解していただけるかと思います。

特に一番最初の、 "再現性" は非常に重要です。

どうせ、1回作ったら同じものは作らないんだから、 GUI でぱぱっと作っちゃったほうが早いじゃん、という意見。
その断面だけ見たら、至極まっとうで正しいのですが...

でも、僕が大事にしているエンジニアの鉄則によると、そもそもその前提が間違っています。 **1度あることは、10度ある** のです。
**似たような構成は、絶対にまた作ります**。会社組織の中でやるなら、あなたでなくても、他の誰かがきっと似たことをやるでしょう。いわんや、世界規模で言えば...

コード化によるオートメーションがもたらす利益こそが、 IaC がもたらす最大のメリットです。

## IaC の進化

上記したように、さまざまなメリットがある IaC ですが、その普及を妨げるダメな点が (主観ですが) 一つだけあります。
**IaC 関連のツールが成熟していない** 点です。学習コストが基本的に高いですし、慣れても、どうしてもある種の使いづらさがあります。

インフラ製品を作っている人は、基本的には IaC のことまで考えて、インフラを作っていません。
IaC 関連ツールを作る人は、常に **既に存在するインフラを、コードの形で記述できるようにする** という、後追いの作業を強いられているのです。
それ故に使いにくさが出てくるのだと思います。

ツールの進化について、私の知る範囲で、ちょっと歴史をさかのぼってみたいと思います。
(多々認識がずれているところがあるかもしれません。間違っていたら、ご指摘くださると嬉しいです)

### 構成管理ツール

もともと、インフラ構成といえば、ハードウェア周りの構築は仕方ないから手動でやって、物理サーバマシン上の (主に) Linux OS の設定と、そこで動作するミドルウェアなどのインストール・設定のことを指していました。故に、基本的には、ツールもその部分の面倒を見てくれるものが中心でした。そのようなツールには、以下のようなものがあります。

- CFEngine
- Puppet
- Chef
- Itamae
- Ansible

これらのツールの誕生と並行して、インフラ構成における最も大きな変化が進んでいきます。仮想化技術の登場です。

### 開発環境での仮想化技術の利用

仮想化技術の利用例の一つが、開発環境を仮想化技術で構築するものです。

一昔前は、これの代表例として Vagrant が VirtualBox との組み合わせでよく使われていました。ローカル環境で仮想マシンを立てようということです。
複雑な構成の場合、これに、さらに Chef や Ansible などを組み合わせて使うこともありました。

しかし、コンテナ技術である Docker が新たに誕生して以降は、開発環境の IaC には (特に Linux 環境において軽量な) Docker が使われることが増えてきました。
Docker は、今では、持ち運び容易かつ軽量な環境として、開発環境のみならず、本番環境でも普通に運用されるようになっています。

### クラウド環境での仮想化技術の利用

仮想化技術の利用例のもう一つ大きなものが、 AWS や GCP などのクラウド環境の登場です。

これによって、従来のインフラは劇的に拡張されることになりました。ハードウェア、ネットワーク周りも、仮想化技術によって、ソフトウェアによる制御が可能になり、また、各社が便利なマネージドサービスの数々を提供することによって、従来ミドルウェアとしてインストールして使っていたような類のものも、インフラ構成のパーツとして管理可能になりました。

![iac_cloud_resources.png](https://files.tearoom6.biz/5f797e27-a7f2-41f5-b465-7f851f51e146.png)

ベースとして OS (仮想マシン) があって、その上でゴニョゴニョ構築する、という世界を離れて、もう一段、抽象化した世界の上で、インフラを構成できるようになったのです。

このクラウド環境に特化したプロビジョニングツールも、サービスごとに出てくるようになりました。

- CloudFormation (AWS)
- Cloud Deployment Manager (GCP)
- Azure Resource Manager (Azure)
- Terraform

最後の Terraform だけは、特定のクラウドサービスに特化するものではなく、サービス横断的に利用できるものになります。

そして、1年くらい前には AWS から新手のツールとして CDK がリリースされました。

CDK は、基本的には CloudFormation をラップしたものではあるんですが、もっと良質にクラウド上のリソースが抽象化されており、さらに、完全にプログラミング言語によるコードとして記載することができます。今の僕の推しツールになります。 [CDK for Terraform](https://www.hashicorp.com/blog/cdk-for-terraform-enabling-python-and-typescript-support) なるものも出てきていますので、 AWS 以外のリソースも構築できるようになりそうですし、今後も要注目です。

### 命令的な記述と宣言的な記述

プロビジョニングツールの大きな流派の違いに、命令的 (逐次実行的) に記述するか、宣言的に記述するか、というものがあります。

コードの記述が命令的 (Imperative/Procedural) であるというのは、インフラ構成が、最新形になるまでの過程を記述するということです。
Rails のマイグレーションに例えるなら、各 migration ファイルだと言えますね。

一方、コードの記述が宣言的 (Declarative) であるとは、過程はどうでもいい(!?)、結果としてどうなるのかを記述するということです。
Rails のマイグレーションに例えるなら、 schema.rb, structure.sql です。

この宣言的な記述とおそらく関係があるのが Immutable Infrastructure という考え方です。
ざっくりといえば、この考え方は、インフラの構成は、簡単に破棄・同じものを再構築 (冪等性) できるようにしておき、構成に変更を与えたい場合は、作り直したものに切り替えればいい (Blue-Green Deployment なんて呼ばれる)、ということです。まさに IaC がピタッとはまる考え方ですね。

その考え方で言えば、まさに最新のインフラ構成さえ分かればいいのだから、逐次処理的に書く必要はなくて、宣言的に書けば十分だし、その方がシンプルなわけです。
順を追ってコードを追わなくても、宣言的に書いておけば、「今、どういう構成になっているのか」 (現在のインフラ構成のスナップショット) が、一目瞭然なのです。

ただ、宣言的なコードの場合には、一つ大きな問題があります。
現実的には、コストが高すぎて、なかなか全部を破棄して、再構築したものに切り替える、みたいなことはできないケースがあることです。

Database のマイグレーションの場合も、まさか、毎回全てのデータを移したコピーを作ってから、そちらに切り替えるみたいな暴挙はできません。
それと同じく、複雑なクラウド環境などの場合には、わざわざ全てのリソースを作り直すみたいなことは、現実問題できないことです。
(もちろん、うまく部分部分に切り出せると、その各部分で Blue-Green Deployment のようなことをするという手もありますが)

そうした場合に問題になるのが、宣言的な書き方しか用意していないツールでは、各 "snapshot" 間を移行するにはどうすればよいのかを、ツール側が解析して、解読しなければいけないという点です。そこはツール側任せの部分になります。場合によっては、ここで移行が難しかったり、移行中にサービスのダウンタイムが発生してしまったりしてしまいます。

このツールの気持ちに立って、段階的な移行をかけられるかどうかが、宣言的記述の IaC ツールを使いこなす上で大事なポイントになってきます。
でも、考えてみてほしいのですが、 GUI でやる場合も、段階を追って、ダウンタイムが生じないように移行するのだから、そこはプロビジョニングツールを使う場合も同じです。
一度に変更できない場合は、何段階かに分けて変更を行った Pull Request を作成して、それらを順次デプロイすることで移行すればいいのです。

具体例がなくて、抽象的な表現になってしまいましたが。

でも、宣言的なプロビジョニングツールでも、 Rails のマイグレーションと同じく、差分を実装して、その結果として宣言的なコードが吐かれる、みたいな感じでもよいよね、っていうのは思ったりします。

> References

- [IaC(Infrastructure as Code) における Declarative vs. Imperative - Qiita](https://qiita.com/san-tak/items/9d9d8da4295e51f19653)
- [Why we use Terraform and not Chef, Puppet, Ansible, SaltStack, or CloudFormation | by Yevgeniy Brikman | Gruntwork](https://blog.gruntwork.io/7989dad2865c)
- [What is Mutable vs. Immutable Infrastructure?](https://www.hashicorp.com/resources/what-is-mutable-vs-immutable-infrastructure)

## 私の考える IaC の未来

IaC でいろいろ書いてきて、わたくし思うことがあります。
さまざまなシステムがあるけど、 **インフラ構成は、ベースの部分では、共通したものが使われることが多い** ということです。

アプリケーションコードもそういうところがあって、だからこそ、ライブラリやフレームワーク的なものが使われてきました。
インフラ構成も IaC が進めば、きっと同じような形になっていくでしょう。テンプレート的に使えるもの、部分的な形で取り込めるものがきっと出てくるのではないかと。
最近たまに見ますが、よく使われる構成の CDK のコードが公開されてくるだけでも、だいぶ助かる気がします。

(2020-10-05 追記) ちょうど Classmethod さんの記事で、ここで書いていたようなパターンが紹介されていましたので、掲載しておきます: [AWS CDK+Serverlessのアーキテクチャパターンの実装が勢揃い！CDK Patternsの紹介 | Developers.IO](https://dev.classmethod.jp/articles/cdk-patterns-introduction/)

まぁ、広く言えば (最近あんまり言葉としては使われませんが) SaaS, PaaS なども、コードとして設定管理できる場合は IaC の一種ですし、 Serverless Framework や Amplify, Firebase, Netlify などは、アプリケーションコードと絡めた形で、裏側でゴリゴリ、インフラを構築しますので、その意味では、既にそういうものは出ているのですが、でもそれらはアプリケーションのおまけに過ぎないので、僕の思っているのとちょっと違う。 (構築メインで、その後の保守はあんまり考えられていない気がする)

そういう形ではなく、インフラ面にも注力した、もうちょっとクレバーなプロダクトが出てくることを期待しています。いずれにしても、今は IaC は過渡期でしかないので、その覚悟は必要かなと思います。

## 蛇足: DaC

ちなみに、私は DaC (Documentation as Code, Docs as Code) も推進しています。完全オリジナルな造語だと思っていたら、ネット調べたら普通にあったです・・・
メリットは、だいたい IaC のそれと同じです。

ということで、この記事も https://github.com/tearoom6/articles で見れます。この記事、全く上手く書けた自信がないので、書き直したいってなった時に、差分が確認できたりします。

> References

- [Docs as Code — Write the Docs](https://www.writethedocs.org/guide/docs-as-code/)
