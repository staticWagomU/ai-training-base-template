# OpenRouter チャットアプリケーション ドキュメント

このディレクトリには、OpenRouterとVercel AI SDKを統合したチャットアプリケーションの技術ドキュメントが含まれています。

## 📚 ドキュメント一覧

### 1. [OpenRouter統合ガイド](./openrouter-integration.md)
OpenRouterの統合方法と設定について説明します。

**内容**:
- 環境設定（APIキー、モデル設定）
- OpenRouterの初期化方法
- モデルの選択と変更方法
- 利用可能なモデル一覧
- 高度な設定オプション

**対象読者**: 開発者、システム管理者

---

### 2. [APIリファレンス](./api-reference.md)
チャットAPIのエンドポイント仕様とレスポンス形式について説明します。

**内容**:
- `/api/chat` エンドポイント仕様
- リクエスト/レスポンス形式
- ストリーミングレスポンスの詳細
- エラーレスポンス一覧
- Server-Sent Events (SSE) の説明

**対象読者**: 開発者、APIユーザー

---

### 3. [型定義リファレンス](./type-definitions.md)
アプリケーションで使用される全ての型定義について説明します。

**内容**:
- アプリケーション型定義（Message, ChatRequest, ErrorType等）
- Vercel AI SDK型定義（UIMessage, UseChatReturn等）
- OpenRouter AI SDK Provider型定義
- 型変換の例
- 型安全性のベストプラクティス

**対象読者**: 開発者

---

### 4. [実装ガイド](./implementation-guide.md)
クライアントサイドとサーバーサイドの実装方法を詳しく説明します。

**内容**:
- `useChat`フックの使用方法
- メッセージ送信とストリーミング状態管理
- エラーハンドリング実装
- API Route実装
- コンポーネント実装例
- ベストプラクティス

**対象読者**: 開発者

## 🚀 クイックスタート

### 1. 環境設定

まず[OpenRouter統合ガイド](./openrouter-integration.md)を参照して、APIキーを設定します：

```bash
# .env.local
OPENROUTER_API_KEY=sk-or-v1-your-api-key-here
OPENROUTER_MODEL=anthropic/claude-3.7-sonnet:thinking
```

### 2. 基本的な使用方法

クライアントサイドで`useChat`フックを使用：

```typescript
import { useChat } from "@ai-sdk/react";

const { messages, sendMessage, status } = useChat();
```

詳細は[実装ガイド](./implementation-guide.md)を参照してください。

### 3. APIエンドポイント

`POST /api/chat`にメッセージを送信してAIレスポンスを取得。

詳細は[APIリファレンス](./api-reference.md)を参照してください。

## 📖 ドキュメントの読み方

### 初めて使う場合
1. [OpenRouter統合ガイド](./openrouter-integration.md) - 環境設定から始める
2. [実装ガイド](./implementation-guide.md) - 基本的な実装方法を学ぶ

### 開発中
- [型定義リファレンス](./type-definitions.md) - 型定義を確認する
- [APIリファレンス](./api-reference.md) - API仕様を確認する
- [実装ガイド](./implementation-guide.md) - コンポーネント実装例を参照する

### トラブルシューティング
- [OpenRouter統合ガイド - エラーハンドリング](./openrouter-integration.md#エラーハンドリング)
- [実装ガイド - ベストプラクティス](./implementation-guide.md#ベストプラクティス)

## 🔗 関連リソース

### 公式ドキュメント
- [OpenRouter公式サイト](https://openrouter.ai/)
- [OpenRouter API Documentation](https://openrouter.ai/docs)
- [Vercel AI SDK Documentation](https://ai-sdk.dev/)
- [Next.js Documentation](https://nextjs.org/docs)

### パッケージ
- [@openrouter/ai-sdk-provider](https://www.npmjs.com/package/@openrouter/ai-sdk-provider)
- [ai (Vercel AI SDK)](https://www.npmjs.com/package/ai)
- [@ai-sdk/react](https://www.npmjs.com/package/@ai-sdk/react)

## 🏗️ アーキテクチャ概要

```
クライアント (React)
  ├── ChatUI Component
  └── useChat Hook (Vercel AI SDK)
      ↓ HTTP POST
API Layer (Next.js)
  └── /api/chat Route Handler
      ↓ streamText()
AI SDK Integration
  └── Vercel AI SDK
      ↓ createOpenRouter()
External Service
  └── OpenRouter API
      └── 数百のAIモデルへのアクセス
```

## 📝 使用技術

| 技術 | バージョン | 用途 |
|------|-----------|------|
| Next.js | 15.5.4 | フレームワーク |
| React | 19.1.0 | UIライブラリ |
| Vercel AI SDK | 5.0.60 | AI統合 |
| @ai-sdk/react | 2.0.60 | React統合 |
| @openrouter/ai-sdk-provider | 1.2.0 | OpenRouter統合 |
| TypeScript | 5.x | 型安全性 |

## 💡 よくある質問

### Q. どのAIモデルが使えますか？
A. OpenRouterは数百のモデルをサポートしています。詳細は[OpenRouter Models](https://openrouter.ai/models)を参照してください。

### Q. APIキーはどこで取得できますか？
A. [OpenRouter Keys](https://openrouter.ai/keys)ページで取得できます。

### Q. ストリーミングレスポンスの仕組みは？
A. Server-Sent Events (SSE) を使用しています。詳細は[APIリファレンス](./api-reference.md#ストリーミングの詳細)を参照してください。

### Q. エラーが発生した場合は？
A. [実装ガイド - エラーハンドリング](./implementation-guide.md#エラーハンドリング)を参照してください。

## 📄 ライセンス

このドキュメントは、プロジェクトのライセンスに従います。

## 🤝 貢献

ドキュメントの改善提案は、プロジェクトのIssueまたはPull Requestで受け付けています。

---

最終更新: 2025-10-14
