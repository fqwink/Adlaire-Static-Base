# Adlaire-Static-Base

Adlaire-Static-Base（ASB）は、セルフホスト環境向けの静的コンテンツ配信ホスティングシステムです。

ASB 本体はヘッドレスな Go 製 HTTPS サーバーとして、プロジェクト管理、ドメイン管理、無料独自SSL・SSL証明書管理、ファイル管理、GitHub Webhook、バックアップ、監視ログを提供することを目的とします。

ASB は単一ユーザー・単一システム管理者モデルとし、管理 HTTPS JSON API へのアクセスでは毎回システム管理者パスワードを要求します。初期デフォルトパスワードは bootstrap 用で、通常運用前に変更必須です。

ASB SDK は単一の公式SDKとして扱い、Browser JavaScript、Deno専用 TypeScript、Go の対応実装を提供します。各対応実装は ASB 管理 HTTPS JSON API と通信します。

ASB Web UI は、ASB 本体外の内製標準管理画面クライアントとして扱い、静的 HTML / CSS / JavaScript とブラウザ標準 API のみで実装します。ASB 標準Web UI は、ASB SDK の Browser JavaScript 実装を利用して ASB 管理 HTTPS JSON API と通信します。

Rev.75 では、実行時データの初期化・自動生成・不足補完を禁止し、起動設定ファイルと `storage.basePath` 配下の実行時 JSON を分離しています。実行時 JSON、必須ディレクトリ、ログファイルは ASB 起動前に運用者が事前配置します。

旧 Auteur 仕様は ASB に仕様のみを吸収し、`https://github.com/fqwink/Auteur` を移管元として記録します。旧名称の別枠は作らず、Content Pipeline、Site Routing、Site Rendering、Site Output、Blog、Docs、Sitemap、Ad Slot などの ASB 通常機能名へ分解します。Auteur の source code、runtime、CLI、fixture、CI、package、設定ファイル、生成物は移管しません。

外部開発者は、ASB SDK を使用する限り、ASB 標準Web UIのカスタマイズまたは独自フロントエンド実装を自由に行えます。ただし、外部開発者の成果物は Adlaire Group が開発元の公式 ASB、公式 ASB 標準Web UI、公式 ASB SDK、公式仕様の一部として扱いません。

## Status

現在は仕様・実装タスク整理段階です。Go実装コードはまだ作成されていません。

## Documents

- `ASB-spec.md`: 仕様正本（Rev.75）
- `ASB-spec.html`: `ASB-spec.md` から生成するHTML版仕様書
- `DOCUMENT_INDEX.md`: 文書索引
- `IMPLEMENTATION_TASKS.md`: `ASB-spec.md` に基づく実装フェーズ別タスクリスト
- `AGENTS.md`: 最上位作業ルールブック
