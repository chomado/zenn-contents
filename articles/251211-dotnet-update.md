---
title: "補足記事：AI ネイティブ開発を加速する .NET 10 と Visual Studio 2026 最新アップデート"
emoji: "📖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [CSharp, Azure, AzureFunctions, AI, Copilot]
published: true
publication_name: microsoft
---

2025/12/9 (火) - 12/11 (木) に オンライン開催された、  
日本マイクロソフト公式イベント『**Microsoft Ignite & 最新テクノロジー総まとめ Best of Microsoft Recap Days Japan**』にて

同じくマイクロソフトの C# オタクである井上章さんと二人で  
.NET update のセッション『**AI ネイティブ開発を加速する .NET 10 と Visual Studio 2026 最新アップデート**』をお話しさせていただきました。  

↓ イベント URL

https://www.microsoft.com/ja-jp/events/top/bmrj2025/

↓ セッション個別 URL

https://msevents.microsoft.com/event?id=52809505429

その補足記事がこちらです。 

# 注意

この記事は、あくまで（参考リンクなどを示した）「補足」記事であるので、

セッション自体の内容についてはアーカイブやスライドなどご参照ください。

# なぜ補足記事を書いたのか

理由は２つあります。

## 1. 事前収録なので、公開日までに何か更新があった場合に補足したい

このセッションは 12/3(水) に事前収録しましたが、   
セッション公開日は 12/11(木) となり、１週間以上開きます。

その間に何か更新があった場合に、もう収録済みの動画は編集は難しいのですが、記事だと補足が簡単ですので、記事を書きました。

## 2. 参考記事のリンクを示したい

セッション中でいくつか参考記事や公式アナウンス記事の URL をご案内しましたが、  
動画見ながら、その紹介された URL を直打ちするのは大変なので、   
ちゃんとリンクの形でお届けしたいと思い、フォローアップブログとして公開しようと思いました


# イベント概要

項目|値
---|---
イベント名|**Microsoft Ignite & 最新テクノロジー総まとめ Best of Microsoft Recap Days Japan**
主催|日本マイクロソフト株式会社
日程|2025/12/9 (火) - 12/11 (木)
イベント URL|https://www.microsoft.com/ja-jp/events/top/bmrj2025/
私のセッション名|**AI ネイティブ開発を加速する .NET 10 と Visual Studio 2026 最新アップデート**
登壇者|井上章（プリンシパル ソリューション エンジニア 兼 エバンジェリスト） ([@chack411](https://x.com/chack411)) <br />千代田まどか（Cloud Developer Advocate） ([@chomado]([https://x.com/chomado))
セッション URL|https://msevents.microsoft.com/event?id=52809505429

# セッション内容

目次

1. Visual Studio 2026
1. .NET 10
1. Microsoft Agent Framework
1. Durable task extension for Microsoft Agent Framework
1. MCP C# SDK
1. Aspire 13.0
1. GitHub Copilot app modernization
1. GitHub Copilot testing for .NET

## スライド

[ TODO: あとでスライド埋め込む]

## 1. Visual Studio 2026

11月11日（現地時間）、「Visual Studio 2026」が一般提供 (GA) されました。

最新の Visual Studio をダウンロードしよう！

https://visualstudio.microsoft.com/downloads/

### 参考記事

公式アナウンス記事『Visual Studio 2026 is here: faster, smarter, and a hit with early adopters』

https://devblogs.microsoft.com/visualstudio/visual-studio-2026-is-here-faster-smarter-and-a-hit-with-early-adopters/

VS 2022 とのパフォーマンスの比較などが書いてあります。

## 2. .NET 10

.NET 10 をインストールしよう！


(2025/12/10 現在、`SDK 10.0.1` が最新です (12/9 にリリースされました) (ちなみにセッション事前収録時は `10.0.0` が最新でした))

https://dotnet.microsoft.com/en-us/download/dotnet/10.0

Visual Studio 2026 には最初から .NET 10 が入っていますが、たとえば mac ユーザの方など、ぜひ↑のリンクから入れてみてください！

### 参考: .NET 10 の深淵

**.NET ランタイムガチ勢** の方は [こちらの記事『Performance Improvements in .NET 10 (.NET 10 の性能向上について)』](https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-10/) をどうぞ。（めちゃくちゃ長文）（毎回本当に本当に長くてビックリするので読まなくてもいいから一度ページ開いてみて欲しい）

https://devblogs.microsoft.com/dotnet/performance-improvements-in-net-10/

ざっくりとした要約

* .NET 10 は「最速の .NET」として、ランタイム、JIT コンパイラ、ライブラリなどにわたる多数の小さな最適化を積み重ねることで、全体のパフォーマンスを大きく改善。
  - JIT コンパイラの強化
  - 仮想呼び出し・インターフェースの最適化（「Devirtualization」）
  - Tiered Compilation（ティアードコンパイルの強化）
  - などなど
* 開発者が書いた既存の C# コードをほとんど変更せずとも、.NET 10 に対応するだけで、高速化などの恩恵を受けられるようになっている。

### .NET サポートポリシー


![.NET support](https://dotnet.microsoft.com/blob-assets/images/illustrations/release-schedule-dark.svg)

https://dotnet.microsoft.com/ja-jp/platform/support/policy/dotnet-core

今回の .NET 10 は LTM (ロングタームサポート) なので、最初のリリースから 3 年間サポートされます。

ちなみに、STS (Standard Term Support: 標準期間サポート)　は、以前は 1.5 年間でしたが、.NET 9 から 2 年間と変更になりました。

### 活発な OSS ランキング

https://github.com/cncf/velocity/blob/main/reports/top_projects_by_activity.txt

Top projects by activity:

1) `pytorch` (pytorch.org) activity: 573780
2) `Linux` (kernel.org) activity: 536855
3) `Kubernetes` (kubernetes.io) activity: 493882
4) `dotnet` (microsoft.com/net) activity: 423380　← やった〜！
5) `LLVM` (llvm.org) activity: 390025

## 3. Microsoft Agent Framework

### Microsoft Agent Framework とは？

**Microsoft Agent Framework** は、Microsoft が OSS として提供する AI エージェント & マルチエージェントワークフローを開発する フレームワークです

https://learn.microsoft.com/ja-jp/agent-framework/overview/agent-framework-overview


対応言語： C#, Python

オープンソースで開発されています

https://github.com/microsoft/agent-framework

### おすすめ記事

#### 『Microsoft Agent Framework開発入門』(最終更新日 2025/10/31)

https://qiita.com/matayuuu/items/e028904453dded34d18f

マイクロソフト社員の Matayoshi さんによる記事。図がたくさんで大変わかりやすい。

#### 『「雑感」とハローワールド - Microsoft Agent Framework (C#) その1』(最終更新日 2025/11/25)

https://zenn.dev/microsoft/articles/agent-framework-001

マイクロソフト社員のかずきさんによる Agent Framework シリーズ記事。  
12/11 現在、「[その19](https://zenn.dev/microsoft/articles/agent-framework-019)」まで存在する。瞬きしてる間に良質な記事が増えてる。かずきさんすごい。   
Agent Framework 極めたい方はぜひ。


### 公式のクイックスタート

https://learn.microsoft.com/ja-jp/agent-framework/tutorials/quick-start?pivots=programming-language-csharp


## 4. Durable task extension for Microsoft Agent Framework

Microsoft Agent Framework の拡張フレームワーク的な位置付け。

個人的に私の爆推し。一緒に推してくださいお願いします。

７年+ の実績を誇る、[Azure Durable Functions](https://github.com/Azure/azure-functions-durable-extension) の、さまざまな本番運用に耐え続けている「堅牢性」(サーバの突然のクラッシュや再起動に耐えられる、スケール(イン|アウト)してくれる、human-in-the-loop 対応、など) を、  
Agent workflow 構築にも適用したもの。   

なので "ポッと出" ではないので安心できる。（先月出たばかりで一応まだ public preview という位置付けだけど、中で使っている技術は "枯れた技術" (もちろん良い意味よ) です。）

一応今までも「Durable Functions 使えば良い感じの堅牢な Agent workflows 組めるのでは？🤔」と思ったゴリゴリのつよつよエンジニアたちが、    
それでマルチエージェントのワークフロー組んで本番運用したりしてたけど、   

今回は、公式がそれのエージェント最適化版を作って、Microsoft Agent Framework の拡張という形で出してきた感じ。やったぜ。


### 公式アナウンス記事

『Bulletproof agents with the durable task extension for Microsoft Agent Framework』

https://techcommunity.microsoft.com/blog/appsonazureblog/bulletproof-agents-with-the-durable-task-extension-for-microsoft-agent-framework/4467122

の日本語訳の記事『Durable Task Extension for Microsoft Agent Framework で、堅牢なエージェントを構築する』

https://techcommunity.microsoft.com/blog/azuredevcommunityblog/durable-task-extension-for-microsoft-agent-framework-%E3%81%A7%E3%80%81%E5%A0%85%E7%89%A2%E3%81%AA%E3%82%A8%E3%83%BC%E3%82%B8%E3%82%A7%E3%83%B3%E3%83%88%E3%82%92%E6%A7%8B%E7%AF%89%E3%81%99%E3%82%8B/4470826

（↑ 手前味噌ですが私が翻訳した記事です。）

### おすすめサンプル集

12/11 現在、シナリオ別に 7 つ (Python 版は 8 つ) のサンプルがあります

C#

https://github.com/microsoft/agent-framework/tree/main/dotnet/samples/AzureFunctions

Python

https://github.com/microsoft/agent-framework/tree/main/python/samples/getting_started/azure_functions

### 参照セッション

Microsoft Ignite でのセッション
「Orchestrate distributed AI Agents with Azure Functions」

https://youtu.be/pSBrgsmB-zs?si=qH1A3qKOxI1BYpJP

↑ 製品作者とプロマネが登壇！熱い！（ちなみにクリスさんは日本語喋れます。めちゃくちゃ丁寧な日本語を話します。プログラミング言語だけじゃなく自然言語もプロいのすごい）

### おすすめ記事

#### 『Durable Agent を試してみよう - Microsoft Agent Framework (C#) その14』

https://zenn.dev/microsoft/articles/agent-framework-014


## 5. MCP C# SDK

『MCP C# SDK』のリポジトリ

https://github.com/modelcontextprotocol/csharp-sdk

おれらの大好きな C# で MCP サーバ作るぞ〜！ってときにどうぞ！

## 6. Aspire 13.0

ずっと「.NET Aspire」だったのが、今回名前から ".NET" が外れて "Aspire" になりました

https://aspire.dev/

## 7. GitHub Copilot app modernization

GitHub Copilot を用いて、古いコードを新しいバージョンへと自動的に変換する機能

『GitHub Copilot で .NET アプリを手間なく最新化』

https://dotnet.microsoft.com/ja-jp/platform/modernize

『Upgrade a .NET app with GitHub Copilot app modernization』

https://learn.microsoft.com/en-us/dotnet/core/porting/github-copilot-app-modernization/how-to-upgrade-with-github-copilot


### 参考動画

『Modernize .NET apps in days with GitHub Copilot』(2025/09/24)

https://www.youtube.com/watch?v=-YKguff5GY8

## 8. GitHub Copilot testing for .NET

GitHub Copilot くんがガリガリテストを書いてくれて、カバレッジなどのレポートもしっかり出してくれる新機能『**GitHub Copilot testing for .NET**』です

12/11 現在、Visual Studio 2026 Insiders ビルドで利用可能です。

『Overview of GitHub Copilot testing for .NET』

https://learn.microsoft.com/ja-jp/visualstudio/test/github-copilot-test-dotnet-overview?view=visualstudio

『Supercharge Your Test Coverage with GitHub Copilot Testing for .NET』

https://devblogs.microsoft.com/dotnet/github-copilot-testing-for-dotnet/

