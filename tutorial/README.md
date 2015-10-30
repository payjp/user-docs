
# チュートリアル

## はじめての決済


PAY.JPでは、カード決済を安全で簡単に行えるよう、PAY.JP CheckoutというJavaScriptライブラリを用意しています。
PAY.JP Checkoutは &lt;script&gt; タグを1行記述するだけで、 デザインされた決済フォーム、カード情報のバリデーション、セキュアなカードトークン化を行い、簡単に決済を組み込むことができます。

下記のリンク先のページにあるボタンを押して、実際のテスト決済を試してみましょう。

<a href="http://payjp.github.io/sample/payjp-checkout/">PAY.JP Checkout サンプルページ</a>

* カード番号：4242 4242 4242 4242
* 有効期限：12 / 20
* CVC：123
* カード名義：YUI ARAGAKI

と記入して、「カード情報を入力する」のボタンをクリックします。そうすると、カード情報の代わりとなる token が作成されます。この token を使ってサーバー側でAPIを叩くだけで決済が完了します。

### 使い方

PAY.JP Checkout は、HTMLで下記のようなコードを書くだけで、決済ができるようになります。

```html
<form action="/pay" method="post">
  <script src="https://checkout.pay.jp/" class="payjp-button" data-key=""></script>
</form>
```

`data-key` には各ユーザーの公開キーを入力します。この例では、`/pay` に `payjp-token` パラメーターの値として token が送られます。
サーバ側では、PAY.JPで作られたトークンを受け取り、下記のようなリクエストをするだけで実際の決済が行われます。

```shell
$ curl "https://api.pay.jp/v1/charges" \
  -u "sk_test_c62fade9d045b54cd76d7036": \
  -d "amount=400" \
  -d "currency=jpy" \
  -d "card=payjp_token"
```

また、トークンを使ってカードを所有する顧客を作成したり、あなたの用途に合わせてさまざまな使い方ができます。

```shell
$curl "https://api.pay.jp/v1/customers" \
  -u "sk_test_c62fade9d045b54cd76d7036": \
  -d "card=payjp_token"
```


### フォーム内に埋め込む

カード情報の入力だけでなく、他の情報もフォームから送信をする場合にカードの情報の入力次第で自動送信を行いたくないときがあります。そのような場合、

```html
<form action="/pay" method="post">
  <script src="https://checkout.pay.jp/" class="payjp-button" data-key="pk_test_0383a1b8f91e8a6e3ea0e2a9" data-lang="ja" data-partial="true"></script>
  <input type="text" name="other" value="他の情報" />
</form>
```

というように `data-partial="true"` と設定することで自動的な送信を止めることができます。 token の取得後、ボタンには `✔ カード情報入力済み` と表示され、フォーム自体は送信されません。

現在1ページ内に設置できる PAY.JP Checkout は1つとなっています。

### パラメータ

PAY.JP Checkout を柔軟に利用するために以下のパラメータを利用することができます。

<table>
  <thead>
    <tr>
      <th style="width: 20%;">パラメータ名</th>
      <th style="width: 50%;">説明</th>
      <th>既定値</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>
        data-key
        <strong>（必須）</strong>
      </td>
      <td>あなたのアカウントの公開鍵。"pk_test_0383a1b8f91e8a6e3ea0e2a9"</td>
      <td>なし</td>
    </tr>
    <tr>
      <td>data-partial</td>
      <td>
        <strong>"true"</strong>
        と設定することで、カード情報フォーム入力後に自動的な submit を行わない
      </td>
      <td>false</td>
    </tr>
    <tr>
      <td>data-text</td>
      <td>ページ上に表示されるカード情報入力フォームを開くためのボタンのテキスト</td>
      <td>
        data-partial="false" のとき「カードで支払う」
        <br>
        data-partial="true" のとき「カード情報を入力する」
      </td>
    </tr>
    <tr>
      <td>data-submit-text</td>
      <td>カード情報入力フォームのボタンのテキスト</td>
      <td>
        data-partial="false" のとき「カードで支払う」
        <br>
        data-partial="true" のとき「カード情報を送信する」
      </td>
    </tr>
    <tr>
      <td>data-token-name</td>
      <td>作成された token が挿入されるhiddenなinputのname</td>
      <td>payjp-token</td>
    </tr>
    <tr>
      <td>data-key-color</td>
      <td>フォームに表示されるアイコンなどの色を16進数(例 #198fcc)で指定できます</td>
      <td></td>
    </tr>
    <tr>
      <td>data-previous-token</td>
      <td>エラーにより戻ってきた場合など、ユーザに再入力をさせないために作成済みの token を入れるパラメータ</td>
      <td>なし</td>
    </tr>
    <tr>
      <td>data-lang</td>
      <td>サーバからのエラーメッセージなどを表示する言語。"ja" と "en" が利用可能</td>
      <td>ja</td>
    </tr>
    <tr>
      <td>data-on-created</td>
      <td>token の生成に成功し、フォームの値が設定された直後に、生成した token を引数に呼び出される関数。関数名で指定。自動送信を行う設定（data-partial="false"）になっており、かつ関数が "false" を返した場合は、自動送信を行わずに処理を終了する</td>
      <td>なし</td>
    </tr>
    <tr>
      <td>data-on-failed</td>
      <td>token の生成に失敗した時に、HTTPステータスコードを第一引数、エラーの内容を第二引数として呼び出される関数。関数名で指定</td>
      <td>なし</td>
    </tr>
  </tbody>
</table>

### 自由なフォーム
PAY.JPではフォームデザインがあらかじめ用意されたPAY.JP Checkoutだけではなく、フォームデザインを自由に設定可能な `payjp.js` を用意しています。こちらはトークン取得に特化しており、PAY.JP CheckoutのようなUIやバリデーションの機能はもたずに、あなたの好きなデザインや挙動でPAY.JPを組み込むことができます。
下記は実際の `payjp.js` を用いたサンプルページです。

<a href="http://payjp.github.io/sample/payjp-js/">payjp.js サンプルページ</a>

「サンプルのカードを使う」をクリックし、送信を行うと、PAY.JP Checkoutと同様にトークンが作成されます。  
このサンプルページのソースコードは下記にあるので、参考にして試してみてください。

https://github.com/payjp/payjp.github.io/blob/master/sample/payjp-js/index.html


## テストカード

APIで使用するテストカードはこちらです。テストモードでは、本物のカード番号ではなく、必ず下記のテスト番号を用いるようにしてください。

| 番号 | ブランド |
|:-----------|------------:|
| 4242424242424242    |      Visa |
| 4012888888881881    |      Visa |
| 5555555555554444    |      MasterCard |
| 5105105105105100    |      MasterCard |
| 3530111333300000    |      JCB |
| 3566002020360505    |      JCB |
| 378282246310005     |      American Express |
| 371449635398431     |      American Express |
| 38520000023237       |     Diners Club |
| 30569309025904         |   Diners Club |
| 6011111111111117       |   Discover |
| 6011000990139424    |      Discover |


下記の番号は、PAY.JPで返されるエラーコードや特定のステータスを返す番号です。

| 番号 | 内容 |
|:-----------|------------:|
|   4000000000000070     | address_zip_check=failed を返す不正な郵便番号を意味する番号 |
| 4000000000000100     |   cvc_check=failed を返す不正なCVCを意味する番号 |
| 4000000000000150       |    cvc_check=unavailable を返す利用不可能なCVCを意味する番号   |
| 4000000000000002         |  declined cardを返す利用不可能を意味する番号   |
| 4000000000000066       |       expired_cardを返す有効期限切れの番号 |
| 4000000000000890    |     invalid_cvcを返す不正なCVCを意味する番号 |
|   4000000000000123     |    processing_errorを返す決済サーバーエラーを意味する番号 |


## 定期課金
PAY.JPでは、定期課金を簡単に組み込むためのAPIを用意しています。ここでは定期課金を実装する流れについて説明します。

### プランを作成
まずはじめに定期課金を行うためのプランを作成します。プランでは、金額、課金日、トライアル日数など定期課金における細かい設定ができます。例えば、月額500円のノーマルプラン、月額2000円のゴールドプランというように、定期課金のタイプに応じて適したプランを使い分けることができます。

```shell
curl "https://api.pay.jp/v1/plans" \
-u "sk_test_c62fade9d045b54cd76d7036": \
-d "id=normal" \
-d "amount=500" \
-d "interval=month" \
-d "currency=jpy"
```

### 顧客を登録して定期課金を行う
次に実際に定期課金を行うために、課金の対象となる顧客を作成します。クレジットカード課金なので、カード情報の紐付いた顧客である必要があります。

```shell
curl "https://api.pay.jp/v1/customers" \
-u "sk_test_c62fade9d045b54cd76d7036": \
-d "card=tok_8ec984635ae5d7a187f4f2bdda84" \
-d "email=subscriber@pay.jp"
```

カード情報が正常であれば、顧客の作成に成功します。

```json
{
  "cards": {
    "count": 1,
    "data": [
      {
        "address_city": null,
        "address_line1": null,
        "address_line2": null,
        "address_state": null,
        "address_zip": null,
        "address_zip_check": "unchecked",
        "brand": "Visa",
        "country": null,
        "created": 1441537780,
        "customer": "cus_2849e3adb18f8997760001007bbd",
        "cvc_check": "passed",
        "exp_month": 12,
        "exp_year": 2020,
        "fingerprint": "e1d8225886e3a7211127df751c86787f",
        "id": "car_d9024617c1aa88e611b994610825",
        "last4": "4242",
        "name": null,
        "object": "card"
      }
    ],
    "has_more": false,
    "object": "list",
    "url": "/v1/customers/cus_2849e3adb18f8997760001007bbd/cards"
  },
  "created": 1441537780,
  "default_card": "car_d9024617c1aa88e611b994610825",
  "description": null,
  "email": "subscriber@pay.jp",
  "id": "cus_2849e3adb18f8997760001007bbd",
  "livemode": false,
  "object": "customer",
  "subscriptions": {
    "count": 0,
    "data": [],
    "has_more": false,
    "object": "list",
    "url": "/v1/customers/cus_2849e3adb18f8997760001007bbd/subscriptions"
  }
}
```

ここでかえってきた顧客ID "cus_2849e3adb18f8997760001007bbd" と課金を行いたいプランを紐付けて、定期課金リクエストを行います。

```shell
curl "https://api.pay.jp/v1/subscriptions" -u "sk_test_c62fade9d045b54cd76d7036": -d "customer=cus_2849e3adb18f8997760001007bbd" -d "plan=normal"
```

### 課金サイクルについて
現在PAY.JPでは、月次で1ヶ月の課金サイクルをサポートしています。定期課金は月に一度行われ、月末を基準とした定期課金の場合は、確実に月末に課金が回るようになっています。例えば、月末に定期課金が行われた場合、1月31日に課金 => 2月28日に課金 => 3月31日に課金というように、確実に月末に課金が回るようなしくみとなっています。

### 課金日
課金日は任意で設定することが可能です。プランを作成するときに課金日となる "billing_day" をセットすると、その日に課金が行われるように適用がされます。この場合、設定した課金日の午前9:00頃に課金が回るようになっています。
課金日を設定しない場合は、定期課金が始まった日が課金日の基準となり、時間もその時間に課金が行われます。

### トライアル
定期課金ではトライアル日数を設定して、無料期間を設けることができます。例えば、30日のトライアルを設定したプランであれば、30日後に実際の課金が行われ、その日を基準として定期課金が回ります。

### 課金の失敗
顧客のカードの有効期限が切れていたり、カードが不正であると、支払いが失敗します。この場合定期課金はストップし、再開リクエストがされるまで今後の課金は行われません。定期課金を再開するには、顧客のカードを有効なものに更新してから、再開リクエストを行います。再開された定期課金は再開日時が基準となり、次回の課金日はその１ヶ月後となります。

```shell
curl "https://api.pay.jp/v1/subscriptions/sub_573a09399fa5bcc53b58911afd10/resume" -u "sk_test_c62fade9d045b54cd76d7036": -XPOST
```

### キャンセル
定期課金をキャンセルする場合は、下記のようなリクエストを送ることで可能です。現在のサイクルの終了日に定期課金が削除されます。

```shell
curl "https://api.pay.jp/v1/subscriptions/sub_573a09399fa5bcc53b58911afd10/cancel" -u "sk_test_c62fade9d045b54cd76d7036": -XPOST
```

即座に定期課金を削除する場合は、下記のリクエストを送ります。

```shell
curl "https://api.pay.jp/v1/subscriptions/sub_573a09399fa5bcc53b58911afd10" -u "sk_test_c62fade9d045b54cd76d7036": -XDELETE
```

## Webhook
Webhookとは、あるサービスで発生したイベントの通知を、HTTP経由の外部URLで受け取る仕組みのことです。
Webhookを使うと、例えば下記のようなPAY.JPで起きたイベントをあなたの任意のURLに対して、通知することができます。

- 支払いが正常に行われたとき
- 定期課金による引き落としが行われ、期間が更新されたとき
- 定期課金がキャンセルされたとき

PAY.JPの管理画面から送信先URLを追加するだけで、上記のようなイベントが自動的に通知されるようになります。

### 設定
PAY.JPにログインして、下記の管理画面からWebhookの送信先を追加できます。URLを入力し、イベントが送信されるモードをテストモードか本番モードかで選択し、追加を押せば完了です。Webhook送信先は複数追加することが可能です。

https://pay.jp/dashboard/settings#webhook

### リクエスト形式
PAY.JPのWebhookはすべてPOSTリクエストで送信され、リクエストボディには、イベントのJSONデータが含まれています。

### イベント
下記は実際にPAY.JPから送信されるイベントの種類です。それぞれの内容に応じて、イベントのJSONが送信されます。 `type` はイベントのJSONデータに含まれます。

| type | 内容 |
|:-----------|------------:|
| charge.succeeded |  支払いの成功 |
| charge.failed |  支払いの失敗 |
| charge.updated |  支払いの更新 |
| charge.refunded |  支払いの返金 |
| charge.captured |  支払いの確定 |
| token.created |  トークンの作成 |
| customer.created |  顧客の作成 |
| customer.updated |  顧客の更新 |
| customer.deleted |  顧客の削除 |
| customer.card.created |  顧客のカード作成 |
| customer.card.updated |  顧客のカード更新 |
| customer.card.deleted |  顧客のカード削除 |
| plan.created |  プランの作成 |
| plan.updated |  プランの更新 |
| plan.deleted |  プランの削除 |
| subscription.created |  定期課金の作成 |
| subscription.updated |  定期課金の更新 |
| subscription.deleted |  定期課金の削除 |
| subscription.paused |  定期課金の停止 |
| subscription.resumed |  定期課金の再開 |
| subscription.canceled |  定期課金のキャンセル |
| subscription.renewed |  定期課金の期間更新 |

下記は、5000円の支払いが成功したときにWebhookで送信されるイベントのデータ例です。

```
{
  "created": 1442212986,
  "data": {
    "amount": 5000,
    "amount_refunded": 0,
    "captured": true,
    "captured_at": 1442212986,
    "card": {
      "address_city": null,
      "address_line1": null,
      "address_line2": null,
      "address_state": null,
      "address_zip": null,
      "address_zip_check": "unchecked",
      "brand": "Visa",
      "country": null,
      "created": 1442212986,
      "customer": null,
      "cvc_check": "passed",
      "exp_month": 1,
      "exp_year": 2016,
      "fingerprint": "e1d8225886e3a7211127df751c86787f",
      "id": "car_f0984a6f68a730b7e1814ceabfe1",
      "last4": "4242",
      "name": null,
      "object": "card"
    },
    "created": 1442212986,
    "currency": "jpy",
    "customer": null,
    "description": null,
    "expired_at": null,
    "failure_code": null,
    "failure_message": null,
    "id": "ch_bcb7776459913c743c20e9f9351d4",
    "livemode": false,
    "object": "charge",
    "paid": true,
    "refund_reason": null,
    "refunded": false,
    "subscription": null
  },
  "id": "evnt_5328acdbdb5294d6fc9cc903f8c",
  "livemode": false,
  "object": "event",
  "pending_webhooks": 1,
  "type": "charge.succeeded"
}
```

`data` にはイベントの詳細内容が入ります。APIで返ってくるレスポンスと同様のJSONデータです。 `pending_webhooks` では、まだWebhookが正常に受信されていない送信先URLの数を示しています。

### リトライ
Webhookの送信先から `4xx` や `5xx` のステータスコードが返ってくる場合、Webhookは自動的にリトライを行います。リトライは3分間隔で、最大3回行われます。

### 安全性
本番環境でWebhookを使用する場合は、必ずHTTPS経由で受け取るようにしましょう。Webhookでは、顧客情報や支払いの詳細データなども送信されるため、HTTPSによる暗号化通信を強く推奨いたします。

## ライブラリ

PAY.JPでは下記の言語のライブラリをサポートしております。より幅広く開発者の方をサポートするため、今後どんどんライブラリは増やしていく予定です。

- [payjp-python](https://github.com/payjp/payjp-python)
- [payjp-ruby](https://github.com/payjp/payjp-ruby)
- [payjp-php](https://github.com/payjp/payjp-php)
- [payjp-java](https://github.com/payjp/payjp-java)
