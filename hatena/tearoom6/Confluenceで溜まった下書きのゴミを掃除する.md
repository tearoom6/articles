Confluence を使う機会があるのですが、 **個人の感想** としては、 Confluence は GitHub と並んで、基幹サービスにしては、障害が起きることが多いという印象を持っています。

それで、ある時、 Confluence の Drafts (下書き) 一覧を覗いてみると、なんじゃこりゃ！

![confluence_drafts_before_cleanup.png](https://files.tearoom6.biz/0335d3a3-0a21-480f-9cc4-27942652abc5.png)

こんなふうに "Untitled" の下書きが軽く数えた感じ 100 個以上生成されています。

いや、絶対こんなたくさん下書き作ったことないし。断じてない！
基本的には、 **サービスやライブラリのバグを疑う前に、己の不肖を疑え、(その上でどうしてもサービス・ライブラリ側のバグっぽい場合は、調査の上、エスカレーションなり、プルリクエストなりを出せ)** というのが、できるエンジニアのモットーだと思ってるんですが、いや、今回に限っては、ほんとに断じて、こんな事した覚えないのです。。

きっと、ページが保存できなくなる障害が起きたときに、ページを開きっぱにしていて、こんな風にどんどんゴミができていったのでしょう。

それで、この微妙に手で消せそうな数。手動で消しちゃおうか、という誘惑にも駆られましたが、ここはもう一つのできるエンジニアの鉄則、 **1度あることは、10度ある** を思い出し、スクリプト書いて消すことにしました。ちょっと面倒でも自動でできるようにしておけば、それ自体が資産になって、次回以降同様の問題に素早く対処できます。

結果的に、以下のようなスクリプトを書きました。

- `USER_NAME` はあなたのログイン時の email
- `API_TOKEN` は https://id.atlassian.com/manage-profile/security/api-tokens から取得
- `BASE_URL`, `SPACE_KEY` は、対象スペースをブラウザで開いた場合に表示される URL からそれぞれ取得
   - `BASE_URL` は Confluence Cloud を使っているか、 Confluence Server を使っているか、によってちょっと形式が変わってくると思います

```sh
BASE_URL="https://foobar.atlassian.net/wiki"
USER_NAME="hogehoge@fugafuga.com"
API_TOKEN="HIMITSU1X1HIMITSU"
SPACE_KEY="~111111111"
TITLE="" # Empty string means "Untitled"

curl \
  -u $USER_NAME:$API_TOKEN \
  -H "Content-Type: application/json" \
  -X GET "$BASE_URL/rest/api/content?type=page&spaceKey=$SPACE_KEY&title=$TITLE&status=draft&limit=100&expand=body.storage,version" \
  | jq -c '.results[]' \
  | while read -r content;
do
  content_id=$(echo "$content" | tr -d '[:cntrl:]' | jq -r '.id')
  content_view_body=$(echo "$content" | tr -d '[:cntrl:]' | jq -r '.body.storage.value')
  editor=$(echo "$content" | tr -d '[:cntrl:]' | jq -r '.version.by.displayName')
  echo
  echo "ID: $content_id"
  echo "Body: ${content_view_body:0:30}..."
  echo "Editor: $editor"

  echo "Are you sure to delete this draft?"
  select answer in "Yes" "No"; do
    case $answer in
      Yes)
        curl \
          -u $USER_NAME:$API_TOKEN \
          -H "Content-Type: application/json" \
          -X DELETE "$BASE_URL/rest/api/content/$content_id?status=draft"
        break
        ;;
      No)
        break
        ;;
    esac
  done
done
```

最大100件しか取得してないので、もしそれ以上ある場合は、繰り返し実行してください。複数スペースにまたがっている場合も、その都度実行してください。

一応、用心深く、1つずつ、 `Body` や `Editor` を確認しつつ、削除するようにしました。基本的に、自分の編集した、内容が空の下書きのみを削除しました。
もし、えいやっっと一括削除したい場合は、よしなに改変してください、責任は問いません。。

ちなみに途中で `tr -d '[:cntrl:]'` ってしてるのは、 `jq` さんが、途中で (ある種の) 改行文字が入っている JSON を正しく処理してくれないため、一括して制御文字を取り除いている、というものです。1回、変数 `$content` にいれた上で、 `echo` で出力するタイミングで `\n` がこんな風に改行文字に変わっちゃうっぽいですね。どうにかできそうだけど、いったん雑に対処。

実行後は、このようにきれいスッキリ。明日からも気持ちよく仕事ができそうです！

![confluence_drafts_after_cleanup.png](https://files.tearoom6.biz/0869dee2-b64d-428b-931a-2bb6a997e784.png)

> References

- [Basic auth for REST APIs](https://developer.atlassian.com/cloud/confluence/basic-auth-for-rest-apis/)
- [Confluence REST API documentation](https://docs.atlassian.com/atlassian-confluence/REST/5.9.6/)
- [curl - Loop into JSON Array in Bash Script - Unix & Linux Stack Exchange](https://unix.stackexchange.com/questions/507026/)
- [bash - How do I prompt for Yes/No/Cancel input in a Linux shell script? - Stack Overflow](https://stackoverflow.com/questions/226703/)
- [shell - Invalid string: control characters from U+0000 through U+001F must be escaped using Bash? - Stack Overflow](https://stackoverflow.com/questions/52399819/)
- [jqでparse error: Invalid string: control characters from U+0000 through U+001F must be escapedが出るときの対応 - Qiita](https://qiita.com/re-sasaki/items/7d34109cf209a8ac753c)
