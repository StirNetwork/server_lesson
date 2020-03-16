# ssh/config を用いたssh


# 環境

- OS: Mac, Ubuntu

# 使い方

秘密鍵を用いたssh は次のようなコマンドで実行できる.

```
$ ssh labot_user@1.2.3.4 -i  ~/.ssh/key_name.pem
```

しかし、毎回このようにオプションを追加して記載するのは面倒である.

また、scp コマンドなどで鍵ログインを使う場合にも面倒である。

そういう時に、 `~/.ssh/config` に ssh の方法を記載することでコマンドを簡便に記述することができる.

例えば、次のように記載するとする.

```
$ vim ~/.ssh/config
 Host labot
   User labot_user
   hostname 1.2.3.4
   IdentityFile ~/.ssh/key_name.pem
```

この場合次の二つのコマンドは同じ意味になります。

```
$ ssh labot_user@1.2.3.4 -i  ~/.ssh/key_name.pem
$ ssh labot
```

`ssh labot` の方が圧倒的に簡便である。
