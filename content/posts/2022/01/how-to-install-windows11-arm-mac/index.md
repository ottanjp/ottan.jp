---
author: ottan
date: 2022-01-13T22:34:38+09:00
draft: False
title: "UTMでM1 MacにWindows 11 Insider Previewをインストールする方法"
type: post
categories: ["Mac"]
tags:
toc: False
---

Apple Silicon搭載のM1 Mac（MacBook Air 2020のエントリーモデル）に、Windows 11（Insider Preview版）をインストールしてみました。今回は、UTMを利用します。

## 事前準備

今回、試した環境は以下の通りです。

### 環境

```
Darwin MacBook-Air.local 21.2.0 Darwin Kernel Version 21.2.0: Sun Nov 28 20:29:10 PST 2021; root:xnu-8019.61.5~1/RELEASE_ARM64_T8101 arm64
```

macOS Monterey（12.1）で試しました。

```
ProductName:	macOS
ProductVersion:	12.1
BuildVersion:	21C52
```

UTMは、Homebrewでインストールしました。

```
utm: 2.4.1
https://getutm.app/
```

また、VHDX形式の仮想ディスクイメージファイルをQCOW2形式へ変換するためのツールが必要です。

```
qemu: stable 6.2.0 (bottled), HEAD
Emulator for x86 and PowerPC
https://www.qemu.org/
```

### UTMのインストール

UTMは、Homebrewからインストールできます。

```zsh
brew install --cask utm
```

### ARM64アーキテクチャのWindows 11ディスクイメージのダウンロード

現在、Windows 11（ARM64）版は、Windows Insider Preview版のみ提供されています。Micorosoftアカウントでサインインして、Insider Preview版への参加に同意が必要です。

* <https://www.microsoft.com/en-us/software-download/windowsinsiderpreviewARM64>

### SPICE Guest Toolsのダウンロード

Windows 11 Insider Preview版インストール直後は、ネットワークへ接続ができません。UTM用のドライバーソフトウェアをダウンロードしておきます。

* <https://mac.getutm.app/support/>

### 仮想ディスクイメージの形式変換（VHDX→QCOW2）

Windows 11 Insider Preview版は、仮想マシンのディスクイメージファイル（VHDX形式）で提供されています。そのままの形式で利用することも可能なのですが、QEMUを使用しているUTMで仮想マシンがクラッシュするなど、動作が不安定です。そのため、VHDX→QCOW2に変換しておきましょう。

```zsh
brew install qemu
```

QCOW2形式への変換はUTMでは提供されていないため、Homebrewでインストールします。

```zsh
qemu-img convert -p -O qcow2 Windows11_InsiderPreview_Client_ARM64_en-us_22523.VHDX Windows11_InsiderPreview_Client_ARM64_en-us_22523.qcow2
```

## 仮想マシンの作成

「Create a New Virtual Machine」をクリックします。

![](9613ecf8f9a3b4e68ad891988a611ecc52cf96e0a782f52b1d45c2c41177ae8b.png)

### Information

「Name」に仮想マシン名、「Style」は「Operating System」を選択します。

![](b4dd3e7df3de036fa32f55dc3c695671b614ec26d3315a53531912172ec28c2e.png)

### System

「Architecture」は「ARM64（aarch64）」を選択すると、「System」がApple Siliconに合わせて自動選択（QEMU 6.x ARM Virtual Machine）されます。仮想マシンとM1 Macのアーキテクチャが一致している（ARM64）ため、UTMはエミュレーションしません。「Memory」は、最低「4096MB」以上にします。

任意ですが、「Show Advanced Settings」をチェックし、「CPU」を「Default」へ、「Force Multicore」をチェックします。マルチコアを有効にする場合、選択してください。また、「CPU Cores」にコア数を入力します。

![](582d2db470e455a76ed059b827d13432d8f29ba6f7682d20bf4c09f65a49b1fa.png)

### Drives

「Import Drive」で、ダウンロードしたISOファイルを選択します。Windowsのディスクイメージファイルには「virtio」のドライバーが付属していないため、 **「Interface」を「NVMe」へ変更する**必要があります。また、「New Drive」で、「Removable Drive」を作成します。

![](e44b329e2a62279612f362eac91f0cc983cd973b0404be03ad4dec4d826b25b9.png)

ここまでで仮想マシンの作成は完了です。「Save」ボタンをクリックします。

## 仮想マシンの起動

作成した仮想マシンを選択して起動します。SPICE Guest Toolsは、OSセットアップ完了後にインストールするため、この段階ではCD/DVDは空の状態にしておきます。

![](e58321c53e1dc74c52cec494788d508faf1b1f20d3d66e80cfe027efeb9cc9d0.png)

解像度が低く、Windows 11の初期セットアップ時に支障が出るため、ブートメニューから変更します。「Start boot option」の文字が表示されたら、Escキーを押します。

![](e3ed8d20b3e34cf5e954d72f1bb272a28690e59aee8dc75f8d65e84340b08f7b.png)

「Device Manager」を選択します。

![](d0d2c9ea6ebb3824bca50ea12ab4c54a1a3627e0d20253416c300df0e224b0cd.png)

「OVMF Platform Configuration」を選択します。

![](2517cb9d3015c1f96adffe01458778c466ca8ba85ca606cf3b5fef865d320852.png)

解像度を「1024x768」へ変更します。F10キーを押して内容を保存します。

![](e27047166a9cd42da8185bdfc09f8cb70e5843ae6c274b071a025a1af3717909.png)

トップへ戻り、「Reset」を選択し、仮想マシンを再起動します。再起動後に、設定した解像度が適用されます。

![](b76a31b99a85ae86bcfc9a767fe05fdcb0f3caa770d453a12932184b09ef25f9.png)

以降、Windows 11のインストールウィザードにしたがってインストールを進めます。途中、Wi-Fiへの接続を求められますが、現時点でネットワークへ接続できないため、スキップして進めてください。なお、画面からカーソルが抜け出せなくなったら、Control + Optionキーを押してください。

![](133668a44a3c703db8324075b8e89cf3d7cd227d1f45610aeb2e65a892206cfd.png)

## SPICE Guest Toolsのインストール

ウインドウのタイトルバー右端にある、CD/DVDのアイコンをクリックし、あらかじめダウンロードしておいた、SPICE Guest ToolsのISOファイルを選択します。

![](55b38110bedc1243965ac0535d3f2686ab0a770cf5068283527be91025186cf6.png)

セットアップファイルを起動し、ウィザードの内容にしたがいインストールします。インストール後、仮想マシンを再起動しましょう。これで、仮想マシンがネットワークに接続されます。

![](665ce3c6cf4cef4a07c3de83976e8bfb978674654078a65fa2c9bd7a405f4c70.png)

## まとめ

UTM（QEMU）を用いて、Apple Silicon搭載のM1 Mac（MacBook Air, 2020）にWindows 11（ARM64）Insider Preview版をインストールしました。現在は、Insider Preview版しかARM64は用意されておらず、Microsoftの公式のサポートが待ち遠しいですね。

## 参考

* <https://mac.getutm.app/gallery/windows-11-arm>