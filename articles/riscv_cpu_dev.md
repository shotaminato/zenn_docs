---
title: "RISC-V CPU自作備忘録（学習編）"
emoji: "📚"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [docker, ubuntu, riscv, chisel]
published: true
---

# はじめに
RISC-Vの理解のため「RISC-VとChiselで学ぶ はじめてのCPU自作 ――オープンソース命令セットによるカスタムCPU実装への第一歩」を参考にして、CPU自作に挑戦中です。  
本記事は、本書からの学びや補足、変更点などの備忘録です。  

環境構築編はこちら（VSCODEでの環境構築方法をまとめました）：
@[card](https://zenn.dev/shotaminato/articles/riscv_cpu_dev_env)

## 参考図書
- RISC-VとChiselで学ぶ はじめてのCPU自作 ――オープンソース命令セットによるカスタムCPU実装への第一歩  

@[card](https://amzn.asia/d/aXO4PQ9)

## 著者の開発環境（参考）

| 項目    | 内容             |
| ----   | ----             |
| OS     | Ubuntu 22.04.3   |
| CPU    | AMD Ryzen5 5600X |
| GPU    | NVIDIA RTX3060   |
| Memory | 32GB             |

# 第4章　環境構築

## git のインストール
gitがない場合は、下記コマンドでインストールします。

```bash
sudo apt install git
```

## 4-1 chisel-templateのダウンロード
Githubリポジトリの作成者の名前が本書の内容から変わっています。よってクローンもとのアドレスが異なります。
（記載のアドレスでもクローンはできるが、Github上で発見できなくて困惑しました）  
下記のリポジトリをクローンします。
@[card](https://github.com/chipsalliance/chisel-template.git)

## 4-2 Dockerによる実行環境の構築
### 4-2-1 Dockerのインストール
Ubuntuなので、下記のURLに従ってインストールしました。  
@[card](https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository)
「Installation methods」の「Install using the apt repository」を参考にしました。  
また、下記のURLを参考にして、sudoなしでもdockerを扱えるように変更しました。  
@[card](https://docs.docker.com/engine/install/linux-postinstall/)
以下、インストールで実行したコマンドです。

```bash: インストールで実行したコマンド
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/ apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

```bash: インストール確認
sudo docker run hello-world
```

```bash: sudoなし対応で実行したコマンド
sudo groupadd docker
sudo usermod -aG docker $USER
一旦、ターミナルを再起動
newgrp docker
```

```bash: sudoなしで実行できるか確認
docker run hello-world
```

### 4-2-2 Dockerfileの作成
本書の内容そのままではビルドが通らないため、下記の通り変更した。
（参考：https://zenn.dev/rm48/articles/22712c0aeff168）

```dockerfile: 作成したDockerfile
# ベースイメージの設定（22.04に変更。18.04は古すぎるためビルドが通らないとのこと）
FROM ubuntu:22.04

# 環境変数の定義
ENV RISCV=/opt/riscv
ENV PATH=$RiSCV/bin:$PATH
ENV MAKEFLAGS=-j4
WORKDIR $RISCV

# 基本ツールのインストール
RUN apt update
RUN apt install -y autoconf automake autotools-dev curl libmpc-dev libmpfr-dev libgmp-dev gawk build-essential bison flex texinfo gperf libtool patchutils bc zlib1g-dev libexpat-dev pkg-config git libusb-1.0-0-dev device-tree-compiler default-jdk gnupg vim 
# 以下のツールも追加インストール（RISC-V GNU Compiler Toolchain のビルドに必要なため）
RUN apt install -y python3 python3-pip ninja-build cmake libglib2.0-dev default-jre nano

# riscv-gnu-toolchain(ベクトル対応ver.)のビルド（使用ブランチを最新に変更）
RUN git clone https://github.com/riscv/riscv-gnu-toolchain 
RUN cd riscv-gnu-toolchain && mkdir build && cd build && ../configure --prefix=${RISCV} --enable-multilib && make

# riscv-testsのダウンロード（変更なし）
RUN git clone -b master --single-branch https://github.com/riscv/riscv-tests && \
	cd riscv-tests && git checkout c4217d88bce9f805a81f42e86ff56ed363931d69 && \
	git submodule update --init --recursive

# sbtのインストール（変更なし）
RUN echo "deb https://repo.scala-sbt.org/scalasbt/debian all main" | tee -a /etc/apt/sources.list.d/sbt.list && \
	echo "deb https://repo.scala-sbt.org/scalasbt/debian /" | tee /etc/apt/sources.list.d/sbt_old.list && \
	curl -sL "https://keyserver.ubuntu.com/pks/lookup?op=get&search=0x2EE0EA64E40A89B84B2DF73499E82A75642AC823" | apt-key add && \
	apt-get update && apt-get install -y sbt
```

### 4-2-3 イメージの作成
```bash: 実行したコマンド
cd <Docker fileのあるディレクトリ>
docker build . -t riscv/mycpu
docker images
```

# 第6章 ChiselTestによる命令フェッチテスト

## 6.1 ChiselTestのインストール

本書には「build.sbtを変更する必要はない」という旨の記載ありますが、実際には変更しなければビルドが通りません。  
（リポジトリの内容が更新されているため）  
著者のリポジトリ：https://github.com/chadyuu/riscv-chisel-book を参考にしましょう。  
私は、 https://github.com/chipsalliance/chisel-template.git の update-chisel-and-scala ブランチの内容を参考にして、下記の通り変更しました。  

```scala:build.sbt
// See README.md for license details.

ThisBuild / scalaVersion     := "2.13.7"
ThisBuild / version          := "0.1.0"
ThisBuild / organization     := "%ORGANIZATION%"

val chiselVersion = "3.5.0-RC2"

lazy val root = (project in file("."))
  .settings(
    name := "%NAME%",
    libraryDependencies ++= Seq(
      "edu.berkeley.cs" %% "chisel3" % chiselVersion,
      "edu.berkeley.cs" %% "chiseltest" % "0.5.0-RC2" % "test"
    ),
    scalacOptions ++= Seq(
      "-language:reflectiveCalls",
      "-deprecation",
      "-feature",
      "-Xcheckinit",
    ),
    addCompilerPlugin("edu.berkeley.cs" % "chisel3-plugin" % chiselVersion cross CrossVersion.full),
  )
```
また、いくつかscalaの修正が必要です。  
このページを参考にさせていただきました。  
https://www.rm48.net/post/risc-v%E3%81%A8chisel%E3%81%A7%E5%AD%A6%E3%81%B6-%E3%81%AF%E3%81%98%E3%82%81%E3%81%A6%E3%81%AE%E8%87%AA%E4%BD%9Ccpu-%E3%83%A1%E3%83%A2

私の環境では以下の結果となりました。参考書の内容とは異なり、0x34333231のサイクルは結果が出力されませんでした。  
念の為、fetch.hex を変更して確認しましたが、やはり最終サイクルの結果は出力されませんでした。

```テスト結果
pc_reg : 0x00000000
inst   : 0x14131211
-------------------------------
pc_reg : 0x00000004
inst   : 0x24232221
-------------------------------
```

# 第8章 LW命令の実装
## 8-3 テストの実行

本の内容そのままではテストが通りません。下記のエラーが出ます。  
```bash
[info] HexTest:
[info] mycpu
[info] - should work through hex *** FAILED ***
[info]   firrtl.passes.PassExceptions: firrtl.passes.CheckInitialization$RefNotInitializedException:  @[Top.scala 13:24] : [module Top]  Reference memory is not fully initialized.
[info]    : memory.io.dmem.addr <= VOID
[info] firrtl.passes.CheckInitialization$RefNotInitializedException:  @[Top.scala 12:24] : [module Top]  Reference core is not fully initialized.
[info]    : core.io.dmem.rdata <= VOID
[info] firrtl.passes.PassException: 2 errors detected!
[info]   ...
[info] Run completed in 1 second, 557 milliseconds.
[info] Total number of tests run: 1
[info] Suites: completed 1, aborted 0
[info] Tests: succeeded 0, failed 1, canceled 0, ignored 0, pending 0
[info] *** 1 TEST FAILED ***
```

これは Top.scala 内の記述が漏れていることが原因です。  
第8章では、coreとmemoryに新しいインターフェースを追加したたため、それらを接続してあげる必要があります。  
具体的には、Top.scalaに以下の記述を追加します。  

```scala
class Top extends Module {
    ...
    core.io.imem <> memory.io.imem
    core.io.dmem <> memory.io.dmem // 追加
    ...
}
```
（初学者ですが、自分でコードを解釈して、修正箇所と修正方法を見つけることができました。嬉しかったです笑）

# 続きを作成中（20230318）

# メモ
## Chisel 記号の意味
```:=```のあとには、次のクロックで更新する値を記載する。
```=```のあとには、初期化時の設定を記載する。
