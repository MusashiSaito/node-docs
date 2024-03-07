# Garoon SOAP API でワークフローの情報を取得してみた

---

title: Garoon SOAP API でワークフローの情報を取得してみた。
tags: #garoon #SOAP #SoapApi AWS
author: mu-prog
slide: false

---

今回、SOAP API を使って情報を取得しました。公式ドキュメントよりもわかりやすく説明していきたいと思います。

https://developer.cybozu.io/hc/ja/articles/202228424

まず、公式ドキュメントの左のバーからどの情報を取得したいか選びます。

```
* 		スケジュール
* 		掲示板
* 		メール
* 		通知
* 		ワークフロー
* 		アドレス帳
* 		ファイル管理
* 		メッセージ
* 		マルチレポート
* 		お気に入り
* 		システム管理
* 		連携API
```

次に、これらを取得するための URL を取得するリクエストを送ります。
サブドメインと Username と Password 以外は変更不要です。

```
URL: https://（サブドメイン名）.cybozu.com/g/index.csp?WSDL
data: <?xml version="1.0" encoding="UTF-8"?>
<soap:Envelope xmlns:soap="http://www.w3.org/2003/05/soap-envelope">
  <soap:Header>
    <Action>
      BaseGetApplicationStatus
    </Action>
    <Security>
      <UsernameToken>
        <Username>xxxxxxxxxx</Username>
        <Password>xxxxxxx</Password>
      </UsernameToken>
    </Security>
    <Timestamp>
      <Created>2010-08-12T14:45:00Z</Created>
      <Expires>2037-08-12T14:45:00Z</Expires>
    </Timestamp>
    <Locale>jp</Locale>
  </soap:Header>
  <soap:Body>
    <BaseGetApplicationStatus>
      <parameters></parameters>
    </BaseGetApplicationStatus>
  </soap:Body>
</soap:Envelope>
```

このリクエストを送るとこのように返ってきます。

```
<?xml version="1.0" encoding="utf-8"?>
<soap:Envelope xmlns:soap="http://www.w3.org/2003/05/soap-envelope"
 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
 xmlns:xsd="http://www.w3.org/2001/XMLSchema"
 xmlns:base="http://wsdl.cybozu.co.jp/base/2008">
  <soap:Header>
    <vendor>Cybozu</vendor>
    <product>Garoon</product>
    <product_type>2</product_type>
    <version>3.8.0</version>
    <apiversion>1.3.0</apiversion>
  </soap:Header>
  <soap:Body>
    <base:BaseGetApplicationStatusResponse>
      <returns>
        <application code="portal" status="active"/>　<=　「それぞれにリンクがあります。
        <application code="space" status="active"/>　　　　　　　　　そのリンクを次のリクエストで使います」
        <application code="link" status="active"/>
        <application code="schedule" status="active"/>
        <application code="message" status="active"/>
        <application code="bulletin" status="active"/>
        <application code="cabinet" status="active"/>
        <application code="memo" status="active"/>
        <application code="phonemessage" status="active"/>
        <application code="timecard" status="active"/>
        <application code="todo" status="active"/>
        <application code="address" status="active"/>
        <application code="mail" status="active"/>
        <application code="workflow" status="active"/>
        <application code="report" status="active"/>
        <application code="cbwebsrv" status="active"/>
        <application code="rss" status="active"/>
        <application code="cbdnet" status="active"/>
        <application code="presence" status="active"/>
        <application code="star" status="active"/>
        <application code="notification" status="active"/>
        <application code="kunai" status="active"/>
        <application code="favour" status="deactive"/>
      </returns>
    </base:BaseGetApplicationStatusResponse>
  </soap:Body>
</soap:Envelope>
```

例えば、今回は『ワークフローのそれぞれのフォームの名前と Id』の取得をします。
https://developer.cybozu.io/hc/ja/articles/202270334
![スクリーンショット 2021-06-28 20.55.55.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1129553/dadffad5-b36b-61ed-32b6-1af0128ca8c5.png)
ここを確認してください。
![スクリーンショット 2021-06-28 20.57.03.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/1129553/08973606-349d-7162-a9b2-075b56816ba0.png)

ではフォーム情報を取得するリクエストの書き方を説明します。
先程のリクエストに API 名とパラメーターを入力し、変更していきます。

```
URL:
data: <?xml version="1.0" encoding="UTF-8"?>
<soap:Envelope xmlns:soap="http://www.w3.org/2003/05/soap-envelope">
  <soap:Header>
    <Action>
      WorkflowGetRequestFormByCategoryIds(API名)
    </Action>
    <Security>
      <UsernameToken>
        <Username>xxxxxxxxxx</Username>
        <Password>xxxxxxx</Password>
      </UsernameToken>
    </Security>
    <Timestamp>
      <Created>2010-08-12T14:45:00Z</Created>
      <Expires>2037-08-12T14:45:00Z</Expires>
    </Timestamp>
    <Locale>jp</Locale>
  </soap:Header>
  <soap:Body>
    <WorkflowGetRequestFormByCategoryIds(API名)>
      <parameters>
       <category_id xmlns="">(カテゴリーのid)</category_id>
      </parameters>
    </WorkflowGetRequestFormByCategoryIds(API名)>
  </soap:Body>
</soap:Envelope>
```

このリクエストを送ると下記のように返ってきます。それぞれのデータについても記載しています。

```
<returns>
  <category category_id="2">
    <requestForm
     fid="1"       フォームID
     type="0"      0：申請フォーム 1：区切り線
     name="Name"   フォーム名
     active="1"    0：無効 1：有効
     icon_type="0" 0：標準アイコン 1：URL指定アイコン
     icon_id="0"   アイコンID
     icon_url=""   URL指定アイコンのURL.         >
    </requestForm>
  </category>
</returns>
```

これらデータ構造は　https://developer.cybozu.io/hc/ja/articles/202270364　
で確認できます。
GaroonSoapAPI は全てこのワークフローと同じ流れで取得できます。

```
①リンクの取得
↓
②API名とパラメーターの確認
↓
③API名とパラメーターの入力
↓
④リクエスト
```

#まとめ
今回は GaroonSOAPAPI を使用してリクエストを送ってみました。JavascriptAPI に比べても多くの情報が取得できるのでおすすめです。
是非、Garoon の情報を取得する際は、SOAPAPI を使ってみてください。

```
<?xml version="1.0" ?>
<definitions name="GaroonServices"
             targetNamespace="http://wsdl.cybozu.co.jp/api/2008"

    xmlns="http://schemas.xmlsoap.org/wsdl/"

    xmlns:wsdl="http://schemas.xmlsoap.org/wsdl/"

    xmlns:soap12="http://schemas.xmlsoap.org/wsdl/soap12/"

    xmlns:tns="http://wsdl.cybozu.co.jp/api/2008"

    xmlns:base_services="http://wsdl.cybozu.co.jp/base/2008"

    xmlns:schedule_services="http://wsdl.cybozu.co.jp/schedule/2008"

    xmlns:address_services="http://wsdl.cybozu.co.jp/address/2008"

    xmlns:workflow_services="http://wsdl.cybozu.co.jp/workflow/2008"

    xmlns:mail_services="http://wsdl.cybozu.co.jp/mail/2008"

    xmlns:message_services="http://wsdl.cybozu.co.jp/message/2008"

    xmlns:notification_services="http://wsdl.cybozu.co.jp/notification/2008"

    xmlns:report_services="http://wsdl.cybozu.co.jp/report/2008"

    xmlns:cabinet_services="http://wsdl.cybozu.co.jp/cabinet/2008"

    xmlns:admin_services="http://wsdl.cybozu.co.jp/admin/2008"

    xmlns:util_api_services="http://wsdl.cybozu.co.jp/util_api/2008"

    xmlns:star_services="http://wsdl.cybozu.co.jp/star/2008"

    xmlns:bulletin_services="http://wsdl.cybozu.co.jp/bulletin/2008"
             >
    <wsdl:import namespace="http://wsdl.cybozu.co.jp/base/2008" location="https://static.cybozu.com/g/F22.12_1020470/api/2008/base.wsdl" />
    <wsdl:import namespace="http://wsdl.cybozu.co.jp/schedule/2008" location="https://static.cybozu.com/g/F22.12_1020470/api/2008/schedule.wsdl" />
    <wsdl:import namespace="http://wsdl.cybozu.co.jp/address/2008" location="https://static.cybozu.com/g/F22.12_1020470/api/2008/address.wsdl" />
    <wsdl:import namespace="http://wsdl.cybozu.co.jp/workflow/2008" location="https://static.cybozu.com/g/F22.12_1020470/api/2008/workflow.wsdl" />
    <wsdl:import namespace="http://wsdl.cybozu.co.jp/mail/2008" location="https://static.cybozu.com/g/F22.12_1020470/api/2008/mail.wsdl" />
    <wsdl:import namespace="http://wsdl.cybozu.co.jp/message/2008" location="https://static.cybozu.com/g/F22.12_1020470/api/2008/message.wsdl" />
    <wsdl:import namespace="http://wsdl.cybozu.co.jp/notification/2008" location="https://static.cybozu.com/g/F22.12_1020470/api/2008/notification.wsdl" />
    <wsdl:import namespace="http://wsdl.cybozu.co.jp/report/2008" location="https://static.cybozu.com/g/F22.12_1020470/api/2008/report.wsdl" />
    <wsdl:import namespace="http://wsdl.cybozu.co.jp/cabinet/2008" location="https://static.cybozu.com/g/F22.12_1020470/api/2008/cabinet.wsdl" />
    <wsdl:import namespace="http://wsdl.cybozu.co.jp/admin/2008" location="https://static.cybozu.com/g/F22.12_1020470/api/2008/admin.wsdl" />
    <wsdl:import namespace="http://wsdl.cybozu.co.jp/util_api/2008" location="https://static.cybozu.com/g/F22.12_1020470/api/2008/util_api.wsdl" />
    <wsdl:import namespace="http://wsdl.cybozu.co.jp/star/2008" location="https://static.cybozu.com/g/F22.12_1020470/api/2008/star.wsdl" />
    <wsdl:import namespace="http://wsdl.cybozu.co.jp/bulletin/2008" location="https://static.cybozu.com/g/F22.12_1020470/api/2008/bulletin.wsdl" />
    <service name="BaseService">
        <port name="BasePort" binding="base_services:BaseBinding">
            <soap12:address location="https://wr0dd.cybozu.com/g/cbpapi/base/api.csp?" />
        </port>
    </service>
    <service name="ScheduleService">
        <port name="SchedulePort" binding="schedule_services:ScheduleBinding">
            <soap12:address location="https://wr0dd.cybozu.com/g/cbpapi/schedule/api.csp?" />
        </port>
    </service>
    <service name="AddressService">
        <port name="AddressPort" binding="address_services:AddressBinding">
            <soap12:address location="https://wr0dd.cybozu.com/g/cbpapi/address/api.csp?" />
        </port>
    </service>
    <service name="WorkflowService">
        <port name="WorkflowPort" binding="workflow_services:WorkflowBinding">
            <soap12:address location="https://wr0dd.cybozu.com/g/cbpapi/workflow/api.csp?" />
        </port>
    </service>
    <service name="MailService">
        <port name="MailPort" binding="mail_services:MailBinding">
            <soap12:address location="https://wr0dd.cybozu.com/g/cbpapi/mail/api.csp?" />
        </port>
    </service>
    <service name="MessageService">
        <port name="MessagePort" binding="message_services:MessageBinding">
            <soap12:address location="https://wr0dd.cybozu.com/g/cbpapi/message/api.csp?" />
        </port>
    </service>
    <service name="NotificationService">
        <port name="NotificationPort" binding="notification_services:NotificationBinding">
            <soap12:address location="https://wr0dd.cybozu.com/g/cbpapi/notification/api.csp?" />
        </port>
    </service>
    <service name="ReportService">
        <port name="ReportPort" binding="report_services:ReportBinding">
            <soap12:address location="https://wr0dd.cybozu.com/g/cbpapi/report/api.csp?" />
        </port>
    </service>
    <service name="CabinetService">
        <port name="CabinetPort" binding="cabinet_services:CabinetBinding">
            <soap12:address location="https://wr0dd.cybozu.com/g/cbpapi/cabinet/api.csp?" />
        </port>
    </service>
    <service name="AdminService">
        <port name="AdminPort" binding="admin_services:AdminBinding">
            <soap12:address location="https://wr0dd.cybozu.com/g/sysapi/admin/api.csp?" />
        </port>
    </service>
    <service name="UtilService">
        <port name="UtilPort" binding="util_api_services:UtilBinding">
            <soap12:address location="https://wr0dd.cybozu.com/g/util_api/util/api.csp?" />
        </port>
    </service>
    <service name="StarService">
        <port name="StarPort" binding="star_services:StarBinding">
            <soap12:address location="https://wr0dd.cybozu.com/g/cbpapi/star/api.csp?" />
        </port>
    </service>
    <service name="BulletinService">
        <port name="BulletinPort" binding="bulletin_services:BulletinBinding">
            <soap12:address location="https://wr0dd.cybozu.com/g/cbpapi/bulletin/api.csp?" />
        </port>
    </service>
</definitions>
```
