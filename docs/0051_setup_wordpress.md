# Wordpress を構築してみよう


# 環境

- OS :  Ubuntu18.04
- DB :  MySQL 5.7
- web:  Nginx
- php:  7.2

# 必要になるかもしれない知識

- [ ] `.ssh/config` の使い方
- [ ] scp の使い方 (mac からサーバへのデータやファイルの転送)
- [ ] 圧縮ファイルの展開

## 構築


[こちら](https://wpdocs.osdn.jp/WordPress_%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB) を参考に自分のサーバに WordPress を構築してみよう.

`https://<あなたのgithub アカウント>.<指定したドメイン>/wordpress` に表示させるようにしましょう.


## PHP の設定

- そのままでは phpからmysql を実行できないので[LAMP環境の構築](0024_lamp.md) を行う.

## Nginx の設定

課題
- [ ] `https://<あなたのgithub アカウント>.<指定したドメイン>/wordpress` に wordpress が表示されるような設定をしてください.

## MySQL の設定

課題
- [ ] wordpress 用のデータベース(ローカルからのアクセスのみ可能)を作成してください
- [ ] wordpress 用のデータベースにだけアクセスできるユーザを作成してください.

### Wordpress の設定 

課題

- [ ] `https://<あなたのgithub アカウント>.<指定したドメイン>/wordpress` に wordpress が表示されるように wordpress のディレクトリを設定してください.
