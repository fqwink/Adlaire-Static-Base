# Adlaire-Static-Base

Adlaire-Static-Base（ASB）は、セルフホスト環境向けの静的コンテンツ配信ホスティングシステムです。

ASB 本体はヘッドレスな Go 製 HTTPS サーバーとして、プロジェクト管理、ドメイン管理、無料独自SSL・SSL証明書管理、ファイル管理、GitHub Webhook、バックアップ、監視ログを提供することを目的とします。

ASB SDK は単一の公式SDKとして扱い、Browser JavaScript、Deno専用 TypeScript、Go の対応実装を提供します。各対応実装は ASB 管理 HTTPS JSON API と通信します。

ASB Web UI は、ASB 本体外の内製標準管理画面クライアントとして扱い、静的 HTML / CSS / JavaScript とブラウザ標準 API のみで実装します。ASB 標準Web UI は、ASB SDK の Browser JavaScript 実装を利用して ASB 管理 HTTPS JSON API と通信します。

外部開発者は、ASB SDK を使用する限り、ASB 標準Web UIのカスタマイズまたは独自フロントエンド実装を自由に行えます。ただし、外部開発者の成果物は Adlaire Group が開発元の公式 ASB、公式 ASB 標準Web UI、公式 ASB SDK、公式仕様の一部として扱いません。

## Status

現在は仕様・実装タスク整理段階です。Go実装コードはまだ作成されていません。

## Documents

- `ASB-spec.md`: 仕様正本（Rev.59）
- `ASB-spec.html`: `ASB-spec.md` から生成するHTML版仕様書
- `DOCUMENT_INDEX.md`: 文書索引
- `IMPLEMENTATION_TASKS.md`: `ASB-spec.md` に基づく実装フェーズ別タスクリスト
- `AGENTS.md`: 最上位作業ルールブック
