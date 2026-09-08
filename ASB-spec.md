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
| 本書バージョン | Rev.16 |

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

**ASB互換目標**

- 静的コンテンツ専用ホスティングとして動作する
- HTML、CSS、JavaScript、画像等の静的ファイルを配信する
- プロジェクト単位で公開対象を管理する
- 独自ドメインをプロジェクトへ割り当てる
- SSL証明書の管理に対応する
- ACME による証明書取得・更新を目標に含める
- HTTP/2 対応を目標とする
- GitHub 連携による自動デプロイを提供する
- ファイルアップロードおよびフォルダ階層を保持したファイル管理を提供する
- SSL更新状態、デプロイ状態、ログを確認できる

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
- SDK、CA、ACME 実通信など詳細未確定の機能は、仕様確定後に実装対象へ昇格する

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
| SSL 証明書 | SSL証明書管理境界（ACME 実通信・CA選定は詳細仕様確定後） |
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
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -o asb-linux-amd64 ./main.go

# Linux arm64
CGO_ENABLED=0 GOOS=linux GOARCH=arm64 go build -o asb-linux-arm64 ./main.go
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
- 証明書生成・更新失敗時のログ記録
- ACME 実通信は詳細仕様確定後に実装

### 7.4 ファイル管理

**ファイルアップロード**
- 最大容量：1GB/プロジェクト（設定可能）
- 形式：制限なし（HTML, CSS, JavaScript, 画像等）
- 圧縮：Gzip による自動圧縮
- Brotli は外部ライブラリ例外が仕様に追加されるまで実装対象外

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
| レスポンス | `{ "status": "received" }` |
| ステータス | 200 OK, 400 Bad Request |

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
  "path": "string"
}
```

**フィールド説明**
- `id`：バックアップの一意識別子（UUID）
- `projectId`：バックアップ対象プロジェクトID
- `createdAt`：バックアップ作成日時（ISO 8601形式）
- `size`：バックアップサイズ（バイト）
- `path`：ストレージ内のバックアップファイルパス

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
      "path": "/storage/backups/backup-001.tar.gz"
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
      "receivedAt": "2026-09-08T00:00:00Z"
    }
  ]
}
```

**フィールド説明**
- `key`：Webhook 冪等キー（branch + ":" + after）
- `branch`：GitHub Push イベントの対象ブランチ
- `after`：GitHub Push イベントの after commit hash
- `receivedAt`：Webhook 処理成功日時（ISO 8601形式）

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

Webhook 処理失敗は、システムログへ記録し、自動リトライスケジュールの仕様を備える。

### 11.3 Data Domain ポリシー

Data Domain は、バックアップ・ストレージの責務を担う。

**責務優先順位**
1. データ整合性
2. データ保全
3. 可用性

データ保全をData Domain の最優先ポリシーとする。バックアップはtar.gz形式で実施し、復元前に整合性検査（SHA-256ハッシュ検証）を実施する。

**バックアップ・復旧方針**
- バックアップ取得後はハッシュ検証を実施
- バックアップ保存先は別個の障害領域へ配置
- 復旧は隔離環境で検証後、運用者承認を経て実施
- バックアップ・復旧・検証失敗・復旧操作は監査ログへ記録

### 11.4 System Domain ポリシー

System Domain は、監視・ログ管理の責務を担う。

**構造化ログ**
- JSON形式で出力
- ログレベル：DEBUG / INFO / WARN / ERROR
- 本番デフォルトレベル：INFO
- 標準構成では `storage.basePath/logs/` 配下へ JSON Lines として保存する
- 標準出力（stdout）出力は運用方式として別途検討する

**必須フィールド**
- `time`：RFC 3339（ナノ秒精度）
- `level`：ログレベル
- `msg`：メッセージ
- `domain`：機能ドメイン（`mgmt` / `delivery` / `data` / `system`）
- `timestamp`：操作日時

**監視項目**
- CPU使用率、メモリ使用率、ディスク使用率
- 接続数・リクエスト数
- バックアップ状態、証明書有効期限

### 11.5 API/SDK ポリシー

ASB はヘッドレスアーキテクチャを採用し、UI層に依存しない。

Rev.16 時点の確定対象は、ASB 本体が提供する HTTP JSON API である。

SDK は実装対象外とし、通信仕様および配布方針が確定した後に実装対象へ昇格する。

**API 設計原則**
- HTTP + JSON を使用する
- HTTP/2 対応は ASB互換目標として扱い、実装詳細は別途確定する
- デフォルト接続境界は `localhost:3000` とする
- 認証、APIキー管理、SDK通信規格は保留事項として扱う

**エラーレスポンス形式**
```json
{
  "error": "string (エラーメッセージ)",
  "code": "string (エラーコード)",
  "timestamp": "string (ISO 8601)",
  "httpStatus": 400
}
```

**API 互換性**
- マイナーバージョン：完全互換
- メジャーバージョン：後方互換性なし、マイグレーション仕様を提供

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
- リモートアクセス：リバースプロキシ（nginx等）経由での公開推奨
- SSL/TLS：本番環境では必須（リバースプロキシで対応）

**レート制限**
- Rev.16 時点では実装対象外とし、保留事項として扱う

**タイムアウト**
- リクエスト読み込み：30秒
- レスポンス書き込み：60秒
- ログ保持時間：7日間

**ファイルアップロード**
- 最大サイズ：1GB
- 形式制限：なし
- ウイルススキャン：ASB では提供しない（別途対応推奨）
- 置換許可：同名ファイル上書き可能

**ログ出力**
- アクセスログ：全HTTP リクエスト（JSON形式）
- エラーログ：エラー・例外・警告
- ログファイル暗号化：将来計画

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

- 表記は`v0.N`（Nは1から始まる連番）。先頭の0は固定とする（将来1への切り替えを検討する余地は残すが、基準は未策定。切り替えてもNはリセットしない）
- 本仕様書の変更を伴うすべての変更に付与する
- Nは変更のたびに1ずつ増加する。桁揃えは行わない（v0.1, v0.2, … v0.9, v0.10, …）
- いかなる理由があってもリセット（巻き戻し・1からの数え直し）はしない

##### 11.9.2.3 安定版バージョン（vX.Y）

リリース（外部への公開・配布）は安定版のみを対象とする。開発版バージョンの全エントリがリリースされるわけではなく、安定していると判断された時点の開発版を選んで安定版として切り出す。「安定している」の具体的な判断基準は未策定であり、別途策定するリリースポリシーで定める。

- 表記は`vX.Y`（例: v1.9, v3.20, v12.35）
- X = 安定版リリースの通し番号（1件目を1、2件目を2、…）。メジャー/マイナーのような重大度の意味は持たない
- Y = そのリリースを切り出した時点の開発版バージョンのN（例: 開発版v0.35を3件目の安定版として切り出した場合、v3.35）
- X・Yともにいかなる理由でもリセットしない
- 同一の安定版リリースの中でX・Yが指す時点がずれることはない（Yは常にそのリリース時点の開発版Nと一致する）
- 現時点ではまだ安定版リリースを1件も出していない

##### 11.9.2.4 互換性対応表

**現行互換性対応表**

|ASB本体|Go|ファイルストレージ形式|
|---|---|---|
|v0.1以降|1.21|JSON ファイル形式|

- ASB本体バージョンとファイルストレージ形式の組み合わせ変更は仕様改訂を必要とする。
- Goバージョンの更新は§11.6の手順を経て本表を更新する。
- 互換性検証の実施記録はログへ保存する。

### 11.10 将来計画管理ポリシー

- 段階的対応対象：GUI、モバイルアプリ、クラウドサービス化、複数インスタンス対応
- 保留事項：ユーザー認証、マルチテナント対応
- 検討・調査中事項：複数インスタンス時の NFS/分散ストレージ選定

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

#### 11.11.3 インストール手順

**前提**
- Linux amd64 または arm64 環境
- SSH アクセス可能

**手順**

```bash
# 1. バイナリをダウンロード
$ wget https://releases.example.com/asb-linux-amd64

# 2. 実行権限を付与
$ chmod +x asb-linux-amd64

# 3. 起動テスト
$ ./asb-linux-amd64 --version

# 4. バックグラウンド起動
$ ./asb-linux-amd64 &

# または systemd での管理（オプション）
$ sudo mv asb-linux-amd64 /usr/local/bin/asb
$ sudo systemctl enable asb
$ sudo systemctl start asb
```

#### 11.11.4 アップデート手順

**既存バイナリの置き換え**

```bash
# 1. 新しいバイナリをダウンロード
$ wget https://releases.example.com/asb-linux-amd64-v1.1

# 2. 既存バイナリを停止
$ pkill asb
# または
$ sudo systemctl stop asb

# 3. バイナリを置き換え
$ mv asb-linux-amd64-v1.1 asb-linux-amd64
$ chmod +x asb-linux-amd64

# 4. 起動
$ ./asb-linux-amd64 &
# または
$ sudo systemctl start asb
```

#### 11.11.5 ダウンロード・リリース管理

- リリース形式：`asb-linux-{architecture}-v{version}`
  - 例：`asb-linux-amd64-v1.0`, `asb-linux-arm64-v1.0`
- リリースページ：GitHub Releases 等で公開予定
- チェックサム検証：SHA-256 ハッシュを提供（整合性確認用）

#### 11.11.6 インストール・アップデート自動化

ASB の初回インストールとアップデートを自動化するためのスクリプトを提供する。

**提供ファイル**

| ファイル | 用途 | 説明 |
|---------|------|------|
| install.sh | 初回インストール | バイナリダウンロード、権限設定、systemd登録を自動実行 |
| update.sh | アップデート | 最新バイナリダウンロード、サービス再起動を自動実行 |
| asb.service | systemd ユニット | systemctl での自動起動・停止・再起動・ログ管理 |

**install.sh 実行例**

```bash
$ chmod +x install.sh
$ ./install.sh
# または
$ sudo ./install.sh  # systemd 登録時は sudo 必要
```

実行内容：
1. 環境（amd64/arm64）を自動判定
2. 最新バイナリをダウンロード
3. 実行権限を付与
4. `/usr/local/bin/asb` にコピー
5. asb.service を systemd に登録
6. サービス自動起動を有効化
7. サービス起動

**update.sh 実行例**

```bash
$ chmod +x update.sh
$ ./update.sh
```

実行内容：
1. 最新バージョンをチェック
2. アップデート必要な場合：
   - 新しいバイナリをダウンロード
   - サービスを停止
   - バイナリを置き換え
   - サービスを再起動

**asb.service（systemd ユニット）**

```ini
[Unit]
Description=Adlaire-Static-Base HTTP Server
After=network.target
Wants=network-online.target

[Service]
Type=simple
User=asb
Group=asb
ExecStart=/usr/local/bin/asb
Restart=always
RestartSec=5
StandardOutput=journal
StandardError=journal
SyslogIdentifier=asb

[Install]
WantedBy=multi-user.target
```

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

| 対象 | 位置づけ | 対応段階 | 備考 |
|-----|---------|--------|------|
| GUI（デスクトップアプリ、Web UI） | 計画 | Phase 2 | バックエンド仕様確定後に検討 |
| モバイルアプリ（iOS/Android） | 計画 | Phase 3 | GUI の後、モバイル化を検討 |
| クラウドサービス化 | 検討 | Phase 4 | セルフホスト版の仕様確定後に検討 |
| 複数インスタンス対応 | 保留 | 将来 | NFS/ストレージ共有の課題解消待ち |

### 12.1.1 Phase 別詳細

**Phase 2：GUI 実装**
- Desktop App（Electron等）または Web UI（React等）の検討
- 既存 HTTP API を利用した独立実装
- ASB バックエンドと別リポジトリ化想定

**Phase 3：モバイルアプリ**
- iOS/Android ネイティブアプリの検討
- 既存 HTTP API を利用した実装

**Phase 4：クラウドサービス化**
- マルチテナント対応の検討
- 認証・認可システムの実装
- ユーザー管理機能の追加

### 12.2 保留事項

- ユーザー認証（現在：不要） 
- マルチテナント対応
- API キー管理
- SDK通信規格
- Rate limiting
- 外部ストレージサービス統合（AWS S3 等）

### 12.3 検討・調査中事項

- 複数インスタンス時の NFS/分散ストレージ選定
- ACME 実通信、CA選定、ワイルドカード証明書対応
- バックアップのクラウドストレージ連携（オプション）

---

## 13 実装詳細仕様

### 13.1 実装対象の基準

Rev.16 時点の実装対象は、ASB のセルフホスト型静的コンテンツ配信ホスティングに必要なバックエンド機能に限定する。

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
- Brotli 圧縮は Go 標準ライブラリに存在しないため、外部ライブラリ例外が仕様に追加されるまで実装対象外とする

### 13.8 ドメイン管理詳細

- ドメインは小文字へ正規化して保存する
- ドメインは RFC 1035 に準拠し、各ラベルは 1〜63 文字とする
- ドメイン全体は 253 文字以下とする
- ドメインは最大3階層までとする
- 既に別プロジェクトへ割り当て済みの場合は `ERR_DOMAIN_ALREADY_ASSIGNED` を返す
- プロジェクト削除時は関連ドメインを削除する

### 13.9 SSL 管理詳細

Rev.16 時点では、SSL 管理は管理境界とデータモデルを実装対象とし、ACME は ASB互換目標に含める。ACME 実通信、CA選定、ワイルドカード証明書対応は詳細仕様確定後に実装対象へ昇格する。

実装対象：

- SSL 証明書IDの管理
- 証明書保存先 `certs/` の存在確認
- 証明書有効期限の監視モデル
- SSL 証明書生成失敗時の `ERR_SSL_CERT_GENERATION_FAILED`

実装保留：

- ACME アカウント登録
- HTTP-01 / DNS-01 challenge
- 証明書発行リクエスト
- 証明書自動更新の実通信

### 13.10 Webhook 詳細

GitHub Webhook は Push イベントのみを対象とする。

- `POST /api/webhook/github` は GitHub Webhook ペイロードを受け取る
- 対象ブランチは設定ファイルで指定する
- 対象外ブランチのイベントは成功扱いで無視する
- ペイロード形式が不正な場合は `400 Bad Request` を返す
- デプロイ処理に失敗した場合は `ERR_WEBHOOK_PROCESSING_FAILED` を返す
- 同一 GitHub Push イベントを重複受信した場合は、同一 commit hash と対象ブランチの組み合わせを冪等キーとして扱い、二重デプロイを避ける
- 冪等キーの保存方式は JSON ファイルベースとし、保存先は `storage.basePath` 配下に限定する
- Webhook 署名検証の必須化は詳細仕様確定後に実装する

### 13.11 バックアップ・復旧詳細

**バックアップ**

- バックアップ対象は `config/` と `storage/` とする
- バックアップ形式は tar.gz とする
- バックアップ作成後に SHA-256 ハッシュを計算する
- バックアップ履歴は `backups.json` に記録する
- バックアップ失敗時は成功履歴を作成してはならない

**復旧**

- 復旧前にバックアップファイルの存在と SHA-256 ハッシュを検証する
- 既存データを退避してから復旧する
- 復旧後に JSON ファイル構文と必須フィールドを検証する
- 復旧失敗時は `ERR_BACKUP_RESTORE_FAILED` を返す

### 13.12 監視・ログ詳細

**監視**

- `/api/monitoring/stats` は CPU、メモリ、ディスク、接続数、リクエスト数を返す
- OS 依存で取得できない項目は `null` とし、レスポンス自体は成功扱いとする

**ログ**

- アクセスログは全 HTTP リクエストを対象とする
- エラーログは WARN 以上を対象とする
- ログ形式は JSON Lines とする
- ログ API は `limit` と `offset` を受け付ける
- `limit` のデフォルトは 100、最大は 1000 とする
- ログには機密値を含めてはならない

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

本節は Rev.16 時点の実装契約である。実装者は本節に反する判断をコード側で独自に行ってはならない。

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

`main.go` は設定読み込み、依存関係生成、HTTP サーバー起動、graceful shutdown のみを行う。

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
| Backup 作成 | config/storage | backup tar.gz、backups JSON | なし |
| Backup 復旧 | backup tar.gz、config/storage | config/storage | 復旧対象の既存データ退避 |
| Log API | logs | なし | なし |

複数ファイルにまたがる操作では、途中失敗時に成功レスポンスを返してはならない。

途中失敗時はエラーログを記録し、可能な範囲で整合性検証を行う。

#### 13.14.9 保留機能の実装禁止契約

Rev.16 時点では以下を実装してはならない。

- SDK
- APIキー管理
- Rate limiting
- ACME 実通信
- CA 選定固定
- Webhook 署名検証の必須化
- 外部DB
- 外部ストレージ連携
- `.gitignore` を必要とする生成物設計

上記を実装する場合は、先に `ASB-spec.md` を改訂し、確定仕様として昇格させる。

### 13.15 実装詳細固定仕様

本節は Rev.16 時点で実装時に固定する詳細仕様である。

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
| Backup 復旧 | POST | `/api/backups/restore/:id` | なし | なし | 200 `{status,backupId,restoredAt}` | 404, 500 |
| Monitoring | GET | `/api/monitoring/stats` | なし | なし | 200 Monitoring | 500 |
| Access log | GET | `/api/logs/access` | なし | `limit`,`offset` | 200 `{logs:[]}` | 400, 500 |
| Error log | GET | `/api/logs/error` | なし | `limit`,`offset` | 200 `{logs:[]}` | 400, 500 |
| GitHub Webhook | POST | `/api/webhook/github` | GitHub Push JSON | なし | 200 `{status}` | 400, 500 |

`:id` は UUID 形式の文字列のみ許可する。

`:domain` は URL decode 後に小文字正規化し、ドメイン検証を行う。

`:name` は URL decode 後にファイル名検証を行い、パス区切り文字を含む値は拒否する。

`limit` は未指定時 `100`、最小 `1`、最大 `1000` とする。

`offset` は未指定時 `0`、最小 `0` とする。

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

設定ファイルに未知フィールドがある場合は起動失敗とする。

任意項目が未指定の場合はデフォルト値を適用する。

#### 13.15.3 JSON ファイル固定仕様

| ファイル | 空状態 | 必須トップレベル | 更新主体 | 備考 |
|---------|--------|------------------|----------|------|
| `config/projects.json` | `{"projects":[]}` | `projects` | Project Service | Project 配列を `createdAt` 昇順で保存 |
| `config/domains.json` | `{"domains":[]}` | `domains` | Domain Service | `domain` は小文字で保存 |
| `config/backups.json` | `{"backups":[]}` | `backups` | Backup Service | `createdAt` 降順で保存 |
| `storage/projects/:projectId/files.json` | `{"files":[]}` | `files` | File Service | `name` 昇順で保存 |
| `config/webhooks.json` | `{"events":[]}` | `events` | Webhook Service | 冪等キー履歴を保存 |

上記 JSON ファイルは起動時に存在していなければならない。

ASB は上記 JSON ファイルを起動時に作成しない。

`config/webhooks.json` は Webhook 冪等性管理に使用する実行時 JSON ファイルであり、外部DBを使用しない。

#### 13.15.4 静的配信固定仕様

静的配信は API パスに一致しない GET または HEAD のみ対象とする。

対象プロジェクトの決定は Host ヘッダーのドメイン割り当てにより行う。

Host ヘッダーが割り当て済みドメインに一致しない場合は `404 Not Found` とする。

リクエストパスが `/` の場合は `index.html` を探索する。

ファイル探索は `storage/projects/:projectId/contents/` 配下に限定する。

`..`、絶対パス、URL decode 後のパス区切り脱出を含むリクエストは `404 Not Found` とする。

存在しない静的ファイルは `404 Not Found` とする。

Content-Type は Go 標準ライブラリで判定し、判定不能な場合は `application/octet-stream` とする。

Gzip 圧縮済みファイルを返す場合は `Content-Encoding: gzip` を設定する。

#### 13.15.5 Webhook 固定仕様

GitHub Webhook は `push` event のみ処理する。

`X-GitHub-Event` が `push` 以外の場合は `200 OK` とし、`{"status":"ignored"}` を返す。

対象ブランチは設定ファイルで指定する。

対象外ブランチの場合は `200 OK` とし、`{"status":"ignored"}` を返す。

冪等キーは `branch + ":" + after` とする。

同一冪等キーが `config/webhooks.json` に存在する場合は `200 OK` とし、`{"status":"duplicate"}` を返す。

Webhook 処理成功後に冪等キーを `config/webhooks.json` へ保存する。

Webhook 署名検証は Rev.16 時点では必須化しない。

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

本節は Rev.16 時点で API、JSON保存、ログ、起動時検証の入出力を固定する仕様である。

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
| GitHub Webhook 処理 | 200 | `{"status":"received","branch":string,"after":string,"processedAt":string}` |
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
  "sha256": "string"
}
```

`config/webhooks.json` の `events[]` は以下の形式とする。

```json
{
  "key": "main:abcdef1234567890",
  "branch": "main",
  "after": "abcdef1234567890",
  "receivedAt": "2026-09-08T00:00:00Z"
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

アクセスログは1リクエストにつき1行の JSON Lines とし、以下のフィールドを固定する。

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

エラーログは WARN 以上を対象とし、以下のフィールドを固定する。

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

ログには内部ファイルパス、スタックトレース、環境変数、シークレットを含めてはならない。

#### 13.16.8 テスト固定項目

Rev.16 の実装では、以下のテストを必須とする。

- 全API成功レスポンスの固定JSONキー検証
- 全APIエラーレスポンスの固定JSONキー検証
- 保存JSONの未知フィールド拒否
- 保存JSONの相対パス保存検証
- 保存JSONのソート順検証
- 起動時検証の順序、終了コード、標準エラー形式検証
- アクセスログとエラーログのJSON Linesフィールド検証

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
- ACME 実通信および CA 連携（Rev.16 時点では実通信を実装対象外とし、SSL管理境界のみ検証）
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

### 16.1 バージョン互換性

**マイナーバージョン（v1.0 → v1.1）**
- JSON スキーマ変更なし
- 既存データとの互換性保証
- 自動マイグレーション不要
- 互換性：完全互換

**メジャーバージョン（v1.x → v2.0）**
- スキーマ変更の可能性
- マイグレーションスクリプト提供予定
- バックアップから復元可能
- 互換性：後方互換性なし

### 16.2 マイグレーション手順

**準備**

```bash
# 1. バックアップ作成
$ asb-backup-create --output backup-v1.tar.gz

# 2. バックアップ整合性確認
$ asb-backup-verify backup-v1.tar.gz
```

**実行**

```bash
# 3. 新バイナリ起動（旧バージョンと並行動作テスト）
$ ./asb-linux-amd64-v2.0 --dry-run

# 4. 旧バイナリ停止
$ sudo systemctl stop asb

# 5. バイナリ置き換え
$ cp asb-linux-amd64-v2.0 /usr/local/bin/asb
$ chmod +x /usr/local/bin/asb

# 6. マイグレーション実行（必要な場合）
$ asb-migrate --from v1.0 --to v2.0

# 7. サービス起動
$ sudo systemctl start asb
```

**検証**

```bash
# 8. ログ確認
$ sudo journalctl -u asb -n 50

# 9. API 動作確認
$ curl http://localhost:3000/api/projects

# 10. データ整合性確認
$ asb-verify-data
```

**ロールバック（必要な場合）**

```bash
# 11. バイナリ置き換え（旧）
$ cp /usr/local/bin/asb-v1.0 /usr/local/bin/asb

# 12. サービス再起動
$ sudo systemctl restart asb

# 13. バックアップから復元（必要な場合）
$ asb-backup-restore backup-v1.tar.gz
```

### 16.3 データ互換性の考慮

- JSON フォーマット：バージョン情報を付与（将来の互換性判定用）
- スキーマ変更時：変更前後のスキーマを記録
- マイグレーション情報：実行したマイグレーション履歴をログ記録

---

## 17 変更履歴

| バージョン | 日付 | 内容 |
|-----------|------|------|
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
