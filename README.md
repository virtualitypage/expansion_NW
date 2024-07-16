![AdGuard_Filter Version](https://img.shields.io/badge/AdGuard_Filter-v1.4.2-blue?style=flat)
![Release Date](https://img.shields.io/badge/Release_Date-July_14_2024-green?style=flat)
![GitHub repo size](https://img.shields.io/github/repo-size/virtualitypage/expansion_NW)

> Table of Contents
- [AdGuard Filter](#adguard-filter)
- [Document - Expansion](#document---expansion)
- [Document - Troubleshooting](#document---troubleshooting)
- [Management Rules](#management-rules)
- [SSH to Github](#ssh-to-github)

## AdGuard Filter

> 基本的な例

* `||`：サブドメインを含むホスト名の先頭に一致します。例えば `||example.org` は `example.org` および `test.example.org` に一致しますが `testexample.org` には一致しません。これは AdGuard Home のカスタム・フィルタリングで使用されます。

* `@@`：例外ルールで使用されるマーカー。一致するホスト名のフィルタリングをオフにする場合は、このマーカーでルールを開始します。

* `@@||example.org`：`example.org` ドメインとそのすべてのサブドメインへのアクセスをブロック解除します。

* `1.2.3.4 example.org`：ドメイン `example.org` のクエリには `1.2.3.4` で応答しますが、サブドメイン `www.example.org` には応答しません。引き続き許可されます。 *注意: これは古い `/etc/hosts` スタイルの構文

  ホストに未指定の IP アドレス `0.0.0.0` またはローカルアドレス `127.0.0.1` などを使用することで、そのホストをブロックすることが出来ます。

```
# example.org の場合 IP アドレス 1.2.3.4 を返す
1.2.3.4 example.org

# 0.0.0.0 で応答して example.com をブロックする
0.0.0.0 example.com
```

* `example.org`：シンプルなドメインルール。ドメイン `example.org` をブロックしますがサブドメイン `www.example.org` はブロックしません。許可されたままになります。

* `/REGEX/`：指定された正規表現に一致するドメインへのアクセスをブロックします。

> コメント

* `!` または `#` で始まる行はコメントであり、フィルタリングエンジンによって無視されます。コメントは通常、ルールの上に配置され、ルールの動作を説明するために使用します。

例：
```
! This is a comment.
# This is also a comment.
```

---

> AdGuard Homeの「クエリ・ログ」取得設定手順 (再起動しなければクエリ・ログを取得できないため非推奨)

* 参照元：https://github.com/AdguardTeam/AdGuardHome/wiki/Configuration#configuration-file

1. ターミナルを開いて SSH でルーターにログインします。

　`$ ssh root@{ip_address|host_name}`

2. クエリログを格納するフォルダを任意の場所に作成します。

3. 以下の箇所を編集します。

```
dir_path: ""
file_enabled: false
```

　`$ vi /etc/AdGuardHome/config.yaml`

```
querylog:
  dir_path: /etc/AdGuard_query
  ignored: []
  interval: 2160h
  size_memory: 1000
  enabled: true
  file_enabled: true
```

4. 設定適用の為にルーターを再起動します。

5. 実行ファイルにログファイルのパスを渡します。ここで指定したファイル名(ここでは"querylog")のjsonファイルが作成され、書き込まれます。

　`$ AdGuardHome -l /etc/AdGuard_query/querylog`

6. 以下のような内容になっていることを確認します。

　`$ cat /etc/AdGuard_query/querylog`

```
YYYY/MM/DD HH24:MI:SS.FF [info] AdGuard Home, version v0.107.46
YYYY/MM/DD HH24:MI:SS.FF [info] This is the first time AdGuard Home is launched
YYYY/MM/DD HH24:MI:SS.FF [info] Checking if AdGuard Home has necessary permissions
YYYY/MM/DD HH24:MI:SS.FF [info] AdGuard failed to bind to port 53: listen tcp 127.0.0.1:53: bind: address already in use

Please note, that this is crucial for a DNS server to be able to use that port.
YYYY/MM/DD HH24:MI:SS.FF [info] AdGuard Home can bind to port 53
YYYY/MM/DD HH24:MI:SS.FF [info] safesearch default: disabled
YYYY/MM/DD HH24:MI:SS.FF [info] Initializing auth module: /etc/AdGuardHome/data/sessions.db
```

* アプリ "AdGuard Home" を再起動(停止・起動)すると、以下の内容が追加されて初期化されるので注意

```
YYYY/MM/DD HH24:MI:SS.FF [info] auth: initialized.  users:0  sessions:0
YYYY/MM/DD HH24:MI:SS.FF [info] web: initializing
YYYY/MM/DD HH24:MI:SS.FF [info] This is the first launch of AdGuard Home, redirecting everything to /install.html
YYYY/MM/DD HH24:MI:SS.FF [info] AdGuard Home is available at the following addresses:
YYYY/MM/DD HH24:MI:SS.FF [info] go to http://127.0.0.1:3000
YYYY/MM/DD HH24:MI:SS.FF [info] go to http://[::1]:3000
YYYY/MM/DD HH24:MI:SS.FF [info] go to http://192.168.1.1:3000
YYYY/MM/DD HH24:MI:SS.FF [info] go to http://[fe80::9683:c4ff:fe42:657e%eth0]:3000
YYYY/MM/DD HH24:MI:SS.FF [info] go to http://192.168.8.1:3000
YYYY/MM/DD HH24:MI:SS.FF [info] Received signal "interrupt"
YYYY/MM/DD HH24:MI:SS.FF [info] stopping AdGuard Home
YYYY/MM/DD HH24:MI:SS.FF [info] stopping http server...
YYYY/MM/DD HH24:MI:SS.FF [info] stopped http server
YYYY/MM/DD HH24:MI:SS.FF [info] stopped
```

## Document - Expansion

> ルーターからメール送信を行う

1. ターミナルを開いて SSH でルーターにログインします。

　`$ ssh root@{ip_address|host_name}`

2. 以下のコマンドを実行します。

　`$ opkg update`

　`$ opkg install msmtp`

3. Google アカウント上でアプリ パスワードを取得します。(手順3の "password" で使用します)

* [アプリ パスワードを作成、管理する](https://myaccount.google.com/apppasswords) をクリックします。(本人確認画面に遷移します)

* アプリ名入力欄に "msmtp" と入力して「作成」をクリックします。表示された 16 桁のアプリ パスワードをメモ等に控えて下さい。

4. "msmtprc" ファイルを以下のように設定します。 *`your-email@gmail.com`の部分は自身の Google アカウントのメールアドレスを入力して下さい。

　`$ vi /etc/msmtprc`

```
# Default settings
defaults
auth           on
tls            on
tls_trust_file /etc/ssl/certs/ca-certificates.crt
logfile        /var/log/msmtp.log

# Gmail account settings
account        gmail
host           smtp.gmail.com
port           587
from           your-email@gmail.com
user           your-email@gmail.com
password       app-password

# Set a default account
account default : gmail
```

5. 設定が完了したら任意のメールアドレスにテストメールを送信します。

　`$ echo "This is a test email." | msmtp -a gmail user@example.com`

6. プロンプト上に何も表示されなければ完了です。

> ルーターからメール送信を行う(メール用のファイルを読み込んでメール送信)

1. メール用のファイル "test.mail" を作成します。

　`$ vi /etc/test.mail`

```
to: your-email@gmail.com
From: user@example.com
Subject: Test mail

This is a test email.
```

2. ターミナルで以下を実行します。

　`$ msmtp user@example.com < /etc/test.mail`

3. プロンプト上に何も表示されなければ完了です。

## Document - Troubleshooting

> 毎分出力されるログを解決する

* 参照元：https://forum.gl-inet.com/t/many-log-data-after-4-4-5-axt1800-mt2500/32854/11

```
DY MON DD HH24:MI:SS YYYY cron.err crond[12345]: USER root pid 1234 cmd . /lib/functions/modem.sh;check_ip
DY MON DD HH24:MI:SS YYYY cron.err crond[12345]: USER root pid 1234 cmd sleep 30;. /lib/functions/modem.sh;check_ip
DY MON DD HH24:MI:SS YYYY cron.err crond[12345]: USER root pid 1234 cmd . /lib/functions/modem.sh;modem_net_monitor
```

1. ターミナルを開いて SSH でルーターにログインします。

　`$ ssh root@{ip_address|host_name}`

2. 以下のコマンドを実行します。

　`$ uci set system.@system[0].cronloglevel="10"`

　`$ /etc/init.d/gl_timer restart`

3. 設定適用の為にルーターを再起動します。

4. 以下のような内容になっていることを確認します。

　`$ cat /etc/config/system`

```
config system
	option urandom_seed '0'
	option hostname 'GL-MT3000'
	option compat_version '1.0'
	option ttylogin '1'
	option timezone 'JST-9'
	option zonename 'Asia/Tokyo'
	option log_proto 'udp'
	option log_size '62500'
	option log_file '/tmp/system.log'
	option conloglevel '8'
	option cronloglevel '10'
```

5. ログが表示されなくなれば完了です。　

> 毎分出力されるログを解決する(通常の cron ログを表示しつつ、該当のログを非表示にする)

1. ターミナルを開いて SSH でルーターにログインします。

　`$ ssh root@{ip_address|host_name}`

2. 以下の箇所を編集します。(設定値に変更が無い場合は次の手順に移ります)

```
option conloglevel ''
option cronloglevel ''
```

　`$ vi /etc/config/system`

```
config system
	option urandom_seed '0'
	option hostname 'GL-MT3000'
	option compat_version '1.0'
	option ttylogin '1'
	option timezone 'JST-9'
	option zonename 'Asia/Tokyo'
	option log_proto 'udp'
	option log_size '62500'
	option log_file '/tmp/system.log'
	option conloglevel '4'
	option cronloglevel '5'
```

3. 続けて、以下のファイルの全行をコメントアウトします。

　`$ vi /tmp/gl_crontabs/root`

```
#* * * * * . /lib/functions/modem.sh;check_ip
#* * * * * sleep 30;. /lib/functions/modem.sh;check_ip
#*/2 * * * * . /lib/functions/modem.sh;modem_net_monitor
```

4. 以下のコマンドを実行します。

　`$ /etc/init.d/cron reload`

5. ログが表示されなくなれば完了です。

## Management Rules

> 本リポジトリでの作業を開始する前に

1. ブランチを切る  
2. コミットはファイル毎に行う *コミットメッセージ命名規則を参照
3. 毎週日曜日にプルリクエストを作成する  
4. Squash and merge で main ブランチに適用する
5. Delete branch でブランチを削除する

> コミットメッセージ命名規則

| 実施内容 | メッセージ |
| :---- | :---- |
| 新規作成 | Create |
| 変更 | Update |
| 削除 | Delete |
| 競合の解決 | Conflict_Resolution |

コミットメッセージ構文： `[YYYY-mm-dd] {メッセージ} {ファイル名}`

> PR用ブランチ／PRタイトル命名規則

| 月数 | month |
| :---- | :---- |
| 1月 | January |
| 2月 | February |
| 3月 | March |
| 4月 | April |
| 5月 | May |
| 6月 | June |
| 7月 | July |
| 8月 | August |
| 9月 | September |
| 10月 | October |
| 11月 | November |
| 12月 | December |

PR用ブランチ構文： `release_{month}_week_{1-5}`

PRタイトル構文： `{month} Week {1-5} Release`

## SSH to Github

> SSHキーの新規取得

1. ターミナルを開いて以下のコマンドを実行します。 *`email@example.com` の部分は自身のメールアドレスに変換して下さい。

　`$ ssh-keygen -t ed25519 -C "email@example.com"`

2. "Enter file in which to save the key" というメッセージが表示されたら Enter キーを押して既定のファイルの場所を使用します。

```
Generating public/private ed25519 key pair.
Enter file in which to save the key (/Users/user/.ssh/id_ed25519): [Press Enter]
```

3. 以下のプロンプトが表示されたらセキュアなパスフレーズを入力します。 *ここで入力したパスフレーズはメモ等に控えて下さい。

```
Enter passphrase (empty for no passphrase): [Type a passphrase]
Enter same passphrase again: [Type a passphrase]
```

4. SSHキーが発行されます。 *下記の値やランダムアートは一例です

```
Your identification has been saved in /Users/user/.ssh/id_ed25519
Your public key has been saved in /Users/user/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:aBCdE1f2ghIjklm3no4PQ5sT6uVwxYZa78bCD90EfGh email@example.com
The key's randomart image is:
+--[ED25519 256]--+
|　　　　　　　　　    　　　　　　　　　    |
|　　　　　　　　　    　　　　　　　　　    |
|　　　　　　　　　    　　　　　　　　　    |
|　　　　　　　　　    　　　　　　　　　    |
|　　　　　　　　　    　　　　　　　　　    |
|　　　　　　　　　    　　　　　　　　　    |
|　　　　　　　　　    　　　　　　　　　    |
|　　　　　　　　　    　　　　　　　　　    |
|　　　　　　　　　    　　　　　　　　　    |
+----[SHA256]-----+
```

5. 手順4で示されたファイルパスを含むコマンドを入力して、以下のような SSH キーを取得します。 *下記は一例です

　`$ cat /Users/user/.ssh/id_ed25519.pub`

```
ssh-ed25519 AAAAC1AabC1cABC1D5DEGHIJKLMopqrStuv2WxY3Z4abCDefg+H5I6JKlmNOPQr7St8u email@example.com
```

6. https://github.com/settings/ssh/new に移動します。

7. "Add new SSH Key" にて以下の設定を実施して "Add SSH key" をクリックします。

```
Title:　任意の　Key_Name
Key type: Authentication Key
Key：　手順5で取得した SSH キーを貼り付け
```

8. `You have successfully added the key '[Key_Name]'.` と表示されたら SSH を使った CLI 操作が可能になります。

---

> CLIを使ったリポジトリのクローン

1. GitHub.com でリポジトリのメインページへ移動します。

2. ファイルの一覧の上にある "＜＞ Code" をクリックします。

3. SSH キーを使ってリポジトリをクローンします。"SSH" をクリックしてリポジトリの URL をコピーします。

4. ターミナルを開いて、カレントディレクトリをクローンしたい場所に変更します。

　`$ cd [Repository_Name]`

5. "git clone" と入力し、手順3でコピーした URL を貼り付けます。

　`$ git clone git@github.com:[Account_Name]/[Repository_Name].git`

6. 初回接続、又は known_hosts に必要な情報が存在しない場合は以下のプロンプトが出力されます。接続するために "yes" を入力します。 *known_hosts ファイルに接続先サーバのIPアドレスやホスト名、公開鍵を保存します

```
Cloning into 'expansion_NW'...
The authenticity of host 'github.com (20.27.177.113)' can't be established.
ED25519 key fingerprint is SHA256:+AbC1defG2HiJklmnOpqR/sTUV3wXYZaBcde4GhIKlL.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
```

7. パスフレーズの入力を求められるので SSH キー生成時に登録したパスフレーズを入力します。

```
Warning: Permanently added 'github.com' (ED25519) to the list of known hosts.
Enter passphrase for key '/Users/rum/.ssh/id_ed25519':
```

8. 以下のようなプロセスが開始・終了したらリポジトリのクローンは成功です。 *下記は一例です

```
remote: Enumerating objects: 28, done.
remote: Counting objects: 100% (28/28), done.
remote: Compressing objects: 100% (23/23), done.
remote: Total 28 (delta 8), reused 18 (delta 3), pack-reused 0
Receiving objects: 100% (28/28), 6.12 MiB | 662.00 KiB/s, done.
Resolving deltas: 100% (8/8), done.
Updating files: 100% (10/10), done.
```

---

> リポジトリ内での追加・変更、プルリクエスト作成からマージまでの手順

1. ブランチを作成・切り替えます。

　`$ git checkout -b release_[month]_week_[1-5]`

　*ブランチを削除する場合　`$ git branch -d release_[month]_week_[1-5]`

　*ブランチ名を変更する場合　`$ git branch -m release_[month]_week_[1-5]`

2. git pull 実行前に設定します。(デフォルトの pull の挙動にする)

　`$ git config pull.rebase false`

3. リモートから main ブランチへの変更を取得します。

　`$ git pull origin main`

4. ローカルの変更を確認します。

　`$ git status`

5. リモートとローカルのファイルの差分を抽出します。

　`$ git diff`

6. ファイルの追加・変更をコミットします。 *コミットメッセージの命名規則について

　`$ git commit -m "[Commit_Message]"`

　*no changes added to commit (use "git add" and/or "git commit -a")と出た場合は "-am" でコミット

7. ローカルの変更を確認します。

　`$ git status`

8. プルリクエスト用のブランチにプッシュします。

　`$ git push origin release_[month]_week_[1-5]`

9. https://github.com/[Account_Name]/[Repository_Name]/pulls に移動します。

10. "Compare & pull request" をクリックします。

11. プルリクエストのタイトルを「Release Week [1-5]」に設定して "Create pull request" をクリックします。

12. 問題なければマージ方法が書かれたボタンが使用可能になります。"Squash and merge" を選択してクリックします。

13. "Confirm squash and merge" をクリックしてマージを完了させます。 *タイトルが "Release Week [1-5] (#[1-99])"になっているので "(#[1-99])" を削除します。

14. "Pull request successfully merged and closed" と表示されるので、"Delete branch" をクリックしてブランチを削除します。

15. 最後に "ターミナル" を開いて既存のブランチを削除します。

　`$ git checkout main`

　`$ git branch -D release_[month]_week_[1-5]`