---
title: "Semantic Kernel (RC-3) + GPT 3.5 turbo で chat bot"
emoji: "💻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [SemanticKernel, CSharp]
published: true
---

Azure OpenAI では GPT 4 turbo も使えるのですが、   
今回は (お金をケチって) GPT 3.5 turbo を使用しています。

Semantic Kernel は 2023/12/08 現在最新の RC-3 を使用しています。   
今までのコードが（破壊的変更により）コンパイルエラー祭りで動かなくなったので、最新版で動くやつを書きました。

また、認証は（API キーではなく）Azure Managed ID を使用しています

```csharp
using Azure;
using Azure.Identity;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.Connectors.AI.OpenAI;

var builder = new KernelBuilder();

builder.AddAzureOpenAIChatCompletion(
        deploymentName: "gpt-35-turbo",
        modelId: "gpt-35-turbo",
        endpoint: "https://エンドポイント.openai.azure.com/",
        credentials: new DefaultAzureCredential()
);

var kernel = builder.Build();

const string skPrompt = @"
チャットボットはあなたと楽しく会話ができます。
明確な指示を与えることもできますし、答えがない場合は「わかりません」と言うこともできます。

{{$history}}

あなた: {{$userInput}}
ChatBot:";

var executionSettings = new OpenAIPromptExecutionSettings
{
    MaxTokens = 2000,
    Temperature = 0.7,
    TopP = 0.5
};

var chatFunction = kernel.CreateFunctionFromPrompt(skPrompt, executionSettings);

var history = "";
var arguments = new KernelArguments()
{
    ["history"] = history
};

while (true)
{
    Console.Write("あなた: ");
    var userInput = Console.ReadLine();
    arguments["userInput"] = userInput;

    var bot_answer = await chatFunction.InvokeAsync(kernel, arguments);
    Console.WriteLine($"ChatBot: {bot_answer}\n");

    history += $"あなた: {userInput}\nChatBot: {bot_answer}\n";
    arguments["history"] = history;
    // Console.WriteLine(history);
}

## 実行結果例

```
![](https://storage.googleapis.com/zenn-user-upload/e12c5af245a8-20231208.png)

あなた: こんにちは。おすすめの本を探しています。

ChatBot: こんにちは。おすすめの本を探しているのですね。ジャンルや興味のあるテーマなど、具体的な 希望があれば教えていただけますか？

あなた: ワクワクする感じのをお願いします

ChatBot: 「ワクワクする感じ」の本というと、冒険やファンタジー、サスペンスなどが含まれるかもしれ ませんね。例えば、ジョー・アバーシリーの『ミステリー・マンション』やJ.K.ローリングの『ハリー・ポッターシリーズ』などがおすすめです。興味があるジャンルや具体的な希望があれば教えていただけますか？

あなた: ハリーポッターいいですねえ

ChatBot: はい、ハリーポッターシリーズは多くの人に愛されている人気のある本ですね。魔法や冒険、友 情などがテーマとなっており、多くの読者を魅了しています。もし他にもおすすめの本があれば、お知らせいたしますので、お気軽にお尋ねください。

あなた: ありがとうございました。

ChatBot: どういたしまして。他にも何かお手伝いできることがあれば、お知らせくださいね。

