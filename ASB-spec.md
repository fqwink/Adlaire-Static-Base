# セルフホスト型静的コンテンツ配信ホスティングシステム仕様書

## 1 本書の位置づけ

本書は、セルフホスト型静的コンテンツ配信ホスティングシステム「Adlaire-Static-Base（ASB）」の仕様正本である。

本システムはゼロベースで設計され、完全な内製化を原則とする。本書には、ASB の確定仕様、将来計画、保留事項を記載する。

### 1.1 機能一覧

| 区分 | 主要機能 |
|-----|--------|
| プロジェクト管理 | プロジェクト作成・削除・情報取得 |
| ドメイン管理 | DNS ドメイン割り当て・管理 |
| SSL管理 | SSL証明書管理境界 |
| ファイル管理 | ファイルアップロード・削除・圧縮 |
| Webhook | GitHub自動デプロイ処理 |
| バックアップ・復旧 | データ保全・復旧手順 |
| 監視・ログ | システム監視・アクセスログ・エラーログ |

### 1.2 仕様駆動システム

ASB は仕様駆動システムである。

- 仕様正本を最上位の判断基準とする
- 仕様に存在しない機能は実装前に仕様正本へ明記する
- 仕様と実装に不整合がある場合は仕様正本を正とする

---

## 2 プロジェクト情報

| 項目 | 内容 |
|-----|------|
| プロジェクト名称 | Adlaire-Static-Base（ASB） |
| 開発者 | 倉田和宏 |
| 実装言語 | Go |
| Go固定採用バージョン | 1.21 以上 |
| 提供形態 | HTTP サーバー（単一バイナリ） |
| データ保存 | JSON ファイルベース（外部DB不使用） |
| ライセンス | クローズドライセンス |
| 本書バージョン | Rev.32 |

---

## 3 ASB の責務範囲

### 3.1 ASB の目的および責任範囲

**目的**

ASB は、セルフホスト環境において以下を提供する：
- プロジェクト単位での静的コンテンツ管理
- 複数ドメイン対応とSSL証明書管理
- GitHub自動デプロイ機能
- システムモニタリングとバックアップ

**責任範囲（担う責務）**

| 責務 | 説明 |
|-----|------|
| プロジェクト管理 | プロジェクト作成・削除・情報管理 |
| ドメイン管理 | DNS ドメイン割り当て・管理 |
| SSL管理 | SSL証明書管理境界 |
| ファイル管理 | ファイルアップロード・削除・圧縮 |
| Webhook 処理 | GitHub との連携・自動デプロイ |
| バックアップ・復旧 | データ保全・復旧手順 |
| 監視・ログ | システム監視・アクセスログ・エラーログ |

**責任範囲外（担わない責務）**

| 対象 | 理由 |
|-----|------|
| GUI・UI | バックエンドのみ提供 |
| ユーザー認証・管理 | セルフホスト・スタンドアロン運用 |
| 組織・チーム管理 | 単一ユーザー想定 |

### 3.2 ASB互換目標と ASB 独自仕様

ASB は、静的コンテンツ専用ホスティングとして XServer Static 等の利用体験を参考基準とし、ASB互換目標として定義する。

本節の「ASB互換目標」とは、外部サービスとの完全互換ではなく、セルフホスト環境で同種の運用体験を提供するための機能目標を指す。

ASB互換目標は将来の到達目標であり、Rev.32 時点の実装対象ではない。

ASB互換目標に含まれることは、未確定機能を実装してよい根拠にならない。

ASB互換目標に含まれる機能を実装対象へ昇格する場合は、事前に `ASB-spec.md` を改訂し、実装範囲、入出力、保存形式、副作用、生成物、テスト条件を確定しなければならない。

**ASB互換目標**

- 静的コンテンツ専用ホスティングとして動作する
- HTML、CSS、JavaScript、画像等の静的ファイルを配信する
- プロジェクト単位で公開対象を管理する
- 独自ドメインをプロジェクトへ割り当てる
- SSL証明書の管理に対応する
- SSL / ACME 相当の運用体験を目標に含める
- ACME による証明書取得・更新を目標に含める
- ワイルドカード証明書を目標に含める
- 複数 CA 対応を目標に含める
- HTTP/2 対応を目標とする
- GitHub 連携による自動デプロイを提供する
- ファイルアップロードおよびフォルダ階層を保持したファイル管理を提供する
- SSL更新状態、デプロイ状態、ログを確認できる

**Rev.32 時点で ASB互換目標に含めないもの**

- 外部サービスとのAPI完全互換
- 外部サービスの管理画面互換
- 外部サービスの課金、契約、アカウント管理互換
- 外部サービスのDNS管理機能互換
- 外部サービスのCDN機能完全互換
- 外部サービスの SLA / サポート体制互換
- 外部サービス固有の内部実装再現

**ASB互換目標による実装禁止**

ASB互換目標を理由に、以下を追加してはならない。

- `.gitignore`
- 外部DB
- 未承認の外部ライブラリ
- 未承認の外部サービス連携
- 開発リポジトリ内の実行時データ
- 起動時の実行時データ自動生成
- ビルド成果物の開発リポジトリ内自動生成
- APIキー管理
- ユーザー認証
- Rate limiting
- Brotli 圧縮
- ACME 実通信
- CA 選定
- SDK 本体
- SDK 専用通信

**ASB 独自仕様**

- セルフホスト型として動作する
- Go 単一バイナリとして提供する
- JSON ファイルベースで実行時データを管理し、外部DBを使用しない
- 実行時データは `storage.basePath` 配下にのみ配置する
- 開発リポジトリを実行時データ保存先として扱わない
- 起動時に実行時データ用ディレクトリまたはファイルを自動生成しない
- `.gitignore` を作成・使用しない
- ASB API を外部操作面とするヘッドレス構成を採用する
- Go標準ライブラリで実装可能な部分は Go標準ライブラリで実装する
- SDK 本体、SDK 配布、SDK 認証、CA、ACME 実通信、証明書自動更新など詳細未確定の機能は、仕様確定後に実装対象へ昇格する

---

## 4 設計思想

### 4.1 内製化・ゼロ外部依存設計

ASB は完全内製化を原則とする。

- 外部ライブラリは最小限（Go標準ライブラリ中心）
- HTTP サーバーとして単一バイナリ化
- セルフホスト・スタンドアロン運用を前提
- ヘッドレスアーキテクチャ採用（UI に依存しない）
- JSON ファイルで全データを管理

### 4.2 設計思想：責務駆動設計

ASB は責務駆動設計（Responsibility-Driven Design）を採用する。

**原則**
- 各責務を明確に定義
- 責務ごとにコンポーネント化
- コンポーネントをライブラリとしてパッケージ化
- 責務間は Go interface で疎結合
- 循環依存を禁止

**メリット**
- 責務が明確で保守性が高い
- コンポーネント単位のテストが容易
- 将来の再利用性を確保
- スケーラビリティが高い

---

## 5 技術スタック

### 5.1 バックエンド

| 項目 | 内容 |
|-----|------|
| 実装言語 | Go（1.21 以上） |
| HTTP サーバー | Go 標準 `net/http` |
| JSON 処理 | Go 標準 `encoding/json` |
| ファイル操作 | Go 標準 `os`, `io` |
| 圧縮 | Go 標準 `archive/tar`, `compress/gzip` |
| 暗号化 | Go 標準 `crypto` |
| SSL 証明書 | SSL証明書管理境界（ACME 実通信・CA選定・証明書自動更新は Rev.32 時点では実装しない） |
| 対応 OS | Linux |
| 対応アーキテクチャ | amd64（x86_64）、arm64（aarch64） |

### 5.2 外部ライブラリポリシー

- 外部ライブラリ禁止（原則）
- Go 標準ライブラリも最小限に使用
- 理由：外部依存を禁止、シンプル性・内製化を優先
- 例外：「例外外部ライブラリリスト」に記載されたライブラリのみ採用可能
- 例外採用時：最新の安定版リリースのみ使用
- CGO 要件は避ける

### 5.2.1 例外外部ライブラリリスト

| ライブラリ | 用途 | 状態 |
|-----------|------|------|
| （現在：指定なし） | - | - |

注：追加の際は、必須性・メンテナンス・ライセンスを確認の上、仕様に追記

### 5.3 ビルド

```sh
# Linux amd64
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o asb-linux-amd64 ./cmd/asb

# Linux arm64
CGO_ENABLED=0 GOOS=linux GOARCH=arm64 go build -o asb-linux-arm64 ./cmd/asb
```

---

## 6 ディレクトリ構成

```
static-hosting-system/
├── main.go             # エントリーポイント（config + crypto + router + server 統合）

# Management Domain（プロジェクト・ドメイン・SSL管理）
├── management/
│   ├── project_handler.go
│   ├── project_service.go
│   ├── project_entity.go
│   ├── domain_handler.go
│   ├── domain_service.go
│   ├── domain_entity.go
│   ├── ssl_handler.go
│   ├── ssl_service.go
│   └── ssl_entity.go

# Delivery Domain（ファイル・Webhook管理）
├── delivery/
│   ├── file_handler.go
│   ├── file_service.go
│   ├── file_entity.go
│   ├── webhook_handler.go
│   ├── webhook_service.go
│   └── webhook_entity.go

# Data Domain（バックアップ・ストレージ管理）
├── data/
│   ├── backup_handler.go
│   ├── backup_service.go
│   ├── backup_entity.go
│   ├── storage_handler.go
│   ├── storage_service.go
│   └── storage_entity.go

# System Domain（監視・ログ管理）
├── system/
│   ├── monitor_handler.go
│   ├── monitor_service.go
│   ├── monitor_entity.go
│   ├── logger_handler.go
│   ├── logger_service.go
│   └── logger_entity.go

# Go Project Files
├── go.mod
├── go.sum
└── README.md
```

### 責務駆動設計アーキテクチャ

ASB は責務駆動設計の思想に基づき、4つのドメイングループに組織された責務ごとに実装する。

**4ドメイングループ**
- Management Domain：プロジェクト・ドメイン・SSL管理
- Delivery Domain：ファイル・Webhook管理
- Data Domain：バックアップ・ストレージ管理
- System Domain：監視・ログ管理

**実装パターン**

各責務は以下の層構造を持つ：
- Handler（API エンドポイント）
- Service（ビジネスロジック）
- Entity（データ構造定義）

各層は責務の分離と再利用性を確保するよう設計される。ドメイン間の接続は main.go で一元管理。

### 実行時データの配置と起動時検証

ASB は起動時に、実行時データ用のディレクトリまたはファイルを自動生成しない。

ASB 起動時には、設定された保存先に以下の実行時データ領域が存在すること、および必要な読み書き権限を持つことを検証する。存在しない、または権限が不足する場合は起動失敗とする：

- `config/` - 設定ファイル（projects.json、backups.json、config.json等）
- `storage/` - プロジェクトファイルストレージ
- `logs/` - ログファイル（access.log、error.log等）
- `certs/` - SSL証明書ストレージ

実行時データは JSON ファイルベースで管理し、外部DBは使用しない。実行時データの保存先は `storage.basePath` によって指定する。

## 7 機能仕様

### 7.1 プロジェクト管理

**プロジェクト構造**
```json
{
  "id": "proj-001",
  "name": "my-blog",
  "quota": 1073741824,
  "used": 536870912,
  "domains": ["my-blog.example.com"],
  "createdAt": "2025-01-15T10:00:00Z"
}
```

**操作**
- 作成：名前、割り当てディスク容量を指定
- 削除：プロジェクト・ドメイン・ファイル・バックアップ全削除
- 情報取得：プロジェクト詳細・ディスク使用状況

### 7.2 ドメイン管理

**機能**
- 複数ドメイン割り当て対応
- サブドメイン対応
- SSL 証明書管理対象ドメイン対応

**ドメイン構造**
```json
{
  "domain": "my-blog.example.com",
  "projectId": "proj-001",
  "isCustom": true,
  "sslCert": "cert-001",
  "createdAt": "2025-01-15T10:00:00Z"
}
```

### 7.3 SSL 管理

**SSL 管理境界**
- 証明書IDと有効期限の管理
- 証明書保存先 `certs/` の検証
- 証明書ファイルの存在、パス、有効期限の検証
- 証明書生成・更新は ASB 外部の手動配置または外部運用で扱う
- ACME 実通信は Rev.32 時点では ASB 本体に実装しない

### 7.4 ファイル管理

**ファイルアップロード**
- 最大容量：1GB/プロジェクト（設定可能）
- 形式：制限なし（HTML, CSS, JavaScript, 画像等）
- 圧縮：Gzip による自動圧縮
- Brotli は Rev.32 時点では ASB 本体に実装しない

**ファイル削除**
- 個別削除、一括削除に対応
- 削除ログを記録

### 7.5 GitHub Webhook

**機能**
- Push イベント検出
- 指定ブランチの変更を自動デプロイ
- デプロイ成功・失敗をログ記録

### 7.6 バックアップ・復旧

**バックアップ対象**
- config/（設定・管理JSON）
- storage/（プロジェクトファイルデータ）

**バックアップ形式**
- tar.gz 形式で全データをアーカイブ
- 整合性検証：SHA-256 ハッシュを付与

**復旧手順**
1. バックアップファイルを検証
2. 既存データを退避
3. アーカイブを展開
4. 整合性確認後に完了

### 7.7 システム監視

**監視項目**
- CPU 使用率
- メモリ使用率
- ディスク使用率
- 接続数・リクエスト数

### 7.8 ログ管理

**ログ出力**
- アクセスログ：全 HTTP リクエスト
- エラーログ：エラー・例外・警告

---

## 8 API 仕様

### 8.1 プロジェクト API

**POST /api/projects - プロジェクト作成**

| 項目 | 内容 |
|-----|------|
| リクエスト | `{ "name": "string", "quota": number }` |
| レスポンス | `{ "id": "string", "name": "string", ... }` |
| ステータス | 201 Created, 400 Bad Request |

**リクエスト例**
```json
{
  "name": "my-blog",
  "quota": 1073741824
}
```

**レスポンス例（成功）**
```json
{
  "id": "proj-001",
  "name": "my-blog",
  "quota": 1073741824,
  "used": 0,
  "domains": [],
  "createdAt": "2025-01-15T10:00:00Z"
}
```

**レスポンス例（エラー）**
```json
{
  "error": "Invalid project name",
  "code": "ERR_INVALID_PROJECT_NAME",
  "message": "Project name must be 1-255 characters, alphanumeric, hyphen, underscore only",
  "httpStatus": 400
}
```

**GET /api/projects - プロジェクト一覧**

| 項目 | 内容 |
|-----|------|
| リクエスト | なし |
| レスポンス | `{ "projects": [ ... ] }` |
| ステータス | 200 OK |

**レスポンス例**
```json
{
  "projects": [
    {
      "id": "proj-001",
      "name": "my-blog",
      "quota": 1073741824,
      "used": 536870912,
      "domains": ["my-blog.example.com"],
      "createdAt": "2025-01-15T10:00:00Z"
    }
  ]
}
```

**DELETE /api/projects/:id - プロジェクト削除**

| 項目 | 内容 |
|-----|------|
| リクエスト | ID (URL パラメータ) |
| レスポンス | `{ "status": "deleted" }` |
| ステータス | 200 OK, 404 Not Found |

**レスポンス例（成功）**
```json
{
  "status": "deleted",
  "projectId": "proj-001",
  "deletedAt": "2025-01-15T10:05:00Z"
}
```

**レスポンス例（エラー）**
```json
{
  "error": "Project not found",
  "code": "ERR_PROJECT_NOT_FOUND",
  "projectId": "proj-999",
  "httpStatus": 404
}
```

### 8.2 ドメイン API

**POST /api/projects/:id/domains - ドメイン追加**
| 項目 | 内容 |
|-----|------|
| リクエスト | `{ "domain": "string" }` |
| レスポンス | `{ "domain": "string", "isCustom": boolean, ... }` |
| ステータス | 201 Created, 400 Bad Request |

**GET /api/projects/:id/domains - ドメイン一覧**
| 項目 | 内容 |
|-----|------|
| リクエスト | ID (URL パラメータ) |
| レスポンス | `{ "domains": [ ... ] }` |
| ステータス | 200 OK |

**DELETE /api/projects/:id/domains/:domain - ドメイン削除**
| 項目 | 内容 |
|-----|------|
| リクエスト | ID, domain (URL パラメータ) |
| レスポンス | `{ "status": "deleted" }` |
| ステータス | 200 OK, 404 Not Found |

### 8.3 ファイル API

**POST /api/projects/:id/files/upload - ファイルアップロード**
| 項目 | 内容 |
|-----|------|
| リクエスト | `multipart/form-data (file)` |
| レスポンス | `{ "id": "string", "name": "string", "size": number, ... }` |
| ステータス | 201 Created, 413 Payload Too Large |

**GET /api/projects/:id/files - ファイル一覧**
| 項目 | 内容 |
|-----|------|
| リクエスト | ID (URL パラメータ) |
| レスポンス | `{ "files": [ ... ] }` |
| ステータス | 200 OK |

**DELETE /api/projects/:id/files/:name - ファイル削除**
| 項目 | 内容 |
|-----|------|
| リクエスト | ID, name (URL パラメータ) |
| レスポンス | `{ "status": "deleted" }` |
| ステータス | 200 OK, 404 Not Found |

### 8.4 バックアップ API

**GET /api/backups - バックアップ一覧**
| 項目 | 内容 |
|-----|------|
| リクエスト | なし |
| レスポンス | `{ "backups": [ { "id": "string", "projectId": "string", ... } ] }` |
| ステータス | 200 OK |

**POST /api/backups/restore/:id - バックアップ復旧**
| 項目 | 内容 |
|-----|------|
| リクエスト | ID (URL パラメータ) |
| レスポンス | `{ "status": "restored" }` |
| ステータス | 200 OK, 404 Not Found |

### 8.5 システム API

**GET /api/monitoring/stats - システムステータス**
| 項目 | 内容 |
|-----|------|
| リクエスト | なし |
| レスポンス | `{ "cpu": number, "memory": number, "disk": number, ... }` |
| ステータス | 200 OK |

**GET /api/logs/access - アクセスログ**
| 項目 | 内容 |
|-----|------|
| リクエスト | `limit`, `offset` (クエリパラメータ) |
| レスポンス | `{ "logs": [ { "timestamp": "string", "method": "string", ... } ] }` |
| ステータス | 200 OK |

**GET /api/logs/error - エラーログ**
| 項目 | 内容 |
|-----|------|
| リクエスト | `limit`, `offset` (クエリパラメータ) |
| レスポンス | `{ "logs": [ { "timestamp": "string", "level": "string", "message": "string" } ] }` |
| ステータス | 200 OK |

### 8.6 Webhook API

**POST /api/webhook/github - GitHub Webhook**
| 項目 | 内容 |
|-----|------|
| リクエスト | GitHub Webhook ペイロード |
| レスポンス | `{ "status": "deployed" }`、`{ "status": "ignored" }`、`{ "status": "duplicate" }` |
| ステータス | 200 OK, 400 Bad Request, 401 Unauthorized, 500 Internal Server Error |

### 8.7 エラーハンドリング

**共通エラーレスポンス形式**

```json
{
  "error": "string (エラーメッセージ)",
  "code": "string (エラーコード)",
  "timestamp": "string (ISO 8601)"
}
```

**標準HTTPステータスコード**

| コード | 意味 | 用途 |
|------|------|------|
| 200 | OK | リクエスト成功 |
| 201 | Created | リソース作成成功 |
| 400 | Bad Request | リクエスト形式エラー（バリデーション失敗等） |
| 401 | Unauthorized | Webhook署名検証失敗 |
| 404 | Not Found | リソース未検出 |
| 409 | Conflict | リソース競合（既存データの重複等） |
| 413 | Payload Too Large | ファイルサイズ超過 |
| 500 | Internal Server Error | サーバー内部エラー |
| 503 | Service Unavailable | サーバー一時利用不可 |

**エラーコード定義**

| エラーコード | HTTPステータス | メッセージ | 原因 |
|------------|--------------|---------|------|
| ERR_INVALID_PROJECT_NAME | 400 | Invalid project name | プロジェクト名が規則に違反 |
| ERR_PROJECT_NOT_FOUND | 404 | Project not found | プロジェクトが存在しない |
| ERR_PROJECT_ALREADY_EXISTS | 409 | Project already exists | プロジェクト名が重複 |
| ERR_QUOTA_EXCEEDED | 413 | Disk quota exceeded | ディスク容量超過 |
| ERR_INVALID_DOMAIN | 400 | Invalid domain format | ドメイン形式が不正 |
| ERR_DOMAIN_NOT_FOUND | 404 | Domain not found | ドメインが存在しない |
| ERR_DOMAIN_ALREADY_ASSIGNED | 409 | Domain already assigned | ドメインが既に割り当て済み |
| ERR_FILE_NOT_FOUND | 404 | File not found | ファイルが存在しない |
| ERR_FILE_UPLOAD_FAILED | 500 | File upload failed | ファイルアップロード失敗 |
| ERR_INVALID_FILE_SIZE | 413 | File size exceeds limit | ファイルサイズ超過 |
| ERR_BACKUP_NOT_FOUND | 404 | Backup not found | バックアップが存在しない |
| ERR_BACKUP_RESTORE_FAILED | 500 | Backup restore failed | バックアップ復旧失敗 |
| ERR_SSL_CERT_GENERATION_FAILED | 500 | SSL certificate generation failed | SSL証明書生成失敗 |
| ERR_WEBHOOK_SIGNATURE_INVALID | 401 | Invalid webhook signature | Webhook署名が不正 |
| ERR_WEBHOOK_SOURCE_INVALID | 500 | Webhook source invalid | Webhookデプロイ元が不正 |
| ERR_WEBHOOK_PROJECT_NOT_CONFIGURED | 500 | Webhook project not configured | Webhookデプロイ先Projectが未設定 |
| ERR_WEBHOOK_PROCESSING_FAILED | 500 | Webhook processing failed | Webhook処理失敗 |
| ERR_INVALID_JSON | 400 | Invalid JSON | JSON構文不正 |
| ERR_UNKNOWN_FIELD | 400 | Unknown field | 未知フィールド |
| ERR_EMPTY_BODY | 400 | Empty request body | 必須Body未指定 |
| ERR_UNSUPPORTED_CONTENT_TYPE | 400 | Unsupported Content-Type | Content-Type不正 |
| ERR_STORAGE_VALIDATION_FAILED | 500 | Storage validation failed | 実行時データ検証失敗 |
| ERR_JSON_SAVE_FAILED | 500 | JSON save failed | JSON保存失敗 |
| ERR_WEBHOOK_DUPLICATE | 200 | Webhook duplicate ignored | Webhook重複受信 |
| ERR_INTERNAL_SERVER_ERROR | 500 | Internal server error | サーバー内部エラー |

### 8.8 データバリデーション仕様

**Project**

| フィールド | 制約 | 説明 |
|-----------|------|------|
| name | 1～255字、英数字・ハイフン・アンダースコアのみ | プロジェクト名 |
| quota | 100MB～1TB、デフォルト1GB | ディスク容量上限 |

**File**

| フィールド | 制約 | 説明 |
|-----------|------|------|
| name | 1～255字、パス区切り文字なし | ファイル名 |
| size | 1B～1GB（プロジェクト quota による制限あり） | ファイルサイズ |

**Domain**

| フィールド | 制約 | 説明 |
|-----------|------|------|
| domain | RFC 1035 準拠（63字/ラベル、3階層まで） | ドメイン名 |
| isCustom | true/false | カスタムドメイン判定 |

---

## 9 データモデル

```json
{
  "id": "string (UUID)",
  "name": "string",
  "quota": "number (バイト)",
  "used": "number (バイト)",
  "domains": "string[]",
  "createdAt": "string (ISO 8601)"
}
```

**フィールド説明**
- `id`：プロジェクトの一意識別子（UUID）
- `name`：プロジェクト名
- `quota`：割り当てディスク容量（バイト）
- `used`：現在使用中のディスク容量（バイト）
- `domains`：割り当てられたドメイン配列
- `createdAt`：作成日時（ISO 8601形式）

### 9.2 File

```json
{
  "id": "string (UUID)",
  "projectId": "string",
  "name": "string",
  "size": "number",
  "path": "string",
  "uploadedAt": "string (ISO 8601)"
}
```

**フィールド説明**
- `id`：ファイルの一意識別子（UUID）
- `projectId`：所属プロジェクトID
- `name`：ファイル名
- `size`：ファイルサイズ（バイト）
- `path`：ストレージ内のファイルパス
- `uploadedAt`：アップロード日時（ISO 8601形式）

### 9.3 Domain

```json
{
  "domain": "string",
  "projectId": "string",
  "isCustom": "boolean",
  "sslCert": "string",
  "createdAt": "string (ISO 8601)"
}
```

**フィールド説明**
- `domain`：ドメイン名（例：my-blog.example.com）
- `projectId`：所属プロジェクトID
- `isCustom`：カスタムドメイン判定（true=独自ドメイン、false=ASB提供ドメイン）
- `sslCert`：SSL証明書ID
- `createdAt`：割り当て日時（ISO 8601形式）

### 9.4 Backup

```json
{
  "id": "string (UUID)",
  "projectId": "string",
  "createdAt": "string (ISO 8601)",
  "size": "number",
  "path": "string",
  "sha256": "string",
  "status": "string"
}
```

**フィールド説明**
- `id`：バックアップの一意識別子（UUID）
- `projectId`：バックアップ対象プロジェクトID
- `createdAt`：バックアップ作成日時（ISO 8601形式）
- `size`：バックアップサイズ（バイト）
- `path`：`storage.basePath` からの相対バックアップファイルパス
- `sha256`：バックアップtar.gzのSHA-256
- `status`：`completed` または `failed`

---

## 10 JSON ファイル構造

### 10.1 config/projects.json

```json
{
  "projects": [
    {
      "id": "proj-001",
      "name": "my-blog",
      "quota": 1073741824,
      "used": 536870912,
      "domains": ["my-blog.example.com"],
      "createdAt": "2025-01-15T10:00:00Z"
    }
  ]
}
```

### 10.2 config/backups.json

```json
{
  "backups": [
    {
      "id": "backup-001",
      "projectId": "proj-001",
      "createdAt": "2025-01-15T10:00:00Z",
      "size": 536870912,
      "path": "backups/backup-001.tar.gz",
      "sha256": "string",
      "status": "completed"
    }
  ]
}
```

### 10.3 config/config.json

```json
{
  "server": {
    "port": 3000,
    "host": "localhost",
    "shutdownTimeout": 30
  },
  "storage": {
    "basePath": "/var/asb",
    "maxProjectSize": 1073741824
  },
  "ssl": {
    "email": "admin@example.com",
    "renewBefore": 7776000
  },
  "log": {
    "level": "info",
    "format": "json",
    "maxSize": 104857600
  },
  "deploy": {
    "projectId": "",
    "sourcePath": "/srv/asb/source",
    "branch": "main"
  },
  "webhook": {
    "githubSecret": ""
  }
}
```

**フィールド説明**

| セクション | フィールド | 型 | デフォルト値 | 説明 |
|-----------|-----------|-----|------------|------|
| server | port | int | 3000 | バインドポート（0-65535） |
| server | host | string | localhost | バインドホスト |
| server | shutdownTimeout | int | 30 | グレースフルシャットダウン秒数 |
| storage | basePath | string | /var/asb | ストレージベースパス |
| storage | maxProjectSize | number | 1073741824 | プロジェクト最大容量（バイト、デフォルト1GB） |
| ssl | email | string | admin@example.com | ACME 等の証明書管理通知用メール |
| ssl | renewBefore | int | 7776000 | 更新タイミング（秒、デフォルト90日前） |
| log | level | string | info | ログレベル（debug/info/warn/error） |
| log | format | string | json | ログ形式（JSON Lines） |
| log | maxSize | number | 104857600 | ログファイル最大サイズ（バイト、デフォルト100MB） |
| deploy | projectId | string | 空文字 | Webhookデプロイ先Project ID。空文字の場合、Webhookデプロイは失敗扱い |
| deploy | sourcePath | string | /srv/asb/source | Webhookデプロイ元ローカルcheckout絶対パス |
| deploy | branch | string | main | Webhookデプロイ対象ブランチ |
| webhook | githubSecret | string | 空文字 | GitHub Webhook署名検証用secret。空文字の場合は署名検証を行わない |

### 10.4 config/domains.json

```json
{
  "domains": [
    {
      "domain": "my-blog.example.com",
      "projectId": "proj-001",
      "isCustom": true,
      "sslCert": "cert-001",
      "createdAt": "2025-01-15T10:00:00Z"
    }
  ]
}
```

### 10.5 storage/projects/:projectId/files.json

```json
{
  "files": [
    {
      "id": "file-001",
      "projectId": "proj-001",
      "name": "index.html",
      "size": 2048,
      "path": "/storage/projects/proj-001/index.html",
      "uploadedAt": "2025-01-15T10:00:00Z"
    }
  ]
}
```

### 10.6 config/webhooks.json

```json
{
  "events": [
    {
      "key": "main:abcdef1234567890",
      "branch": "main",
      "after": "abcdef1234567890",
      "status": "deployed",
      "receivedAt": "2026-09-08T00:00:00Z",
      "completedAt": "2026-09-08T00:00:05Z",
      "errorCode": ""
    }
  ]
}
```

**フィールド説明**
- `key`：Webhook 冪等キー（branch + ":" + after）
- `branch`：GitHub Push イベントの対象ブランチ
- `after`：GitHub Push イベントの after commit hash
- `status`：`deployed`、`failed`、`duplicate`、`ignored` のいずれか
- `receivedAt`：Webhook 受信日時（ISO 8601形式）
- `completedAt`：Webhook 処理完了日時（ISO 8601形式）
- `errorCode`：失敗時のエラーコード。成功時は空文字

### 10.7 config/migrations.json

```json
{
  "schemaVersion": 1,
  "migrations": [
    {
      "id": "string(UUID)",
      "fromSchema": 0,
      "toSchema": 1,
      "startedAt": "2026-09-08T00:00:00Z",
      "finishedAt": "2026-09-08T00:00:00Z",
      "status": "applied",
      "backupPath": "backups/migrations/migration-id.tar.gz"
    }
  ]
}
```

**フィールド説明**
- `schemaVersion`：マイグレーション履歴ファイルのスキーマバージョン
- `migrations`：実行済みまたは失敗したマイグレーション履歴
- `status`：`applied` または `failed`
- `backupPath`：`storage.basePath` からの相対バックアップパス

---

## 11 ポリシー

ASB の主要ポリシーは、以下の 10 種類で構成される。各ポリシーはドメイン別、機能別、運用別に管理され、仕様正本として機能する。

### 11.1 Management Domain ポリシー

Management Domain は、プロジェクト・ドメイン・SSL管理の責務を担う。

**責務優先順位**
1. プロジェクト整合性
2. ドメイン割り当て一貫性
3. SSL証明書有効性

プロジェクト削除時は、関連するドメイン・ファイル・バックアップの整合性を保証し、整合性を犯す操作を禁止する。

### 11.2 Delivery Domain ポリシー

Delivery Domain は、ファイル管理・GitHub Webhook の責務を担う。

**責務**
- ファイルアップロード：プロジェクト quota の範囲内で管理
- Webhook：指定ブランチの変更を検出し自動デプロイ
- エラー時の通知とログ記録

Webhook 処理失敗は、システムログおよび `config/webhooks.json` へ記録する。

Rev.32 時点では、Webhook失敗時の自動リトライスケジュールを実装しない。

### 11.3 Data Domain ポリシー

Data Domain は、バックアップ・ストレージの責務を担う。

**責務優先順位**
1. データ整合性
2. データ保全
3. 可用性

データ保全をData Domain の最優先ポリシーとする。バックアップはtar.gz形式で実施し、復元前に整合性検査（SHA-256ハッシュ検証）を実施する。

**バックアップ・復旧方針**
- バックアップ取得後はハッシュ検証を実施
- ASB標準バックアップ保存先は `storage.basePath/backups/` とする
- バックアップ保存先を別障害領域へ複製する作業は Rev.32 時点ではASB外の運用責務とする
- 外部ストレージ連携は Rev.32 時点では実装対象外とする
- 復旧は対象バックアップの存在、SHA-256、JSON構文、スキーマ検証後に実施
- バックアップ・復旧・検証失敗・復旧操作は監査ログへ記録

### 11.4 System Domain ポリシー

System Domain は、監視・ログ管理の責務を担う。

**構造化ログ**
- JSON形式で出力
- ログレベル：DEBUG / INFO / WARN / ERROR
- 本番デフォルトレベル：INFO
- 標準構成では `storage.basePath/logs/` 配下へ JSON Lines として保存する
- アクセスログは `storage.basePath/logs/access.log` に保存する
- エラーログは `storage.basePath/logs/error.log` に保存する
- 標準出力（stdout）への通常ログ出力は Rev.32 時点では実装しない
- 起動失敗時のみ標準エラー（stderr）へ単一行の起動エラーを出力する

**必須フィールド**
- `time`：UTC RFC3339 秒精度
- `level`：ログレベル
- `domain`：機能ドメイン（`management` / `delivery` / `data` / `system`）
- `message`：ログメッセージ
- `requestId`：HTTP リクエストに紐づく UUID。リクエスト外ログでは空文字を許可する

**監視項目**
- CPU使用率、メモリ使用率、ディスク使用率
- 接続数・リクエスト数
- バックアップ状態、証明書有効期限

### 11.5 API/SDK ポリシー

ASB はヘッドレスアーキテクチャを採用し、UI層に依存しない。

Rev.32 時点の確定対象は、ASB 本体が提供する HTTP JSON API、および将来 SDK が使用する通信規格である。

SDK 本体、SDK 配布方針、SDK 認証仕様は Rev.32 時点では実装対象外とする。

SDK 通信規格は、ASB 本体が提供する HTTP JSON API と同一とする。

**API 設計原則**
- HTTP + JSON を使用する
- HTTP/2 対応は ASB互換目標として扱い、Rev.32 時点では実装詳細を確定しない
- デフォルト接続境界は `localhost:3000` とする
- 管理 API は Rev.32 時点では認証なしとする
- SDK 通信は ASB 管理 API と同じ request / response / error / timestamp / pagination / upload 規約に従う
- SDK 専用プロトコル、SDK 専用エンドポイント、SDK 専用セッションを追加しない
- APIキー管理、ユーザー認証、SDK認証仕様は保留事項として扱う
- 本番公開時の管理 API 保護は、リバースプロキシ、ファイアウォール、VPN、localhost bind 等のASB外の運用境界で行う

**エラーレスポンス形式**
```json
{
  "error": "string (エラーメッセージ)",
  "code": "string (エラーコード)",
  "timestamp": "string (ISO 8601)",
  "httpStatus": 400
}
```

**API / スキーマ互換性**
- API互換性は仕様書RevとASB本体バージョンの組み合わせで判断する
- 実行時JSON互換性は `schemaVersion` で判断する
- `schemaVersion` 変更時は §16 マイグレーション戦略に従う

### 11.6 技術・依存ポリシー

- Go 1.21 以上を固定採用バージョンとする
- 外部ライブラリ禁止（原則）
- Go 標準ライブラリも最小限に使用
- 理由：外部依存を禁止、シンプル性・内製化を優先
- 例外：「例外外部ライブラリリスト」に記載されたライブラリのみ採用可能
- 例外採用時：最新の安定版リリースのみ使用
- CGO 要件は避ける

#### 11.6.1 例外外部ライブラリリスト

| ライブラリ | 用途 | 状態 |
|-----------|------|------|
| （現在：指定なし） | - | - |

注：追加の際は、必須性・メンテナンス・ライセンスを確認の上、仕様に追記

### 11.7 テスト・品質ポリシー

**テスト方針**

ユニットテスト
- 対象：各Service の単体テスト
- カバレッジ目標：80%以上
- ツール：Go testing パッケージ

統合テスト
- 対象：API エンドポイント単位（全エンドポイント）
- 検証項目：リクエスト/レスポンス仕様、トランザクション・ロールバック、データ整合性
- ツール：Go testing パッケージ + テスト用 HTTP クライアント

E2E テスト
- 対象：ワークフロー全体
- 検証シナリオ：プロジェクト作成 → ファイルアップロード → 削除、ドメイン割り当て → SSL管理データ確認、バックアップ → 復旧
- ツール：bash スクリプト + curl

**品質基準**
- バグゼロ化に向けたコード品質を維持
- 仕様と実装の不整合を許さない
- テスト失敗時は本番リリースを禁止

### 11.8 セキュリティポリシー

**アクセス制限**
- API バインドアドレス：localhost のみ（デフォルト）
- ASB 本体の標準 listen address は `127.0.0.1` とする
- ASB 本体は Rev.32 時点ではインターネット公開用 listen 設定を既定値として提供しない
- リモートアクセスは ASB 外部のリバースプロキシ、VPN、SSH tunnel、ファイアウォール等の運用境界で扱う
- ASB 本体は Rev.32 時点ではリバースプロキシ設定ファイルを生成しない
- SSL/TLS：本番環境では必須とするが、Rev.32 時点では ASB 本体では TLS 終端を実装しない
- TLS 終端は ASB 外部のリバースプロキシまたはロードバランサで扱う

**レート制限**
- Rev.32 時点では ASB 本体に実装しない
- Rate limiting は、本番公開時に ASB 外部のリバースプロキシ、WAF、CDN、ファイアウォール等で扱う

**タイムアウト**
- リクエスト読み込み：30秒
- レスポンス書き込み：60秒
- ログ保持時間：7日間

**ファイルアップロード**
- 最大サイズ：1GB
- 形式制限：なし
- ウイルススキャン：Rev.32 時点では ASB 本体に実装しない
- ASB 本体はアップロードファイルのマルウェア判定、隔離、駆除、外部スキャンAPI連携を行わない
- ウイルススキャン用の JSON、設定項目、隔離ディレクトリ、スキャンログを生成してはならない
- 置換許可：同名ファイル上書き可能

**ログ出力**
- アクセスログ：全HTTP リクエスト（JSON形式）
- エラーログ：エラー・例外・警告
- ログファイル暗号化：Rev.32 時点では ASB 本体に実装しない
- ログ暗号化用の鍵管理、鍵生成、鍵保存、暗号化ログ形式、復号API、外部KMS連携を追加してはならない

### 11.9 ライセンス・バージョンポリシー

#### 11.9.1 ライセンス方針

##### 11.9.1.1 現時点のライセンス

Adlaire-Static-Base（ASB）のライセンスは、現時点ではクローズドライセンスとする。

##### 11.9.1.2 クローズドライセンス方針

以下を禁止する。

- 無断複製
- 無断再配布
- 無断改変物の公開
- 無断商用利用
- 無断OSS化
- ライセンス未承認状態での第三者提供

以下を許可する。

- 学習・研究目的でのソースコード閲覧

##### 11.9.1.3 権利帰属

Adlaire-Static-Base（ASB）に含まれるソースコード、ドキュメント、設計、仕様、名称、関連成果物の権利は、正式な権利者またはプロジェクト管理主体に帰属する。

##### 11.9.1.4 ライセンス変更

クローズドライセンスから別ライセンスへ変更する場合は、安定版バージョンの切り出し候補とする。

ライセンス変更時は、ユーザー承認、関連ポリシーとの整合確認、本仕様書更新を必須とする。

#### 11.9.2 バージョン方針

本仕様書およびASB製品には、以下の2種類のバージョン管理を適用する。どちらも重大度によって桁や増分を使い分けない、単純な積み上げ式の累積連番を採用し、いかなる理由があってもリセット（巻き戻し・1からの数え直し）はしない。

##### 11.9.2.1 ドキュメントバージョン（Rev.N）

本仕様書ドキュメント自体のバージョン。

- 表記は`Rev.N`（Nは1から始まる連番）
- 本ファイルの変更を伴うすべての改訂に付与する
- 1エントリ＝1バージョンの原則で、§17 変更履歴の行数とNが一致する
- 桁揃え（ゼロ埋め）は行わない（Rev.1, Rev.2, … Rev.9, Rev.10, …）
- 表記箇所: 本ファイル冒頭「本書バージョン」欄（§2）が正本

##### 11.9.2.2 開発版バージョン（v0.N）

ASB製品（ソフトウェア）の開発版バージョン。

- 表記は`v0.N`（Nは1から始まる連番）。先頭の0は固定とする
- 本仕様書の変更を伴うすべての変更に付与する
- Nは変更のたびに1ずつ増加する。桁揃えは行わない（v0.1, v0.2, … v0.9, v0.10, …）
- いかなる理由があってもリセット（巻き戻し・1からの数え直し）はしない

##### 11.9.2.3 安定版バージョン（vX.Y）

リリース（外部への公開・配布）は安定版のみを対象とする。開発版バージョンの全エントリがリリースされるわけではなく、以下の判定基準をすべて満たした開発版を安定版として切り出す。

- 表記は`vX.Y`（例: v1.9, v3.20, v12.35）
- X = 安定版リリースの通し番号（1件目を1、2件目を2、…）。メジャー/マイナーのような重大度の意味は持たない
- Y = そのリリースを切り出した時点の開発版バージョンのN（例: 開発版v0.35を3件目の安定版として切り出した場合、v3.35）
- X・Yともにいかなる理由でもリセットしない
- 同一の安定版リリースの中でX・Yが指す時点がずれることはない（Yは常にそのリリース時点の開発版Nと一致する）
- 現時点ではまだ安定版リリースを1件も出していない

安定版切り出し判定基準は以下とする。

1. `ASB-spec.md`、`ASB-spec.html`、`IMPLEMENTATION_TASKS.md` の参照Revが一致している
2. `go test ./...` が成功している
3. `git diff --check` が成功している
4. Linux amd64 と Linux arm64 のビルドが成功している
5. リリース対象バイナリの `--version` 出力が安定版バージョンと一致している
6. `checksums.txt` に全リリース成果物のSHA-256が記録されている
7. `.gitignore` が存在しない
8. 開発リポジトリ内に実行時データ、ビルド成果物、一時ファイル、ログファイル、移行作業ファイルが残っていない
9. Pull Request 経由で `main` に反映済みである
10. 対象 commit hash をリリース記録へ残している

上記のいずれかを満たさない場合、安定版として配布してはならない。

##### 11.9.2.4 互換性対応表

**現行互換性対応表**

|ASB本体|Go|ファイルストレージ形式|
|---|---|---|
|v0.1以降|1.21|JSON ファイル形式|

- ASB本体バージョンとファイルストレージ形式の組み合わせ変更は仕様改訂を必要とする。
- Goバージョンの更新は§11.6の手順を経て本表を更新する。
- 互換性検証の実施記録はログへ保存する。

##### 11.9.2.5 実装フェーズバージョン

ASB の実装は、`IMPLEMENTATION_TASKS.md` に定義する実装フェーズ単位で進める。

実装フェーズは `P0` から始まる連番とし、各フェーズに開発版バージョン `v0.N` を1つ対応させる。

実装フェーズと開発版バージョンの対応は以下を固定値とする。

| フェーズ | 開発版バージョン | 実装単位 |
|---------|----------------|----------|
| P0 | v0.1 | 基盤 |
| P1 | v0.2 | API・JSON・起動検証 |
| P2 | v0.3 | 静的配信・ファイル管理 |
| P3 | v0.4 | ドメイン・SSL管理境界 |
| P4 | v0.5 | GitHub Webhook デプロイ |
| P5 | v0.6 | バックアップ・復旧 |
| P6 | v0.7 | ログ・監視 |
| P7 | v0.8 | マイグレーション |
| P8 | v0.9 | 配布・install/update |
| P9 | v0.10 | 禁止機能・非実装確認 |

各フェーズは、目的、実装タスク、完了条件、非対象範囲を持つ。

フェーズの完了は、当該フェーズの完了条件をすべて満たし、`go test ./...`、`git diff --check`、開発リポジトリ内生成物確認、`.gitignore` 不在確認が完了した状態とする。

後続フェーズは、原則として直前フェーズの完了後に着手する。

ただし、仕様整理、テスト設計、レビュー指摘対応は、該当フェーズの実装着手前に準備作業として行うことを許可する。

フェーズを追加、削除、分割、統合、順序変更、バージョン変更する場合は、事前に `ASB-spec.md` を改訂し、`IMPLEMENTATION_TASKS.md` を同期する。

### 11.10 将来計画管理ポリシー

将来計画は、Rev.32 時点の実装対象ではない。

将来計画に記載された項目は、実装、設定追加、API追加、JSON追加、ディレクトリ追加、外部依存追加、実行時データ生成の根拠として扱ってはならない。

Rev.32 時点で実装対象外とする将来計画は以下とする。

- GUI
- Web UI
- デスクトップアプリ
- モバイルアプリ
- クラウドサービス化
- 複数インスタンス対応
- NFS 連携
- 分散ストレージ連携
- ユーザー認証
- マルチテナント対応
- 外部ストレージ連携
- ログファイル暗号化
- ウイルススキャン
- HTTP/2 実装詳細

上記を実装対象へ昇格する場合は、実装前に `ASB-spec.md` を改訂し、実装対象範囲、非対象範囲、API、保存 JSON、設定項目、生成ファイル、生成ディレクトリ、外部依存、開発リポジトリ非生成の保証、テスト条件を確定する。

### 11.11 設定管理・インストール・アップデート

#### 11.11.1 設定管理ポリシー

- 設定は `config.json` で一元管理
- 起動時にバリデーションを実行
- 無効な設定は起動失敗とする
- デフォルト値を提供

#### 11.11.2 単一バイナリ形式の利点

ASB は単一バイナリとして提供される。この形式により以下の利点がある：

- 外部依存がない（スタンドアロン実行可能）
- SSH 経由でのリモートインストール・アップデートが容易
- VPS・レンタルサーバー環境での導入が簡単
- 権限設定のみで実行可能
- ローリングアップデートが可能

#### 11.11.3 配布成果物固定仕様

ASB の標準配布先は GitHub Releases とする。

リリースタグは安定版バージョンと同一文字列にする。

例：

```text
v1.19
```

標準配布成果物は以下に固定する。

| ファイル | 内容 | 必須 |
|---------|------|------|
| `asb-linux-amd64-vX.Y` | Linux amd64 向けASB本体バイナリ | 必須 |
| `asb-linux-arm64-vX.Y` | Linux arm64 向けASB本体バイナリ | 必須 |
| `checksums.txt` | SHA-256 checksum 一覧 | 必須 |

配布成果物をGit管理対象として開発リポジトリ内へ保存してはならない。

`checksums.txt` は以下の形式とする。

```text
<sha256>  asb-linux-amd64-vX.Y
<sha256>  asb-linux-arm64-vX.Y
```

`checksums.txt` に記載するファイル名は、GitHub Releases 上の配布ファイル名と完全一致させる。

#### 11.11.4 ビルド固定仕様

標準ビルドコマンドは以下に固定する。

```bash
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -trimpath -ldflags "-s -w" -o dist/asb-linux-amd64-vX.Y ./cmd/asb
CGO_ENABLED=0 GOOS=linux GOARCH=arm64 go build -trimpath -ldflags "-s -w" -o dist/asb-linux-arm64-vX.Y ./cmd/asb
```

`dist/` はリリース作業用の一時出力先であり、開発リポジトリへ残してはならない。

ビルド成果物は、GitHub Releases へアップロードした後に開発リポジトリから削除する。

#### 11.11.5 インストール手順

**前提**
- Linux amd64 または arm64 環境
- SSH アクセス可能
- GitHub Releases から対象バージョンの配布成果物を取得可能
- `sha256sum` または `shasum -a 256` を利用可能

**手順**

```bash
# 1. バイナリと checksums.txt をダウンロード
$ curl -fL -O https://github.com/fqwink/Adlaire-Static-Base/releases/download/vX.Y/asb-linux-amd64-vX.Y
$ curl -fL -O https://github.com/fqwink/Adlaire-Static-Base/releases/download/vX.Y/checksums.txt

# 2. checksum を検証
$ sha256sum -c checksums.txt --ignore-missing

# 3. 実行権限を付与
$ chmod +x asb-linux-amd64-vX.Y

# 4. 起動テスト
$ ./asb-linux-amd64-vX.Y --version

# 5. systemd で管理
$ sudo install -o root -g root -m 0755 asb-linux-amd64-vX.Y /usr/local/bin/asb
$ sudo systemctl enable asb
$ sudo systemctl start asb
```

`--version` の出力が対象安定版バージョンと一致しない場合、インストールしてはならない。

#### 11.11.6 アップデート手順

**既存バイナリの置き換え**

```bash
# 1. 新しいバイナリと checksums.txt をダウンロード
$ curl -fL -O https://github.com/fqwink/Adlaire-Static-Base/releases/download/vX.Y/asb-linux-amd64-vX.Y
$ curl -fL -O https://github.com/fqwink/Adlaire-Static-Base/releases/download/vX.Y/checksums.txt

# 2. checksum を検証
$ sha256sum -c checksums.txt --ignore-missing

# 3. バージョンを確認
$ ./asb-linux-amd64-vX.Y --version

# 4. 既存サービスを停止
$ sudo systemctl stop asb

# 5. 既存バイナリを退避して置き換え
$ sudo cp /usr/local/bin/asb /usr/local/bin/asb.previous
$ sudo install -o root -g root -m 0755 asb-linux-amd64-vX.Y /usr/local/bin/asb

# 6. 起動
$ sudo systemctl start asb

# 7. 起動状態を確認
$ sudo systemctl is-active --quiet asb
```

アップデート後に起動確認が失敗した場合、`/usr/local/bin/asb.previous` を `/usr/local/bin/asb` へ戻し、`systemctl start asb` を再実行する。

#### 11.11.7 install.sh / update.sh 固定仕様

ASB の初回インストールとアップデートを自動化するため、`install.sh` と `update.sh` を提供する。

両スクリプトはPOSIX sh互換で実装する。

**提供ファイル**

| ファイル | 用途 | 説明 |
|---------|------|------|
| install.sh | 初回インストール | 指定バージョンのダウンロード、checksum検証、権限設定、systemd登録を実行 |
| update.sh | アップデート | 指定バージョンのダウンロード、checksum検証、サービス停止、置換、再起動、失敗時復旧を実行 |
| asb.service | systemd ユニット | systemctl での自動起動・停止・再起動・ログ管理 |

**install.sh 引数**

```bash
./install.sh --version vX.Y --arch amd64
./install.sh --version vX.Y --arch arm64
```

| 引数 | 必須 | 内容 |
|------|------|------|
| `--version` | 必須 | インストール対象の安定版バージョン |
| `--arch` | 任意 | `amd64` または `arm64`。未指定時は `uname -m` から判定 |

**update.sh 引数**

```bash
./update.sh --version vX.Y --arch amd64
./update.sh --version vX.Y --arch arm64
```

| 引数 | 必須 | 内容 |
|------|------|------|
| `--version` | 必須 | 更新対象の安定版バージョン |
| `--arch` | 任意 | `amd64` または `arm64`。未指定時は `uname -m` から判定 |

`latest` 指定、自動最新版選択、未指定バージョンでの実行は Rev.32 時点では禁止する。

スクリプトは以下を満たす。

1. `set -eu` で実行する
2. 一時作業ディレクトリは `mktemp -d` で作成する
3. 一時作業ディレクトリは終了時に削除する
4. 開発リポジトリ内にビルド成果物、一時ファイル、ログファイルを作成しない
5. ダウンロード失敗時は終了コード `1` で失敗する
6. checksum不一致時は終了コード `1` で失敗する
7. `--version` 出力不一致時は終了コード `1` で失敗する
8. systemd 操作失敗時は終了コード `1` で失敗する
9. `update.sh` は起動失敗時に `/usr/local/bin/asb.previous` から復旧を試行する
10. 復旧に失敗した場合も成功扱いしてはならない

#### 11.11.8 asb.service 固定仕様

```ini
[Unit]
Description=Adlaire-Static-Base HTTP Server
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=asb
Group=asb
ExecStart=/usr/local/bin/asb --config /etc/asb/config.json
Restart=on-failure
RestartSec=5
StandardOutput=journal
StandardError=journal
SyslogIdentifier=asb
NoNewPrivileges=true

[Install]
WantedBy=multi-user.target
```

`asb.service` は `/etc/systemd/system/asb.service` へ配置する。

`asb` ユーザーおよび `asb` グループが存在しない場合、`install.sh` は system user として作成する。

**systemctl での管理**

```bash
# サービス状態確認
$ sudo systemctl status asb

# サービス再起動
$ sudo systemctl restart asb

# ログ表示
$ sudo journalctl -u asb -f

# 自動起動設定
$ sudo systemctl enable asb

# 手動停止
$ sudo systemctl stop asb
```

---
## 12 将来計画

### 12.1 段階的対応対象

本章は将来計画の記録であり、Rev.32 時点の実装対象を増やすものではない。

以下は Rev.32 時点では実装対象外とする。

| 対象 | Rev.32 時点の扱い | 実装禁止範囲 |
|-----|------------------|------------|
| GUI | 実装対象外 | UI サーバー、画面、画面用API、画面用設定 |
| Web UI | 実装対象外 | SPA、SSR、管理画面、フロントエンドビルド |
| デスクトップアプリ | 実装対象外 | Electron等の外部フレームワーク、アプリ配布物 |
| モバイルアプリ | 実装対象外 | iOS/Android アプリ、モバイル専用API |
| クラウドサービス化 | 実装対象外 | SaaS基盤、テナント管理、課金、アカウント管理 |
| 複数インスタンス対応 | 実装対象外 | 分散ロック、クラスタ管理、ノード管理 |
| NFS 連携 | 実装対象外 | NFS 専用設定、NFS 固有のロック処理 |
| 分散ストレージ連携 | 実装対象外 | 分散ストレージAPI、外部ストレージSDK |

### 12.1.1 Phase 別詳細

**Phase 2：GUI 実装**
- Rev.32 時点では実装しない
- GUI 用の API、設定項目、JSON ファイル、ディレクトリを追加しない
- Electron、React、Vue、Svelte 等の外部フレームワークを追加しない

**Phase 3：モバイルアプリ**
- Rev.32 時点では実装しない
- モバイル専用 API、認証、セッション、push通知、配布設定を追加しない

**Phase 4：クラウドサービス化**
- Rev.32 時点では実装しない
- マルチテナント、認証、認可、ユーザー管理、課金、契約、アカウント管理を追加しない

### 12.2 保留事項

以下は Rev.32 時点では実装対象外とする。

- ユーザー認証
- マルチテナント対応
- API キー管理
- SDK 本体
- SDK 配布
- SDK 認証
- Rate limiting
- 外部ストレージサービス統合
- ログファイル暗号化
- ウイルススキャン

上記を理由に、API、設定項目、保存 JSON、ディレクトリ、外部依存、実行時データを追加してはならない。

### 12.3 検討・調査中事項

Rev.32 時点では、検討・調査中事項を実装へ反映してはならない。

以下は調査対象としてのみ記録し、実装対象外とする。

- 複数インスタンス時の NFS / 分散ストレージ選定
- ACME 実通信
- CA選定
- ワイルドカード証明書対応
- 証明書自動更新
- バックアップのクラウドストレージ連携

調査結果を実装対象へ昇格する場合は、`ASB-spec.md` を先に改訂し、最低限以下を確定する。

- 実装対象範囲
- 非対象範囲
- API
- request / response
- error code
- 保存 JSON
- 設定項目
- 生成ファイル
- 生成ディレクトリ
- 外部依存の有無
- 開発リポジトリ非生成の保証
- テスト条件

---

## 13 実装詳細仕様

### 13.1 実装対象の基準

Rev.32 時点の実装対象は、ASB のセルフホスト型静的コンテンツ配信ホスティングに必要なバックエンド機能に限定する。

実装は以下の順序で進める：

1. 起動設定、実行時データ検証、HTTP サーバー、共通レスポンス
2. プロジェクト管理
3. ファイル管理
4. ドメイン管理
5. ログ・監視
6. バックアップ・復旧
7. GitHub Webhook
8. SSL 証明書管理

ASB互換目標は機能目標として扱い、ASB 独自仕様は実装制約として扱う。

ASB互換目標は、実装優先順位、外部依存追加、実行時データ生成、API追加、設定項目追加の根拠として扱ってはならない。

ASB互換目標に含まれる未確定機能は、個別の確定仕様へ昇格するまで実装対象外とする。

保留事項または検討中事項は、仕様上の扱いが確定するまで実装してはならない。

### 13.2 共通実装規約

**HTTP**

- API のレスポンスは `application/json; charset=utf-8` とする
- JSON リクエストは `Content-Type: application/json` を要求する
- ファイルアップロード API のみ `multipart/form-data` を許可する
- 未定義の API パスは `404 Not Found` を返す
- 未対応の HTTP メソッドは `405 Method Not Allowed` を返す
- サーバーは graceful shutdown に対応する

**JSON**

- JSON の未知フィールドは拒否する
- 空リクエストが許可されない API では空 Body を `400 Bad Request` とする
- 日時は UTC の RFC 3339 形式で保存・返却する
- JSON ファイルは UTF-8 で保存する

**ID**

- Project、File、Backup の `id` は UUID 形式の文字列とする
- UUID 生成には Go 標準ライブラリの `crypto/rand` を使用する
- Domain は `domain` 文字列を一意キーとして扱う

**エラー**

- エラーレスポンスには `error`、`code`、`timestamp`、`httpStatus` を含める
- 内部ファイルパス、スタックトレース、機密値をレスポンスへ含めてはならない
- 仕様に定義済みのエラーコードを優先して使用する

### 13.3 起動時検証

ASB は起動時に実行時データを自動生成しない。起動時には以下を検証する：

- 設定ファイルが存在する
- `storage.basePath` が存在する
- `storage.basePath` 配下に `config/`、`storage/`、`logs/`、`certs/` が存在する
- 必要な JSON ファイルが存在し、JSON として読み込める
- 書き込みが必要な領域に書き込み権限がある
- `server.port`、`server.host`、`shutdownTimeout`、`storage.maxProjectSize`、`ssl.renewBefore`、`log.level` が妥当である

検証に失敗した場合、ASB は HTTP サーバーを開始せず、エラーを標準エラーへ出力して終了する。

### 13.4 実行時データ配置

実行時データは `storage.basePath` 配下に配置する。

標準構成は以下とする：

```text
/var/asb/
├── config/
│   ├── config.json
│   ├── projects.json
│   ├── domains.json
│   ├── backups.json
│   └── webhooks.json
├── storage/
│   └── projects/
│       └── {projectId}/
│           ├── files.json
│           └── contents/
├── logs/
│   ├── access.log
│   └── error.log
└── certs/
```

開発リポジトリは仕様書、ドキュメント、ソースコード、テストコード、運用スクリプトを管理する領域であり、ASB の実行時データ保存先として扱わない。

### 13.5 JSON ファイル更新方針

JSON ファイル更新は以下の方針で行う：

- 更新前に対象 JSON を読み込み、構文と必須フィールドを検証する
- 更新後に保存予定データをメモリ上で再検証する
- 同一 JSON ファイルへの同時書き込みを排他する
- 書き込み失敗時は API エラーとして扱い、成功レスポンスを返してはならない
- プロジェクト削除など複数ファイルにまたがる操作では、失敗時に整合性を検証し、エラーをログへ記録する

### 13.6 プロジェクト管理詳細

**作成**

- `name` は 1〜255 文字とする
- `name` は英数字、ハイフン、アンダースコアのみ許可する
- `quota` が未指定の場合は `storage.maxProjectSize` を使用する
- `quota` は 100MB 以上 1TB 以下とする
- 同名プロジェクトが存在する場合は `ERR_PROJECT_ALREADY_EXISTS` を返す
- 作成時の `used` は `0` とする

**削除**

- 対象プロジェクトが存在しない場合は `ERR_PROJECT_NOT_FOUND` を返す
- 削除対象には、プロジェクト情報、関連ドメイン、関連ファイルメタデータ、関連バックアップ履歴を含める
- 削除完了後に `deletedAt` を含むレスポンスを返す

**一覧**

- `GET /api/projects` は `createdAt` 昇順で返す
- レスポンスには `projects` 配列を含める

### 13.7 ファイル管理詳細

**アップロード**

- 対象プロジェクトが存在しない場合は `ERR_PROJECT_NOT_FOUND` を返す
- multipart のフィールド名は `file` とする
- ファイル名は 1〜255 文字とし、パス区切り文字を含めてはならない
- ファイルサイズは 1B 以上 1GB 以下とする
- アップロード後の使用量が `quota` を超える場合は `ERR_QUOTA_EXCEEDED` を返す
- 同名ファイルは上書き可能とする
- 上書き時は旧ファイルサイズを差し引いた上で `used` を再計算する
- 成功時は File モデルを返す

**一覧**

- `GET /api/projects/:id/files` はファイル名昇順で返す
- レスポンスには `files` 配列を含める

**削除**

- 対象ファイルが存在しない場合は `ERR_FILE_NOT_FOUND` を返す
- 削除後はプロジェクトの `used` を再計算する
- 削除成功時は `status`、`projectId`、`fileName`、`deletedAt` を返す

**圧縮**

- Gzip 圧縮を実装対象とする
- Brotli 圧縮は Rev.32 時点では ASB 本体に実装しない
- `Accept-Encoding: br` を受信しても Brotli 応答へ切り替えない

### 13.8 ドメイン管理詳細

- ドメインは小文字へ正規化して保存する
- ドメインは RFC 1035 に準拠し、各ラベルは 1〜63 文字とする
- ドメイン全体は 253 文字以下とする
- ドメインは最大3階層までとする
- 既に別プロジェクトへ割り当て済みの場合は `ERR_DOMAIN_ALREADY_ASSIGNED` を返す
- プロジェクト削除時は関連ドメインを削除する

### 13.9 SSL 管理詳細

Rev.32 時点では、SSL 管理は証明書メタデータ、証明書配置、証明書検証に限定する。

ACME は ASB互換目標に含めるが、ACME 実通信、CA選定、ワイルドカード証明書対応、証明書自動更新は Rev.32 時点では ASB 本体に実装しない。

実装対象：

- SSL 証明書IDの管理
- 証明書保存先 `certs/` の存在確認
- 証明書ファイルパスの検証
- 証明書有効期限の監視モデル
- 証明書メタデータの JSON 保存
- SSL 証明書生成失敗時の `ERR_SSL_CERT_GENERATION_FAILED`

実装保留：

- ACME client
- ACME アカウント登録
- HTTP-01 / DNS-01 challenge
- TLS-ALPN-01 challenge
- CA 連携
- 証明書発行リクエスト
- 証明書自動更新の実通信
- challenge 状態管理
- ACME account key 管理
- ACME order / authorization / nonce 管理

### 13.10 Webhook 詳細

GitHub Webhook は Push イベントのみを対象とする。

- `POST /api/webhook/github` は GitHub Webhook ペイロードを受け取る
- 対象ブランチは `deploy.branch` で指定する
- 対象外ブランチのイベントは成功扱いで無視する
- Webhook 署名検証は `webhook.githubSecret` が空文字でない場合のみ実行する
- `webhook.githubSecret` が空文字でない場合、`X-Hub-Signature-256` ヘッダーを必須とする
- 署名検証は Go 標準ライブラリの `crypto/hmac` と `crypto/sha256` で行う
- 署名値比較は `hmac.Equal` で行う
- GitHub Push payload の `repository.clone_url` および `repository.ssh_url` をデプロイ元として使用してはならない
- デプロイ先Projectは `deploy.projectId` で指定する
- `deploy.projectId` が空文字の場合は `ERR_WEBHOOK_PROJECT_NOT_CONFIGURED` を返す
- デプロイ元は `deploy.sourcePath` に指定されたローカルcheckoutに限定する
- Webhook受信時は `deploy.sourcePath` が存在し、Git worktreeであり、対象 `after` commit を参照可能であることを検証する
- ペイロード形式が不正な場合は `400 Bad Request` を返す
- デプロイ処理に失敗した場合は `ERR_WEBHOOK_PROCESSING_FAILED` を返す
- 同一 GitHub Push イベントを重複受信した場合は、同一 commit hash と対象ブランチの組み合わせを冪等キーとして扱い、二重デプロイを避ける
- 冪等キーの保存方式は JSON ファイルベースとし、保存先は `storage.basePath` 配下に限定する
- Webhook失敗時の自動リトライは Rev.32 時点では実装しない
- GitHub側からの再送は通常のWebhook受信として扱い、冪等キーで重複判定する

### 13.11 バックアップ・復旧詳細

**バックアップ**

- バックアップ対象は対象 Project の `storage/projects/:projectId/files.json` と `storage/projects/:projectId/contents/` とする
- バックアップ形式は tar.gz とする
- バックアップ保存先は `storage.basePath/backups/` とする
- バックアップファイル名は `backup-{backupId}.tar.gz` とする
- バックアップパスは `config/backups.json` に `storage.basePath` からの相対パスとして保存する
- バックアップ作成後に SHA-256 ハッシュを計算する
- バックアップ履歴は `config/backups.json` に記録する
- バックアップ成功時の `status` は `completed` とする
- バックアップ失敗時は `status: "completed"` の履歴を作成してはならない
- バックアップ作成用の一時tarは `storage.basePath/backups/.tmp/` 配下にのみ作成できる
- 開発リポジトリ内にバックアップtar、一時tar、checksum、退避データを作成してはならない

**復旧**

- 復旧前にバックアップファイルの存在と SHA-256 ハッシュを検証する
- 復旧前退避先は `storage.basePath/backups/restore-staging/{restoreId}/previous/` とする
- 復旧用展開先は `storage.basePath/backups/restore-staging/{restoreId}/next/` とする
- 既存データを退避してから復旧する
- 復旧後に JSON ファイル構文と必須フィールドを検証する
- 復旧失敗時は `ERR_BACKUP_RESTORE_FAILED` を返す
- 復旧失敗時は可能な限り退避済みの `previous/` から復元する
- 退避領域からの復元に失敗した場合も成功扱いしてはならない
- 復旧完了後の `restore-staging/{restoreId}/` 削除は best effort とし、削除失敗時は WARN ログへ記録する

### 13.12 監視・ログ詳細

**監視**

- `/api/monitoring/stats` は CPU、メモリ、ディスク、接続数、リクエスト数を返す
- OS 依存で取得できない項目は `null` とし、レスポンス自体は成功扱いとする

**ログ**

- ログ保存先は `storage.basePath/logs/` とする
- アクセスログは `storage.basePath/logs/access.log` とする
- エラーログは `storage.basePath/logs/error.log` とする
- アクセスログは API、静的配信、Webhook を含む全 HTTP リクエストを対象とする
- エラーログは WARN 以上を対象とする
- ログ形式は UTF-8 JSON Lines とし、1行に1 JSON object のみを出力する
- 各ログ行は末尾に LF を付ける
- ログ API は `limit` と `offset` を受け付ける
- `limit` のデフォルトは 100、最小は 1、最大は 1000 とする
- `offset` のデフォルトは 0、最小は 0 とする
- ログ API は新しいログから順に返す
- ログ API は壊れた JSON 行を検出した場合、当該行を読み飛ばさず `500 Internal Server Error` を返す
- `requestId` はリクエスト受信時に生成し、レスポンスヘッダー `X-Request-Id`、アクセスログ、エラーログに同一値を使用する
- ログには機密値を含めてはならない
- 通常ログを stdout に出力してはならない
- 起動失敗時のみ stderr へ単一行を出力する
- ログローテーションは `log.maxSize` 到達時に実行する
- ローテーション後ファイル名は `access.log.{unixTime}` または `error.log.{unixTime}` とする
- ローテーション済みログは7日経過後に削除対象とする
- ログローテーションおよび期限切れ削除の失敗は WARN として `error.log` へ記録する
- 開発リポジトリ内にログファイルを作成してはならない

### 13.13 テスト受け入れ条件

実装完了判定には以下を必須とする：

- `go test ./...` が成功する
- Project API の作成、一覧、削除テストが成功する
- File API のアップロード、一覧、上書き、削除、quota 超過テストが成功する
- JSON ファイル読み込み失敗時の起動失敗テストが成功する
- 未定義ルート 404、メソッド不一致 405 のテストが成功する
- 共通エラーレスポンス形式のテストが成功する
- 仕様上保留の機能が実装されていないことを確認する

### 13.14 実装契約

本節は Rev.32 時点の実装契約である。実装者は本節に反する判断をコード側で独自に行ってはならない。

#### 13.14.1 パッケージ境界

実装は以下の Go package 境界を基本とする。

| パッケージ | 責務 | 外部副作用 |
|-----------|------|-----------|
| `config` | 設定読み込み、デフォルト適用、起動時検証 | 設定ファイル読み込み |
| `server` | `net/http` サーバー、ルーティング、共通レスポンス | HTTP 入出力 |
| `management` | Project、Domain、SSL 管理 | JSON メタデータ更新 |
| `delivery` | File、静的配信、Webhook | ファイル入出力、JSON メタデータ更新 |
| `data` | Backup、Storage | tar.gz 作成、復旧、JSON メタデータ更新 |
| `system` | Monitoring、Log | ログ読み書き、監視値取得 |

`cmd/asb/main.go` は設定読み込み、依存関係生成、HTTP サーバー起動、graceful shutdown のみを行う。

各 package は他 package の具象型に直接依存せず、必要な境界は interface で受け渡す。

#### 13.14.2 HTTP ルーティング契約

HTTP ルーティングは Go 標準 `net/http` で実装する。

外部ルーターライブラリは採用しない。

API パスは `/api/` で始まる。

API パス判定は静的ファイル配信より優先する。

静的ファイル配信は API パスに一致しないリクエストのみを対象とする。

API パスは `strings.TrimPrefix` と `/` 分割により解析し、空セグメント、余分な末尾スラッシュ、想定外セグメントは `404 Not Found` とする。

定義済みパスで HTTP メソッドだけが不一致の場合は `405 Method Not Allowed` とし、`Allow` ヘッダーに許可メソッドを設定する。

#### 13.14.3 HTTP リクエスト契約

JSON API は `Content-Type: application/json` を要求する。

`Content-Type` に charset が付く場合は許可する。

Body を持たない API では Body を読み込まない。

Body を要求する API で空 Body の場合は `400 Bad Request` とする。

JSON decode は `json.Decoder` を使用し、`DisallowUnknownFields` を有効にする。

1リクエストにつき JSON 値は1個のみ許可し、後続トークンが存在する場合は `400 Bad Request` とする。

multipart upload は `POST /api/projects/:id/files/upload` のみ許可し、フィールド名は `file` に固定する。

#### 13.14.4 HTTP レスポンス契約

成功レスポンスとエラーレスポンスは常に `Content-Type: application/json; charset=utf-8` とする。

エラーレスポンスは以下のフィールドを必須とする。

```json
{
  "error": "string",
  "code": "string",
  "timestamp": "2026-09-08T00:00:00Z",
  "httpStatus": 400
}
```

`timestamp` は UTC の RFC3339 形式とする。

エラーレスポンスに内部ファイルパス、スタックトレース、環境変数、機密値を含めてはならない。

#### 13.14.5 JSON 保存契約

実行時データは `storage.basePath` 配下にのみ保存する。

開発リポジトリ配下を `storage.basePath` に指定してはならない。

ASB は起動時に実行時データ用ディレクトリまたは JSON ファイルを自動生成しない。

JSON ファイル更新は以下の順序で行う。

1. 対象 JSON ファイルを読み込む
2. JSON 構文と必須フィールドを検証する
3. メモリ上で変更後データを作成する
4. 保存前に変更後データを再検証する
5. 同一 JSON ファイル単位の排他を取得する
6. 対象ファイルを truncate せず、同一ディレクトリ内の一時ファイルへ書き込む
7. `fsync` 後に atomic rename で置き換える

一時ファイル名は実行時データ領域内に限り使用できる。開発リポジトリ内へ一時ファイルを作成してはならない。

保存失敗時は成功レスポンスを返してはならない。

#### 13.14.6 起動時検証契約

起動時検証は HTTP サーバー起動前に完了する。

以下のいずれかに該当する場合、ASB は起動失敗とする。

- 設定ファイルが存在しない
- 設定ファイルが JSON として不正
- 未知フィールドが存在する
- `storage.basePath` が存在しない
- `storage.basePath` が開発リポジトリ配下を指す
- `config/`、`storage/`、`logs/`、`certs/` のいずれかが存在しない
- 必須 JSON ファイルが存在しない
- 必須 JSON ファイルが JSON として不正
- 読み込みまたは書き込み権限が不足する
- 設定値が許容範囲外である

起動失敗時は標準エラーへ単一行のエラーを出力し、終了コード `1` で終了する。

#### 13.14.7 Handler / Service / Entity 契約

Handler は HTTP 入出力、パス/クエリ/Body の取り出し、HTTP ステータス決定のみを担当する。

Handler は JSON ファイルを直接読み書きしてはならない。

Service はバリデーション、整合性確認、永続化操作、ドメインルール適用を担当する。

Entity は JSON 保存形式と API レスポンス形式に使う構造体を定義する。

Entity はファイル入出力、HTTP 入出力、時刻取得、ID生成を行ってはならない。

ID 生成、時刻取得、保存処理は Service に注入された依存関係を経由して行う。

#### 13.14.8 機能別副作用契約

| 機能 | 読み込み | 書き込み | 削除 |
|-----|---------|---------|------|
| Project 作成 | `config/projects.json` | `config/projects.json` | なし |
| Project 削除 | projects/domains/files/backups | 関連 JSON | 関連メタデータとファイル |
| File upload | project/files JSON | contents、files JSON、projects JSON | 上書き時の旧ファイル |
| Domain 追加 | projects/domains JSON | `config/domains.json` | なし |
| SSL 管理 | domains/certs | SSL メタデータ | なし |
| Webhook | config/projects/files | contents、files JSON、deploy log | 置換対象ファイル |
| Backup 作成 | 対象Project files/contents | backup tar.gz、backups JSON | なし |
| Backup 復旧 | backup tar.gz、対象Project files/contents | 対象Project files/contents | 復旧対象の既存データ退避 |
| Log API | logs | なし | なし |

複数ファイルにまたがる操作では、途中失敗時に成功レスポンスを返してはならない。

途中失敗時はエラーログを記録し、可能な範囲で整合性検証を行う。

#### 13.14.9 保留機能の実装禁止契約

Rev.32 時点では以下を実装してはならない。

- SDK
- SDK 専用プロトコル
- SDK 専用エンドポイント
- SDK 専用セッション
- WebSocket
- gRPC
- GraphQL
- 独自 TCP プロトコル
- MQTT
- APIキー管理
- ユーザー認証
- 認証 middleware
- `Authorization` ヘッダー検証
- `X-API-Key` ヘッダー検証
- APIキー保存ファイル
- Rate limiting
- Rate limiting middleware
- IP別リクエスト制限
- token bucket / leaky bucket 実装
- Rate limiting 用永続カウンタ
- Rate limiting 用設定項目
- Brotli 圧縮
- Brotli 用外部ライブラリ
- `.br` ファイル自動生成
- Brotli 用キャッシュ生成
- ACME 実通信
- ACME client
- ACME アカウント登録
- DNS-01 challenge
- HTTP-01 challenge
- TLS-ALPN-01 challenge
- CA 連携
- 証明書自動更新
- challenge 状態管理
- ACME account key 管理
- ACME order / authorization / nonce 管理
- CA 選定固定
- `webhook.githubSecret` 未設定時のWebhook署名検証必須化
- Webhook失敗時の自動リトライスケジューラー
- 外部DB
- 外部ストレージ連携
- `.gitignore` を必要とする生成物設計

上記を実装する場合は、先に `ASB-spec.md` を改訂し、確定仕様として昇格させる。

### 13.15 実装詳細固定仕様

本節は Rev.32 時点で実装時に固定する詳細仕様である。

#### 13.15.1 API エンドポイント固定表

| API | Method | Path | Body | Query | 成功 | 主な失敗 |
|-----|--------|------|------|-------|------|----------|
| Project 作成 | POST | `/api/projects` | `{"name":string,"quota":number?}` | なし | 201 Project | 400, 409, 500 |
| Project 一覧 | GET | `/api/projects` | なし | なし | 200 `{projects:[]}` | 500 |
| Project 削除 | DELETE | `/api/projects/:id` | なし | なし | 200 `{status,projectId,deletedAt}` | 404, 500 |
| Domain 追加 | POST | `/api/projects/:id/domains` | `{"domain":string}` | なし | 201 Domain | 400, 404, 409, 500 |
| Domain 一覧 | GET | `/api/projects/:id/domains` | なし | なし | 200 `{domains:[]}` | 404, 500 |
| Domain 削除 | DELETE | `/api/projects/:id/domains/:domain` | なし | なし | 200 `{status,projectId,domain,deletedAt}` | 404, 500 |
| File upload | POST | `/api/projects/:id/files/upload` | multipart `file` | なし | 201 File | 400, 404, 413, 500 |
| File 一覧 | GET | `/api/projects/:id/files` | なし | なし | 200 `{files:[]}` | 404, 500 |
| File 削除 | DELETE | `/api/projects/:id/files/:name` | なし | なし | 200 `{status,projectId,fileName,deletedAt}` | 404, 500 |
| Backup 一覧 | GET | `/api/backups` | なし | なし | 200 `{backups:[]}` | 500 |
| Backup 復旧 | POST | `/api/backups/restore/:id` | なし | なし | 200 `{status,backupId,restoredAt}` | 404, 409, 500 |
| Monitoring | GET | `/api/monitoring/stats` | なし | なし | 200 Monitoring | 500 |
| Access log | GET | `/api/logs/access` | なし | `limit`,`offset` | 200 `{logs:[]}` | 400, 500 |
| Error log | GET | `/api/logs/error` | なし | `limit`,`offset` | 200 `{logs:[]}` | 400, 500 |
| GitHub Webhook | POST | `/api/webhook/github` | GitHub Push JSON | なし | 200 `{status}` | 400, 500 |

`:id` は UUID 形式の文字列のみ許可する。

`:domain` は URL decode 後に小文字正規化し、ドメイン検証を行う。

`:name` は URL decode 後にファイル名検証を行い、パス区切り文字を含む値は拒否する。

`limit` は未指定時 `100`、最小 `1`、最大 `1000` とする。

`offset` は未指定時 `0`、最小 `0` とする。

`limit` または `offset` が整数として解釈できない場合は `400 Bad Request` とする。

`limit` が `1` 未満または `1000` を超える場合は `400 Bad Request` とする。

`offset` が `0` 未満の場合は `400 Bad Request` とする。

#### 13.15.2 設定値固定表

| 設定 | 必須 | デフォルト | 許容範囲 | 起動失敗条件 |
|------|------|------------|----------|--------------|
| `server.port` | 任意 | `3000` | `1`-`65535` | 範囲外、数値以外 |
| `server.host` | 任意 | `localhost` | `localhost`、`127.0.0.1`、`0.0.0.0` | 空文字、許可外 |
| `server.shutdownTimeout` | 任意 | `30` | `1`-`300` 秒 | 範囲外、数値以外 |
| `storage.basePath` | 任意 | `/var/asb` | 絶対パス | 相対パス、開発リポジトリ配下 |
| `storage.maxProjectSize` | 任意 | `1073741824` | `104857600`-`1099511627776` | 範囲外、数値以外 |
| `ssl.email` | 任意 | `admin@example.com` | email 形式 | 形式不正 |
| `ssl.renewBefore` | 任意 | `7776000` | `86400`-`15552000` 秒 | 範囲外、数値以外 |
| `log.level` | 任意 | `info` | `debug`,`info`,`warn`,`error` | 許可外 |
| `log.format` | 任意 | `json` | `json` | `json` 以外 |
| `log.maxSize` | 任意 | `104857600` | `1048576`-`1073741824` | 範囲外、数値以外 |
| `deploy.projectId` | 任意 | 空文字 | UUID形式または空文字 | UUID形式以外 |
| `deploy.sourcePath` | 任意 | `/srv/asb/source` | 絶対パス | 相対パス、存在しない、Git worktreeでない |
| `deploy.branch` | 任意 | `main` | Git ref name | 空文字、空白文字を含む、`..` を含む |
| `webhook.githubSecret` | 任意 | 空文字 | 文字列 | 文字列以外 |

設定ファイルに未知フィールドがある場合は起動失敗とする。

任意項目が未指定の場合はデフォルト値を適用する。

#### 13.15.3 JSON ファイル固定仕様

| ファイル | 空状態 | 必須トップレベル | 更新主体 | 備考 |
|---------|--------|------------------|----------|------|
| `config/projects.json` | `{"projects":[]}` | `projects` | Project Service | Project 配列を `createdAt` 昇順で保存 |
| `config/domains.json` | `{"domains":[]}` | `domains` | Domain Service | `domain` は小文字で保存 |
| `config/backups.json` | `{"backups":[]}` | `backups` | Backup Service | `createdAt` 降順で保存。`path` は `storage.basePath` からの相対パス |
| `storage/projects/:projectId/files.json` | `{"files":[]}` | `files` | File Service | `name` 昇順で保存 |
| `config/webhooks.json` | `{"events":[]}` | `events` | Webhook Service | 冪等キー、処理状態、失敗コードを保存 |

上記 JSON ファイルは起動時に存在していなければならない。

ASB は上記 JSON ファイルを起動時に作成しない。

`config/webhooks.json` は Webhook 冪等性管理に使用する実行時 JSON ファイルであり、外部DBを使用しない。

#### 13.15.4 静的配信固定仕様

静的配信は API パスに一致しない HTTP リクエストのうち、`GET` または `HEAD` のみ対象とする。

API パスとは `/api/` で始まるパスを指す。

`GET` または `HEAD` 以外のメソッドで静的配信対象パスへアクセスした場合は `405 Method Not Allowed` とし、`Allow: GET, HEAD` を返す。

対象 Project は以下の順序で決定する。

1. `Host` ヘッダーを取得する
2. `Host` の port 部分を除去する
3. 末尾の `.` を除去する
4. 小文字へ正規化する
5. 空文字、空白文字、`/`、`\` を含む Host は不正として扱う
6. `config/domains.json` の `domain` と完全一致する Domain を検索する
7. Domain に紐づく `projectId` を配信対象 Project とする

`Host` ヘッダーが存在しない、不正、または割り当て済み Domain に一致しない場合は `404 Not Found` とする。

Domain が存在しても対象 Project が存在しない場合は整合性エラーとして `500 Internal Server Error` とし、`ERR_STORAGE_VALIDATION_FAILED` をエラーログへ記録する。

リクエストパスは以下の順序で正規化する。

1. query string と fragment を除外し、URL path のみを対象とする
2. percent encoding を URL decode する
3. decode に失敗した場合は `404 Not Found` とする
4. `/` の連続は単一 `/` として扱う
5. 先頭 `/` を除去し、相対パスへ変換する
6. 空パスまたは `/` は `index.html` として扱う
7. パスが `/` で終わる場合は末尾に `index.html` を補う
8. `path.Clean` 相当の正規化後も配信ルート外へ出ないことを検証する

以下のパスは `404 Not Found` とする。

- `..` セグメントを含む
- 絶対パスである
- URL decode 後に `/` または `\` による配信ルート外脱出を試みる
- NUL 文字を含む
- 空白のみのセグメントを含む
- `.` で始まるセグメントを含む

ファイル探索は `storage.basePath/storage/projects/{projectId}/contents/` 配下に限定する。

正規化後の相対パスを `contents/` に結合した結果が、`contents/` 配下に収まらない場合は `404 Not Found` とする。

ディレクトリ自体は配信しない。正規化後の対象がディレクトリの場合、末尾 `/` の有無にかかわらず `index.html` を探索する。

存在しない静的ファイルは `404 Not Found` とする。

読み込み権限不足、ファイル情報取得失敗、読み込み途中失敗は `500 Internal Server Error` とする。

静的配信レスポンスの `Content-Type` は、Go 標準ライブラリの拡張子判定を優先し、判定不能な場合は先頭512 bytesによる判定を行う。なお判定不能な場合は `application/octet-stream` とする。

`HEAD` は `GET` と同じヘッダーを返し、レスポンスボディを返してはならない。

`ETag` は `"{size}-{unixModifiedTime}"` 形式の弱い validator とし、レスポンスでは `W/"{size}-{unixModifiedTime}"` として返す。

`Last-Modified` は対象ファイルの更新時刻を HTTP-date 形式で返す。

`Cache-Control` は既定で `public, max-age=60` とする。

`If-None-Match` が `ETag` と一致する場合は `304 Not Modified` を返す。

`If-Modified-Since` が `Last-Modified` 以降の場合は `304 Not Modified` を返す。

`304 Not Modified` ではレスポンスボディを返してはならない。

Range request は Rev.32 時点では実装しない。

`Range` ヘッダーを受信した場合も無視し、通常の `200 OK` または `304 Not Modified` 判定を行う。

Gzip 圧縮済みファイルを返す場合は `Content-Encoding: gzip` を設定する。

静的配信処理は開発リポジトリ内に配信用一時ファイル、キャッシュファイル、ログ以外の実行時データを作成してはならない。

#### 13.15.5 Webhook 固定仕様

GitHub Webhook は `push` event のみ処理する。

`X-GitHub-Event` が `push` 以外の場合は `200 OK` とし、`{"status":"ignored"}` を返す。

対象ブランチは `deploy.branch` で指定する。

対象外ブランチの場合は `200 OK` とし、`{"status":"ignored"}` を返す。

`webhook.githubSecret` が空文字でない場合は、`X-Hub-Signature-256` を必須とする。

署名ヘッダーは `sha256=<hex>` 形式のみ許可する。

署名検証はリクエストBodyの生バイト列に対してHMAC-SHA256を計算し、`hmac.Equal` で比較する。

署名ヘッダーが存在しない、形式不正、または署名不一致の場合は `401 Unauthorized` とし、`ERR_WEBHOOK_SIGNATURE_INVALID` を返す。

`webhook.githubSecret` が空文字の場合、署名ヘッダーの有無にかかわらず署名検証を行わない。

デプロイ元は `deploy.sourcePath` のローカルcheckoutに限定する。

デプロイ先Projectは `deploy.projectId` で指定する。

`deploy.projectId` が空文字の場合は `ERR_WEBHOOK_PROJECT_NOT_CONFIGURED` を返す。

GitHub Push payload の `repository.clone_url`、`repository.ssh_url`、`repository.html_url` から clone、fetch、pull してはならない。

Webhook処理はネットワーク越しにGit操作を行ってはならない。

`deploy.sourcePath` が存在しない、Git worktreeでない、または `after` commit を参照できない場合は `ERR_WEBHOOK_SOURCE_INVALID` を返す。

静的コンテンツ反映元は `deploy.sourcePath` の `after` commit 時点のファイルツリーとする。

Rev.32 時点では、Webhookデプロイ時の対象ファイルパスはリポジトリルート配下の全静的ファイルとする。

`.git/`、`.github/`、`AGENTS.md`、`ASB-spec.md`、`ASB-spec.html`、`IMPLEMENTATION_TASKS.md`、`DOCUMENT_INDEX.md`、`README.md` は配信対象から除外する。

デプロイ先は対象Projectの `storage/projects/:projectId/contents/` 配下に限定する。

Webhookデプロイは開発リポジトリ内へ実行時データ、一時ファイル、ログファイルを作成してはならない。

冪等キーは `branch + ":" + after` とする。

同一冪等キーが `config/webhooks.json` に存在する場合は `200 OK` とし、`{"status":"duplicate"}` を返す。

Webhook 処理完了後に処理状態を `config/webhooks.json` へ保存する。

成功時の `status` は `deployed` とする。

失敗時の `status` は `failed` とし、`errorCode` を保存する。

Webhook失敗時の自動リトライは Rev.32 時点では実装しない。

GitHub側から同一イベントが再送された場合は、`config/webhooks.json` の既存イベントにより重複判定する。

Webhookデプロイ処理順序は以下に固定する。

1. `X-GitHub-Event` を確認する
2. 必要な場合のみ `X-Hub-Signature-256` を検証する
3. GitHub Push payload をJSONとして解析する
4. `ref` から対象ブランチを取得する
5. 対象外ブランチの場合は `ignored` として終了する
6. `after` と `deploy.branch` から冪等キーを生成する
7. 同一冪等キーが存在する場合は `duplicate` として終了する
8. `deploy.sourcePath` の存在、Git worktree、`after` commit 参照可否を検証する
9. `deploy.projectId` の対象Project存在を検証する
10. `after` commit の静的ファイルツリーを列挙する
11. 配信対象外ファイルを除外する
12. 対象Projectの `contents/` へ一時ディレクトリを作成する
13. 静的ファイルを一時ディレクトリへコピーする
14. `fsync` 後に atomic rename で `contents/` を置き換える
15. `storage/projects/:projectId/files.json` を更新する
16. `config/webhooks.json` に `deployed` を保存する
17. 成功レスポンスを返す

8-15 の途中で失敗した場合は `failed` を `config/webhooks.json` に保存し、`ERR_WEBHOOK_PROCESSING_FAILED` または具体的なエラーコードを返す。

Webhook固定仕様のテスト項目は以下とする。

- secret未設定時に署名なしリクエストを受け付ける
- secret設定時に署名なしリクエストを `401` で拒否する
- secret設定時に不正署名を `401` で拒否する
- secret設定時に正しい `X-Hub-Signature-256` を受け付ける
- `push` 以外のイベントを `ignored` とする
- 対象外ブランチを `ignored` とする
- 同一冪等キーを `duplicate` とする
- payload の remote URL をデプロイ元として使わない
- `deploy.sourcePath` が存在しない場合に失敗する
- `after` commit を参照できない場合に失敗する
- Webhook失敗時に自動リトライを作成しない
- 開発リポジトリ内に実行時データ、一時ファイル、ログファイルを作成しない

#### 13.15.6 実装順序固定

初期実装は以下の順序で進める。

1. `config` package と起動時検証
2. JSON 保存基盤
3. `server` package と共通HTTPレスポンス
4. Project Service / Handler
5. File Service / Handler と静的配信
6. Domain Service / Handler
7. Log / Monitoring
8. Backup / Restore
9. Webhook
10. SSL 管理境界

各段階は `go test ./...` が成功する状態で次へ進む。

### 13.16 入出力契約固定仕様

本節は Rev.32 時点で API、JSON保存、ログ、起動時検証の入出力を固定する仕様である。

#### 13.16.1 共通成功レスポンス契約

成功レスポンスは常に JSON object とする。

成功レスポンスの `Content-Type` は `application/json; charset=utf-8` とする。

成功レスポンス内の日時は UTC RFC3339 秒精度とする。

成功レスポンス内のIDは UUID 文字列とする。

配列レスポンスは対象キーを必ず含め、対象データが空の場合は空配列を返す。

#### 13.16.2 API 成功レスポンス固定表

| API | HTTP | 固定レスポンス |
|-----|------|----------------|
| Project 作成 | 201 | `{"id":string,"name":string,"quota":number,"used":number,"domains":[],"createdAt":string}` |
| Project 一覧 | 200 | `{"projects":[Project...]}` |
| Project 削除 | 200 | `{"status":"deleted","projectId":string,"deletedAt":string}` |
| Domain 追加 | 201 | `{"domain":string,"projectId":string,"isCustom":true,"sslCert":string,"createdAt":string}` |
| Domain 一覧 | 200 | `{"domains":[Domain...]}` |
| Domain 削除 | 200 | `{"status":"deleted","projectId":string,"domain":string,"deletedAt":string}` |
| File upload | 201 | `{"id":string,"projectId":string,"name":string,"size":number,"path":string,"uploadedAt":string}` |
| File 一覧 | 200 | `{"files":[File...]}` |
| File 削除 | 200 | `{"status":"deleted","projectId":string,"fileName":string,"deletedAt":string}` |
| Backup 一覧 | 200 | `{"backups":[Backup...]}` |
| Backup 復旧 | 200 | `{"status":"restored","backupId":string,"restoredAt":string}` |
| Monitoring | 200 | `{"cpu":number|null,"memory":number|null,"disk":number|null,"connections":number|null,"requests":number|null,"checkedAt":string}` |
| Access log | 200 | `{"logs":[AccessLog...],"limit":number,"offset":number}` |
| Error log | 200 | `{"logs":[ErrorLog...],"limit":number,"offset":number}` |
| GitHub Webhook 処理 | 200 | `{"status":"deployed","branch":string,"after":string,"processedAt":string}` |
| GitHub Webhook 無視 | 200 | `{"status":"ignored","reason":string}` |
| GitHub Webhook 重複 | 200 | `{"status":"duplicate","key":string}` |

#### 13.16.3 共通エラーレスポンス契約

エラーレスポンスは常に以下の JSON object とする。

```json
{
  "error": "string",
  "code": "string",
  "timestamp": "2026-09-08T00:00:00Z",
  "httpStatus": 400
}
```

`error` は利用者向けの短い英語メッセージとする。

`code` は `8.7 エラーハンドリング` のエラーコード定義に存在する値のみ許可する。

`timestamp` は UTC RFC3339 秒精度とする。

`httpStatus` は実際の HTTP ステータスコードと一致させる。

エラーレスポンスに内部ファイルパス、スタックトレース、環境変数、シークレット、OSユーザー名を含めてはならない。

#### 13.16.4 保存JSON配列要素スキーマ固定

`config/projects.json` の `projects[]` は以下の形式とする。

```json
{
  "id": "string(UUID)",
  "name": "string",
  "quota": 1073741824,
  "used": 0,
  "domains": [],
  "createdAt": "2026-09-08T00:00:00Z"
}
```

`config/domains.json` の `domains[]` は以下の形式とする。

```json
{
  "domain": "example.com",
  "projectId": "string(UUID)",
  "isCustom": true,
  "sslCert": "string",
  "createdAt": "2026-09-08T00:00:00Z"
}
```

`storage/projects/:projectId/files.json` の `files[]` は以下の形式とする。

```json
{
  "id": "string(UUID)",
  "projectId": "string(UUID)",
  "name": "index.html",
  "size": 2048,
  "path": "contents/index.html",
  "uploadedAt": "2026-09-08T00:00:00Z"
}
```

`config/backups.json` の `backups[]` は以下の形式とする。

```json
{
  "id": "string(UUID)",
  "projectId": "string(UUID)",
  "createdAt": "2026-09-08T00:00:00Z",
  "size": 536870912,
  "path": "backups/backup-id.tar.gz",
  "sha256": "string",
  "status": "completed"
}
```

`config/webhooks.json` の `events[]` は以下の形式とする。

```json
{
  "key": "main:abcdef1234567890",
  "branch": "main",
  "after": "abcdef1234567890",
  "status": "deployed",
  "receivedAt": "2026-09-08T00:00:00Z",
  "completedAt": "2026-09-08T00:00:05Z",
  "errorCode": ""
}
```

保存JSON内のパスは `storage.basePath` からの相対パスとし、絶対パスを保存してはならない。

保存JSON内の日時は UTC RFC3339 秒精度とする。

保存JSON内の未知フィールドは読み込み時にエラーとする。

#### 13.16.5 保存順序固定

JSON 保存時の配列順序は以下で固定する。

| ファイル | ソート順 |
|---------|----------|
| `config/projects.json` | `createdAt` 昇順、同一時刻の場合は `id` 昇順 |
| `config/domains.json` | `domain` 昇順 |
| `config/backups.json` | `createdAt` 降順、同一時刻の場合は `id` 昇順 |
| `storage/projects/:projectId/files.json` | `name` 昇順 |
| `config/webhooks.json` | `receivedAt` 降順、同一時刻の場合は `key` 昇順 |

#### 13.16.6 起動時検証出力固定

起動時検証は以下の順序で実行する。

1. `config/config.json` の存在確認
2. `config/config.json` の JSON 構文検証
3. 設定未知フィールド検証
4. 設定値の範囲検証
5. `storage.basePath` の存在確認
6. `storage.basePath` が開発リポジトリ配下でないことの検証
7. 必須ディレクトリ存在確認
8. 必須 JSON ファイル存在確認
9. 必須 JSON ファイルの構文とスキーマ検証
10. 読み込み権限と書き込み権限の検証

起動時検証失敗時は HTTP サーバーを起動せず、終了コード `1` で終了する。

標準エラーには以下の1行のみを出力する。

```text
ASB_STARTUP_ERROR code=ERR_STORAGE_VALIDATION_FAILED message="Storage validation failed"
```

エラーコードは、失敗原因が JSON 構文の場合は `ERR_INVALID_JSON`、保存先検証の場合は `ERR_STORAGE_VALIDATION_FAILED`、設定未知フィールドの場合は `ERR_UNKNOWN_FIELD` とする。

#### 13.16.7 ログJSON Lines固定

ログファイルは `storage.basePath/logs/` 配下に保存する。

アクセスログファイル名は `access.log` とする。

エラーログファイル名は `error.log` とする。

ログファイルは UTF-8 JSON Lines とする。

1行に1 JSON object のみを出力し、各行末尾は LF とする。

アクセスログは1リクエストにつき1行の JSON Lines とし、以下のフィールドと順序を固定する。

```json
{
  "time": "2026-09-08T00:00:00Z",
  "level": "INFO",
  "domain": "system",
  "method": "GET",
  "path": "/api/projects",
  "status": 200,
  "durationMs": 12,
  "remoteAddr": "127.0.0.1",
  "requestId": "string(UUID)"
}
```

アクセスログの `level` は通常 `INFO` とする。

アクセスログの `domain` は処理対象に応じて `management`、`delivery`、`data`、`system` のいずれかとする。

アクセスログの `durationMs` はリクエスト受信からレスポンス書き込み完了までのミリ秒整数とする。

アクセスログはレスポンス書き込み完了後に記録する。

エラーログは WARN 以上を対象とし、以下のフィールドと順序を固定する。

```json
{
  "time": "2026-09-08T00:00:00Z",
  "level": "ERROR",
  "domain": "delivery",
  "code": "ERR_FILE_UPLOAD_FAILED",
  "message": "File upload failed",
  "requestId": "string(UUID)"
}
```

エラーログの `level` は `WARN` または `ERROR` とする。

エラーログの `domain` は `management`、`delivery`、`data`、`system` のいずれかとする。

エラーログの `code` は定義済みエラーコードまたは空文字とする。定義済みエラーコードがある場合は空文字にしてはならない。

エラーログの `message` は利用者向けエラーメッセージではなく、運用者向けの短い固定文言とする。

HTTP リクエスト処理中に発生したエラーでは、エラーログの `requestId` をアクセスログおよびレスポンスヘッダーと一致させる。

リクエスト外のエラーでは `requestId` は空文字を許可する。

`log.level` は出力する最小レベルを表す。

`DEBUG` は `debug` 設定時のみ出力する。

`INFO` は `debug` または `info` 設定時に出力する。

`WARN` は `debug`、`info`、`warn` 設定時に出力する。

`ERROR` はすべての `log.level` 設定で出力する。

`log.format` は Rev.32 時点では `json` のみ許可する。

通常運用ログを stdout へ出力してはならない。

起動時検証失敗時のみ、stderr へ以下の形式で単一行を出力する。

```text
ASB_STARTUP_ERROR code=ERR_STORAGE_VALIDATION_FAILED message="Storage validation failed"
```

ログローテーションは現在ログファイルのサイズが `log.maxSize` 以上になった後、次回書き込み前に実行する。

ローテーションでは現在ログファイルを `access.log.{unixTime}` または `error.log.{unixTime}` へ rename し、新しい `access.log` または `error.log` を作成する。

ローテーション済みログの削除対象は、ファイル名末尾の unixTime が現在時刻から7日より古いものに限定する。

ログローテーション失敗時は対象ログ書き込みを失敗扱いとし、HTTP レスポンスが未送信の場合は `500 Internal Server Error` を返す。

ログ API は `access.log` または `error.log` の現行ファイルのみを読む。ローテーション済みログは Rev.32 時点ではログ API の対象外とする。

ログには内部ファイルパス、スタックトレース、環境変数、シークレットを含めてはならない。

#### 13.16.8 テスト固定項目

Rev.32 の実装では、以下のテストを必須とする。

- 全API成功レスポンスの固定JSONキー検証
- 全APIエラーレスポンスの固定JSONキー検証
- 保存JSONの未知フィールド拒否
- 保存JSONの相対パス保存検証
- 保存JSONのソート順検証
- 起動時検証の順序、終了コード、標準エラー形式検証
- アクセスログとエラーログのJSON Linesフィールド検証

### 13.17 実装境界とファイル操作固定仕様

本節は Rev.32 時点で package 境界、公開 interface、Repository、Storage、複数ファイル更新の実装契約を固定する仕様である。

#### 13.17.1 package 公開 interface 固定

各 package は以下の公開 interface を境界として実装する。

| package | 公開 interface | 主な責務 |
|---------|----------------|----------|
| `config` | `Loader` | 設定読み込み、デフォルト適用、起動時検証 |
| `server` | `Router`, `Responder` | HTTPルーティング、成功/エラーJSON応答 |
| `management` | `ProjectService`, `DomainService`, `SSLService` | Project、Domain、SSL管理境界 |
| `delivery` | `FileService`, `StaticService`, `WebhookService` | ファイル管理、静的配信、Webhook |
| `data` | `JSONRepository`, `StorageService`, `BackupService` | JSON永続化、ストレージ、バックアップ/復旧 |
| `system` | `LogService`, `MonitoringService`, `Clock`, `IDGenerator` | ログ、監視、時刻、ID生成 |

Handler は Service interface のみに依存する。

Service は Repository、Storage、Clock、IDGenerator、LogService interface に依存できる。

Entity は interface を定義せず、保存形式とレスポンス形式の型定義のみを持つ。

他 package の具象型を直接生成してよい場所は `cmd/asb/main.go` の依存関係生成処理のみとする。

#### 13.17.2 Repository / Storage 責務固定

`JSONRepository` は JSON ファイルの読み込み、スキーマ検証、排他、atomic save のみを担当する。

`JSONRepository` は HTTP ステータス、HTTP リクエスト、HTTP レスポンスを扱ってはならない。

`StorageService` は `storage.basePath` 配下のファイル実体操作のみを担当する。

`StorageService` は Project、Domain、Webhook の業務判断を行ってはならない。

Service は業務判断、整合性判断、複数Repository/Storage操作の順序制御を担当する。

複数JSONまたはJSONとファイル実体をまたぐ操作では、Service が処理全体の成功/失敗を決定する。

#### 13.17.3 静的配信処理順序固定

Static delivery は以下の順序で実行する。

1. リクエストパスが `/api/` で始まる場合は API ルーティングへ渡す
2. HTTP method が `GET` または `HEAD` であることを検証する
3. `GET` または `HEAD` 以外の場合は `405 Method Not Allowed` と `Allow: GET, HEAD` を返す
4. `Host` ヘッダーを取得する
5. `Host` の port、末尾 `.`, 大文字小文字を正規化する
6. 正規化後 Host を `config/domains.json` の `domain` と完全一致で照合する
7. Domain が存在しない場合は `404 Not Found` を返す
8. Domain の `projectId` に対応する Project の存在を検証する
9. Project が存在しない場合は `ERR_STORAGE_VALIDATION_FAILED` をログに記録し、`500 Internal Server Error` を返す
10. URL path を decode し、静的配信用相対パスへ正規化する
11. 不正パスまたは配信ルート外脱出は `404 Not Found` を返す
12. 空パス、`/`、ディレクトリパスは `index.html` を探索対象とする
13. `storage.basePath/storage/projects/{projectId}/contents/` と相対パスを結合する
14. 結合後パスが `contents/` 配下に収まることを検証する
15. 対象ファイルの存在と通常ファイルであることを検証する
16. 存在しない場合は `404 Not Found` を返す
17. `Content-Type`、`ETag`、`Last-Modified`、`Cache-Control` を決定する
18. `If-None-Match` または `If-Modified-Since` により未変更と判定できる場合は `304 Not Modified` を返す
19. `HEAD` の場合はヘッダーのみを返す
20. `GET` の場合はファイル内容をレスポンスボディとして返す

静的配信では `Range` ヘッダーを無視し、`206 Partial Content` を返してはならない。

静的配信ではディレクトリ一覧を返してはならない。

静的配信では開発リポジトリ内に配信用一時ファイル、キャッシュファイル、実行時データを作成してはならない。

#### 13.17.4 ファイルアップロード処理順序固定

File upload は以下の順序で実行する。

1. URL `:id` を検証する
2. Project の存在を検証する
3. multipart field `file` の存在を検証する
4. ファイル名、サイズ、quota を検証する
5. 既存 `files.json` を読み込み検証する
6. 保存先相対パスを決定する
7. ファイル実体を `contents/` 配下の一時ファイルへ書き込む
8. 書き込み内容を `fsync` する
9. 一時ファイルを公開先へ atomic rename する
10. `files.json` を更新する
11. `projects.json` の `used` を更新する
12. 成功レスポンスを返す

7-11 の途中で失敗した場合、成功レスポンスを返してはならない。

公開先への rename 後に JSON 更新が失敗した場合は、エラーログを記録し、次回起動時検証または整合性検証で検出できる状態にする。

#### 13.17.5 ファイル上書き処理順序固定

同名ファイル上書きは以下の順序で実行する。

1. 旧ファイルメタデータを読み込む
2. 新ファイルを一時ファイルへ書き込む
3. 新ファイルを `fsync` する
4. 新ファイルを公開先へ atomic rename する
5. `files.json` の `size`、`uploadedAt`、`path` を更新する
6. `projects.json` の `used` を差分更新する

旧ファイルは、新ファイルの atomic rename が成功するまで削除してはならない。

上書き後の `used` は旧サイズを差し引き、新サイズを加算して計算する。

#### 13.17.6 ファイル削除処理順序固定

File delete は以下の順序で実行する。

1. URL `:id` と `:name` を検証する
2. Project の存在を検証する
3. `files.json` から対象ファイルを検出する
4. ファイル実体を削除する
5. `files.json` から対象メタデータを削除する
6. `projects.json` の `used` を差分更新する
7. 成功レスポンスを返す

ファイル実体が存在しないが `files.json` にメタデータが存在する場合は、整合性エラーとして `ERR_FILE_NOT_FOUND` を返す。

JSON 更新失敗時は成功レスポンスを返してはならない。

#### 13.17.7 Project削除処理順序固定

Project delete は以下の順序で実行する。

1. Project の存在を検証する
2. 対象 Project に紐づく Domain を列挙する
3. 対象 Project に紐づく Backup を列挙する
4. 対象 Project の `files.json` を読み込み検証する
5. 対象 Project の `contents/` 配下を削除する
6. 対象 Project の `files.json` を削除する
7. `config/domains.json` から関連 Domain を削除する
8. `config/backups.json` から関連 Backup 履歴を削除する
9. `config/projects.json` から対象 Project を削除する
10. 成功レスポンスを返す

5-9 の途中で失敗した場合、成功レスポンスを返してはならない。

Project削除は best effort 成功扱いにしてはならない。

#### 13.17.8 Backup作成処理順序固定

Backup 作成は以下の順序で実行する。

1. 対象 Project の存在を検証する
2. 対象 Project の JSON と `contents/` を読み込み可能であることを検証する
3. `storage.basePath/backups/` が存在し、書き込み可能であることを検証する
4. `storage.basePath/backups/.tmp/backup-{backupId}.tar.gz.tmp` を作成する
5. tar.gz には対象Projectの `files.json` と `contents/` のみを含める
6. tar.gz 作成後に SHA-256 を計算する
7. `storage.basePath/backups/backup-{backupId}.tar.gz` へ atomic rename する
8. `config/backups.json` に `status: "completed"` の履歴を保存する
9. 成功レスポンスを返す

バックアップtar.gzに `logs/`、`certs/`、他Projectの `contents/` を含めてはならない。

`config/backups.json` の `path` には `backups/backup-{backupId}.tar.gz` を保存する。

tar.gz 作成、ハッシュ計算、atomic rename、履歴保存のいずれかに失敗した場合、成功レスポンスを返してはならない。

tar.gz 作成またはハッシュ計算に失敗した場合、`config/backups.json` に履歴を追加してはならない。

atomic rename 後に履歴保存へ失敗した場合は、作成済みtar.gzを削除し、削除失敗時はエラーログへ記録する。

バックアップ作成に使用する一時ファイルは、処理終了時に削除する。

開発リポジトリ内にバックアップtar、一時tar、checksum、退避データを作成してはならない。

#### 13.17.9 Backup復旧処理順序固定

Backup restore は以下の順序で実行する。

1. Backup 履歴の存在を検証する
2. Backup 履歴の `status` が `completed` であることを検証する
3. Backup ファイルの存在を検証する
4. SHA-256 を検証する
5. `storage.basePath/backups/restore-staging/{restoreId}/previous/` を作成する
6. `storage.basePath/backups/restore-staging/{restoreId}/next/` を作成する
7. 復旧対象Projectの現行 `files.json` と `contents/` を `previous/` へ退避する
8. Backup を `next/` へ展開する
9. 展開後の JSON 構文とスキーマを検証する
10. `next/` の内容を復旧対象へ atomic rename する
11. 成功レスポンスを返す

7-10 の途中で失敗した場合、可能な限り `previous/` から復元する。

復元に失敗した場合は `ERR_BACKUP_RESTORE_FAILED` を返し、成功レスポンスを返してはならない。

復旧対象Projectが存在しない場合は `ERR_PROJECT_NOT_FOUND` を返す。

復旧対象Projectの現行データ退避に失敗した場合は、復旧処理を開始してはならない。

復旧後の `restore-staging/{restoreId}/` 削除は best effort とし、削除失敗時は WARN ログへ記録する。

開発リポジトリ内に復旧用一時ファイル、退避データ、展開データを作成してはならない。

#### 13.17.10 複数JSON更新失敗時契約

複数JSON更新は、操作順序を Service に閉じ込める。

複数JSON更新では、最終JSONの保存が完了するまで成功レスポンスを返してはならない。

途中失敗時は、更新済みJSON名、未更新JSON名、操作名、requestId をエラーログへ記録する。

途中失敗時に自動ロールバックを実装する場合も、ロールバック失敗時は成功扱いにしてはならない。

Rev.32 時点では、複数JSON更新に外部トランザクション機構を導入してはならない。

#### 13.17.11 最低テスト分類固定

`go test ./...` に含める最低テスト分類は以下とする。

| 分類 | 対象 |
|------|------|
| Unit | Entity validation、Service validation、Repository validation |
| Handler | ルーティング、Content-Type、Body decode、レスポンスJSON |
| Repository | unknown field、atomic save、sort order、保存失敗 |
| Storage | path traversal、relative path、upload、overwrite、delete |
| Integration | Project/File/Domain/Backup/Webhook の成功系と主要失敗系 |
| Startup | 起動時検証順序、終了コード、stderr |

#### 13.17.12 実装ファイル構成固定

Rev.32 の初期実装では、Go 実装ファイルを以下の構成で作成する。

```text
cmd/asb/main.go
internal/config/
internal/server/
internal/management/
internal/delivery/
internal/data/
internal/system/
```

`cmd/asb/main.go` は、設定読み込み、依存関係生成、HTTP サーバー起動、signal 受信、graceful shutdown のみを行う。

`internal/config/` は設定構造体、設定読み込み、デフォルト適用、未知フィールド拒否、起動時検証を実装する。

`internal/server/` は `net/http` ルーティング、middleware、requestId 付与、共通 JSON レスポンス、共通エラーレスポンス、HTTP テスト補助を実装する。

`internal/management/` は Project、Domain、SSL 管理境界の Entity、Service interface、Service 実装、Handler を実装する。

`internal/delivery/` は File 管理、静的配信、GitHub Webhook の Entity、Service interface、Service 実装、Handler を実装する。

`internal/data/` は JSON Repository、StorageService、BackupService、atomic save、tar.gz、restore staging を実装する。

`internal/system/` は Clock、IDGenerator、LogService、MonitoringService、ログローテーション、監視値取得を実装する。

Go package 名はディレクトリ名と一致させる。

外部公開 API 用 package を追加してはならない。

`internal/` 外へ ASB 本体の実装 package を追加する場合は、先に本仕様を改訂する。

#### 13.17.13 package 内ファイル役割固定

各 package 内のファイル役割は以下を基本とする。

| package | ファイル | 役割 |
|---------|----------|------|
| `config` | `config.go` | 設定構造体とデフォルト値 |
| `config` | `loader.go` | JSON 読み込み、未知フィールド拒否、デフォルト適用 |
| `config` | `validate.go` | 起動時設定検証 |
| `server` | `router.go` | API と静的配信の振り分け |
| `server` | `response.go` | 成功 JSON とエラー JSON |
| `server` | `middleware.go` | requestId、アクセスログ、panic 抑止 |
| `management` | `project.go` | Project Entity と ProjectService |
| `management` | `domain.go` | Domain Entity と DomainService |
| `management` | `ssl.go` | SSL Entity と SSLService |
| `management` | `handler.go` | Management API Handler |
| `delivery` | `file.go` | File Entity と FileService |
| `delivery` | `static.go` | StaticService |
| `delivery` | `webhook.go` | Webhook Entity と WebhookService |
| `delivery` | `handler.go` | File API と Webhook API Handler |
| `data` | `json_repository.go` | JSON 読み込み、検証、atomic save |
| `data` | `storage.go` | `storage.basePath` 配下の実体ファイル操作 |
| `data` | `backup.go` | Backup 作成と復旧 |
| `system` | `clock.go` | UTC RFC3339 秒精度時刻 |
| `system` | `id.go` | UUID 形式 ID 生成 |
| `system` | `log.go` | JSON Lines ログとローテーション |
| `system` | `monitoring.go` | 監視値取得 |

上記ファイル分割は実装開始時の固定構成とする。

責務が増えてファイル分割が必要な場合でも、package 境界と公開 interface を変更してはならない。

#### 13.17.14 エラーコード固定表

Rev.32 の実装では、API と起動時検証が返すエラーコードを以下に固定する。

| code | HTTP | 用途 |
|------|------|------|
| `ERR_INVALID_JSON` | 400 | リクエスト JSON または保存 JSON の構文不正 |
| `ERR_UNKNOWN_FIELD` | 400 | リクエスト JSON、設定 JSON、保存 JSON の未知フィールド |
| `ERR_INVALID_REQUEST` | 400 | Content-Type、Body、query、path parameter の不正 |
| `ERR_PROJECT_NOT_FOUND` | 404 | Project が存在しない |
| `ERR_PROJECT_ALREADY_EXISTS` | 409 | Project 名が重複している |
| `ERR_PROJECT_QUOTA_EXCEEDED` | 413 | Project quota を超過する |
| `ERR_DOMAIN_NOT_FOUND` | 404 | Domain が存在しない |
| `ERR_DOMAIN_ALREADY_ASSIGNED` | 409 | Domain が別 Project または同一 Project に割り当て済み |
| `ERR_FILE_NOT_FOUND` | 404 | File メタデータまたは実体が存在しない |
| `ERR_FILE_UPLOAD_FAILED` | 500 | File upload の永続化または実体保存に失敗 |
| `ERR_STORAGE_VALIDATION_FAILED` | 500 | `storage.basePath`、必須ディレクトリ、必須 JSON の検証失敗 |
| `ERR_BACKUP_NOT_FOUND` | 404 | Backup 履歴またはファイルが存在しない |
| `ERR_BACKUP_RESTORE_CONFLICT` | 409 | 復旧対象の状態が復旧条件を満たさない |
| `ERR_BACKUP_RESTORE_FAILED` | 500 | Backup 復旧またはロールバックに失敗 |
| `ERR_WEBHOOK_SIGNATURE_INVALID` | 401 | GitHub Webhook 署名が不正 |
| `ERR_WEBHOOK_PROJECT_NOT_CONFIGURED` | 500 | `deploy.projectId` が未設定 |
| `ERR_WEBHOOK_SOURCE_INVALID` | 500 | `deploy.sourcePath` または `after` commit が不正 |
| `ERR_WEBHOOK_PROCESSING_FAILED` | 500 | Webhook デプロイ処理に失敗 |
| `ERR_SSL_CERT_GENERATION_FAILED` | 500 | SSL 証明書管理境界で証明書状態検証に失敗 |
| `ERR_LOG_READ_FAILED` | 500 | ログ API の読み込みまたは JSON Lines 検証に失敗 |
| `ERR_INTERNAL` | 500 | 上記に分類できない内部エラー |

`code` は上記表のいずれかでなければならない。

HTTP ステータスは上記表と `13.15.1 API エンドポイント固定表` の範囲で決定する。

保存JSONの起動時検証失敗では、構文不正を `ERR_INVALID_JSON`、未知フィールドを `ERR_UNKNOWN_FIELD`、ディレクトリ・権限・必須ファイル不備を `ERR_STORAGE_VALIDATION_FAILED` とする。

#### 13.17.15 テストファイル配置固定

Rev.32 の実装では、最低限以下のテストファイルを作成する。

| フェーズ | テストファイル |
|----------|----------------|
| P0 | `internal/system/id_test.go`, `internal/system/clock_test.go` |
| P1 | `internal/config/loader_test.go`, `internal/config/validate_test.go`, `internal/server/response_test.go`, `internal/data/json_repository_test.go` |
| P2 | `internal/delivery/file_test.go`, `internal/delivery/static_test.go`, `internal/data/storage_test.go` |
| P3 | `internal/management/domain_test.go`, `internal/management/ssl_test.go` |
| P4 | `internal/delivery/webhook_test.go` |
| P5 | `internal/data/backup_test.go` |
| P6 | `internal/system/log_test.go`, `internal/system/monitoring_test.go` |
| P7 | `internal/data/migration_test.go` |
| P8 | `scripts/install_test.sh`, `scripts/update_test.sh` |
| P9 | `internal/asb_forbidden_test.go` |

テストは開発リポジトリ内に実行時データ、ログ、一時ファイル、ビルド成果物を残してはならない。

テストで実行時データ領域が必要な場合は、テストごとに OS の一時ディレクトリを作成し、テスト終了時に削除する。

テスト用 fixture をリポジトリ内に追加する場合は、静的な入力データのみ許可する。

テスト fixture は実行結果、ログ、バックアップ、ビルド成果物、coverage 出力を含んではならない。

#### 13.17.16 フェーズ完了判定固定

各実装フェーズは、以下の全条件を満たすまで完了として扱ってはならない。

1. 該当フェーズの実装タスクがすべて完了している
2. 該当フェーズの完了条件テストが成功している
3. `go test ./...` が成功している
4. `git diff --check` が成功している
5. `.gitignore` が存在しない
6. 開発リポジトリ内に実行時データ、ログ、一時ファイル、ビルド成果物、coverage 出力が残っていない
7. `ASB-spec.md`、`ASB-spec.html`、`IMPLEMENTATION_TASKS.md` の参照バージョンが一致している

フェーズ完了時の開発版バージョンは、`11.9.2.5 実装フェーズバージョン` の対応表に従う。

フェーズ途中の commit は許可するが、完了版として扱えるのは上記条件をすべて満たす commit のみとする。

---

### 13.18 APIキー管理・認証固定仕様

Rev.32 時点では、ASB 本体の管理 API に認証機能を実装しない。

管理 API とは `/api/` で始まる全 HTTP JSON API を指す。

管理 API は、ASB の標準構成では `server.host=localhost` で待ち受け、外部公開を前提としない。

本番環境で管理 API を外部ネットワークから利用可能にする場合は、ASB 外部のリバースプロキシ、ファイアウォール、VPN、SSH tunnel、IP制限等で保護する。

ASB 本体は Rev.32 時点では以下を実装してはならない。

- APIキー発行
- APIキー保存
- APIキー照合
- APIキー失効
- APIキーローテーション
- APIキー権限スコープ
- APIキー監査履歴
- ユーザー認証
- セッション管理
- JWT 検証
- OAuth / OIDC 連携
- Basic 認証
- Bearer token 認証
- 認証 middleware

`Authorization` ヘッダーまたは `X-API-Key` ヘッダーを受信しても、ASB は認証判断に使用してはならない。

Rev.32 時点では、`Authorization` ヘッダーまたは `X-API-Key` ヘッダーの有無により、成功・失敗・レスポンス内容を変えてはならない。

ASB は APIキー管理のために以下の JSON ファイル、ディレクトリ、設定項目を作成してはならない。

- `config/api_keys.json`
- `config/users.json`
- `config/sessions.json`
- `storage/api_keys/`
- `storage/users/`
- `auth.*`
- `apiKey.*`

開発リポジトリ内にも、`storage.basePath` 配下にも、APIキー、ユーザー、セッション、認証状態を表す実行時データを生成してはならない。

`config/config.json` に認証関連フィールドが存在する場合は、未知フィールドとして起動失敗とする。

APIキー管理を将来実装する場合は、実装前に `ASB-spec.md` を改訂し、以下を最低限確定する。

- APIキー保存形式
- APIキーのハッシュ化方式
- 発行、失効、ローテーション API
- 権限スコープ
- 管理 API への適用範囲
- SDK との認証連携
- 監査ログ
- 既存の認証なし管理 API からの移行手順

SDK 認証仕様は Rev.32 の対象外とし、実装対象へ昇格する場合は事前に `ASB-spec.md` を改訂する。

---

### 13.19 Rate limiting 固定仕様

Rev.32 時点では、ASB 本体に Rate limiting を実装しない。

Rate limiting とは、送信元IP、Host、Domain、Project、APIキー、ユーザー、HTTPメソッド、URL path、リクエスト数、転送量、同時接続数、時間窓等に基づき、HTTP リクエストの受理、拒否、遅延、または優先度を制御する機能を指す。

ASB 本体は Rev.32 時点では以下を実装してはならない。

- Rate limiting middleware
- IP別リクエスト制限
- Host別リクエスト制限
- Domain別リクエスト制限
- Project別リクエスト制限
- API別リクエスト制限
- Webhook別リクエスト制限
- 静的配信別リクエスト制限
- token bucket
- leaky bucket
- sliding window counter
- fixed window counter
- 同時接続数制限
- 転送量制限
- `429 Too Many Requests` の Rate limiting 用返却
- `Retry-After` ヘッダーの Rate limiting 用返却

ASB は Rate limiting のために以下の JSON ファイル、ディレクトリ、設定項目を作成してはならない。

- `config/rate_limits.json`
- `config/limits.json`
- `storage/rate_limits/`
- `storage/counters/`
- `rateLimit.*`
- `limits.*`

開発リポジトリ内にも、`storage.basePath` 配下にも、Rate limiting の判定状態、集計値、カウンタ、時間窓、送信元別状態を表す実行時データを生成してはならない。

`config/config.json` に Rate limiting 関連フィールドが存在する場合は、未知フィールドとして起動失敗とする。

管理 API、静的配信、Webhook 受信は、Rate limiting の有無により成功・失敗・レスポンス内容を変えてはならない。

本番環境で Rate limiting が必要な場合は、ASB 外部のリバースプロキシ、WAF、CDN、ファイアウォール、ロードバランサ等で実施する。

ASB 本体は、外部 Rate limiting 実装の存在を検出、要求、制御、または設定してはならない。

Rate limiting を将来実装する場合は、実装前に `ASB-spec.md` を改訂し、以下を最低限確定する。

- 適用対象
- 制限単位
- 時間窓
- アルゴリズム
- 永続化要否
- JSON保存形式
- レスポンスステータス
- レスポンスヘッダー
- 管理 API との関係
- 静的配信との関係
- Webhook との関係
- SDK との関係
- ログ、監査、メトリクス
- 既存の Rate limiting なし構成からの移行手順

---

### 13.20 Brotli 圧縮固定仕様

Rev.32 時点では、ASB 本体に Brotli 圧縮を実装しない。

ASB の標準圧縮機能は、Go 標準ライブラリ `compress/gzip` で実装できる Gzip に限定する。

Brotli 圧縮は Go 標準ライブラリに含まれないため、Rev.32 時点では外部ライブラリ例外採用を行わない。

ASB 本体は Rev.32 時点では以下を実装してはならない。

- Brotli 圧縮
- Brotli 展開
- Brotli 用外部ライブラリ
- Brotli 用 middleware
- Brotli 用 precompress 処理
- Brotli 用動的圧縮
- Brotli 用事前圧縮
- `.br` ファイル自動生成
- `.br` ファイル自動削除
- Brotli 用キャッシュ生成
- Brotli 用キャッシュ削除
- Brotli 用 Content negotiation
- Brotli 用 `Content-Encoding: br` 返却

`Accept-Encoding: br` を受信しても、ASB は Brotli 応答へ切り替えてはならない。

`Accept-Encoding` に `gzip` と `br` の両方が含まれる場合でも、ASB が圧縮応答を返す場合は Gzip のみを使用する。

`Accept-Encoding` に `br` のみが含まれる場合、ASB は Brotli 圧縮を行わず、未圧縮応答または既存 Gzip 仕様に従った応答のみを返す。

ASB は Brotli のために以下のファイル、ディレクトリ、設定項目を作成してはならない。

- `*.br`
- `storage/brotli/`
- `storage/cache/brotli/`
- `brotli.*`
- `compression.brotli.*`

開発リポジトリ内にも、`storage.basePath` 配下にも、Brotli 用の事前圧縮ファイル、キャッシュファイル、一時ファイル、メタデータを生成してはならない。

`config/config.json` に Brotli 関連フィールドが存在する場合は、未知フィールドとして起動失敗とする。

Brotli を将来実装する場合は、実装前に `ASB-spec.md` を改訂し、以下を最低限確定する。

- 外部ライブラリ例外採用理由
- 対象 MIME type
- 圧縮タイミング
- 事前圧縮の有無
- 動的圧縮の有無
- 保存先
- キャッシュ方針
- `.br` ファイルの生成、更新、削除条件
- `Content-Encoding: br` の返却条件
- `Accept-Encoding` の優先順位
- Gzip との優先順位
- ETag / Last-Modified / Cache-Control との関係
- 開発リポジトリ非生成の保証方法
- 既存 Gzip 仕様からの移行手順

---

### 13.21 SDK 通信規格固定仕様

Rev.32 時点では、SDK 本体を実装しない。

Rev.32 時点では、SDK 配布方針を確定しない。

Rev.32 時点では、SDK 認証仕様を確定しない。

ただし、将来 SDK が ASB と通信する場合の通信規格は、ASB 本体が提供する HTTP JSON API と同一に固定する。

SDK 通信は、以下の ASB 管理 API 規約に従う。

- HTTP method
- URL path
- query parameter
- path parameter
- request JSON body
- multipart upload
- success response JSON
- error response JSON
- HTTP status code
- error code
- UTC RFC3339 timestamp
- pagination
- static file upload 規約

SDK 通信のために、ASB 本体は Rev.32 時点では以下を実装してはならない。

- SDK 専用 HTTP API
- SDK 専用 URL prefix
- SDK 専用 request body
- SDK 専用 response body
- SDK 専用 error format
- SDK 専用 pagination
- SDK 専用 upload protocol
- SDK 専用 session
- SDK 専用 token
- SDK 専用 cookie
- SDK 専用 handshake
- SDK 専用 protocol negotiation
- SDK 専用 version negotiation

ASB 本体は Rev.32 時点では以下の通信方式を SDK 通信として実装してはならない。

- WebSocket
- gRPC
- GraphQL
- 独自 TCP プロトコル
- UDP
- MQTT
- AMQP
- Server-Sent Events
- long polling

SDK 通信のために以下の JSON ファイル、ディレクトリ、設定項目を作成してはならない。

- `config/sdk.json`
- `config/sdk_clients.json`
- `config/sdk_sessions.json`
- `storage/sdk/`
- `storage/sdk_sessions/`
- `sdk.*`
- `sdkAuth.*`

開発リポジトリ内にも、`storage.basePath` 配下にも、SDK 通信用のセッション、キャッシュ、handshake 状態、protocol negotiation 状態、client registration 状態を表す実行時データを生成してはならない。

`config/config.json` に SDK 通信関連フィールドが存在する場合は、未知フィールドとして起動失敗とする。

SDK から ASB 管理 API を呼び出す場合でも、Rev.32 時点では `Authorization` ヘッダー、`X-API-Key` ヘッダー、cookie、セッションIDを認証判断に使用してはならない。

SDK 認証仕様を将来実装する場合は、実装前に `ASB-spec.md` を改訂し、APIキー管理、ユーザー認証、SDK配布方針との関係を最低限確定する。

SDK 本体を将来実装する場合は、実装前に `ASB-spec.md` を改訂し、以下を最低限確定する。

- 対象言語
- package 名
- 配布方法
- versioning
- ASB API version との互換性
- エラー型
- retry 方針
- timeout 方針
- upload API の扱い
- 認証仕様
- generated client の採用可否
- テスト方法

---

### 13.22 ACME 実通信固定仕様

Rev.32 時点では、ASB 本体に ACME 実通信を実装しない。

ACME 実通信とは、ACME protocol を用いて CA と通信し、account 登録、order 作成、authorization 取得、challenge 応答、証明書発行、証明書更新、失効、nonce 管理を行う機能を指す。

Rev.32 時点では、SSL 管理は以下に限定する。

- 証明書IDの管理
- 証明書メタデータの管理
- 証明書ファイル配置先の管理
- 証明書ファイルパスの検証
- 証明書ファイル存在確認
- 証明書有効期限の検証
- 証明書有効期限の監視モデル
- 証明書検証失敗時のエラー記録

証明書ファイルそのものは、手動配置または ASB 外部の運用で配置する。

ASB 本体は Rev.32 時点では証明書ファイルを生成、取得、更新、削除、失効してはならない。

ASB 本体は Rev.32 時点では以下を実装してはならない。

- ACME client
- ACME account 登録
- ACME account key 生成
- ACME account key 保存
- ACME directory 取得
- ACME nonce 取得
- ACME order 作成
- ACME authorization 取得
- ACME challenge 応答
- ACME finalize
- ACME certificate download
- ACME revoke
- DNS-01 challenge
- HTTP-01 challenge
- TLS-ALPN-01 challenge
- wildcard 証明書自動取得
- 複数 CA 連携
- CA 選定
- 証明書自動更新
- 証明書更新スケジューラー
- challenge 状態管理
- ACME retry
- ACME rate limit 回避

ASB は ACME のために以下の JSON ファイル、ディレクトリ、設定項目を作成してはならない。

- `config/acme.json`
- `config/ca.json`
- `config/acme_accounts.json`
- `config/acme_orders.json`
- `config/acme_authorizations.json`
- `config/acme_challenges.json`
- `storage/acme/`
- `storage/acme/accounts/`
- `storage/acme/orders/`
- `storage/acme/challenges/`
- `storage/certs/acme/`
- `acme.*`
- `ca.*`
- `ssl.acme.*`

開発リポジトリ内にも、`storage.basePath` 配下にも、ACME account、account key、order、authorization、challenge、nonce、retry、renewal、CA選定状態を表す実行時データを生成してはならない。

`config/config.json` に ACME 関連フィールドまたは CA 選定関連フィールドが存在する場合は、未知フィールドとして起動失敗とする。

ACME 実通信を将来実装する場合は、実装前に `ASB-spec.md` を改訂し、以下を最低限確定する。

- ACME protocol 対応範囲
- 採用 CA
- 複数 CA 方針
- account key 保存形式
- account key 保護方式
- DNS-01 対応可否
- HTTP-01 対応可否
- TLS-ALPN-01 対応可否
- wildcard 証明書対応可否
- DNS provider 連携方式
- order / authorization / challenge 保存形式
- nonce 管理
- retry 方針
- rate limit 方針
- 証明書保存先
- 証明書更新スケジュール
- 更新失敗時挙動
- 監査ログ
- テスト CA / staging CA の扱い
- 既存の手動証明書配置からの移行手順

---

### 13.23 ASB互換目標固定仕様

Rev.32 時点では、ASB互換目標は将来の到達目標であり、実装対象ではない。

ASB互換目標は、XServer Static 等の静的コンテンツ専用ホスティングの利用体験を参考にした ASB 独自の目標である。

ASB互換目標は、外部サービスとの完全互換、API互換、管理画面互換、内部実装互換を意味しない。

Rev.32 時点で ASB互換目標に含める対象は以下とする。

- 静的コンテンツ専用ホスティング
- HTML、CSS、JavaScript、画像等の静的ファイル配信
- プロジェクト単位の公開対象管理
- 独自ドメイン割り当て
- SSL 証明書管理
- SSL / ACME 相当の運用体験
- ACME による証明書取得・更新
- ワイルドカード証明書
- 複数 CA
- HTTP/2
- GitHub 連携による自動デプロイ
- ファイルアップロード
- フォルダ階層を保持したファイル管理
- SSL更新状態、デプロイ状態、ログの確認

Rev.32 時点で ASB互換目標に含めない対象は以下とする。

- XServer Static との完全互換
- XServer Static の管理画面再現
- XServer Static の内部実装再現
- 外部サービスのAPI完全互換
- 外部サービスのDNS管理機能
- 外部サービスのCDN機能完全互換
- 外部サービスの課金、契約、アカウント管理
- 外部サービスの SLA / サポート体制

ASB互換目標に含まれる機能であっても、以下は Rev.32 時点では実装対象ではない。

- ACME 実通信
- CA 選定
- ワイルドカード証明書自動取得
- 複数 CA 連携
- 証明書自動更新
- HTTP/2 実装詳細
- SDK 本体
- SDK 配布
- SDK 認証

ASB互換目標を理由に、以下を追加、変更、生成してはならない。

- `.gitignore`
- 外部DB
- 未承認の外部ライブラリ
- 未承認の外部サービス連携
- 開発リポジトリ内の実行時データ
- 起動時の実行時データ自動生成
- ビルド成果物の開発リポジトリ内自動生成
- 実装未確定の API
- 実装未確定の設定項目
- 実装未確定の JSON ファイル
- 実装未確定のディレクトリ

ASB互換目標に含まれる機能を実装対象へ昇格する場合は、実装前に `ASB-spec.md` を改訂し、以下を最低限確定する。

- 実装対象範囲
- 非対象範囲
- API
- request / response
- error code
- 保存 JSON
- 設定項目
- 生成ファイル
- 生成ディレクトリ
- 開発リポジトリ非生成の保証
- 外部依存の有無
- テスト条件
- 既存仕様からの移行手順

---

### 13.24 将来計画機能固定仕様

Rev.32 時点では、将来計画、保留事項、検討・調査中事項は実装対象ではない。

本節は、将来計画に含まれる機能を実装対象外として固定する。

ASB 本体は Rev.32 時点では以下を実装してはならない。

- GUI
- Web UI
- デスクトップアプリ
- モバイルアプリ
- クラウドサービス化
- SaaS 基盤
- ユーザー管理
- マルチテナント
- 課金管理
- 契約管理
- 複数インスタンス管理
- クラスタ管理
- 分散ロック
- NFS 専用連携
- 分散ストレージ専用連携
- 外部ストレージサービス連携
- ウイルススキャン
- ログファイル暗号化
- HTTP/2 実装詳細

将来計画機能を理由に、ASB 本体は Rev.32 時点では以下を追加、変更、生成してはならない。

- UI 用 API
- モバイル専用 API
- クラウド用 API
- テナント用 API
- 課金用 API
- 契約用 API
- 外部ストレージ用 API
- ウイルススキャン用 API
- ログ暗号化用 API
- `ui.*` 設定項目
- `webui.*` 設定項目
- `desktop.*` 設定項目
- `mobile.*` 設定項目
- `cloud.*` 設定項目
- `tenant.*` 設定項目
- `billing.*` 設定項目
- `nfs.*` 設定項目
- `cluster.*` 設定項目
- `distributedStorage.*` 設定項目
- `externalStorage.*` 設定項目
- `virusScan.*` 設定項目
- `logEncryption.*` 設定項目
- UI 用 JSON ファイル
- モバイル用 JSON ファイル
- クラウド用 JSON ファイル
- テナント用 JSON ファイル
- 課金用 JSON ファイル
- 外部ストレージ用 JSON ファイル
- ウイルススキャン用 JSON ファイル
- ログ暗号化用 JSON ファイル
- UI 用ディレクトリ
- モバイル用ディレクトリ
- クラウド用ディレクトリ
- テナント用ディレクトリ
- 課金用ディレクトリ
- 外部ストレージ用ディレクトリ
- ウイルススキャン用ディレクトリ
- ログ暗号化用ディレクトリ
- 外部フレームワーク
- 外部SDK
- 外部DB
- 開発リポジトリ内の実行時データ
- `.gitignore`

将来計画機能を実装対象へ昇格する場合は、実装前に `ASB-spec.md` を改訂し、以下を最低限確定する。

- 実装対象範囲
- 非対象範囲
- 既存仕様との関係
- API
- request / response
- error code
- 保存 JSON
- 設定項目
- 生成ファイル
- 生成ディレクトリ
- 外部依存の有無
- 開発リポジトリ非生成の保証
- セキュリティ境界
- テスト条件
- 移行手順

---

## 14 パフォーマンス要件

### 14.1 推奨リソース

**最小要件**
- CPU：1 core（2GHz以上推奨）
- メモリ：256MB
- ディスク：10GB（システム・ログ用）
- ネットワーク：1Mbps以上

**推奨要件**
- CPU：2 cores（2GHz以上）
- メモリ：1GB
- ディスク：50GB（システム・ログ用）
- ネットワーク：10Mbps以上

**本番運用要件**
- CPU：4 cores
- メモリ：4GB
- ディスク：100GB+（ストレージ容量による）
- ネットワーク：100Mbps以上

### 14.2 スケーリング基準

| 規模 | プロジェクト数 | ストレージ | 推奨構成 |
|-----|------------|--------|--------|
| 小規模 | < 100 | < 100GB | 単一インスタンス（最小要件） |
| 中規模 | 100-1,000 | 100GB-1TB | 単一インスタンス（推奨要件） |
| 大規模 | > 1,000 | > 1TB | 複数インスタンス + NFS/分散ストレージ |

### 14.3 パフォーマンス目標

- API レスポンスタイム：< 500ms（平均）
- ファイルアップロード：> 10Mbps
- 同時接続数：最小100、推奨1,000

---

## 15 テスト戦略

### 15.1 テスト方針

**ユニットテスト**
- 対象：各 Service の単体テスト
- カバレッジ目標：80%以上
- ツール：Go testing パッケージ

**統合テスト**
- 対象：API エンドポイント単位（全エンドポイント）
- 検証項目：
  - リクエスト/レスポンス仕様
  - トランザクション・ロールバック
  - データ整合性
- ツール：Go testing パッケージ + テスト用 HTTP クライアント

**E2E テスト**
- 対象：ワークフロー全体
- 検証シナリオ：
  - プロジェクト作成 → ファイルアップロード → 削除
  - ドメイン割り当て → SSL管理データ確認
  - バックアップ → 復旧
- ツール：bash スクリプト + curl

### 15.2 テスト対象外

以下はモック・スタブで対応：
- ACME 実通信および CA 連携（Rev.32 時点では実通信を実装対象外とし、SSL管理境界のみ検証）
- GitHub Webhook（テスト用ペイロード）
- 実際のファイルストレージ大容量テスト（テスト時は最大100MB）

### 15.3 テスト実行

```bash
# ユニットテスト
$ go test ./...

# カバレッジ
$ go test -cover ./...

# 統合テスト
$ go test -integration ./...

# E2E テスト
$ ./tests/e2e.sh
```

---

## 16 マイグレーション戦略

### 16.1 基本方針

ASB のマイグレーションは、実行時 JSON ファイルのスキーマ変更に限定する。

ASB は外部DBを使用しないため、DBマイグレーション機構、外部トランザクション機構、外部マイグレーションフレームワークを使用しない。

マイグレーションは `storage.basePath` 配下の実行時データに対してのみ行う。

開発リポジトリ内にマイグレーション作業ファイル、一時ファイル、退避ファイル、履歴ファイルを作成してはならない。

ASB の開発版バージョンは累積連番 `v0.N` とし、メジャー/マイナー/パッチの意味を持たせない。

安定版バージョン `vX.Y` は、安定版リリース番号 `X` と切り出し元の開発版 `v0.Y` を示す表示であり、互換性判定には `schemaVersion` を使用する。

### 16.2 マイグレーション対象

マイグレーション対象は以下に限定する。

| 対象 | 説明 |
|------|------|
| `config/projects.json` | Project スキーマ |
| `config/domains.json` | Domain スキーマ |
| `config/backups.json` | Backup スキーマ |
| `config/webhooks.json` | Webhook 冪等キー履歴スキーマ |
| `storage/projects/:projectId/files.json` | File メタデータスキーマ |

静的コンテンツ実体、ログファイル、証明書ファイル、ビルド済みバイナリは、Rev.32 時点のマイグレーション対象外とする。

### 16.3 schemaVersion 固定

各実行時 JSON ファイルはトップレベルに `schemaVersion` を持つ。

Rev.32 時点の `schemaVersion` は `1` とする。

例：

```json
{
  "schemaVersion": 1,
  "projects": []
}
```

`schemaVersion` が存在しない JSON ファイルは、`schemaVersion: 0` として扱う。

ASB 起動時に現在のASBが対応しない `schemaVersion` を検出した場合は起動失敗とする。

起動時に自動マイグレーションを実行してはならない。

### 16.4 実行方式

マイグレーションは ASB 本体バイナリの管理コマンドとして提供する。

外部スクリプトを正本実行方式として扱ってはならない。

管理コマンドは以下を固定する。

```bash
asb migrate --storage /var/asb --from-schema 0 --to-schema 1 --dry-run
asb migrate --storage /var/asb --from-schema 0 --to-schema 1 --apply
```

`--storage` は `storage.basePath` を指定する。

`--dry-run` は読み込み、検証、変換後データ生成、書き込み可否検証までを行い、実行時 JSON ファイルを変更してはならない。

`--apply` は実行時 JSON ファイルを更新する。

`--dry-run` と `--apply` は同時指定してはならない。

### 16.5 実行順序

`--apply` のマイグレーションは以下の順序で実行する。

1. ASB サーバープロセスが停止していることを確認する
2. `storage.basePath` が開発リポジトリ配下でないことを検証する
3. 対象 JSON ファイルの存在、構文、現在スキーマを検証する
4. 対象 JSON ファイルの読み込み権限と書き込み権限を検証する
5. `storage.basePath/backups/migrations/` 配下へ事前バックアップを作成する
6. 変換後 JSON をメモリ上に生成する
7. 変換後 JSON のスキーマを検証する
8. 対象 JSON ファイルと同一ディレクトリ内の一時ファイルへ書き込む
9. `fsync` 後に atomic rename で置き換える
10. `config/migrations.json` に完了履歴を保存する
11. 完了ログを JSON Lines で記録する

5-10 の途中で失敗した場合、成功扱いにしてはならない。

### 16.6 migrationHistory 保存形式

マイグレーション履歴は `config/migrations.json` に保存する。

`config/migrations.json` は起動時必須 JSON ファイルではなく、初回マイグレーション実行時に `storage.basePath/config/` 配下へ作成できる。

作成場所は実行時データ領域に限定し、開発リポジトリ内へ作成してはならない。

保存形式は以下とする。

```json
{
  "schemaVersion": 1,
  "migrations": [
    {
      "id": "string(UUID)",
      "fromSchema": 0,
      "toSchema": 1,
      "startedAt": "2026-09-08T00:00:00Z",
      "finishedAt": "2026-09-08T00:00:00Z",
      "status": "applied",
      "backupPath": "backups/migrations/migration-id.tar.gz"
    }
  ]
}
```

`status` は `applied` または `failed` のみ許可する。

### 16.7 ロールバック

マイグレーション失敗時は、事前バックアップが作成済みであればバックアップから復元する。

ロールバックは `storage.basePath` 配下の実行時データのみを対象とする。

ロールバックに失敗した場合は、標準エラー、エラーログ、`config/migrations.json` に `failed` として記録する。

ロールバック失敗時に成功扱いとしてはならない。

### 16.8 禁止事項

Rev.32 時点では以下を禁止する。

- 起動時の自動マイグレーション
- 開発リポジトリ内でのマイグレーション作業ファイル作成
- 外部DBマイグレーション
- 外部トランザクション機構
- 外部マイグレーションフレームワーク
- `.gitignore` を必要とする移行生成物設計

### 16.9 テスト固定項目

マイグレーション実装では以下のテストを必須とする。

- `schemaVersion` なしを `0` として扱うテスト
- 未対応 `schemaVersion` の起動失敗テスト
- `--dry-run` が実行時 JSON ファイルを変更しないテスト
- `--apply` が対象 JSON ファイルを atomic rename で更新するテスト
- 事前バックアップ作成テスト
- 途中失敗時の成功禁止テスト
- ロールバック成功テスト
- ロールバック失敗時の `failed` 記録テスト
- 開発リポジトリ内に移行作業ファイルを作成しないテスト

---

## 17 変更履歴

| バージョン | 日付 | 内容 |
|-----------|------|------|
| Rev.32 | 2026-09-08 | 実装ファイル構成、package 内ファイル役割、エラーコード固定表、テストファイル配置、フェーズ完了判定を追加し、仕様を実装直前の契約粒度へ具体化 |
| Rev.31 | 2026-09-08 | 実装フェーズ単位の開発版バージョン管理を追加し、IMPLEMENTATION_TASKS.md を P0/v0.1 から P9/v0.10 までの優先度別フェーズ構成へ再編する方針を固定 |
| Rev.30 | 2026-09-08 | 将来計画、保留事項、検討・調査中事項を Rev.30 時点の実装対象外として固定し、GUI/Web UI/モバイル/クラウド/複数インスタンス/外部ストレージ/ログ暗号化/ウイルススキャン等のAPI・設定・JSON・ディレクトリ・外部依存追加禁止を実装レベルで固定 |
| Rev.29 | 2026-09-08 | ASB互換目標を将来到達目標として実装対象から分離し、XServer Static 相当仕様との関係、含む対象、含めない対象、互換目標を理由にした未確定機能実装禁止を実装レベルで固定 |
| Rev.28 | 2026-09-08 | ACME 実通信を ASB 本体に実装しない方針、SSL管理を証明書メタデータ・配置・検証に限定し、ACME/CA/challenge/自動更新関連データ非生成を実装レベルで固定 |
| Rev.27 | 2026-09-08 | SDK 本体は実装対象外のまま、将来 SDK 通信を ASB 管理 HTTP JSON API と同一規格に固定し、SDK 専用プロトコル・エンドポイント・実行時データ非生成を実装レベルで固定 |
| Rev.26 | 2026-09-08 | Brotli 圧縮を ASB 本体に実装しない方針、Gzip 標準限定、Accept-Encoding br 非対応、Brotli 関連ファイル・設定・キャッシュ非生成を実装レベルで固定 |
| Rev.25 | 2026-09-08 | Rate limiting を ASB 本体に実装しない方針、Rate limiting 関連 middleware・設定・JSON・実行時カウンタ非生成、外部運用境界を実装レベルで固定 |
| Rev.24 | 2026-09-08 | APIキー管理とユーザー認証を実装しない方針、認証ヘッダー非使用、認証関連ファイル非生成、外部運用境界を実装レベルで固定 |
| Rev.23 | 2026-09-08 | ログ保存先、JSON Lines項目、requestId、ログAPI、stdout/stderr、ローテーション、開発リポジトリ非生成を実装レベルで固定 |
| Rev.22 | 2026-09-08 | 静的配信のHost解決、パス正規化、HEAD、キャッシュヘッダー、304、Range非対応、開発リポジトリ非生成を実装レベルで固定 |
| Rev.21 | 2026-09-08 | バックアップ保存先、tar.gz構成、checksum、復旧退避先、失敗時復元、開発リポジトリ非生成を実装レベルで固定 |
| Rev.20 | 2026-09-08 | GitHub Webhook署名検証、ローカルcheckoutデプロイ、失敗時リトライ禁止、Webhook処理順序を実装レベルで固定 |
| Rev.19 | 2026-09-08 | 安定版リリース判定、GitHub Releases配布、checksum、install/update、systemd仕様を実装レベルで固定 |
| Rev.18 | 2026-09-08 | マイグレーション戦略をASBのJSONファイルベース実行時データ移行契約へ全面置換 |
| Rev.17 | 2026-09-08 | package公開interface、Repository/Storage責務、ファイル操作、Project削除、Backup/Restore、複数JSON更新失敗時契約を固定 |
| Rev.16 | 2026-09-08 | API成功/エラーレスポンス、保存JSONスキーマ、起動時検証出力、ログJSON Linesを固定 |
| Rev.15 | 2026-09-08 | API、設定値、JSONファイル、静的配信、Webhook、実装順序を実装単位で固定 |
| Rev.14 | 2026-09-08 | 実装契約を追加し、パッケージ境界、HTTP契約、JSON保存、起動時検証、Handler/Service/Entity責務、副作用、保留機能禁止を具体化 |
| Rev.13 | 2026-09-08 | SDK/API、APIキー、Rate limiting、SSL/ACME、CA選定の確定範囲と保留範囲を整理し、機能仕様見出しを補完 |
| Rev.12 | 2026-09-08 | XServer Static 相当仕様を ASB互換目標へ名称変更し、SSL/ACME を ASB互換目標に含める方針へ整理 |
| Rev.11 | 2026-09-08 | XServer Static 相当の機能目標と ASB 独自仕様を追加し、セルフホスト、JSONファイルベース、実行時データ配置、API中心設計、段階実装方針を整理 |
| Rev.10 | 2026-09-08 | 実装詳細仕様を追加し、API共通規約、起動時検証、実行時データ配置、JSON更新、各機能の実装境界とテスト受け入れ条件を具体化 |
| Rev.9 | 2026-09-08 | 起動時の実行時データ自動生成を廃止し、設定済み保存先の存在確認・権限検証へ変更 |
| Rev.8 | 2025-01-15 | ライセンス・バージョンポリシー詳細化（AEB準拠、ドキュメント/開発版/安定版バージョン管理体系、互換性対応表） |
| Rev.7 | 2025-01-15 | ポリシーセクション拡張（10種類へAEB準拠、4ドメイン別ポリシー・SDK/API・テスト・品質・ライセンス・将来計画管理ポリシー追加） |
| Rev.6 | 2025-01-15 | 実装レベル仕様確定（API詳細化、バリデーション、セキュリティ詳細化、パフォーマンス・テスト・マイグレーション戦略追加） |
| Rev.5 | 2025-01-15 | インストール・アップデート自動化スクリプト仕様追加（install.sh、update.sh、asb.service） |
| Rev.4 | 2025-01-15 | インストール・アップデートセクション追加（単一バイナリでのSSH導入） |
| Rev.3 | 2025-01-15 | ビルド対応アーキテクチャ拡張（Linux amd64 + arm64） |
| Rev.2 | 2025-01-15 | 仕様改良（データモデル説明追加、JSON構造拡張、エラーハンドリング定義、Phase詳細化） |
| Rev.1 | 2025-01-15 | ゼロベース仕様策定（責務駆動設計アーキテクチャ採用） |
