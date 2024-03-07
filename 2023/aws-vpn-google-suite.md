# フェデレーション認証＋ MFA 認証

---

title: フェデレーション認証＋ MFA 認証
tags: AWS
author: mu-prog
slide: false

---

# セキュリティの高い社内環境の認証を導入したい

社内の VPN 環境の構築を行うこととなったが、セキュリティについてはあまり知らない。
色々と調べたところ、多要素認証（MFA）というものにたどり着いた。
それ故、「AWS clientVPN ＋ MFA 認証」を組み合わせた vpn 環境を構築することとなった。

# 今回使用する環境

今回使用する環境は以下のものとする

1. Google Workspace(G Suite)
2. AWS client VPN

> Workspace と G Suite の違いは？
> 分かりやすく捉えるのであれば、Google Workspace は G suite の強化版です。

# [Google] SAML アプリを追加

## SAML アプリを追加手順

1. Google Admin（ https://admin.google.com/ ）に移動（管理者アカウントのみ）
2. サイドバーから`アプリ >「webアプリとモバイルアプリ」> アプリ追加 > カスタムSAMLアプリの追加`
3. 以下のように設定してください（URL や ID の変更はしなくても良い）

> ACS の URL： http://127.0.0.1:35001
> エンティティ ID： urn:amazon:webservices:clientvpn
> 署名付き応答： checked
> 名前 ID の形式： UNSPECIFIED
> 名前 ID：Basic Information ＞ Primary Email
> SAML 属性マッピング
> First Name => FirstName
> Last Name => LastName
> Department => memberOf

4. 作成したアプリの「サービスのステータス」を認証を必要とする組織等を指定し「オン」にし「保存」。
5. 以上

ユーザーの追加に関してはこちら

## ローカル http を使用するように構成

ブラウザを使用して、SAML アプリの修正に対する有効な POST 要求を取得し、「https」ではなくローカル「http」を使用するように構成する必要があります。
これを行うには、ブラウザで検証ツール（開発者ツール）を開き、「ネットワーク」タブを使用してログを取得します。
GoogleAdmin コンソールで SAML アプリの「サービス プロバイダーの詳細」をクリックし、ブラウザーの開発者ツールでネットワーク記録をオンにします。ACS の URL を編集して https://127.0.0.1:35001 (ポート 35001) にし、「保存」をクリックします。
「batchexecute」という名前の最初の HTTP POST 要求をクリックし、右クリックして `コピー > cURL としてコピ-` を選択します。 コピーされた文字列は次のようになります。 (一部省略しておりますいます)。

```
curl 'https://admin.google.com/_/DasherSamlAdminConsoleUi/data/batchexecute?rpcids=DcqUrb&source-path=%2Fac%2Fapps%2Fsaml%2F635637890734%2Fsp&f.sid=3568075621442303950&bl=boq_dasher-unifiedapps-adminconsole-ui_20220306.16_p0&hl=en_GB&cid=019do8f3&_reqid=463957&rt=c' -X POST -H 'User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:96.0) Gecko/20100101 Firefox/96.0' -H 'Accept: */*' -H 'Accept-Language: en-US,en;q=0.5' -H 'Accept-Encoding: gzip, deflate, br' -H 'Referer: https://admin.google.com/' -H 'X-Same-Domain: 1' -H 'Content-Type: application/x-www-form-urlencoded;charset=utf-8' -H 'Origin: https://admin.google.com' -H 'DNT: 1' -H 'Alt-Used: admin.google.com' -H 'Connection: keep-alive' -H 'Cookie: ...' -H 'Sec-Fetch-Dest: empty' -H 'Sec-Fetch-Mode: cors' -H 'Sec-Fetch-Site: same-origin' -H 'TE: trailers' --data-raw 'f.req=...https%3A%2F%2F127.0.0.1%3A35001...'
```

しかし、こちらのままではアクセスができません。変更を加えます。
`--data-raw 'f.req=...https%3A%2F%2F127.0.0.1%3A35001...'`
↑ この部分（data-raw）が最後の方にあると思います。以下のように変更を加えてください。
`--data-raw 'f.req=...http%3A%2F%2F127.0.0.1%3A35001...'`

変更したものをご自身のターミナルで実行してください。
実行後、ブラウザをリフレッシュすると、Google Admin コンソールのアプリの ACS URL が`http://127.0.0.1:35001`に変更されているいます。それが確認できた時点で設定は完了です。

## メタデータのダウンロード

最後に、アプリケーションのメインページに戻り、「メタデータのダウンロード」をクリックしてダウンロードします。（以降の手順で AWS で使用しようするため）

# AWS VPC 構築

# AWS ClientVPN 構築

# 休憩

# ユーザー・管理者設定

## クライアント側設定

1. AWS VPN クライアントをダウンロード
   https://aws.amazon.com/jp/vpn/client-vpn-download/
   ↑ こちらをダウンロード
2. ダウンロードした「AWS VPN Client」を開いて、メニューバーの『ファイル』>『プロファイルを管理』を押下
3. VPN 設定プロファイルの「ファイル」を押し、管理者から受け取った、拡張子が「.cvpn」のファイルを追加
4. 表示名を適当なものにし、「プロファイル追加」を押下
5. 「接続」のボタンを押す

# 管理者 設定手順

## ユーザーの追加

- 管理者のアカウントで https://admin.google.com/ にログイン

## 組織に認証を与える方法

- サイドバーの「アプリ＞ウェブアプリとモバイルアプリ」を押下
- 認証を加えるアプリを開く
- 「ユーザーアクセス」を押下
- 認証を加える組織部門を選び、サービスのステータスをオンにする

## 認証を「オン」にする組織へユーザーを追加（組織部門間のユーザーの移動）

- サイドバーの「ディレクトリ＞ユーザー」でユーザーの一覧が確認できるようにする
- 認証を与える組織部門に追加したいユーザーを選択する
- 一覧の上部にある「その他のオプション」から「組織部門の変更」を押下
- 表示された画面から（認証がオンになっている）組織を選択し「続行」
- 以下のように組織に追加されていれば完了です。

## client への「.ovpn」配布

- AWS client VPN コンソールに移動
- [novelworks-client-vpn](https://ap-northeast-1.console.aws.amazon.com/vpc/home?region=ap-northeast-1#ClientVPNEndpointDetails:clientVpnEndpointId=cvpn-endpoint-0a1958615a120ddab)を選択
- 「クライアント設定をダウンロード」を押下
- ダウンロードされた ovpn ファイルをユーザーへ配布
