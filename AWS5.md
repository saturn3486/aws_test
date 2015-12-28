# 触って学ぶクラウドインフラAWS　Chapter5

HTTPに関して少々。

## HTTPとは

Hyper Text Transfer Protocol

HTMLをはじめとする、Webサービスに必要な情報を伝達するための規約。

404 Not foundとか、500 Internal Server ErrorとかもHTTPで定義されたコード。

あれってコッチでカスタマイズ出来るんだけど、それをやってる所って結構好き。


##### レスポンスとリクエストの書式

HTTPはクライアント・サーバー型のアーキテクチャ。送信側と受信側ね。
2者間で「リクエスト」と「レスポンス」をやりとりする方式。

**リクエスト**

- リクエストライン

要求コマンドの事。中身は、要求方法、要求するURL。
冒頭の一行がこれに当たる。

- ヘッダー

ブラウザから送信する追加情報。
要求したいホスト名やブラウザの種類、対応言語、直前に見ていたページURLなど、複数行に渡る。

- ボディ

HTMLフォームや、Ajaxなどで、POSTというメソッドを利用し、データをサーバに送信するときに発生。
ヘッダーとボディとは空行で句切られてる。

で構成される。

**レスポンス**

- ステータスライン

要求の可否を返す。
冒頭の一行。
正常に終了すれば、200 OK  
見つからない時は、404 Not Found  
等など。

- ヘッダー

追加情報を返すための部分。
ボディ部の種類を示す、content-typeヘッダーとか、ボディの長さを示す。content-lengthヘッダーとか有。


- ボディ

要求されたURLに対するコンテンツ。
HTMLのテキストだったり、画像だったり、要求されたコンテンツデータそのもの。
他の要素とは空行で区切られる。


で構成される。


#### ブラウザのデベロッパーツールで覗き見る

本ではfirefoxだけど、入っていないので、chromeで確認する。

1. cmd+option+iとかでデベロッパーツールが開ける。
1. 「Network」タブを確認。
1. 適当に画面を更新。
1. 好きなレコードのName部分をクリック。
1. headersタブをクリックすると、レスポンスヘッダとリクエストヘッダが確認できる。
Generalタブに、リクエストラインとステータスライン。  
レスポンスヘッダとリクエストヘッダにはそれぞれの情報が載ってる。
1. Responseタブでボディ部（HTMLのデータそのもの）が確認できる。

#### リクエストとレスポンスの詳細

**HTTPメソッド**

コンテンツに対する操作コマンドの事。
多くの場合は、GETとPOST。  
PUTメソッドやDELETEメソッドは、  
WebDAV（web-based distributed authoring and versioning）  
というプロトコルで、Webサーバー上のファイルを読み書きする際に使われる。

|メソッド|意味|
|---|---|
|GET|リソースを取得|
|POST|リソースにデータ送信したり、小リソースの作成|
|HEAD|リソースのヘッダー情報だけ取得。|
|PUT|リソースを更新したり、作成したり。|
|DELETE|リソースを削除|
|OPTIONS|サポートしているメソッドを取得|
|TRACE|自分宛てにリクエスト・メッセージを返してループバックテスト|
|CONNECT|プロキシ動作のトンネル接続を変更|


**HTTPステータスコード**

結果の成否を示す値。
3桁の数字で表され、百の位の数字で大まかな成否がきまる。
残りの桁で詳細なステータスが決まる。
リダイレクトを受け取ると、ブラウザは自動的にリダイレクト先に再接続してコンテンツを得ようとする。

|コード|意味|回折|
|----|----|----------------|
|1xx|処理中|あまり使われない。|
|2xx|成功|200 OKとかよく見る。|
|3xx|リダイレクト|別のURLにリダイレクト|
|4xx|クライアントエラー|403 Forbiddonとか404とか有名|
|5xx|サーバーエラー|500 internal server errorはあまり見たくない。503 service unavailableとかも。|





**リクエストヘッダー**

クライアントからサーバーへ送信するときに得られるヘッダー。

```
Host:aws.amazon.com
user-agent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_10_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/47.0.2526.106 Safari/537.36
accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
accept-encoding:gzip, deflate, sdch
accept-language:ja,en-US;q=0.8,en;q=0.6
Cookie:...略...
```

よく使われるリクエストヘッダーは、

- Host

要求を送るホスト名。

- user-agent

ブラウザ種別。
Webシステムを構築する時は、これを判断して、ブラウザの種類で処理分岐を実装することが有る。

- Cookie

例えば、ユーザーがログインした時にそのログイン情報を保存するときなどに使われる。
ログインしたユーザーと結びつける情報が格納サれている事があるため、これが漏洩すると、第三者がなりすます可能性あり。


**レスポンスヘッダー**

サーバーからクライアントへ返すときに送信される情報。

```
Access-Control-Allow-Origin:http://aws.amazon.com
Content-Encoding:gzip
Content-Type:text/html;charset=UTF-8
Date:Sun, 27 Dec 2015 04:08:57 GMT
Last-Modified:Fri, 25 Dec 2015 01:18:51 GMT
Server:Server
Set-Cookie:aws-gz=1; Domain=.amazon.com; Path=/
Transfer-Encoding:chunked
Vary:Accept-Encoding,Avail-Dictionary,User-Agent
x-amz-id-1:0ZWAD03CN7SPTXFXSNZH
X-Content-Type-Options:nosniff
X-Frame-Options:SAMEORIGIN
```

よく使われるレスポンスヘッダーは、

- Content-Type

`text/html`だったらHTML、画像なら`image/jpeg`とかそんな感じ。
このようなコンテンツの種類は、MIME（Multipurpose Internet Mail Extension）タイプと呼ばれ、その一覧は、[IANAのWebサイト](http://www.iana.org/assignments/media-types/media-types.xhtml)に載ってます。

Apacheを含むほとんどのWebサーバは静的なコンテンツを返す時、そのファイルの拡張子によって、適切なContent−typeを返す様に成っているものの、この設定が間違っていると、クライアント側で、ファイルが開けないとか、いつもと違うアプリが開いたりとか問題が発生する。


- Date

コンテンツの日付を返す、

- Set-Cookie

Cokkieを設定する。
次回同じサイトにアクセスするときは、ブラウザはリクエストヘッダーにCookieの値を付けて要求を出す。
これによって、サーバー側で、アクセスして来たユーザーを追跡可能。

## telnetを使ってHTTPを話してみる。

ここまではブラウザがHTTPで通信する所を見てきましたが、自らの手でHTTPで通信しよう！

#### telnetとは

telnetとは、汎用的な双方向8bit通信を提供する端末間及びプロセス間の通信プロトコルで、リモートコンピュータとやり取りするための仕組み。
暗号化サれていないSSH通信みたいな感じ。

ルータにログインして各種の設定を行うときとかによく使う。

telnetクライアントは単純にテキストをやりとりする機能のみ。

通常は、Telnetサーバーに接続。

しかし、接続先を変更すると、任意のサービスに接続可能。

例えば、HTTPのポート80番にHTTPに従って文字列を送れば、サーバーからレスポンスを得られる。

#### 実際にやってみよう。

1. `telnet www.amazon.co.jp 80`
1. `GET / HTTP/1.1`
1. `HOST: www.amatezon.co.jp`

。。。返事が帰ってこない。
うーん大学のプロキシ経由だとダメなのか？
プロキシサーバーを経由して飛ばしてみる。

1. `telnet proxy.noc.titech.ac.jp 3128`
1. `GET http://google.co.jp/ HTTP/1.1`

```
HTTP/1.0 301 Moved Permanently
Location: http://www.google.co.jp/
Content-Type: text/html; charset=UTF-8
Date: Sun, 27 Dec 2015 06:42:09 GMT
Expires: Tue, 26 Jan 2016 06:42:09 GMT
Cache-Control: public, max-age=2592000
Server: gws
Content-Length: 221
X-XSS-Protection: 1; mode=block
X-Frame-Options: SAMEORIGIN
X-Cache: MISS from proxy.noc.titech.ac.jp
X-Cache-Lookup: MISS from proxy.noc.titech.ac.jp:3128
Connection: keep-alive

<HTML><HEAD><meta http-equiv="content-type" content="text/html;charset=utf-8">
<TITLE>301 Moved</TITLE></HEAD><BODY>
<H1>301 Moved</H1>
The document has moved
<A HREF="http://www.google.co.jp/">here</A>.
</BODY></HTML>

```


試しにAmazonでkindleを検索する。

1. `telnet proxy.noc.titech.ac.jp 3128`
1. `GET http://www.amazon.co.jp/s?field-keywords=kindle HTTP/1.1`
1. HTMLがドーンと帰ってきた。検索結果だね。

## まとめ

今回は、chromeのデベロッパーツールを使って、HTTPプロトコルの概要を知る。
そして、TelnetクライアントでHTTPプロトコルで通信してみた。

WebブラウザとWebサーバではこのようなやり取りがなされているわけですね。

Web APIを使った開発や、Webアプリケーション、API自体の開発をするには、HTTPの理解が必須。

開発者の多くは、APIを開発する問には、Chromeのデベロッパーツールを使って通信を覗きながら開発しています。

次の章では、インターネットからみえない、プライベートネットワークを構築し、セキュリティを高める手法を紹介します。


乞うご期待！



## ちょっとだけ追加

### HTTPサーバの気持ちを考える。

HTTPプロトコルでもう少し遊ぶ。
Telnetを使ってクライアント側の気持ちが少し解りました。

次は、サーバーとしてHTTPを話してみましょう。

ここでは、node.jsを用いてWebサーバーをお気軽に作成。

そこでリクエストヘッダーを跳ね返すプログラムを配置。

Telnetからアクセスして、どのような結果が得られるのかを見てみます。

#### node.js

node.jsはJavaScriptでサーバーサイドのプログラムを作れる開発・実行環境です。
ブロックしないI/O処理を実現し、高速で大量のアクセスを捌けます。

#### テストファイルのダウンロード

`http://aws-network-book.s3-ap-northeast-1.amazonaws.com/app.js`からダウンロード。

#### node.jsのインストール

1. `cd ~`
1. `wget http://nodejs.org/dist/v0.10.26/node-v0.10.26-linux-x64.tar.gz`  
今回はちょっとドキュメントに従う。
何かnpmとかをapt-getして来て出来るっぽいけど、あんまりEC2に負荷掛けたくない。  
1. `tar xzvf node-v0.10.26-linux-x64.tar.gz`
1. `export PATH=$PATH:~/node-v0.10.26-linux-x64/bin`
1. これで、nodeコマンドが使える様になりました。

#### ポートの解放

1. AWSのセキュリティグループを選択。
1. WEB-SG。
1. 編集→ルール追加
1. 8080ポートに0.0.0.0/0を追加。
1. 保存。

#### telnetで取得してみる。

GETした場合は。
```
GET / HTTP/1.1 
User-Agent: OreOreAgent
              
HTTP/1.1 200 OK
Content-Type: application/json
Content- Length: 46
Date: Sun, 27 Dec 2015 07:38:51 GMT
Connection: keep-alive
Transfer-Encoding: chunked

2e
{"RequestHeader":{"user-agent":"OreOreAgent"}}
0

```

POSTした場合は。

```
o198-156:aws_test saturn3486$ telnet ec2-52-192-87-75.ap-northeast-1.compute.amazonaws.com 8080
Trying 52.192.87.75...
Connected to ec2-52-192-87-75.ap-northeast-1.compute.amazonaws.com.
Escape character is '^]'.
POST / HTTP/1.0
User-Agent: OreOreAgent
Content-Length: 3

abc
HTTP/1.1 200 OK
Content-Type: application/json
Content- Length: 87
Date: Sun, 27 Dec 2015 07:47:42 GMT
Connection: close

{"RequestHeader":{"user-agent":"OreOreAgent","content-length":"3"},"RequestBody":"abc"}Connection closed by foreign host.

```

POSTの場合は、Content-Lengthを超えてPOSTすると、すぐクローズ、
それ以下の場合でもダメっぽい。

３文字だけ打ち込んでそのあとエンターすると適切なレスポンスが得られる。
