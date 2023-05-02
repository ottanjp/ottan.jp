---
author: ottan
categories: null
date: 2022-03-19 10:27:58+09:00
draft: false
tags:
- x86
- mysql
- シリコン
- docker
- 仮想マシン
title: Appleシリコン搭載Macで手軽にx86_64開発環境をLimaで構築する
toc: true
type: post
---

[Appleシリコン搭載Macで手軽にDocker開発環境をLimaで構築する](/posts/2022/02/how-to-docker-lima-arm-mac/)で、Appleシリコン（M1）搭載のMacでDocker開発環境を構築する手順をご紹介しました。今回は、Appleシリコン（ARM64）上で、x86_64環境をエミュレート（QEMU）したDocker開発環境を構築します。現状x86_64環境でしか動作しない、MySQLのDocker Imageを起動して、接続するところまでを試します。

Limaの基本的な操作は[こちら](/posts/2022/02/how-to-docker-lima-arm-mac/)の記事を参照してください。

## x86_64のLimaを構築する

Docker用の仮想マシンを構築するためのテンプレートは、[GitHub](https://github.com/lima-vm/lima)に用意されているため、それをそのまま用います。

```zsh
limactl start --name=docker-x86_64 https://raw.githubusercontent.com/lima-vm/lima/master/examples/docker.yaml
```

前回、`docker`という名称で仮想マシンを作成したため、今回は`--name`オプションを付与して名前（`docker-x86_64`）を指定しました。

前回同様、「Open an editor to override the configuration
」を選択して、仮想マシン作成前に設定を変更します。

```yaml
arch: x86_64
cpus: 2
memory: "2GiB"
disk: "10GiB"
```

ポイントは、`arch`に`x86_64`を指定している点です。デフォルトでは、実行中のMacに合ったもの（`aarch64`）が自動で指定されます。今回は、`x86_64`の仮想マシンを作成したいため、`arch`を指定します。

Appleシリコン（ARM64）上で、x86_64をエミュレート（QEMU）しながら動作させるため、パフォーマンスは大幅に下がります。そのため、仮想マシンの構築や起動までに時間を要します。

仮想マシンが起動したら、以下のコマンドで、Dockerのコンテキストに追加します。

```zsh
docker context create lima-x86_64 --docker "host=unix:///Users/ottan/.lima/docker/sock/docker.sock"
docker context use lima-x86_64
docker run hello-world
```

前回作成したコンテキストと適宜切り替えて使ってください。

```zsh
docker context use lima # aarch64
docker context use lima-x86_64 # x86_64
```

## x86_64のMySQLイメージから起動する

以下のコマンドを実行し、MySQLの最新版を起動します。`--platform linux/x86_64`を指定するのがポイントです。このオプションを付与しない場合のデフォルトの挙動は、Appleシリコン（ARM64）版のMySQLイメージから起動しようとしますが、Appleシリコンに対応したMySQLイメージは現在存在しないためエラーになります。

```zsh
docker run --platform linux/x86_64 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=password mysql:latest
```

起動後、MySQLクライアントからMySQLへ接続します。`root`ユーザのパスワードは、`MYSQL_ROOT_PASSWORD`環境変数で指定したパスワード（今回の場合、`password`）です。

```zsh
mysql -h127.0.0.1 -uroot -p -P3306
```

無事、MySQLへ接続できることがわかります。

```
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.28 MySQL Community Server - GPL

Copyright (c) 2000, 2022, Oracle and/or its affiliates.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>
```
