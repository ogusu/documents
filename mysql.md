# mysql

## macでmysql5.6をインストール

Homebrewで簡単にインストール可能。ただしリポジトリが消えているようなので注意。
~~~
brew search mysql
# mysql@5.6が見つかるはず。

brew install mysql@5.6

# 正常にインストールされて入ればすでに起動されているはず。
mysql.server status
~~~

PATHなどを設定
~~~
# ~/.bash_profileに以下を追加
export PATH="/usr/local/opt/mysql@5.6/bin:$PATH"
~~~

## 初期設定

rootのパスワードやその他セキュリティの初期設定をする。
~~~
# 以下を実行。rootの初期パスワードはブランクなのでEnterでOK
mysql_secure_installation
~~~

/etc/my.cnfに文字コードなど最低限設定を追加
~~~
[mysqld]
character-set-server=utf8
# mysql8.0用。セキュリティ強化無効化
default_authentication_plugin=mysql_native_password

[mysqldump]
default-character-set=utf8

[mysql]
default-character-set=utf8

[client]
default-character-set=utf8
~~~

再起動して反映させる
~~~
mysql.service restart
~~~

## 接続確認

Workbenchで繋げるか、以下で確認できる。
~~~
# ユーザーroot、パスワード指定で接続
mysql -u root -p
# プロンプトが出たら接続OK。以下を打って設定の反映を確認
show variables like "character_set%";
show variables like "%auth%"; #8.0以降の場合
~~~

## 権限の調整

一度シャットダウンすると以降起動しないことがある。

~~~
koiking:mysql takanori$ mysql.server start
Starting MariaDB
.181017 21:26:50 mysqld_safe Logging to '/usr/local/var/mysql/koiking.local.err'.
181017 21:26:50 mysqld_safe Starting mysqld daemon with databases from /usr/local/var/mysql
..... ERROR! 
~~~

/usr/local/var/mysqlのフォルダのpermissionに問題がある。インストール直後は以下のようにインストールしたユーザーが所有者で、起動ユーザーやグループへ書き込み権限などが足りない。

~~~
koiking:mysql takanori$ ls -la /usr/local/var/mysql
total 362568
drwxr-xr-x   12 user  admin       384 10 17 17:11 .
drwxrwxr-x    5 user  admin       160  8 26 21:03 ..
-rw-rw----    1 user  admin     16384 10 17 21:26 aria_log.00000001
-rw-rw----    1 user  admin        52 10 17 21:26 aria_log_control
drwx------  750 user  admin     24000  9 22 07:47 core_dev
-rw-r-----    1 user  admin     14270 10 16 21:53 ib_buffer_pool
-rw-rw----    1 user  admin  50331648 10 17 21:26 ib_logfile0
-rw-rw----    1 user  admin  50331648 10 16 21:53 ib_logfile1
-rw-rw----    1 user  admin  79691776 10 16 21:53 ibdata1
-rw-rw----    1 user  admin         0  8 25 22:19 multi-master.info
drwx------   91 user  admin      2912  8 25 22:17 mysql
drwx------    3 user  admin        96  8 25 22:17 performance_schema
~~~

専用のユーザーの_mysqlに書き換える。

~~~
koiking:mysql takanori$ sudo chown -R _mysql /usr/local/var/mysql/
koiking:mysql takanori$ sudo chmod -R g+rwx /usr/local/var/mysql/
~~~

以下の状態に書き換わり、起動も成功する。

~~~
koiking:mysql takanori$ ls -la /usr/local/var/mysql/
total 362568
drwxrwxr-x   12 _mysql  admin       384 10 17 17:11 .
drwxrwxr-x    5 user    admin       160  8 26 21:03 ..
-rw-rwx---    1 _mysql  admin     16384 10 17 21:26 aria_log.00000001
-rw-rwx---    1 _mysql  admin        52 10 17 21:26 aria_log_control
drwxrwx---  750 _mysql  admin     24000  9 22 07:47 core_dev
-rw-rwx---    1 _mysql  admin     14270 10 16 21:53 ib_buffer_pool
-rw-rwx---    1 _mysql  admin  50331648 10 17 21:26 ib_logfile0
-rw-rwx---    1 _mysql  admin  50331648 10 16 21:53 ib_logfile1
-rw-rwx---    1 _mysql  admin  79691776 10 16 21:53 ibdata1
-rw-rwx---    1 _mysql  admin         0  8 25 22:19 multi-master.info
drwxrwx---   91 _mysql  admin      2912  8 25 22:17 mysql
drwxrwx---    3 _mysql  admin        96  8 25 22:17 performance_schema

koiking:mysql takanori$ mysql.server start
Starting MariaDB
.181017 21:33:06 mysqld_safe Logging to '/usr/local/var/mysql/koiking.local.err'.
181017 21:33:06 mysqld_safe Starting mysqld daemon with databases from /usr/local/var/mysql
 SUCCESS! 
~~~