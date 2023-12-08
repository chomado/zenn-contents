---
title: ".NET Conf Japan 2023「.NET + AI」補足記事"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [SemanticKernel, CSharp, Azure]
published: true
---

2023/12/19(火) に **.NET Conf 2023 Recap Japan** オンラインイベントが開催され、  
私も「.NET (C#) + Azure OpenAI」についてお話しします。

その補足記事です。

![](https://storage.googleapis.com/zenn-user-upload/3451058409dd-20231208.png)

# なぜ補足記事を書いたのか

このイベントの全セッションは「事前収録」のものとなっており、   
収録日 2023/12/08(金) 時点 (バージョンは RC-3 版) での最新版での話となります。

私のトピック AI は情報の移り変わりが早く、   
とくに (後述する) LLM 用ライブラリ **Semantic Kernel は今週だけで 3 回パッケージの更新** （しかも **破壊的変更まみれ** で既存のコードがコンパイルエラーまみれで動かなくなった）があり、  
**収録から 1 週間以上経つと、当日「セッションで紹介されたコードが動かない！」ということが普通にあり得る** ので、   
この、いつでも手元で更新できる補足記事を書きました。

この記事では 2023/12/19(火) 時点での最新の情報を書いておくように更新し続けます。

# セッションアジェンダ

1. Azure OpenAI
    * ポータル画面お見せする
1. Semantic Kernel とは
    * [Semantic Kernel の開発タスク一覧](https://github.com/orgs/microsoft/projects/866/views/57) をお見せする。現在 15-37 までが破壊的変更を含む可能性
1. [デモ] .NET 8 + Semantic Kernel ハローワールド
1. [デモ] Notebook サンプルを動かす
1. 各種サンプル紹介
    * [Semantic Kernel Bot in-a-box](https://github.com/Azure/AI-in-a-Box/tree/main/semantic-kernel-bot-in-a-box) (SK beta-8)
1. まとめ
    * V1.0.0 正式リリースされたらぜひ使ってみてくださいね！

# Azure OpenAI

公式ドキュメント

https://learn.microsoft.com/ja-jp/azure/ai-services/openai?WT.mc_id=dotnet-9827-machiy

> Azure OpenAI Service は、GPT-4、GPT-3.5-Turbo、Embeddings (埋め込み) モデル シリーズなど OpenAI の強力な言語モデルに、REST API でのアクセスを提供します。


# Semantic Kernel とは

Semantic Kernel とは、    

* OpenAI, Azure OpenAI などの大規模言語モデル (LLM) 用のライブラリ。
* Microsoft が中心となり OSS で開発


### 公式から引用

> Semantic Kernel is an SDK that integrates Large Language Models (LLMs) like OpenAI, Azure OpenAI, and Hugging Face with conventional programming languages like C#, Python, and Java. Semantic Kernel achieves this by allowing you to define plugins that can be chained together in just a few lines of code.

https://github.com/microsoft/semantic-kernel

## Semantic Kernel バージョン履歴 (.NET 版)

日付|バージョン|URL
---|---|---
23/11/10|dotnet-1.0.0-beta6|[GitHub](https://github.com/microsoft/semantic-kernel/releases/tag/dotnet-1.0.0-beta6)
23/11/16|dotnet-1.0.0-beta7|[GitHub](https://github.com/microsoft/semantic-kernel/releases/tag/dotnet-1.0.0-beta7)
23/11/17|dotnet-1.0.0-beta8|[GitHub](https://github.com/microsoft/semantic-kernel/releases/tag/dotnet-1.0.0-beta8)
23/12/05|dotnet-1.0.0-rc1|[GitHub](https://github.com/microsoft/semantic-kernel/releases/tag/dotnet-1.0.0-rc1)
23/12/06|dotnet-1.0.0-rc2|[GitHub](https://github.com/microsoft/semantic-kernel/releases/tag/dotnet-1.0.0-rc2)
23/12/07|dotnet-1.0.0-rc3|[GitHub](https://github.com/microsoft/semantic-kernel/releases/tag/dotnet-1.0.0-rc3)

↑ セッション収録日 (12/08) の１週間で毎日更新されるの本当に運が…


## 使い方

インラインで書いたプラグインを使う例です。

前提：プロジェクトに、NuGet で Semantic Kernel のパッケージをインストールします。

### 01. kernel インスタンス構築

まず kernel のインスタンスを作ります。

↓ Azure OpenAI で GPT-3.5 turbo のモデルをたたく場合。認証には Azure の Managed ID を使用

```csharp
using Azure;
using Azure.Identity;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.Connectors.AI.OpenAI;

// Kernel builder を作る
var builder = new KernelBuilder();

// kernel が使う AI モデル情報,認証情報などを登録
builder.AddAzureOpenAIChatCompletion(
        deploymentName: "gpt-35-turbo",
        modelId: "gpt-35-turbo",
        endpoint: "https://エンドポイント.openai.azure.com/",
        credentials: new DefaultAzureCredential()
);

var kernel = builder.Build();
```

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

プロンプトテンプレートの設定をします。

```csharp
var promptTemplateConfig = new PromptTemplateConfig(skPrompt);

var promptTemplateFactory = new KernelPromptTemplateFactory();
var promptTemplate = promptTemplateFactory.Create(promptTemplateConfig);

var renderedPrompt = await promptTemplate.RenderAsync(kernel);

Console.WriteLine(renderedPrompt);
```

上記のプロンプトテンプレートを、kernel が実行できる形式の function に変換しましょう。

```csharp
var summaryFunction = kernel.CreateFunctionFromPrompt(skPrompt, executionSettings);
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
var summaryResult = await kernel.InvokeAsync(summaryFunction, new KernelArguments(input));

Console.WriteLine(summaryResult);
```

出力例）

> 猫の吾輩は、どこで生まれたかはわからないが、薄暗い場所で泣いていた記憶がある。初めて人間を見たのは書生という獰悪な人間で、煮て食うという話もあったが、当時は恐ろしいとは思わなかった。


## v1.0 正式リリースに向け絶賛開発中

https://github.com/orgs/microsoft/projects/866/views/57

![](https://storage.googleapis.com/zenn-user-upload/b0837baf9ab4-20231208.png)

![](https://storage.googleapis.com/zenn-user-upload/8a61dcabd729-20231208.png)

↑ 2023/12/08 15:20 現在 (RC-3 版) 、No.15 - No.37 までが「破壊的変更を含む可能性」のある変更点。正式リリースまでに破壊的変更が多分に含まれそう…


# [デモ] Semantic Kernel ハローワールド

Azure 側の設定も書いたらちょっと長くなったので別記事に書きました。

https://zenn.dev/chomado/articles/231207-semantic-kernel-rc3


# [デモ] Semantic Kernel chat bot

ちょっと長くなったので別記事に書きました。

https://zenn.dev/chomado/articles/231208-chatbot-semantic-kernel-rc3

# [デモ] Semantic Kernel Notebook

Semanic Kernel の基本的な動きを知るのにとても良いサンプルです。   

### ちなみに：バージョンについて

Beta-6 版 (11/10 にリリース) で作られているのですが、   
基本的な動作は変わらないので、勉強には変わらず良い教材だと思われます。

ちなみに もしこの pull request (Update notebooks to use RC version) が merge されたら RC-3 対応されます

https://github.com/microsoft/semantic-kernel/pull/4077/commits/9efda6b4c4dba57c7c5eb88331659a9febcd30e1

### 必要なもの

* VSCode に Polyglot 拡張機能を入れる
  - Notebook で C# を動かすのに必要
* Azure OpenAI で API キー取得済み
  - (OpenAI 本家でも動くけど今回のデモは Azure でやります)

# おすすめサンプル集

https://github.com/Azure/AI-in-a-Box/tree/main/semantic-kernel-bot-in-a-box

# リファレンス

Microsoft 社 Cloud solution architect の Ryosuke Otaka さんによる Copilot Stack の記事 (2023/06/30)

https://zenn.dev/microsoft/articles/8974330fb534ef

Microsoft MVP くさばさんによる Semantic Kernel 入門動画 (2023/9/01)

https://youtu.be/kdPzg6rd9F4?si=Gr7YyNWBCRpOdYOz
