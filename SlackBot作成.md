# SlackのカレンダーBOTを作るまで

## 必要な準備物

* SlackのBot用チャンネル
* Webhook URL
* GoogleカレンダーID
* Google Apps Script

## 作業手順

1. SlackのBot通知したいチャンネルを作ります。
2. チャンネルの名前を var payloadのchannelに設定します。
3. 次にWebhookをSlackのアプリ経由から有効化しWebhook URLを取得します。  
   * その際インテグレーションの設定内にあるチャンネルへの投稿を設定しておきます。
4. コピーしたWebhook URLをvar url（一番下から4行目）に記載します。
5. GoogleカレンダーのIDをカレンダーの設定から取得し張り付けます。
   * 余談ですが、Workplaceの予定もGoogleカレンダーIDがあるので、自動通知可能です。
6. 完成したソースをGoogle Apps Scriptに張り付けて保存します。
7. 保存したGoogle Apps Scriptを実行するトリガーの時間を指定します。
8. 指定した時間になるとカレンダーに予定がある場合は通知、ない場合は通知は無しとなります。

```javascript
function myFunction() {
  
  var list = "";
  var s;

  s = listupEvent(""); // a.GoogleカレンダーのID
  if (s != "")  list += "\n■本日の予定\n" + s; // b.メッセージの見出し

  Logger.log(list);

  if (list != "") {
    var payload = {
      "text" : "おはようございます。\n本日のご予定をお知らせします。\n" + list, // c.メッセージの本文
      "channel" : "#", // d.Slackのチャネルの指定
      "username" : "カレンダーBot", // f.Botの名前
    }

    postSlack(payload);
  }
}

function listupEvent(cal_id)
{
  var list = "";
  var cal = CalendarApp.getCalendarById(cal_id);
  var events = cal.getEventsForDay(new Date());
  for(var i=0; i < events.length; i++){
    s = "";
    if (events[i].isAllDayEvent()) {
      s += Utilities.formatDate(events[i].getStartTime(),"GMT+0900","MM/dd  ");
    } else {
      s += Utilities.formatDate(events[i].getStartTime(),"GMT+0900","MM/dd HH:mm");
      s += Utilities.formatDate(events[i].getEndTime(), "GMT+0900","-HH:mm  ");
    }
    s += events[i].getTitle();
    Logger.log(s);

    list += s + "\n";
  }

  return list;
}

function postSlack(payload)
{
  var options = {
    "method" : "POST",
    "payload" : JSON.stringify(payload)
  }

  var url = ""; // g. Webhook URL
  var response = UrlFetchApp.fetch(url, options);
  var content = response.getContentText("UTF-8");

}
```
