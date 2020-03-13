## MySQL のインストールと初期設定

## 環境


- OS: Ubuntu 18.04
- MySQL: 5.7

## MySQLのインストール

MySQL をパッケージでインストールします

```
$ sudo apt update
$ sudo apt install mysql-server
```

インストールされたパッケージとバージョンを確認します.

```
$ dpkg -l | grep mysql
ii  mysql-client-5.7               5.7.29-0ubuntu0.18.04.1             amd64        MySQL database client binaries
ii  mysql-client-core-5.7          5.7.29-0ubuntu0.18.04.1             amd64        MySQL database core client binaries
ii  mysql-common                   5.8+1.0.4                           all          MySQL database common files, e.g. /etc/mysql/my.cnf
ii  mysql-server                   5.7.29-0ubuntu0.18.04.1             all          MySQL database server (metapackage depending on the latest version)
ii  mysql-server-5.7               5.7.29-0ubuntu0.18.04.1             amd64        MySQL database server binaries and system database setup
ii  mysql-server-core-5.7          5.7.29-0ubuntu0.18.04.1             amd64        MySQL database server binaries
```

MySQL バージョン 5.7.29 が入っていることがわかります.

## 安全な初期設定

mysql では データにアクセスするため、ユーザとパスワードを使います。

ややこしいですが、 `OS のroot ユーザ` とは別に, `MySQLのroot ユーザ`も存在します。

ここでは、ランダムなパスワードを簡単に生成する `pwgen` を使って `MySQL の root ユーザ ` のパスワードを生成します。

```
// pwgen のインストール
$ sudo apt update
$ sudo apt install pwgen

// pwgen でパスワードを作成. 例では 20文字の大文字小文字英数記号 です
// セキュリティ強度の観点から、12文字以上を推奨します。
$ pwgen -y 20
```

ここで表示されたものから適当なものを選んで、保存しておいてください。

セキュリティの高い初期設定をするために `mysql_secure_instllation` というコマンドが用意されています. 


```
// mysql_secure_installtion の実行

$ sudo mysql_secure_installation


Securing the MySQL server deployment.

Connecting to MySQL using a blank password.

VALIDATE PASSWORD PLUGIN can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD plugin?

Press y|Y for Yes, any other key for No: yes

There are three levels of password validation policy:

LOW    Length >= 8
MEDIUM Length >= 8, numeric, mixed case, and special characters
STRONG Length >= 8, numeric, mixed case, special characters and dictionary                  file

Please enter 0 = LOW, 1 = MEDIUM and 2 = STRONG: 1         // 1 を選択( 8文字以上の大文字小文字数字と記号からなる文字列)
Please set the password for root here.

New password:                                              // mysql の root ユーザのパスワードを入力

Re-enter new password:                                     // mysql の root ユーザのパスワードをもういちど入力

Estimated strength of the password: 100
Do you wish to continue with the password provided?(Press y|Y for Yes, any other key for No) : y // y を入力
By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.

Remove anonymous users? (Press y|Y for Yes, any other key for No) : y // アノニマスユーザの削除. yを入力
Success.


Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.

Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y // root ユーザのリモートアクセス禁止. yを入力 
Success.

By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.


Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y // テストデータベースの削除. yを入力
 - Dropping test database...
Success.

 - Removing privileges on test database...
Success.

Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.

Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y          // 権限テーブルの即時反映. yを入力
Success.

All done!
```

`All done!` と表示されたらOKです。

## mysql にアクセスする

MySQL 5.7 以降では、`auth_socket` がデフォルトで使用されているため、 mysql でのパスワードではなく、OS のパスワードでのログインになります。

次のようにして、 root ユーザでログインします。

```
$ sudo mysql
```


`Your_PassWord` の部分は `mysql_secure_instllation` で利用した `mysql のroot アカウント`のパスワードを利用して下さい. 

注意点は、行末を `;` で終わらせることです.

```
// mysql にログインした状態で、次のコマンドを実行.
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'Your_PassWord';
```


次以降、mysql にログインする場合は、次のようになります

```
$ mysql -u root -p
```

## 新規のデータベースを作成する

`demo000` という名前のデータベースを作成します.

```
mysql> create database demo000;
Query OK, 1 row affected (0.00 sec)


mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| demo000            | // 作成されている
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

```

## MySQLのユーザ作成

次の構文に従って、MySQL のユーザを作成します。

```
//構文: 
GRANT 権限 ON db_name.table_name TO 'user_name'@'src_network' IDENTIFIED BY '****';

```

詳細な権限については [MySQL 5.6 リファレンスマニュアル 6.2.1 MySQL で提供される権限 ](https://dev.mysql.com/doc/refman/5.6/ja/privileges-provided.html) を参照ください。

今回は ローカルから `demo000` データベースにのみ`ALL` 権限を持つ `labot` ユーザを作成します. 

```
mysql> grant all on demo000.* to 'labot'@'localhost' identified by "****"; // "****" の部分は適切なパスワードに変更してください.
Query OK, 0 rows affected, 1 warning (0.00 sec)

```

もし、エラーが出て成功しなかったら、エラー文を読んで解決するよう頑張ってください。


## 新規作成したユーザでログイン確認


```
$ mysql -u labot -p
Enter password:     # パスワードを入力

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 13
Server version: 5.7.29-0ubuntu0.18.04.1 (Ubuntu)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
```

次のようなプロンプトになればOKです

```
mysql>
```

## 権限の確認

mysql にログイン後、次のコマンドで権限を確認します.

```
mysql> show grants;
+------------------------------------------------------------+
| Grants for labot@localhost                                 |
+------------------------------------------------------------+
| GRANT USAGE ON *.* TO 'labot'@'localhost'                  |
| GRANT ALL PRIVILEGES ON `demo000`.* TO 'labot'@'localhost' |
+------------------------------------------------------------+
2 rows in set (0.00 sec)


```

## 変数の確認

`show variables` で変数の値を確認することができますが、量が多いので、フィルターして表示します.

以下の例では、 `server`  を含む文字列を変数名にもつものを確認します.

sql では `%` がワイルドカードな文字列を意味します.

```
mysql> show variables like "%server%";
+---------------------------------+--------------------------------------+
| Variable_name                   | Value                                |
+---------------------------------+--------------------------------------+
| character_set_server            | latin1                               |
| collation_server                | latin1_swedish_ci                    |
| innodb_ft_server_stopword_table |                                      |
| server_id                       | 0                                    |
| server_id_bits                  | 32                                   |
| server_uuid                     | 501c4d73-641a-11ea-a44b-064ff0f4a7fa |
+---------------------------------+--------------------------------------+
6 rows in set (0.00 sec)
```


## 変数の変更

先ほどの変数の中で `character_set_server` が `latin1` ということがわかりました。

日本語などの[マルチバイト文字列](https://ja.wikipedia.org/wiki/マルチバイト文字)を`latin1`で操作すると、文字化けなどの不都合が起こることがあります。

そこでデフォルトの文字を `utf8mb4` に変更します.

```
// 変更前の値を確認します.
mysql>  SHOW VARIABLES like 'char%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | latin1   // latin1         |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | latin1   // latin1         |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)

// character_set_databaseを utf8mb4 に変更します.
mysql> SET character_set_database=utf8mb4;
Query OK, 0 rows affected, 1 warning (0.00 sec)

// character_set_server を utf8mb4 に変更します.
mysql> SET character_set_server=utf8mb4;                                                               
Query OK, 0 rows affected, 1 warning (0.00 sec)

// 変数を確認します
mysql>  SHOW VARIABLES like 'char%';
+--------------------------+----------------------------+
| Variable_name            | Value                      |
+--------------------------+----------------------------+
| character_set_client     | utf8                       |
| character_set_connection | utf8                       |
| character_set_database   | utf8mb4   // not latin1    |
| character_set_filesystem | binary                     |
| character_set_results    | utf8                       |
| character_set_server     | utf8mb4   // not latin1    |
| character_set_system     | utf8                       |
| character_sets_dir       | /usr/share/mysql/charsets/ |
+--------------------------+----------------------------+
8 rows in set (0.00 sec)


```

## 変数を永続的に変更する.

先ほど方法では、一時的に変更することはできましたが、MySQL を再起動すると設定は消えてしまいます。

mysql を再起動して、文字が latin1 になっていることを確認する.
```
// mysql の再起動
$ sudo service mysql restart

// mysql にログイン
$ mysql -u root -p 
Enter password:  // パスワードを入力

// 変数を確認
mysql> show variables like "%char%";
+--------------------------------------+----------------------------+
| Variable_name                        | Value                      |
+--------------------------------------+----------------------------+
| character_set_client                 | utf8                       |
| character_set_connection             | utf8                       |
| character_set_database               | latin1 // latin1           |
| character_set_filesystem             | binary                     |
| character_set_results                | utf8                       |
| character_set_server                 | latin1 // latin1           |
| character_set_system                 | utf8                       |
| character_sets_dir                   | /usr/share/mysql/charsets/ |
| validate_password_special_char_count | 1                          |
+--------------------------------------+----------------------------+
9 rows in set (0.01 sec)
```

mysql の再起動のたびに変数が変わるのは運用上大変なので、再起動しても変更しないようにします.

```
//mysql の設定ファイルを編集
$ sudo vim /etc/mysql/mysql.conf.d/mysqld.cnf

[mysqld]
:
character-set-server = utf8mb4 //mysqld セクションの最後に 追加する
:

// mysql の再起動
$ sudo service mysql restart // reload ではなく restart を使うこと

// mysql にログイン
$ mysql -u root -p
Enter password:          // パスワードを入力

// 変数を確認
mysql> show variables like "%char%";
+--------------------------------------+----------------------------+
| Variable_name                        | Value                      |
+--------------------------------------+----------------------------+
| character_set_client                 | utf8                       |
| character_set_connection             | utf8                       |
| character_set_database               | utf8mb4 // not latin1      |
| character_set_filesystem             | binary                     |
| character_set_results                | utf8                       |
| character_set_server                 | utf8mb4 // not latin1      |
| character_set_system                 | utf8                       |
| character_sets_dir                   | /usr/share/mysql/charsets/ |
| validate_password_special_char_count | 1                          |
+--------------------------------------+----------------------------+
9 rows in set (0.00 sec)

```

同様に、今後 mysql のチューニングなどで変数を変更したい時は、このように設定を変更・反映させます。
