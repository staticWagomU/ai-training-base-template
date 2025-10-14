# 型定義リファレンス

このドキュメントでは、チャットアプリケーションで使用される型定義について説明します。

## 目次

- [アプリケーション型定義](#アプリケーション型定義)
  - [メッセージ型](#メッセージ型)
  - [エラー型](#エラー型)
- [Vercel AI SDK型定義](#vercel-ai-sdk型定義)
- [OpenRouter AI SDK Provider型定義](#openrouter-ai-sdk-provider型定義)
- [型定義ファイルの場所](#型定義ファイルの場所)

## アプリケーション型定義

### メッセージ型

#### MessageRole

メッセージの役割を表す型です。

**定義場所**: `app/types/chat.ts`

```typescript
export type MessageRole = "user" | "assistant" | "system";
```

| 値 | 説明 |
|----|------|
| `user` | ユーザーからのメッセージ |
| `assistant` | AIアシスタントからのメッセージ |
| `system` | システムメッセージ（プロンプトなど） |

#### ChatMessage

チャットメッセージの基本型です。

**定義場所**: `app/types/chat.ts`

```typescript
export interface ChatMessage {
  role: MessageRole;
  content: string;
}
```

| プロパティ | 型 | 説明 |
|-----------|-----|------|
| `role` | `MessageRole` | メッセージの役割 |
| `content` | `string` | メッセージの内容 |

#### Message

UI表示用のメッセージ型です。

**定義場所**: `app/types/chat.ts`

```typescript
export interface Message {
  id: string;
  role: MessageRole;
  content: string;
  createdAt?: Date;
}
```

| プロパティ | 型 | 説明 |
|-----------|-----|------|
| `id` | `string` | メッセージの一意識別子 |
| `role` | `MessageRole` | メッセージの役割 |
| `content` | `string` | メッセージの内容 |
| `createdAt` | `Date` (optional) | メッセージの作成日時 |

#### ChatRequest

API `/api/chat`へのリクエスト型です。

**定義場所**: `app/types/chat.ts`

```typescript
export interface ChatRequest {
  messages: ChatMessage[];
}
```

| プロパティ | 型 | 説明 |
|-----------|-----|------|
| `messages` | `ChatMessage[]` | 会話履歴の配列 |

### エラー型

#### ErrorType

エラーの種類を表す型です。

**定義場所**: `app/types/error.ts`

```typescript
export type ErrorType =
  | "validation_error"
  | "authentication_error"
  | "rate_limit_error"
  | "api_error";
```

| 値 | 説明 |
|----|------|
| `validation_error` | 入力検証エラー |
| `authentication_error` | 認証エラー |
| `rate_limit_error` | レート制限超過エラー |
| `api_error` | APIエラー（サーバーエラー） |

#### OpenRouterError

OpenRouterのエラーレスポンス型です。

**定義場所**: `app/types/error.ts`

```typescript
export interface OpenRouterError {
  error: {
    type: ErrorType;
    message: string;
    code: string;
  };
}
```

| プロパティ | 型 | 説明 |
|-----------|-----|------|
| `error.type` | `ErrorType` | エラーの種類 |
| `error.message` | `string` | エラーメッセージ |
| `error.code` | `string` | エラーコード |

**エラーコード例**:
- `INVALID_MESSAGES`: メッセージ配列が無効
- `OPENROUTER_INIT_ERROR`: OpenRouter初期化エラー
- `INTERNAL_ERROR`: 内部サーバーエラー

## Vercel AI SDK型定義

Vercel AI SDK v2で提供される主要な型定義です。

### UIMessage

Vercel AI SDK v2のメッセージ形式です。

**パッケージ**: `ai`

```typescript
interface UIMessage {
  id: string;
  role: "user" | "assistant" | "system";
  parts: Array<{
    type: "text";
    text: string;
  }>;
}
```

| プロパティ | 型 | 説明 |
|-----------|-----|------|
| `id` | `string` | メッセージの一意識別子 |
| `role` | `string` | メッセージの役割 |
| `parts` | `Array<Part>` | メッセージパーツ（テキスト、画像など） |

**使用例**:
```typescript
const message: UIMessage = {
  id: "msg-1",
  role: "user",
  parts: [
    {
      type: "text",
      text: "こんにちは",
    },
  ],
};
```

### UseChatReturn

`useChat`フックの戻り値型です。

**パッケージ**: `@ai-sdk/react`

```typescript
interface UseChatReturn {
  messages: UIMessage[];
  status: "ready" | "submitted" | "streaming" | "error";
  sendMessage: (message: { text: string }) => void;
  error?: Error;
  regenerate: () => void;
  stop: () => void;
  setMessages: (messages: UIMessage[]) => void;
}
```

| プロパティ | 型 | 説明 |
|-----------|-----|------|
| `messages` | `UIMessage[]` | メッセージ履歴 |
| `status` | `string` | ストリーミング状態 |
| `sendMessage` | `Function` | メッセージ送信関数 |
| `error` | `Error` | エラーオブジェクト |
| `regenerate` | `Function` | 最後のレスポンスを再生成 |
| `stop` | `Function` | ストリーミングを停止 |
| `setMessages` | `Function` | メッセージ配列を設定 |

**ステータス値**:
- `ready`: 待機中
- `submitted`: リクエスト送信済み
- `streaming`: ストリーミング中
- `error`: エラー発生

### StreamTextResult

`streamText()`関数の戻り値型です。

**パッケージ**: `ai`

```typescript
interface StreamTextResult {
  toUIMessageStreamResponse(): Response;
  // その他のメソッド...
}
```

| メソッド | 説明 |
|---------|------|
| `toUIMessageStreamResponse()` | UIMessage形式のストリーミングレスポンスに変換 |

## OpenRouter AI SDK Provider型定義

OpenRouter AI SDK Providerで提供される主要な型定義です。

### OpenRouterProviderSettings

OpenRouterプロバイダーの設定型です。

**パッケージ**: `@openrouter/ai-sdk-provider`

```typescript
interface OpenRouterProviderSettings {
  apiKey?: string;
  baseURL?: string;
  headers?: Record<string, string>;
  compatibility?: 'strict' | 'compatible';
  fetch?: typeof fetch;
  extraBody?: Record<string, unknown>;
}
```

| プロパティ | 型 | 説明 | デフォルト |
|-----------|-----|------|-----------|
| `apiKey` | `string` | OpenRouter APIキー | 環境変数から取得 |
| `baseURL` | `string` | ベースURL | `https://openrouter.ai/api/v1` |
| `headers` | `Record<string, string>` | カスタムヘッダー | なし |
| `compatibility` | `'strict'` \| `'compatible'` | 互換性モード | `'compatible'` |
| `fetch` | `Function` | カスタムfetch関数 | グローバルfetch |
| `extraBody` | `Record<string, unknown>` | 追加リクエストボディ | なし |

### OpenRouterChatSettings

チャットモデルの設定型です。

**パッケージ**: `@openrouter/ai-sdk-provider`

```typescript
type OpenRouterChatSettings = {
  logitBias?: Record<number, number>;
  logprobs?: boolean | number;
  parallelToolCalls?: boolean;
  user?: string;
  plugins?: Array<{
    id: 'web';
    max_results?: number;
    search_prompt?: string;
  }>;
  web_search_options?: {
    max_results?: number;
    search_prompt?: string;
  };
  provider?: {
    order?: string[];
    allow_fallbacks?: boolean;
    require_parameters?: boolean;
    data_collection?: 'allow' | 'deny';
    only?: string[];
    ignore?: string[];
    quantizations?: Array<'int4' | 'int8' | 'fp4' | 'fp6' | 'fp8' | 'fp16' | 'bf16' | 'fp32' | 'unknown'>;
    sort?: 'price' | 'throughput' | 'latency';
    max_price?: {
      prompt?: number | string;
      completion?: number | string;
      image?: number | string;
      audio?: number | string;
      request?: number | string;
    };
  };
  reasoning?: {
    enabled?: boolean;
    exclude?: boolean;
  } & (
    | { max_tokens: number }
    | { effort: 'high' | 'medium' | 'low' }
  );
  usage?: {
    include: boolean;
  };
};
```

**主要プロパティ**:

| プロパティ | 型 | 説明 |
|-----------|-----|------|
| `logitBias` | `Record<number, number>` | トークン出現確率の調整 |
| `logprobs` | `boolean` \| `number` | ログ確率の返却設定 |
| `user` | `string` | エンドユーザー識別子 |
| `reasoning` | `object` | 推論設定（Claude thinking等） |
| `usage` | `object` | 使用量トラッキング設定 |
| `provider` | `object` | プロバイダールーティング設定 |

### OpenRouterUsageAccounting

使用量トラッキングのレスポンス型です。

**パッケージ**: `@openrouter/ai-sdk-provider`

```typescript
type OpenRouterUsageAccounting = {
  promptTokens: number;
  promptTokensDetails?: {
    cachedTokens: number;
  };
  completionTokens: number;
  completionTokensDetails?: {
    reasoningTokens: number;
  };
  totalTokens: number;
  cost?: number;
  costDetails: {
    upstreamInferenceCost: number;
  };
};
```

| プロパティ | 型 | 説明 |
|-----------|-----|------|
| `promptTokens` | `number` | プロンプトのトークン数 |
| `completionTokens` | `number` | 補完のトークン数 |
| `totalTokens` | `number` | 合計トークン数 |
| `cost` | `number` | コスト（USD） |
| `reasoningTokens` | `number` | 推論トークン数（詳細） |
| `upstreamInferenceCost` | `number` | アップストリームコスト |

## 型定義ファイルの場所

### アプリケーション型定義

```
app/
├── types/
│   ├── chat.ts          # メッセージ関連の型
│   └── error.ts         # エラー関連の型
```

### 外部パッケージ型定義

```
node_modules/
├── @openrouter/ai-sdk-provider/
│   └── dist/
│       └── index.d.ts   # OpenRouter型定義
├── ai/
│   └── index.d.ts       # Vercel AI SDK型定義
└── @ai-sdk/react/
    └── index.d.ts       # React統合型定義
```

## 型変換の例

### UIMessageからMessageへの変換

`ChatUI.tsx`で実装されている変換例：

```typescript
import type { UIMessage } from "ai";
import type { Message } from "@/app/types/chat";

// UIMessage (Vercel AI SDK v2) から Message (アプリケーション型) への変換
const typedMessages: Message[] = messages.map((msg: UIMessage) => {
  const textContent = msg.parts
    .filter((part) => part.type === "text")
    .map((part) => (part as { text: string }).text)
    .join("");

  return {
    id: msg.id,
    role: msg.role as "user" | "assistant" | "system",
    content: textContent,
  };
});
```

### MessageからUIMessageへの変換

```typescript
import type { Message } from "@/app/types/chat";
import type { UIMessage } from "ai";

function convertToUIMessage(message: Message): UIMessage {
  return {
    id: message.id,
    role: message.role,
    parts: [
      {
        type: "text",
        text: message.content,
      },
    ],
  };
}
```

## 型安全性のベストプラクティス

### 1. 厳格な型定義を使用

```typescript
// ❌ 避けるべき
const messages: any[] = [];

// ✅ 推奨
const messages: Message[] = [];
```

### 2. 型ガードを使用

```typescript
function isUIMessage(obj: unknown): obj is UIMessage {
  return (
    typeof obj === "object" &&
    obj !== null &&
    "id" in obj &&
    "role" in obj &&
    "parts" in obj
  );
}
```

### 3. 型アサーションは最小限に

```typescript
// ❌ 避けるべき
const message = msg as Message;

// ✅ 推奨：型変換関数を使用
const message = convertToMessage(msg);
```

## 参考リンク

- [APIリファレンス](./api-reference.md) - API仕様の詳細
- [OpenRouter統合ガイド](./openrouter-integration.md) - OpenRouterの設定
- [実装ガイド](./implementation-guide.md) - 実装の詳細
- [TypeScript公式ドキュメント](https://www.typescriptlang.org/) - TypeScript言語仕様
- [Vercel AI SDK型定義](https://ai-sdk.dev/docs/reference) - Vercel AI SDKの型定義
