mastodon-ansible-playbook
=========================

## このプレイブックの目的

* 多くのユーザに使っていただくためのサーバ構築ではなく、少人数で使うとか検証用のためのサーバ構築を想定しています。
* とりあえず動く状態を作っているだけで、チューニングは一切していません。
* pull requestは基本受け付けてないので送らないでください。自分の必要に応じて書いているだけで、汎用性を高めて複雑にはしたくないからです。
* forkしてお好みの改変を加えて使っていただくと良いと思います。

## 前提条件

### さくらのVPS

- Ubuntu 16.04 amd64をインストールしてください。
- カスタムインストールを使ってください。標準OSインストールだとファイアウォールの設定が違うらしくてlet's encrpytの証明書インストールがうまく行かなかったです。
- インストール時に管理用のユーザ名はubuntuにしてください。
- 利用するサーバのホスト名のAレコードをDNSで設定しておいてください。
- 一度sshで接続して `Are you sure you want to continue connecting (yes/no)?` に `yes` と回答して `~/.ssh/known_hosts` にエントリが追加された状態にしてください (`yes` と回答さえすればログインはしなくてよいです)。

### Ansible実行環境を準備

- Ubuntu 16.04 (xenial)
- Bash on Ubuntu on Windows

```
sudo apt install git python3-venv build-essential python3-dev openssl-dev libssl-dev sshpass
git clone https://github.com/hnakamur/mastodon-ansible-playbook
python3 -m venv venv
source venv/bin/activate
pip install --upgrade pip
pip install ansible
```

## 環境に応じてAnsibleプレイブックを調整

* `hosts` ファイル
   - ホスト名と `public_ip` でグローバルIPアドレス  
* `group_vars/vps/vars.yml` ファイル
   - `authorized_key_key` : リモートサーバの鍵認証で使う鍵。公開鍵のファイル名が違う場合は調整してください。
   - `nginx_mastodon_server_name` : mastodonサーバのホスト名。
   - `lets_encrypt_domains` : let's encryptで取得する証明書のドメイン名。mastodon以外も同居する場合は複数設定可能です。
   - `lets_encrypt_admin_email` : let's encrypt に登録する管理者のメールアドレス。
   - `smtp_*` : smtp関連の設定。現在はGmail用の設定を書いています。Gmailを使う場合はメールアドレスだけ書き換えればOKです。
   - `ntp_servers` : ntpサーバ名。
* `passwords.ini.sample` を `password.ini` にコピーしてパスワードを変更する
   - `db_password` : PostgreSQLのmastodonユーザのパスワード。
   - `smtp_password` : smtpサーバに接続するときのパスワード。Gmailで2段階認証を使っている場合は[アプリ パスワードでログイン - Gmail ヘルプ](https://support.google.com/mail/answer/185833?hl=ja)を参照してアプリパスワードを生成して指定してください。

## Ansibleを実行

```
source venv/bin/activate
```

を実行してvirtualenv環境を実行した状態で以下のコマンドを実行してください。


### 環境構築準備 (初回のみ実行)

```
ansible-playbook vps-firsttime.yml -v -D -K -k
```

- ubuntu ユーザで接続します。
- ubuntu ユーザの sshパスワードとSUDOパスワードを入力するプロンプトが出ますので入力してください。
- ubuntu ユーザと root ユーザにsshの公開鍵を設置します。
- 設置後はどちらのユーザもパスワード認証は禁止して、公開鍵認証のみ許可する状態になります。

### アプリケーション環境構築

ssh-agentで鍵のパスフレーズをあらかじめ入力しておきます。
秘密鍵のパスは環境に応じて適宜変更してください。

```
eval `ssh-agent`
ssh-add ~/.ssh/id_rsa
```

```
ansible-playbook vps.yml -v -D
```

- root ユーザで鍵認証で接続します。
- 途中sudoでpostgresユーザやmastdonユーザに切り替えて実行します (rootユーザで接続するのはこの都合です)。

## 作成したアカウントに管理者権限を設定する

[Dockerで雑にMastodonを起動する方法 - Qiita](http://qiita.com/zembutsu/items/fd52a504321dd5d6f0b8) で紹介されていました。公式ドキュメントでは [Turning into an admin](https://github.com/tootsuite/documentation/blob/8367c216524513ca9ed1e6fb4505a734853c308b/Running-Mastodon/Administration-guide.md#turning-into-an-admin) です。

サインアップ画面から通常のアカウントと同じ手順でアカウントを作成した後、以下のコマンドを実行すると管理権限が付与されます。

```
ssh ubuntu@mastodonサーバ
sudo su - mastodone
cd live
RAILS_ENV=production bundle exec rails mastodon:make_admin USERNAME=対象のユーザ名 
```

管理権限を持つユーザで設定画面を開くと「管理」というメニュー項目が追加されています。

## 他のユーザからの登録を受け付けない設定

「管理」メニューの「サイト設定」で
「新規登録を受け付ける」の右の「有効」をクリックするとトグルで「無効」になります。

## Ansible以外でのセットアップ

* いきなり自動化は心配なので最初は手動が良い方は
  [さくらのVPSで自分の Mastodon サーバを最速でつくる方法 - Qiita](http://qiita.com/hekki/items/c3f42c31632105389c79) をどうぞ。
* 手っ取り早くDockerで動かしてみたい方は
  [Dockerで雑にMastodonを起動する方法 - Qiita](http://qiita.com/zembutsu/items/fd52a504321dd5d6f0b8) をどうぞ。
* DockerじゃなくてVagrantが良い方は [mastdonのレポジトリ](https://github.com/tootsuite/mastodon/)内の [Vagrantfile](https://github.com/tootsuite/mastodon/blob/master/Vagrantfile) をどうぞ。
