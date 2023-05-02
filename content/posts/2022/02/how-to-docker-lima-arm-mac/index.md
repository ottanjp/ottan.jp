---
author: ottan
categories: null
date: 2022-02-19 20:47:09+09:00
draft: false
tags:
- 仮想マシン
- docker
- en
- require
- cli
title: Appleシリコン搭載Macで手軽にDocker開発環境をLimaで構築する
toc: true
type: post
---

Limaは、GitHubで「unofficial "containerd for Mac"」と謳われる、macOSのためのコンテナー（のようなもの）です。Windows 10以降に搭載されているWSL2と、同様の環境をmacOSで実現します。なお、Limaの仕組みは、Docker Dsektopのオープンソース版とも言える、[Rancher Desktop](https://rancherdesktop.io/)でも利用されています。

## macOSのためのLinux環境LimaでDockerを動かす

[lima-vm/lima: Linux virtual machines, typically on macOS, for running containerd](https://github.com/lima-vm/lima)は、Homebrewで簡単にインストールできます。

```zsh
brew install lima
```

GitHubの「examples」配下に多数のテンプレートが用意されています。今回は、Docker用のテンプレートである「docker.yaml」を用います。`limactl`コマンドで、仮想マシンを構築、起動できます。その際、Docker用のテンプレートを指定します。

```zsh
limactl start https://raw.githubusercontent.com/lima-vm/lima/master/examples/docker.yaml
```

Macのアーキテクチャ（Apple SiliconまたはIntel）に応じた、適切なUbuntu Serverのイメージのダウンロード、およびDockerのセットアップまで自動的に実施してくれます。

```yaml
images:
# Hint: run `limactl prune` to invalidate the "current" cache
- location: "https://cloud-images.ubuntu.com/impish/current/impish-server-cloudimg-amd64.img"
  arch: "x86_64"
- location: "https://cloud-images.ubuntu.com/impish/current/impish-server-cloudimg-arm64.img"
  arch: "aarch64"

```

はじめて起動する場合、指定したテンプレートをそのまま利用するか、カスタマイズするか確認されます。

```
Proceed with the default configuration
Open an editor to override the configuration
```

デフォルトでは、仮想マシン用にCPUが4コア、メモリが4GiB、ディスクが100GiB割り当てられますが、変更することも可能です。「Open an editor to override the configuration」を選択し、YAMLファイルの任意の場所に、以下を追記します。以下は、CPUが2コア、メモリが2GiB、ディスクが10GiBの場合の例です。なお、メモリが少なすぎる場合、仮想マシン構築中にメモリ不足エラーを引き起こします。

```yaml
cpus: 2
memory: "2GiB"
disk: "10GiB"
```

正常に仮想マシンの作成が終わると、以下のようなログが標準出力に出力されているはずです。

```
? Creating an instance "docker" Open an editor to override the configuration
INFO[0021] Attempting to download the image from "https://cloud-images.ubuntu.com/impish/current/impish-server-cloudimg-arm64.img"  digest=
INFO[0021] Using cache "/Users/ottan/Library/Caches/lima/download/by-url-sha256/a9f81252e41821dac2357ea4c9b5a5a1c71526b41bc4473d6365fa3594b86dd9/data"
INFO[0022] [hostagent] Starting QEMU (hint: to watch the boot progress, see "/Users/ottan/.lima/docker/serial.log")
INFO[0022] SSH Local Port: 52919
INFO[0022] [hostagent] Waiting for the essential requirement 1 of 5: "ssh"
INFO[0053] [hostagent] Waiting for the essential requirement 1 of 5: "ssh"
INFO[0054] [hostagent] The essential requirement 1 of 5 is satisfied
INFO[0054] [hostagent] Waiting for the essential requirement 2 of 5: "user session is ready for ssh"
INFO[0089] [hostagent] Waiting for the essential requirement 2 of 5: "user session is ready for ssh"
INFO[0090] [hostagent] The essential requirement 2 of 5 is satisfied
INFO[0090] [hostagent] Waiting for the essential requirement 3 of 5: "sshfs binary to be installed"
INFO[0096] [hostagent] The essential requirement 3 of 5 is satisfied
INFO[0096] [hostagent] Waiting for the essential requirement 4 of 5: "/etc/fuse.conf to contain \"user_allow_other\""
INFO[0099] [hostagent] The essential requirement 4 of 5 is satisfied
INFO[0099] [hostagent] Waiting for the essential requirement 5 of 5: "the guest agent to be running"
INFO[0099] [hostagent] The essential requirement 5 of 5 is satisfied
INFO[0099] [hostagent] Mounting "/Users/ottan"
INFO[0099] [hostagent] Mounting "/tmp/lima"
INFO[0099] [hostagent] Waiting for the optional requirement 1 of 1: "user probe 1/1"
INFO[0099] [hostagent] Forwarding "/run/user/501/docker.sock" (guest) to "/Users/ottan/.lima/docker/sock/docker.sock" (host)
INFO[0099] [hostagent] Forwarding "/run/lima-guestagent.sock" (guest) to "/Users/ottan/.lima/docker/ga.sock" (host)
INFO[0099] [hostagent] Not forwarding TCP 127.0.0.53:53
INFO[0099] [hostagent] Not forwarding TCP 0.0.0.0:22
INFO[0099] [hostagent] Not forwarding TCP [::]:22
INFO[0126] [hostagent] The optional requirement 1 of 1 is satisfied
INFO[0126] [hostagent] Waiting for the final requirement 1 of 1: "boot scripts must have finished"
INFO[0129] [hostagent] The final requirement 1 of 1 is satisfied
INFO[0129] READY. Run `limactl shell docker` to open the shell.
INFO[0129] Message from the instance "docker":
To run `docker` on the host (assumes docker-cli is installed), run the following commands:
------
docker context create lima --docker "host=unix:///Users/ottan/.lima/docker/sock/docker.sock"
docker context use lima
docker run hello-world
------
```

### 仮想マシンへログインする

仮想マシンへログインするには、以下のコマンドを実行します。仮想マシン名は、起動時に指定したYAMLファイルのファイル名です。`limactl list`で起動中の仮想マシン名を確認できます。

```zsh
limactl shell <仮想マシン名>
```

### 仮想マシンを停止する

仮想マシンを停止するには、以下のコマンドを実行します。

```zsh
limactl stop <仮想マシン名>
```

### 仮想マシンを起動する

仮想マシンを起動するには、以下のコマンドを実行します。

```zsh
limactl start <仮想マシン名>
```

## ホストOS（macOS）のDocker CLIを用いてContainerを操作する

まず、Docker CLIをインストールします。

```zsh
brew install docker
```

続いて、ホストであるmacOSのDocker CLIから、Lima（Ubuntu）上のDockerを操作できるよう、DockerのコンテキストにLimaの設定を追加します。

```zsh
docker context create lima --docker "host=unix:///Users/ottan/.lima/docker/sock/docker.sock"
docker context use lima
docker run hello-world
```

これらのコマンドは、`limactl start`時のログの最後に表示されているため、実際の実行時にはそちらのコマンドを実行してください。

```
INFO[0129] Message from the instance "docker":
To run `docker` on the host (assumes docker-cli is installed), run the following commands:
------
docker context create lima --docker "host=unix:///Users/ottan/.lima/docker/sock/docker.sock"
docker context use lima
docker run hello-world
------
```

`docker run hello-world`で`Hello from Docker!`と表示されればOKです。

## 参考リンク

* [lima-vm/lima: Linux virtual machines, typically on macOS, for running containerd](https://github.com/lima-vm/lima)
* [Rancher Desktop](https://rancherdesktop.io/)
