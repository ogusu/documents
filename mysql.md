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