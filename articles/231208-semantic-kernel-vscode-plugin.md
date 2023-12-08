---
title: "VSCode に Semantic Kernel E拡張機能を導入する"
emoji: "🐈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [SemanticKernel, VSCode]
published: true
---

マイクロソフト公式が Semantic Kernel 用の VSCode 拡張機能を公開しているので  
導入してみようと思います。

公式ドキュメント：

https://learn.microsoft.com/ja-jp/semantic-kernel/vs-code-tools?WT.mc_id=dotnet-9827-machiy

この **Semantic Kernel Tools** を使用すると、コードを記述することなく、新しい semantic functions を簡単に作成してテストできます。

![](https://learn.microsoft.com/ja-jp/semantic-kernel/media/vs-code-extension.png)

## VSCode にインストール

https://marketplace.visualstudio.com/items?itemName=ms-semantic-kernel.semantic-kernel

![](https://storage.googleapis.com/zenn-user-upload/18f938a5e1fd-20231207.png)

これを VSCode にインストールしてください

## Azure OpenAI エンドポイントへの接続

スキルのテストをする (モデルを実際に叩く) 必要があるので、  
エンドポイントへの接続が必要となります。

（事前に Azure に Azure OpenAI のリソースを作成、モデルをデプロイしておきましょう）

VSCode のコマンドパレットを開いて  
`Add Azure OpenAI Endpoint` します。

![](https://storage.googleapis.com/zenn-user-upload/5219b876bba0-20231207.png)

web ブラウザが開いて Azure にサインインしたり、  
コマンドパレットで  
以下の選択をします：

1. サブスクリプション
2. リソースグループ
3. Azure OpenAI のリソース
4. デプロイしたモデル

それぞれ選びましょう。

すると VSCode の notification に  
次のように「エンドポイント構成できました」という通知が出ます。

![](https://storage.googleapis.com/zenn-user-upload/2919764f343a-20231207.png)

## 新規作成

Semantic Kernel の VSCode extension で Semantic Kernel のプロジェクトを新規作成することができます。

### 例) ハローワールドの場合

![](https://storage.googleapis.com/zenn-user-upload/a4f1ab265a72-20231208.png)

これの下の ChatGPT plugin もいいですね

もっと試したら追記するかも
