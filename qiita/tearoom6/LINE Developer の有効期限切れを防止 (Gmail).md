
無料アカウントで LINE Developer に60日間ログインしていないと、開発者アカウントが有効期限切れになってしまうようで、 "[WARNING] RE-AUTHENTICATION for LINE developers site" というタイトルのメールが届くみたい。。

メールに記載された URL を叩くだけで有効期限が伸びるんだけど、2-3回届くっぽいメールを見逃したら、終わりだよね・・・ってことで、ちょうど Gmail を使っているので、 Google Apps Script で確実に有効期限を伸ばすスクリプトを書いてみた。

Google Drive で [新規] > [その他] > [Google Apps Script] を作成。以下のコードをまるっとコピって、 `SLACK_POST_URL` の値である Slack の webhook token を書き換えてください。 (Slack 通知しない場合は、 `postToSlack` を呼んでいるところをコメントアウトでもok)

それで、一旦 [実行] > [triggerMonitor] を実行します。認可を求められるので、おｋします。そして、 [リソース] > [現在のプロジェクトのトリガ] で新規トリガとして、デイリーでどの時間帯でもいいので [triggerMonitor] を呼ぶようにすれば、これで安心して夜も寝れます。(｀･ω･´)

```javascript
//
// re-authenticate_line_developer.gs
//
// Extend the expiration date of the LINE developer by accessing URL of a notification mail.
//
// @author tearoom6 2016/12/15-
//

var SLACK_POST_URL = 'https://hooks.slack.com/services/xxxxxx/xxxxxxxxxxxxxxxxxxxx';
var SLACK_NOTIFY_CHANNEL = '#general';
var SLACK_NOTIFY_FROM_USER = 'Gmail';
var SLACK_NOTIFY_TO_USERS = ['tomohiro'];

/**
 * triggerMonitor()
 */
function triggerMonitor() {
  monitorMails();
}

/**
 * monitorMails
 */
function monitorMails() {
  var threads = GmailApp.search('is:unread from:(do_not_reply@line.me) subject:(RE-AUTHENTICATION)');
  for (var i = 0; i < threads.length; i++) {
    var messages = threads[i].getMessages();
    for (var j = 0; j < messages.length; j++) {
      if (! messages[j].isUnread()) {
        continue;
      }
      var subject = messages[j].getSubject();
      var messageBody = messages[j].getPlainBody();
      var receiveDate = messages[j].getDate();
      var datetime = receiveDate.getFullYear() + '/' + (receiveDate.getMonth() + 1) + '/' + receiveDate.getDate()
        + ' ' + receiveDate.getHours() + ':' + receiveDate.getMinutes() + ':' + receiveDate.getSeconds();
      Logger.log(datetime + ' Subject: ' + subject);
      Logger.log('Body: ' + messageBody);

      var foundUrls = findUrlsInText(messageBody);
      var extendLink = null;
      for (var k = 0; k < foundUrls.length; k++) {
        Logger.log('Found url: ' + foundUrls[k]);
        if (foundUrls[k].indexOf('re-authenticate') >= 0) {
          var extendLink = foundUrls[k];
        }
      }
      if (extendLink) {
        Logger.log('Extend url: ' + extendLink);
        UrlFetchApp.fetch(extendLink);
      }

      messages[j].markRead();
      postToSlack("Extend the expiration date of the LINE developer:\nmail:" + threads[i].getPermalink(), SLACK_NOTIFY_CHANNEL, SLACK_NOTIFY_FROM_USER, SLACK_NOTIFY_TO_USERS);
    }
  }
}

/**
 * A utility function to find all URLs - FTP, HTTP(S) and Email - in a text string
 * and return them in an array.  Note, the URLs returned are exactly as found in the text.
 * http://stackoverflow.com/questions/4504853/
 *
 * @param {string} text - the text to be searched.
 * @return {array} an array of URLs.
 */
function findUrlsInText(text) {
  var source = (text || '').toString();
  var foundUrls = [];
  var matchArray;

  // Regular expression to find FTP, HTTP(S) and email URLs.
  var regexToken = /(((ftp|https?):\/\/)[\-\w@:%_\+.~#?,&\/\/=]+)|((mailto:)?[_.\w-]+@([\w][\w\-]+\.)+[a-zA-Z]{2,3})/g;
  // Iterate through any URLs in the text.
  while( (matchArray = regexToken.exec( source )) !== null ) {
    var token = matchArray[0];
    foundUrls.push(token);
  }

  return foundUrls;
}

/**
 * postToSlack(message, channel, fromUserName, toUserNames)
 *
 * @param {string} message - the messaage to be post.
 * @param {string} channel - the channel to post message to.
 * @param {string} fromUserName - the user name to be displayed as the sender.
 * @param {array} toUserNames - the user names to notify to.
 */
function postToSlack(message, channel, fromUserName, toUserNames) {
  var toAnnotations = '';
  for (var i = 0; i < toUserNames.length; i++) {
    toAnnotations += ('@' + toUserNames[i] + ' ');
  }
  var jsonData = {
    'channel' : channel,
    'username' : fromUserName,
    'text' : toAnnotations + message
  };
  var payload = JSON.stringify(jsonData);

  var options = {
    'method' : 'post',
    'contentType' : 'application/json',
    'payload' : payload
  };
  UrlFetchApp.fetch(SLACK_POST_URL, options);
}
```

