# はじめに
RISC-Vの理解のため、「RISC-VとChiselで学ぶ はじめてのCPU自作 ――オープンソース命令セットによるカスタムCPU実装への第一歩」を参考にして、CPU自作に挑戦中です。  

本記事は、Vscodeで本書の内容を学習するための、環境構築メモです。  
sbtを使ったScalaの開発をVscodeで行う場合は、Metalsという拡張機能を使用すると良いです。


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

# Vscode インストール
下記のページに従って、Vscodeをインストールしてください。  
すでにインストール済みの方はスキップしてください。  
https://code.visualstudio.com/


# Vscode 拡張機能インストール
下記拡張機能をインストールします
- Dev Containers
    - DockerとVscodeを接続するための拡張機能です。
- Scala Syntax (official)
    - Scalaのシンタックスハイライトのための拡張機能です。
    - ただし、この拡張機能のみでは十分な開発サポートを受けられないため、下記のMetalsをインストールします。
- Scala (Metals)
    - .sbt ファイルで定義されたワークスペースを取り込み、importの補完などを行ってくれます。


# Dockerイメージ立ち上げ
本の内容に従って、Dockerイメージを作成します。  
注意点として、本に記載の内容は古いため、一部内容の修正が必要です。  
下記記事にまとめてありますので、参考にしてください。  


# Vscode に docker をアタッチ

# Metal に build.sbt を import