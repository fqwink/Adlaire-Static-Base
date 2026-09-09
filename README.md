# Adlaire-Static-Base

Adlaire-Static-Base（ASB）は、セルフホスト環境向けの静的コンテンツ配信ホスティングシステムです。

ASB 本体はヘッドレスな Go 製 HTTP サーバーとして、プロジェクト管理、ドメイン管理、無料独自SSL・SSL証明書管理、ファイル管理、GitHub Webhook、バックアップ、監視ログを提供することを目的とします。

ASB Web UI は、ASB 本体外の内製管理画面クライアントとして扱い、ブラウザ専用 JavaScript SDK である ASB SDK を利用して ASB 管理 HTTP JSON API と通信します。

## Status

現在は仕様・実装タスク整理段階です。Go実装コードはまだ作成されていません。

## Documents

- `ASB-spec.md`: 仕様正本（Rev.47）
- `ASB-spec.html`: `ASB-spec.md` から生成するHTML版仕様書
- `DOCUMENT_INDEX.md`: 文書索引
- `IMPLEMENTATION_TASKS.md`: `ASB-spec.md` に基づく実装フェーズ別タスクリスト
- `AGENTS.md`: 最上位作業ルールブック
