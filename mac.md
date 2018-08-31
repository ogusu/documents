# mac
macにphpなどをインストールする手順。HomeBrewというパッケージ管理ソフトがあるためそれを利用する。

macOS High Sierra(10.13)にはデフォルトでphp7.1が入っているが、composerが使うintlがサポート切れで単体で入れることができない。HomeBrewで配布されているphp@7.1にはコンポーネントが同梱されており、intlも含んでいるため、HomeBrewでインストールして、デフォルトで入っているphp7.1と差し替えることで済む。以下手順。

[High Sierraでintlのインストール方法](https://stackoverflow.com/questions/46652968/install-intl-php-extension-osx-high-sierra)

# HomeBrew

```php
# インストール
https://brew.sh/
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
# アップデート
$ brew update
$ brew upgrade
```

# PHP7.1
```sh
# インストール
$ brew install php@7.1
# アップグレード
$ brew upgrade php@7.1
# Symlinks for references in Cellar:
$ brew link --overwrite --force php@7.1
# Change PHP path in my bash profile
$ echo 'export PATH="/usr/local/opt/php@7.1/bin:$PATH"' >> ~/.bash_profile
$ echo 'export PATH="/usr/local/opt/php@7.1/sbin:$PATH"' >> ~/.bash_profile
# Check for Intl
$ php -m | grep intl
```

# composer

[composer](https://getcomposer.org/download/)

## システム要件
* HTTP サーバー。例: Apache。mod_rewrite が推奨されますが、必須ではありません。
* PHP 5.6.0 以上 (PHP 7.1 も含む)
* mbstring PHP 拡張
* intl PHP 拡張
* simplexml PHP 拡張

## インストール
```php
# インストーラーのダウンロード
$ php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
$ php -r "if (hash_file('SHA384', 'composer-setup.php') === '544e09ee996cdf60ece3804abc52599c22b1f40f4323403c44d44fdfdd586475ca9813a858088ffbc1f233e9b180f061') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
# インストール
$ php composer-setup.php
# 後始末
$ php -r "unlink('composer-setup.php');"
$ mv composer.phar /usr/local/bin/composer
```

## CakePHPアプリケーションの作成
```sh
$ sudo php /usr/local/bin/composer create-project --prefer-dist cakephp/app cake_study1
```

### トラブルシューティング
intlがない場合
>Problem 1	
	- cakephp/cakephp 3.6.5 requires ext-intl * -> the requested PHP extension intl is missing from your system.

timeoutする場合
```sh
# タイムアウトを伸ばし、キャッシュは使わない設定にする。
$ vim ~/.composer/config.json
```
```json
{
    "config": {
        "process-timeout":      600,
        "preferred-install":    "dist",
        "github-protocols":     ["https"]
    }
}
```

```sh
# 途中で止まった場合はキャッシュをクリアする。
$ composer clear-cache
# もしpermissionなしの場合はsudoでcomposer実行してキャッシュがrootになっているかもしれないので、キャッシュをクリア。
$ rm -rf ~/.composer/cache
```

opensll

>The OpenSSL library (0.9.8zc) used by PHP does not support TLSv1.2 or TLSv1.1.			
If possible you should upgrade OpenSSL to version 1.0.1 or above.
```sh
# 既存のものとbrewで入っているもののパス確認
$ which openssl		
    /usr/bin/openssl
$ /usr/bin/openssl version		
    OpenSSL 0.9.8zg 14 July 2015	
		
$ brew list openssl		
    /usr/local/Cellar/openssl/1.0.2o_2/bin/openssl	
$ /usr/local/Cellar/openssl/1.0.2o_2/bin/openssl version
    OpenSSL 1.0.2o  27 Mar 2018
# 入っているがPATHがとおっていない。		
パスを上書き		
$ echo "export PATH=/usr/local/Cellar/openssl/1.0.2o_2/bin:$PATH" >> ~/.bash_profile		
$ source ~/.bash_profile		
# 確認		
$ which openssl
$ openssl version
```

