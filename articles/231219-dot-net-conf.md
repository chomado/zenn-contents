---
title: ".NET Conf Japan 2023「.NET + AI」補足記事"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [SemanticKernel, CSharp, Azure]
published: true
publication_name: "microsoft"
---

2023/12/19(火) に **.NET Conf 2023 Recap Japan** オンラインイベントが開催され、  
私も「.NET (C#) + Azure OpenAI」についてお話しします。

https://mktoevents.com/Microsoft+Event/415522/157-GQE-382

その補足記事です。

![](https://storage.googleapis.com/zenn-user-upload/3451058409dd-20231208.png)

# なぜ補足記事を書いたのか

このイベントの全セッションは「事前収録」のものとなっており、   
収録日 2023/12/08(金) 時点 (最新バージョンは RC-3 版) での話となります。

私のトピック AI は情報の移り変わりが早く、   
とくに (後述する) LLM 用ライブラリ **Semantic Kernel は今週だけで 3 回パッケージの更新** （しかも **破壊的変更まみれ** で既存のコードがコンパイルエラーまみれで動かなくなった）があり、  
**収録から 1 週間以上経つと、当日「セッションで紹介されたコードが動かない！」ということが普通にあり得る** ので、   
この、いつでも手元で更新できる補足記事を書きました。

この記事では 2023/12/19(火) 時点での最新の情報を書いておくように更新し続けます。

:::message
追記： 2023/12/19(火)

予想通り、あの後   
次の週に RC-4 (Release Candidate 4) 版、   
さらに今日 12/19(金) **当日（！）** に [正式版 v1.0.1](https://github.com/microsoft/semantic-kernel/releases/tag/dotnet-1.0.1)が出て、   
完全に既存のコードが動かなくなりましたので、この記事を更新しました。    
この記事は現在 最新の正式版に対応しています (イベント当日の朝に更新) (がんばりました…！)
:::

![](https://storage.googleapis.com/zenn-user-upload/66f81895ed61-20231219.png)

# セッションアジェンダ

1. Azure OpenAI
1. Semantic Kernel とは
1. [デモ] Semantic Kernel ハローワールド 
    * とりあえず叩いてみる
1. [デモ] Semantic Kernel chat bot
    * ユーザとマルチターンで会話のできる chat bot
1. [デモ] Notebook サンプル
    * いろんなサンプルコードが見られます
1. 各種サンプル紹介
    * [Semantic Kernel Bot in-a-box](https://github.com/Azure/AI-in-a-Box/tree/main/semantic-kernel-bot-in-a-box) (SK beta-8)
1. まとめ
    * V1.0.0 正式リリースされたらぜひ使ってみてくださいね！

# Azure OpenAI

![](https://storage.googleapis.com/zenn-user-upload/f4902c11e28b-20231219.png)

公式ドキュメント

https://learn.microsoft.com/ja-jp/azure/ai-services/openai?WT.mc_id=dotnet-9827-machiy

> Azure OpenAI Service は、GPT-4、GPT-3.5-Turbo、Embeddings (埋め込み) モデル シリーズなど OpenAI の強力な言語モデルに、REST API でのアクセスを提供します。


## データポリシー

https://learn.microsoft.com/ja-jp/legal/cognitive-services/openai/data-privacy?WT.mc_id=dotnet-9827-machiy

リクエストやレスポンスに含まれるデータは保存されることなく、(マイクロソフトや OpenAI などの) モデルの学習にも使用されることは無いし、もちろん他の客とかに見られることも無い、と明記されている


:::message

原文

Your prompts (inputs) and completions (outputs), your embeddings, and your training data:

* are NOT available to other customers.
* are NOT available to OpenAI.
* are NOT used to improve OpenAI models.
* are NOT used to improve any Microsoft or 3rd party products or services.
* are NOT used for automatically improving Azure OpenAI models for your use in your resource (The models are stateless, unless you explicitly fine-tune models with your training data).
* Your fine-tuned Azure OpenAI models are available exclusively for your use.

The Azure OpenAI Service is fully controlled by Microsoft; Microsoft hosts the OpenAI models in Microsoft’s Azure environment and the Service does NOT interact with any services operated by OpenAI (e.g. ChatGPT, or the OpenAI API).

:::

# Semantic Kernel とは

Semantic Kernel とは、    

* OpenAI, Azure OpenAI などの大規模言語モデル (LLM) 用のライブラリ。
* Microsoft が中心となり OSS で開発

https://github.com/microsoft/semantic-kernel

### 公式から引用

> Semantic Kernel is an SDK that integrates Large Language Models (LLMs) like OpenAI, Azure OpenAI, and Hugging Face with conventional programming languages like C#, Python, and Java. Semantic Kernel achieves this by allowing you to define plugins that can be chained together in just a few lines of code.

https://github.com/microsoft/semantic-kernel


![](https://storage.googleapis.com/zenn-user-upload/1e827c8511a2-20231219.png)

![](https://storage.googleapis.com/zenn-user-upload/5bb0c8b73eac-20231219.png)

## Semantic Kernel バージョン履歴 (.NET 版)

日付|バージョン|URL
---|---|---
11/10(金)|dotnet-1.0.0-beta6|[GitHub](https://github.com/microsoft/semantic-kernel/releases/tag/dotnet-1.0.0-beta6)
11/16(木)|dotnet-1.0.0-beta7|[GitHub](https://github.com/microsoft/semantic-kernel/releases/tag/dotnet-1.0.0-beta7)
11/17(金)|dotnet-1.0.0-beta8|[GitHub](https://github.com/microsoft/semantic-kernel/releases/tag/dotnet-1.0.0-beta8)
12/05(火)|dotnet-1.0.0-rc1|[GitHub](https://github.com/microsoft/semantic-kernel/releases/tag/dotnet-1.0.0-rc1)
12/06(水)|dotnet-1.0.0-rc2|[GitHub](https://github.com/microsoft/semantic-kernel/releases/tag/dotnet-1.0.0-rc2)
12/07(木)|dotnet-1.0.0-rc3|[GitHub](https://github.com/microsoft/semantic-kernel/releases/tag/dotnet-1.0.0-rc3)
12/08(金)|(セッション収録日)|
12/14(木)|dotnet-1.0.0-rc4|[GitHub](https://github.com/microsoft/semantic-kernel/releases/tag/dotnet-1.0.0-rc4)
12/19(火)|(イベント開催日)|
12/19(火)|dotnet-1.0.1 [正式版]|[GitHub](https://github.com/microsoft/semantic-kernel/releases/tag/dotnet-1.0.1)

↑ セッション収録日 (12/08) の１週間で毎日更新されたの本当に運が…   
↑ (追記) イベント (放送日) 当日に正式版リリースされたの本当に運が…（当日朝にこの記事更新対応）


## 使い方 (正式版 v1.0.1)

インラインで書いたプラグインを使う例です。

前提：

* プロジェクトに、NuGet で Semantic Kernel のパッケージをインストールします。
* Azure OpenAI サービスに GPT-3.5 Turbo のモデルをデプロイしておく
* Azure CLI インストール済み (認証に必要)

### 01. kernel インスタンス構築

まず kernel のインスタンスを作ります。

↓ Azure OpenAI で GPT-3.5 turbo のモデルをたたく場合。認証には Azure の Managed ID を使用

```csharp
using Azure.Identity;
using Microsoft.SemanticKernel;

// Kernel builder を作る
var builder = Kernel.CreateBuilder();

// kernel が使う AI モデル情報,認証情報などを登録
builder.AddAzureOpenAIChatCompletion(
        deploymentName: "gpt-35-turbo",
        modelId: "gpt-35-turbo",
        endpoint: "https://エンドポイント.openai.azure.com/",
        credentials: new DefaultAzureCredential()
);

var kernel = builder.Build();
```

足りてないと怒られている `Azure.Identity` のパッケージは以下のようにもインストールすることができます。（`ctrl + .` から）

![](https://storage.googleapis.com/zenn-user-upload/7e2d5344d832-20231219.png)

:::message
認証の点で、もし Azure ManagedID ではなく API キーべた書きが良い方は、以下のコードに差し替えてください
```csharp
builder.AddAzureOpenAIChatCompletion(
        deploymentName: "自分で名付けたデプロイメントname",  
        modelId: "gpt-35-turbo", 
        endpoint: "https://エンドポイント.openai.azure.com/",
        apiKey: "APIキー" 
); 
```
:::

:::message
もし Azure OpenAI ではなく OpenAI 本家を使う場合は、こちらを使用ください
```csharp
builder.AddOpenAIChatCompletion(
    "gpt-3.5-turbo",  // OpenAI Model name
    "APIキー");       // OpenAI API Key
```
:::

:::message alert
RC3 時点での書き方は
```csharp
// 古い
var builder = new KernelBuilder();
```
だったので、.NET Conf Recap Japan イベントではこのように書いていますが、   
正式版 v1.0.1 だと動きません。   
本文中に書いたように   
```csharp
// 新しい
var builder = Kernel.CreateBuilder();
```
としてください。
:::

### 02. プロンプト登録

```csharp
var skPrompt = @"{{$input}}

上の文章を、なるべく短く要約してください。";
```

`@"{{$input}}` にユーザからの入力が入ります。

また、最大トークン数など諸々のパラメータの設定も行います

```csharp
var executionSettings = new OpenAIPromptExecutionSettings 
{
    MaxTokens = 2000,
    Temperature = 0.2,
    TopP = 0.5
};
```

### 03. 実行してみる

サンプル入力

```csharp
var input = """
吾輩は猫である。名前はまだ無い。
どこで生れたかとんと見当がつかぬ。何でも薄暗いじめじめした所でニャーニャー泣いていた事だけは記憶している。
吾輩はここで始めて人間というものを見た。
しかもあとで聞くとそれは書生という人間中で一番獰悪な種族であったそうだ。
この書生というのは時々我々を捕えて煮て食うという話である。
しかしその当時は何という考もなかったから別段恐しいとも思わなかった。
ただ彼の掌に載せられてスーと持ち上げられた時何だかフワフワした感じがあったばかりである。
掌の上で少し落ちついて書生の顔を見たのがいわゆる人間というものの見始であろう。
""";
```

実行します

```csharp
var summaryResult = await kernel.InvokeAsync(
    summaryFunction, 
    new KernelArguments() { ["input"] = input }
);

Console.WriteLine(summaryResult);
```

出力例）

> 猫の吾輩は、どこで生まれたかはわからないが、薄暗い場所で泣いていた記憶がある。初めて人間を見たのは書生という獰悪な人間で、煮て食うという話もあったが、当時は恐ろしいとは思わなかった。


:::message alert
RC3 時点での書き方は
```csharp
// 古い
var summaryResult = await kernel.InvokeAsync(
    summaryFunction, 
    new KernelArguments(input)
);
```
だったので、.NET Conf Recap Japan イベントではこのように書いていますが、   
最新の正式版 v1.01 だと動きません。   
本文中に書いたように   
```csharp
// 新しい
var summaryResult = await kernel.InvokeAsync(
    summaryFunction, 
    new KernelArguments() { ["input"] = input }
);
```
としてください。
:::

### 動かしてみた記事

なんと当日中（というかセッション中）に、ymd65536 さんが実際に上記のデモを動かしてくださいました！ありがとうございます！
Azure CLI が入ってなかったなど、つまずきポイントも書いてくださっています

https://zenn.dev/ymd65536/scraps/608eeaf9e8b652

## v1.0 正式リリースに向け絶賛開発中

https://github.com/orgs/microsoft/projects/866/views/57

![](https://storage.googleapis.com/zenn-user-upload/b0837baf9ab4-20231208.png)

![](https://storage.googleapis.com/zenn-user-upload/8a61dcabd729-20231208.png)

↑ 2023/12/08 15:20 現在 (RC-3 版) 、No.15 - No.37 までが「破壊的変更を含む可能性」のある変更点。正式リリースまでに破壊的変更が多分に含まれそう…


:::message
追記： 2023/12/19(火)

(日本時間) 12/19(火) 朝に正式版リリースされました   

https://github.com/microsoft/semantic-kernel/releases/tag/dotnet-1.0.1
:::


# [デモ] Semantic Kernel ハローワールド

Azure 側の設定も書いたらちょっと長くなったので別記事に書きました。

https://zenn.dev/chomado/articles/231207-semantic-kernel-rc3

:::message
追記： 2023/12/19(火)
↑ 正式版 v1.0.1 に対応しました
:::

# [デモ] Semantic Kernel chat bot

ちょっと長くなったので別記事に書きました。

https://zenn.dev/chomado/articles/231208-chatbot-semantic-kernel-rc3

:::message
追記： 2023/12/19(火)
↑ 正式版 v1.0.1 に対応しました
:::

# [デモ] Semantic Kernel Notebook

Semanic Kernel の基本的な動きを知るのにとても良いサンプルです。   

https://github.com/microsoft/semantic-kernel/tree/main/dotnet/notebooks

### ちなみに：バージョンについて

RC-4 版 (12/14 にリリース) で作られているのですが、   
基本的な動作は変わらないので、勉強には変わらず良い教材だと思われます。

![](https://storage.googleapis.com/zenn-user-upload/5982645dd849-20231219.png)

(ちなみにセッション収録した 12/8(金) の時点では Beta-6 版で動いていました。ちゃんとあれからアプデされてるんですね)

### 必要なもの

* VSCode に Polyglot Notebooks 拡張機能を入れる
  - Notebook で C# を動かすのに必要
* Azure OpenAI で API キー取得済み
  - (OpenAI 本家でも動くけど今回のデモは Azure でやります)

https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.dotnet-interactive-vscode

# おすすめサンプル集

https://github.com/Azure/AI-in-a-Box/tree/main/semantic-kernel-bot-in-a-box

# リファレンス

Microsoft 社 Cloud solution architect の Ryosuke Otaka さんによる Copilot Stack の記事 (2023/06/30)

https://zenn.dev/microsoft/articles/8974330fb534ef

Microsoft MVP くさばさんによる Semantic Kernel 入門動画 (2023/9/01)

https://youtu.be/kdPzg6rd9F4?si=Gr7YyNWBCRpOdYOz
