![AdGuard_Filter Version](https://img.shields.io/badge/AdGuard_Filter-v6.4.9-blue?style=flat)
![Release Date](https://img.shields.io/badge/Release_Date-June_8_2025-green?style=flat)
![GitHub repo size](https://img.shields.io/github/repo-size/virtualitypage/expansion_NW)

> Table of Contents
- [AdGuard Filter](#adguard-filter)
- [Document - Expansion](#document---expansion)
  - [Send e-mail from the router](#ルーターからメール送信を行う)
  - [Send e-mail from the router - Load a file for e-mail and send it by e-mail](#ルーターからメール送信を行う---メール用のファイルを読み込んでメール送信)
  - [Send e-mail from the router - Insert attached file](#ルーターからメール送信を行う---添付ファイルを挿入)
  - [Download files from a private Github repository](#github-のプライベートリポジトリからファイルをダウンロードする)
  - [Use WakeOnLAN to activate devices in the network](#Wake-On-LAN-を使ってネットワーク内の装置を起動する)
  - [How to add a cache invalidation setting for internal web sites](#内部-Web-サイトにおけるキャッシュ無効化設定の追加方法)
  - [Computer to computer file transfer using FTP](#FTP-を使用したコンピュータ間ファイル転送)
- [Document - Troubleshooting](#document---troubleshooting)
  - [Hide irregular log output](#不規則なログ出力を非表示にする)
  - [Hide irregular log output - Displays normal cron logs and hides target logs](#不規則なログ出力を非表示にする---通常の-cron-ログを表示しつつ対象のログを非表示にする)
  - [Unable to connect to the Internet with GL-MT3000](#gl-mt3000-でインターネットに接続できない)
  - [Undo the previous commit](#直前のコミットを取り消す)
  - [Unable to apply previous commit undo to remote repository](#直前のコミット取り消しをリモートリポジトリに適用できない)
  - [Authentication error occurs in "Push origin" in "GitHub Desktop.app"](#github-desktop-アプリの-push-origin-にて認証エラーが発生する)
  - [Authentication errors occur with "git clone", "git push origin main", etc.](#git-clone-や-git-push-origin-main-等で認証エラーが発生する)
  - [The GL-MT3000 administration screen does not appear](gl-mt3000-の-admin-panel-が表示されない)
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

> AdGuard Home の「クエリ・ログ」取得設定手順 ~~(再起動しなければクエリ・ログを取得できないため非推奨)~~

* 参照元：https://github.com/AdguardTeam/AdGuardHome/wiki/Configuration#configuration-file

1. ターミナルを開いて SSH でルーターにログインします。

　`$ ssh root@{ip_address|host_name}`

2. 以下の箇所を編集します。

・クエリ・ログ設定

```
dir_path:            // クエリ・ログファイルを保存するためのカスタムディレクトリ
file_enabled: false  // クエリ・ログファイル保存設定
```

・ログ設定

```
file:              // ログファイルへのフルパス
max_age: 3         // ログファイルがローテーションされるまでの最大日数
local_time: false  // タイムスタンプのフォーマット設定
```

　`$ vi /etc/AdGuardHome/config.yaml`

```
querylog:
  dir_path: /etc/AdGuardHome/data
  ignored: []
  interval: 2160h
  size_memory: 1000
  enabled: true
  file_enabled: true // false から true に変更
```

```
log:
  file: /etc/AdGuardHome/data/querylog.json
  max_backups: 0
  max_size: 100
  max_age: 7 // 3 から 7 に変更
  compress: false
  local_time: true // false から true に変更
  verbose: false
```

3. 設定適用の為にルーターを再起動します。

4. 実行ファイルにログファイルのフルパスを渡します。ここで指定したファイル名(ここでは"querylog.json")の json ファイルが作成され、クエリ・ログが書き込まれます。

　`$ AdGuardHome --logfile /etc/AdGuardHome/data/querylog.json`

5. 以下のような内容になっていることを確認します。

　`$ cat /etc/AdGuardHome/querylog.json`

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

* GUI 上でアプリ "AdGuard Home" を再起動(停止・起動)すると、以下の内容が追加されて初期化されるので注意

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

* 設定適用により http://192.168.8.1:3000/#logs のクエリ・ログも指定期間保存されている。

以下、検証メモ(2024/10/11)
>
> 本設定の第二検証時、ルーター再起動前に以下コマンドを実行している。コマンド入力の必要性は今後調査予定
>
>　`$ AdGuardHome --logfile /etc/AdGuardHome/data/querylog.json` * プロンプトが表示されなくなった為、Control+Cで強制終了
>
>　`$ cat /etc/AdGuardHome/querylog.json` * 手順 5 と同様の内容、しかし強制終了した為それに関する情報が追記されていた
>
>　`$ AdGuardHome --service stop`
>
>　`$ AdGuardHome --service start`
>
>　`$ AdGuardHome --service restart`
>
>動作確認のため GUI 上でアプリの再起動(停止・起動)を行うと AdGuardHome が初期化されてしまった。その際、不要となった querylog.json は再起動前に破棄。
>
>そして、検証失敗として切り戻し(再起動を伴うリストア)を実施。
>
>しかし、上記設定はバックアップ対象外だった為、再起動により設定が適用されてクエリ・ログ取得処理が開始された。
>
>以上の事から再検証時に以下の実施を行うこと
>
>・設定前にバックアップを取得する("/etc/AdGuardHome"配下は対象外)
>
>・AdGuardHome が初期化される別の動作とその対策方法、要因等の調査
>
>・必要に応じて手順の修正を実施
>
>・AdGuardHome コマンド入力の必要性について調査

## Document - Expansion

### ルーターからメール送信を行う

1. ターミナルを開いて SSH でルーターにログインします。

　`$ ssh root@{ip_address|host_name}`

2. 以下のコマンドを実行します。

　`$ opkg update`

　`$ opkg install msmtp`

3. Google アカウント上でアプリ パスワードを取得します。(手順3の "password" で使用します)

* [アプリ パスワードを作成、管理する](https://myaccount.google.com/apppasswords) をクリックします。(本人確認画面に遷移します)

* アプリ名入力欄に "msmtp" と入力して「作成」をクリックします。表示された 16 桁のアプリ パスワードをメモ等に控えて下さい。

4. "msmtprc" ファイルを以下のように設定します。 *`your-email@gmail.com` の部分は自身の Google アカウントのメールアドレスを入力して下さい。

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

5. 設定が完了したら任意のメールアドレスにテストメールを送信します。 *`user@example.com` の部分は宛先メールアドレスを入力して下さい。

　`$ echo "This is a test email." | msmtp -a gmail user@example.com`

6. プロンプト上に何も表示されなければ完了です。

---

### ルーターからメール送信を行う - メール用のファイルを読み込んでメール送信

1. ターミナルを開いて SSH でルーターにログインします。

　`$ ssh root@{ip_address|host_name}`

2. メール用のファイル "test.mail" を作成します。

　`$ vi /etc/test.mail`

```
to: your-email@gmail.com
From: user@example.com
Subject: Test mail

This is a test email.
```

3. ターミナルで以下を実行します。 *`user@example.com` の部分は宛先メールアドレスを入力して下さい。

　`$ msmtp user@example.com < /etc/test.mail`

4. プロンプト上に何も表示されなければ完了です。

---

### ルーターからメール送信を行う - 添付ファイルを挿入

1. ターミナルを開いて SSH でルーターにログインします。

　`$ ssh root@{ip_address|host_name}`

2. 以下のコマンドを実行します。

　`$ opkg update`

　`$ opkg install mutt`

　`$ opkg install msmtp-mta`

3. "mutt" ファイルを以下のように設定します。 *`your-email@gmail.com` の部分は自身の Google アカウントのメールアドレスを入力して下さい。

```
set sendmail="/etc/msmtp"
set use_from=yes
set realname="root"
set from=your-email@gmail.com
```

4. ターミナルで以下を実行します。(-a で添付ファイルのパスを渡します) *`user@example.com` の部分は宛先メールアドレスを入力して下さい。

　`$ echo "This is a test email." | mutt -s "Test mail" -a /tmp/test.txt -- user@example.com`

* または、以下のようにファイルの内容をメールの本文とした上で実行します。

　`$ vi /etc/test.mail`

```
This is a test email.
```

　`$ mutt -s "Test mail" -a /tmp/test.txt -- user@example.com < /etc/test.mail`

5. プロンプト上に何も表示されなければ完了です。

---

### Github のプライベートリポジトリからファイルをダウンロードする

参照元：https://stackoverflow.com/questions/53083479/wget-a-raw-file-from-github-from-a-private-repo

1. [github.com](https://github.com/login) にログインして [New personal access token (classic)](https://github.com/settings/tokens/new) に移動します。

2. "Note" に、パーソナルアクセストークンに使用する任意の名前を入れて下さい。

3. "Select scopes" の "repo" にチェックを入れて画面下部の "Generate token" をクリックします。

4. 発行されたパーソナルアクセストークンをコピーしてメモ等に控えて下さい。

5. 対象のファイルがあるプライベートリポジトリに移動します。

6. 対象のファイルをクリックした後、画面右側にある "Raw" をクリックして URL をコピーします。

7. ターミナルを開いて SSH でルーターにログインします。

　`$ ssh root@{ip_address|host_name}`

8. 以下のコマンドを実行します。

* `PERSONAL_ACCESS_TOKEN`は手順3で保存したものを使用、`absolute_path` は保存先の絶対パス。

* `https://` に続く URL は手順6で取得したものに変換して下さい。

```
$ curl -H 'Authorization: token PERSONAL_ACCESS_TOKEN' -L -o [absolute_path] https://raw.githubusercontent.com/[repoOwner]/[repoName]/main/[folder]/[filename]
```

9. 以下のように出力されたら完了です。 *下記は一例です

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 28976  100 28976    0     0  63498      0 --:--:-- --:--:-- --:--:-- 63964
```

#### ・画像や PDF ファイルをダウンロードする場合

1. 以下のコマンドを実行します。

* `PERSONAL_ACCESS_TOKEN`は前項の手順3で保存したものを使用、`absolute_path` は保存先の絶対パス。

* `https://` に続く URL は対象の画像や PDF ファイルをクリックして取得したものに変換して下さい。

```
$ curl -H "Authorization: token PERSONAL_ACCESS_TOKEN" -H "Accept: application/vnd.github.v3.raw" -L "https://api.github.com/repos/[repoOwner]/[repoName]/contents/[folder]/[filename]" -o [absolute_path]
```

2. 以下のように出力されたら完了です。 *下記は一例です

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 8279k  100 8279k    0     0  4170k      0  0:00:01  0:00:01 --:--:-- 4168k
```

* 以下のコマンドを実行することで、ダウンロードしたファイルの形式を確認することが出来ます。 *ファイルの種類毎に出力結果は異なります

　`$ file [filename].png`

```
[filename].png: PNG image data, 3360 x 1894, 8-bit/color RGBA, non-interlaced
```

#### ・対象のフォルダから複数のファイルをダウンロードする場合

1. ターミナルを開いて SSH でルーターにログインします。

　`$ ssh root@{ip_address|host_name}`

2. ダウンロードするファイルのURLリストを以下のように設定します。 *`https://` に続く URL は前項の手順6で取得したものに変換して下さい。

　`$ vi url_list.txt`

```
https://raw.githubusercontent.com/[repoOwner]/[repoName]/main/[folder]/[filename_1]
https://raw.githubusercontent.com/[repoOwner]/[repoName]/main/[folder]/[filename_2]
https://raw.githubusercontent.com/[repoOwner]/[repoName]/main/[folder]/[filename_3]
```

3. 以下のシェルスクリプトを作成します。 * `PERSONAL_ACCESS_TOKEN`は前項の手順3で保存したものを使用

* 変数 `dest_dir` `auth_token` の値を変更すること

　`$ vi github_private_download.sh`

```
#!/bin/bash

dest_dir="path/to/dest_dir" # 保存先ディレクトリ
url_list="url_list.txt" # URLリストのファイル
auth_token="PERSONAL_ACCESS_TOKEN" # パーソナルアクセストークン

while IFS= read -r url; do
  filename=$(basename "$url")
  curl -H "Authorization: token $auth_token" -L -o "$dest_dir/$filename" "$url"
done < "$url_list"
```

4. 作成したシェルスクリプトに実行権限を付与します。

　`$ chmod +x github_private_download.sh`

5. 作成したシェルスクリプトを実行します。

　`$ ash github_private_download.sh`

6. 以下のように出力されたら完了です。 *下記は一例です

```
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 28976  100 28976    0     0  63498      0 --:--:-- --:--:-- --:--:-- 63964
```

---

### Wake On LAN を使ってネットワーク内の装置を起動する

1. ターミナルを開いて SSH でルーターにログインします。

　`$ ssh root@{ip_address|host_name}`

2. 以下のコマンドを実行します。

　`$ opkg update`

　`$ opkg install wakeonlan`

3. DHCPサーバーのリース情報を表示します。

　`$ cat /tmp/dhcp.leases`

```
0123456789 00:11:2c:3d:4e:5f 192.168.8.100 host_name_1 01:1b:2c:3d:4e:5f:6a
1234567890 11:22:3c:4d:5e:6f 192.168.8.101 host_name_2 01:2b:3c:4d:5e:6f:7a
2345678901 22:33:4c:5d:6e:7f 192.168.8.102 host_name_3 01:3b:4c:5d:6e:7f:8a
3456789012 33:44:5c:6d:7e:8f 192.168.8.103 host_name_4 01:4b:5c:6d:7e:8f:9a
4567890123 44:55:6c:7d:8e:9f 192.168.8.104 host_name_5 01:5b:6c:7d:8e:9f:0a
```

4. 手順3で取得した対象の情報を入力して、以下のコマンドを実行します。

* `Private_IP`は手順3で表示されたものを使用、`MAC_Address` は2列目にある16進数。

　`$ wakeonlan -i [Private_IP] [MAC_Address]`

```
Sending magic packet to [Private_IP]:9 with [MAC_Address]
```

5. 接続元の機器で、対象の装置が起動しているか確認します。 * `Tailscale_IP` とは Wireguard 接続管理サービス "Tailscale" で使用される IP アドレス

　`$ ping {Private_IP|Tailscale_IP}`

6. 以下のようなプロセスが開始・終了したら成功です。 *下記は一例です

```
PING 192.168.8.100 (192.168.8.100): 56 data bytes
64 bytes from 192.168.8.100: icmp_seq=0 ttl=64 time=225.876 ms
64 bytes from 192.168.8.100: icmp_seq=1 ttl=64 time=337.344 ms
64 bytes from 192.168.8.100: icmp_seq=2 ttl=64 time=30.910 ms
64 bytes from 192.168.8.100: icmp_seq=3 ttl=64 time=69.797 ms
64 bytes from 192.168.8.100: icmp_seq=4 ttl=64 time=78.320 ms

--- 192.168.8.100 ping statistics ---
5 packets transmitted, 5 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 30.910/148.449/337.344/115.409 ms
```

---

### 内部 Web サイトにおけるキャッシュ無効化設定の追加方法

1. ターミナルを開いて SSH でルーターにログインします。

　`$ ssh root@{ip_address|host_name}`

2. "nginx.conf" ファイルを以下のように設定します。 * `# サーバーのキャッシュ制御設定を追加` の部分を追加

　`$ vi /etc/nginx/nginx.conf`

```
user  root;
worker_processes  auto;

pid /var/run/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile on;
    keepalive_timeout 5;

    client_body_buffer_size 10K;
    client_header_buffer_size 1k;
    client_max_body_size 1G;
    large_client_header_buffers 2 2k;

    gzip_static on;

    root /www;

    access_log off;

    include /etc/nginx/conf.d/*.conf;

    # サーバーのキャッシュ制御設定を追加
    server {
        listen 80;
        server_name localhost;

        location / {
            root /www;

            # キャッシュ制御設定
            add_header Cache-Control "no-cache, no-store, must-revalidate";
            add_header Pragma "no-cache";
            add_header Expires "0";
        }
    }
}
```

3. 追加した設定が正しく動作するかをテストします。

　`$ nginx -t`

4. 以下のように出力されたら設定完了です。

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

5. nginx を再起動します。

　`$ /etc/init.d/nginx restart`

6. JavaScript やその他リソースを更新した際、ブラウザ上で表示されるページが最新のものになっていることを確認して下さい。

---

### FTP を使用したコンピュータ間ファイル転送

1. ターミナルを開いて SSH でルーターにログインします。

　`$ ssh root@{ip_address|host_name}`

2. 以下のコマンドを実行します。

　`$ opkg update`

　`$ opkg install vsftpd`

3. 以下のコマンドを実行して `"listen=YES"` を追記します。

　`$ vi /etc/vsftpd.conf`

```
#Initialized
background=YES
#listen=NO // コメントアウト
listen=YES // 追記
```

4. 以下のコマンドを実行して FTP サーバを起動します。

　`$ /etc/init.d/vsftpd reload`

　`$ /etc/init.d/vsftpd start`

5. FTP ソリューション [FileZilla](https://filezilla-project.org/download.php?type=client) で以下の設定を行い、FTP サーバー(GL-MT3000)に接続します。 * 使用するには操作端末へのインストールが必要

```
プロトコル: FTP - ファイル転送プロトコル
ホスト: 192.168.8.100 // GL-MT3000 のIP アドレス
ポート: (ここは空欄)
暗号化: 使用可能なら明示的な FTP over TLS を使用
-------------------------------------------
ログオンタイプ: パスワードを尋ねる
ユーザー: user // 登録されているユーザー名
パスワード: (ここは空欄)
```

6. 接続時、以下のポップアップが表示されますが「OK」で接続を続行します。* 下記は一例です

```
このサーバーは FTP over TLS をサポートしていません。
続行すると、パスワードとファイルはインターネット経由でそのまま送信されます。

  ホスト: 192.168.8.100
  ポート: 21
```

7. FileZilla 上で、以下のようなメッセージが出力されたら完了です。 * 下記は一例です

```
状態:          	192.168.8.100:21 に接続中...
状態:         	接続を確立しました。ウェルカム メッセージを待っています...
状態:          	FTP over TLS がサポートされていないセキュアでないサーバーです
状態:          	ログインしました
状態:          	ディレクトリ リストを取得中...
状態:          	サーバーのタイムゾーン オフセットを計算しています...
状態:          	Timezone offset of server is 0 seconds.
```

8. FTP サーバを停止する場合は、以下のコマンドを入力します。

　`$ /etc/init.d/vsftpd stop`

## Document - Troubleshooting

### 不規則なログ出力を非表示にする

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
	option log_file '/tmp/system.log'
	option conloglevel '8'
	option log_size '62500'
	option cronloglevel '10'
```

5. ログが表示されなくなれば完了です。

---

### 不規則なログ出力を非表示にする - 通常の cron ログを表示しつつ対象のログを非表示にする

> 再起動時、本設定がリセットされるため再設定が必要

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
	option log_file '/tmp/system.log'
	option conloglevel '4'
	option log_size '62500'
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

---

### GL-MT3000 でインターネットに接続できない

1. http://192.168.8.1/#/clients にて、対象のホストのプライベート IP アドレスが表示されているか確認します。 *表示されていない場合、 DHCP サーバに原因があると思われます。

2. ターミナルを開いて SSH でルーターにログインします。

　`$ ssh root@{ip_address|host_name}`

3. 以下のコマンドを実行して DHCP サーバーの静的リースを設定します。ここでは接続できないホストを登録して下さい。

　`$ vi /etc/config/dhcp`

```
config host
	option name 'host_name'
	option mac '00:11:22:33:44:55'
	option ip 'ip_address'
```

4. dnsmasq のログにて DHCP リースのリクエストとレスポンスが正しく行われているか確認します。 *表示されない場合、数分経過後に再度実行して下さい。

　`$ logread | grep dnsmasq-dhcp`

```
YYYY/MM/DD HH24:MI:SS.FF daemon.info dnsmasq-dhcp[12345]: DHCPDISCOVER(br-lan) ip_address 00:11:22:33:44:55
YYYY/MM/DD HH24:MI:SS.FF daemon.info dnsmasq-dhcp[12345]: DHCPOFFER(br-lan) ip_address 00:11:22:33:44:55
YYYY/MM/DD HH24:MI:SS.FF daemon.info dnsmasq-dhcp[12345]: DHCPREQUEST(br-lan) ip_address 00:11:22:33:44:55
YYYY/MM/DD HH24:MI:SS.FF daemon.info dnsmasq-dhcp[12345]: DHCPACK(br-lan) ip_address 00:11:22:33:44:55 host_name
```

5. DHCP サーバーを再起動します。

　`$ /etc/init.d/dnsmasq restart`

6. http://192.168.8.1/cgi-bin/luci/admin/network/dhcp に移動します。

7. "Network" > "Static Leases" の "Active DHCP Leases" に DHCP リースが登録されているか確認します。

* 以下のファイルでも DHCP リースの登録が確認できます。

　`$ cat /tmp/dhcp.leases`

```
1234567890 00:11:22:33:44:55 ip_address host_name 00:11:22:33:44:55
```

* DHCP サーバーを再起動するだけでもこの問題が解決した事例有り

　`$ /etc/init.d/dnsmasq restart`

　`$ logread | grep dnsmasq-dhcp`

```
YYYY/MM/DD HH24:MI:SS.FF daemon.info dnsmasq-dhcp[12345]: DHCPREQUEST(br-lan) ip_address 00:11:22:33:44:55
YYYY/MM/DD HH24:MI:SS.FF daemon.info dnsmasq-dhcp[12345]: DHCPACK(br-lan) ip_address 00:11:22:33:44:55 host_name
```

---

### 直前のコミットを取り消す

1. ターミナルを開いて、カレントディレクトリを作業ディレクトリに変更します。

　`$ cd [repoName]`

2. 以下のコマンドを実行して直前のコミットを取り消します。

　`$ git reset HEAD^`

`HEAD is now at 1a2b3c4 [YYYY-mm-dd] {メッセージ} {ファイル名}`

3. 直前のコミットが取り消されていることを確認します。

　`$ git log`

4. リモートリポジトリにローカルリポジトリの変更(直前のコミット取り消し)を適用します。

　`$ git push origin release_{month}_week_{1-5} --force`

```
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:[repoOwner]/[repoName].git
 + 9h0i1j2...5e6f7g8 release_{month}_week_{1-5} -> release_{month}_week_{1-5} (forced update)
```

---

### 直前のコミット取り消しをリモートリポジトリに適用できない

> リモートリポジトリにローカルリポジトリの変更(直前のコミット取り消し)適用時、以下のメッセージが出力される。

　`$ git push origin release_{month}_week_{1-5} --force`

```
Username for 'https://github.com': [repoOwner]
Password for 'https://[repoOwner]@github.com':
remote: Support for password authentication was removed on August 13, 2021.
remote: Please see https://docs.github.com/get-started/getting-started-with-git/about-remote-repositories#cloning-with-https-urls for information on currently recommended modes of authentication.
fatal: Authentication failed for 'https://github.com/[repoOwner]/[repoName].git/'
```

1. ターミナルを開いて GitHub への接続テストを行います。

　`$ ssh -T git@github.com`

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the RSA key sent by the remote host is
SHA256:aBCdE1f2ghIjklm3no4PQ5sT6uVwxYZa78bCD90EfGh.
Please contact your system administrator.
Add correct host key in /Users/user/.ssh/known_hosts to get rid of this message.
Offending RSA key in /Users/user/.ssh/known_hosts:1
Host key for github.com has changed and you have requested strict checking.
Host key verification failed.
```

2. "Offending RSA key in /Users/user/.ssh/known_hosts:1" で示された行を削除します。

　`$ vi /Users/user/.ssh/known_hosts`

3. [SSH to Github](#ssh-to-github) の手順1〜8までを実施します。

4. SSH 設定ファイルを以下のように設定します。

　`$ vi /Users/user/.ssh/config`

```
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519
```

5. リモート URL を SSH の URL に設定します。

　`$ git remote set-url origin [repoOwner]@github.com:[repoOwner]/[repoName].git`

6. リモート URL の設定が正しいか確認します。

　`$ git remote -v`

```
origin	git@github.com:[repoOwner]/[repoName].git (fetch)
origin	git@github.com:[repoOwner]/[repoName].git (push)
```

7. 再度 GitHub への接続テストを行います。

　`$ ssh -T git@github.com`

```
The authenticity of host 'github.com (20.27.177.113)' can't be established.
ED25519 key fingerprint is SHA256:+AbC1defG2HiJklmnOpqR/sTUV3wXYZaBcde4GhIKlL.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'github.com' (ED25519) to the list of known hosts.
Hi [repoOwner]! You've successfully authenticated, but GitHub does not provide shell access.
```

8. リモートリポジトリにローカルリポジトリの変更(直前のコミット取り消し)を適用します。

　`$ git push origin release_{month}_week_{1-5} --force`

```
Total 0 (delta 0), reused 0 (delta 0), pack-reused 0
To github.com:[repoOwner]/[repoName].git
 + 9h0i1j2...5e6f7g8 release_{month}_week_{1-5} -> release_{month}_week_{1-5} (forced update)
```

---

### "GitHub Desktop" アプリの "Push origin" にて認証エラーが発生する

```
Authentication failed. Some common reasons include:

- You are not logged in to your account: see GitHub Desktop > Settings.
- You may need to log out and log back in to refresh your token.
- You do not have permission to access this repository.
- The repository is archived on GitHub. Check the repository settings to confirm you are still permitted to push commits.
- If you use SSH authentication, check that your key is added to the ssh-agent and associated with your account.
- If you use SSH authentication, ensure the host key verification passes for your repository hosting service.
- If you used username / password authentication, you might need to use a Personal Access Token instead of your account password. Check the documentation of your repository hosting service
```

> "id_ed25519" と "id_ed25519.pub" を変更した場合に発生します。

```
$ ssh -T git@github.com
git@github.com: Permission denied (publickey).
```

1. ターミナルを開いて "known_hosts" ファイル内の "Github" を含む行を削除します。

　`$ vi /Users/user/.ssh/known_hosts`

2. "id_ed25519" と "id_ed25519.pub" を最新のものに置き換えます。

　`$ vi /Users/user/.ssh/id_ed25519`

　`$ vi /Users/user/.ssh/id_ed25519.pub`

3. GitHub への接続テストを行います。

　`$ ssh -T git@github.com`

```
The authenticity of host 'github.com (20.27.177.113)' can't be established.
ED25519 key fingerprint is SHA256:+AbC1defG2HiJklmnOpqR/sTUV3wXYZaBcde4GhIKlL.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'github.com' (ED25519) to the list of known hosts.
Hi [repoOwner]! You've successfully authenticated, but GitHub does not provide shell access.
```

4. 上記のように表示されたら "GitHub Desktop.app" で "Push" が可能になります。

---

### "git clone" や "git push origin main" 等で認証エラーが発生する

　`$ git clone https://github.com/[repoOwner]/[repoName].git`

```
Cloning into '[repoName]'...
Username for 'https://github.com': [repoOwner]
Password for 'https://[repoName]@github.com': 
remote: Support for password authentication was removed on August 13, 2021.
remote: Please see https://docs.github.com/get-started/getting-started-with-git/about-remote-repositories#cloning-with-https-urls for information on currently recommended modes of authentication.
fatal: Authentication failed for 'https://github.com/[repoOwner]/[repoName].git/'
```

* `Password for 'https://[repoName]@github.com': ` 行にパーソナルアクセストークンが入力されていない、かつリモートURLが以下のようになっている場合に発生します。

```
origin	https://github.com/[repoOwner]/[repoName].git (fetch)
origin	https://github.com/[repoOwner]/[repoName].git (push)
```

* パーソナルアクセストークンを使用しない場合、リモート URL を SSH の URL に設定することで解決します。

　`$ git remote set-url origin [repoOwner]@github.com:[repoOwner]/[repoName].git`

> パーソナルアクセストークンを使用する場合

1. [GitHub のプライベートリポジトリからファイルをダウンロードする](#github-のプライベートリポジトリからファイルをダウンロードする) の手順1〜4を実施します。

2. 以下のコマンドを実行して `Password for 'https://[repoName]@github.com': ` 行にパーソナルアクセストークンをペーストします。

　`$ git clone https://github.com/[repoOwner]/[repoName].git`

```
Cloning into '[repoName]'...
Username for 'https://github.com': [repoOwner]
Password for 'https://[repoName]@github.com': 
```

3. 以下のようなプロセスが開始・終了したら成功です。 *下記は一例です

```
remote: Enumerating objects: 10, done.
remote: Counting objects: 100% (10/10), done.
remote: Compressing objects: 100% (7/7), done.
remote: Total 10 (delta 2), reused 10 (delta 2), pack-reused 0
Receiving objects: 100% (10/10), 36.33 KiB | 6.05 MiB/s, done.
Resolving deltas: 100% (2/2), done.
```

---

### GL-MT3000 の Admin Panel が表示されない

> 事前調査
>
> 1. http://192.168.8.1/#/login にて右クリックで"要素の詳細を表示"、もしくは"検証"でモードに移行。
>
> 2. "ネットワーク"で赤く表示されているファイル（.js）をクリック。
>
> 3. "ヘッダ"にてステータスを確認。
>
> * Safari
>
> ```
> ステータス: —
> ```
>
> * Google Chrome
>
> ```
> ステータスコード: 404 Not Found
> ```
>
> このようになっている場合、ファイルが存在しないため Admin Panel を表示することが出来ない。

1. ターミナルを開いて SSH でルーターにログインします。

　`$ ssh root@{ip_address|host_name}`

2. 参照元である js ディレクトリを表示、先頭に `app.` が付くものを探します。

　`$ ls -l /www/js`

3. app.1a2b3456.js.gz を解凍します。 *ファイル名は一例です `-d` で解凍、`-k` で解凍前のファイルを残す

　`$ gzip -dk /www/js/app.1a2b3456.js.gz`

4. Admin Panel を表示する html ファイルを編集します。 * 以下の箇所を新しい js ファイル名に変更

　`$ vi /www/gl_home.html`

```
<link href="/js/app.1a2b3456.js" rel="preload" as="script">
<script src="/js/app.1a2b3456.js"></script>
```

5. http://192.168.8.1/#/login にて Admin Panel が表示されたら完了です。

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

> PR 用ブランチ／PR タイトル命名規則

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

PR 用ブランチ構文： `release_{month}_week_{1-5}`

PR タイトル構文： `{month} Week {1-5} Release`

## SSH to Github

> SSH キーの新規取得

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

4. SSH キーが発行されます。 *下記の値やランダムアートは一例です

```
Your identification has been saved in /Users/user/.ssh/id_ed25519
Your public key has been saved in /Users/user/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:aBCdE1f2ghIjklm3no4PQ5sT6uVwxYZa78bCD90EfGh email@example.com
The key's randomart image is:
+--[ED25519 256]--+
|　　　　　　　　　  |
|　　　　　　　　　  |
|　　　　　　　　　  |
|　　　　　　　　　  |
|　　　　　　　　　  |
|　　　　　　　　　  |
|　　　　　　　　　  |
|　　　　　　　　　  |
|　　　　　　　　　  |
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

> CLI を使ったリポジトリのクローン

1. GitHub.com でリポジトリのメインページへ移動します。

2. ファイルの一覧の上にある "＜＞ Code" をクリックします。

3. SSH キーを使ってリポジトリをクローンします。"SSH" をクリックしてリポジトリの URL をコピーします。

4. ターミナルを開いて、カレントディレクトリをクローンしたい場所に変更します。

　`$ cd [Repository_Name]`

5. "git clone" と入力し、手順3でコピーした URL を貼り付けます。

　`$ git clone git@github.com:[repoOwner]/[repoName].git`

6. 初回接続、又は known_hosts に必要な情報が存在しない場合は以下のプロンプトが出力されます。接続するために "yes" を入力します。 *known_hosts ファイルに接続先サーバの IP アドレスやホスト名、公開鍵を保存します

```
Cloning into '[Repository_Name]'...
The authenticity of host 'github.com (20.27.177.113)' can't be established.
ED25519 key fingerprint is SHA256:+AbC1defG2HiJklmnOpqR/sTUV3wXYZaBcde4GhIKlL.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
```

7. パスフレーズの入力を求められるので SSH キー生成時に登録したパスフレーズを入力します。

```
Warning: Permanently added 'github.com' (ED25519) to the list of known hosts.
Enter passphrase for key '/Users/user/.ssh/id_ed25519':
```

8. 以下のようなプロセスが開始・終了したらリポジトリのクローンは成功です。 *下記は一例です

```
remote: Enumerating objects: 10, done.
remote: Counting objects: 100% (10/10), done.
remote: Compressing objects: 100% (7/7), done.
remote: Total 10 (delta 2), reused 10 (delta 2), pack-reused 0
Receiving objects: 100% (10/10), 36.33 KiB | 6.05 MiB/s, done.
Resolving deltas: 100% (2/2), done.
```

---

> リポジトリ内での追加・変更、プルリクエスト作成からマージまでの手順

1. ブランチを作成・切り替えます。

　`$ git checkout -b release_[month]_week_[1-5]`

* ブランチを削除する場合　`$ git branch -d release_[month]_week_[1-5]`

* ブランチ名を変更する場合　`$ git branch -m release_[month]_week_[1-5]`

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

* no changes added to commit (use "git add" and/or "git commit -a")と出た場合は "-am" でコミット

7. ローカルの変更を確認します。

　`$ git status`

8. プルリクエスト用のブランチにプッシュします。

　`$ git push origin release_[month]_week_[1-5]`

9. https://github.com/[repoOwner]/[repoName]/pulls に移動します。

10. "Compare & pull request" をクリックします。

11. プルリクエストのタイトルを「Release Week [1-5]」に設定して "Create pull request" をクリックします。

12. 問題なければマージ方法が書かれたボタンが使用可能になります。"Squash and merge" を選択してクリックします。

13. "Confirm squash and merge" をクリックしてマージを完了させます。 *タイトルが "Release Week [1-5] (#[1-99])"になっているので "(#[1-99])" を削除します。

14. "Pull request successfully merged and closed" と表示されるので、"Delete branch" をクリックしてブランチを削除します。

15. 最後に "ターミナル" を開いて既存のブランチを削除します。

　`$ git checkout main`

　`$ git branch -D release_[month]_week_[1-5]`