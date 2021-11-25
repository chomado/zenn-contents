---
title: "無料で手早く作る：Teams で動く FAQ bot 開発 [後編] (2021 年 12 月版)"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [CSharp, Azure, Bot, BotFramework, VisualStudio]
published: true
---

![](https://storage.googleapis.com/zenn-user-upload/eao06tepir342vqj6wgw09vxtiij)

[前編の記事](https://zenn.dev/chomado/articles/8ab50af04b52cc) では、QnA Maker でのナレッジベース作成まで完了しましたね。
この後半の記事ではクライアント (bot) のコードをガリガリ書いていきます。

0. ~~事前準備~~
1. ~~Azure で bot ホスト先を用意~~
2. ~~QnA Maker でナレッジベースを作る~~ `← ここまで完了`
3. bot クライアント開発 (Visual Studio で) `← ここから`
4. Teams と繋げる

# 3: bot クライアント開発

## 3-1: Visual Studio に Botframework テンプレート

Visual Studio 2019 の拡張機能から Bot Framework v4 SDK Templates for Visual Studio を入れます。詳しく手順を示します。

まず VS2019 を開いて、「Continue without code」をクリック。（私は VS の言語を英語にしているので、日本語にしている方は適宜読み替えてください）

![](https://storage.googleapis.com/zenn-user-upload/dx2khs0qw0woz7wpdd9nxjev81sv)

メニューバーから `Extentions` (拡張機能) → `Manage Extentions`

![](https://storage.googleapis.com/zenn-user-upload/n2x0aglj59phh0u2vsfw6dsmcvtp)

左のタブの「Online」を選択した状態で検索バーに `Bot Framework v4 SDK Templates` を入力しましょう。



クエリが走り、一番 上の「`Bot Framework v4 SDK Templates for Visual Studio`」を `Download` したら、VS2019 を再起動します。

## 3-2: プロジェクト作成

Visual Studio 2019 の `Create a new project` (新規作成) から 
Echo Bot (Bot Framework v4 - .NET Core 3.1) を選択して作ります。

![](https://storage.googleapis.com/zenn-user-upload/gn2jwp9maixwotack1ylslxxqmts)

プロジェクト名は任意で。（私は `MyQnABot` にしました）

![](https://storage.googleapis.com/zenn-user-upload/ty89lytpdibvb4dklaactgn2g1ba)

「`Create`」(作成) を押します。

## 3-3: 実行してみる

プロジェクトが開くので、取りあえずなにもいじらずデバッグ実行してみます。(`F5` 押して実行)

![](https://storage.googleapis.com/zenn-user-upload/x70qoe7esxbziemnsx53qfjbm1jd)

web ブラウザが立ち上がります。`http://localhost:3978/`

![](https://storage.googleapis.com/zenn-user-upload/ta4c9ldzzhumzu7srm6x5uj9whbk)

## 3-4: Bot Framewrork Emulator で実行

エミュレータ (`Bot Framewrork Emulator`) を開きます。

![](https://storage.googleapis.com/zenn-user-upload/kqd8tgfqdcvcxnmumyrr0y18d7qg)

「`Open Bot`」ボタンをクリック

Bot URL (エンドポイント) に `http://localhost:3978/api/messages` を入力し「Connect」

![](https://storage.googleapis.com/zenn-user-upload/p98syl55ky5eo98lrgrf6ej1285y)

チャット画面が開きます。

![](https://storage.googleapis.com/zenn-user-upload/oyoi3zn86jdzzuhqp6qkmtdgsrki)

開いたとき、最初は `Hello and welcome!` の定型文だけ返してきていますが、
こちらが何か言うと、その送った文字列そのままを返す、という動きをしています。（エコー bot）

## 3-5: QnA Maker に繋げる準備：接続情報を記述

プロジェクトの設定ファイル `appsettings.json` に QnA Maker の設定を書きます。

QnA Maker の Publish 画面にある以下の *** で囲んだ部分の値を入れる

```
POST /knowledgebases/***00000000-0000-0000-0000-00000000000***/generateAnswer
Host: ***https://qna-m365kaota.azurewebsites.net/qnamaker***
Authorization: EndpointKey ***11111111-1111-1111-1111-11111111111***
Content-Type: application/json
{"question":"<Your question>"}
```

![](https://storage.googleapis.com/zenn-user-upload/4htrf9uluqlmmfossclf98192x7k)

`appsettings.json` サンプル

````json
{
  "MicrosoftAppId": "",
  "MicrosoftAppPassword": "",
  "QnAKnowledgebaseId": "00000000-0000-0000-0000-00000000000",
  "QnAEndpointKey": "11111111-1111-1111-1111-11111111111",
  "QnAEndpointHostName": "https://自分でつけた名前.azurewebsites.net/qnamaker"
}
````


（今回はサンプルなので構成ファイルに直書きしますが、本番では Azure `App Service` の `アプリケーション設定` に書くほうが安全です）

## 3-6: プロジェクトに QnA Maker の SDK 入れる

プロジェクトに `Microsoft.Bot.Builder.AI.QnA` を NuGet (パッケージマネージャー) から入れます。詳しい方法を以下に書きます。

プロジェクト右クリックで「`Manage NuGet Packages`」

![](https://storage.googleapis.com/zenn-user-upload/z76lxcenpy9o6cxgn6vohtot23kp)

`Browse` タブの検索バー `Microsoft.Bot.Builder.AI.QnA` 入れて出てきたパッケージをインストールします。

![](https://storage.googleapis.com/zenn-user-upload/sqttgrk2fbbh6tq7birnec16c8ga)

## 3-7: bot のコードを書いていく

### 3-7-1: Startup.cs

Bot から `QnA Maker` に接続するために `HttpClient` が必要なので
`Startup.cs` に `HttpClient` を使うための
下準備のコード `services.AddHttpClient();` を追加します

```csharp
public void ConfigureServices(IServiceCollection services)
        {
            // Http Client を使うため
            services.AddHttpClient();
```
### 3-7-2: EchoBot.cs: QnA Maker を呼び出し

Bots フォルダの下の `EchoBot.cs` に bot のクラスがあります。

ここの `OnMessageActivityAsync` メソッドが
ユーザーから話しかけられた時に呼ばれるメソッドになります。

なのでここで `QnA Maker` を呼び出せば QA bot が作れます。

まず、必要なものを EchoBot で使えるようにしましょう。

コンストラクタを作って ASP.NET Core から必要なものを渡してもらいます。

今回は QnA Maker への接続情報と HttpClient を作るための IHttpClientFactory を受け取ります。

まず using を追加します

```csharp
using Microsoft.Extensions.Configuration;
using System.Net.Http;
```

次にコンストラクタを書いていきます

```csharp
private readonly IConfiguration _configuration;
private readonly IHttpClientFactory _httpClientFactory;

public EchoBot(IConfiguration configuration, IHttpClientFactory httpClientFactory)
{
   _configuration = configuration;
   _httpClientFactory = httpClientFactory;
}
```
次に、`OnMessageActivityAsync` メソッドを置き換えていきます。
足りない using は適宜インテリセンスで追加していってください


![](https://storage.googleapis.com/zenn-user-upload/i8kbp8f2pkik8k5hg5zkck26g9tx)


```csharp
// ユーザーから話しかけられた時に呼ばれるメソッド。
// なのでここで QnA Maker を呼び出す
protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{
    // QnA Maker に接続するためのクライアントを作る
    var qnaMaker = new QnAMaker(new QnAMakerEndpoint
        {
            // appsetting.json に書いた設定項目 
            KnowledgeBaseId = _configuration["QnAKnowledgebaseId"],
            EndpointKey = _configuration["QnAEndpointKey"],
            Host = _configuration["QnAEndpointHostName"]
        }, 
        options: null,
        httpClient: _httpClientFactory.CreateClient()
    );

    // QnA Maker から一番マッチした質問の回答を受け取る
    var options = new QnAMakerOptions { Top = 1 };

    // デバッグ用にオウム返し
    await turnContext.SendActivityAsync(
        MessageFactory.Text(text: $"(*ﾟ▽ﾟ* っ)З 質問は『{turnContext.Activity.Text}』だね！")
    );

    var response = await qnaMaker.GetAnswersAsync(turnContext, options);

    // 回答が存在したら応答する
    if (response != null && response.Length > 0)
    {
        await turnContext.SendActivityAsync(
                activity: MessageFactory.Text(response[0].Answer),
                cancellationToken: cancellationToken
            );
    }
    else
    {
        await turnContext.SendActivityAsync(
                activity: MessageFactory.Text("質問に対する回答が見つかりませんでした"),
                cancellationToken: cancellationToken
            );
    }
}
```

実行してエミュレータでちゃんと動いているのを確認します

![](https://storage.googleapis.com/zenn-user-upload/pry8wyp41ty1htwr3scuimgoycu5)

たとえば以下のメッセージを試してみてください

```
大学で買った Windows はどこでダウンロードできるの？

Windows の 32 bit 版と 64 bit 版って何？

Office のインストールについてもっと教えて
```

### 3-7-3: EchoBot.cs: 最初のメッセージを編集

EchoBot クラスの `OnMembersAddedAsync` でメンバーが追加されたときのメッセージがあるので、ここを日本語にしておきましょう。

```csharp
var welcomeText = "(*ﾟ▽ﾟ* っ)З こんにちは！何でも聞いてね";

```

![](https://storage.googleapis.com/zenn-user-upload/mr15alci91l0tl0v06ha7b4qgepg)

## 3-8: Azure にデプロイ

Visual Studio のプロジェクトの右クリックメニューから「発行」

![](https://storage.googleapis.com/zenn-user-upload/6xu6x7v0pnpey0t2yhsvd1xyftxw)

事前に作っておいた App Service を選びます。

![](https://storage.googleapis.com/zenn-user-upload/8fxb3o4ztzefdx6sjlfdce7ful9a)

publish (発行) しましょう

![](https://storage.googleapis.com/zenn-user-upload/4pypqwiaohg2rcutdp4fp9ofgfam)

# 4: Teams と繋げる

今までエミュレータでの実行でしたが、いよいよ Teams 上で動かしてみましょう。

## 4-1: チャンネル登録



最初に作った `Web App Bot` のリソースに移動します。

![](https://storage.googleapis.com/zenn-user-upload/8hmdilvc6adcj01xrsrd8e0iefc0)

その左の「チャンネル」から「Microsoft Teams チャンネルを構成」をクリック。

![](https://storage.googleapis.com/zenn-user-upload/mzpuzzrey1fhn17cyw3wod4bj72i)

「保存」を押します。

## 4-2: Microsoft App ID コピー

また左のメニューの「`Configuration` (設定)」から
`Microsoft App ID` をコピーします。

![](https://storage.googleapis.com/zenn-user-upload/hkn29jgtzf8v150vda6zu24yswqz)

## 4-3: Teams bot 設定ファイル manifest.json

Teams bot 設定ファイル `manifest.json` を書きます。

「アプリ名は何」「作者名」などのアプリの設定や
アプリの接続情報「Microsoft App ID/ Secret」(Azure からコピペしてくるやつ)などを
json で記述する、設定ファイルです。

（詳細：[公式ドキュメント『Reference: Manifest schema for Microsoft Teams』](https://docs.microsoft.com/en-us/microsoftteams/platform/resources/schema/manifest-schema?WT.mc_id=docs-blog-machiy) ）

色々書けるのですが、今回は最小の要素で構成されたものでいきましょう。

こちらを適当なテキストエディタにコピペして `manifest.json` という名前で保存してください。

```json
{
    "$schema": " https://developer.microsoft.com/en-us/json-schemas/teams/v1.3/MicrosoftTeams.schema.json",
    "manifestVersion": "1.3",
    "version": "1.0.0",
    "id": "ボットチャンネル登録にあるボットのAppId",
    "packageName": "com.example.mysamplebot",
    "developer": {
        "name": "開発者名",
        "websiteUrl": "https://example.com/",
        "privacyUrl": "https://example.com/privacy",
        "termsOfUseUrl": "https://example.com/app-tos"
    },
    "name": {
        "short": "FAQbot",
        "full": "FAQ-bot"
    },
    "description": {
        "short": "サンプルのボット",
        "full": "サンプルのボットです。"
    },
    "icons": {
        "outline": "icon32x32.png",
        "color": "icon192x192.png"
    },
    "accentColor": "#ff0000",
    "bots": [
        {
            "botId": "ボットチャンネル登録にあるボットのAppId",
            "needsChannelSelector": false,
            "isNotificationOnly": false,
            "scopes": [
                "personal", "team", "groupchat"
            ],
            "supportsFiles": false,
            "commandLists": []
        }
    ]
}
```

こちらに先ほどコピーしておいた Microsoft App ID をコピペします。2 か所あります。

## 4-4: bot アイコンをつくる

アイコン用に 32 x 32 と 192 x 192 の画像を
それぞれ `icon32x32.png`, `icon192x192.png` などという名前で保存しましょう。

アイコン画像作るのが面倒な方は
こちらからご自由にお使いください

[https://github.com/chomado/210219_FAQbot-on-Teams/tree/main/teams-settings](https://github.com/chomado/210219_FAQbot-on-Teams/tree/main/teams-settings)

## 4-5: zip で固める

エクスプローラーで manifest.json, icon32x32.png, icon192x192.png を選択して右クリックから圧縮します。

![](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F24609%2F5adae526-9d29-59ac-9482-b9254c5c0d06.png?ixlib=rb-1.2.2&auto=format&gif-q=60&q=75&w=1400&fit=max&s=0f9e496cb65c269159e888f3afac3e95)
（図：私が過去書いた記事『[【第4/5】Teams bot をローカル (Visual Studio 2019) で開発し、Azure で無料で動かす【その４：Teams に繋げてデバッグ編】](https://qiita.com/chomado/items/23c66a975e21265d99ae#4-3-teams-%E3%81%AB%E3%82%AB%E3%82%B9%E3%82%BF%E3%83%A0%E3%82%A2%E3%83%97%E3%83%AA%E3%81%A8%E3%81%97%E3%81%A6%E7%99%BB%E9%8C%B2)』）

## 4-6: Teams にカスタムアプリとして登録

さて、いよいよ Teams での作業となります。
Teams を開いて左下の「カスタムアプリをアップロード」をクリックします。

![](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F24609%2F75714667-4307-0679-e311-91840a3c84f1.png?ixlib=rb-1.2.2&auto=format&gif-q=60&q=75&w=1400&fit=max&s=3de6c7bc4bb5ef73cb88997e83ef9fac)
（図：私が過去書いた記事『[【第4/5】Teams bot をローカル (Visual Studio 2019) で開発し、Azure で無料で動かす【その４：Teams に繋げてデバッグ編】](https://qiita.com/chomado/items/23c66a975e21265d99ae#4-3-teams-%E3%81%AB%E3%82%AB%E3%82%B9%E3%82%BF%E3%83%A0%E3%82%A2%E3%83%97%E3%83%AA%E3%81%A8%E3%81%97%E3%81%A6%E7%99%BB%E9%8C%B2)』）

![](https://storage.googleapis.com/zenn-user-upload/fy0ggmidmixkx79n6sp7zd8ydvjl)

![](https://storage.googleapis.com/zenn-user-upload/y8ei5kpwgtr4fgajsog3i78vhc52)

あとはメッセージの体裁を適当に整えれば完成です！

![](https://storage.googleapis.com/zenn-user-upload/ae2p13yvkgwtpnnu5o26qv13gsl8)

コードは全て GitHub に公開しました

https://github.com/chomado/210219_FAQbot-on-Teams

読んで頂きましてありがとうございました😊

ご質問やご感想などありましたら、ぜひツイッター ([@chomado](https://twitter.com/chomado)) でお気軽にお問い合わせください。