---
pubDatetime: 2023-12-04T23:24:00+09:00
title: Bedrock + Claude 2.1 + Lambda@Edge を Chatbot に繋げてみた
postSlug: bedrock-claude-chatbot
featured: false
draft: false
tags:
  - docs
  - LLM
  - AWS
  - Lambda@Edge
  - Node.js
  - Hono
  - Bedrock
  - Claude
  - LINE
  - Chatbot
description:
  Open AI の ChatGPT3.5 で作成していたChatbotを、Bedrock + Claude 2.1に接続変更してみた話です
---

## やってみたこと

家族が気軽にLLMに触れることができるようにLINEのChatbotでOpenAIのChatGPTに繋げるということをやってきました。

- [Cloudflare Workers で OpenAI の LINE Chatbotを作ってみた](./cloudflare-workers-open-ai-chatbot)
- [AWS Lambda@Edge + Hono で OpenAI の LINE Chatbotを作ってみた](./lambda-edge-hono-open-ai-chatbot)

Bedrock で Claude 2.1 が使えるようになり、 ChatGPT3.5 より高速で精度が良いという噂だったので、
ChatBotが繋げるAIを Claude 2.1 に変更してみました。

## 前提

- [利用しているAWSアカウントでBedrock + Claude 2.1が利用可能になっていること](./trial-bedrock)

## Lambda から Bedrock に繋げるコードを書く

- まず、 AWS SDK の `yarn add -D @aws-sdk/client-bedrock-runtime` をインストールします

```sh
yarn add -D @aws-sdk/client-bedrock-runtime
```

- ライブラリを使い、Bedrockにメッセージを送るコードを書きます

```ts
import { logger } from './logger'
import { BedrockRuntime } from '@aws-sdk/client-bedrock-runtime'
const client = new BedrockRuntime({ region: 'us-east-1' })

export async function chat(chat_message: string) {
 try {
   const start = new Date().getTime()
   logger.info(`start call Bedrock Claude.`)
   let answer = ''
   if (chat_message.length > 5) {
     logger.info(`Request to Bedrock Claude. ${chat_message}`)

     const res = await client.invokeModel({
       modelId: 'anthropic.claude-v2',
       accept: 'application/json',
       contentType: 'application/json',
       body: JSON.stringify({
         prompt: `\n\nHuman: ${chat_message} \n\nAssistant:`,
         max_tokens_to_sample: 1000,
         temperature: 1,
         top_k: 250,
         top_p: 0.999,
         anthropic_version: 'bedrock-2023-05-31',
       }),
     })
     const body: {
       completion: string
       stop_reason: string
       stop: string
     } = JSON.parse(Buffer.from(res.body).toString('utf-8'))
     logger.trace(body)
     answer = body.completion
   } else {
     answer = '質問は5文字以上で入力してください'
   }
   logger.info(`Result from Bedrock Claude. ${answer}`)
   logger.info(`finish call Bedrock Claude. ${new Date().getTime() - start}[ms]`)
   return answer
 } catch (e) {
   logger.error(e)
   return 'AIが回答を見つけられませんでした。'
 }
}
```

- コードの特徴的な部分を説明します

```ts
import { logger } from './logger'
import { BedrockRuntime } from '@aws-sdk/client-bedrock-runtime'
const client = new BedrockRuntime({ region: 'us-east-1' })
```

- `@aws-sdk/client-bedrock-runtime` を使い、 Bedrock にアクセスするためのオブジェクトをインスタンス化しています

```ts
    const res = await client.invokeModel({
       modelId: 'anthropic.claude-v2:1',
       accept: 'application/json',
       contentType: 'application/json',
       body: JSON.stringify({
         prompt: `\n\nHuman: ${chat_message} \n\nAssistant:`,
         max_tokens_to_sample: 1000,
         temperature: 1,
         top_k: 250,
         top_p: 0.999,
         anthropic_version: 'bedrock-2023-05-31',
       }),
     })
```

- 実際に Bedrockにリクエストを送り、結果を取得している部分です
- `modelId: 'anthropic.claude-v2:1'` で Claude 2.1 を利用するようになります
  - https://docs.aws.amazon.com/bedrock/latest/userguide/model-ids-arns.html に、modelIdに指定できるモデルの選択肢が書かれています
- prompt には `\n\nHuman: ${chat_message} \n\nAssistant:` の形式で指定する必要があります

```ts
     const body: {
       completion: string
       stop_reason: string
       stop: string
     } = JSON.parse(Buffer.from(res.body).toString('utf-8'))
     logger.debug(body)
```

- Bedrock から取得したレスポンスから、結果のオブジェクトを取得しています
- `completion` にAIからの回答が入っているため、これをChatbot経由で利用者に返しています

## LambdaからBedrockを利用できるように、権限をセットする

- https://docs.aws.amazon.com/ja_jp/bedrock/latest/userguide/api-setup.html の記述に従いLambdaに権限をセットします
- Lambdaの権限に以下を追加します

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "bedrock:*",
            "Resource": "*"
        }
    ]
}
```

- 信頼関係に以下を追加します

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": {
                "Service": "bedrock.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        },
        {
            "Sid": "",
            "Effect": "Allow",
            "Principal": {
                "Service": "sagemaker.amazonaws.com"
            },
            "Action": "sts:AssumeRole"
        }
    ]
}
```

## 実際にLINEのChatbot経由で利用してみる

![Chatbot](/assets/bedrock-claude-chatbot/chatbot.png)

- 無事に Bedrock + Claude 2.1 に接続先を切り替えられました

## 作ってみての感想

- `@aws-sdk/client-bedrock-runtime` のライブラリを使えば、特に問題なく呼び出しはできる
  - OpenAIのAPIも `openai` のライブラリがあるので、難易度はほぼ変わらない
  - 呼び出す際の prompt が `\n\nHuman: ${chat_message} \n\nAssistant:` の形式で呼び出すなど少しクセはある
  - ただの文字列なのでできれば型が使えるとコードが書きやすい（せっかくTypeScriptを使っているし）
  - 戻り値も `JSON.parse` で取得するため型情報がないのは少し書きづらい
- 今回は AWS SDK を使って接続したが、 `LangChain` を使う形に変更すれば、型問題などは解決するかも？
  - `LangChain` を使ったアクセスに変更してみたい
- AWSは有料アカウント(OpenAIが無料枠利用)ということもあってかもだが、Claude 2.1の方が結果が出るまでに時間が短いので利用しやすくなった

LambdaからBedrockに繋げて結果を取得することも難しくはなかったので、興味のあるかたは試してみてください。
