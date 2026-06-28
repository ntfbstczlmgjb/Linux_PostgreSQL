### rpmパッケージ確認
```
# rpm -qa | grep ^post
```
上記でインストール済か否か確認できる

# インストール

# 設定
### 初期化
rpmでインストールしたとき？
```
# /etc/rc.d/init.d/postgresql initdb
```
ここで文字コード指定しないとデフォルトで「en_US.UTF-8」になるの？
```
# /etc/rc.d/init.d/postgresql*** initdb -E EUC_JP
# /etc/rc.d/init.d/postgresql*** initdb -E SJIS
```
上記だとエラーが発生する。
初期設定などで変更・指定する術はないのか？
Windowsの場合「Japanese_Japan.932」となっていることからインストーラーが設定を変更しているっぽい。

「initdb」の検証は大変だよな

osstestのCSV取込の所から大分脱線してしてしまった


yumだと
```
$ su -l postgres
$ /usr/pgsql**/bin/initdb
```

う～んこれだけみてもなにか違うような
rpmの方はサービスの引数で初期化している、下はインストール先フォルダで初期化している。

### サーバーの設定
```
# nano /var/lib/pgsql/data/postgresql.conf
# nano /var/lib/pgsql/9.6/data/postgresql.conf【yumでインストール】

59行目
listen_address = '*'

334行目
log_line_prefix = '%t %u %d '
```

### クライアントからの接続設定
```
# nano /var/lib/pgsql/バージョン/data/pg_hba.conf
```
追加：host       all           all          192.168.85.0/24            md5

「hba」は「host-based authorization」の意味らしい

```
postgres=# pg_ctl reload
hbaの設定だけを反映するなら
```

### iptablesに追加
```
# nano /etc/sysconfig/iptables
```
ポート：5432

直接編集？
-A INPUT -p tcp --dport 5432 -j ACCEPT
[参照](https://qiita.com/YusukeHigaki/items/9bd0c21fbcc47e12b5c1)

[参照](https://www.kakiro-web.com/linux/centos6-iptables.html)

iptablesの設定は「直接編集」と「コマンド」があるけど正式にはどっちなんだろう？

再起動
```
# /etc/rc.d/init.d/iptables restart
```

# 起動
### posgre起動
```
# /etc/rc.d/init.d/postgresql start
```
### posgre自動起動設定
```
# chkconfig postgresql on
```
```
# chkconfig postgresql-9.6 on
```
この名前はどう探すの？

### posgre接続
これは任意の作業？
管理者ユーザー「postgres」のパスワード設定
```
# su -l postgres
-bash-4.1$ psql -c "alter user postgres with password '任意のパスワード' "
```
postgresユーザーはDB作成時に作られる。

一般ユーザー「cent」作成
```
-bash-4.1$ createuser cent
Shall the new role be a superuser? (y/n) y
```
管理者権限を与える場合、Y
この作業はDBのユーザーを追加する場合。スーパーユーザー（postgres）で接続が可能になった時点でクライアントから操作してもいいのでは？

OS側の専用ユーザーは
```
useradd ユーザー名
```
で作る。パスワードは設定しない。DB側のユーザと同じ名前が良い
DBの移設なのか新規なのかで対応が別れる

### psqlの終了
```
postgres# \n
```
「\」はバックスラッシュ


# 削除








*** 以上 ***

[PostgreSQLインストール](https://www.server-world.info/query?os=CentOS_6&p=postgresql)


### これから
PostgreSQL 9.6.5
が現在稼働中、データ移行するなら・・・

~~任意のバージョンをインストールしてみる
ダウンロードはしたので、なんとかしてVMに入れたい
sambaを使って共有領域を作り入れようとおもったけどもしかして
yumで古いバージョンもインストールできる？~~

~~一応「9.6.11」がインストールできた
データもロードしてみた~~

Windowsからのダンプ
```
C:\ pg_dumpall -f d:\dump -U postgres
```
メリット：全てのデータが出力できる。（インデックス情報なども？）
　　　　　簡単
デメリット：DB単位なのでパスワードを数回入力する必要がある

リストア
```
# psql -f ファイル名 -U postgres
```

一般ユーザーでログインしたい

***
一応ある
### 過去導入手順１
rpmパッケージのダウンロードとインストール
```
# rpm -ivh /home/ユーザ/Downloads/パッケージ名.rpm
```
    確認
    # rpm -qa | grep ^pg
    なし

        削除
        #rpm -e pgdg-centos11-11-2.noarch

    確認
    # yum list | grep postgre
    なし

インストール（DBとLibが別）※v11の場合
```
# yum -y install postgresql11（-Libsとpgdg）
# yum -y install postgresql11-server（-Server）
```
インストールの確認
```
# rpm -qa | grep postgre
postgresql
postgresql-libs
postgresql-server
```

バージョン確認（バージョンで違う）
suした後に
```
# psql --version
```

Ver8.x
```
# postgres --version（ホントのPathは/usr/bin/postgres --version）
```
Ver10.x
```
# /usr/pgsql-10/bin/postgres --version
```
アンインストール
```
# yum -y remove postgresql-server
```
-------------------設定----------------------------
DB初期化
#
※rootユーザーだとできない？


外部（クライアント）からの接続
```
# vi /var/lib/pgsql/10/data/postgresql.conf
```
下記を探して編集
#---------------------------------------------------------
#　CONNECTION AND AUTHENTICATION
#---------------------------------------------------------

#Listen_address = 'localhost'　→　Listen_address = '*'
#port = 5432　→　port = 5432


編集が終わったらPostgreSQLの再起動
/etc/rc.d/init.d/postgresql-10 restart

### 過去導入手順２

・インストール

　yum　パッケージ確認　既存で入っているバージョンを確認。
　# yum list | grep postgre

　rpm パッケージのインストール
　# rpm -ivh /home/ユーザー名/Downloads/rpmパッケージ名.rpm

　yum　でインストール
　# yum -y install postgresql10
　# yum -y install postgresql10-server

　yum インストールの確認
　# rpm -qa | grep postgres
　
・DB初期化
　# service postgresql-10 initdb
　※エンコーディングどこで指定する？

・DB設定ファイル編集
　# vi /var/lib/pgsql/10/data/postgresql.conf
　下記を探して編集
　#---------------------------------------------------------
　#　CONNECTION AND AUTHENTICATION
　#---------------------------------------------------------
　#Listen_address = 'localhost'　→　Listen_address = '*'
　#port = 5432　→　port = 5432

　# vi /var/lib/pgsql/10/data/pg_hba.conf
　追加：host       all           all          192.168.85.0/24            md5

　あとiptablesにポート5432の通信許可

・DB起動
　# /etc/rc.d/init.d/postgresql-10 start

・DBに接続
　# su postgres
　bash-4.1$ psql -U postgres
　若しくは
　# vi /var/lib/pgsql/10/data/pg_hba.conf　の編集で「Peer」を「md5」に変更する
　これはセキュリティの一環で管理者でもログインできなくして、専用ユーザを作って運用する方針だと思われる
