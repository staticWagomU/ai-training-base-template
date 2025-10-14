# OpenRouter統合ガイド

このドキュメントでは、このアプリケーションにおけるOpenRouterの統合方法について説明します。

## 目次

- [概要](#概要)
- [環境設定](#環境設定)
- [OpenRouterの初期化](#openrouterの初期化)
- [モデルの選択](#モデルの選択)
- [利用可能なモデル](#利用可能なモデル)
- [エラーハンドリング](#エラーハンドリング)

## 概要

このアプリケーションは、[OpenRouter](https://openrouter.ai/)を使用して、複数のAIモデルへの統一的なアクセスを提供しています。

### 使用技術

- **OpenRouter AI SDK Provider**: v1.2.0 (`@openrouter/ai-sdk-provider`)
- **Vercel AI SDK**: v5.0.60 (`ai`)
- 統合により、数百のAIモデルに単一のAPIで簡単にアクセスできます

### アーキテクチャ

```
Client (useChat)
    ↓
API Route (/api/chat)
    ↓
Vercel AI SDK (streamText)
    ↓
OpenRouter Provider (createOpenRouter)
    ↓
OpenRouter API
```

## 環境設定

### 1. OpenRouter APIキーの取得

1. [OpenRouter](https://openrouter.ai/)にアクセス
2. アカウントを作成またはログイン
3. [API Keys](https://openrouter.ai/keys)ページでAPIキーを生成

### 2. 環境変数の設定

プロジェクトルートに`.env.local`ファイルを作成し、以下の環境変数を設定します：

```bash
# OpenRouter API Key（必須）
OPENROUTER_API_KEY=sk-or-v1-your-api-key-here

# 使用するモデル（オプション）
# デフォルト: anthropic/claude-3.7-sonnet:thinking
OPENROUTER_MODEL=anthropic/claude-3.7-sonnet:thinking
```

**重要**: `.env.local`ファイルは`.gitignore`に登録されており、Gitにコミットされません。

### 3. 環境変数の例

`.env.example`ファイルに設定例があります：

```bash
# OpenRouter API Key
# Get your API key from: https://openrouter.ai/keys
OPENROUTER_API_KEY=sk-or-v1-your-api-key-here

# OpenRouter Model (optional)
# Default: anthropic/claude-3.7-sonnet:thinking
# See available models: https://openrouter.ai/models
OPENROUTER_MODEL=anthropic/claude-3.7-sonnet:thinking
```

## OpenRouterの初期化

### 初期化関数

`app/lib/openrouter.ts`で定義されています：

```typescript
import { createOpenRouter } from "@openrouter/ai-sdk-provider";

const DEFAULT_MODEL = "anthropic/claude-3.7-sonnet:thinking";

/**
 * OpenRouterインスタンスを初期化する
 * @throws {Error} APIキーが設定されていない場合
 * @returns OpenRouterインスタンス
 */
export function initializeOpenRouter() {
  const apiKey = process.env.OPENROUTER_API_KEY;

  if (!apiKey || apiKey.trim() === "") {
    throw new Error("OPENROUTER_API_KEY environment variable is not set");
  }

  return createOpenRouter({
    apiKey,
  });
}
```

### モデル取得関数

```typescript
/**
 * OpenRouterモデルインスタンスを取得する
 * @param modelName - 使用するモデル名(オプション)
 * @returns モデルインスタンス
 */
export function getOpenRouterModel(modelName?: string) {
  const openrouter = initializeOpenRouter();
  const model = modelName || process.env.OPENROUTER_MODEL || DEFAULT_MODEL;

  return openrouter(model);
}
```

### 使用例

API Route (`app/api/chat/route.ts`)での使用例：

```typescript
import { streamText } from "ai";
import { getOpenRouterModel } from "@/app/lib/openrouter";

export async function POST(request: Request) {
  try {
    const { messages } = await request.json();

    // OpenRouterモデルの取得
    const model = getOpenRouterModel();

    // ストリーミングレスポンスの生成
    const result = streamText({
      model,
      messages,
    });

    return result.toUIMessageStreamResponse();
  } catch (error) {
    console.error("Chat API error:", error);
    // エラー処理...
  }
}
```

## モデルの選択

### デフォルトモデル

デフォルトでは、`anthropic/claude-3.7-sonnet:thinking`を使用します。

### モデルの変更方法

#### 方法1: 環境変数で設定

`.env.local`ファイルで`OPENROUTER_MODEL`を設定：

```bash
OPENROUTER_MODEL=openai/gpt-4-turbo
```

#### 方法2: コードで直接指定

`getOpenRouterModel()`関数に引数を渡す：

```typescript
const model = getOpenRouterModel("openai/gpt-4-turbo");
```

### モデル名の形式

OpenRouterでは、モデル名は`provider/model-name`の形式で指定します：

```
anthropic/claude-3.7-sonnet:thinking
openai/gpt-4-turbo
meta-llama/llama-3.1-405b
google/gemini-pro-1.5
```

## 利用可能なモデル

OpenRouterは数百のAIモデルへのアクセスを提供しています。

### モデル一覧の確認

公式サイトで利用可能なモデルを確認できます：
- [OpenRouter Models](https://openrouter.ai/models)

### 主要なモデル

| プロバイダー | モデル名 | 用途 |
|------------|---------|------|
| Anthropic | `anthropic/claude-3.7-sonnet:thinking` | 高度な推論タスク |
| OpenAI | `openai/gpt-4-turbo` | 汎用的な会話・テキスト生成 |
| Meta | `meta-llama/llama-3.1-405b` | オープンソース大規模モデル |
| Google | `google/gemini-pro-1.5` | マルチモーダル対応 |

### モデルの価格

各モデルの価格情報は[OpenRouter Models](https://openrouter.ai/models)ページで確認できます。

## エラーハンドリング

### APIキー未設定エラー

`OPENROUTER_API_KEY`が設定されていない場合、`initializeOpenRouter()`は`Error`をスローします：

```typescript
try {
  const model = getOpenRouterModel();
} catch (error) {
  console.error("OpenRouter initialization error:", error.message);
  // エラー: "OPENROUTER_API_KEY environment variable is not set"
}
```

### APIエラー

OpenRouter APIからのエラーは、Vercel AI SDKを通じて処理されます。詳細は[APIリファレンス](./api-reference.md)を参照してください。

## 追加設定

### OpenRouterの高度な設定

`createOpenRouter()`関数は、追加のオプションをサポートしています：

```typescript
import { createOpenRouter } from "@openrouter/ai-sdk-provider";

const openrouter = createOpenRouter({
  apiKey: process.env.OPENROUTER_API_KEY,
  // カスタムヘッダー
  headers: {
    "HTTP-Referer": "https://your-app.com",
    "X-Title": "Your App Name",
  },
  // カスタムベースURL（プロキシ経由など）
  baseURL: "https://your-proxy.com/api/v1",
});
```

### プロバイダー設定

OpenRouterの特定機能を有効にする：

```typescript
const model = openrouter("anthropic/claude-3.7-sonnet:thinking", {
  // 推論トークンの設定
  reasoning: {
    enabled: true,
    exclude: false,
    effort: "high", // "high" | "medium" | "low"
  },
  // 使用量トラッキング
  usage: {
    include: true,
  },
  // プロバイダールーティング設定
  provider: {
    order: ["anthropic", "openai"],
    allow_fallbacks: true,
  },
});
```

## 参考リンク

- [OpenRouter公式サイト](https://openrouter.ai/)
- [OpenRouter API Documentation](https://openrouter.ai/docs)
- [利用可能なモデル一覧](https://openrouter.ai/models)
- [Vercel AI SDK Documentation](https://ai-sdk.dev/)
- [@openrouter/ai-sdk-provider on npm](https://www.npmjs.com/package/@openrouter/ai-sdk-provider)
