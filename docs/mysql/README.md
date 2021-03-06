# RDBMSの基礎知識

ここではRDBMSの基礎を学びます。

## RDBMSの概要

RDBMSはRelational DataBase Management Systemの略です。

日本語にすると「関係データベース管理システム」となります。

リレーショナルデータベースを管理する為の専用ソフトウエアを指します。

リレーショナルデータベースは1件のデータを表形式で管理します。

ざっくり言うとExcelの表をイメージしてもらうと分かりやすいです。

下記は商品テーブルのイメージです。

| 商品ID | 商品名 | 価格 |カテゴリ|
|:---|:---:|---:|---:|
|1 |コーラ |100 |10|
|2 |USB |2000 |20|
|3 |傘 |500 |30|
|4 |お茶 |100 |10|

`商品ID` や `商品名` 等の項目を「カラム（列）」、1件のデータを「レコード（行）」、レコードの集まりをテーブル（表）と呼びます。

リレーショナルデータベースの名の通り復数のテーブル（表）を結合（リレーション）して利用する事が出来ます。

下記の画像のようなイメージです。

![rdbimages](https://user-images.githubusercontent.com/11032365/37476473-9bdfb96e-28b8-11e8-91f2-e9e504f719be.png)

## RDBMSの特徴

テーブル形式でデータの出し入れが出来る以外にも様々な特徴があります。

例えば以下のような特徴です。

- 不正なデータ記録を防ぐ仕組みがありデータ不整合を起こしにくい（もちろん正しく利用すればという前提条件はつきます）
- 権限を持たない不正なユーザー操作を防ぐ仕組みがある
- トランザクション処理（後で解説します）が利用出来るのでデータ不整合が起きにくい

## RDBMSの種類

以下のような製品があります。

- [Oracle](https://ja.wikipedia.org/wiki/Oracle_Database)
- [Microsoft SQL Server](https://ja.wikipedia.org/wiki/Microsoft_SQL_Server)
- [PostgreSQL](https://ja.wikipedia.org/wiki/PostgreSQL)
- [MySQL](https://ja.wikipedia.org/wiki/MySQL)

月額数千万円する値段が高い物からオープンソースで無料提供されている物まで様々です。

本カリキュラムではWeb開発でもっとも多く利用されている [MySQL](https://ja.wikipedia.org/wiki/MySQL) を利用します。

## MySQL公式サイト

世界で最も普及しているオープンソースのRDBMSです。

[こちら](https://dev.mysql.com/) が公式サイトになります。

## MySQLのインストール

`yum` を使ってインストールするのが最も簡単です。

https://dev.mysql.com/downloads/repo/yum/ より公式提供されているyumリポジトリを利用しましょう。

※ なお、本カリキュラムでは [Amazon Linux AMI 2017.09](https://aws.amazon.com/jp/blogs/news/now-available-amazon-linux-ami-2017-09/) を利用している前提で話を進めさせて頂きます。

他のLinuxディストリビューションを利用している場合はインストール方法が多少変わってきます。

インストールするMySQLのバージョンは5.7系です。

以下のコマンドを実行します。

```bash
sudo yum install https://dev.mysql.com/get/mysql57-community-release-el6-11.noarch.rpm
```

これで公式のyumリポジトリがインストールされました。

続いてMySQLの本体をインストールします。

```bash
sudo yum install mysql-community-client mysql-community-common mysql-community-devel mysql-community-libs mysql-community-server
```

多少時間がかかりますがインストールが完了するまで待ちましょう。

エラーが出なければインストールが完了しています。

## MySQLの起動

以下のコマンドで起動を行います。

```bash
sudo service mysqld start
```

以下のように表示されれば起動は成功です。

```
Initializing MySQL database:                               [  OK  ]
Starting mysqld:                                           [  OK  ]
```

## MySQLの自動起動設定

OSの再起動を行ってもMySQLサーバが自動起動する設定を行っておきましょう。

`sudo su` でrootユーザーになり、以下のコマンドを実行しましょう。

```bash
chkconfig mysqld on
```

## MySQLサーバにログインを行う

MySQLサーバにログインを行って操作を行ってみましょう。

初期状態だと `root` ユーザーのパスワードが自動的に設定されたものになっていますので以下のコマンドで確認を行います。

```bash
sudo cat /var/log/mysqld.log | grep 'temporary password'
```

実行すると下記のように出力されます。

```
2018-03-16T15:20:39.532915Z 1 [Note] A temporary password is generated for root@localhost: Qzon6<:YgfeX
```

この場合 `Qzon6<:YgfeX` がrootパスワードになりますのでどこかにメモしておきます。

以下のコマンドでログインを行います。

```bash
mysql -u root -h localhost -p
```

`-u` はユーザーを表すオプションです。最初はrootしかユーザーがいないのでrootを入力します。

`-h` はMySQLサーバが存在するIPアドレスやホスト名を指定します。

今回のケースだとログインしているLinuxサーバ上にMySQLサーバが存在するので `localhost` を指定します。

`-p` はパスワードを入力するという意味です。

先程メモしたrootパスワードを入力しましょう。

下記のように表示されればログイン成功です。

```
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 5.7.21

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```

## セキュリティ設定を行う

ログインは出来ましたが、今の状態だと正しいコマンドを打っても以下のエラーが表示されてしまいます。

```
ERROR 1820 (HY000): You must reset your password using ALTER USER statement before executing this statement.
```

これはセキュリティ上の理由から初期設定を行わないとMySQLサーバが利用出来ない状態になっている為です。

初期設定を行う為に一度MySQLサーバからログアウトを行います。

コンソールに `exit` を入力して下さい。

```
mysql> exit
Bye
[root@localhost vagrant]#
```

これでMySQLサーバからログアウトが出来ます。

ここから初回セキュリティ設定を行っていきます。

以下のコマンドを実行しましょう。（rootユーザーで実行する必要があります）

```
mysql_secure_installation
```

ここからは対話式インターフェースになりますので画面の指示に従い情報を入力します。

`Enter password for user root:` と聞かれるのでrootパスワードを入力します。

新しいrootパスワードの入力を求められるので入力します。

```root
The existing password for the user account root has expired. Please set a new password.

New password:
```

ここで注意点が1点あります。

パスワードは「英数字、記号、混在の8文字以上」である必要があります。

条件に合うパスワードを入力しましょう。

例えば `(MySQLRootPassword1234)` とかです。

`Re-enter new password:` と聞かれるので再度同じパスワードを入力します。

`Estimated strength of the password: 100` みたいなメッセージが表示されます。

パスワードの推定強度を数値で表しています。

`Do you wish to continue with the password provided?` と聞かれるので `y` を入力しEnterを押します。

`Remove anonymous users?` と聞かれます。

MySQLには匿名ユーザーという特殊なユーザーが存在しており、匿名ユーザーはパスワードなしでMySQLにログインする事が可能です。

セキュリティ的に大きな問題があるので `y` Enterで匿名ユーザーを削除します。

`Disallow root login remotely?` と聞かれます。

意味は「他のサーバからこのMySQLサーバへrootユーザーでログインする事を拒否しますか」です。

rootユーザーはMySQLの操作が全て実行可能なので他サーバからログインする事を許可すべきではありません。

よって `y` Enterを押します。

`Remove test database and access to it?` と聞かれます。

「テスト用に作られたデータベースを削除してアクセスしますか」なので `y` Enterを押します。

`Reload privilege tables now?` と聞かれます。

これは「今までの設定を今すぐ反映しますか」という意味になりますので `y` Enterを押します。

下記のように表示されれば初期設定は完了です。

```
Success.

All done!
```

再度 `mysql -u root -p` を行い先程設定したパスワードでログインが出来る事を確認しましょう。

rootパスワードを忘れると結構面倒な作業をしなければならないので忘れないようにしましょう。

一応忘れた時用の参考記事を載せておきます。

- [MySQL の root パスワード忘れた時](https://qiita.com/y1row/items/994ecf8b478b7aac4c7d)
- [MySQL5.7でrootパスワード初期化変更メモ / 2017年10月](https://qiita.com/ononoy/items/7732a2e97b3901eb9d57)

## MySQLサーバの設定

MySQLを使う上で事実上必須になっている設定を行っていきます。

### 文字コードの設定

MySQLサーバにログインを行い `SHOW VARIABLES LIKE 'char%';` というSQLを実行してみて下さい。

初期状態では下記のような状態になっている事が確認出来ます。

```
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | latin1                     |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | latin1                     |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)
```

`latin1` という文字コードはMySQLのデフォルト設定なのですが、これだと日本語を正しく扱う事が出来ません。

MySQL5.7系で日本語を含めたマルチバイト文字を正しく扱えるのは `utf8mb4` だけです。

`utf8mb4` は 👌や🐱等の4バイト文字も扱う事が可能です。

最も多くの言語にも対応しているので文字コードは `utf8mb4` の一択で問題ありません。

他にもいくつか変更すべき設定項目があるので、設定ファイルを修正し設定を変更します。

rootユーザーで `/etc/my.cnf` というファイルを `vim` 等で開いて下さい。

初期状態は下記のようになっています。

```
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/5.7/en/server-configuration-defaults.html

[mysqld]
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```

以下の記述を追記して下さい。

```
character-set-server=utf8mb4
collation-server=utf8mb4_bin
skip-character-set-client-handshake
default-storage-engine=InnoDB
innodb_file_per_table=1
innodb_large_prefix=1
innodb_file_format=Barracuda
innodb_default_row_format=DYNAMIC
default_password_lifetime = 0
slow_query_log = 1
long_query_time = 0.1
slow_query_log_file = /var/log/mysql-slow-query.log

[mysql]
auto-rehash
default-character-set = utf8mb4

[mysqldump]
default-character-set = utf8mb4
```

### 文字コードに関する設定

追記した内容のうち文字コードに関する設定は以下になります。

```
character-set-server=utf8mb4
collation-server=utf8mb4_bin
skip-character-set-client-handshake

[mysql]
default-character-set = utf8mb4

[mysqldump]
default-character-set = utf8mb4
```

`[mysql]` 、`[mysqldump]` セクションに記載してあるのはそのまま文字コードを `utf8mb4` に設定するという意味です。

- `skip-character-set-client-handshake`

MySQLのクライアント側（接続側）が設定する文字コード設定を無効化する事が出来ます。

常にMySQLサーバ側の設定（ここでは `utf8mb4`）が設定されて欲しいのでこの設定を有効にします。

https://dev.mysql.com/doc/refman/5.6/ja/faqs-cjk.html

- `character-set-server`

MySQLサーバ側の文字コード設定です。

- `collation-server`

文字の照合規則・照合順序（並べ替えルールみたいなもの）を表す項目です。

MySQLでマルチバイト文字を扱うには考慮しておくべき問題があります。

- 絵文字の 🍣と🍺が同様のデータとして扱われてしまう「🍣🍺問題」
- カタカナの「ハ」と「パ」が同じものとして扱われてしまう「ハハパパ問題」

現状、この2つを解決する設定は `collation-server=utf8mb4_bin` のみです。

これを設定する事によりデフォルト状態とは少し違う挙動になります。

MySQLは通常 `a` と `A` を同じ物として認識する仕様です。

`utf8mb4_bin` を設定すると明確に別の文字として扱われるようになりますので注意が必要です。

普通にプログラムを組んでいればこの点が問題になる事は少ないハズです。

MySQLの文字コードに関しては [こちらのスライド](https://www.slideshare.net/tmtm/mysql-62004569) が参考になります。

### ストレージエンジンに関する設定

ストレージエンジンとはRDBMSの核になる部分で主にデータの格納形式等を決定している部分です。

詳しくは下記の記事を参考にして下さい。

- [徹底比較!! MySQLエンジン](https://thinkit.co.jp/free/article/0608/1/1/)
- [公式](https://dev.mysql.com/doc/refman/5.6/ja/storage-engines.html) を

結論から言うとよほど特殊な状況化を除き `InnoDB` の一択で良いです。

理由はRDBMSで重要な機能であるトランザクションを扱えるのはこのエンジンだけだからです。

ちなみにテーブルを作成する時もストレージエンジンを設定する事は可能です。

万が一特定のテーブルだけ別のストレージエンジンを利用したい場合はテーブル作成時に指定する方法を取れば良いでしょう。

### innodb_file_per_table

ストレージエンジン `InnoDB` に関する設定の1つです。

`innodb_file_per_table` はテーブルごとに専用のテーブルスペースを作成する為の設定です。

これを設定しておかないと古いデータを消してディスク容量を確保したい時にかなり面倒です。

詳しくは下記の記事を参考にして下さい。

- [【AWS】RDS for MySQLで共有テーブルスペースに構成変更する際の所要時間を実測してみた](https://dev.classmethod.jp/server-side/db/rds-mysql-innodb_file_per_table/)
- [【MySQL】肥大化したInnoDBテーブルを圧縮機能で縮小する方法！](https://engineers.weddingpark.co.jp/?p=622)

### innodb_file_format

ストレージエンジン `InnoDB` に関する設定の1つです。

圧縮機能が利用可能になる `Barracuda` へ変更しておきます。

### innodb_default_row_format

ストレージエンジン `InnoDB` に関する設定の1つです。

ここでは `DYNAMIC` を指定しています。

ちなみに圧縮機能が有効なのは `COMPRESSED` です。

圧縮機能を有効にすれば確かにテーブルのデータ容量を節約する事は出来ますが、パフォーマンスが低下するという弱点があります。

その為、普段は `DYNAMIC`を指定しておくのが無難だと私は考えます。

テーブル作成時に `ROW_FORMAT=COMPRESSED` を指定すれば圧縮機能を有効に出来ます。

詳しくは [こちらの記事](https://engineers.weddingpark.co.jp/?p=622) を参考にして下さい。

### default_password_lifetime

デフォルト設定だと365日でパスワードを強制変更するようにMySQLからエラーが出るようになっています。

運用中にこのエラーが出るとサービスが止まってしまうのでこの設定を明示的に無効化しています。

※ MySQL 5.7.11からはデフォルトで0に設定が変更されたようなのでこの設定は不要ですが念の為記載しておきます。

### auto-rehash

LinuxのようにTABキーでテーブル名やSQLの補完が出来るようになります。

よって設定しておきましょう。

### slow_query_log

スロークエリを出力する設定です。

名前の通り時間がかかっているSQLをログとして出力する為の設定です。

### long_query_time

どの程度時間がかかったらスロークエリと見なすかの設定です。

個人的には厳し目の設定にしておくのが良いと考えます。

ここでは0.1秒を設定しています。

遅いSQLはそれだけサーバのリソースを消費し、ユーザー体験にも悪影響を与えるので厳し目に設定を行っておき常に監視しておくのが良いでしょう。

### slow_query_log_file

スロークエリが出力されるファイルパスです。

ここでは `/var/log/mysql-slow-query.log` を設定しています。

スロークエリは `mysqldumpslow` というコマンドで集計する事が可能です。

詳しくは [こちらの記事](https://webmake.info/mysql%E3%81%AE%E3%82%B9%E3%83%AD%E3%83%BC%E3%82%AF%E3%82%A8%E3%83%AA%E3%83%AD%E3%82%B0%E3%82%92mysqldumpslow%E3%81%A7%E5%88%86%E6%9E%90%E3%81%99%E3%82%8B/) を参考にして下さい。

## 設定を反映する

説明が長くなってしまいましたが、以上がMySQLで最低限設定しておくべき項目です。

設定内容を反映した `/etc/my.cnf` を載せておきます。

```
# For advice on how to change settings please see
# http://dev.mysql.com/doc/refman/5.7/en/server-configuration-defaults.html

[mysqld]
#
# Remove leading # and set to the amount of RAM for the most important data
# cache in MySQL. Start at 70% of total RAM for dedicated server, else 10%.
# innodb_buffer_pool_size = 128M
#
# Remove leading # to turn on a very important data integrity option: logging
# changes to the binary log between backups.
# log_bin
#
# Remove leading # to set options mainly useful for reporting servers.
# The server defaults are faster for transactions and fast SELECTs.
# Adjust sizes as needed, experiment to find the optimal values.
# join_buffer_size = 128M
# sort_buffer_size = 2M
# read_rnd_buffer_size = 2M
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock

# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid

character-set-server=utf8mb4
collation-server=utf8mb4_bin
skip-character-set-client-handshake
default-storage-engine=InnoDB
innodb_file_per_table=1
innodb_large_prefix=1
innodb_file_format=Barracuda
innodb_default_row_format=DYNAMIC
default_password_lifetime = 0
slow_query_log = 1
long_query_time = 0.1
slow_query_log_file = /var/log/mysql-slow-query.log

[mysql]
auto-rehash
default-character-set = utf8mb4

[mysqldump]
default-character-set = utf8mb4
```

スロークエリログの設定に関しては `slow_query_log_file` で指定したファイルを作成し書き込み権限を与えておく必要があります。

以下のコマンドを実行しておきましょう。

```bash
sudo touch /var/log/mysql-slow-query.log
sudo chown mysql:mysql /var/log/mysql-slow-query.log

```

準備が終わったら、MySQLサーバの再起動を行いましょう。

```bash
sudo service mysqld restart
```

続いてMySQLサーバにログインを行い、設定が反映されているか確認を行います。

`SHOW VARIABLES LIKE 'char%';` で文字コードの確認を行います。

以下のように表示されていればOKです。

```
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8mb4                    |
| character_set_connection | utf8mb4                    |
| character_set_database   | utf8mb4                    |
| character_set_filesystem | binary                     |
| character_set_results    | utf8mb4                    |
| character_set_server     | utf8mb4                    |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.01 sec)
```

※ `character_set_system` と `character_set_system` に関しては `utf8mb4` になっていなくてもOKです。

続いて `SHOW VARIABLES LIKE 'slow%';` でスロークエリの設定を確認します。

以下のように `slow_query_log` が `ON` になっていればOKです。

```
+---------------------+-------------------------------+
| Variable_name       | Value                         |
+---------------------+-------------------------------+
| slow_launch_time    | 2                             |
| slow_query_log      | ON                            |
| slow_query_log_file | /var/log/mysql-slow-query.log |
+---------------------+-------------------------------+
3 rows in set (0.00 sec)
```

`SHOW VARIABLES LIKE 'long_query%';` でスロークエリの秒数設定を確認します。

以下のように表示されていればOKです。

```
+-----------------+----------+
| Variable_name   | Value    |
+-----------------+----------+
| long_query_time | 0.100000 |
+-----------------+----------+
1 row in set (0.00 sec)
```

ここで紹介した物以外にも様々な設定があります。

近年では [Amazon Aurora](https://aws.amazon.com/jp/rds/aurora/) のような便利なサービスが出てきたので昔ほど設定内容に詳しい必要はなくなりました。

しかしSQLの性能はアプリケーションの応答速度やサーバリソースに大きな影響を与えるので知っておいて損はない知識です。

興味がある方はより詳しく調べてみると良いでしょう。

特に `innodb_buffer_pool` は重要な設定項目の1つです。

（参考）[MySQL 5.7 インストールと設定メモ](https://blog.apar.jp/linux/6769/)

## データベースとユーザーの作成

次にデータベースとユーザーを作成します。

MySQLサーバにrootユーザーでログインを行います。

最初にテーブルを格納する為のデータベースを作成します。

データベース名は `ojt_db` とします。

MySQLを操作する為には [SQL](https://ja.wikipedia.org/wiki/SQL) という言語を利用します。

SQLはStructured English Query Languageの略でリレーショナルデータベースと対話する為の言語です。

データベースを作るには以下のSQLを実行します。

```sql
CREATE DATABASE ojt_db;
```

以下のように表示されれば成功です。

```
mysql> create database ojt_db;
Query OK, 1 row affected (0.00 sec)
```

SQLは小文字でも大文字でも動きますが、SQLで使われている文法と利用者が決めたデータベース名、テーブル名等と区別する為、大文字で記載する事が多いです。

同様の理由からテーブル名やデータベース名には小文字と `_` のみを使って命名するケースが多いです。

次にユーザーの作成を行います。

以下のSQLを実行します。

```sql
GRANT CREATE, SELECT, UPDATE, INSERT, DELETE ON ojt_db.* TO `ojt_user`@`localhost` IDENTIFIED BY '(YourPassword999)';
```

意味を簡単に説明すると以下のようになります。

- ユーザー名： ojt_user
- パスワード： (YourPassword999)
- 操作権限があるデータベースとテーブル： ojt_dbに存在する全てのテーブル
- 実行可能なSQL： CREATE、SELECT、UPDATE、INSERT、DELETE
- 有効な接続元： localhost

`CREATE、SELECT` 等は実行出来るSQLの種類を表します。

例えば `CREATE` はテーブルの新規作成、`SELECT` はテーブルからデータを閲覧出来る権限です。

[こちら](https://dev.mysql.com/doc/refman/5.7/en/privileges-provided.html) を参照して下さい。

`ALL` とすると全てのSQLが実行可能になりますが基本的には必要な権限だけを追加するのがセキュリティ的に推奨されます。

ユーザー名は `ユーザー名@接続元ホスト` でユニークになります。

その為、ホスト名が異なれば同じユーザー名を使う事が出来ます。

例えば下記のようなSQLを実行すると、IPアドレスが `192.168.` から始まる全てのサーバから `ojt_db` にアクセス出来る `ojt_user` が作成されます。

```sql
GRANT CREATE, SELECT, UPDATE, INSERT, DELETE ON ojt_db.* TO `ojt_user`@`192.168.%` IDENTIFIED BY '(YourPassword999)';
```

一度MySQLサーバからログアウトして今度は作成した `ojt_user` でログイン出来るか確認しましょう。

`mysql -u ojt_user -p` でパスワードを入力します。

ログインが出来たら `SHOW DATABASES;` というSQLを実行します。

```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| ojt_db             |
+--------------------+
2 rows in set (0.00 sec)
```

`SHOW DATABASES;` は存在するデータベースの一覧を表示させる命令ですが、この時に接続権限がないデータベースは表示されません。

このようにMySQLではユーザーに適切な権限を与える事で不正な操作を予め防ぐ事が出来ます。

## テーブルの作成

ますは簡単なテーブルを作成してみましょう。

この資料の最初で例に上げた商品テーブルを作成するSQLは下記のようになります。

mysqlサーバにログインを行い以下のSQLを実行しましょう。

`USE ojt_db;` を実行してデータベースの選択をするのを忘れないようにしましょう。

```mysql
CREATE TABLE `products_categories` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `name` varchar(255) COLLATE utf8mb4_bin NOT NULL,
  `lock_version` int(10) unsigned NOT NULL DEFAULT '0',
  `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin ROW_FORMAT=DYNAMIC;
```

```mysql
CREATE TABLE `products` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `category_id` int(10) unsigned NOT NULL,
  `name` varchar(255) COLLATE utf8mb4_bin NOT NULL,
  `price` int(10) unsigned NOT NULL DEFAULT '0',
  `lock_version` int(10) unsigned NOT NULL DEFAULT '0',
  `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  CONSTRAINT `fk_products_01` FOREIGN KEY (`category_id`) REFERENCES `products_categories` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin ROW_FORMAT=DYNAMIC;
```

`SHOW TABLES;` を実行してみましょう。

テーブルが作成されている事が確認出来ます。

```sql
mysql> SHOW TABLES;
+---------------------+
| Tables_in_ojt_db    |
+---------------------+
| products            |
| products_categories |
+---------------------+
2 rows in set (0.00 sec)
```

`SHOW TABLES;` で確認出来るのはテーブルの一覧だけです。

テーブルの構造を再度確認したい場合は `SHOW CREATE TABLE [テーブル名]` を実行します。

```sql
mysql> SHOW CREATE TABLE products;
+----------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table    | Create Table                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
+----------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| products | CREATE TABLE `products` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `category_id` int(10) unsigned NOT NULL,
  `name` varchar(255) COLLATE utf8mb4_bin NOT NULL,
  `price` int(10) unsigned NOT NULL DEFAULT '0',
  `lock_version` int(10) unsigned NOT NULL DEFAULT '0',
  `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  KEY `fk_products_01` (`category_id`),
  CONSTRAINT `fk_products_01` FOREIGN KEY (`category_id`) REFERENCES `products_categories` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin ROW_FORMAT=DYNAMIC |
+----------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```

## データの登録

次にデータの登録を行ってみましょう。

以下のSQLを実行して下さい。

```mysql
INSERT INTO products_categories (`name`) VALUES('ドリンク');
INSERT INTO products_categories (`name`) VALUES('お菓子');
```

```mysql
INSERT INTO products (`category_id`, `name`, `price`) VALUES(1, 'コーラ', 150);
INSERT INTO products (`category_id`, `name`, `price`) VALUES(2, 'ポテチ', 100);
```

`Query OK, 1 row affected (0.00 sec)` と表示されればデータ登録に成功しています。

これがデータ登録を行う為のINSERT文の基本になります。

書式をまとめると下記のようになります。

プログラミング言語と比べると簡単に感じるのではないでしょうか。

```sql
INSERT INTO [テーブル名] (`カラム名`, `カラム名`, ...) VALUES ([値], [値], ...);
```

### データ登録の制限

RDBMSはテーブル構造に様々な制約を持たせる事が可能です。

制約がある事で不整合データの混入を防止しています。

#### プライマリーキー違反

テーブルには必ずプライマリーキーと呼ばれるカラムが必要です。

正確にはプライマリーキーがなくてもテーブルを作る事は可能なのですが、データ整合性を保つ事が困難になる為、事実上必須と言えます。

`CREATE TABLE` を実行した時の以下の記述に注目して下さい。

`id int(10) unsigned NOT NULL AUTO_INCREMENT`

これはAUTO_INCREMENTと呼ばれる形でDBが勝手に重複しない連番を作ってくれる形式です。

先程の `INSERT INTO` 実行時にidカラムの指定をしなくてもエラーにならなかったのは、AUTO_INCREMENT機能が働いていた為になります。

プライマリーキーはテーブル内で必ずユニーク（重複しない）事が保証されます。

つまり下記のSQLを実行するとエラーになります。

```sql
mysql> INSERT INTO products (`id`, `category_id`, `name`, `price`) VALUES(1, 1, 'ファンタ', 150);
ERROR 1062 (23000): Duplicate entry '1' for key 'PRIMARY'
```

#### NOT NULL制約違反

以下の記述に注目しましょう。

`name varchar(255) COLLATE utf8mb4_bin NOT NULL`

`NOT NULL` というのはNULLデータの追加を認めないという意味になります。

なので、以下のSQLはエラーになります。

```sql
mysql> INSERT INTO products (`category_id`, `name`, `price`) VALUES(1, null, 150);
ERROR 1048 (23000): Column 'name' cannot be null
```

#### 外部キー制約違反

`products` テーブルの下記に注目しましょう。

`CONSTRAINT fk_products_01 FOREIGN KEY (category_id) REFERENCES products_categories (id)`

このテーブル構造では `products` テーブルに `products_categories` のidを一緒に保存する事で商品カテゴリが分かるようになっています。

その為、`products.category_id` に存在しない `category_id` を入れられると不整合データが発生してしまいます。

それを防ぐ為の仕組みが外部キー制約（FOREIGN KEY）です。

つまり以下のSQLはエラーになります。

```sql
mysql> INSERT INTO products (`category_id`, `name`, `price`) VALUES(100, 'ファンタ', 150);
ERROR 1452 (23000): Cannot add or update a child row: a foreign key constraint fails (`ojt_db`.`products`, CONSTRAINT `fk_products_01` FOREIGN KEY (`category_id`) REFERENCES `products_categories` (`id`))
```

他にも例えば数値型のカラムに文字列を入れると起こるTypeError等があります。

このように様々な制約があるおかげで不整合データの発生を未然に防ぐ事が可能です。

## データを取得する

次にデータを取得する方法を見ていきます。

データ取得は `SELECT` を利用します。

以下のSQLを実行してみましょう。

```sql
mysql> SELECT * FROM products;
+----+-------------+-----------+-------+--------------+---------------------+---------------------+
| id | category_id | name      | price | lock_version | created_at          | updated_at          |
+----+-------------+-----------+-------+--------------+---------------------+---------------------+
|  1 |           1 | コーラ    |   150 |            0 | 2018-03-22 14:08:08 | 2018-03-22 14:08:08 |
|  2 |           2 | ポテチ    |   100 |            0 | 2018-03-22 14:08:08 | 2018-03-22 14:08:08 |
+----+-------------+-----------+-------+--------------+---------------------+---------------------+
2 rows in set (0.00 sec)
```

これは `products` テーブルのデータを全件取得する命令になります。

`SELECT` の後にはカラム名が入ります。 `*` は全てのカラムという意味になります。

以下のように見たいカラムだけを指定すれば見たいカラムだけを抽出出来ます。

```sql
mysql> SELECT id, name, price FROM products;
+----+-----------+-------+
| id | name      | price |
+----+-----------+-------+
|  1 | コーラ    |   150 |
|  2 | ポテチ    |   100 |
+----+-----------+-------+
2 rows in set (0.00 sec)
```

`FROM` の後にはテーブル名が入ります。

今のようにデータ量が少ない場合は `SELECT * FROM products;` で全件取得で問題ないですが、現実的にはデータ量はどんどん増えていきます。

その度に全件取得を行っていてはデータベースにも負担がかかるので通常は条件を指定してデータを抽出します。

そういう時は `WHERE` を使います。

```sql
mysql> SELECT * FROM products WHERE id = 1;
+----+-------------+-----------+-------+--------------+---------------------+---------------------+
| id | category_id | name      | price | lock_version | created_at          | updated_at          |
+----+-------------+-----------+-------+--------------+---------------------+---------------------+
|  1 |           1 | コーラ    |   150 |            0 | 2018-03-22 14:08:08 | 2018-03-22 14:08:08 |
+----+-------------+-----------+-------+--------------+---------------------+---------------------+
1 row in set (0.00 sec)
```

この例ではidが1番のデータを取得してという意味です。

`products` テーブルには商品カテゴリを示す `category_id` が入っています。

しかし人間にも分かりやすい、カテゴリ名は `products_categories` テーブルに入っています。

次は2つのテーブルを結合する方法を見ていきましょう。

今までより少し長いですが、以下のSQLを実行してみましょう。

```mysql
SELECT
  products.id,
  products.category_id,
  products.name,
  products.price,
  products_categories.name AS category_name 
FROM
  products 
LEFT JOIN
  products_categories
ON
  products.category_id = products_categories.id;
```

以下のように別テーブルにあるカテゴリ名を同じ表として扱う事が出来ます。

```sql
mysql> SELECT
    ->   products.id,
    ->   products.category_id,
    ->   products.name,
    ->   products.price,
    ->   products_categories.name AS category_name
    -> FROM
    ->   products
    -> LEFT JOIN
    ->   products_categories
    -> ON
    ->   products.category_id = products_categories.id;
+----+-------------+-----------+-------+---------------+
| id | category_id | name      | price | category_name |
+----+-------------+-----------+-------+---------------+
|  1 |           1 | コーラ    |   150 | ドリンク      |
|  2 |           2 | ポテチ    |   100 | お菓子        |
+----+-------------+-----------+-------+---------------+
2 rows in set (0.00 sec)
```

`LEFT JOIN` という記述がテーブルとテーブルを繋ぐ為の構文になります。

ここで使った `LEFT JOIN` は外部結合と呼ばれる方法です。

他にも `INNER JOIN` や `OUTER JOIN` もあります。

ここでは詳しく解説しませんので参考になりそうな記事を載せておきます。

- [13.2.9.2 JOIN 構文](https://dev.mysql.com/doc/refman/5.6/ja/join.html)
- [これでわかった!? LEFT / RIGHT JOIN.](https://qiita.com/zaburo/items/548b3c40fee68cd1e3b7)

最初のうちは `LEFT JOIN` と `INNER JOIN` が理解出来れば、だいたいの状況に対応出来ます。

JOINやWHEREを使い様々な条件でデータを抽出出来るようになりましょう。

## データを更新する

次にデータを更新する方法を見ていきましょう。

データ更新は `UPDATE` を利用します。

以下の例は商品ID2番のポテチの価格を150に修正する例です。

```mysql
UPDATE products SET price = 150 WHERE id = 2;
```

`SELECT` で確認すると `price` が更新されている事が確認出来ます。

```sql
mysql> SELECT * FROM products WHERE id = 2;
+----+-------------+-----------+-------+--------------+---------------------+---------------------+
| id | category_id | name      | price | lock_version | created_at          | updated_at          |
+----+-------------+-----------+-------+--------------+---------------------+---------------------+
|  2 |           2 | ポテチ    |   150 |            0 | 2018-03-22 14:08:08 | 2018-03-22 15:31:13 |
+----+-------------+-----------+-------+--------------+---------------------+---------------------+
1 row in set (0.00 sec)
```

`UPDATE` の場合 `WHERE` を付け忘れると全てのデータを更新してしまうので注意が必要です。

その為、必ず最初に `SELECT WHERE` で目的のデータを抽出出来ている事を確認してから実行しましょう。

（参考）[13.2.11 UPDATE 構文](https://dev.mysql.com/doc/refman/5.6/ja/update.html)

## データを削除する

データを削除する方法です。

カラムの指定がないので簡単ですが、こちらも `WHERE` を付け忘れると全データ削除という取り返しがつかない事になります。

※ 戻す方法がない訳ではありませんが、それなりに時間はかかります。

（参考）[13.2.2 DELETE 構文](https://dev.mysql.com/doc/refman/5.6/ja/delete.html)

## バックアップとリカバリ

データベースはバックアップが非常に重要になります。

データを誤って消してしまった場合はこのバックアップが頼りになります。

最も簡単なバックアップ方法は `mysqldump` を使う方法です。

MySQLサーバからログアウトした状態で以下のコマンドを実行して下さい。

※ MySQLのrootパスワードを求められるので入力して下さい。

```bash
mysqldump --add-drop-table -q -c -h localhost -u root -p ojt_db > ojt_db.sql
```

そうすると `ojt_db.sql` というファイルが生成されている事が確認出来ます。

中身を確認すると今まで作成したテーブルやデータの中身が全て記録されている事が確認出来ます。

次にバックアップからリカバリする方法を試してみましょう。

mysqlサーバにrootでログインして以下のSQLを実行して下さい。

`DROP DATABASE ojt_db;`

これはデータベースを削除するSQLです。

続いて再度 `CREATE DATABASE ojt_db;` を実行します。

次にMySQLサーバからログアウトした状態で以下のコマンドを実行して下さい。

※ MySQLのrootパスワードを求められるので入力して下さい。

```bash
mysql -u root -h localhost -p ojt_db < ojt_db.sql
```

再度MySQLサーバにログインすると先程消したデータやテーブルが復元されている事が確認出来ます。

ここで紹介したのは手動でバックアップを取る一番簡単な方法です。

（参考）[7.4.1 mysqldump による SQL フォーマットでのデータのダンプ](https://dev.mysql.com/doc/refman/5.6/ja/mysqldump-sql-format.html)

他にもバックアップの方法は様々な物があります。

（参考）[第 7 章 バックアップとリカバリ](https://dev.mysql.com/doc/refman/5.6/ja/backup-and-recovery.html)

最近では [Amazon RDS](https://aws.amazon.com/jp/rds/) 等のバックアップを自動でやってくれる便利なサービスもあります。

バックアップは運用を行っていく上で非常に大切です。

少々コストがかかりますが本番環境での運用時にはこのようなサービスを利用する事をオススメします。

## トランザクション

MySQLにおけるトランザクションは簡単に言うと、ある連続した処理を1つのかたまりとして実行出来る機能です。

銀行口座を模した以下のテーブルで説明します。

まず下記のテーブルを作って下さい。

```mysql
CREATE TABLE `bank_accounts` (
  `id` int(10) unsigned NOT NULL AUTO_INCREMENT,
  `number` varchar(64) COLLATE utf8mb4_bin NOT NULL,
  `amount` int(10) NOT NULL,
  `lock_version` int(10) unsigned NOT NULL DEFAULT '0',
  `created_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP,
  `updated_at` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  PRIMARY KEY (`id`),
  KEY `idx_bank_accounts_01` (`number`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_bin ROW_FORMAT=DYNAMIC;
```

次に以下のデータを登録して下さい。

```mysql
INSERT INTO bank_accounts (`number`, `amount`) VALUES('0123456', 10000);
INSERT INTO bank_accounts (`number`, `amount`) VALUES('0123456', 10000);
INSERT INTO bank_accounts (`number`, `amount`) VALUES('0999999', 5000);
INSERT INTO bank_accounts (`number`, `amount`) VALUES('0999999', 2000);
```

テーブルの中身は以下の形になっています。

```sql
mysql> SELECT * FROM bank_accounts;
+----+---------+--------+--------------+---------------------+---------------------+
| id | number  | amount | lock_version | created_at          | updated_at          |
+----+---------+--------+--------------+---------------------+---------------------+
|  1 | 0123456 |  10000 |            0 | 2018-03-24 09:12:34 | 2018-03-24 09:12:34 |
|  2 | 0123456 |  10000 |            0 | 2018-03-24 09:12:34 | 2018-03-24 09:12:34 |
|  3 | 0999999 |   5000 |            0 | 2018-03-24 09:12:34 | 2018-03-24 09:12:34 |
|  4 | 0999999 |   2000 |            0 | 2018-03-24 09:12:35 | 2018-03-24 09:12:35 |
+----+---------+--------+--------------+---------------------+---------------------+
4 rows in set (0.00 sec)
```

口座番号 `0123456` と `0999999` の2人が存在しており、それぞれの預金残高は以下の通りになります。

- `0123456` は10000円
- `0999999` は7000円

これをSQLで求めると下記のようになります。

- 口座番号 `0123456` の預金残高

```sql
mysql> SELECT SUM(amount) FROM bank_accounts WHERE number = '0123456';
+-------------+
| SUM(amount) |
+-------------+
|       20000 |
+-------------+
1 row in set (0.00 sec)
```

- 口座番号 `0999999` の預金残高

```sql
mysql> SELECT SUM(amount) FROM bank_accounts WHERE number = '0999999';
+-------------+
| SUM(amount) |
+-------------+
|        7000 |
+-------------+
1 row in set (0.00 sec)
```

では `0123456` の口座から `0999999` の口座に5000円送金する処理を考えてみましょう。

このテーブル構造の場合以下の処理を実行する必要があります。

1. 口座番号 `0123456` に対して `-5000` のレコードを登録する
1. 口座番号 `0999999` に対して `5000` のレコードを登録する

SQLで書くと以下のようになります。

```sql
INSERT INTO bank_accounts (`number`, `amount`) VALUES('0123456', -5000);
INSERT INTO bank_accounts (`number`, `amount`) VALUES('0999999', 5000);
```

この処理は2つのSQLが必ずセットで成功する必要があります。

もし `1` の処理だけ成功して `2` の処理が失敗すると `0123456` の口座からお金だけ取られて取引相手である `0999999` にはお金が入金されない事になります。

このような状況を防ぐ為に利用するのがトランザクション機能です。

まず以下のSQLを実行してみましょう。

これはトランザクションを開始する為のSQLになります。

```mysql
START TRANSACTION;
```

以下のように表示されればOKです。

```sql
mysql> START TRANSACTION;
Query OK, 0 rows affected (0.00 sec)
```

次に `1` のSQLを実行してみます。

```mysql
INSERT INTO bank_accounts (`number`, `amount`) VALUES('0123456', -5000);
```

この時点で口座全体を確認します。

```sql
mysql> SELECT * FROM bank_accounts;
+----+---------+--------+--------------+---------------------+---------------------+
| id | number  | amount | lock_version | created_at          | updated_at          |
+----+---------+--------+--------------+---------------------+---------------------+
|  1 | 0123456 |  10000 |            0 | 2018-03-24 09:12:34 | 2018-03-24 09:12:34 |
|  2 | 0123456 |  10000 |            0 | 2018-03-24 09:12:34 | 2018-03-24 09:12:34 |
|  3 | 0999999 |   5000 |            0 | 2018-03-24 09:12:34 | 2018-03-24 09:12:34 |
|  4 | 0999999 |   2000 |            0 | 2018-03-24 09:12:35 | 2018-03-24 09:12:35 |
|  5 | 0123456 |  -5000 |            0 | 2018-03-24 09:36:12 | 2018-03-24 09:36:12 |
+----+---------+--------+--------------+---------------------+---------------------+
5 rows in set (0.00 sec)
```

すると当然ですが、先程登録した `-5000` の記録がテーブルに記録されています。

次に以下のSQLを実行してみましょう。

```sql
ROLLBACK;
```

すると先程記録された `-5000` の記録がなかった事になっています。

```sql
mysql> SELECT * FROM bank_accounts;
+----+---------+--------+--------------+---------------------+---------------------+
| id | number  | amount | lock_version | created_at          | updated_at          |
+----+---------+--------+--------------+---------------------+---------------------+
|  1 | 0123456 |  10000 |            0 | 2018-03-24 09:12:34 | 2018-03-24 09:12:34 |
|  2 | 0123456 |  10000 |            0 | 2018-03-24 09:12:34 | 2018-03-24 09:12:34 |
|  3 | 0999999 |   5000 |            0 | 2018-03-24 09:12:34 | 2018-03-24 09:12:34 |
|  4 | 0999999 |   2000 |            0 | 2018-03-24 09:12:35 | 2018-03-24 09:12:35 |
+----+---------+--------+--------------+---------------------+---------------------+
4 rows in set (0.00 sec)
```

これはロールバックと呼ばれる機能でトランザクション中の変更をなかった事にする命令です。

トランザクション中に何らかの問題が起こった場合このロールバック機能を使えば、`0123456` の口座から `-5000` されただけという状況を回避出来ます。

次に最後まで取引を成立させる例を見てみます。

今度は以下の順番でSQLを実行して下さい。

```sql
START TRANSACTION;
INSERT INTO bank_accounts (`number`, `amount`) VALUES('0123456', -5000);
INSERT INTO bank_accounts (`number`, `amount`) VALUES('0999999', 5000);
COMMIT;
```

`COMMIT;` はトランザクションの正常終了を宣言する命令です。

テーブルの中身を確認すると以下のように意図したレコードが入っている事を確認出来ます。

```sql
mysql> SELECT * FROM bank_accounts;
+----+---------+--------+--------------+---------------------+---------------------+
| id | number  | amount | lock_version | created_at          | updated_at          |
+----+---------+--------+--------------+---------------------+---------------------+
|  1 | 0123456 |  10000 |            0 | 2018-03-24 09:12:34 | 2018-03-24 09:12:34 |
|  2 | 0123456 |  10000 |            0 | 2018-03-24 09:12:34 | 2018-03-24 09:12:34 |
|  3 | 0999999 |   5000 |            0 | 2018-03-24 09:12:34 | 2018-03-24 09:12:34 |
|  4 | 0999999 |   2000 |            0 | 2018-03-24 09:12:35 | 2018-03-24 09:12:35 |
|  6 | 0123456 |  -5000 |            0 | 2018-03-24 09:43:40 | 2018-03-24 09:43:40 |
|  7 | 0999999 |   5000 |            0 | 2018-03-24 09:43:41 | 2018-03-24 09:43:41 |
+----+---------+--------+--------------+---------------------+---------------------+
6 rows in set (0.00 sec)
```

2つの口座残高を確認しても意図した通りに処理が終わった事を確認出来ます。

```sql
mysql> SELECT SUM(amount) FROM bank_accounts WHERE number = '0123456';
+-------------+
| SUM(amount) |
+-------------+
|       15000 |
+-------------+
1 row in set (0.00 sec)

mysql> SELECT SUM(amount) FROM bank_accounts WHERE number = '0999999';
+-------------+
| SUM(amount) |
+-------------+
|       12000 |
+-------------+
1 row in set (0.00 sec)
```

### トランザクションが持つ4つの特性

トランザクション機能は以下の4つを満たしている必要があります。

MySQLで実装されているトランザクション機能も以下の条件を満たしています。

それぞれの頭文字を取って「ACID特性」と呼ばれます。

Qiitaに分かりやすい説明記事があったので載せておきます。

[データベースさわったこと無い新人向けトランザクション入門](https://qiita.com/komattio/items/838ea5df68eb076e8099)

※ 以下の説明は上記記事からの引用です。

- 原子性（Atomicity）

>トランザクションが終わった時に、そこに含まれていた更新処理は全て実行されるか、全て実行されない状態で終わることを保証する性質。
COMMITで更新が確定されるか、ROLLBACKで元に戻されるかの二択というイメージです。

- 一貫性（Consistency）

>トランザクションに含まれる処理はそれぞれの制約を満たすという性質。
トランザクションの途中で制約違反の処理があった場合、処理を中断してトランザクション実行前に戻すことでデータの一貫性が保たれる。
そのため、あるデータは更新されているが、その他のデータは更新されないというような状態を持たないということ。

- 独立性（Isolation）

>あるトランザクションが実行中の場合、もう一方のその他のトランザクションの影響を受けないという性質。
トランザクション実行中に、その他のトランザクションがデータの更新を行ったとしても実行中のトランザクションはその更新結果の影響を受けないというイメージ。

- 永続性（Durability）

>トランザクションが完了した後、そのデータ状態が保存され失われることは無いという性質。
この性質の保証のために、一般的にはトランザクションのログを記録しておき、障害が起きた場合はログを使用して障害発生前の状態に復旧する。


## 最後に

以上がMySQLの基礎になります。

MySQLは非常に奥が深く、ここに書いてある内容以外にも様々なテクニックが存在します。

特にレプリケーションに関する知識、テクニックは重要です。

困った際は公式リファレンスを読む事をオススメします。

日本語版は1つ前のバージョンなので注意して下さい。

- [（公式）英語](https://dev.mysql.com/doc/relnotes/mysql/5.7/en/)
- [公式（日本語）](https://dev.mysql.com/doc/refman/5.6/ja/preface.html)

レプリケーションや良いテーブル設計等のテクニックは設計レッスンで詳しく解説します。

出来る限り重要な技術を効率良く習得出来る内容を目指していきます。
