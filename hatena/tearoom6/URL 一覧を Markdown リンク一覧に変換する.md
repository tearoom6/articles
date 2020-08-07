一時的に取っておいた URL 一覧を見やすい Markdown の形に変換したいというニーズが生じたため、サクッと行いました。

基本的には References に挙げた Gist を参考にさせていただきました。
ただ、元コードが CORS 制限回避のため [crossorigin.me](https://corsproxy.github.io/) という proxy ツールを使っていたんですが、こちらのサービスサイト (https://crossorigin.me/) がドメイン切れを起こしていたため、代替として [CORS Anywhere](https://cors-anywhere.herokuapp.com/) というツールを使うように変更しました。

モダンブラウザで新規 blank タブを開いた状態で DevTools を開き、 Console に以下のスニペットを打ち込めば、そのまま動きます。
応用すれば、 HTML リンク一覧や、 Facebook や Twitter みたいなカード形式での一覧とか、いろいろできそうです。

```javascript
// Only using native browser features (no jQuery).
// Uses `fetch`, `DOMParser` and `querySelectorAll`.

const urls = [
  // Replace them with what you wanna save.
  'https://dev.classmethod.jp/articles/cdk-over-21-lambda-create-error/',
  'https://github.com/ReactorKit/ReactorKit',
  'https://github.com/date-fns/date-fns',
];

const getTitle = async (url) => {
  return fetch(`https://cors-anywhere.herokuapp.com/${url}`)
    .then(response => response.text())
    .then(html => {
      const doc = new DOMParser().parseFromString(html, 'text/html');
      const title = doc.querySelectorAll('title')[0];
      return title.innerText;
    });
};

const convertToMarkdownLink = async (url) => {
  const title = await getTitle(url);
  return `[${title}](${url})`;
};

const links = await Promise.all(
  urls.map(url => convertToMarkdownLink(url))
);

console.log(links.map(link => `- ${link}`).join('\n'));
```

> References

- [Get title from remote HTML URL - without jQuery](https://gist.github.com/jbinto/119c3f0e5735ab73faaa)
- [crossorigin.me](https://corsproxy.github.io/)
- [corihudson/crossorigin.me: A CORS proxy for everyone.](https://github.com/corihudson/crossorigin.me)
- [Demo of CORS Anywhere](https://robwu.nl/cors-anywhere.html)
- [Rob--W/cors-anywhere: CORS Anywhere is a NodeJS reverse proxy which adds CORS headers to the proxied request.](https://github.com/Rob--W/cors-anywhere/)
- [mozilla/page-metadata-parser: A Javascript library for parsing metadata on a web page.](https://github.com/mozilla/page-metadata-parser)
