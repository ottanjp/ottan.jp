---
author: ottan
date: 2022-04-20T22:36:01+09:00
draft: false
title: "Appleシリコン搭載MacでJupyter Lab/Notebookを動かす"
type: post
categories:
tags:
toc: true
---

Appleシリコン（M1）搭載Macで、Jupyter Lab/Notebookを動かします。

## Miniforgeのインストール

[conda-forge/miniforge: A conda-forge distribution.](https://github.com/conda-forge/miniforge)は、`conda-forge`に特化したミニマルなAnaconda環境で、Apple M1をはじめとしたARM64環境などさまざまなアーキテクチャをサポートしているのが特徴です。[macOS（またはLinux）用パッケージマネージャー — Homebrew](https://brew.sh/index_ja)を用いると、簡単にインストールできます。

```zsh
brew install --cask miniforge
```

### Miniforgeの初期設定

`.zshrc`（Zshの場合）に、`conda`コマンドのパスの追記など、必要な設定を追記します。`conda init`コマンドを実行すると、自動的に現在のシェルの起動スクリプトが書き換えられます。

```zsh
conda init      # シェル環境の起動スクリプトの書き換え
exec $SHELL -l  # シェル環境の再起動
```

## Jupyter Lab/Notebook用の仮想環境を作成

ここからAnacondaの仮想環境を作成します。仮想環境の作成は、`conda create`コマンドを使用します。`--name`オプションで仮想環境に任意の名前を付与します。また、pythonのバージョンを指定することもできます。今回は、Python 3.8の仮想環境を作成します。

```zsh
conda create --name python38 python=3.8
```

作成した環境に切り替えるには、`conda activate`コマンドを使用します。元の環境（`base`と呼ばれます）に戻るには、`conda deactivate`コマンドを使用します。

```zsh
conda activate python38
```

仮想環境がアクティベートされたら（プロンプトにも環境名が表示されています）、`conda-forge`から`jupyter`、`jupyterlab`をインストールします。

```zsh
conda install jupyter jupyterlab
```

あとは、Jupyter Lab/Notebookを起動するだけです！簡単ですね。

```zsh
jupyter lab
```
