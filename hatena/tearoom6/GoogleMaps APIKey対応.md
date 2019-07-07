# 状況

2018-07-16 から、 Google Maps API を呼ぶのに API key が必須になります。
ずっと以前から運用しているレガシーなサイトで Google Maps JS API (v3) を API key なしで使っていたので対応する必要がありました。

まず API key が必須になる要件ですが、単に Google Maps をサイト内に埋め込んでいる場合などは対象ではありません。
Google オフィシャルの Chrome 拡張 ([Google Maps Platform API Checker - Chrome ウェブストア](https://chrome.google.com/webstore/detail/google-maps-platform-api/mlikepnkghhlnkgeejmlkfeheihlehne?hl=ja)) があるので、これを入れて確かめてみましょう。

![google_maps_api_checker_info.png](https://files.tearoom6.biz/4d2e1e5a-c8fe-4997-8c55-0256cc24c3b4.png)

こんなふうに表示されれば、埋め込みで使っているだけなので対応は不要です。

![google_maps_api_checker_error.png](https://files.tearoom6.biz/f0669b4e-fc15-40b2-a869-f5c3dbeb48f0.png)

このようにエラーが表示されれば対応が必要です。

> References

- [サイト埋め込みGoogleマップが7月16日からAPIキー必須で有料に、ただし対応不要の場合も多いので慌てずに | 編集長ブログ―安田英久 | Web担当者Forum](https://webtan.impress.co.jp/e/2018/06/26/29696)


# 要件

API key の対応にはクレジットカードの登録が求められるなど、小規模なサイト運用者にはハードルが高い点も。
不安を取り除くため、課金条件を確認しておきましょう。

課金に関する話ですので、実際のサイトの本文を、以下に一部引用しておきます。 (Google のサイトでは日本語訳もちゃんと用意されています)

- Starting July 16, 2018, a new pay-as-you-go pricing plan will go into effect for Maps, Routes, and Places. This new plan gives you more flexibility and control over how you use our APIs: You can use as much or as little as you need and only pay for what you use each month. We would also like to highlight that beginning July 16th, we’re changing the pricing for our Maps, Routes, and Places products.
- Starting July 16, 2018, when you enable billing, you get $200 free usage every month for Maps, Routes, or Places. Based on the millions of users using our APIs today, most of them can continue to use Google Maps Platform for free with this credit.
- You only pay for what you use. You can review rates and access your spending any time in your [Google Cloud Platform Console](https://cloud.google.com/console/google/maps-apis/overview), where you can also set daily quotas to protect against unexpected increases. You can also set billing alerts to receive email notifications when charges reach a preset threshold determined by you.
- In addition to the $200 monthly free credit, all users get:
   - Free Maps usage for iOS, Android, and Embed (for displaying maps only)
   - Free Maps URLs
- Even though the first $200 a month is free, we ask for your credit card or billing account to cover any amount you spend over this free credit.

僕の英語力が正しければ、要約すると 2018-07-16 からクレジットカード登録が必要な API Key 取得が必要になるけど、毎月 $200 の無料使用枠が付いてくるから、大抵の場合、今までどおり無料で使えるはずだよ。超過が心配な場合は GCP コンソールで使用制限の設定をしておいてね、ということ。

課金要件に引っかかっても私の方では保証できないので、実際にサイトを見て確かめておいてください。

実際の利用量に対する課金金額は [Pricing Calculator](https://mapsplatformtransition.withgoogle.com/calculator) で確認できます。

> References

- [Pricing & Plans | Google Maps Platform  |  Google Maps Platform  |  Google Cloud](https://cloud.google.com/maps-platform/pricing/)
- [Understanding Billing for Maps, Routes, and Places  |  Google Maps Platform  |  Google Developers](https://developers.google.com/maps/billing/understanding-cost-of-use)
- [User Guide | Google Maps Platform  |  Google Maps Platform  |  Google Cloud](https://cloud.google.com/maps-platform/user-guide/)


# 対応

それでは実際の対応です。

まず [こちらのページ](https://cloud.google.com/maps-platform/) から API key の取得を開始してください。

1. 使用する機能を選択します (Maps / Routes / Places)
2. 使用する Google API のプロジェクトを選択します (プロジェクト未作成の場合は作成してください)
3. Google の billing account を登録します (クレカの登録が必要です)

API key が作成されたら、 [こちら](https://console.cloud.google.com/apis/credentials) で作成された key を選択し、 Key restrictions の設定を必ず行ってください。

対象のプロジェクトを選択したら、 [API keys] のところに作成した API key があると思うので、これの名前のところをクリックして、次の画面で Key restrictions のところを編集します。

今回のように JS API で使うということであれば

- Application restrictions で "HTTP referrers (web sites)" を選び `https://hogehoge.com/*` のように使いたいページの URL を指定
- API restrictions で "Maps JavaScript API" を選択

![google_maps_api_key_restriction.png](https://files.tearoom6.biz/bd032865-1ca7-419a-9ad7-bea30b9d36b4.png)

これをやっておかないと、 API key は外部にさらすことになるので、悪用されかねません。

つづいて、 Quotas (使用量制限) の設定をします。この制限を超過すると、マップは表示されないようになりますが、 $200 の範囲内に収めておけば課金はされなくなります。もちろん、超過しそうな場合で、課金に値するし、課金する余裕もあると思ったら、制限値は緩めておいてくださいね！

[GCP console](https://console.cloud.google.com/google/maps-apis/apis/maps-backend.googleapis.com/quotas) で対象プロジェクトを選択して、デイリーでの map load 上限値などを設定します。

![google_maps_api_quotas.png](https://files.tearoom6.biz/a4e5702c-4b9f-4ece-838f-eb9bc2b95305.png)

最後に API key を API コール時に渡すようにします。
`script` タグで読み込んでいる場合は、 `key` パラメータで API key を渡します。

```html
<script src="https://maps.googleapis.com/maps/api/js?key=API_KEY&callback=initMap"
        async defer></script>
```

JS の Google API Client Library を使用して load している場合は、次のように渡します。

```javascript
google.load("maps", "3", {
  "other_params" : "libraries=weather&sensor=false&key=API_KEY"
});
google.setOnLoadCallback(function() {
  //...
})
```

設定反映後、サイトにアクセスして、このように API checker の表示が SUCCESS に変われば OK です！

![google_maps_api_checker_success.png](https://files.tearoom6.biz/9ebdb21a-7950-475b-954c-d67aeec242e6.png)

> References

- [Get API Key  |  Maps JavaScript API  |  Google Developers](https://developers.google.com/maps/documentation/javascript/get-api-key)
- [Getting Started  |  API Client Library for JavaScript  |  Google Developers](https://developers.google.com/api-client-library/javascript/start/start-js)
- [How to set an API key when using Google's script loader? - Stack Overflow](https://stackoverflow.com/questions/14410167/)
