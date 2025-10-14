# 実装ガイド

このドキュメントでは、チャットアプリケーションのクライアントサイド・サーバーサイド実装について詳しく説明します。

## 目次

- [概要](#概要)
- [クライアントサイド実装](#クライアントサイド実装)
  - [useChatフックの使用](#usechatフックの使用)
  - [メッセージ送信](#メッセージ送信)
  - [ストリーミング状態の管理](#ストリーミング状態の管理)
  - [エラーハンドリング](#エラーハンドリング)
- [サーバーサイド実装](#サーバーサイド実装)
  - [API Route実装](#api-route実装)
  - [OpenRouter統合](#openrouter統合)
  - [ストリーミングレスポンス](#ストリーミングレスポンス)
- [コンポーネント実装例](#コンポーネント実装例)
- [ベストプラクティス](#ベストプラクティス)

## 概要

このアプリケーションは、以下のアーキテクチャで構成されています：

```
クライアント層
├── ChatUI Component
│   └── useChat Hook (Vercel AI SDK)
│
サーバー層
├── API Route (/api/chat)
│   └── streamText (Vercel AI SDK)
│       └── OpenRouter Provider
│
外部サービス
└── OpenRouter API
```

## クライアントサイド実装

### useChatフックの使用

Vercel AI SDKの`useChat`フックを使用して、チャット機能を実装します。

#### 基本的な使用方法

```typescript
"use client";

import { useChat } from "@ai-sdk/react";

export function ChatUI() {
  const { messages, sendMessage, status, error } = useChat();

  return (
    <div>
      {/* メッセージ表示 */}
      <div>
        {messages.map((message) => (
          <div key={message.id}>
            <strong>{message.role}:</strong> {message.content}
          </div>
        ))}
      </div>

      {/* 入力フォーム */}
      <input
        type="text"
        onKeyDown={(e) => {
          if (e.key === "Enter") {
            sendMessage({ text: e.currentTarget.value });
            e.currentTarget.value = "";
          }
        }}
      />

      {/* ストリーミング状態表示 */}
      {status === "streaming" && <p>AIが応答中...</p>}

      {/* エラー表示 */}
      {error && <p>エラー: {error.message}</p>}
    </div>
  );
}
```

#### useChatフックのオプション

```typescript
const { messages, sendMessage, status, error } = useChat({
  // APIエンドポイント（デフォルト: /api/chat）
  api: "/api/chat",

  // 初期メッセージ
  initialMessages: [
    {
      id: "1",
      role: "system",
      parts: [{ type: "text", text: "あなたは親切なアシスタントです。" }],
    },
  ],

  // エラーハンドラー
  onError: (error) => {
    console.error("Chat error:", error);
  },

  // レスポンス完了時のコールバック
  onFinish: (message) => {
    console.log("Response finished:", message);
  },
});
```

### メッセージ送信

#### テキストメッセージの送信

```typescript
// 基本的な送信
sendMessage({ text: "こんにちは" });

// 変数を使用した送信
const userInput = "OpenRouterについて教えて";
sendMessage({ text: userInput });
```

#### 送信前のバリデーション

```typescript
function handleSubmit(text: string) {
  // 空文字チェック
  if (!text.trim()) {
    console.warn("Empty message");
    return;
  }

  // 長さチェック
  if (text.length > 10000) {
    console.error("Message too long");
    return;
  }

  sendMessage({ text });
}
```

### ストリーミング状態の管理

#### ステータスの確認

```typescript
const { status } = useChat();

switch (status) {
  case "ready":
    console.log("待機中");
    break;
  case "submitted":
    console.log("リクエスト送信済み");
    break;
  case "streaming":
    console.log("ストリーミング中");
    break;
  case "error":
    console.log("エラー発生");
    break;
}
```

#### ローディング状態の表示

```typescript
const { status } = useChat();
const isLoading = status === "streaming" || status === "submitted";

return (
  <div>
    <button disabled={isLoading}>
      {isLoading ? "送信中..." : "送信"}
    </button>
  </div>
);
```

#### ストリーミングの停止

```typescript
const { stop } = useChat();

<button onClick={() => stop()}>
  停止
</button>
```

### エラーハンドリング

#### エラーの表示

```typescript
const { error } = useChat();

if (error) {
  return (
    <div className="error">
      <h3>エラーが発生しました</h3>
      <p>{error.message}</p>
    </div>
  );
}
```

#### エラーからの回復

```typescript
const { error, regenerate } = useChat();

if (error) {
  return (
    <div className="error">
      <p>{error.message}</p>
      <button onClick={() => regenerate()}>
        再試行
      </button>
    </div>
  );
}
```

#### カスタムエラーハンドリング

```typescript
import { getErrorMessage } from "@/app/lib/errorHelpers";

const { error } = useChat();
const errorInfo = error ? getErrorMessage(error) : null;

if (errorInfo) {
  return (
    <div className="error">
      <h3>{errorInfo.title}</h3>
      <p>{errorInfo.message}</p>
      {errorInfo.actionText && (
        <button onClick={() => regenerate()}>
          {errorInfo.actionText}
        </button>
      )}
    </div>
  );
}
```

## サーバーサイド実装

### API Route実装

#### 基本的な実装

```typescript
// app/api/chat/route.ts
import { streamText, type UIMessage } from "ai";
import { getOpenRouterModel } from "@/app/lib/openrouter";

export async function POST(request: Request) {
  try {
    // リクエストボディのパース
    const { messages }: { messages: UIMessage[] } = await request.json();

    // OpenRouterモデルの取得
    const model = getOpenRouterModel();

    // ストリーミングレスポンスの生成
    const result = streamText({
      model,
      messages,
    });

    // レスポンスを返す
    return result.toUIMessageStreamResponse();
  } catch (error) {
    console.error("Chat API error:", error);

    return Response.json(
      {
        error: {
          type: "api_error",
          message: error instanceof Error ? error.message : "Unknown error",
          code: "INTERNAL_ERROR",
        },
      },
      { status: 500 }
    );
  }
}
```

#### バリデーション付き実装

```typescript
export async function POST(request: Request) {
  try {
    const { messages }: { messages: UIMessage[] } = await request.json();

    // バリデーション
    if (!messages || messages.length === 0) {
      return Response.json(
        {
          error: {
            type: "validation_error",
            message: "Messages array must not be empty",
            code: "INVALID_MESSAGES",
          },
        },
        { status: 400 }
      );
    }

    const model = getOpenRouterModel();
    const result = streamText({
      model,
      messages,
    });

    return result.toUIMessageStreamResponse();
  } catch (error) {
    console.error("Chat API error:", error);

    return Response.json(
      {
        error: {
          type: "api_error",
          message: error instanceof Error ? error.message : "Unknown error",
          code: "INTERNAL_ERROR",
        },
      },
      { status: 500 }
    );
  }
}
```

### OpenRouter統合

#### OpenRouterの初期化

```typescript
// app/lib/openrouter.ts
import { createOpenRouter } from "@openrouter/ai-sdk-provider";

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

#### モデルの取得

```typescript
export function getOpenRouterModel(modelName?: string) {
  const openrouter = initializeOpenRouter();
  const model = modelName || process.env.OPENROUTER_MODEL || DEFAULT_MODEL;

  return openrouter(model);
}
```

#### 高度な設定

```typescript
export function getOpenRouterModelWithSettings(
  modelName: string,
  settings?: {
    temperature?: number;
    maxTokens?: number;
    reasoning?: {
      enabled: boolean;
      effort: "high" | "medium" | "low";
    };
  }
) {
  const openrouter = initializeOpenRouter();

  return openrouter(modelName, {
    // 推論設定
    reasoning: settings?.reasoning,
    // 使用量トラッキング
    usage: {
      include: true,
    },
  });
}
```

### ストリーミングレスポンス

#### streamText関数の使用

```typescript
import { streamText, convertToModelMessages } from "ai";

const result = streamText({
  model: getOpenRouterModel(),
  messages: convertToModelMessages(messages),
  temperature: 0.7,
  maxTokens: 1000,
});
```

#### システムプロンプトの追加

```typescript
const result = streamText({
  model: getOpenRouterModel(),
  system: "あなたは親切で知識豊富なアシスタントです。",
  messages: convertToModelMessages(messages),
});
```

#### ストリーミングレスポンスの変換

```typescript
// UIMessage形式でストリーミング
return result.toUIMessageStreamResponse();

// テキストストリーム形式
return result.toTextStreamResponse();

// データストリーム形式
return result.toDataStreamResponse();
```

## コンポーネント実装例

### 完全なChatUIコンポーネント

```typescript
"use client";

import { useChat } from "@ai-sdk/react";
import { useState } from "react";
import { getErrorMessage } from "@/app/lib/errorHelpers";
import type { Message } from "@/app/types/chat";
import { ChatInput } from "./ChatInput";
import { ErrorMessage } from "./ErrorMessage";
import { MessageList } from "./MessageList";

export function ChatUI() {
  const [inputValue, setInputValue] = useState("");

  const { messages, sendMessage, status, error, regenerate, setMessages } =
    useChat();

  const isLoading = status === "streaming" || status === "submitted";

  const handleSubmit = () => {
    if (inputValue.trim()) {
      sendMessage({ text: inputValue });
      setInputValue("");
    }
  };

  const handleClearChat = () => {
    setMessages([]);
    setInputValue("");
  };

  // messagesをMessage型に変換（partsからテキストを抽出）
  const typedMessages: Message[] = messages.map((msg) => {
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

  const errorInfo = error ? getErrorMessage(error) : null;

  return (
    <div className="flex h-screen flex-col">
      {/* エラー表示 */}
      {error && errorInfo && (
        <ErrorMessage
          title={errorInfo.title}
          message={errorInfo.message}
          actionText={errorInfo.actionText}
          onRetry={() => regenerate()}
          onDismiss={() => {
            /* エラーは再試行で自動クリアされる */
          }}
        />
      )}

      {/* メッセージリスト */}
      <div className="flex-1 overflow-y-auto p-4">
        <MessageList messages={typedMessages} isStreaming={isLoading} />
      </div>

      {/* 入力エリア */}
      <div className="border-t p-4">
        <div className="flex gap-2">
          <div className="flex-1">
            <ChatInput
              value={inputValue}
              onChange={setInputValue}
              onSubmit={handleSubmit}
              disabled={isLoading}
            />
          </div>
          <button
            type="button"
            onClick={handleClearChat}
            disabled={messages.length === 0}
            className="rounded bg-gray-500 px-4 py-2 text-white hover:bg-gray-600 disabled:bg-gray-300 disabled:cursor-not-allowed whitespace-nowrap self-end"
          >
            クリア
          </button>
        </div>
      </div>
    </div>
  );
}
```

### ChatInputコンポーネント

```typescript
"use client";

interface ChatInputProps {
  value: string;
  onChange: (value: string) => void;
  onSubmit: () => void;
  disabled: boolean;
  placeholder?: string;
}

export function ChatInput({
  value,
  onChange,
  onSubmit,
  disabled,
  placeholder = "メッセージを入力...",
}: ChatInputProps) {
  const handleKeyDown = (e: React.KeyboardEvent<HTMLTextAreaElement>) => {
    // Enterキーで送信（Shift+Enterは改行）
    if (e.key === "Enter" && !e.shiftKey) {
      e.preventDefault();
      if (value.trim() && !disabled) {
        onSubmit();
      }
    }
  };

  return (
    <div className="flex gap-2">
      <textarea
        value={value}
        onChange={(e) => onChange(e.target.value)}
        onKeyDown={handleKeyDown}
        disabled={disabled}
        placeholder={placeholder}
        className="flex-1 rounded border p-2 resize-none"
        rows={3}
      />
      <button
        type="button"
        onClick={onSubmit}
        disabled={disabled || !value.trim()}
        className="rounded bg-blue-500 px-4 py-2 text-white hover:bg-blue-600 disabled:bg-gray-300 disabled:cursor-not-allowed"
      >
        送信
      </button>
    </div>
  );
}
```

### MessageListコンポーネント

```typescript
"use client";

import { useEffect, useRef } from "react";
import type { Message } from "@/app/types/chat";
import { MessageItem } from "./MessageItem";

interface MessageListProps {
  messages: Message[];
  isStreaming: boolean;
}

export function MessageList({ messages, isStreaming }: MessageListProps) {
  const bottomRef = useRef<HTMLDivElement>(null);

  // 新しいメッセージが追加されたら自動スクロール
  useEffect(() => {
    if (bottomRef.current && typeof bottomRef.current.scrollIntoView === "function") {
      bottomRef.current.scrollIntoView({ behavior: "smooth" });
    }
  }, [messages]);

  if (messages.length === 0) {
    return (
      <div className="text-center text-gray-500">
        メッセージがありません。会話を始めましょう！
      </div>
    );
  }

  return (
    <div className="space-y-4">
      {messages.map((message) => (
        <MessageItem key={message.id} message={message} />
      ))}
      {isStreaming && (
        <div className="text-gray-500">
          <span className="animate-pulse">●</span> AIが応答中...
        </div>
      )}
      <div ref={bottomRef} />
    </div>
  );
}
```

### MessageItemコンポーネント

```typescript
import type { Message } from "@/app/types/chat";

interface MessageItemProps {
  message: Message;
}

export function MessageItem({ message }: MessageItemProps) {
  const isUser = message.role === "user";

  return (
    <div className={`flex ${isUser ? "justify-end" : "justify-start"}`}>
      <div
        className={`max-w-[70%] rounded-lg p-3 ${
          isUser
            ? "bg-blue-500 text-white"
            : "bg-gray-200 text-gray-800"
        }`}
      >
        <div className="text-xs opacity-70 mb-1">
          {message.role === "user" ? "あなた" : "AI"}
        </div>
        <div className="whitespace-pre-wrap">{message.content}</div>
      </div>
    </div>
  );
}
```

## ベストプラクティス

### 1. エラーハンドリング

```typescript
// ✅ 推奨: 詳細なエラー情報を提供
try {
  const model = getOpenRouterModel();
} catch (error) {
  console.error("OpenRouter initialization error:", error);
  return Response.json(
    {
      error: {
        type: "api_error",
        message: error instanceof Error ? error.message : "Unknown error",
        code: "OPENROUTER_INIT_ERROR",
      },
    },
    { status: 500 }
  );
}

// ❌ 避けるべき: エラー情報が不十分
catch (error) {
  return Response.json({ error: "Error" }, { status: 500 });
}
```

### 2. 型安全性

```typescript
// ✅ 推奨: 明示的な型定義
interface ChatUIProps {
  initialMessages?: Message[];
}

// ❌ 避けるべき: any型の使用
const messages: any = [];
```

### 3. パフォーマンス最適化

```typescript
// ✅ 推奨: メモ化を使用
import { useMemo } from "react";

const typedMessages = useMemo(() => {
  return messages.map((msg) => convertToMessage(msg));
}, [messages]);

// ❌ 避けるべき: 毎回変換処理を実行
const typedMessages = messages.map((msg) => convertToMessage(msg));
```

### 4. セキュリティ

```typescript
// ✅ 推奨: APIキーはサーバーサイドで管理
const apiKey = process.env.OPENROUTER_API_KEY;

// ❌ 避けるべき: クライアントサイドでAPIキーを使用
const apiKey = "sk-or-v1-..."; // セキュリティリスク！
```

### 5. ユーザーエクスペリエンス

```typescript
// ✅ 推奨: ローディング状態を明示
{isLoading && <p>AIが応答中...</p>}

// ✅ 推奨: エラーからの回復手段を提供
{error && <button onClick={regenerate}>再試行</button>}

// ✅ 推奨: 入力バリデーション
if (!inputValue.trim()) {
  console.warn("Empty message");
  return;
}
```

## 参考リンク

- [APIリファレンス](./api-reference.md) - API仕様の詳細
- [型定義リファレンス](./type-definitions.md) - 型定義の詳細
- [OpenRouter統合ガイド](./openrouter-integration.md) - OpenRouterの設定
- [Vercel AI SDK Documentation](https://ai-sdk.dev/) - Vercel AI SDKの公式ドキュメント
- [Next.js App Router](https://nextjs.org/docs/app) - Next.jsの公式ドキュメント
