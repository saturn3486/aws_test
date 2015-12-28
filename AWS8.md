## 触って学ぶクラウドインフラAWS　Chapter8

前回はNATサーバーを設置して、プライベートサブネットからのインターネット接続を可能にしました！

というわけで、プライベートサブネット内のDBサーバーの環境を構築しましょう！

そしてとうとう、WebサーバへのWordpressのインストール！！

ブログサーバーが要約完成する。。。

このまとめも終わりが近づいております。。。

では行ってみましょう！！！

### 今回の内容

1. DBサーバーの構成  
DBサーバにMySQLをインストール。
WordPressから保存出来るようにデータベースを作製します。
1. WebサーバへWordPressのインストール  
1. WordPressの初期設定。
WordpressでDBサーバ内のMySQLを利用するように設定します。

#### DBサーバにMySQLをインストールする。

1. DBサーバへSSH接続。
1. MySQLのインストール→`sudo yum -y install mysql-server`
1. MySQLの自動起動→`sudo service mysqld start`
1. 管理者パスワードの設定→`mysqladmin -u root password`


#### WordPress用のデータベースを作成

1. mysqlの実行→`mysql -u root -p`  
パスワード認証でrootユーザでログイン
1. wordpress用のデータベースを作成→`CREATE DATABASE wordpress DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;`  
wordpressって言うDBを作成。utf8が基本で,collate(照合順序)は`utf8_general_ci`。
1. ユーザの作成→`grant all on wordpress.* to wordpress@"%" identified by 'wordpresspasswd';`  
wordpress.*はwordpress内の全テーブル。それを、`wordpress@"%"`wordpressにログインしてる全てのホストに与える。
ログインパスワードは、wordpresspasswd。
1. 上記内容の反映→`flush privileges;`
1. 確認→`select user, host from mysql.user;`
```
mysql> select user, host from mysql.user;
+-----------+--------------+
| user      | host         |
+-----------+--------------+
| wordpress | %            |
| root      | 127.0.0.1    |
| root      | ::1          |
|           | ip-10-0-2-10 |
| root      | ip-10-0-2-10 |
|           | localhost    |
| root      | localhost    |
+-----------+--------------+
7 rows in set (0.00 sec)
```
1. 自動起動の設定→`sudo chkconfig mysqld on`
```
[ec2-user@ip-10-0-2-10 ~]$ chkconfig --list mysqld
mysqld          0:off   1:off   2:on    3:on    4:on    5:on    6:off
```

## WebサーバーにWordpressをインストール。

#### 周辺ライブラリの導入とDBサーバーのMySQLをWebサーバーから利用するテスト。

1. SSHでWebサーバーへアクセス。→　`ssh -i private/my-key.pem ec2-user@ec2-52-192-87-75.ap-northeast-1.compute.amazonaws.com`
1. PHPと周辺ライブラリのインストール。→`sudo yum -y install php php-mysql php-mbstring`
1. mysqlコマンドのインストール→`sudo yum -y install mysql`
1. DBサーバのMySQLへのアクセス→`mysql -h 10.0.2.10 -u wordpress -p`
1. 接続されたらそのまま抜ける。→`exit;`
1. webサーバのホームへカレントディレクトリを移動→`cd ~`
1. Wordpressのダウンロード→`wget http://ja.wordpress.org/latest-ja.tar.gz`
1. ファイルの解答→`tar xzvf latest-ja.tar.gz`
1. wordpress内へ移動→`cd wordpress`
1. 現在のディレクトリの中身すべてをapacheの公開用フォルダへコピー→`sudo cp -r * /var/www/html/`
1. コピーしたディレクトリの所有者/グループをapacheに変更→`sudo chown apache:apache /var/www/html/`

#### wordpressを設定。

1. apacheの再起動→`sudo service httpd restart`
1. ウェブブラウザでWebサーバーへアクセス。`ec2-52-192-87-75.ap-northeast-1.compute.amazonaws.com`
1. さあ、はじめましょう！をクリック
1. DB名→`wordpress`、ユーザ名→`wordpress`、パスワード、DBのホスト名→`10.0.2.10`、テーブル接頭語→`wp_`→送信
1. インストール実行をクリック
1. 各々自分の情報を追加。
1. wordpressをインストール
1. ログイン
1. 管理ページへログインする。


## まとめ

以上でWordPressを用いたブログシステムの構築は終了です。

うぉぉぉ終わったぁぁぁｌ。

これで後はアップデートとかの時だけNATつければ何とかOK!

最後の最後に全サーバで`sudo yum update`して終わりとします笑

セキュリティのことにも考慮した、AWSを用いたネットワークインフラの作成。
いかがでしたでしょうか？

最後の最後に、TCP/Ipの仕組みに関して、詳細をご紹介して終了としましょうか。

最後の一回よろしくお願いします。

乞うご期待！  


## 付録
MySQLのインストール→`sudo yum -y install mysql-server`
```
[ec2-user@ip-10-0-2-10 ~]$ sudo yum -y install mysql-server
読み込んだプラグイン:priorities, update-motd, upgrade-helper
amzn-main/latest                                                   | 2.1 kB     00:00     
amzn-main/latest/group                                             |  35 kB     00:00     
amzn-main/latest/primary_db                                        | 3.4 MB     00:00     
amzn-updates/latest                                                | 2.3 kB     00:00     
amzn-updates/latest/group                                          |  35 kB     00:00     
amzn-updates/latest/updateinfo                                     | 275 kB     00:00     
amzn-updates/latest/primary_db                                     | 355 kB     00:00     
依存性の解決をしています
--> トランザクションの確認を実行しています。
---> パッケージ mysql-server.noarch 0:5.5-1.6.amzn1 を インストール
--> 依存性の処理をしています: mysql55-server >= 5.5 のパッケージ: mysql-server-5.5-1.6.amzn1.noarch
--> トランザクションの確認を実行しています。
---> パッケージ mysql55-server.x86_64 0:5.5.46-1.10.amzn1 を インストール
--> 依存性の処理をしています: real-mysql55(x86-64) = 5.5.46-1.10.amzn1 のパッケージ: mysql55-server-5.5.46-1.10.amzn1.x86_64
--> 依存性の処理をしています: real-mysql55-libs(x86-64) = 5.5.46-1.10.amzn1 のパッケージ: mysql55-server-5.5.46-1.10.amzn1.x86_64
--> 依存性の処理をしています: perl(Data::Dumper) のパッケージ: mysql55-server-5.5.46-1.10.amzn1.x86_64
--> 依存性の処理をしています: perl-DBD-MySQL(mysql55) のパッケージ: mysql55-server-5.5.46-1.10.amzn1.x86_64
--> 依存性の処理をしています: mysql55(alternatives) のパッケージ: mysql55-server-5.5.46-1.10.amzn1.x86_64
--> 依存性の処理をしています: perl(DBI) のパッケージ: mysql55-server-5.5.46-1.10.amzn1.x86_64
--> 依存性の処理をしています: mysql-config のパッケージ: mysql55-server-5.5.46-1.10.amzn1.x86_64
--> トランザクションの確認を実行しています。
---> パッケージ mysql-config.x86_64 0:5.5.46-1.10.amzn1 を インストール
---> パッケージ mysql55.x86_64 0:5.5.46-1.10.amzn1 を インストール
---> パッケージ mysql55-libs.x86_64 0:5.5.46-1.10.amzn1 を インストール
---> パッケージ perl-DBD-MySQL55.x86_64 0:4.023-5.23.amzn1 を インストール
---> パッケージ perl-DBI.x86_64 0:1.627-4.8.amzn1 を インストール
--> 依存性の処理をしています: perl(RPC::PlClient) >= 0.2000 のパッケージ: perl-DBI-1.627-4.8.amzn1.x86_64
--> 依存性の処理をしています: perl(RPC::PlServer) >= 0.2001 のパッケージ: perl-DBI-1.627-4.8.amzn1.x86_64
---> パッケージ perl-Data-Dumper.x86_64 0:2.145-3.5.amzn1 を インストール
--> トランザクションの確認を実行しています。
---> パッケージ perl-PlRPC.noarch 0:0.2020-14.7.amzn1 を インストール
--> 依存性の処理をしています: perl(Net::Daemon) >= 0.13 のパッケージ: perl-PlRPC-0.2020-14.7.amzn1.noarch
--> 依存性の処理をしています: perl(Compress::Zlib) のパッケージ: perl-PlRPC-0.2020-14.7.amzn1.noarch
--> 依存性の処理をしています: perl(Net::Daemon::Test) のパッケージ: perl-PlRPC-0.2020-14.7.amzn1.noarch
--> 依存性の処理をしています: perl(Net::Daemon::Log) のパッケージ: perl-PlRPC-0.2020-14.7.amzn1.noarch
--> トランザクションの確認を実行しています。
---> パッケージ perl-IO-Compress.noarch 0:2.061-2.12.amzn1 を インストール
--> 依存性の処理をしています: perl(Compress::Raw::Zlib) >= 2.061 のパッケージ: perl-IO-Compress-2.061-2.12.amzn1.noarch
--> 依存性の処理をしています: perl(Compress::Raw::Bzip2) >= 2.061 のパッケージ: perl-IO-Compress-2.061-2.12.amzn1.noarch
---> パッケージ perl-Net-Daemon.noarch 0:0.48-5.5.amzn1 を インストール
--> トランザクションの確認を実行しています。
---> パッケージ perl-Compress-Raw-Bzip2.x86_64 0:2.061-3.11.amzn1 を インストール
---> パッケージ perl-Compress-Raw-Zlib.x86_64 1:2.061-4.1.amzn1 を インストール
--> 依存性解決を終了しました。

依存性を解決しました

==========================================================================================
 Package                      アーキテクチャー
                                          バージョン              リポジトリー       容量
==========================================================================================
インストール中:
 mysql-server                 noarch      5.5-1.6.amzn1           amzn-main         2.8 k
依存性関連でのインストールをします:
 mysql-config                 x86_64      5.5.46-1.10.amzn1       amzn-updates       49 k
 mysql55                      x86_64      5.5.46-1.10.amzn1       amzn-updates      7.5 M
 mysql55-libs                 x86_64      5.5.46-1.10.amzn1       amzn-updates      814 k
 mysql55-server               x86_64      5.5.46-1.10.amzn1       amzn-updates       13 M
 perl-Compress-Raw-Bzip2      x86_64      2.061-3.11.amzn1        amzn-main          33 k
 perl-Compress-Raw-Zlib       x86_64      1:2.061-4.1.amzn1       amzn-main          61 k
 perl-DBD-MySQL55             x86_64      4.023-5.23.amzn1        amzn-main         149 k
 perl-DBI                     x86_64      1.627-4.8.amzn1         amzn-main         855 k
 perl-Data-Dumper             x86_64      2.145-3.5.amzn1         amzn-main          49 k
 perl-IO-Compress             noarch      2.061-2.12.amzn1        amzn-main         298 k
 perl-Net-Daemon              noarch      0.48-5.5.amzn1          amzn-main          58 k
 perl-PlRPC                   noarch      0.2020-14.7.amzn1       amzn-main          39 k

トランザクションの要約
==========================================================================================
インストール  1 パッケージ (+12 個の依存関係のパッケージ)

総ダウンロード容量: 23 M
インストール容量: 80 M
Downloading packages:
(1/13): mysql-config-5.5.46-1.10.amzn1.x86_64.rpm                  |  49 kB     00:00     
(2/13): mysql-server-5.5-1.6.amzn1.noarch.rpm                      | 2.8 kB     00:00     
(3/13): mysql55-5.5.46-1.10.amzn1.x86_64.rpm                       | 7.5 MB     00:00     
(4/13): mysql55-libs-5.5.46-1.10.amzn1.x86_64.rpm                  | 814 kB     00:00     
(5/13): mysql55-server-5.5.46-1.10.amzn1.x86_64.rpm                |  13 MB     00:01     
(6/13): perl-Compress-Raw-Bzip2-2.061-3.11.amzn1.x86_64.rpm        |  33 kB     00:00     
(7/13): perl-Compress-Raw-Zlib-2.061-4.1.amzn1.x86_64.rpm          |  61 kB     00:00     
(8/13): perl-DBD-MySQL55-4.023-5.23.amzn1.x86_64.rpm               | 149 kB     00:00     
(9/13): perl-DBI-1.627-4.8.amzn1.x86_64.rpm                        | 855 kB     00:00     
(10/13): perl-Data-Dumper-2.145-3.5.amzn1.x86_64.rpm               |  49 kB     00:00     
(11/13): perl-IO-Compress-2.061-2.12.amzn1.noarch.rpm              | 298 kB     00:00     
(12/13): perl-Net-Daemon-0.48-5.5.amzn1.noarch.rpm                 |  58 kB     00:00     
(13/13): perl-PlRPC-0.2020-14.7.amzn1.noarch.rpm                   |  39 kB     00:00     
------------------------------------------------------------------------------------------
合計                                                      9.1 MB/s |  23 MB  00:00:02     
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  インストール中          : mysql55-libs-5.5.46-1.10.amzn1.x86_64                    1/13 
  インストール中          : mysql-config-5.5.46-1.10.amzn1.x86_64                    2/13 
  インストール中          : perl-Data-Dumper-2.145-3.5.amzn1.x86_64                  3/13 
  インストール中          : mysql55-5.5.46-1.10.amzn1.x86_64                         4/13 
  インストール中          : perl-Compress-Raw-Bzip2-2.061-3.11.amzn1.x86_64          5/13 
  インストール中          : perl-Net-Daemon-0.48-5.5.amzn1.noarch                    6/13 
  インストール中          : 1:perl-Compress-Raw-Zlib-2.061-4.1.amzn1.x86_64          7/13 
  インストール中          : perl-IO-Compress-2.061-2.12.amzn1.noarch                 8/13 
  インストール中          : perl-PlRPC-0.2020-14.7.amzn1.noarch                      9/13 
  インストール中          : perl-DBI-1.627-4.8.amzn1.x86_64                         10/13 
  インストール中          : perl-DBD-MySQL55-4.023-5.23.amzn1.x86_64                11/13 
  インストール中          : mysql55-server-5.5.46-1.10.amzn1.x86_64                 12/13 
  インストール中          : mysql-server-5.5-1.6.amzn1.noarch                       13/13 
  検証中                  : perl-DBI-1.627-4.8.amzn1.x86_64                          1/13 
  検証中                  : mysql55-libs-5.5.46-1.10.amzn1.x86_64                    2/13 
  検証中                  : perl-IO-Compress-2.061-2.12.amzn1.noarch                 3/13 
  検証中                  : perl-PlRPC-0.2020-14.7.amzn1.noarch                      4/13 
  検証中                  : mysql55-server-5.5.46-1.10.amzn1.x86_64                  5/13 
  検証中                  : 1:perl-Compress-Raw-Zlib-2.061-4.1.amzn1.x86_64          6/13 
  検証中                  : mysql55-5.5.46-1.10.amzn1.x86_64                         7/13 
  検証中                  : perl-DBD-MySQL55-4.023-5.23.amzn1.x86_64                 8/13 
  検証中                  : perl-Net-Daemon-0.48-5.5.amzn1.noarch                    9/13 
  検証中                  : perl-Compress-Raw-Bzip2-2.061-3.11.amzn1.x86_64         10/13 
  検証中                  : perl-Data-Dumper-2.145-3.5.amzn1.x86_64                 11/13 
  検証中                  : mysql-server-5.5-1.6.amzn1.noarch                       12/13 
  検証中                  : mysql-config-5.5.46-1.10.amzn1.x86_64                   13/13 

インストール:
  mysql-server.noarch 0:5.5-1.6.amzn1                                                     

依存性関連をインストールしました:
  mysql-config.x86_64 0:5.5.46-1.10.amzn1                                                 
  mysql55.x86_64 0:5.5.46-1.10.amzn1                                                      
  mysql55-libs.x86_64 0:5.5.46-1.10.amzn1                                                 
  mysql55-server.x86_64 0:5.5.46-1.10.amzn1                                               
  perl-Compress-Raw-Bzip2.x86_64 0:2.061-3.11.amzn1                                       
  perl-Compress-Raw-Zlib.x86_64 1:2.061-4.1.amzn1                                         
  perl-DBD-MySQL55.x86_64 0:4.023-5.23.amzn1                                              
  perl-DBI.x86_64 0:1.627-4.8.amzn1                                                       
  perl-Data-Dumper.x86_64 0:2.145-3.5.amzn1                                               
  perl-IO-Compress.noarch 0:2.061-2.12.amzn1                                              
  perl-Net-Daemon.noarch 0:0.48-5.5.amzn1                                                 
  perl-PlRPC.noarch 0:0.2020-14.7.amzn1                                                   

完了しました!


```


MySQLの自動起動→`sudo service mysqld start`
```
[ec2-user@ip-10-0-2-10 ~]$ sudo service mysqld start
Initializing MySQL database:  Installing MySQL system tables...
151228 11:42:24 [Note] /usr/libexec/mysql55/mysqld (mysqld 5.5.46) starting as process 23109 ...
OK
Filling help tables...
151228 11:42:24 [Note] /usr/libexec/mysql55/mysqld (mysqld 5.5.46) starting as process 23116 ...
OK

To start mysqld at boot time you have to copy
support-files/mysql.server to the right place for your system

PLEASE REMEMBER TO SET A PASSWORD FOR THE MySQL root USER !
To do so, start the server, then issue the following commands:

/usr/libexec/mysql55/mysqladmin -u root password 'new-password'
/usr/libexec/mysql55/mysqladmin -u root -h ip-10-0-2-10 password 'new-password'

Alternatively you can run:
/usr/libexec/mysql55/mysql_secure_installation

which will also give you the option of removing the test
databases and anonymous user created by default.  This is
strongly recommended for production servers.

See the manual for more instructions.

You can start the MySQL daemon with:
cd /usr ; /usr/libexec/mysql55/mysqld_safe &

You can test the MySQL daemon with mysql-test-run.pl
cd /usr/mysql-test ; perl mysql-test-run.pl

Please report any problems at http://bugs.mysql.com/

                                                           [  OK  ]
Starting mysqld:                                      

```

