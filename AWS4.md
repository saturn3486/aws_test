# 触って学ぶクラウドインフラAWS　Chapter4

昨日は、サブネット内にサーバーを構築、SSHを使った接続をした。

今回は、サーバーへのApacheの導入。

Webサーバーになってもらいます！

## Apache HTTP Serverのインストール

#### Apache のインストール手順

では実際にやってみよう！

1. EC2インスタンスへSSHを使ってログイン（`ssh -i ./private/my-key.pem ec2-user@52.192.87.75`）  
githubにちょこちょこ資料とこアップしながらやってるので、アップしたくない奴はフォルダにまとめています。

1. apacheのインストール（`sudo yum -y install httpd`）  
`yum`コマンドはアプリのインストール用のコマンド、-yはユーザーの確認ナシですぐインストールするオプション。  
ec2-userでログインしていますが、これは管理者（`root`）では無いので、sudoが必要になります。 

1. apacheの起動(`sudo service httpd start`)  
serviceコマンドは、指定したコマンド（ココではApache本体）を「起動start」、「停止stop」、「再起動restart」するコマンド

1. 自動起動するように構成する、（`sudo chkconfig httpd on`）  
今のままだと、再起動する度に起動コマンドを打ち込まないとイケないので、自動で起動するようにしてやる。 
`chkconfig`コマンドは、自動起動に関しての設定を「on」「off」「--list自動起動するリストを出す」を行うコマンド。  
chkconfigの結果は、0-6までがon,offのどちらに成っているかを表示する。
この数字は、ランレベル、と呼ばれシステムの状態を表す。
例えば、停止状態が0,通常起動状態では3だったりする。3:onならOK.

1. Apacheのプロセスを確認する（`ps -ax|grep httpd`）  
httpdのプロセスが稼働しているか確認する。
psコマンドはプロセスを表示する。`-ax`は`-a`（全て表示）、`-x`（他の端末に結び付けられているプロセスも表示）の組み合わせ。ここでは、結果をgrepして手っ取り早く検索。  
`24370 ?        Ss     0:00 /usr/sbin/httpd`が表示サれてればOK。
最初の数字はPID（プロセスの番号。環境により異なる。）

1. ネットワークの待受状態を確認する。（`sudo lsof -i -n -P`）  
これポート番号の確認のところでも出たね。
`httpd     24372   apache    4u  IPv6  41315      0t0  TCP *:80 (LISTEN)`TCPポート80番で待ってるらしい。
何かIPv6で待ってるっぽい。実際には、IPv4互換アドレスとかでIPv6として接続したりするのでIPv4でも通信可能。
httpsの443番ポートは、暗号化通信（SSL通信）用のポートなんだけど、デフォルトではオフになってるのでココでは開いてない。
SSL通信する場合は、SSL証明書の設定をしないと有効にならない。→ダルい。ので省く。

## ファイアウォールの設定

Apacheのインストールして起動させたので、既にWebサーバーとして機能しているはず。

ブラウザでアクセスしてみる。

#### Webサーバーにブラウザでアクセスしてみる。

1. マネジメントコンソールから、EC2を選択して東京リージョンの自分のWebサーバーのインスタンスを確認
1. パブリックIPアドレスを指定してブラウザでアクセス。

→無理！
現状では、ポート80番がファイアウォールによってブロックされてる。
セキュリティグループの自分の選択したやつを見れば分かるけど、SSH用のポート22番しか解放サれてない。。。
次はそのポートを設定しよう。

#### ファイアウォールの構成（httpのポート80番を開けてみる）

1. AWSの左のメニューからセキュリティグループを選択
1. 自分で作った、WEB-SGをチェック
1. 外部からこのインスタンスに向けてのデータなので、インバウンドタブを下でクリック
1. 編集をクリック
1. ルールの追加をクリック
1. カスタムTCPルール、TCPのポート80番、任意の場所（すべてのホスト）、ちゃんとCIDRが`0.0.0.0/0`だけ確認
1. 保存する。
1. 再度ブラウザでアクセスしてみる。
1. Apacheののテストページが表示される！  
通常は、`/var/www/html`のディレクトリの内容が表示されますが、ディレクトリが空の場合は、`/var/www/error/noindex.html`が表示される。

## ドメイン名でも付けますか

ウェブブラウザでパブリックIPアドレスで接続出来た！
だが覚えづらい。
そんな時のドメイン名。

#### ドメインに関する基礎知識

##### ■ドメイン名の構造

IPアドレスの英数字バージョン。やはり一意じゃないと意味が無い。

**ドメインの階層**

ドメイン名はピリオドで区切られてます。
`google.co.jp`みたいな感じ。

もっとも右側`.jp`をトップレベルドメイン  
左へ移動していって`.co`第2レベルドメイン、`google`第3レベルドメイン  
ってな感じ。

`.jp`が日本の事。  
`.co`がカンパニー系  
`google`って会社。


**ドメインの管理**

ドメイン名は、やっぱりICANNが管理していて、トップレベルドメインごとにそれぞれの事業者が管理している。
jpならJPRSとかいうジャパンレジストリサービスとかいう会社。

#### DNSとは

ドメイン名とIPアドレスを変換。このことを名前解決と呼ぶ。
DNS domain name service。
DNSのシステムは、DNSのサーバ群からなる巨大分散型データベース。

DNSサーバーは、トップレベルドメインのDNSサーバーから段々下がっていくイメージで名前解決する。

ではではDNSサーバーを構築しましょうか。

#### インスタンスの名前解決を有効にする！

Amazon VPCには、VPN(virtual private network)内の名前解決をするオプション機能があり、その機能を有効にすると、インスタンスにDNS名が設定される。

1. VPCの設定ページを開く。
1. 自分のVPC領域をチェック
1. アクションから、DNS解決の編集、DNSホスト名の編集を選び、それぞれ両方をチェック。
1. 下の概要から、DNS解決、DNSホスト名が「はい」になっている事を確認。  
ホスト名の方をチェックしたことで、インスタンスにホスト名が設定されます。  
DNS解決の設定の方は、AmazonVPCの提供するDNSサーバーを利用する設定になる。
自分のDNSサーバを使う場合などは、チェックを外して設定することになる。

#### ドメイン名でアクセスしてみる！

1. EC2のメニューを開く。
1. 自分のインスタンスを確認。  
パブリックDNSとプライベートDNSは名前の通り、
前者はネットから参照できるDNS名。
後者がVPC内からしか参照出来ないDNS名。
1. パブリックDNSをURL欄に入力
1. アクセス出来た！


### DNSサーバーの動きを確認する。

nslookupコマンドを使ってDNSの名前解決を観察。
もっと細かいのはdigコマンドから可能。でもココでは触らなかったり。

#### クライアントからnslookup

`nslookup ec2-52-192-87-75.ap-northeast-1.compute.amazonaws.com`を実行するだけ。

実行結果

```
o198-156:aws_test saturn3486$ nslookup ec2-52-192-87-75.ap-northeast-1.compute.amazonaws.com
Server:     131.112.125.58
Address:    131.112.125.58#53

Non-authoritative answer:
Name:   ec2-52-192-87-75.ap-northeast-1.compute.amazonaws.com
Address: 52.192.87.75

```

逆引き。IPアドレス→DNS
`nslookup 52.192.87.75`


```
o198-156:aws_test saturn3486$ nslookup 52.192.87.75
Server:     131.112.125.58
Address:    131.112.125.58#53

Non-authoritative answer:
75.87.192.52.in-addr.arpa   name = ec2-52-192-87-75.ap-northeast-1.compute.amazonaws.com.

Authoritative answers can be found from:
192.52.in-addr.arpa nameserver = x1.amazonaws.com.
192.52.in-addr.arpa nameserver = x2.amazonaws.com.
192.52.in-addr.arpa nameserver = x4.amazonaws.org.
192.52.in-addr.arpa nameserver = x3.amazonaws.org.
192.52.in-addr.arpa nameserver = pdns1.ultradns.net.
pdns1.ultradns.net  internet address = 204.74.108.1
pdns1.ultradns.net  has AAAA address 2001:502:f3ff::1
```


#### サーバからnslookup


```
[ec2-user@ip-10-0-1-10 ~]$ nslookup www.google.co.jp
Server:     10.0.0.2
Address:    10.0.0.2#53

Non-authoritative answer:
Name:   www.google.co.jp
Address: 216.58.220.195
```

## まとめ

Apacheのインストールを行いました。
その後、httpによる通信用のポートを開放し、Webサーバとして動作することを確認しました。
WebブラウザからパブリックIPアドレスを用いたアクセスしました。
DNS名が設定される様にVPC内の設定を変更し、DNS名を用いてアクセスしました。
nslookupを用いてすこしだけ、DNSサーバとのやり取りを確認しました。

つぎは、Webサーバとブラウザの相互のやり取りを確認してみましょう！

乞うご期待！  





----------



## 付録

#### 実行結果群

#### Apache のインストール手順のところ。

2の後。
```bash
[ec2-user@ip-10-0-1-10 ~]$ sudo yum -y install httpd
....
インストール:
  httpd.x86_64 0:2.2.31-1.6.amzn1                                                         

依存性関連をインストールしました:
  apr.x86_64 0:1.5.0-2.11.amzn1                apr-util.x86_64 0:1.4.1-4.17.amzn1        
  apr-util-ldap.x86_64 0:1.4.1-4.17.amzn1      httpd-tools.x86_64 0:2.2.31-1.6.amzn1     
  mailcap.noarch 0:2.1.31-2.7.amzn1           

完了しました!
```

3の後。
```
ec2-user@ip-10-0-1-10 ~]$ sudo service httpd start
Starting httpd: httpd: apr_sockaddr_info_get() failed for ip-10-0-1-10
httpd: Could not reliably determine the server's fully qualified domain name, using 127.0.0.1 for ServerName
                                                           [  OK  ]
```

4の後。
```
[ec2-user@ip-10-0-1-10 ~]$ sudo chkconfig --list httpd
httpd           0:off   1:off   2:off   3:off   4:off   5:off   6:off
[ec2-user@ip-10-0-1-10 ~]$ sudo chkconfig httpd on
[ec2-user@ip-10-0-1-10 ~]$ sudo chkconfig --list httpd
httpd           0:off   1:off   2:on    3:on    4:on    5:on    6:off
```

5の後。
```
[ec2-user@ip-10-0-1-10 ~]$ ps -ax|grep httpd
Warning: bad syntax, perhaps a bogus '-'? See /usr/share/doc/procps-3.2.8/FAQ
24370 ?        Ss     0:00 /usr/sbin/httpd
24372 ?        S      0:00 /usr/sbin/httpd
24373 ?        S      0:00 /usr/sbin/httpd
24374 ?        S      0:00 /usr/sbin/httpd
24375 ?        S      0:00 /usr/sbin/httpd
24376 ?        S      0:00 /usr/sbin/httpd
24377 ?        S      0:00 /usr/sbin/httpd
24378 ?        S      0:00 /usr/sbin/httpd
24379 ?        S      0:00 /usr/sbin/httpd
```

6の後。
```
httpd     24370     root    4u  IPv6  41315      0t0  TCP *:80 (LISTEN)
httpd     24372   apache    4u  IPv6  41315      0t0  TCP *:80 (LISTEN)
httpd     24373   apache    4u  IPv6  41315      0t0  TCP *:80 (LISTEN)
httpd     24374   apache    4u  IPv6  41315      0t0  TCP *:80 (LISTEN)
httpd     24375   apache    4u  IPv6  41315      0t0  TCP *:80 (LISTEN)
httpd     24376   apache    4u  IPv6  41315      0t0  TCP *:80 (LISTEN)
httpd     24377   apache    4u  IPv6  41315      0t0  TCP *:80 (LISTEN)
httpd     24378   apache    4u  IPv6  41315      0t0  TCP *:80 (LISTEN)
httpd     24379   apache    4u  IPv6  41315      0t0  TCP *:80 (LISTEN)
```