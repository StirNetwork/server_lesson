# nginx のインストール

## 概要

このページでは web サーバとしてnginx を起動して、php のプログラムが起動するするのを確認します。

- ubuntu への nginx のインストール
- ubuntu への php のインストール
- php プログラムを起動するためのnginx の設定

## nginx のインストール

```
$ sudo apt install nginx
```

## php のインストール


```
$ sudo apt install php7.2-fpm
```

```
// 初期設定の確認とphpの設定

$ sudo vim /etc/nginx/sites-enabled/default 

:

        root /var/www/html; #←  /var/www/html がルートディレクトリ

:
```


動作確認用のファイルを設置する
```
// 確認用のファイル phpinfo.php を作成します。
// cat ~ から EOF 文末のEOF までコピーして使ってください。 (ヒアドキュメント)
$ cat <<EOF > phpinfo.php
<?php
phpinfo()
?>
EOF

// ルートディレクトリにphpinfo.phpを移動する
$ sudo mv phpinfo.php /var/www/html

// phpinfo.php をサーバで実行する
$ php /var/www/html/phpinfo.php

// 現在の パブリックIPアドレスを確認する

$ curl ifconfig.me  // この結果が、現在ログインしているサーバのIPアドレスです

```

## welcome ページの表示確認

ブラウザにて `http://<アクセスしているサーバのグローバルIP>` にアクセスします。

welcome to nginx! と表示されることを確認します。


###  php プログラムが動かないことの確認

ブラウザにて `http://<アクセスしているサーバのグローバルIP>/phpinfo.php` にアクセスします。

この段階では nginx にphpの設定をしていないので、ファイルがダウンロードされるに止まります。

## nginx でphp が使えるように設定する

`/etc/nginx/sites-enabled/default` を編集します。


```
$ sudo vim /etc/nginx/sites-enabled/default
:
:
        # pass PHP scripts to FastCGI server
        #
        location ~ \.php$ {                                                # コメントアウトを外す
                include snippets/fastcgi-php.conf;                         # コメントアウトを外す
                                                                           # コメントアウトを外す
                # With php-fpm (or other unix sockets):                    # コメントアウトを外す
                #fastcgi_pass unix:/var/run/php/php7.0-fpm.sock;           
                fastcgi_pass unix:/var/run/php/php7.2-fpm.sock;            # コメントアウトを外す. php7.2-fpm.sock に書き換える
                # With php-cgi (or other tcp sockets):
                #fastcgi_pass 127.0.0.1:9000;
        }                                                                  # コメントアウトを外す
:
:
```

nginx への設定反映

```
$ sudo service nginx restart
```

ブラウザにて `http://<アクセスしているサーバのグローバルIP>/phpinfo.php` にアクセスします。

`phpinfo()` の情報が表示されたらOK


```
// テストファイルの削除

$ sudo rm /var/www/html/phpinfo.php
```
