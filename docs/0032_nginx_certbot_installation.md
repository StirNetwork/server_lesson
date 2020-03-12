# nginx と certbot で https 通信を行う


ドメインとグローバルIPアドレスがあっていることを確認してください。 

あっていない場合は、担当の人に聞いてみましょう.

例: domain: demo.quelcode.k80s.net


## certbot のインストール

[certbot](https://certbot.eff.org/) を参考にインストールしてみましょう. 




```
// ppa のインストール
$ sudo apt-get update
$ sudo apt-get install software-properties-common
$ sudo add-apt-repository universe
$ sudo add-apt-repository ppa:certbot/certbot
$ sudo apt-get update
```


```
note: ppa とはなんでしょうか？調べてみましょう.
```

## Install Certbot

```
$ sudo apt-get install certbot python-certbot-nginx
```

letsencrypt 向けのバーチャルドメインの設定を行います. `demo.quelcode.k80s.net`の部分は適宜自分のドメインに読み替えてください.

```
$ sudo vim  /etc/nginx/conf.d/demo.quelcode.k80s.net.conf
server { 
    listen   80;
    server_name demo.quelcode.k80s.net;
    location '/.well-known/acme-challenge' { 
    default_type "text/plain";
      root       /var/www/html;
    } 
    location / {
      return              301 https://$server_name$request_uri;
    }
}
```

## letsencrypt を用いて証明書を作成

letsencrypt を用いて証明書を作成します.

demo.quelcode.k80s.net はご自身のドメインに, <your@mailaddress.co> は自分のメールアドレスに適宜読み替えてください. 

```
$ sudo certbot certonly --webroot --webroot-path /var/www/html \
     -d demo.quelcode.k80s.net --server \
      https://acme-v02.api.letsencrypt.org/directory \
     --agree-tos -m your@mailaddress.co

```

途中で聞かれたら `y`を打鍵してください。

CLI上で`Waiting for verification...`と出たら、登録したメールに「Please Confirm Your EFF Subscription」というメールが届きます。

`Click thi link to confirm your email`という文の下記リンクをクリックします。

Confirmが完了し、CLI上が進行します。


## 証明書ファイルの確認


証明書が一式生成されたことを確認します。

`demo.quelcode.k80s.net` は適宜読み替えてください.

```
$ sudo ls -lah /etc/letsencrypt/live/demo.quelcode.k80s.net
total 12K
drwxr-xr-x 2 root root 4.0K Mar 11 16:40 .
drwx------ 3 root root 4.0K Mar 11 16:40 ..
lrwxrwxrwx 1 root root   46 Mar 11 16:40 cert.pem -> ../../archive/demo.quelcode.k80s.net/cert1.pem
lrwxrwxrwx 1 root root   47 Mar 11 16:40 chain.pem -> ../../archive/demo.quelcode.k80s.net/chain1.pem
lrwxrwxrwx 1 root root   51 Mar 11 16:40 fullchain.pem -> ../../archive/demo.quelcode.k80s.net/fullchain1.pem
lrwxrwxrwx 1 root root   49 Mar 11 16:40 privkey.pem -> ../../archive/demo.quelcode.k80s.net/privkey1.pem
-rw-r--r-- 1 root root  682 Mar 11 16:40 README
```


##  証明書を用いたnginx の https 設定


```
server {
    listen   80;
    server_name demo.quelcode.k80s.net;
    location '/.well-known/acme-challenge' {
    default_type "text/plain";
      root       /var/www/html;
    }

    location / {
      return              301 https://$server_name$request_uri;
    }
}


server {
    listen   443;
    server_name demo.quelcode.k80s.net;
    ssl                  on;
    ssl_certificate      /etc/letsencrypt/live/demo.quelcode.k80s.net/fullchain.pem;
    ssl_certificate_key  /etc/letsencrypt/live/demo.quelcode.k80s.net/privkey.pem;

    ssl_protocols        TLSv1.0 TLSv1.1 TLSv1.2 TLSv1.3;
    #https://weakdh.org/sysadmin.html
    ssl_ciphers 'TLS13+AESGCM+AES128:TLS13+CHACHA20:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-DSS-AES128-GCM-SHA256:kEDH+AESGCM:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:ECDHE-ECDSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-DSS-AES128-SHA256:DHE-RSA-AES256-SHA256:DHE-DSS-AES256-SHA:DHE-RSA-AES256-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:AES:CAMELLIA:DES-CBC3-SHA:!aNULL:!eNULL:!EXPORT:!DES:!RC4:!MD5:!PSK:!aECDH:!EDH-DSS-DES-CBC3-SHA:!EDH-RSA-DES-CBC3-SHA:!KRB5-DES-CBC3-SHA';
    ssl_prefer_server_ciphers on;
    ssl_dhparam /etc/nginx/ssl/dhparams.pem;
    ssl_session_timeout  10m;
    root   /var/www/html;
    index  index.php index.html;
    access_log /var/log/nginx/ssl_app_access.log;
    error_log  /var/log/nginx/ssl_app_error.log;
    location / {
        try_files $uri $uri/ /index.php?$uri&$args;
    }

    location ~ \.php$ {
        try_files $uri =404;
        include /etc/nginx/fastcgi_params;
        fastcgi_pass    unix:/run/php/php7.2-fpm.sock;
        fastcgi_index   index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param  HTTPS on;
    }

}
```

# dhparam ファイルの生成

```

$ sudo mkdir  /etc/nginx/ssl/
$ sudo openssl dhparam -out /etc/nginx/ssl/dhparams.pem 2048
````

# 表示ファイルの生成

```
$echo "hello world" |sudo tee -a /var/www/html/index.html
```

# nginx の再起動

```
$ sudo nginx -t ←ここでエラーが出たら、configなどを見直す.
$ sudo service nginx restart
```

### https 通信を確認する


1. https://demo.quelcode.k80s.net にアクセスする.
2. https://www.ssllabs.com/ssltest/ に移動して、https の強度を確認する.

