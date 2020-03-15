# mysql のバックアップと復元

## 環境

- OS: Ubuntu 18.04
- DB: MySQL 5.7 (InnoDB)

# バックアップ

ここでは MySQL ([InnoDB](https://dev.mysql.com/doc/refman/5.6/ja/innodb-default-se.html)) のデータをバックアップする方法をおこないます。

MySQL(MyISAM) のバックアップでも使えますが、その場合は最適なオプションが異なります。

2種類の方法で行います. 

- 特定のデータベースをバックアップする方法
- 全てのデータベースをバックアップする方法

## 特定のデータベースをバックアップする方法

```
構文:  mysqldump [オプション] --single-transaction  -u DBユーザ名 -p DB名 > 出力先ファイル名 2> dump.err
```

InnoDBの場合, `オプション` には `--single-transaction` をつけることを推奨します。 このオプションをつけることによって、データベースをロックせずにバックアップを取得することができます.

`2> dump.errr` は[標準エラー出力](https://duckduckgo.com/?q=%E6%A8%99%E6%BA%96%E3%82%A8%E3%83%A9%E3%83%BC%E5%87%BA%E5%8A%9B)をテキストに吐き出す時につかいます.

```
// labot ユーザで demo_db_000 データベースをバックアップする
$ mysqldump --single-transaction -u labot -p demo_db_000 > demo_db_00.sql 2> dump.err
```

   
## 全てのデータベースをバックアップする方法

全てのデータベースをバックアップする際には `--all-databases` オプションを利用します.

```
構文:  mysqldump [オプション] --single-transaction  -u DBユーザ名 -p --all-databases > 出力先ファイル名 2> dump.err
```

```
// labot ユーザでアクセスできる全データベースをバックアップする
$ mysqldump --single-transaction -u labot -p --all-databases > demo_db_00_all.sql 2> dump.err
```

# 復元

ここではMySQL の復元を行います。

復元のことを `リストア` とも言います。

2種類の方法を紹介します.

- 特定のデータベースを復元する方法
- 全てのデータベースを復元する方法

## 特定のデータベースを復元する方法

```
構文:  mysqldump [オプション] -u DBユーザ名 -p DB名 < ダンプファイル > backup.out 2>&1
```

## 全てのデータベースを復元する方法

```
構文:  mysqldump [オプション] -u root -p < ダンプファイル > backup.out 2>&1
```


## 付録

以下のサイトでは、バックアップや復元などの際に気をつけることなどがまとまっています。ぜひ目を通してください。

- [DBAのためのmysqldumpのtips 25選 :Yakst](https://yakst.com/ja/posts/1355)
