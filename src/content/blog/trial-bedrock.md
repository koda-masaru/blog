---
pubDatetime: 2023-12-02T20:36:00+09:00
title: Amazon Bedrock で Claude 2.1 を試してみる
postSlug: trial-bedrock
featured: false
draft: false
tags:
  - docs
  - AWS
  - Bedrock
  - Claude
  - LLM
description:
  11月27日から始まった AWS re:Invent でアナウンスされた Claude 2.1 を試してみます
---

## 導入

生成系AIが非常に流行っている今日この頃なので、とりあえず触ってみる事が大事だと思っています。
というわけで、re:InventでアナウンスがあったClaude 2.1をさっそく触ってみます。
とくに難しいところも無かったのですが、忘備録を書いておきます。

## Claude 2.1を有効にする

Claudeのモデルを使えるようにするために、AWSコンソールにログインしいくつか設定を行います。

- Bedrockのメニューに進み、 Model access を表示します
  - ここで利用できるモデルの管理ができます
- Manage mode; access を選択します

![Base access](/assets/trial-bedrock/02-model-access-top.png)

- Claude を使えるようにするためにに、Anthropic のところにある　`Submit use case details` を登録します

![Submit use case details](/assets/trial-bedrock/03-submit-use-case-details.png)

- use case details のダイアログはそれっぽい内容を登録します

![use case details](/assets/trial-bedrock/04-use-case-details.png)

- use case details を入力したら、Claudeを使うように選択し、 `Request model access` を押します

![Request model access](/assets/trial-bedrock/05-request-model-access.png)

- Claude が使えるように処理が開始されます

![In progress](/assets/trial-bedrock/06-in-progress.png)

- 数分待つと、`In progress` が `Access granted` になり利用可能になります

![Access granted](/assets/trial-bedrock/07-access-granted.png)

## Claude 2.1を使ってみる

- AWSコンソールのPlaygroundでお試しできます
- Claude2.1, Claude2, Claude1.3 で結果がどう変化するかを試してみました
- ChatGPTとの出力の差も見てみる

### Claude2.1

![Claude2.1](/assets/trial-bedrock/10-Claude2.1.png)

### Claude2

![Claude2](/assets/trial-bedrock/11-Claude2.png)

### Claude1.3

![Claude1.3](/assets/trial-bedrock/12-Claude1.3.png)

### ChatGPT3.5

![ChatGPT3.5](/assets/trial-bedrock/13-ChatGPT3.5.png)

### 試してみて

- バージョンが上がると、より簡潔に答えているように感じる
- 1.3系と2系では大きく差があるように感じるが、2と2.1は大きな差は感じない
- 1.3系やChatGPT3.5は説明的であり、Claude2系はより自然な人に近くなるようにしている？

上記の手順で簡単に試す事ができるので、興味のあるかたは試してみてください。
