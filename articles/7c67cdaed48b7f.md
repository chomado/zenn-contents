---
title: "無料で手早く作る：Teams で動く FAQ bot 開発 [後編] (2021 年 12 月版)"
emoji: "🤖"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [CSharp, Azure, Bot, BotFramework, VisualStudio]
published: true
---

![](https://storage.googleapis.com/zenn-user-upload/ce7d4ce66d17-20211126.png)


:::message
注意書き）2021/11/26
この記事はもともと 2021 年 2 月に書かれたものですが、この 9 カ月間に色々アップデートがあったので、諸々 書き直しました！　差分の気になる方は GitHub の commit log に。(この記事は git 管理されています)
2021 年 11 月末現在で最新の情報を書いています。
:::

[前編の記事](https://zenn.dev/chomado/articles/8ab50af04b52cc) では、Azure Cognitive Service for Language でのナレッジベース作成まで完了しましたね。
この後半の記事ではクライアント (bot) のコードをガリガリ書いていきます。

0. ~~事前準備~~
1. ~~Azure で bot ホスト先を用意~~
2. ~~Azure Cognitive Service for Language の question answering 機能でナレッジベースを作る~~ `← ここまで完了`
3. bot クライアント開発 (Visual Studio で) `← ここから`
4. Teams と繋げる

# 参考ドキュメント

公式ドキュメント：『[クイックスタート: 質問応答](https://docs.microsoft.com/ja-jp/azure/cognitive-services/language-service/question-answering/quickstart/sdk?pivots=programming-language-csharp&WT.mc_id=spatial-8948-machiy)』

# 3: bot クライアント開発

Microsoft 公式が用意してくれている、Botframework のエコー bot テンプレートに手を加える形で進めていきましょう。


:::message
エコー bot は、
ユーザの入力した文字列をそのまま返してくれる bot で、

ユーザの入力をきちんと取れて、また、ちゃんとチャットを返してくれる、ということで、チャットボットのハローワールド的なものによく使われる題材です
:::

ちなみにソースコードはすべて GitHub に上げてあります
https://github.com/chomado/211125-FAQBot

## 3-1: Visual Studio に Botframework テンプレート

Visual Studio 2022 の拡張機能から Bot Framework v4 SDK Templates for Visual Studio を入れます。詳しく手順を示します。

まず VS2022 を開いて、「Continue without code」をクリック。（私は VS の言語を英語にしているので、日本語にしている方は適宜読み替えてください）

![](https://storage.googleapis.com/zenn-user-upload/a15f1a7020fa-20211126.png)

メニューバーから `Extentions` (拡張機能) → `Manage Extentions`

![](https://storage.googleapis.com/zenn-user-upload/n2x0aglj59phh0u2vsfw6dsmcvtp)

左のタブの「Online」を選択した状態で検索バーに `Bot Framework v4 SDK Templates` を入力しましょう。



クエリが走り、一番 上の「`Bot Framework v4 SDK Templates for Visual Studio`」を `Download` したら、VS2022 を再起動します。

## 3-2: プロジェクト作成

Visual Studio 2022 の `Create a new project` (新規作成) から 
Echo Bot (Bot Framework v4 - .NET Core 3.1) を選択して作ります。

![](https://storage.googleapis.com/zenn-user-upload/gn2jwp9maixwotack1ylslxxqmts)

プロジェクト名は任意で。

![](https://storage.googleapis.com/zenn-user-upload/ty89lytpdibvb4dklaactgn2g1ba)

「`Create`」(作成) を押します。

NuGet Package Manager から
`Microsoft.Bot.Builder.Integration.AspNet.Core` を
更新しておきます。

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

## 3-5: Azure Cognitive Service for Language に繋げる準備：接続情報を記述

User Secrets に Azure Cognitive Service for Language の設定を書きます。

プロジェクト右クリックから `Manage User Secrets`
で、空の json ファイルが生えてきます。

![](https://storage.googleapis.com/zenn-user-upload/0e3175281220-20211126.jpg)

**Language Studio** ([https://language.azure.com/](https://language.azure.com/))  の デプロイ画面の `Get prediction URL` から以下の部分の値を入れる

![](https://storage.googleapis.com/zenn-user-upload/f9d725786a24-20211126.png)

![](https://storage.googleapis.com/zenn-user-upload/45267c969052-20211126.jpg)

`secrets.json` サンプル

````json
{
  "Endpoint": "https://japaneast.api.cognitive.microsoft.com/",
  "ProjectName": "自分が付けた名前",
  "Key": "キー文字列"
}
````

（これはローカルでの動作用なので、Azure にデプロイ後は `App Service` の `アプリケーション設定` に同じものを書きます (後述) ）

## 3-6: プロジェクトに Azure Cognitive Service for Language の SDK 入れる

プロジェクトに `Azure.AI.Language.QuestionAnswering` を NuGet (パッケージマネージャー) から入れます。詳しい方法を以下に書きます。

プロジェクト右クリックで「`Manage NuGet Packages`」

![](https://storage.googleapis.com/zenn-user-upload/z76lxcenpy9o6cxgn6vohtot23kp)

`Browse` タブの検索バー `Azure.AI.Language.QuestionAnswering` 入れて出てきたパッケージをインストールします。

![](https://storage.googleapis.com/zenn-user-upload/62ec01d7b451-20211126.png)

## 3-7: bot のコードを書いていく

### 3-7-1: EchoBot.cs: Azure Cognitive Service for Language を呼び出し

Bots フォルダの下の `EchoBot.cs` に bot のクラスがあります。

ここの `OnMessageActivityAsync` メソッドが
ユーザーから話しかけられた時に呼ばれるメソッドになります。

なのでここで `Azure Cognitive Service for Language` を呼び出せば QA bot が作れます。

まず、必要なものを EchoBot で使えるようにしましょう。

:::message
この `3-7-1` セクションでやる分のコード書き書きについては
こちらのコミットにまとめています
https://github.com/chomado/211125-FAQBot/commit/c49082b32f397fd5d5bd8e57f193f8f22a9ab115
(`3-6` でやった NuGet package 追加のぶんも含んでる)
:::

`EchoBot.cs` 、まず using を追加します。(追加分だけじゃなくもともと書いてたやつも一緒にここに書きますね (コピペフレンドリー) )

```csharp
using Azure;
using Azure.AI.Language.QuestionAnswering;
using Microsoft.Bot.Builder;
using Microsoft.Bot.Schema;
using Microsoft.Extensions.Configuration;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading;
using System.Threading.Tasks;
```

次にコンストラクタを書いていきます

```csharp
private readonly IConfiguration configuration;

public EchoBot(IConfiguration configuration)
{
    this.configuration = configuration;
}
```
次に、`OnMessageActivityAsync` メソッドを置き換えていきます。


```csharp
// ユーザーから話しかけられた時に呼ばれるメソッド。
// なのでここで Azure Cognitive Service for Language を呼び出す
protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, CancellationToken cancellationToken)
{
    // Azure Cognitive Service for Language に接続するための情報
    var endpoint = new Uri(configuration["Endpoint"]);
    var credential = new AzureKeyCredential(configuration["Key"]);
    var projectName = configuration["ProjectName"];
    var deploymentName = "production";
    
    // 上の接続情報を使い、
    // Azure Cognitive Service for Language に接続するためのクライアントを作る
    var client = new QuestionAnsweringClient(endpoint, credential);
    var project = new QuestionAnsweringProject(projectName, deploymentName);

    // もし必要ならデバッグ用にユーザの入力のオウム返しもできます
    /* await turnContext.SendActivityAsync(
        MessageFactory.Text(text: $"質問は『{turnContext.Activity.Text}』だね！")
    ); */

    // Azure Cognitive Service for Language からのレスポンスを取得
    Response<AnswersResult> response = await client.GetAnswersAsync(turnContext.Activity.Text, project);

    // bot の返答文を作る。
    // 今回は Azure Cognitive Service for Language からのレスポンスの回答部分をそのまま入れる
    var replyText = response.Value.Answers.First().Answer;

    // bot に喋ってもらう
    await turnContext.SendActivityAsync(MessageFactory.Text(replyText, replyText), cancellationToken);
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

学習のために、ブレイクポイントを張って
レスポンスの中身を見てみるのも良いですね

https://twitter.com/chomado/status/1463831754125418497

### 3-7-3: EchoBot.cs: 最初のメッセージを編集

EchoBot クラスの `OnMembersAddedAsync` でメンバーが追加されたときのメッセージ (デフォルトでは `var welcomeText = "Hello and welcome!";`) があるので、
ここを日本語にしておきましょう。

```csharp
var welcomeText = "(*ﾟ▽ﾟ* っ)З こんにちは！何でも聞いてね";

```

![](https://storage.googleapis.com/zenn-user-upload/mr15alci91l0tl0v06ha7b4qgepg)

## 3-8: Azure にデプロイ

Visual Studio のプロジェクトの右クリックメニューから「発行」

![](https://storage.googleapis.com/zenn-user-upload/6xu6x7v0pnpey0t2yhsvd1xyftxw)

どこにデプロイしたいのか聞かれるダイアログが出るので
ポチポチしていきます。

![](https://storage.googleapis.com/zenn-user-upload/8cd563efbf69-20211126.png)

`Azure` 
-> `Azure App Service (Windows)` 
-> (必要によっては Azure のサブスクリプションと紐づいてる MS アカウントでログインしてね)
-> 該当する、事前に作っておいた App Service を選ぶ 

![](https://storage.googleapis.com/zenn-user-upload/4257db5ee6fc-20211126.jpg)

publish (発行) しましょう

![](https://storage.googleapis.com/zenn-user-upload/20375c11a108-20211126.jpg)

# 4: Teams と繋げる

今までエミュレータでの実行でしたが、いよいよ Teams 上で動かしてみましょう。

## 4-0: Azure ポータル画面

最初に作ったリソースグループの画面に行きましょう。
そのリソースグループの下に作られているリソースの一覧を見ます

![](https://storage.googleapis.com/zenn-user-upload/0a38c7f6840d-20211126.png)

## 4-1: Azure Bot を作る

このリソースグループの中に `Azure Bot` を作成します。
まず `＋作成` をクリックします

![](https://storage.googleapis.com/zenn-user-upload/bcf3ee6d5013-20211126.png)

検索窓に `Azure Bot` と打ち込みます
![](https://storage.googleapis.com/zenn-user-upload/7545745430db-20211126.png)

`Azure Bot` 出てきましたね。
`作成` を推します

![](https://storage.googleapis.com/zenn-user-upload/135ca9bb5af4-20211126.png)

## 4-1-a: Azure Bot の価格帯を無料版に

デフォルトでは価格帯がスタンダードになっているので、無料版を指定します

![](https://storage.googleapis.com/zenn-user-upload/ec5b66fb796f-20211126.png)

参考）[Azure Bot Service の価格](https://azure.microsoft.com/ja-jp/pricing/details/bot-services/?WT.mc_id=spatial-8948-machiy#pricing)

## 4-1-b: Microsoft App ID の項で

Microsoft App ID の項で、アプリの種類を「ユーザー割り当て済みマネージド ID」にします。
これにすると、Azure AD へのアプリ登録などをやらなくて済んで便利です

![](https://storage.googleapis.com/zenn-user-upload/1fe9aa2e97c0-20211126.jpg)

これで作成します

## 4-1-c: Azure bot 作成完了。リソースグループに戻る

作成が完了すると「デプロイが完了しました」と出ます。

リソースグループに戻りましょう。

現在のリソース一覧はこうなっています

![](https://storage.googleapis.com/zenn-user-upload/b4ef39cb340a-20211126.png)

`Azure Bot` と一緒に
`Key Vault` (`キーコンテナ`) と `マネージド ID` が新規に作成されていますね。

これらは「誰でも私の bot が使える状態」にならないように（たとえば「うちの会社の人だけのアクセスにする」など）の
セキュアな認証などに必要なリソースです。


## 4-2: マネージド ID を Web App にも割り当てる

さっきのリソースグループのリソース一覧から
Web App (App Service) のリソースをクリックしましょう。
（私の場合 `app-211126-faqbot` って名前のリソース）

1. 左側のメニューの `設定` のなかの `ID` をクリック
2. 「`ユーザー割り当て済み`」を選択
3. `追加`

![](https://storage.googleapis.com/zenn-user-upload/225bd958eeb9-20211126.jpg)

`Azure Bot` 作成時に作られた `マネージド ID` を選択します

![](https://storage.googleapis.com/zenn-user-upload/6ce7218c0f5d-20211126.jpg)

また左のメニューの「`Configuration` (設定)」から
`Microsoft App ID` をコピーします。

![](https://storage.googleapis.com/zenn-user-upload/hkn29jgtzf8v150vda6zu24yswqz)

## 4-3: Web App の構成のアプリケーション設定に必要情報を追加

`Azure Bot` と `Web App` (App Service) (bot クライアントが載ってるところ) を繋げる作業をします。

まず必要な情報をメモします。

またリソースグループに戻り、
リソース一覧から `Azure Bot` をクリック。
（私の場合 `bot-211126-faqbot` って名前のリソース）

![](https://storage.googleapis.com/zenn-user-upload/61d7c1b9754f-20211126.jpg)

`Azure Bot` の `構成` ページの

* Microsoft App ID
* アプリ テナント ID

の値をコピーしておきます。（メモ帳かどこかにコピペしておいてください）

そしてさっきのリソースグループのリソース一覧から
Web App (App Service) のリソースに戻ります。
（私の場合 `app-211126-faqbot` って名前のリソース）

![](https://storage.googleapis.com/zenn-user-upload/3bf86aa98812-20211126.jpg)

`構成` -> `+ 新しいアプリケーション設定`

以下の 4 つの情報を Web App の構成のアプリケーション設定に追加します

＃|名前|値
---|---|---
1|MicrosoftAppType|`UserAssignedMSI`
2|MicrosoftAppId|Microsoft App ID
3|MicrosoftAppPassword|(空文字)
4|MicrosoftAppTenantId|アプリ テナント ID

設定した後は必ず `保存` を押しましょう。(私はいつもこれを忘れて「あれ～？更新されてないぞ」となる)

![](https://storage.googleapis.com/zenn-user-upload/1fca614e11ac-20211126.jpg)

## 4-4: Cognitive Service for Language の接続情報を教える

`Visual Studio 2022` のほうの `secrets.json` に書いてた、API キー文字列的な 3 つの情報を
デプロイ先の Azure にも教えてあげます。

![](https://storage.googleapis.com/zenn-user-upload/1c3de7cbb9c0-20211126.jpg)

＃|名前|値
---|---|---
1|`Endpoint`|`https://japaneast.api.cognitive.microsoft.com/`
2|`ProjectName`|プロジェクトの名前（私の場合 qna-211125-software-download-faq）
3|`Key`|キー文字列


![](https://storage.googleapis.com/zenn-user-upload/5acd7ffbc010-20211126.jpg)

忘れず「保存」をします

## 4-5: Azure Bot にエンドポイントを教える

Web App (App Service) のリソースの `概要` から
URL をコピーしておく。

![](https://storage.googleapis.com/zenn-user-upload/240133a45349-20211126.jpg)


またリソースグループに戻り、
リソース一覧から `Azure Bot` をクリック。
（私の場合 `bot-211126-faqbot` って名前のリソース）

`構成` -> `メッセージング エンドポイント`
で、さっきの URL に `api/messages` を追加したものを入れて
「`適用`」

![](https://storage.googleapis.com/zenn-user-upload/26a95b07fa2f-20211126.jpg)

## 4-6: Web chat でテスト

`Azure Bot` の左のメニュー `Web チャットでテスト` から
テストができます。
Teams で動かす前にこちらでちゃんと動いているかの確認をしましょう

![](https://storage.googleapis.com/zenn-user-upload/5ea21b6c3a07-20211126.png)

## 4-7: Azure Bot のチャンネルの構成

`Azure Bot` の左のメニューの `チャンネル` から `Microsoft Teams` を追加しましょう。

![](https://storage.googleapis.com/zenn-user-upload/db80767c1e30-20211126.png)

![](https://storage.googleapis.com/zenn-user-upload/a06e9027232b-20211126.png)


## 4-8: Teams で動かす

![](https://storage.googleapis.com/zenn-user-upload/fc110bc23161-20211126.jpg)

![](https://storage.googleapis.com/zenn-user-upload/02c38cd1786f-20211126.png)

動いた！！！！！！！！！！！！

# 5. 他の人の環境でも動かしたい

他の人の環境でも動かしたい場合、もう少し手順が必要です。（配布用の zip を作らないといけない）

## 5-1: 設定ファイル manifest.json

まず　Teams bot 設定ファイル `manifest.json` を書きます。

「アプリ名は何」「作者名」などのアプリの設定や
アプリの接続情報「Microsoft App ID/ Secret」(Azure からコピペしてくるやつ)などを
json で記述する、設定ファイルです。

詳細：[公式ドキュメント『Reference: Manifest schema for Microsoft Teams』](https://docs.microsoft.com/en-us/microsoftteams/platform/resources/schema/manifest-schema?WT.mc_id=docs-blog-machiy) 

## 5-2: bot アイコンをつくる

アイコン用に 32 x 32 と 192 x 192 の画像を
それぞれ `icon32x32.png`, `icon192x192.png` などという名前で保存しましょう。

アイコン画像作るのが面倒な方は
こちらからご自由にお使いください

[https://github.com/chomado/210219_FAQbot-on-Teams/tree/main/teams-settings](https://github.com/chomado/210219_FAQbot-on-Teams/tree/main/teams-settings)

## 5-3: zip で固める

エクスプローラーで manifest.json, icon32x32.png, icon192x192.png を選択して右クリックから圧縮します。

![](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F24609%2F5adae526-9d29-59ac-9482-b9254c5c0d06.png?ixlib=rb-1.2.2&auto=format&gif-q=60&q=75&w=1400&fit=max&s=0f9e496cb65c269159e888f3afac3e95)
（図：私が過去書いた記事『[【第4/5】Teams bot をローカル (Visual Studio 2022) で開発し、Azure で無料で動かす【その４：Teams に繋げてデバッグ編】](https://qiita.com/chomado/items/23c66a975e21265d99ae#4-3-teams-%E3%81%AB%E3%82%AB%E3%82%B9%E3%82%BF%E3%83%A0%E3%82%A2%E3%83%97%E3%83%AA%E3%81%A8%E3%81%97%E3%81%A6%E7%99%BB%E9%8C%B2)』）

## 5-4 Teams にカスタムアプリとして登録

さて、いよいよ Teams での作業となります。
Teams を開いて左下の「カスタムアプリをアップロード」をクリックします。

![](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F24609%2F75714667-4307-0679-e311-91840a3c84f1.png?ixlib=rb-1.2.2&auto=format&gif-q=60&q=75&w=1400&fit=max&s=3de6c7bc4bb5ef73cb88997e83ef9fac)
（図：私が過去書いた記事『[【第4/5】Teams bot をローカル (Visual Studio 2022) で開発し、Azure で無料で動かす【その４：Teams に繋げてデバッグ編】](https://qiita.com/chomado/items/23c66a975e21265d99ae#4-3-teams-%E3%81%AB%E3%82%AB%E3%82%B9%E3%82%BF%E3%83%A0%E3%82%A2%E3%83%97%E3%83%AA%E3%81%A8%E3%81%97%E3%81%A6%E7%99%BB%E9%8C%B2)』）

![](https://storage.googleapis.com/zenn-user-upload/fy0ggmidmixkx79n6sp7zd8ydvjl)

![](https://storage.googleapis.com/zenn-user-upload/y8ei5kpwgtr4fgajsog3i78vhc52)

あとはメッセージの体裁を適当に整えれば完成です！

![](https://storage.googleapis.com/zenn-user-upload/ae2p13yvkgwtpnnu5o26qv13gsl8)


読んで頂きましてありがとうございました😊

ご質問やご感想などありましたら、ぜひツイッター ([@chomado](https://twitter.com/chomado)) でお気軽にお問い合わせください。