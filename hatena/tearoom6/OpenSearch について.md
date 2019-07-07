Chrome のアドレスバーに特定の文字列を打ち込んで、直後にスペースを入れると、下画像のように `Search XXX |` みたいな表示に切り替わることがあると思います。

![opensearch_address_bar.png](https://files.tearoom6.biz/9e838e26-68cb-4009-bfb4-4f24c9716ab6.png)

これは OpenSearch という仕様の Autodiscovery 機能に対応した Chrome の実装となっています。
OpenSearch 自体は Chrome 以外でも有効な標準仕様ですが、以下では Chrome を前提に記載します。

OpenSearch 対応しているサイトでは、 HTML ヘッダに以下のような `link` タグが設置されています。

![opensearch_link.png](https://files.tearoom6.biz/221cdf76-c050-477b-bcd1-d282f7023d24.png)

```html
<link rel="search" type="application/opensearchdescription+xml" title="RubyGems.org" href="/opensearch.xml">
```

この中身は以下のような感じです。

> opensearch.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<OpenSearchDescription xmlns="http://a9.com/-/spec/opensearch/1.1/" xmlns:moz="http://www.mozilla.org/2006/browser/search/">
  <AdultContent>false</AdultContent>
  <Description>Search RubyGems.org</Description>
  <Image width="16" height="16">data:image/x-icon;base64,AAABAAEAEBAAAAEAIABoBAAAFgAAACgAAAAQAAAAIAAAAAEAIAAAAAAAAAQAAAAAAAAAAAAAAAAAAAAAAACChr5+BAx//wQMgv8EDIT/BA2G/wQNiP8EDYn/BA2K/wQNi/8EDYv/BA2K/wQNif8EDYf/BAyG/wQMgv+Chr9+BAyA/wQMhv8EDIf/BA2K/wYPjP8HD47/BQ6P/wUNkP8FDZH/Bw+R/wcQkf8GD5D/BA2P/wQNjf8EDYz/BAyF/wQMhf8EDYr/BA2N/wQMj/8ABZD/AAWS/wQNlf8FDpb/Bg+X/wAHlv8AA5b/AAiX/wYPlv8EDZT/BQ2S/wQNjf8EDYr/BQ6P/wEKkv8TG5n/jZC5/4qNuf8QGJ7/BQ+d/wADm/9NU67/n6G//3t+tP8GD53/BA6c/wUOmv8FDpX/BA2Q/wYPlv8ACZj/GSGg/83NyP/Jycf/GiKl/wAEof8iK6v/yMjI/9/eyv9ITqz/AAWk/wcRpP8FDqL/BQ6e/wUNlv8GD5z/AAmf/xghpv/Gxsv/wsLK/wkTqv8AAqn/oKLG/+Hgz/9zd7n/AAKr/wgSrv8GD6z/BQ+r/wUPqP8FDp3/Bg+j/wAKpf8aI63/zc3S/9PT0/+Wmcz/o6bO/9ra1f/Ky9H/ISm4/wAEtP8JE7b/Bg+1/wYQtP8GD7H/BQ6j/wYQqv8ACqz/GyS1/9TU2f/d3dv/5eXZ/+Pi2P/d3Nj/5+bd/8nK2/8pMcT/AAm9/wgSvv8GEL3/BhC7/wUPqv8GELH/AAq0/xwlvP/e3+L/3Nzh/zQ8wv8iK8H/KjLC/5ue1P///+f/nqHb/wAFxv8KFMj/BxHH/wcRxP8FD7H/BxG4/wAKu/8cJsT/5+fq/+Hi6f8KFMr/AAHK/wAAyv80PdP///7u/8TG5v8GEdH/BxHR/wcS0f8HEc//BQ+3/wcRv/8ACsP/HifL/+vr8f/o6PH/OkLZ/yYv2f8rNdz/oqXr////9/+4uur/AAfa/wkU3P8HEtv/BxLZ/wUQvf8HEsb/AArK/yEq1P///vv////9/////P////z////9/////f////b/SFDl/wAH5v8LFeb/BxLm/wcS5P8GEMP/BxLM/wIN0v8ZI9n/tLbs/7u+7v+5vO//ur3v/7i77/+Znuv/OEHl/wAI6P8KFen/CBPo/wgT6f8IE+j/BhDI/wYR0v8HEtr/BhDf/wAH4f8AB+T/AAfl/wAH5f8AB+X/AALm/wAK6f8LFuj/CBTo/wkU6P8JFOj/CRTo/wYQzv8HEtz/BxHg/wcS5v8KFen/Chbp/wsW6f8LFun/Cxfp/w0Y6f8LF+j/CRTo/wkU6P8JFOj/CRXt/wkU6f+DiOh+BxHc/wcS5v8HE+j/CBPo/wgT6P8IFOj/CRTo/wkU6P8JFej/CRXo/wkV6P8JFej/CRXo/wkV6f+EivR+gAEAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAgAEAAA==</Image>
  <InputEncoding>UTF-8</InputEncoding>
  <OutputEncoding>UTF-8</OutputEncoding>
  <SearchForm>https://rubygems.org/search</SearchForm>
  <ShortName>RubyGems.org</ShortName>
  <Url type="text/html" rel="results" method="GET" template="https://rubygems.org/search?query={searchTerms}"/>
  <Url type="application/opensearchdescription+xml" rel="self" method="GET" template="https://rubygems.org/opensearch.xml"/>
  <moz:SearchForm>https://rubygems.org/search</moz:SearchForm>
</OpenSearchDescription>
```

ざっと見て分かる通り、当該サイト内を検索する場合の URL などの仕様が書かれています。
つまり、そのサイトを検索する際の仕様を外部アプリケーション向けに公開したものです。
Chrome はこの情報を読み取って記録し、該当ドメイン (もしくは上記の `ShortName`) を打ち込んで、直後にスペースを入れると、 OpenSearch 用の検索ボックスに仕立ててくれるというわけです。

Chrome の場合は、対象サイトが OpenSearch 仕様を公開していない場合でも、ブラウザの設定 (`chrome://settings/searchEngines`) から任意の検索仕様を設定することも可能です。

他にも、RSS や Atom フィードの形でその時点での検索結果を返す方式も仕様化されていたりします。 (実際には、各サイト独自方式で同様の機能を提供していることが多い気がしますが)
興味があれば調べてみてください。
