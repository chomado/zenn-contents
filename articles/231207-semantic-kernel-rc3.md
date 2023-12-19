---
title: "Semantic Kernel (正式版 v1.0.1) ハローワールドを日本語で動かす"
emoji: "⛏️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [CSharp, Azure, OpenAI, SemanticKernel]
published: true
---
公式ドキュメントのハローワールドを  
(今日 2023/12/19(金) 現在の) 最新版 ([今日リリースされたばかりの 正式版 v1.0.1](https://github.com/microsoft/semantic-kernel/releases/tag/dotnet-1.0.1)) で  
日本語で動かします。

:::message
追記：12/19(金)   
この記事はもともと 12/07(木) に RC-3 版が出た当日に RC-3 版で書いたもの（なので URL に rc3 と入っている）でしたが、     
今日 12/19(金) に正式版がリリースされたので、そちらでアップデートして書き直しております。
:::

↓ 動かすコード:公式ドキュメント（12/19 現在、ドキュメントが古いままでこれ↓だと動かない（そのうち更新されると思う（まあ更新されたらこの記事の存在意義消えるけど、、、）））

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
dotnet add package Microsoft.SemanticKernel
```

画面でやる場合は、プロジェクト名右クリックで「Manage NuGet packages」、
`include prerelease` にチェックを入れた状態で
`SemanticKernel` で検索、
「install」クリック

![](https://storage.googleapis.com/zenn-user-upload/d769af1f6a0f-20231219.png)

## 入力したパラメータを含んだプロンプト実行

[公式ドキュメント] に書いてあるコードは
2023年12月7日現在、古くて動かない（コンパイルエラー）ので、
RC3 で動く版はこちらでお願いします。
（[この修正プルリク](https://github.com/microsoft/semantic-kernel/pull/4077/commits/9f600b37d01fffc71f39ae7fb207bdf13772f1dc)がマージされたら公式ドキュメントのコード動くようになります）

```csharp
using Azure.Identity;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.Connectors.OpenAI;

var builder = Kernel.CreateBuilder();

builder.AddAzureOpenAIChatCompletion(
        deploymentName: "gpt-35-turbo",
        modelId: "gpt-35-turbo",
        endpoint: "https://エンドポイント.openai.azure.com/",
        credentials: new DefaultAzureCredential()
);

var kernel = builder.Build();

var prompt = @"{{$input}}

なるべく短く要約してください。";

var summarize = kernel.CreateFunctionFromPrompt(
    promptTemplate: prompt, 
    executionSettings: new OpenAIPromptExecutionSettings { MaxTokens = 200 }
);

string text1 = @"
吾輩は猫である。名前はまだ無い。
どこで生れたかとんと見当がつかぬ。何でも薄暗いじめじめした所でニャーニャー泣いていた事だけは記憶している。
吾輩はここで始めて人間というものを見た。
しかもあとで聞くとそれは書生という人間中で一番獰悪な種族であったそうだ。
この書生というのは時々我々を捕えて煮て食うという話である。
しかしその当時は何という考もなかったから別段恐しいとも思わなかった。
ただ彼の掌に載せられてスーと持ち上げられた時何だかフワフワした感じがあったばかりである。
掌の上で少し落ちついて書生の顔を見たのがいわゆる人間というものの見始であろう。
";

string text2 = @"
メロスは激怒した。必ず、かの邪智暴虐の王を除かなければならぬと決意した。
メロスには政治がわからぬ。メロスは、村の牧人である。笛を吹き、羊と遊んで暮して来た。
けれども邪悪に対しては、人一倍に敏感であった。
きょう未明メロスは村を出発し、野を越え山越え、十里はなれた此このシラクスの市にやって来た。
メロスには父も、母も無い。女房も無い。十六の、内気な妹と二人暮しだ。この妹は、村の或る律気な一牧人を、近々、花婿として迎える事になっていた。結婚式も間近かなのである。
メロスは、それゆえ、花嫁の衣裳やら祝宴の御馳走やらを買いに、はるばる市にやって来たのだ。
先ず、その品々を買い集め、それから都の大路をぶらぶら歩いた。メロスには竹馬の友があった。セリヌンティウスである。今は此のシラクスの市で、石工をしている。
その友を、これから訪ねてみるつもりなのだ。久しく逢わなかったのだから、訪ねて行くのが楽しみである。
";

Console.WriteLine("----- text1 --------");

Console.WriteLine(await kernel.InvokeAsync(summarize, new KernelArguments() { ["input"] = text1 }));

Console.WriteLine("----- text2 --------");

Console.WriteLine(await kernel.InvokeAsync(summarize, new KernelArguments() { ["input"] = text2 }));
```

:::message 
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

### アウトプット例

![](https://storage.googleapis.com/zenn-user-upload/08329d8e3685-20231207.png)

```
----- text1 --------
猫である吾輩は、名前がまだない。生まれた場所や状況は覚えていないが、最初に人間を見たのは書生と呼ばれる獰悪な人間だった。当時は人間の恐ろしさについて考える余裕はなかったが、書生の手の上で落ち着いてから、人間というものを見始めた。
----- text2 --------
メロスは激怒し、邪智暴虐の王を倒すことを決意する。彼は政治には疎く、村の牧人であるが、邪悪には敏感だ。村を出てシラクスの市に向かい、妹の結婚のための品々を買い集める。そして、かつての友人である石工のセリヌンティウスを訪ねることを楽しみにしている。
```


## 余談

Pull Request 出そうと思ったら  
1 時間前に出されていた

https://twitter.com/chomado/status/1732728756698222959

**正直くっそ悔しい**

（推しプロジェクトのコントリビューターに名前が載るのは全エンジニアの夢…！）
