---
id: dist
{{if .Meta.Status}}status: {{.Meta.Status}}{{end}}
{{if .Meta.Summary}}summary: {{.Meta.Summary}}{{end}}
{{if .Meta.Author}}author: {{.Meta.Author}}{{end}}
{{if .Meta.Categories}}categories: {{commaSep .Meta.Categories}}{{end}}
{{if .Meta.Tags}}tags: {{commaSep .Meta.Tags}}{{end}}
{{if .Meta.Feedback}}feedback link: {{.Meta.Feedback}}{{end}}
{{if .Meta.GA}}analytics account: {{.Meta.GA}}{{end}}

---

# LINE Clova + enebular + Amazon Connect連携ハンズオン

## 問い合わせフローを作ろう！
### Amazon Connect電話番号取得

すでにAmazon Connectの電話番号を取得している前提で開始します。
まだ未取得な方はこちらから番号を取得しておいてください。

[https://ac-handson-00.netlify.com](https://ac-handson-00.netlify.com)

### 1-1. 問い合わせフローの作成
AWSにログインして、サービスから「Amazon Connect」を検索して、出てきたものをクリックします。

![s000](images/s000.png)

作成したインスタンスエイリアスをクリックします。

![s001](images/s001.png)

［管理者としてログイン］をクリックします。

![s002](images/s002.png)

左側メニューのルーティングから「問い合わせフロー」をクリックします。

![s100](images/s100.png)

［問い合わせフローの作成］をクリックします。

![s101](images/s101.png)

名前を「enebular-AmazonConnect」と入力します。

![s102](images/s102.png)

設定カテゴリにある「音声の設定」ブロックをドラッグアンドドロップして、ドロップしたブロックをクリックします。

![s103](images/s103.png)

言語は「日本語」でお好きな音声を選択してください。

![s104](images/s104.png)

エントリポイントと音声の設定ブロックを繋げます。

![s105](images/s105.png)

設定カテゴリにある「問い合わせ属性の設定」をドラッグアンドドロップします。

![s106](images/s106.png)

属性の設定を行います。「属性を使用する」を選択してください。

|項目|値|
|:--|:--|
|宛先キー|message|
|タイプ|ユーザー定義|
|属性|message|

![s107](images/s107.png)

ブロックを繋げます。

![s108](images/s108.png)

操作カテゴリにある「プロンプトの再生」をドラッグアンドドロップします。

![s109](images/s109.png)

属性の設定を行います。「テキスト読み上げ機能(アドホック)」を選択してください。

|項目|値|
|:--|:--|
|テキスト読み上げ機能(アドホック)|動的に入力する|
|タイプ|ユーザー定義|
|属性|message|

![s110](images/s110.png)

ブロックを繋げます。

![s111](images/s111.png)

ブランチカテゴリにある「ループ」をドラッグアンドドロップします。
ループ回数はお好きな数を指定してください。

![s112](images/s112.png)

ブロックを繋げます。

![s113](images/s113.png)

ループとプロンプトの再生を繋げます。

![s114](images/s114.png)

終了 / 転送カテゴリにある「切断/ハングアップ」をドラッグアンドドロップします。

![s115](images/s115.png)

まだ繋いでいない部分を全て「切断/ハングアップ」に繋ぎます。

![s116](images/s116.png)

右上の①［保存］と②「公開」ボタンを順番にクリックします。

![s117](images/s117.png)

### 1-2. IDをメモしておく
問い合わせフローの名前の下に「追加のフロー情報の表示」という項目があるので、それを展開します。展開するとARNの情報が表示されるのでinstanceのIDとconstact-flowのIDをそれぞれメモしておきます。

![s118](images/s118.png)

## Lambda関数を作成しよう！
### 2-1. Lambda関数を作成する
サービスから「Lambda」を検索して、出てきたものをクリックします。

![s120](images/s120.png)

Lambdaから新規で関数を作成します。［関数の作成］ボタンをクリックします。

![s130](images/s130.png)

関数は以下の通り入力して、［関数の作成］ボタンをクリックします。

| 項目       |       値 |
|:-----------------|:------------------|
|①関数名|enebular-AmazonConnect|
|②ランタイム|Node.js 10.x|
|③実行ロール|新しいロールを作成|
|④ロール名|enebular-AmazonConnect-Role|
|⑤ポリシーテンプレート|基本的なLambda@Edgeのアクセス権限|

![s131](images/s131.png)

### 2-2. Amazon Connectアクセス権限を追加する
enebular-AmazonConnect-Roleロールを表示をクリックします。

![s132](images/s132.png)

［インラインポリシーの追加］をクリックします。

![s133](images/s133.png)

サービスを展開して、検索窓に「Connect」と入れて検索します。出てきた［Connect］をクリックします。

![s134](images/s134.png)

アクションのアクセスレベルにある「書き込み」部分を展開して、その中にある`StartOutboundVoiceContact`のチェックを入れます。

![s135](images/s135.png)

すべてのリソースを選択して、右下の［ポリシーの確認］ボタンをクリックします。

![s136](images/s136.png)

ポリシー名を入力します。`enebular-AmazonConnect-Policy`としました。右下の［ポリシーの作成］ボタンをクリックします。

![s137](images/s137.png)

Lambda画面に戻り、画面更新するとAmazon Connectの権限が追加されます。

![s138](images/s138.png)

### 2-3. プログラムを書き込む
index.jsを開き、下記プログラムをコピペしてください。
enebularからリクエストが飛んでくるので、bodyから対象値を取得します。

```javascript
const Util = require('./util.js');

exports.handler = async (event) => {
    const myBody = JSON.parse(event.body);

    // Clovaから飛んでくるデータを取得
    const message = myBody.response.outputSpeech.values.value;

    const response = {
        statusCode: 200,
        body: {"result":"completed!"},
    };
    
    if (message != undefined) {
        // Amazon Connect送信
        await Util.callMessageAction(message);

    }  
    return response;
};
```

新規ファイルを作成します。

![s150](images/s150.png)

下記コードをコピペしてください。

```javascript:util.js
'use strict';
const AWS = require('aws-sdk');
var connect = new AWS.Connect();

// 電話をかける処理
module.exports.callMessageAction = async function callMessageAction(message) {
    return new Promise(((resolve, reject) => {

        // Attributesに発話する内容を設定
        var params = {
            Attributes: {"message": message},
            InstanceId: process.env.INSTANCEID,
            ContactFlowId: process.env.CONTACTFLOWID,
            DestinationPhoneNumber: process.env.PHONENUMBER,
            SourcePhoneNumber: process.env.SOURCEPHONENUMBER
        };

        // 電話をかける
        connect.startOutboundVoiceContact(params, function(err, data) {
            if (err) {
                console.log(err);
                reject();
            } else {
                resolve(data);
            }
        });
    }));
};
```

保存する際はファイル名を「util.js」にしてください。

![s151](images/s151.png)

### 2-4. 環境変数を設定する
Amazon Connectと連携するための環境変数を設定します。

| キー名       |       値 |
|:-----------------|:------------------|
|INSTANCEID|1-2でメモした<span style="color: red; ">instance</span>のID|
|CONTACTFLOWID|1-2でメモした<span style="color: blue; ">contact-flow</span>のID|
|PHONENUMBER|ご自身の携帯電話番号 ※+81を先頭につけて数字のみにします<br>例)090-1234-5678 👉+819012345678|
|SOURCEPHONENUMBER|Amazon Connectで取得した電話番号 ※+81を先頭につけて数字のみにします|

![s152](images/s152.png)

### 2-5. API Gatewayを設定する
LINE ThingsからアクセスするためのURLを発行します。
［トリガーを追加］をクリックします。

![s153](images/s153.png)

トリガーの設定は下記を指定します。最後に［追加］ボタンをクリックします。

| 項目       |       値 |
|:-----------------|:------------------|
|トリガー|API Gateway|
|API|新規APIの作成|
|セキュリティ|オープン|

![s154](images/s154.png)

APIエンドポイントのURLをメモしておきましょう

![s155](images/s155.png)

右上の保存ボタンをクリックします。

![s156](images/s156.png)

## Clovaプロジェクトを作ろう！
### 3-1. スキルチャネルの作成
下記にアクセスしてログインしてください。
[https://clova-developers.line.biz/](https://clova-developers.line.biz/#/)

［スキルを開発する］をクリックします。

![s300](images/s300.png)

［LINE Developersでスキルチャネルを新規作成］ボタンをクリックします。

![s301](images/s301.png)

プロバイダーがまだ無い方は作成お願いします。
新規チャネルを作成します。

チャネルは`handson-0819`としました。［入力内容を確認する］をクリックします。

![s302](images/s302.png)

2つのチェックを入れてから、［スキル開発を始める］ボタンをクリックします。

![s303](images/s303.png)

### 3-2. スキルの設定
Extension IDとスキル名を入力します。

| 項目       |       値 |
|:-----------------|:------------------|
|Extension ID|com.あなたの名前.handson-0819 <br/>※例 com.gaomar.handson-0819|
|スキル名|コネクトスキル|

![s304](images/s304.png)

呼び出し名（メイン）を設定します。

| 項目       |       値 |
|:-----------------|:------------------|
|呼び出し名（メイン）|コネクトスキル|

![s305](images/s305.png)

最後に［作成］ボタンをクリックします。

![s306](images/s306.png)

### 3-3. 対話モデルを設定する
Clovaと対話するためのモデルを作成します。
左側メニューの対話モデルをクリックし、［対話モデルを編集する］をクリックします。

![s400](images/s400.png)

ビルトインスロットタイプの［＋］をクリックします。

![s401](images/s401.png)

「数値と単位を取得」部分を展開して`CLOVA.NUMBER`を有効にします。

![s402](images/s402.png)

カスタムインテントの右側にある［＋］をクリックします。インテント名は「MainIntent」と入力し、［作成］ボタンをクリックします。

![s403](images/s403.png)

スロットリスト部分に「num」と入力し、［＋］をクリックします。

![s404](images/s404.png)

スロットタイプのプルダウンメニューから`CLOVA.NUMBER`を選択します。

![s405](images/s405.png)

サンプル発話リスト部分に「num」を入力し、［＋］をクリックします。

![s406](images/s406.png)

`num`部分をマウスで選択して、出てきたポップアップメニューでnumスロットを選択します。最後に必ず［保存］ボタンをクリックします。

![s407](images/s407.png)

ビルドを行います。約3分ほどビルドに時間がかかります。

![s408](images/s408.png)

## enebularでフローを作成しよう！
### 5-1. enebularプロジェクトを作成する
いよいよenebularを使ってClovaと繋いでいきます。
enebularのページにログインしてください。

[https://enebular.com/sign-in](https://enebular.com/sign-in)

ログインしたら［Create Project］ボタンをクリックします。

![s500](images/s500.png)

プロジェクト名を入力して、［Submit］ボタンをクリックします。

| 項目       |       値 |
|:-----------------|:------------------|
|プロジェクト名|handson-0819|

![s501](images/s501.png)

左側メニューの「Flows」をクリックし、右下の［＋］ボタンをクリックします。

![s502](images/s502.png)

Flow名を入力して、カテゴリは`other`を選択し、［Continue］ボタンをクリックします。

![s503](images/s503.png)

［Edit］ボタンをクリックして、Flow画面を表示します。

![s504](images/s504.png)

### 5-2. Flowを作成する
Flow画面を表示して、Clovaに発話させるフローを作成します。
左側メニューの「入力」カテゴリにある`http`ノードをエディタにドラッグアンドドロップします。

ノードをクリックして、メソッドは`POST`を選択し、URLに`/clova`と入力します。

![s505](images/s505.png)

機能カテゴリにある`switch`ノードをドラッグアンドドロップします。
ノードをクリックして、各項目を埋めていきます。

| 項目       |       値 |
|:-----------------|:------------------|
|③プロパティ|payload.request.type|
|④ == |LaunchRequest|

![s506](images/s506.png)

`http`ノードと`switch`ノードを繋ぎます。

![s507](images/s507.png)

機能カテゴリにある`functions`ノードをドラッグアンドドロップします。ノードをクリックして、コード部分に書きコードを記述します。

```javascript
msg.payload =
{
    "version": "1.0",
    "sessionAttributes": {},
    "response": {
        "outputSpeech": {
            "type": "SimpleSpeech",
            "values": {
                "type": "PlainText",
                "lang": "ja",
                "value": "エネブラーからこんにちは！"
            }
        },
        "card": {},
        "directives": [],
        "shouldEndSession": false
    }
}
return msg;
```

![s508](images/s508.png)

`switch`ノードを`function`ノードを繋ぎます。

![s509](images/s509.png)

出力カテゴリにある`http response`ノードをドラッグアンドドロップします。
このノードは特に設定することはありません。

![s510](images/s510.png)

`function`ノードと`http response`ノードを繋ぎます。

![s511](images/s511.png)

### 5-3. Clovaと連携する
画面右上の［デプロイ］ボタンをクリックして、デプロイを行います。

![s512](images/s512.png)

デプロイボタンの左にiボタンがあるのでクリックします。ポップアップで表示されるアクセスURLをメモしておきます。

![s513](images/s513.png)

Clova Developer Centerページを開きます。左側メニューの開発設定にある［サーバー設定］をクリックします。
サーバーURLに先程メモしたURLを貼り付けて、その末尾に`/clova`を追加します。

![s514](images/s514.png)

### 5-4. シミュレーターで確認する
Clova Developer Centerページのテストをクリックします。
シナリオテストに切り替えて、［○○を起動して］ボタンをクリックします。すると、enebularで設定した値が返ってきます。

![s515](images/s515.png)

## BMI測定に対応しよう！
### 6-1. セッションの受け渡し
身長と体重を答えさせて、結果を発話する流れを作っていきます。
セッションを使って、身長データを一時的に保持して、次に来る体重の数値を使って計算してBMIを求めていきます。

まず、LaunchRequestの発話を「エネブラーからこんにちは！」から「BMIを測定するよ！身長を教えてね！」に変更します。

```コード
msg.payload =
{
    "version": "1.0",
    "sessionAttributes": {},
    "response": {
        "outputSpeech": {
            "type": "SimpleSpeech",
            "values": {
                "type": "PlainText",
                "lang": "ja",
                "value": "BMIを測定するよ！身長を教えてね！"
            }
        },
        "card": {},
        "directives": [],
        "shouldEndSession": false
    }
}
return msg;
```
![s600](images/s600.png)

request.type判定をクリックして、［+追加］ボタンを2回クリックします。
プロパティ値に`IntentRequest`と`SessionEndedRequest`を追加します。
順番も気をつけてください。

| 項目       |       値 |
|:-----------------|:------------------|
|→ 2|IntentRequest|
|→ 3|SessionEndedRequest|

![s601](images/s601.png)

機能カテゴリにある`switch`ノードをドラッグアンドドロップします。
ノードをクリックし、セッション値のnullチェックを行います。

payloadのsessionAttributesにセッション情報が格納されています。初回の場合は、セッション情報が無いので、null値だったらslotのheightから値を取得しています。

| 項目       |       値 |
|:-----------------|:------------------|
|プロパティ|payload.session.sessionAttributes.height|
|→ 1| is not null  ※プルダウンメニューから選択|
|→ 2| is null ※プルダウンメニューから選択|

![s602](images/s602.png)

ノードを繋ぎます

![s603](images/s603.png)

機能カテゴリにある`change`ノードをドラッグアンドドロップします。
このノードは指定した値を別の変数に格納することができます。

| 項目       |       値 |
|:-----------------|:------------------|
|③値の代入|height|
|④対象の値プルダウンメニュー|msg.|
|⑤対象の値|payload.session.sessionAttributes.height|

![s604](images/s604.png)

ノードを繋ぎます。

![s605](images/s605.png)

機能カテゴリにある`change`ノードをドラッグアンドドロップし、ノードを繋ぎます。
ノードをクリックして、スロットから対象の値を取得して変数に代入します。

| 項目       |       値 |
|:-----------------|:------------------|
|③値の代入|weight|
|④対象の値プルダウンメニュー|msg.|
|⑤対象の値|payload.request.intent.slots.num.value|

![s606](images/s606.png)

機能カテゴリにある`functions`ノードをドラッグアンドドロップし、ノードを繋ぎます。
ノードをクリックして、BMI値の計算を行います。

コードは以下の通り。

```javascript
const height = msg.height;
const weight = msg.weight;
const bmiVal = (parseFloat(weight) / (parseFloat(height)/100 * parseFloat(height)/100)).toFixed(1);
const speechText = `あなたのBMIは${bmiVal}です。`;

msg.payload =
{
    "version": "1.0",
    "sessionAttributes": {},
    "response": {
        "outputSpeech": {
            "type": "SimpleSpeech",
            "values": {
                "type": "PlainText",
                "lang": "ja",
                "value": speechText
            }
        },
        "card": {},
        "directives": [],
        "shouldEndSession": false
    }
}
return msg;
```

![s607](images/s607.png)

出力カテゴリにある`http response`ノードをドラッグアンドドロップし、ノードを繋ぎます。

![s608](images/s608.png)

### 6-2. 体重を聞き出す
身長のセッションがまだ格納されていない場合は値を一旦変数に格納しておきます。
機能カテゴリにある`change`ノードをドラッグアンドドロップし、線で繋ぎます。
ノードをクリックして、変数に格納していきます。

| 項目       |       値 |
|:-----------------|:------------------|
|④値の代入|height|
|⑤対象の値プルダウンメニュー|msg.|
|⑥対象の値|payload.request.intent.slots.num.value|

![s609](images/s609.png)

機能カテゴリにある`function`ノードをドラッグアンドドロップし、ノードを繋ぎます。
ノードをクリックして、Clovaのセッションに身長データを渡しています。
4行目の`sessionAttributes`にセッション情報を格納することができます。

```javascript
msg.payload =
{
    "version": "1.0",
    "sessionAttributes": {
        "height": msg.height
    },
    "response": {
        "outputSpeech": {
            "type": "SimpleSpeech",
            "values": {
                "type": "PlainText",
                "lang": "ja",
                "value": "体重をキログラムで教えてね"
            }
        },
        "card": {},
        "directives": [],
        "shouldEndSession": false
    }
}
return msg;
```

![s610](images/s610.png)

出力カテゴリにある`http response`ノードをドラッグアンドドロップし、ノードを繋ぎます。

![s611](images/s611.png)

### 6-3. スキル終了に対応する
機能カテゴリの`functions`ノードをドラッグアンドドロップし、ノードを繋ぎます。
ノードをクリックして、コードを入力します。
16行目の`shouldEndSession`がtrueだと会話が終わり、スキルを終了することができるようになります。

```javascript
msg.payload =
{
    "version": "1.0",
    "sessionAttributes": {},
    "response": {
        "outputSpeech": {
            "type": "SimpleSpeech",
            "values": {
                "type": "PlainText",
                "lang": "ja",
                "value": "ばいばい！"
            }
        },
        "card": {},
        "directives": [],
        "shouldEndSession": true
    }
}
return msg;
```

![s612](images/s612.png)

出力カテゴリにある`http response`ノードをドラッグアンドドロップし、ノードを繋ぎます。

![s613](images/s613.png)

### 6-4. デプロイする
全体の図は以下の通りです。全てのノードが繋がっているか確認して、
デプロイボタンをクリックしてください。

![s614](images/s614.png)

デプロイが終われば、シミュレーターでテストしてみましょう。
身長と体重を入力するとBMIが返ってきます。

![s615](images/s615.png)

## Amazon Connectから結果を聞こう！
### 7-1. Amazon Connectから結果を聞く
BMIの結果をAmazon Connectから電話で知らせてもらいましょう。

機能カテゴリの`http request`ノードをドラッグアンドドロップし、ノードを繋ぎます。

| 項目       |       値 |
|:-----------------|:------------------|
|④メソッド|POST|
|⑤URL|2-5で作成したAPI GatewayのURLを貼り付ける|
|⑥出力形式|JSON|

![s616](images/s616.png)

デプロイボタンをクリックして、再度Clovaのシミュレーターから実行してみてください。
するとAmazon Connectから電話がかかってきます。
