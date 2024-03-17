---
title: "RISC-V CPU自作備忘録（環境構築）"
emoji: "🐙"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [riscv, chisel, ubuntu, docker]
published: true
---

# はじめに
RISC-Vの理解のため、「RISC-VとChiselで学ぶ はじめてのCPU自作 ――オープンソース命令セットによるカスタムCPU実装への第一歩」を参考にして、CPU自作に挑戦中です。  

本記事は、Vscodeで本書の内容を学習するための、環境構築メモです。  
VscodeでScalaを開発する場合は、Metalsという拡張機能を使用すると良いです。  

本編は以下のページです。
@[card](https://zenn.dev/shotaminato/articles/riscv_cpu_dev)

## 参考図書
- RISC-VとChiselで学ぶ はじめてのCPU自作 ――オープンソース命令セットによるカスタムCPU実装への第一歩  

https://amzn.asia/d/aXO4PQ9


## 著者の開発環境（参考）

| 項目    | 内容             |
| ----   | ----             |
| OS     | Ubuntu 22.04.3   |
| CPU    | AMD Ryzen5 5600X |
| GPU    | NVIDIA RTX3060   |
| Memory | 32GB             |

# Dockerイメージ立ち上げ
本の内容に従って、Dockerイメージを作成します。  
注意点として、本に記載の内容は古いため、一部内容の修正が必要です。  
修正内容はまとめてありますので、参考にしてください。  
@[card](https://zenn.dev/shotaminato/articles/riscv_cpu_dev)

# Vscode インストール
下記のページに従って、Vscodeをインストールしてください。  
すでにインストール済みの方はスキップしてください。  
https://code.visualstudio.com/


# Vscode 拡張機能インストール
Vscodeを立ち上げます。任意のディレクトリで、下記コマンドを実行してください。
```bash:Vscode立ち上げ
$ code
```
Vscodeの画面が現れます。
![Vscode画面](/images/24031703.png)

サイドバーの「Extensions」ボタンをクリックし、下記拡張機能をインストールします。

| 拡張機能名                | 内容   |
| ----                    | ----   |
| Dev Containers          | VSCodeをDockerコンテナにアタッチできるようにするための拡張機能です。  |
| Scala Syntax (official) | Scalaのシンタックスハイライトのための拡張機能です。ただし、この拡張機能のみでは十分な開発サポートを受けられないため、下記のMetalsをインストールします。|
| Scala (Metals)          | .sbtファイルで定義されたワークスペースを取り込み、importの補完などを行ってくれます。  |

![Extensions](/images/24031705.png)


# Vscode に docker をアタッチ

拡張機能「Dev Containers」のインストールが完了すると、サイドバーにリモートエクスプローラーが表示されます。  
また、画面左下に、下記のような緑のアイコンが表示されています。  
![Open a remote windowボタン](/images/24031701.png)

これをクリックすると、コマンドパネルメニューが表示されます。  
![コマンドパネルメニュー](/images/24031702.png)

「Attach to Running Container (実行中のコンテナにアタッチ)」から、任意のコンテナにアタッチすることができます。  
本通りに作業していれば、下記の通り「riscv/mycpu」というコンテナがあるはずです。そちらを選びます。
![Attach to Running Container](/images/24031704.png)

これでDockerとVscodeの接続は完了です。


# Metal に build.sbt を import
拡張機能「Metals」のインストールが完了すると、サイドバーにMetalsのマークが表示されます。  
クリックすると下記の画面が現れます。  
![Metals](/images/24031706.png)

前章の手順で、Vscodeを起動済みの「riscv/mycpu」にアタッチします。  
本通りの手順であれば、起動済みのdockerコンテナには、sbtがインストールされているはずです。  
この状態であれば、Metalsを使用することができます。  

Metalsのメニューから、「Import build」を選択すると、build.sbtの内容が読み込まれます。  
これでMetalsの設定が完了です。下記のように、import や 変数が正しくシンタックスハイライトされていれば成功です。
![起動確認](/images/24031707.png)