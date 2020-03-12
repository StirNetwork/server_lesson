## 一般ユーザーの作成&削除

### （1）管理者権限アカウントでのログイン

ubuntu ユーザでのssh が管理者権限でのログインに該当するので、省略します。

### （2）一般ユーザーの作成&削除(例として一般ユーザー名をlabotとする)

一般ユーザ labot の作成と削除を行います。

一般ユーザとは、管理者権限のないユーザのことを指します.

```
[ubuntu@ip-172-16-1-21]# sudo useradd labot　←　一般ユーザーlabotの作成
[ubuntu@ip-172-16-1-21]# id labot　←labot ユーザの情報確認
uid=1001(labot) gid=1001(labot) groups=1001(labot)
[ubuntu@ip-172-16-1-21]# sudo passwd labot　← labot のパスワード変更
Enter new UNIX password:                    ← labot ユーザのパスワードを入力
Retype new UNIX password:                   ← 同じパスワードを入力
passwd: password updated successfully

[ubuntu@ip-172-16-1-21]# sudo userdel -r labot 　←　一般ユーザーlabotの削除
userdel: labot mail spool (/var/mail/labot) not found
userdel: labot home directory (/home/labot) not found

[ubuntu@ip-172-16-1-21]# id labot　         ←labot ユーザの情報確認
id: ‘labot’: no such user                 ← labot ユーザが削除されていることがわかる

```


### （3）管理者権限のユーザの作成(例として一般ユーザー名をlabotとする)

まず、一般ユーザを作成します。
```
[ubuntu@ip-172-16-1-21]# sudo useradd labot　←　一般ユーザーlabotの作成
[ubuntu@ip-172-16-1-21]# id labot　←labot ユーザの情報確認
[ubuntu@ip-172-16-1-21]# sudo passwd labot　← labot のパスワード変更(任意の文字列)
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully

```

   一般ユーザではrootになれないことを確認します。
```
[ubuntu@ip-172-16-1-21]# sudo su - labot ← まず 一般ユーザ labot になる
$ sudo su - root  ← labot ユーザから root ユーザになる
[sudo] password for labot:  ← labot ユーザのパスワードを入力
labot is not in the sudoers file.  This incident will be reported. ← root ユーザになれないことを確認。labot ユーザにsudoers ファイルにない
$ exit                      ← labot ユーザでログアウトします
```

一般ユーザに管理者権限を付与する。
管理者権限になるにはどのグループに所属したらいいのか確認する。

```
   // 管理者権限 についての設定ファイルの確認
   sudo cat /etc/sudoers
   
#
# This file MUST be edited with the 'visudo' command as root.
#
# Please consider adding local content in /etc/sudoers.d/ instead of
# directly modifying this file.
#
# See the man page for details on how to write a sudoers file.
#
   Defaults        env_reset
   Defaults        mail_badpass
   Defaults        secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"

# Host alias specification

# User alias specification

# Cmnd alias specification

# User privilege specification
root    ALL=(ALL:ALL) ALL

# Members of the admin group may gain root privileges
%admin ALL=(ALL) ALL                              ←admin グループに所属していたら、root 権限を持てるかもしれない

# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL                         ← sudo グループに所属していたら、sudo コマンドが使える(管理者権限になる)

# See sudoers(5) for more information on "#include" directives:

#includedir /etc/sudoers.d
```

   今回は `sudo` グループに追加します。
  


## ユーザをsudo グループに追加する

```
[ubuntu@ip-172-16-1-21]# id labot　                   ←labot ユーザの情報確認
uid=1001(labot) gid=1001(labot) groups=1001(labot)    ← sudo グループに所属していないことを確認
[ubuntu@ip-172-16-1-21]# sudo usermod -aG sudo labot  ← labot ユーザを sudo グループに追加
[ubuntu@ip-172-16-1-211]# id labot                    ← labot ユーザのグループを確認
uid=1001(labot) gid=1001(labot) groups=1001(labot),27(sudo)
```

 labot ユーザから root ユーザになってみます.
```
[ubuntu@ip-172-16-1-211]# sudo su - labot             ← まず、labot ユーザになります。
No directory, logging in with HOME=/
$ sudo su -                                           ← root になります。
[sudo] password for labot: 
[root@ip-172-16-1-211]# id                            ← 現在のユーザ情報を確認します。
uid=0(root) gid=0(root) groups=0(root)                ← root になっています。
[root@ip-172-16-1-211]# exit                          ← 一つ前のユーザに戻ります(root ユーザをログアウトする)。
logout                                                ← ログアウトしました。
$ exit                                                ← labot ユーザをログアウトする
ubuntu@ip-172-16-1-211:~$                             ← ログアウトしました。

```

## パッケージ管理システム設定

```
// パッケージの更新情報を確認します. 
[ubuntu@ip-172-16-1-211]# sudo  apt update

// パッケージの更新があるものを全て更新します。
[ubuntu@ip-172-16-1-211]# sudo  apt upgrade


// セキュリティ対応のあるパッケージ全て更新します。
[ubuntu@ip-172-16-1-211]# sudo unattended-upgrades


[ubuntu@ip-172-16-1-211]# sudo dpkg-reconfigure -plow unattended-upgrades  //セキュリティ関連のアップデートのみ自動的に更新する設定を有効化する設更新ァイル  ※GUIがでるのでYESを選択して下さい。


```

パッケージの自動更新の詳細については[こちら: help.ubuntu.com](https://help.ubuntu.com/lts/serverguide/automatic-updates.html)をご確認ください。

## ファイアウォール

 Ubuntu でのファイアウォールは[UFW(Ubuntu FireWall)](https://help.ubuntu.com/lts/serverguide/firewall.html)を使うのが便利です。

なお、ファイアウォールの設定をミスると、ログインできなくなる場合もあります。注意して設定しましょう。


```
// ufw の状態を確認する (inactive であることを確認)
[ubuntu@ip-172-16-1-211]# sudo ufw status
Status: inactive 

// port 80 を開ける
[ubuntu@ip-172-16-1-211]# sudo ufw allow 80 ← 80portの接続を許可(allow)する
Rules updated
Rules updated (v6)

// port 22 を開ける
[ubuntu@ip-172-16-1-211]# sudo ufw limit 22 ← 30秒に6回以上のアクセスの場合そのIPを一定時間無効化(limit)する
Rules updated
Rules updated (v6)

// ufw を有効化する
[ubuntu@ip-172-16-1-211]# sudo ufw enable
Command may disrupt existing ssh connections. Proceed with operation (y|n)? y ← エンターを押す
Firewall is active and enabled on system startup

```

このターミナルは開きっぱなしのままにします。

別のターミナルを開いて、サーバにログインできることを確認します。


```
別のターミナルを開いて、サーバにログインできることを確認します。
```

もし、ログインできなかったら、もともと開いたターミナルで ufw を停止してみましょう。

```
 // ufw の無効化 
[ubuntu@ip-172-16-1-211]# sudo ufw disable
```






## システムの停止または再起動


サーバのIPアドレスを固定していないと、再起動時にログインできなくなってしまいます。


今回は、コマンドの紹介だけ行います。 **実行しないでください。**


AWS の時にやってみましょう. 

```
// OS の再起動
[ubuntu@ip-172-16-1-211]# sudo reboot
// OS の停止
[ubuntu@ip-172-16-1-211]# sudo shutdown -h now





