# Misskey install shell script v1.0.0
Misskeyを簡単にインストールするためのシェルスクリプトができました！

いくつかの質問に答えるだけで、UbuntuサーバーへMisskey(v12)を簡単にインストールできます！

また、アップデートスクリプトもあります。

[**日本語版はこちら**](./README.md)

## 準備するもの
1. ドメイン
2. Ubuntuがインストールされたサーバー
3. Cloudflareアカウント（推奨）

## 操作
### 1. SSH
サーバーにSSH接続します。  
（デスクトップを開いている方はシェルを開きます。）

### 2. 環境を最新にする
すべてのパッケージを最新にし、再起動します。

```
sudo apt update; sudo apt full-upgrade -y; sudo reboot
```

### 3. インストールをはじめる
SSHを接続しなおして、Misskeyのインストールを始めましょう。

ただ、インストール前に[Tips](#Tips)を読むことを強くお勧めします。

```
wget https://raw.githubusercontent.com/joinmisskey/bash-install/main/ubuntu.sh -O ubuntu.sh; sudo bash ubuntu.sh
```

example.comは自分のドメインに置き換えてください。

### 4. アップデートする
アップデートのためのスクリプトもあります。

まずはダウンロードします。

```
wget https://raw.githubusercontent.com/joinmisskey/bash-install/main/update.ubuntu.sh -O update.sh;
```

アップデートしたいときにスクリプトを実行してください。

```
sudo bash update.sh
```

- systemd環境では、`-r`オプションでシステムのアップデートと再起動を行うことができます。
- docker環境では、引数に更新後のリポジトリ名:タグ名を指定することができます。

## 動作を確認した環境

### Oracle Cloud Infrastructure

このスクリプトは、Oracle Cloud InfrastructureのAlways Freeサービスで提供されている2種類のシェイプのいずれにおいても動作します。

- VM.Standard.E2.1.Micro (AMD)
- VM.Standard.A1.Flex (ARM) [1OCPU RAM6GB or greater]

## Issues & PRs Welcome
上記の環境で動作しない場合、バグの可能性があります。インストールの際に指定された条件を記載の上、GitHubのIssue機能にてご報告いただければ幸いです。

上記以外の環境についてのサポートは難しいですが、状況を詳しくお教えいただければ解決できる可能性があります。

機能の提案についても歓迎いたします。

# Tips
仕様や選択肢の選び方など。

## Systemd or Docker?
v1から、インストールメソッドにsystemdとDockerとを選べるようにしました。

Dockerと言っても、**MisskeyだけをDockerで実行します**。  
RedisやPostgresはホストマシンで実行します。  
[docker-composeですべての機能を動かす方法については、mamemonongaさんが作成したこちらの記事がおすすめです。](https://gist.github.com/mamemomonga/5549bb69cad8e5618e5527593d4890e0)

Docker Hubイメージを使う設定であれば、Misskeyのビルドが不要になります。  
**一番お勧めです。**  
ただし、マイグレーションは必要なので、Misskeyを使えない時間がゼロになるわけではありません。  
さらに、Misskeyのビルド環境を準備しないので、自分のフォークを動かしたくなった時に設定が面倒になります。

逆に、ローカルでDockerをビルドする方式は、パフォーマンス面で非推奨です。

systemdは、Docker Hubにイメージを上げるまでもないけれど、自分のフォークを使いたい場合にお勧めです。

お勧めする順番は次の通りです。

1. Docker Hub
2. systemd
3. Dockerビルド

## .envファイルについて
インストールスクリプトは、2つの.envファイルを作成します。  
アップデートの際に使用します。

### /root/.misskey.env
misskeyを実行するユーザーを覚えておくために必要です。

### /home/(misskeyユーザー)/.misskey.env
systemdの場合に生成されます。  
主にディレクトリを覚えておくのに使用します。

### /home/(misskeyユーザー)/.misskey-docker.env
Dockerの場合に生成されます。  
実行されているコンテナとイメージの番号を保存しています。  
コンテナの番号はアップデートの際に更新されます。古いイメージは削除されます。

## 自分で管理する
インストール後、構成を変更する際に役立つはずのメモです。

"example.com"を自分のドメインに置き換えて読んでください。

### Misskeyディレクトリ
Misskeyのソースは`/home/ユーザー/ディレクトリ`としてcloneされます。  
（ユーザー、ディレクトリの初期値はともにmisskeyです。）

Misskeyディレクトリへは、以下のように移動するとよいでしょう。

```
sudo su - ユーザー
cd ディレクトリ
```

もとのユーザーに戻るにはexitを実行します。

```
exit
```

### systemd
systemdのプロセス名はexample.comです。  
たとえば再起動するには次のようにします。

```
sudo systemctl restart example.com
```

journalctlでログを確認できます。

```
journalctl -t example.com
```

設定ファイルは`/etc/systemd/system/example.com.service`として保存されています。

### Docker
DockerはMisskeyユーザーでrootless実行されています。

sudo suでMisskeyユーザーに入るときは、`XDG_RUNTIME_DIR`と`DOCKER_HOST`を変更する必要があります。

```
sudo su - ユーザー
export XDG_RUNTIME_DIR=/run/user/$UID
export DOCKER_HOST=unix://$XDG_RUNTIME_DIR/docker.sock

docker ps
```

### nginx
nginxの設定は`/etc/nginx/conf.d/example.com.conf`として保存されています。

### Redis
requirepassとbindを`/etc/redis/misskey.conf`で設定しています。

## Q. アップデート後に502でアクセスできない
アップデート後にアクセスできない、ということが稀にあります。

スクリプトのバグはともかくとして、一般的に一番多い理由は、yarn installに失敗している場合です。  

Misskeyディレクトリで次の内容を実行し、もう一度アップデートを実行してみてください。

```
npm run cleanall
```

journalctlでログを確認すると、たいていre2が云々という記述が見当たります。

## Q. 同じサーバーにもう1つMisskeyを建てたい
スクリプトは同じサーバーに追加でMisskeyをインストールすることは想定していません。  
幾つかの設定が上書きされるか、途中でエラーになってしまうでしょう。
