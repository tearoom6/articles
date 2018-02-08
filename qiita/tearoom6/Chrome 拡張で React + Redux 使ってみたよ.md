
最近、不動産広告のキャッチコピーを見るのがマイブームな tearoom6 ですw


## 概要

さて、2年ぶりくらいにフロントエンドをキャッチアップした内容を、備忘録を兼ねて書き綴ります。というか、備忘録に過ぎません・・・。。
タイトルと裏腹に内容が広範になっております。メインとしては React + Redux の記事になります。誤りがあれば、ご指摘頂けると嬉しいですmm

材料として、2年前くらいにオレオレ流 Angular1 で作った Chrome 拡張を同一仕様で書き直すことにしました。Google Drive のファイル・フォルダに簡単にアクセスする機能をもったもので、書き直した後のソースコードを以下に置いています。

<https://github.com/tearoom6/QuickDrive2>

技術的には、以下のようなものを使用しています。
一応、 Chrome 拡張ならではのところは、デバッグがやりづらいのと、Build スクリプトを結構カスタムで作っているところとかですかね〜

- ES2015
- Yarn 0.16.1
- React 15.3.2
- Redux 3.6.0
- webpack 1.13.3
- gulp 3.9.1
- babel 6.5.2


## ES2015

JS の2015年版仕様。もうこれについては、巷に情報が溢れているので、あえて細かく書くことはないでしょう。
他の選択肢としては TypeScript / CoffeeScript を使う等があるかと思います。
また [Flow](https://flowtype.org/) を使った型チェックを行うのもありですが、この程度の規模ならまず要らないかなと。

### 行末に`;`を付けるか否か問題

JS の仕様的には、 ES2015 であろうと、`;`を付けて、文を分けるのが安心安全なはずです。ただ、普段 Ruby とか Python を書いていると、この`;`は鬱陶しく感じてくるんですよね。
改行を挟んでいる場合、`;`を明示的に付けなくてもいい場合が圧倒的に多いので、ここに`:`付ける派と付けない派が生まれるわけです。
Redux のオフィシャルチュートリアルは付けていないですね。ちゃんと分かって付けない分にはいいんじゃないでしょうか。ということで、今回は付けずに書いてみました。

### Arrow 関数

ES2015 で書けるようになった Arrow 関数は、 JS の `function` 書きまくる問題から開放してくれる素敵な存在です。
ただし、注意しなければならないのが、 Arrow 関数と通常の関数は全く同じではないという事実です。というのは `this` の意味が両者で異なるというもので、以下のように従来よく使われた jQuery を使った `$(this)` としてイベントの発生源を得るコードは、通常の関数でなければ機能しないという事になります。(あるいは、このような書き方は避けるべきか...)

```javascript
$('#item-list').scroll(function() {
  const scrollTop = $(this).scrollTop()
  //...
})
```

### Promise

Redux との絡みで、 非同期処理部分で Promise オブジェクトを使う箇所がありました。ベーシックな構成は以下のようになります。

```javascript
return new Promise((resolve, reject) => {
  // ... (何らかの時間のかかる処理) (API呼び出しなど)

  // コールバックの中で resolve (正常系) or reject (異常系) を呼び出す
  // 引数複数渡したいときは配列に入れる
  resolve(params)
  reject(params)
}).then( (params) => {
  // 正常系の場合
  // resolve が呼び出された時点で、 then 内部が呼ばれる
  // 引数も受け取れる
}).catch( (params) => {
  // 異常系の場合
  // reject が呼び出された時点で、 catch 内部が呼ばれる
  // 引数も受け取れる
})
```


## Yarn

package.json がそのまま使えるので、 NPM からの移行は非常に簡単でした。 Yarn の方が速いので、基本的に乗り換えて良いのではないかと思います。
ただ、依存性の問題か、 gulp-imagemin というモジュールがうまく機能せず、 Yarn のバージョンを上げると解決したということがありました。安定するまでは移行は慎重に行ったほうがいい場合もあるかもしれません。

<https://github.com/sindresorhus/gulp-imagemin/issues/232>


## Build (gulp + webpack + babel)

Build スクリプトは Chrome 拡張機能特有のクセのある処理も行いたいため、汎用性のある gulp を引き続きベースに利用しました。
(単純に通常のwebアプリで React 使うには [create-react-app](https://github.com/facebookincubator/create-react-app) を使うのが簡単だと思います)

JS のビルドに関しては、後述の CSS Modules を使いたいなどの理由もあり、 webpack に任せることに。gulp から webpack を呼び出すために gulp の webpack プラグインである webpack-stream を利用しました。

また、 ES2016 ベースのコードをビルドするため、 webpack の Babel プラグインである babel-loader を使いました。 JSX を含む React コードのコンパイルも Babel に任せました。

さらに webpack で CSS Modules (CSS の JS での import) を利用するため、 style-loader + css-loader を使いました。

- [gulp 設定](https://github.com/tearoom6/QuickDrive2/blob/master/gulpfile.js)
- [webpack 設定](https://github.com/tearoom6/QuickDrive2/blob/master/webpack.config.js)
- [babel 設定](https://github.com/tearoom6/QuickDrive2/blob/master/.babelrc)

まとめると、こんな感じ。

```
gulp
  ->webpack (webpack-stream)
    ->babel (babel-loader)
      ->babel-preset-es2015
      ->babel-preset-react
    ->CSS Modules (style-loader + css-loader)
```


## React + Redux

平たく言えば、 React は web アプリを再利用可能な部品に分けて整理して書くもの、 Redux は Flux アーキテクチャに基づいてデータの流れを一方向に流すことでデータの整合性を保つもの、でしょうか。別に React や Redux でなくてもいい訳ですが、一応代表選手ということでこれらを使うことにします。

### ディレクトリ構成

まず、ディレクトリ構成を以下のように分けました。

- actions
- reducers
- components (Presentational Components = Component の見た目)
- containers (Container Components = Component の動き)

極論を言えば、 JS のファイルは分けずに1つでもいいわけですが、ファイルを分けることで役割がはっきりして見通しが良くなります。

上記の分類方法のうち、特に components と containers は必ずしも分けなくてもいいかなーとは思いましたが、分けておくことで "How things look" と "How things work" がはっきり区別できる効果はありそうです。

> References

- [Presentational / Container componentの分離 - react-redux.connect()のつかいかた](http://qiita.com/yuichiroTCY/items/a3ca7d9d415049d02d60)
- [Container Components](https://medium.com/@learnreact/container-components-c0e67432e005)
- [Presentational and Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)


では、1つ1つみていきます。

### actions

Redux アプリケーション上で起こりうる何らかの状態変化 (action) を定義します。action 関数の戻り値には、type 属性を持ったオブジェクトを返します。(というか、このオブジェクト自身を action と呼びます)
type 属性以外にも任意のデータを action に詰め込むことが可能です。

```javascript
const showItems = (itemType, items) => {
  return {
    type: TYPE_SHOW_ITEMS,
    itemType,
    items
  }
}
```

action 関数の戻り値 (つまり action) を `dispatch` の引数にして呼び出すことで、 action を引き起こすことが可能です。

```javascript
dispatch(showItems(itemType, items))
```

action は後述の reducer で受け取ることができます。

非同期 action を定義する場合には、後述のミドルウェアとして、[redux-thunk](https://github.com/gaearon/redux-thunk) を利用するのがデファクトスタンダードになっているようです。
すると、 action オブジェクトを返す代わりに、以下のように dispatch を引数を持つ関数を返す感じで書けます。

```javascript
function asyncAction() {
  return (dispatch) => {
    // ... (非同期処理)
    // callback で dispatch を呼び出す
    dispatch(increment());
  };
}
```

### reducers

reducers は state (現在の状態) および action を受け取って、次の状態を返す関数として定義します。 action の前後で、状態がどう変化するかを定義するわけです。
reducers は複数定義可能で、それぞれは、単一の state に対してどう変化するかだけを記載すれば OK です。たとえば、ボタンを押しているかどうかとか、表示しているアイテムのリストとか、それらが1つ1つ state として定義されることになります。

```javascript
const items = (state = [], action) => {
  switch (action.type) {
    case actions.TYPE_SHOW_ITEMS:
      return action.items

    case actions.TYPE_SHOW_ADDITIONAL_ITEMS:
      return state.concat(action.items)

    default:
      return state
  }
}
```

複数定義した reducers は、最後に `combineReducers` でまとめ上げて、`createStore` で Store (全ての状態を管理するところ) に登録します。

```javascript
const reducer = combineReducers({
  items,
  activeItemType,
  resetAuthDisabled,
  isLoading
})

const store = createStore(
  reducer,
  applyMiddleware(...middlewares)
)
```

### components

Component の見た目を定義するところで、ようやく React を使った部分になります。 JSX による記述をするのはここになります。

React と Redux の連携を簡単に行うため [React Redux](https://github.com/reactjs/react-redux) を用いることにします。

この場合、もっとも単純な React Component は単なる関数として定義可能です。

```javascript
const Menu = ({ [properties...] }) => (
  // ... JSX
)
```

ただし、 `componentDidMount` などのライフサイクルフックを利用したい場合は、 `React.Component` を extends したクラスとして定義します。

```javascript
class ItemList extends React.Component {
  componentDidMount() {
    // ... 初期化処理
  }

  render() {
    const { [properties...] } = this.props
    return (
      // ... JSX
    )
  }
}
```

Component が持つプロパティの型等について定義することもできますが、これは必須では無さそう。

```javascript
Menu.propTypes = {
  onMenuClick: PropTypes.func.isRequired,
  active: PropTypes.bool.isRequired,
  children: PropTypes.node.isRequired
}
```

<https://facebook.github.io/react/docs/typechecking-with-proptypes.html>

### containers

containers では、 Component の動きの部分を組み合わします。
`mapStateToProps` および `mapDispatchToProps` の2つの関数 (どちらか片方でも良い) を定義して、これを `connect` により先程定義した component と組み合わせてやります。

```javascript
const SearchBoxSet = connect(
  mapStateToProps,
  mapDispatchToProps
)(SearchBox)
```

`mapStateToProps` は、Store から state を受け取って、 Component の状態に反映を行う処理、 `mapDispatchToProps` は逆に Component でイベントが起こった場合に、 `dispatch` を呼び出して action を発生させる処理になります。

なお、定義した components / containers はさらに階層的に組み合わせて上位の components / containers を作り、最終的に最上位の component を render します。

### Middlewares

Redux におけるミドルウェアは、いわゆる Plugins 的な、便利機能を提供してくれる存在です。
Store を生成する時に一緒に渡してあげます。
今回は、先述した thunk の他、 dev 環境の場合に、[redux-logger](https://github.com/evgenyrodionov/redux-logger) を入れて、開発時のデバッグをやりやすくしました。

```javascript
const middlewares = [thunk]
if ('production' !== process.env.NODE_ENV) {
  const logger = createLogger({
    level: 'info',
    duration: true,
    diff: true
  })
  middlewares.push(logger)
}
const store = createStore(
  reducer,
  applyMiddleware(...middlewares)
)
```

### 備考

React / Redux で

- `Adjacent JSX elements must be wrapped in an enclosing tag` というエラーが起きた場合、"Component のトップレベル (一番親となる階層) の要素を一つにする必要がある" という制約に違反しているので、修正する必要があります
   - [Reactで「Adjacent JSX elements must be wrapped in an enclosing tag」となる場合の対処](https://utano.jp/entry/2016/07/react-adjacent-jsx-elements-must-be-wrapped-in-an-enclosing-tag/)

- エラーではないけど、 `Each child in an array or iterator should have a unique "key" prop` という warning が出る場合、Component 内のリスト要素に一意の `key` 属性を持たせていない場合に出力されるので、 key 属性を追加してあげます
   - レンダリングの際のキャッシュ利用のために使われるみたいです
   - [Lists and Keys](https://facebook.github.io/react/docs/lists-and-keys.html)

本番環境にする！

- NODE_ENV をセットせずに使うと、以下のような warnings が出ました。

  ```
  Warning: It looks like you're using a minified copy of the development build of React. When deploying React apps to production, make sure to use the production build which skips development warnings and is faster. See https://fb.me/react-minification for more details.
  ```

- [gulp-env](https://www.npmjs.com/package/gulp-env) と [Webpack DefinePlugin](https://webpack.github.io/docs/list-of-plugins.html#defineplugin) を使って、本番ビルド時には NODE_ENV を `production` にセットするようにしました。
- [React - Use The Production Build](https://facebook.github.io/react/docs/optimizing-performance.html#use-the-production-build)


ナウいやり方に移行しきれなかったポイント

- 今回 jQuery や Bootstrap を従来通り使ったのですが、 ES2015 の import の枠組みではやらず、 CDN を使って global に load しました。できないことはないんでしょうが、結構大変そうなのを感じたので、そこは折れました。
- また、 [gapi](https://developers.google.com/api-client-library/javascript/) も初期化処理の中の load 時に onload パラメータとして global なコールバック関数を指定する必要があり、これは `window` 配下に置くことで global スコープに配置しました。


## CSS Modules

React によって HTML と JS を含めた Component 化が達成できているのだから、そこに CSS も含めたいと思うのは当然です。
CSS のファイルをインポートを JS からできるようにするのが CSS Modules という考え方です。
他の選択肢としては、 BEM や SMACSS などの考え方でスタイルを整理したり、 Sass や Less でネストさせてスタイルを整理するなどがあるかと思いますが、 CSS Modules は React との親和性が高いんじゃないかと。

先述した通り、 webpack で CSS Modules 実装をビルドするには css loader (および style loader) を用います。
Browserify で CSS Modules 実装をビルドするには css-modulesify を用います。

> References

- [ReactコンポーネントのスタイルをCSS Modulesで設計する方法](http://post.simplie.jp/posts/80)
- [CSS Modules Welcome to the Future](https://glenmaddern.com/articles/css-modules)
- [CSS Modules Webpack Demo](https://css-modules.github.io/webpack-demo/)
- [CSS Modules Browserify Demo](https://css-modules.github.io/browserify-demo/)

(画像とかもこの考えでやる方法がきっとあるんだろうけど、やる必然性が無いので調べてません(ｷﾘｯ))

