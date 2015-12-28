## 触って学ぶクラウドインフラAWS　Chapter6

## プライベートサブネットとは。。。

インターネットに接続している限りは、攻撃を可能性が有る、
そこで、セキュリティを高める方法として検討したいのが、インターネットから隔離したプライベートサブネット。
いま作っているWordoressを用いたWebサイトに関しても、DBサーバーは是非プライベートサブネット内に配置して、セキュリティを高めたいですな。


#### プライベートサブネット

システムを構成するサーバ群の中には、インターネットから直接アクセスサれては困るものが有ります。

それが、データベースなどのバックエンド師ステッ無は、その典型例です。

隠したいサーバーは、インターネットから値接続できないサブネットに配置するようにします、
このようなサブネットの事をプライベートサブネットと呼びます、


## プライベートサブネットを作る！

前に作成してるけど、今回も一応新規に作製する方法を載せておきます！



#### アベイラビリティゾーンを確認する

パブリックサブネットとプライベートサブネットが異なるアベイラビリティゾーンに存在する場合は、通信が遅くなったり余計なコストが掛かることになります。

そのため、今回は、まずパブリックサブネットのアベイラビリティゾーンを確認し、同じリージョンの同じアベイラビリティゾーンにプライベートサブネットを作成することにします。

※とは言っても異なるアベイラビリティゾーンも高速な回線で接続サれているので、そのレイテンシーは数ミリ秒と言った所でしょう。。。

1. AWSを開いて、サブネットを確認する。  
`アベイラビリティーゾーン:ap-northeast-1a`

#### プライベートサブネットを作る

1. 前回作ったのでココは省くが、同じアベイラビリティゾーンを使ってサブネットを作る。  
CIDR->`10.0.2.0/24`

#### ルートテーブルを確認する。

1. AWSからサブネットを選択して、ルートテーブルを確認する。  
おそらく、自分のVPC内に対する送信をローカル（自分のVPC内）に流すという設定になってるはず。
インターネットには接続しないので、これでOK。


## プライベートサブネットにサーバーを構築する。　

#### 実際の手順

1. AWSでEC2へ。
1. Amazon Linux AMI選択
1. t2.micro選択
1. VPC領域を選択。
1. プライベートサブネットを選択
1. 自動割当パブリックIPを無効化。
1. プライベートIPはとりあえず、`10.0.2.10/24`を選択
1. サーバー名は「DBサーバー」
1. セキュリティグループは新たに選択「DB-SG」
1. ポートはSSHとMySQL（3306）を解放。送信元は任意。

### pingコマンドを使った疎通確認

このDBサーバーがWebサーバーからアクセスできるかを確認。
この時に使うのがpingコマンド。

pingコマンドは、ICMP(Internet Control Message Protocol)を使っている。

pingコマンドを実行すると、対象のホストへICMPエコー要求（Echo Request）というパケットを送信します。

pingコマンドでは、このICMPエコー要求、及びICMPエコー応答から、疎通を確認したり、時間を計測したりする。

##### ICMPが通るように設定。

まだセキュリティグループではICMPを設定していなかったので、これを設定。

1. AWSのセキュリティグループでDB-SGを選択
1. インバウンドを開いて、編集
1. ルールの追加から、全てのICMPに任意の場所からの送信を許可。


#### ping！

1. sshを用いてWebサーバーへログイン  
(`ssh -i ./private/my-key.pem ec2-user@52.192.87.75`)
1. ping コマンド実行(`ping 10.0.2.10`)


```
[ec2-user@ip-10-0-1-10 ~]$ ping 10.0.2.10
PING 10.0.2.10 (10.0.2.10) 56(84) bytes of data.
64 bytes from 10.0.2.10: icmp_seq=1 ttl=64 time=0.539 ms
64 bytes from 10.0.2.10: icmp_seq=2 ttl=64 time=0.579 ms
64 bytes from 10.0.2.10: icmp_seq=3 ttl=64 time=0.499 ms
64 bytes from 10.0.2.10: icmp_seq=4 ttl=64 time=0.535 ms
64 bytes from 10.0.2.10: icmp_seq=5 ttl=64 time=0.627 ms
64 bytes from 10.0.2.10: icmp_seq=6 ttl=64 time=0.647 ms
64 bytes from 10.0.2.10: icmp_seq=7 ttl=64 time=0.604 ms
^C
--- 10.0.2.10 ping statistics ---
7 packets transmitted, 7 received, 0% packet loss, time 6691ms
rtt min/avg/max/mdev = 0.499/0.575/0.647/0.057 ms
```
1. ちなみにローカル環境からWebサーバーへpingを飛ばしてみる。  
```
saturn3486PC:aws_test saturn3486$ ping ec2-52-192-87-75.ap-northeast-1.compute.amazonaws.com
PING ec2-52-192-87-75.ap-northeast-1.compute.amazonaws.com (52.192.87.75): 56 data bytes
Request timeout for icmp_seq 0
Request timeout for icmp_seq 1
^C
--- ec2-52-192-87-75.ap-northeast-1.compute.amazonaws.com ping statistics ---
3 packets transmitted, 0 packets received, 100.0% packet loss
```

## 踏み台サーバーを経由してSSHで接続する。

今回、DBサーバーをMySQLをインストールする必要が有りますが、どうやってSSH接続すれば良いのか？

まあ答えは簡単。

ローカルから、WebサーバーへいつもどおりSSH。

WebサーバーからDBサーバーへSSH。

これでローカル→DBサーバーへ繋ぐ事が可能です！

VPC内の通信なので、WebサーバからはDBサーバへアクセス出来るわけですね。

#### 秘密鍵のアップロード。

DBサーバにSSHするためには、Webサーバに秘密鍵を置いておかねばいけません。

ローカルにしかないので、そのファイルをDBサーバへアップロードする必要があるというわけです。

このように、サーバにファイルを転送するときは、SCP（Secure Copy）というプロトコルを用います。

1. `scp -i mykey.pem ec2-user@ ec2-52-192-87-75.ap-northeast-1.compute.amazonaws.com:~/`
1. ```
saturn3486PC:aws_test saturn3486$ scp -i ./private/my-key.pem ec2-user@ 52.192.87.75:~/
Permission denied (publickey).
lost connection
```うぐぐ。  
1. sshdを再起動してみる。  `sudo service sshd restart`
1. 現状を確認してみる。
SSHは繋がる。→ホスト名とか公開鍵とかは合ってるはず。
SCPの設定か？
1. sshとscpで-vオプションを付ける。
scpの時。
```
ssaturn3486PC:aws_test saturn3486$scp -v -i ./private/my-key.pem ec2-user@ 52.192.87.75:~/ 
Executing: program /usr/bin/ssh host 52.192.87.75, user (unspecified), command scp -v -t ~/
OpenSSH_6.2p2, OSSLShim 0.9.8r 8 Dec 2011
debug1: Reading configuration data /Users/saturn3486/.ssh/config
debug1: Reading configuration data /etc/ssh_config
debug1: /etc/ssh_config line 20: Applying options for *
debug1: /etc/ssh_config line 103: Applying options for *
debug1: Connecting to 52.192.87.75 [52.192.87.75] port 22.
debug1: Connection established.
debug1: identity file ./private/my-key.pem type -1
debug1: identity file ./private/my-key.pem-cert type -1
debug1: Enabling compatibility mode for protocol 2.0
debug1: Local version string SSH-2.0-OpenSSH_6.2
debug1: Remote protocol version 2.0, remote software version OpenSSH_6.6.1
debug1: match: OpenSSH_6.6.1 pat OpenSSH*
debug1: SSH2_MSG_KEXINIT sent
debug1: SSH2_MSG_KEXINIT received
debug1: kex: server->client aes128-ctr hmac-md5-etm@openssh.com none
debug1: kex: client->server aes128-ctr hmac-md5-etm@openssh.com none
debug1: SSH2_MSG_KEX_DH_GEX_REQUEST(1024<1024<8192) sent
debug1: expecting SSH2_MSG_KEX_DH_GEX_GROUP
debug1: SSH2_MSG_KEX_DH_GEX_INIT sent
debug1: expecting SSH2_MSG_KEX_DH_GEX_REPLY
debug1: Server host key: RSA c9:3d:80:d0:c5:0e:97:bf:fb:a7:8d:f2:ce:64:13:44
debug1: Host '52.192.87.75' is known and matches the RSA host key.
debug1: Found key in /Users/saturn3486/.ssh/known_hosts:8
debug1: ssh_rsa_verify: signature correct
debug1: SSH2_MSG_NEWKEYS sent
debug1: expecting SSH2_MSG_NEWKEYS
debug1: SSH2_MSG_NEWKEYS received
debug1: Roaming not allowed by server
debug1: SSH2_MSG_SERVICE_REQUEST sent
debug1: SSH2_MSG_SERVICE_ACCEPT received
debug1: Authentications that can continue: publickey
debug1: Next authentication method: publickey
debug1: Offering RSA public key: /Users/saturn3486/.ssh/id_rsa
debug1: Authentications that can continue: publickey
debug1: Trying private key: ./private/my-key.pem
debug1: read PEM private key done: type RSA
debug1: Authentications that can continue: publickey
debug1: No more authentication methods to try.
Permission denied (publickey).
lost connection
```
sshの時。
```
saturn3486PC:aws_test saturn3486$ ssh -v -i ./private/my-key.pem ec2-user@52.192.87.75
OpenSSH_6.2p2, OSSLShim 0.9.8r 8 Dec 2011
debug1: Reading configuration data /Users/saturn3486/.ssh/config
debug1: Reading configuration data /etc/ssh_config
debug1: /etc/ssh_config line 20: Applying options for *
debug1: /etc/ssh_config line 103: Applying options for *
debug1: Connecting to 52.192.87.75 [52.192.87.75] port 22.
debug1: Connection established.
debug1: identity file ./private/my-key.pem type -1
debug1: identity file ./private/my-key.pem-cert type -1
debug1: Enabling compatibility mode for protocol 2.0
debug1: Local version string SSH-2.0-OpenSSH_6.2
debug1: Remote protocol version 2.0, remote software version OpenSSH_6.6.1
debug1: match: OpenSSH_6.6.1 pat OpenSSH*
debug1: SSH2_MSG_KEXINIT sent
debug1: SSH2_MSG_KEXINIT received
debug1: kex: server->client aes128-ctr hmac-md5-etm@openssh.com none
debug1: kex: client->server aes128-ctr hmac-md5-etm@openssh.com none
debug1: SSH2_MSG_KEX_DH_GEX_REQUEST(1024<1024<8192) sent
debug1: expecting SSH2_MSG_KEX_DH_GEX_GROUP
debug1: SSH2_MSG_KEX_DH_GEX_INIT sent
debug1: expecting SSH2_MSG_KEX_DH_GEX_REPLY
debug1: Server host key: RSA c9:3d:80:d0:c5:0e:97:bf:fb:a7:8d:f2:ce:64:13:44
debug1: Host '52.192.87.75' is known and matches the RSA host key.
debug1: Found key in /Users/saturn3486/.ssh/known_hosts:8
debug1: ssh_rsa_verify: signature correct
debug1: SSH2_MSG_NEWKEYS sent
debug1: expecting SSH2_MSG_NEWKEYS
debug1: SSH2_MSG_NEWKEYS received
debug1: Roaming not allowed by server
debug1: SSH2_MSG_SERVICE_REQUEST sent
debug1: SSH2_MSG_SERVICE_ACCEPT received
debug1: Authentications that can continue: publickey
debug1: Next authentication method: publickey
debug1: Offering RSA public key: /Users/saturn3486/.ssh/id_rsa
debug1: Authentications that can continue: publickey
debug1: Trying private key: ./private/my-key.pem
debug1: read PEM private key done: type RSA
debug1: Authentication succeeded (publickey).
Authenticated to 52.192.87.75 ([52.192.87.75]:22).
debug1: channel 0: new [client-session]
debug1: Requesting no-more-sessions@openssh.com
debug1: Entering interactive session.
debug1: Requesting X11 forwarding with authentication spoofing.
debug1: Sending environment.
debug1: Sending env LANG = ja_JP.UTF-8
debug1: Remote: No xauth program; cannot forward with spoofing.
X11 forwarding request failed on channel 0
```
**途中まではほぼ一緒。**  
SCP
```
debug1: Authentications that can continue: publickey
debug1: Trying private key: ./private/my-key.pem
debug1: read PEM private key done: type RSA
debug1: Authentications that can continue: publickey
```
SSH
```
debug1: Authentications that can continue: publickey
debug1: Trying private key: ./private/my-key.pem
debug1: read PEM private key done: type RSA
debug1: Authentication succeeded (publickey).
Authenticated to 52.192.87.75 ([52.192.87.75]:22).
```
Publickeyの認証で、そのまま通ってない？えぇぇぇ。  
1. 別の方法で送信。`sftp -i my-key.pem ec2-user@ec2-52-192-87-75.ap-northeast-1.compute.amazonaws.com` 　
`PUT private/my-key.pem`
イケた！！！。

なぜSSHは繋がるのにSCPは無理だったんだろうか？
双方向の認証でも必要になるのか？
SCPの通信の流れを確認しないとなんとも。
1. `ssh -i my-key.pem ec2-user@10.0.2.10`
やっとつながったぁぁぁぁ。
うぉぉぉ。。。。こんなところでドハマリするとは。。。
原因何なんだろうか？
**要検証問題。EC2インスタンス側の、sshd_configをいじるのかな・・・。**

成功結果。
```
saturn3486PC:aws_test saturn3486$ ssh -i private/my-key.pem ec2-user@ec2-52-192-87-75.ap-northeast-1.compute.amazonaws.com
Last login: Mon Dec 28 09:14:09 2015 from 103.5.140.185

       __|  __|_  )
       _|  (     /   Amazon Linux AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-ami/2015.09-release-notes/
11 package(s) needed for security, out of 27 available
Run "sudo yum update" to apply all updates.
[ec2-user@ip-10-0-1-10 ~]$ ssh -i my-key.pem ec2-user@10.0.2.10
Last login: Mon Dec 28 09:17:51 2015 from ip-10-0-1-10.ap-northeast-1.compute.internal

       __|  __|_  )
       _|  (     /   Amazon Linux AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-ami/2015.09-release-notes/
[ec2-user@ip-10-0-2-10 ~]$ 
```

## まとめ

この章では、インターネットから直接接続させないプライベートサブネットに関して色々やりました。
プライベートIPアドレスのみをもたせるプライベートサブネットを作製しました。
プライベートサブネットは、ローカル環境からSSHによる接続が出来ないので、それように踏み台サーバーであるWebサーバを用いてSSH接続に成功しました。
踏み台サーバには、プライベートサブネット上のインスタンスの秘密鍵をアップロードしてあげました。

しかし、現状では、プライベートサブネット内のインスタンスからインターネットへ接続することが出来ません。
このままだと、必要なアプリケーションをダウンロードしたり、OSをアップデートしたり、ということが出来ないので、

次回はNATサーバーを設置し、ローカルから一方通行でインターネットへの接続を可能にします！

乞うご期待。


#### 感想

ハゲそう。
結局SCP使えない問題を何とか解決したい所。
どうやってそれを確認しましょうか・・・。
