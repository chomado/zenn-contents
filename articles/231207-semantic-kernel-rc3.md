---
title: "Semantic Kernel (RC-3 版) ハローワールド"
emoji: "⛏️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [CSharp, Azure, OpenAI, SemanticKernel]
published: true
---
公式ドキュメントのハローワールドを  
(今日 2023/12/07 現在の) 最新版 ([今日リリースされたばかりの RC-3](https://github.com/microsoft/semantic-kernel/releases/tag/dotnet-1.0.0-rc3)) で  
日本語で動かします。

https://github.com/microsoft/semantic-kernel/blob/main/dotnet/README.md

## Azure OpenAI 側の準備

### Managed ID の設定

[元記事](https://github.com/microsoft/semantic-kernel/blob/main/dotnet/README.md) では API キーをクライアントにベタ書きしていました (おそらく OpenAI 本家との互換性のためにそうしてるんだと思う) が、

せっかく Azure を使っているので、
もっとセキュアに認証できたらと思うので、今回は Managed ID を使ってみましょう。

(もし「(俺|私) は API キーべた書きがいいんじゃ」信者の方はこのセクションは読み飛ばしてください。後述のコードのほうで一応 API キーべた書き版のコードも載せておいています)

Azure ポータルで、
Azure OpenAI リソースの「アクセス制御 (IAM)」を開きます。
(IAM: Identity and Access Management)

「＋追加」→「ロールの割り当て」

![Azure OpenAI マネージドID](https://storage.googleapis.com/zenn-user-upload/aff2dc82335f-20231207.png)

自分を検索します。
この時、この後使う Visual Studio でログインしているアカウントを選んでください。

![Azure OpenAI マネージドID](https://storage.googleapis.com/zenn-user-upload/6e11b4c6f680-20231207.png)

「レビューと割り当て」をクリックします。

## Visual Studio 側での作業

まず適当に console app のプロジェクトを new しましょう。（ターゲットは .NET 8 を指定しました）

### NuGet パッケージを入れる

CLI

```shell
dotnet add package Microsoft.SemanticKernel --prerelease
```

画面でやる場合は、プロジェクト名右クリックで「Manage NuGet packages」、
`include prerelease` にチェックを入れた状態で
`SemanticKernel` で検索、
「install」クリック

![](https://storage.googleapis.com/zenn-user-upload/9342f8c5234d-20231207.png)

## 入力したパラメータを含んだプロンプト実行

[公式ドキュメント] に書いてあるコードは
2023年12月7日現在、古くて動かない（コンパイルエラー）ので、
RC3 で動く版はこちらでお願いします。
（[この修正プルリク](https://github.com/microsoft/semantic-kernel/pull/4077/commits/9f600b37d01fffc71f39ae7fb207bdf13772f1dc)がマージされたら公式ドキュメントのコード動くようになります）

```csharp
using Azure;
using Azure.Identity;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.Connectors.AI.OpenAI;

var builder = new KernelBuilder();

builder.AddAzureOpenAIChatCompletion(
        deploymentName: "gpt-35-turbo",                // Azure OpenAI Deployment Name
        modelId: "gpt-35-turbo",                      // Azure OpenAI Model Name
        endpoint: "https://openai-231207.openai.azure.com/", // Azure OpenAI Endpoint
        credentials: new DefaultAzureCredential()
);      

var kernel = builder.Build();

var prompt = @"{{$input}}

One line TLDR with the fewest words.";

var summarize = kernel.CreateFunctionFromPrompt(prompt, executionSettings: new OpenAIPromptExecutionSettings { MaxTokens = 100 });

string text1 = @"
1st Law of Thermodynamics - Energy cannot be created or destroyed.
2nd Law of Thermodynamics - For a spontaneous process, the entropy of the universe increases.
3rd Law of Thermodynamics - A perfect crystal at zero Kelvin has zero entropy.";

string text2 = @"
1. An object at rest remains at rest, and an object in motion remains in motion at constant speed and in a straight line unless acted on by an unbalanced force.
2. The acceleration of an object depends on the mass of the object and the amount of force applied.
3. Whenever one object exerts a force on another object, the second object exerts an equal and opposite on the first.";

Console.WriteLine(await kernel.InvokeAsync(summarize, new KernelArguments(text1)));

Console.WriteLine(await kernel.InvokeAsync(summarize, new KernelArguments(text2)));

// Output:
//   Energy conserved, entropy increases, zero entropy at 0K.
//   Objects move in response to forces.
```

:::message API キーべた書き版
認証の点で、もし Azure ManagedID ではなく API キーべた書きが良い方は、以下のコードに差し替えてください。

```csharp
builder.AddAzureOpenAIChatCompletion(
        deploymentName: "自分で名付けたデプロイメントname",  
        modelId: "gpt-35-turbo", 
        endpoint: "https://エンドポイント.openai.azure.com/",
        apiKey: "APIキー" 
); 
```
:::

## 余談

Pull Request 出そうと思ったら  
1 時間前に出されていた

https://twitter.com/chomado/status/1732728756698222959
