# Adlaire-Static-Base

Adlaire-Static-Base（ASB）は、セルフホスト環境向けの静的コンテンツ配信ホスティングシステムです。

ASB 本体はヘッドレスな Go 製 HTTPS サーバーとして、プロジェクト管理、ドメイン管理、無料独自SSL・SSL証明書管理、ファイル管理、GitHub Webhook、バックアップ、監視ログを提供することを目的とします。

ASB は単一ユーザー・単一システム管理者モデルとし、管理 HTTPS JSON API へのアクセスでは毎回システム管理者パスワードを要求します。初期デフォルトパスワードは bootstrap 用で、通常運用前に変更必須です。

ASB SDK は単一の公式SDKとして扱い、Browser JavaScript、Deno専用 TypeScript、Go の対応実装を提供します。各対応実装は ASB 管理 HTTPS JSON API と通信します。SDK専用プロトコル、SDK専用エンドポイント、SDK専用保存JSONは持ちません。

ASB Web UI は、ASB 本体外の内製標準管理画面クライアントとして扱い、静的 HTML / CSS / JavaScript とブラウザ標準 API のみで実装します。ASB 標準Web UI は、ASB SDK の Browser JavaScript 実装を利用して ASB 管理 HTTPS JSON API と通信します。

Rev.90 では、READMEを仕様判断の正本ではなく入口文書として位置づけ、ASB本体、ASB SDK、ASB標準Web UI、将来計画、実装フェーズ管理の境界を仕様正本と一致させています。

旧 Auteur 仕様は ASB に仕様のみを吸収し、`https://github.com/fqwink/Auteur` を移管元として記録します。旧名称の別枠は作らず、Content Pipeline、Site Routing、Site Rendering、Site Output、Blog、Docs、Sitemap、Ad Slot などの ASB 通常機能名へ分解します。Auteur の source code、runtime、CLI、fixture、CI、package、設定ファイル、生成物は移管しません。

外部開発者は、ASB SDK を使用する限り、ASB 標準Web UIのカスタマイズまたは独自フロントエンド実装を自由に行えます。ただし、外部開発者の成果物は Adlaire Group が開発元の公式 ASB、公式 ASB 標準Web UI、公式 ASB SDK、公式仕様の一部として扱いません。

## Status

現在は仕様・実装タスク整理段階です。Go実装コードはまだ作成されていません。

## Documents

- `ASB-spec.md`: 仕様正本（Rev.90）
- `ASB-spec.html`: `ASB-spec.md` から生成するHTML版仕様書
- `DOCUMENT_INDEX.md`: 文書索引
- `IMPLEMENTATION_TASKS.md`: `ASB-spec.md` に基づく実装フェーズ別タスクリスト
- `AGENTS.md`: 最上位作業ルールブック

## Document Roles

README は、ASB の概要、現在状態、主要文書への入口を示す文書です。仕様判断、実装判断、フェーズ判断の正本ではありません。

仕様内容は `ASB-spec.md` を正とします。実装フェーズ番号、優先度、開発版バージョン、フェーズ別タスク、フェーズ別完了条件は `IMPLEMENTATION_TASKS.md` を正とします。`ASB-spec.html` は `ASB-spec.md` から生成される閲覧用HTMLです。

## Implementation Order

実装は `IMPLEMENTATION_TASKS.md` のフェーズ順に進めます。現在の順序は、基盤、API・JSON・起動検証・単一システム管理者認証、静的配信・ファイル管理、ドメイン管理・SSL状態基盤、ログ・監視、バックアップ・復旧、GitHub Webhook、無料独自SSL / ACME、マイグレーション、配布、SDK、標準Web UI、禁止機能確認です。
