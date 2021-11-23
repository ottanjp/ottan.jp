---
draft: false
title: Docker Desktop for Macに依存しないminikube + hyperkitによるDocker環境の構築
date: 2021-09-11T08:31:30Z
type: post
categories: ["Mac"]
---

Docker Desktopが有料化しました。個人や非商用のオープンソースの開発では引き続き無償で利用できます。Dockerを便利に利用している身としては、Docker社の利益に貢献したいという気持ちはあるものの、「無償のまま」利用し続ける方法を模索したくなるものです。そこで、Mac（Intel）版で引き続き、Docker環境を無償のまま利用し続ける方法を模索してみました。

* [Docker Desktopが有料化へ　従業員数250人未満・年間売り上げ1000万ドル未満の組織などは引き続き無料 - ITmedia NEWS](https://www.itmedia.co.jp/news/articles/2109/02/news112.html)

## Docker Desktop for Macのアンインストール

まずは、Docker Desktop for Macをアンインストールします。Homebrewでインストールしている場合は、以下のコマンドでアンインストールします。後述しますが、Homebrew経由でのDocker Desktop for Macのインストール、アンインストールは、`--cask`オプションを付与します。`--cask`オプション無しの場合、Docker CLIがインストールされます。

```zsh
brew uninstall docker --cask
```

## minikube + hyperkitのインストール

Docker Desktop for Macの代わりにKubernetes環境をローカル環境に簡単に構築できるminikubeを使用します。ただし、Kubernetesの機能は使用しません。コンテナーの実行エンジンとして利用します。Homebrew経由で簡単にインストールできるので、楽ちんです。

また、macOSはUNIXの派生ですがLinuxではありません。コンテナーを動かすためのVMが必要です。OSレベルでLinux環境を統合できるWindows強いですね。今回は、Docker Desktop for Macのコアエンジンでも利用されているHyperKitを使用します。HyperKitは、macOS標準のHypervisor.frameworkを使用します。他にもVirtualBoxを利用する方法などがあります。

* [Docker、MacOS X対応の軽量な仮想化ツール「HyperKit」をオープンソースで公開。Docker for Macの仮想化機能を取り出したもの － Publickey](https://www.publickey1.jp/blog/16/docker_hyperkit.html)

Homebrew経由で、どちらも簡単にインストールできます。

```zsh
brew install minikube hyperkit
```

### Docker CLIのインストール

続いて、Docker CLIをインストールします。記事執筆時点で、Docker CLI自体は無償のまま利用可能です。こちらも、Homebrewで簡単にインストールできます。

```zsh
brew install docker
```

なお、`--cask`オプションを付与すると、Docker Desktop for Mac（GUI）がインストールされてしまうため注意してください。

```zsh
brew install docker --cask
```

### minikubeの開始・終了

続いて、minikubeを開始します。minikubeを開始する場合、以下のコマンドを実行します。

```zsh
minikube start
```

VirtualBoxなどの仮想化ソフトウェが別途インストールされている場合、minikubeがスタート時に何を利用してVMを構築するかを選択します。明示的にドライバーを指定する場合、以下のコマンドを実行してください。

```zsh
minikube start --driver=hyperkit
```

minikubeの環境を初期化したい場合、以下を実行します。

```zsh
minikube delete
```

minikubeを停止する場合、以下を実行します。

```zsh
minikube stop
```

### Docker CLIをminikubeで使用する

minikubeを開始し、Docker CLIを使用しようと、たとえば以下のコマンドを実行します。

```zsh
docker ps
```

Dockerのデーモンが起動していないのでは？と怒られてしまいます。Dockerのデーモンは、HyperKit上で動作しており、Macのローカル環境では動作していないためです。

```txt
Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
```

そこで、Docker CLIの向き先をHyperKitに向けさせる必要があります。そのためには、以下のコマンドを実行します。

```zsh
minikube docker-env
```

すると、以下のような出力が得られます。

```zsh
export DOCKER_TLS_VERIFY="1"
export DOCKER_HOST="tcp://172.16.239.7:2376"
export DOCKER_CERT_PATH="/Users/ottan/.minikube/certs"
export MINIKUBE_ACTIVE_DOCKERD="minikube"

# To point your shell to minikube's docker-daemon, run:
# eval $(minikube -p minikube docker-env)
```

末尾に記載があるように、以下のコマンドを実行します。

```zsh
eval $(minikube -p minikube docker-env)
```

毎回このコマンドを実行するの面倒なので、`~/.zshrc`の末尾に追加しておくのが良さそうです。

### `docker login`時にエラーが発生

`docker login`時にエラーが発生します。

```txt
raise errors.InitializationError(
docker.credentials.errors.InitializationError: docker-credential-desktop not installed or not available in PATH
```

`docker-credential-desktop`がインストールされていない、と怒られます。Docker Hubを引き続き利用するには、Docker Credential Helperをダウンロードします。

```zsh
brew install docker-credential-helper
```

再び、`docker login`しようとして以下のエラーに遭遇するかもしれません。

```txt
Removing login credentials for https://index.docker.io/v1/
WARNING: could not erase credentials:
https://index.docker.io/v1/: error erasing credentials - err: exec: "docker-credential-desktop": executable file not found in $PATH, out: ``
```

再び、`docker-credential-desktop`がインストールされていない旨のメッセージです。Docker CLIが利用する際の認証情報として、macOSのキーチェーンを指定するようにしてあげましょう。

そのためには、`~/.docker/config.json`を修正します。

```diff
5c5
<   "credsStore": "osxkeychain",
---
>   "credsStore": "desktop",
```

ここまでくれば、Docker Desktop for Macのインストールなしに、`docker`、`docker-compose`（別途、Homebrewでインストール）等が動作するようになるはずです。

## まとめ

必要なものをまとめておきます。

```zsh
brew install docker docker-compose docker-credential-helper minikube hyperkit
```

また、`~/.zshrc`の末尾に以下を追記します。（オプション）

```zsh
eval $(minikube -p minikube docker-env)
```

Docker Hubを利用する場合、必要に応じて`~/.docker/config.json`を修正します。

```diff
5c5
<   "credsStore": "osxkeychain",
---
>   "credsStore": "desktop",
```
