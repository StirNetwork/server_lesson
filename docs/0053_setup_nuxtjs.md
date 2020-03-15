# nuxt.js のインストール

# 環境

- OS: Ubuntu18.04


## npx のインストール

nuxt.js のインストールには npx を利用する。

npx は npm 5.2 以降で標準で使えるが、 Ubutntu18.04 の標準パッケージでは npm のバージョンが古いため、 npx を使うことができない

[こちらの記事(英語)](https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-ubuntu-18-04) を参考にして、 npm5.2 以降のをインストールする

npm のバージョンを確認し, npx が使えるようになっていることを確認する.

```
$ npx -v
9.7.1
```

## npx 経由での nuxt アプリの作成

サンプルアプリ `sample-app-000` を作成する

```
// npx create-nuxt-app コマンドで アプリを作成する
// 今回はテストアプリなので、選択肢は適当に選ぶ
$ npx create-nuxt-app sample-app-000

// nuxt アプリの起動
$ cd sample-app-000

// nuxt アプリを起動する
$ npm run dev  &

// プロセスが動いていることを確認する
$ ps aux | grep node
ubuntu    2430 51.4 32.0 1452708 323012 pts/0  Sl+  16:25   0:10 node  //node プロセスが確認できたらok


// ポートを確認する
$ netstat -ant | grep 3000 // default のポートが 3000 
tcp        0      0 127.0.0.1:3000          0.0.0.0:*               LISTEN   // こうなっていたらOK

```

## nginx への設定追加


nginx に 設定を加えて、テストアプリを公開します.


```
// nginx のコンフィグファイルを編集します
$ sudo vim /etc/nginx/conf.d/path_to_config_file.conf

upstream nuxt_app {            // ファイルの一番上に追記する
   server 127.0.0.1:3000;      // 追記     
}                              // 追記

server {
  :
    # port 443 の項目
    location ~ \.php$ {
        try_files $uri =404;
        include /etc/nginx/fastcgi_params;
        fastcgi_pass    unix:/run/php/php7.2-fpm.sock;
        fastcgi_index   index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param  HTTPS on;
    }
    # port 443 のい項目の一番下に以下を追記する.
    # /nuxt で nuxt アプリを表示するようにする設定
    location /nuxt {                                  
        proxy_pass http://nuxt_app/;
        access_log /var/log/nginx/ssl_nuxt_app_access.log;
        error_log  /var/log/nginx/ssl_nuxt_app_error.log;
    }
}

// nginx の設定が正しいか確認する
$ sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful

上記のように出ればOK
エラーがあらば、エラー文を参考に修正する.

// nginx の再起動
// nginx の設定が正しことを確認したら、nginx を再起動する.
// 正しくない状態で再起動すると、起動しないので注意すること.
$ sudo service nginx restart
```

`https://<your.domain>/nuxt` にアクセスして, nuxt のテストアプリが表示されることを確認してください.


無事、サンプルアプリが表示されれば成功です！

## nuxt の永続化

なんらかの不具合で npm のプロセスが落ちると、サービスが停止してしまいます.

また、先ほどの動作確認は開発者向けモードで行いましたが、様々な理由で本番環境(一般のユーザが使う環境)ではビルドしたものを使います. 

node アプリの永続化には `forever` や `pm2` が用いられます。

調べて永続化してみましょう。
