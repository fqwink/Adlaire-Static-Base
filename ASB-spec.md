# セルフホスト型静的コンテンツ配信ホスティングシステム仕様書

## 1 本書の位置づけ

本書は、セルフホスト型静的コンテンツ配信ホスティングシステム「Adlaire-Static-Base（ASB）」の仕様正本である。

本システムはゼロベースで設計され、完全な内製化を原則とする。本書には、ASB の確定仕様、将来計画、保留事項を記載する。

### 1.1 機能一覧

ASB の機能は、Project、Domain と同じ粒度の通常機能名として横並びに管理する。

| 機能 | 主要内容 | Rev.90 時点の状態 |
|-----|---------|------------------|
| Project | プロジェクト作成・削除・情報取得 | 実装対象 |
| Domain | DNS ドメイン割り当て・管理 | 実装対象 |
| SSL | 無料独自SSL・SSL証明書管理 | 実装対象 |
| File | ファイルアップロード・削除・圧縮 | 実装対象 |
| GitHub Webhook | GitHub自動デプロイ処理 | 実装対象 |
| Backup | データ保全・復旧手順 | 実装対象 |
| Log | システム監視・アクセスログ・エラーログ | 実装対象 |
| Content Pipeline | Markdown / MDX / JSON、JSON Front Matter、content metadata、schema validation | 将来計画 |
| Site Routing | 公開サイト route 検出、route path、404、公開 route 境界 | 将来計画 |
| Site Rendering | SSG、SSR、Hybrid Rendering、prerender 制御 | 将来計画 |
| Site Output | Blog、Docs、Sitemap、SEO、i18n、Ad Slot 等の公開サイト出力 | 将来計画 |
| Blog | 記事管理、記事一覧、pagination、draft 管理 | 将来計画 |
| Docs | ドキュメントサイト、階層、前後リンク、sidebar metadata | 将来計画 |
| Sitemap | `sitemap.xml`、route 出力、hreflang、lastmod | 将来計画 |
| Ad Slot | 広告枠定義、表示条件、外部広告SDK非内蔵 | 将来計画 |

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
| 提供形態 | HTTPS サーバー（単一バイナリ） |
| データ保存 | JSON ファイルベース（外部DB不使用） |
| ライセンス | クローズドライセンス |
| 本書バージョン | Rev.90 |

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
| SSL管理 | 無料独自SSL・SSL証明書管理 |
| ファイル管理 | ファイルアップロード・削除・圧縮 |
| Webhook 処理 | GitHub との連携・自動デプロイ |
| バックアップ・復旧 | データ保全・復旧手順 |
| 監視・ログ | システム監視・アクセスログ・エラーログ |

**責任範囲外（担わない責務）**

| 対象 | 理由 |
|-----|------|
| GUI・UI | ASB 本体はヘッドレスとし、ASB Web UI は本体外の内製クライアントとして扱う |
| ユーザー管理 | 単一システム管理者のみを扱い、複数ユーザー管理は将来計画とする |
| 組織・チーム管理 | 単一システム管理者想定 |
| FTP / FTPS / SFTP | ファイル操作面は HTTPS JSON API、multipart upload、GitHub Webhook デプロイに限定 |

### 3.2 ASB互換目標と ASB 独自仕様

ASB は、静的コンテンツ専用ホスティングとして XServer Static 等の利用体験を参考基準とし、ASB互換目標として定義する。

本節の「ASB互換目標」とは、外部サービスとの完全互換ではなく、セルフホスト環境で同種の運用体験を提供するための機能目標を指す。

XServer Static 等の外部サービスは、利用体験を整理するための参考基準であり、仕様入力元、実装入力元、API互換元、設定互換元、保存形式互換元として扱ってはならない。

ASB互換目標における「参考」「相当」「目標」は、実装対象、外部依存、API、設定項目、JSON保存形式、生成ファイル、生成ディレクトリを増やす根拠ではない。

ASB互換目標は将来の到達目標であり、個別の確定仕様へ昇格した項目のみ実装対象とする。

Rev.90 時点では、無料独自SSLのみを XServer Static 互換目標から確定仕様へ昇格する。

ASB互換目標に含まれることは、未昇格機能を実装してよい根拠にならない。

ASB互換目標に含まれる機能を実装対象へ昇格する場合は、事前に `ASB-spec.md` を改訂し、実装範囲、入出力、保存形式、副作用、生成物、テスト条件を確定しなければならない。

**ASB互換目標**

- 静的コンテンツ専用ホスティングとして動作する
- HTML、CSS、JavaScript、画像等の静的ファイルを配信する
- プロジェクト単位で公開対象を管理する
- 独自ドメインをプロジェクトへ割り当てる
- 無料独自SSLに対応する
- 無料独自SSLの証明書取得と更新を自動化する
- HTTP/2 対応を目標とする
- GitHub 連携による自動デプロイを提供する
- ファイルアップロードおよびフォルダ階層を保持したファイル管理を提供する
- SSL更新状態、デプロイ状態、ログを確認できる

**Rev.90 時点で ASB互換目標に含めないもの**

- 外部サービスとのAPI完全互換
- 外部サービスの管理画面互換
- 外部サービスの課金、契約、アカウント管理互換
- 外部サービスのDNS管理機能互換
- 外部サービスのCDN機能完全互換
- 外部サービスのFTP / FTPS / SFTP等の転送プロトコル互換
- 外部サービスの SLA / サポート体制互換
- 外部サービス固有の内部実装再現

**XServer Static互換機能セット**

Rev.90 時点の ASB は、XServer Static 互換性を以下の利用者向け機能表面に限定する。

| 機能 | ASBでの扱い |
|-----|------------|
| 静的コンテンツ配信 | 実装対象。HTML、CSS、JavaScript、画像等の静的ファイルを配信する。 |
| `index.html` 配信 | 実装対象。ディレクトリパスへのアクセス時に `index.html` を配信する。 |
| 404 応答 | 実装対象。未存在ファイル、未割当Domain、不正パスは仕様に従って `404 Not Found` を返す。 |
| MIME type | 実装対象。Go標準ライブラリを基本に Content-Type を決定する。 |
| Gzip | 実装対象。条件を満たす静的ファイルをGzip応答する。 |
| 独自ドメイン | 実装対象。Domain を Project へ割り当てる。 |
| サブドメイン | 実装対象。通常Domainとして個別に登録・検証する。 |
| 無料独自SSL | 実装対象。Let’s Encrypt ACME v2 と HTTP-01 により証明書取得・更新を自動化する。 |
| GitHub 自動デプロイ | 実装対象。GitHub Push Webhook を受け、指定branchの静的ファイルを反映する。 |
| ファイル管理 | 実装対象。HTTPS JSON API、`multipart/form-data` upload、GitHub Webhook デプロイで扱う。 |
| ログ・状態確認 | 実装対象。アクセスログ、エラーログ、デプロイ状態、SSL状態、ストレージ使用量を確認可能にする。 |
| バックアップ・復旧 | 実装対象。ASB独自の運用補強としてJSONファイルベースのバックアップ・復旧を提供する。 |

Rev.90 時点の ASB は、XServer Static 互換性に以下を含めない。

| 対象 | ASBでの扱い |
|-----|------------|
| XServer Static 完全互換 | 対象外。利用体験の一部互換目標であり、完全互換ではない。 |
| XServer Static 管理画面再現 | 対象外。ASBはHTTPS JSON API中心のヘッドレス構成とする。 |
| XServer Static 内部実装再現 | 対象外。ASB独自仕様を正とする。 |
| FTP / FTPS / SFTP | 対象外。将来計画にも含めない。 |
| DNS 管理機能 | 対象外。DNSレコード管理はASB外部の運用責務とする。 |
| DNS provider API | 対象外。DNS-01とwildcardを実装しないため不要とする。 |
| DNS-01 | 対象外。domain validation は HTTP-01 のみに限定する。 |
| wildcard 証明書 | 対象外。サブドメインは個別Domainとして扱う。 |
| 複数CA | 対象外。Let’s Encrypt のみに固定する。 |
| CA選定 / CA failover | 対象外。CA選択を利用者向け機能にしない。 |
| CDN完全互換 | 対象外。静的配信はASB本体またはASB外部の運用境界で扱う。 |
| 課金・契約・アカウント管理 | 対象外。セルフホスト・スタンドアロン運用とする。 |

**ASB互換目標による実装禁止**

ASB互換目標を理由に、以下を追加してはならない。

- `.gitignore`
- 外部DB
- 未承認の外部ライブラリ
- 未承認の外部サービス連携
- 開発リポジトリ内の実行時データ
- 起動時の実行時データ自動生成
- ビルド成果物の開発リポジトリ内自動生成
- 複数ユーザー管理
- APIキー管理
- Rate limiting
- Brotli 圧縮
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
- ASB 本体は Web UI、GUI、デスクトップアプリ、モバイルアプリを内包しない
- `ASB Core` という名称を使用しない
- ASB Web UI は本体外の内製標準管理画面として対応必須とする
- ASB Web UI は ASB 標準Web UIとして扱う
- ASB 標準Web UI は ASB SDK を利用して ASB 管理 HTTPS JSON API と通信する
- ASB SDK は単一の公式SDKとして扱う
- ASB SDK は Browser JavaScript、Deno専用 TypeScript、Go の対応実装を持つ
- ASB SDK の Browser JavaScript 実装は ASB 標準Web UI 用のブラウザ専用実装として扱う
- ASB SDK の Deno専用 TypeScript 実装は Deno 利用者向け実装として扱う
- ASB SDK の Go 実装は Go 利用者向けの ASB 管理 HTTPS JSON API クライアント実装として扱う
- ASB SDK は ASB 管理 HTTPS JSON API の通信層として扱う
- ASB SDK は SDK 専用プロトコル、SDK 専用エンドポイント、SDK 専用保存状態を前提としない
- 外部開発者は ASB SDK を使用する限り、ASB 標準Web UI のカスタマイズまたは独自フロントエンド実装を自由に行える
- 外部開発者の独自フロントエンド実装では、外部フロントエンドフレームワークを採用できる
- 外部開発者は、ASB、ASB 標準Web UI、ASB SDK など、Adlaire Group が開発元の公式プロジェクトに関与できない
- Go標準ライブラリで実装可能な部分は Go標準ライブラリで実装する
- 複数ユーザー化、APIキー管理など詳細未確定の機能は、仕様確定後に実装対象へ昇格する
- FTP、FTPS、SFTP は将来計画、保留事項、ASB互換目標、実装対象に含めない

---

## 4 設計思想

### 4.1 内製化・ゼロ外部依存設計

ASB は完全内製化を原則とする。

- 外部ライブラリは最小限（Go標準ライブラリ中心）
- HTTPS サーバーとして単一バイナリ化
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
| HTTPS サーバー | Go 標準 `net/http` + `crypto/tls` |
| JSON 処理 | Go 標準 `encoding/json` |
| ファイル操作 | Go 標準 `os`, `io` |
| 圧縮 | Go 標準 `archive/tar`, `compress/gzip` |
| 暗号化 | Go 標準 `crypto` |
| SSL 証明書 | 無料独自SSL（Let’s Encrypt ACME v2 / HTTP-01 に限定して証明書取得・更新を自動化） |
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

ASB は実行時データの初期化、自動生成、不足補完を行ってはならない。

ASB は起動時に、実行時データ用のディレクトリまたはファイルを自動生成しない。

ASB 起動時には、設定された保存先に以下の実行時データ領域が存在すること、および必要な読み書き権限を持つことを検証する。存在しない、または権限が不足する場合は起動失敗とする：

- `config/` - 実行時 JSON ファイル（projects.json、domains.json、backups.json等）
- `storage/` - プロジェクトファイルストレージ
- `logs/` - ログファイル（access.log、error.log等）
- `certs/` - SSL証明書ストレージ

実行時データは JSON ファイルベースで管理し、外部DBは使用しない。実行時データの保存先は `storage.basePath` によって指定する。

`storage.basePath` は実行時データの参照・保存境界であり、ASB が不足データを作成してよい場所ではない。

実行時データの配置、初期 JSON、必須 directory は、ASB 実行前に運用者が用意する。

起動設定ファイル `config.json` は `--config` で指定されたパスから読み込む。標準運用パスは `/etc/asb/config.json` とする。

起動設定ファイル `config.json` は `storage.basePath` 配下の実行時 JSON ではない。`storage.basePath/config/config.json` を ASB 標準実行時データとして扱ってはならない。

ASB は `/etc/asb/config.json`、`storage.basePath/config/config.json`、その他の起動設定ファイルを自動生成、補完、修復、移動、コピーしてはならない。

ASB は、明示的な利用者操作による対象 resource の作成、更新、削除を除き、実行時データを生成してはならない。

明示的な利用者操作とは、管理 HTTPS JSON API、GitHub Webhook deploy、Backup create、Restore、Log write / rotation、`asb migrate --apply` のうち、仕様で保存状態変更が定義された操作のみを指す。

`asb init-runtime` は Rev.90 時点では実装対象外とし、実行時データ初期化コマンドとして提供してはならない。

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
- 無料独自SSLの有効化、証明書取得、証明書更新、状態確認を管理する
- 無料独自SSLは Let’s Encrypt ACME v2 を内部実装方式として使用する
- domain validation は HTTP-01 challenge のみに限定する

### 7.4 ファイル管理

**ファイルアップロード**
- 最大容量：1GB/プロジェクト（設定可能）
- 形式：制限なし（HTML, CSS, JavaScript, 画像等）
- 圧縮：Gzip による自動圧縮
- Brotli は Rev.90 時点では ASB 本体に実装しない

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

監視値は `GET /api/monitoring/stats` の各リクエスト時に都度計算し、監視履歴、メトリクスJSON、キャッシュ、一時ファイル、監視DBとして保存しない。

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
  "code": "ERR_INVALID_REQUEST",
  "timestamp": "2026-09-08T00:00:00Z",
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

ASB のファイル操作は HTTPS JSON API、`multipart/form-data` upload、GitHub Webhook デプロイに限定する。

FTP、FTPS、SFTP はファイル操作手段として実装しない。

FTP、FTPS、SFTP 用のユーザー、認証、接続管理、転送ログ、設定項目、JSON ファイル、ディレクトリ、外部ライブラリを追加してはならない。

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
| 401 | Unauthorized | Webhook署名検証失敗、管理API認証失敗 |
| 403 | Forbidden | 初期デフォルトパスワード変更必須 |
| 404 | Not Found | リソース未検出 |
| 409 | Conflict | リソース競合（既存データの重複等） |
| 413 | Payload Too Large | ファイルサイズ超過 |
| 500 | Internal Server Error | サーバー内部エラー |
| 503 | Service Unavailable | サーバー一時利用不可 |

**エラーコード定義**

| エラーコード | HTTPステータス | メッセージ | 原因 |
|------------|--------------|---------|------|
| ERR_INVALID_JSON | 400 | Invalid JSON | リクエストJSONまたは保存JSONの構文不正 |
| ERR_UNKNOWN_FIELD | 400 | Unknown field | リクエストJSON、設定JSON、保存JSONの未知フィールド |
| ERR_INVALID_REQUEST | 400 | Invalid request | Content-Type、Body、query、path parameterの不正 |
| ERR_AUTH_FAILED | 401 | Authentication failed | システム管理者パスワードが未指定または不正 |
| ERR_AUTH_PASSWORD_CHANGE_REQUIRED | 403 | Password change required | 初期デフォルトパスワードの変更が必要 |
| ERR_PROJECT_NOT_FOUND | 404 | Project not found | プロジェクトが存在しない |
| ERR_PROJECT_ALREADY_EXISTS | 409 | Project already exists | プロジェクト名が重複 |
| ERR_PROJECT_QUOTA_EXCEEDED | 413 | Project quota exceeded | Project quotaを超過 |
| ERR_DOMAIN_NOT_FOUND | 404 | Domain not found | ドメインが存在しない |
| ERR_DOMAIN_ALREADY_ASSIGNED | 409 | Domain already assigned | ドメインが既に割り当て済み |
| ERR_OPERATION_CONFLICT | 409 | Operation conflict | 対象 resource が別操作中であり排他制御により操作できない |
| ERR_FILE_NOT_FOUND | 404 | File not found | ファイルが存在しない |
| ERR_FILE_UPLOAD_FAILED | 500 | File upload failed | ファイルアップロード失敗 |
| ERR_STORAGE_VALIDATION_FAILED | 500 | Storage validation failed | 実行時データ領域、必須ディレクトリ、必須JSONの検証失敗 |
| ERR_BACKUP_NOT_FOUND | 404 | Backup not found | バックアップが存在しない |
| ERR_BACKUP_RESTORE_CONFLICT | 409 | Backup restore conflict | 復旧対象の状態が復旧条件を満たさない |
| ERR_BACKUP_RESTORE_FAILED | 500 | Backup restore failed | バックアップ復旧失敗 |
| ERR_WEBHOOK_SIGNATURE_INVALID | 401 | Invalid webhook signature | Webhook署名が不正 |
| ERR_WEBHOOK_PROJECT_NOT_CONFIGURED | 500 | Webhook project not configured | Webhookデプロイ先Projectが未設定 |
| ERR_WEBHOOK_SOURCE_INVALID | 500 | Webhook source invalid | Webhookデプロイ元が不正 |
| ERR_WEBHOOK_PROCESSING_FAILED | 500 | Webhook processing failed | Webhook処理失敗 |
| ERR_SSL_OPERATION_CONFLICT | 409 | SSL operation conflict | 無料独自SSL操作が現在状態と競合 |
| ERR_SSL_CERT_GENERATION_FAILED | 500 | SSL certificate validation failed | 無料独自SSL・SSL証明書管理で証明書状態検証に失敗 |
| ERR_LOG_READ_FAILED | 500 | Log read failed | ログAPIの読み込みまたはJSON Lines検証に失敗 |
| ERR_LOG_WRITE_FAILED | 500 | Log write failed | ログ書き込み、fsync、rotation、保存前検証に失敗 |
| ERR_INTERNAL | 500 | Internal server error | 上記に分類できない内部エラー |

エラーレスポンスの `code` は上記表または `13.17.14 エラーコード固定表` の値のみ許可する。

Webhook の `ignored` と `duplicate` は正常応答であり、エラーコードを返してはならない。

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

本章に記載する `config/projects.json`、`config/domains.json`、`config/backups.json`、`config/auth.json`、`config/webhooks.json`、`config/acme_accounts.json`、`config/acme_orders.json`、`config/acme_authorizations.json`、`config/acme_challenges.json`、`config/acme_renewals.json` は、すべて `storage.basePath` 配下の実行時 JSON とする。

起動設定ファイル `config.json` は、`--config` で指定された単一ファイルであり、標準パスは `/etc/asb/config.json` とする。本章で `config.json` という表記を使用する場合でも、実行時 JSON ではなく起動設定ファイルの論理名を指す。

ASB は本章の空状態 JSON を自動生成してはならない。空状態 JSON は、ASB 起動前に運用者が事前配置する内容としてのみ定義する。

### 10.1 config/projects.json

空状態は以下とする。

```json
{
  "schemaVersion": 1,
  "projects": []
}
```

```json
{
  "schemaVersion": 1,
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

空状態は以下とする。

```json
{
  "schemaVersion": 1,
  "backups": []
}
```

```json
{
  "schemaVersion": 1,
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

### 10.3 起動設定ファイル config.json

起動設定ファイル `config.json` は、`--config` で指定されたパスから読み込む。標準パスは `/etc/asb/config.json` とする。

起動設定ファイルは、ASB の実行時データではない。ASB は起動設定ファイルを作成、更新、削除してはならない。

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
    "renewBefore": 7776000,
    "renewCheckInterval": 21600
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
| ssl | email | string | admin@example.com | Let’s Encrypt ACME account 用メール |
| ssl | renewBefore | int | 7776000 | 更新タイミング（秒、デフォルト90日前） |
| ssl | renewCheckInterval | int | 21600 | 証明書更新チェック間隔（秒、デフォルト6時間） |
| log | level | string | info | ログレベル（debug/info/warn/error） |
| log | format | string | json | ログ形式（JSON Lines） |
| log | maxSize | number | 104857600 | ログファイル最大サイズ（バイト、デフォルト100MB） |
| deploy | projectId | string | 空文字 | Webhookデプロイ先Project ID。空文字の場合、Webhookデプロイは失敗扱い |
| deploy | sourcePath | string | /srv/asb/source | Webhookデプロイ元ローカルcheckout絶対パス |
| deploy | branch | string | main | Webhookデプロイ対象ブランチ |
| webhook | githubSecret | string | 空文字 | GitHub Webhook署名検証用secret。空文字の場合は署名検証を行わない |

起動設定ファイル `config.json` の最終仕様は §13.17.15 を正とする。

### 10.4 config/domains.json

空状態は以下とする。

```json
{
  "schemaVersion": 1,
  "domains": []
}
```

```json
{
  "schemaVersion": 1,
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

### 10.5 config/auth.json

`config/auth.json` は単一システム管理者パスワード認証の状態を保存する。

空状態は以下とする。

```json
{
  "schemaVersion": 1,
  "admin": {
    "passwordHash": "",
    "passwordSalt": "",
    "passwordChanged": false,
    "updatedAt": ""
  }
}
```

`passwordHash` と `passwordSalt` は平文パスワードを保存してはならない。

`passwordChanged` が `false` の場合、初期デフォルトパスワードからの変更が完了していない状態を表す。

### 10.6 storage/projects/:projectId/files.json

空状態は以下とする。

```json
{
  "schemaVersion": 1,
  "files": []
}
```

```json
{
  "schemaVersion": 1,
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

### 10.7 config/webhooks.json

空状態は以下とする。

```json
{
  "schemaVersion": 1,
  "events": []
}
```

```json
{
  "schemaVersion": 1,
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

### 10.8 config/migrations.json

`config/migrations.json` はマイグレーション履歴専用 JSON ファイルである。

`config/migrations.json` は起動時必須 JSON ファイルではない。

`config/migrations.json` は実行時データ初期化コマンドの作成対象にしてはならない。

`config/migrations.json` は通常の API 処理、静的配信、Webhook、Backup、Log API、SSL 管理境界では作成してはならない。

`config/migrations.json` は ASB が初回作成してはならない。

`asb migrate --apply` は、`config/migrations.json` が存在しない場合、履歴ファイルを作成せず失敗しなければならない。

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

### 10.9 ACME 実行時 JSON 空状態

ACME 実行時 JSON の空状態は以下に固定する。

| ファイル | 空状態 |
|---------|--------|
| `config/acme_accounts.json` | `{"schemaVersion":1,"accounts":[]}` |
| `config/acme_orders.json` | `{"schemaVersion":1,"orders":[]}` |
| `config/acme_authorizations.json` | `{"schemaVersion":1,"authorizations":[]}` |
| `config/acme_challenges.json` | `{"schemaVersion":1,"challenges":[]}` |
| `config/acme_renewals.json` | `{"schemaVersion":1,"renewals":[]}` |

上記ファイルは ASB 起動前に運用者が事前配置する。

ASB は上記ファイルを起動時、SSL状態確認時、無料独自SSL有効化時、証明書更新チェック時に初回作成してはならない。

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

Rev.90 時点では、Webhook失敗時の自動リトライスケジュールを実装しない。

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
- バックアップ保存先を別障害領域へ複製する作業は Rev.90 時点ではASB外の運用責務とする
- 外部ストレージ連携は Rev.90 時点では実装対象外とする
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
- 標準出力（stdout）への通常ログ出力は Rev.90 時点では実装しない
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

Rev.90 時点の確定対象は、ASB 本体が提供する HTTPS JSON API、ASB SDK が使用する通信規格、ASB SDK の Browser JavaScript 実装、Deno専用 TypeScript 実装、Go 実装、ASB 標準Web UI、および ASB 標準Web UI が ASB SDK を利用して ASB と通信する構成である。

ASB SDK は、単一の公式SDKとして扱う。

ASB SDK は、Browser JavaScript、Deno専用 TypeScript、Go の対応実装を持つ。

ASB SDK の Browser JavaScript 実装は、ASB 標準Web UI から使用するブラウザ専用 JavaScript 実装として対応必須とする。

ASB SDK の Deno専用 TypeScript 実装は、Deno 専用ランタイムで動作する TypeScript 実装として対応必須とする。

ASB SDK の Go 実装は、Go 利用者向けの ASB 管理 HTTPS JSON API クライアント実装として対応必須とする。

ASB Web UI は、ASB 本体外の内製標準管理画面クライアントとして対応必須とする。

ASB Web UI は、ASB 標準Web UIとして扱う。

ASB Web UI は、静的 HTML / CSS / JavaScript とブラウザ標準 API のみで実装する。

ASB 標準Web UI は、ASB SDK の Browser JavaScript 実装を利用して ASB 管理 HTTPS JSON API と通信する。

ASB Web UI は、ASB の内部 JSON、Service、Repository、Storage を直接参照または呼び出してはならない。

ASB 本体は、Web UI 画面、Web UI テンプレート、Web UI フロントエンドビルド、Web UI 専用保存 JSON、Web UI 専用実行時データを持たない。

ASB SDK は、管理 API 呼び出し時に単一システム管理者パスワードを `X-ASB-Admin-Password` ヘッダーとして送信する。

ASB SDK は、SDK 専用プロトコル、SDK 専用エンドポイント、SDK 専用セッション、SDK 専用保存 JSON、SDK 専用実行時データを前提としてはならない。

ASB SDK の Browser JavaScript 実装は、Node.js 実行環境、npm 配布、package manager、bundler、transpiler、外部ライブラリを前提としてはならない。

ASB SDK の Deno専用 TypeScript 実装は、Deno 以外のランタイムを前提としてはならない。

ASB SDK の Deno専用 TypeScript 実装は、Node.js、npm、package manager、`package.json`、`node_modules/`、`deno.json`、`deno.lock`、bundler、transpiler、外部ライブラリを前提としてはならない。

ASB SDK の Go 実装は、Go標準ライブラリで実装可能な部分を Go標準ライブラリで実装する。

SDK 認証拡張仕様、デスクトップアプリ向け SDK 利用、モバイルアプリ向け SDK 利用は Rev.90 時点では実装対象外とし、確定仕様へ昇格するまで API、設定項目、JSON、ディレクトリ、外部依存、実行時データを追加してはならない。

SDK 通信規格は、ASB 本体が提供する HTTPS JSON API と同一とする。

外部開発者は、ASB SDK を使用する限り、Web UI のカスタマイズまたは独自フロントエンド実装を自由に行える。

外部開発者の独自フロントエンド実装では、外部フロントエンドフレームワーク、bundler、transpiler、package manager を採用できる。

外部開発者の独自フロントエンド実装は、Adlaire Group が開発元の公式 ASB、公式 ASB 標準Web UI、公式 ASB SDK、公式仕様の一部として扱わない。

外部開発者は、ASB、ASB 標準Web UI、ASB SDK など、Adlaire Group が開発元の公式プロジェクトに関与できない。

**API 設計原則**
- HTTPS + JSON を使用する
- HTTP/2 対応は ASB互換目標として扱い、Rev.90 時点では ASB 本体に実装しない
- デフォルト接続境界は `https://localhost:3000` とする
- 管理 API は Rev.90 時点では単一システム管理者パスワード認証を必須とする
- ASB SDK の Browser JavaScript 実装はブラウザ Web 標準 API のみを使用する
- ASB SDK の Deno専用 TypeScript 実装は Deno runtime API と Web 標準 API の範囲で実装する
- ASB SDK の Go 実装は Go標準ライブラリを中心に実装する
- SDK 通信は ASB 管理 API と同じ request / response / error / timestamp / pagination / upload 規約に従う
- SDK 専用プロトコル、SDK 専用エンドポイント、SDK 専用セッションを追加しない
- APIキー管理、複数ユーザー化、SDK認証拡張仕様は保留事項として扱う
- 開発ローカルおよび本番環境の管理 API は HTTPS JSON API として提供する
- 管理 API の HTTP 平文提供を前提としてはならない

**Rev.90 実装確定境界**
- Rev.90 の実装対象は、ASB 本体、ASB SDK、ASB 標準Web UI、無料独自SSL、マイグレーション、配布手順、単一システム管理者認証に限定する
- Rev.90 の実装対象外項目は、実装禁止契約、生成禁止対象、昇格条件のみを仕様として固定する
- Rev.90 の実装対象外項目を理由に、API、設定項目、保存JSON、ディレクトリ、外部依存、実行時データを追加してはならない
- Rev.90 の実装対象外項目は、実装してよい余地ではなく、実装禁止対象として扱う

**エラーレスポンス形式**
```json
{
  "error": "string (エラーメッセージ)",
  "code": "string (エラーコード)",
  "timestamp": "string (UTC RFC3339 秒精度)",
  "httpStatus": 400
}
```

管理 HTTPS JSON API の成功レスポンス、エラーレスポンス、共通ヘッダー、requestId、timestamp、body不許可時の扱いは、§13.16「入出力契約固定仕様」を正とする。

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
- ASB 本体の標準 listen address は `localhost` とする
- ASB 本体の標準管理 API URL は `https://localhost:3000` とする
- 開発ローカルおよび本番環境の管理 API は HTTPS JSON API とする
- ASB 本体は HTTPS による管理 API 提供を実装する
- ASB 本体は Rev.90 時点ではインターネット公開用 listen 設定を既定値として提供しない
- リモートアクセス制御はファイアウォール、VPN、SSH tunnel、IP制限等の運用境界で補強する
- ASB 本体は Rev.90 時点ではリバースプロキシ設定ファイルを生成しない
- SSL/TLS：開発ローカルおよび本番環境の管理 API で必須とする
- ASB 本体は Rev.90 時点では forwarded header trust list、trusted proxy list、proxy mode 設定を提供しない
- ASB 本体は `X-Forwarded-Host`、`Forwarded`、`X-Forwarded-For`、`X-Real-IP` を信頼境界として使用しない

**レート制限**
- Rev.90 時点では ASB 本体に実装しない
- Rate limiting は、本番公開時に ASB 外部のリバースプロキシ、WAF、CDN、ファイアウォール等で扱う

**タイムアウト**
- リクエスト読み込み：30秒
- レスポンス書き込み：60秒
- ログ保持時間：7日間

サーバー実行境界、timeout、request size、panic recovery、client disconnect、graceful shutdown、CORS、Health Check の実装契約は §13.15.4.2 を正とする。

**ファイルアップロード**
- 最大サイズ：1GB
- 形式制限：なし
- 置換許可：同名ファイル上書き可能

**ログ出力**
- アクセスログ：全HTTP リクエスト（JSON形式）
- エラーログ：エラー・例外・警告
- ログファイル暗号化：Rev.90 時点では ASB 本体に実装しない
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

1. `ASB-spec.md`、`ASB-spec.html`、`IMPLEMENTATION_TASKS.md`、`README.md` の参照Revが一致している
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

ASB の実装フェーズ管理は、`IMPLEMENTATION_TASKS.md` に限定する。

`ASB-spec.md` は仕様正本であり、実装フェーズ管理表、優先度表、フェーズ別タスクリスト、フェーズ別完了条件を記載してはならない。

`ASB-spec.md` に具体的なフェーズ番号やフェーズ別詳細を追加してはならない。

フェーズ番号、優先度、開発版バージョン、実装順序、実装タスク、フェーズ別完了条件は、`IMPLEMENTATION_TASKS.md` を正とする。

`ASB-spec.md` には、API、JSON、エラー、保存、起動、ログ、通信、セキュリティ、禁止事項など、実装時に守る仕様契約のみを記載する。

`IMPLEMENTATION_TASKS.md` が `ASB-spec.md` と矛盾する場合は、仕様内容については `ASB-spec.md` を正とし、フェーズ管理については `IMPLEMENTATION_TASKS.md` を修正する。

##### 11.9.2.6 README 管理方針

`README.md` は、ASB の概要、現在状態、主要文書への入口を示す文書である。

`README.md` は、仕様判断、実装判断、フェーズ判断の正本ではない。

`README.md` に記載できる内容は、`ASB-spec.md` で確定済みの仕様、`IMPLEMENTATION_TASKS.md` で管理する実装フェーズの要約、主要文書の役割説明に限定する。

`README.md` を理由に、API、設定項目、保存JSON、ディレクトリ、外部依存、実行時データ、実装フェーズ、優先度、開発版バージョンを追加または変更してはならない。

`README.md` の機能説明、現在状態、文書一覧、参照Revが `ASB-spec.md` または `IMPLEMENTATION_TASKS.md` と矛盾する場合は、`README.md` を修正する。

`ASB-spec.md` の本書バージョンを更新する場合は、`ASB-spec.html`、`IMPLEMENTATION_TASKS.md` と同様に `README.md` の参照Revと概要文の整合を確認する。

### 11.10 将来計画管理ポリシー

将来計画は、Rev.90 時点の実装対象ではない。

将来計画に記載された項目は、実装、設定追加、API追加、JSON追加、ディレクトリ追加、外部依存追加、実行時データ生成の根拠として扱ってはならない。

Rev.90 時点で実装対象外とする将来計画は以下とする。

- デスクトップアプリ（実装対象外、ASB SDK 利用クライアント候補）
- モバイルアプリ（実装対象外、ASB SDK 利用クライアント候補）
- 複数インスタンス対応
- NFS 連携
- 分散ストレージ連携
- 複数ユーザー化
- マルチテナント対応
- 外部ストレージ連携
- ログファイル暗号化
- HTTP/2 実装詳細

ASB Web UI は将来計画ではなく、ASB 本体外の内製管理画面クライアントとして対応必須とする。

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

GitHub Releases 以外の配布元を標準配布元として扱ってはならない。

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

標準配布成果物以外を同一 release に含める場合は、事前に `ASB-spec.md` を改訂し、ファイル名、用途、checksum 対象性、install/update 対象性を固定しなければならない。

配布成果物をGit管理対象として開発リポジトリ内へ保存してはならない。

開発リポジトリ内に配布成果物、圧縮済み配布物、release 作業用一時ファイル、download 済み release asset、署名検証用一時ファイルを残してはならない。

`checksums.txt` は以下の形式とする。

```text
<sha256>  asb-linux-amd64-vX.Y
<sha256>  asb-linux-arm64-vX.Y
```

`checksums.txt` に記載するファイル名は、GitHub Releases 上の配布ファイル名と完全一致させる。

`checksums.txt` の各行は、小文字16進64文字のSHA-256、半角スペース2文字、ファイル名、改行の順に固定する。

`checksums.txt` に空行、コメント行、相対path、絶対path、URL、glob、タブ区切り、CRLF、未配布ファイル名を含めてはならない。

install/update は、対象 arch のバイナリ1件と `checksums.txt` のみを検証対象として扱う。

`checksums.txt` に対象 arch のファイル名が存在しない場合、install/update は失敗しなければならない。

対象 arch 以外の checksum 行が不正な場合でも、対象 arch と `checksums.txt` 自体の形式検証に失敗するなら install/update は成功扱いしてはならない。

#### 11.11.4 ビルド固定仕様

標準ビルドコマンドは以下に固定する。

```bash
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -trimpath -ldflags "-s -w" -o dist/asb-linux-amd64-vX.Y ./cmd/asb
CGO_ENABLED=0 GOOS=linux GOARCH=arm64 go build -trimpath -ldflags "-s -w" -o dist/asb-linux-arm64-vX.Y ./cmd/asb
```

`dist/` はリリース作業用の一時出力先であり、開発リポジトリへ残してはならない。

ビルド成果物は、GitHub Releases へアップロードした後に開発リポジトリから削除する。

ビルド成果物の削除に失敗した状態で安定版リリース完了として扱ってはならない。

標準ビルドでは、外部 build tool、外部 packaging tool、container image、installer generator、archive generator を必須前提にしてはならない。

`go build` 実行時に module download が必要な外部依存を追加する場合は、外部依存ルールに従い事前に例外採用を固定しなければならない。

#### 11.11.5 インストール手順

**前提**
- Linux amd64 または arm64 環境
- SSH アクセス可能
- GitHub Releases から対象バージョンの配布成果物を取得可能
- `sha256sum` または `shasum -a 256` を利用可能
- systemd を利用可能
- root 権限で `/usr/local/bin/` と `/etc/systemd/system/` へ配置可能

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

install は、既存 `/usr/local/bin/asb`、既存 `/etc/systemd/system/asb.service`、既存 `asb` service のいずれかが存在する場合、既存環境の上書きを行わず失敗する。

install は、`/etc/asb/config.json` を自動生成してはならない。

install は、`storage.basePath` 配下の実行時データを生成してはならない。

初回 runtime JSON 作成は ASB の責務ではない。

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
$ sudo mv /usr/local/bin/asb /usr/local/bin/asb.previous
$ sudo install -o root -g root -m 0755 asb-linux-amd64-vX.Y /usr/local/bin/asb

# 6. 起動
$ sudo systemctl start asb

# 7. 起動状態を確認
$ sudo systemctl is-active --quiet asb
```

アップデート後に起動確認が失敗した場合、`/usr/local/bin/asb.previous` を `/usr/local/bin/asb` へ戻し、`systemctl start asb` を再実行する。

update は、既存 `/usr/local/bin/asb` が存在しない場合、install へフォールバックしてはならない。

update は、既存 `/usr/local/bin/asb.previous` が存在する場合、上書きしてはならない。

update は、`/etc/asb/config.json`、`storage.basePath`、runtime JSON、静的コンテンツ、証明書、ログを変更してはならない。

update は ASB 本体バイナリと `asb.service` のみを更新対象にできる。

#### 11.11.7 install.sh / update.sh 固定仕様

ASB の初回インストールとアップデートを自動化するため、`install.sh` と `update.sh` を提供する。

両スクリプトはPOSIX sh互換で実装する。

両スクリプトは Linux systemd 環境専用とする。

macOS、Windows、非 systemd Linux、container 専用環境向けの install/update 分岐を Rev.90 時点では実装しない。

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

`latest` 指定、自動最新版選択、未指定バージョンでの実行は Rev.90 時点では禁止する。

`--version` は `^v[1-9][0-9]*\.[1-9][0-9]*$` に一致する値のみ許可する。

`v0.N`、`main`、branch 名、commit hash、tag 未指定、空文字、`latest`、`stable`、`nightly` は拒否する。

`--arch` 未指定時の自動判定は `uname -m` の結果のみを使用する。

`uname -m` が `x86_64` または `amd64` の場合は `amd64` とする。

`uname -m` が `aarch64` または `arm64` の場合は `arm64` とする。

上記以外の値では実行を中止し、ダウンロード、ファイル置換、systemd 操作を行ってはならない。

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
11. 途中失敗時に未検証バイナリを `/usr/local/bin/asb` へ配置しない
12. root 権限が必要な処理の前に検証可能な入力、arch、URL、checksum、version を検証する
13. curl が失敗した場合に wget へ暗黙 fallback してはならない
14. checksum 検証コマンドが存在しない場合は失敗する
15. `systemctl is-active --quiet asb` が失敗した場合は起動失敗として扱う

`install.sh` は既存 `/usr/local/bin/asb` が存在する場合、上書きしてはならず、終了コード `1` で失敗する。

`install.sh` は既存 `/etc/systemd/system/asb.service` が存在する場合、上書きしてはならず、終了コード `1` で失敗する。

`install.sh` は `systemctl is-enabled asb` または `systemctl status asb` で既存 service を確認できる場合、終了コード `1` で失敗する。

`install.sh` は checksum 検証、`--version` 出力確認、`install` による配置、`asb.service` 配置、`systemctl daemon-reload`、`systemctl enable asb`、`systemctl start asb` の順で実行する。

`install.sh` は上記のいずれかに失敗した場合、成功扱いしてはならない。

`update.sh` は checksum 検証と `--version` 出力確認が完了するまで、既存サービス停止、既存バイナリ退避、バイナリ置換を実行してはならない。

`update.sh` は既存 `/usr/local/bin/asb` が存在しない場合、終了コード `1` で失敗する。

`update.sh` は `/usr/local/bin/asb.previous` が既に存在する場合、上書きしてはならず、終了コード `1` で失敗する。

`update.sh` は既存サービス停止後、既存 `/usr/local/bin/asb` を `/usr/local/bin/asb.previous` へ rename し、新バイナリを `/usr/local/bin/asb` へ配置する。

新バイナリ配置、`systemctl start asb`、起動状態確認のいずれかに失敗した場合は、`/usr/local/bin/asb.previous` を `/usr/local/bin/asb` へ戻す復旧を1回だけ試行する。

復旧では、失敗した新バイナリを `/usr/local/bin/asb.failed` 等へ退避してはならない。

復旧では、`/usr/local/bin/asb.previous` を `/usr/local/bin/asb` へ rename し、`systemctl start asb`、`systemctl is-active --quiet asb` の順に実行する。

復旧後、`/usr/local/bin/asb.previous` は存在しない状態に戻さなければならない。

復旧に成功した場合でも、`update.sh` は終了コード `1` で失敗する。

復旧に失敗した場合も、`update.sh` は終了コード `1` で失敗する。

`install.sh` と `update.sh` は、標準出力に進行状況を出力してよいが、シークレット、環境変数一覧、内部一時ディレクトリの詳細一覧を出力してはならない。

`install.sh` と `update.sh` が配置する `/usr/local/bin/asb` は owner `root`、group `root`、mode `0755` とする。

`update.sh` が作成する `/usr/local/bin/asb.previous` は owner `root`、group `root`、mode `0755` とする。

`install.sh` と `update.sh` は `/tmp` または `mktemp -d` の作業ディレクトリ以外にダウンロード中ファイルを作成してはならない。

`install.sh` と `update.sh` は失敗終了時に、作成した一時作業ディレクトリの削除を1回だけ試行する。

一時作業ディレクトリ削除に失敗した場合でも、終了コードは元の失敗理由に従い `1` とする。

`install.sh` と `update.sh` は `.gitignore`、`go.mod`、`deno.json`、`package.json`、lock file、cache directory、log file を開発リポジトリ内に作成してはならない。

#### 11.11.8 asb.service 固定仕様

```ini
[Unit]
Description=Adlaire-Static-Base HTTPS Server
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
PrivateTmp=true
ProtectSystem=full
ProtectHome=true

[Install]
WantedBy=multi-user.target
```

`asb.service` は `/etc/systemd/system/asb.service` へ配置する。

`asb.service` の owner は `root`、group は `root`、mode は `0644` とする。

`asb` ユーザーおよび `asb` グループが存在しない場合、`install.sh` は system user として作成する。

`asb` ユーザーはログイン不可の system user とし、home directory は作成しない。

`asb.service` は `ExecStart` で `asb start` ではなく `/usr/local/bin/asb --config /etc/asb/config.json` を実行する。

`asb.service` は `/etc/asb/config.json` を自動生成してはならない。

`asb.service` は起動時に runtime directory、runtime JSON、証明書、ログファイルを自動生成してはならない。

`asb.service` の変更時は、`systemctl daemon-reload` 後に `systemctl restart asb` を実行し、`systemctl is-active --quiet asb` で確認する。

#### 11.11.9 リリース・配布・運用境界固定仕様

Rev.90 時点では、ASB のリリース、配布、install、update、運用責務境界を本節に固定する。

開発版バージョン `v0.N` は実装フェーズ管理用であり、GitHub Releases の配布タグとして使用してはならない。

安定版バージョン `vX.Y` の `Y` は、切り出し元の開発版 `v0.Y` と一致させる。

Git tag は安定版バージョン `vX.Y` と完全一致させる。`latest`、`stable`、`nightly`、branch名、commit hash、日付文字列を release tag として扱ってはならない。

GitHub Releases は配布成果物置き場のみとし、runtime state、起動設定、runtime JSON、ログ、証明書、ACME token、Webhook secret、管理者パスワード、backup archive、migration 作業ファイルを含めてはならない。

GitHub が自動生成する source archive は ASB の運用入力、install/update 入力、runtime 初期化入力として扱ってはならない。

release 作成は、安定版切り出し判定基準を満たした commit に対する明示操作のみで実行する。

release artifact は ASB 本体単一バイナリ2件と `checksums.txt` のみに限定する。

ASB 本体 release artifact に `config.json`、runtime JSON、空状態 JSON、ログファイル、証明書ファイル、systemd 実行時状態、Web UI artifact、SDK artifact、`.gitignore` を同梱してはならない。

`install.sh` と `update.sh` は配布補助スクリプトであり、runtime 初期化、設定生成、証明書生成、ログ生成、空状態 JSON 生成を行ってはならない。

`install.sh` が新規配置できる対象は `/usr/local/bin/asb`、`/etc/systemd/system/asb.service`、必要な `asb` system user / group のみに限定する。

`update.sh` が変更できる対象は `/usr/local/bin/asb`、`/usr/local/bin/asb.previous`、`/etc/systemd/system/asb.service` のみに限定する。

`install.sh` と `update.sh` は `/etc/asb/config.json`、`storage.basePath`、runtime JSON、静的コンテンツ、証明書、ログを作成、更新、削除、修復してはならない。

`update.sh` は install へ fallback してはならない。`install.sh` は update へ fallback してはならない。

rollback は `update.sh` の既存バイナリ退避後、新バイナリ起動失敗時のみ実行できる。

rollback は `/usr/local/bin/asb.previous` を `/usr/local/bin/asb` へ戻す1回の rename と service 起動確認に限定する。

rollback 失敗時に追加退避ファイル、rollback state file、復旧ログ、cache、queue を作成してはならない。

運用者の事前責務は以下に固定する。

- `/etc/asb/config.json` の作成と配置
- `storage.basePath` 配下の必須 directory 作成
- runtime JSON 空状態の作成と配置
- `logs/access.log`、`logs/error.log` の作成と権限設定
- 管理 HTTPS API 用 TLS 証明書ファイルと秘密鍵ファイルの初期配置
- reverse proxy、firewall、VPN、SSH tunnel、IP制限、DNS設定、service manager の環境設定

ASB 本体、release artifact、install/update scripts、systemd unit は、上記の運用者責務を代行してはならない。

ASB の配布・運用境界を理由に、開発リポジトリ内へ release artifact、download 済み asset、checksum 作業ファイル、install/update log、runtime state、cache、一時ファイル、`.gitignore` を生成してはならない。

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

本章は将来計画の記録であり、Rev.90 時点の実装対象を増やすものではない。

以下は Rev.90 時点では実装対象外とする。

| 対象 | Rev.90 時点の扱い | 実装禁止範囲 |
|-----|------------------|------------|
| デスクトップアプリ | 将来計画・実装対象外 | ASB SDK 利用クライアント候補、GUIライブラリ採否、配布方式、OS対応 |
| モバイルアプリ | 将来計画・実装対象外 | ASB SDK 利用クライアント候補、GUIライブラリ採否、配布方式、iOS/Android対応 |
| 複数インスタンス対応 | 実装対象外 | 分散ロック、クラスタ管理、ノード管理 |
| NFS 連携 | 実装対象外 | NFS 専用設定、NFS 固有のロック処理 |
| 分散ストレージ連携 | 実装対象外 | 分散ストレージAPI、外部ストレージSDK |
| 複数ユーザー化 | 将来計画・実装対象外 | ユーザー管理、ロール、権限分離、セッション、招待、組織管理 |

### 12.1.1 将来計画詳細

**デスクトップアプリ**
- 将来計画とする
- Rev.90 時点では実装対象外とする
- ASB SDK 利用クライアント候補として扱う
- GUIライブラリ採否、配布方式、OS対応は昇格前の検討事項として扱い、実装根拠にしてはならない
- 実装前に、配布方式、OS対応、署名、更新方式、ASB SDK との関係を仕様で確定する

**モバイルアプリ**
- 将来計画とする
- Rev.90 時点では実装対象外とする
- ASB SDK 利用クライアント候補として扱う
- GUIライブラリ採否、配布方式、iOS/Android対応は昇格前の検討事項として扱い、実装根拠にしてはならない
- 実装前に、iOS/Android対応、配布方式、認証、通知、ASB SDK との関係を仕様で確定する

**複数ユーザー化**
- 将来計画とする
- Rev.90 時点では実装対象外とする
- Rev.90 の単一システム管理者モデルを前提に、ユーザー識別子、認証方式、セッション方式、権限モデル、システム管理者権限、Project / Domain / File / Backup / Log 操作権限、保存JSON、SDK認証拡張、Web UI ロール表示、監査ログ、移行手順を仕様で確定する

### 12.2 保留事項

以下は Rev.90 時点では実装対象外とする。

- 複数ユーザー化
- マルチテナント対応
- API キー管理
- SDK 外部配布 / npm 配布
- SDK 認証拡張
- Rate limiting
- 外部ストレージサービス統合
- ログファイル暗号化

上記を理由に、API、設定項目、保存 JSON、ディレクトリ、外部依存、実行時データを追加してはならない。

### 12.3 検討・調査中事項

Rev.90 時点では、検討・調査中事項を実装へ反映してはならない。

以下は調査対象としてのみ記録し、実装対象外とする。

- 複数インスタンス時の NFS / 分散ストレージ選定
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

Rev.90 時点の ASB 本体の実装対象は、ASB のセルフホスト型静的コンテンツ配信ホスティングに必要なバックエンド機能に限定する。

ASB 本体外の実装対象は、ASB SDK と ASB Web UI に限定する。

実装は以下の順序で進める：

1. 起動設定、実行時データ検証、HTTPS サーバー、共通レスポンス
2. プロジェクト管理
3. ファイル管理
4. ドメイン管理
5. ログ・監視
6. バックアップ・復旧
7. GitHub Webhook
8. SSL 証明書管理

ASB互換目標は機能目標として扱い、ASB 独自仕様は実装制約として扱う。

ASB互換目標は、実装優先順位、外部依存追加、実行時データ生成、API追加、設定項目追加の根拠として扱ってはならない。

ASB互換目標に含まれる未昇格機能は、個別の確定仕様へ昇格するまで実装対象外とする。

保留事項または検討中事項は、仕様上の扱いが確定するまで実装してはならない。

### 13.1.1 実装開始条件

ASB の実装開始前に、以下を満たしていなければならない。

- `AGENTS.md` を確認し、承認ルール、Git運用ルール、`.gitignore` 非使用ルールを把握していること
- 実装対象が `ASB-spec.md` に確定仕様として記載されていること
- 実装対象が `IMPLEMENTATION_TASKS.md` の該当実装フェーズに存在すること
- 作業ブランチ上で作業しており、`main` へ直接 push しないこと
- 開発リポジトリ内に `.gitignore` が存在しないこと
- 開発リポジトリ内に実行時データ、ログ、一時ファイル、cache、coverage output、build output、release artifact、download 済み asset、`dist/`、`.asb/` が存在しないこと
- 実装で必要な一時データ、テスト用 runtime JSON、テスト用 TLS 証明書、テストログの配置先が OS 一時領域または明示された実行環境領域であること
- 実装開始時点で未確定の API、設定項目、保存JSON、生成ファイル、生成ディレクトリ、外部依存をコード側で補完しないこと
- 仕様不足が見つかった場合は、実装を進めず `ASB-spec.md` の改訂へ戻すこと

実装開始時に以下を行ってはならない。

- `.gitignore` の作成
- 開発リポジトリ内への実行時データ生成
- 開発リポジトリ内への設定ファイル自動生成
- 開発リポジトリ内へのログ、一時ファイル、cache、coverage output、build output の生成
- 仕様未記載の API、CLI、設定項目、JSON、ディレクトリ、外部依存の追加
- 実装都合による仕様の黙示的変更
- 未確定の将来計画機能の先行実装
- GitHub Actions workflow の先行作成

Go 実装コード導入後にのみ、CI workflow の実体化を検討できる。

CI workflow 実体化は、`15.4 CI / 品質ゲート固定仕様` および `15.5 CI 実体化前提固定仕様` に従う。

実装フェーズ内の初回実装単位、作成許可ファイル、作成禁止ファイル、実行コマンド、完了確認は `IMPLEMENTATION_TASKS.md` で管理する。

実装者は、該当フェーズの初回実装単位を超えて API、CLI、設定読み込み、HTTPS server、runtime JSON、Web UI、SDK、CI workflow、release artifact を先行実装してはならない。

初回実装単位で仕様不足が見つかった場合は、対象コードを拡張せず、`ASB-spec.md` と `IMPLEMENTATION_TASKS.md` の整合改訂へ戻す。

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

UUID は RFC 4122 variant の version 4 UUID とし、小文字16進、ハイフン付き36文字の正規表現 `^[0-9a-f]{8}-[0-9a-f]{4}-4[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$` に一致する値のみ許可する。

UUID 生成時に `crypto/rand` の読み込みへ失敗した場合は、対象操作を失敗扱いとし、`ERR_INTERNAL` を返す。

時刻取得は Service に注入された Clock から行う。

Clock が時刻取得に失敗する実装を注入したテストでは、対象操作を失敗扱いとし、`ERR_INTERNAL` を返す。

Project `name` は正規表現 `^[A-Za-z0-9_-]{1,255}$` に一致する値のみ許可し、前後空白の trim、Unicode 正規化、大文字小文字変換を行ってはならない。

File `name` は URL decode 後の値を検証し、長さ 1〜255 bytes、NUL 文字なし、`/` なし、`\` なし、空白のみ不可、`.` 不可、`..` 不可、先頭 `.` 不可とする。

保存する File `path` は `contents/` で始まる相対パスのみ許可し、絶対パス、空文字、NUL 文字、`\`、`..` セグメント、`.` セグメント、空白のみのセグメントを許可しない。

Domain は小文字 ASCII のみ保存し、入力に大文字が含まれる場合は小文字化してから検証する。

Domain label は正規表現 `^[a-z0-9]([a-z0-9-]{0,61}[a-z0-9])?$` に一致する値のみ許可する。

Domain 全体は 1〜253 bytes、label 数は 2〜3、末尾 `.` は入力時に除去して保存する。

`deploy.branch` は正規表現 `^[A-Za-z0-9._/-]{1,255}$` に一致する値のみ許可し、先頭 `/`、末尾 `/`、`//`、`..`、`@{`、空白文字、制御文字、末尾 `.lock` を許可しない。

**エラー**

- エラーレスポンスには `error`、`code`、`timestamp`、`httpStatus` を含める
- 内部ファイルパス、スタックトレース、機密値をレスポンスへ含めてはならない
- 仕様に定義済みのエラーコードを優先して使用する

### 13.3 起動時検証

ASB は起動時に実行時データを自動生成しない。起動時検証は読み取り、権限確認、構文検証、スキーマ検証のみを行い、ディレクトリ、JSON、ログファイル、証明書、ACME challenge file、一時ファイルを作成、更新、削除、移動、rename してはならない。

起動時には以下を検証する：

- `--config` で指定された起動設定ファイルが存在する
- `storage.basePath` が存在する
- `storage.basePath` 配下に `config/`、`storage/`、`storage/projects/`、`logs/`、`certs/` が存在する
- 必要な JSON ファイルが存在し、JSON として読み込める
- `logs/access.log`、`logs/error.log` が存在し、追記可能である
- 書き込みが必要な領域に書き込み権限がある
- `server.port`、`server.host`、`shutdownTimeout`、`storage.maxProjectSize`、`ssl.renewBefore`、`log.level` が妥当である

検証に失敗した場合、ASB は HTTPS サーバーを開始せず、エラーを標準エラーへ出力して終了する。

起動時検証順序は以下に固定する。

1. `--config` で指定された起動設定ファイルの存在確認
2. 起動設定ファイルの通常ファイル確認
3. 起動設定ファイルの読み取り権限確認
4. 起動設定ファイルの JSON 構文検証
5. 起動設定ファイルの未知フィールド拒否
6. 起動設定値の型、範囲、空文字、絶対パス制約検証
7. `storage.basePath` が開発リポジトリ配下でないことの検証
8. `storage.basePath` の存在、directory、読み取り権限確認
9. `storage.basePath/config/`、`storage.basePath/storage/`、`storage.basePath/storage/projects/`、`storage.basePath/logs/`、`storage.basePath/certs/` の順序付き存在、directory、読み取り権限確認
10. `storage.basePath/config/projects.json`、`domains.json`、`backups.json`、`auth.json`、`webhooks.json`、`acme_accounts.json`、`acme_orders.json`、`acme_authorizations.json`、`acme_challenges.json`、`acme_renewals.json` の順序付き存在、通常ファイル、読み取り権限、書き込み権限、JSON 構文、`schemaVersion: 1`、必須トップレベルキー、未知フィールド不在検証
11. `storage.basePath/logs/access.log`、`storage.basePath/logs/error.log` の順序付き存在、通常ファイル、読み取り権限、追記権限確認
12. `config/projects.json` に存在する各 Project について、`storage.basePath/storage/projects/{projectId}/`、`contents/`、`files.json` の存在、権限、`files.json` の JSON 構文、`schemaVersion: 1`、必須トップレベルキー、未知フィールド不在検証
13. `config/domains.json` で SSL 状態が `issued`、`renewing`、`expired` の Domain について、証明書ファイルと秘密鍵ファイルの存在、権限、対応、有効期限確認

起動時検証失敗時の標準エラーは以下の形式に固定する。

```text
ASB_STARTUP_ERROR code=<errorCode> message="<message>"
```

起動設定ファイルの構文、未知フィールド、設定値不正は `ERR_INVALID_REQUEST` を使用する。

実行時ディレクトリ、実行時 JSON、ログファイル、証明書、権限、スキーマ不正は `ERR_STORAGE_VALIDATION_FAILED` を使用する。ただし証明書ファイルと秘密鍵の存在、読み込み、対応、有効期限検証に失敗した場合は `ERR_SSL_CERT_GENERATION_FAILED` を使用する。

起動時検証失敗時の終了コードは `1` とする。

### 13.4 実行時データ配置

実行時データは `storage.basePath` 配下に配置する。

起動設定ファイル `config.json` は実行時データ配置に含めない。標準運用では `/etc/asb/config.json` を使用する。

標準構成は以下とする：

```text
/var/asb/
├── config/
│   ├── projects.json
│   ├── domains.json
│   ├── backups.json
│   ├── auth.json
│   ├── webhooks.json
│   ├── acme_accounts.json
│   ├── acme_orders.json
│   ├── acme_authorizations.json
│   ├── acme_challenges.json
│   └── acme_renewals.json
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

初回起動前の空状態では、`storage/projects/`、`logs/access.log`、`logs/error.log`、`certs/`、および必須実行時 JSON を運用者が事前配置する。Project が存在しない場合、`storage/projects/{projectId}/` は存在しない。

本書で `config/`、`storage/`、`logs/`、`certs/` から始まる相対パスを記載する場合、特記がない限り `storage.basePath` 配下の相対パスを指す。

`storage.basePath/storage/projects/{projectId}/contents/` は、`storage.basePath` 直下の `storage/` ディレクトリ配下を指す固定パスである。

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
- アップロード後の使用量が `quota` を超える場合は `ERR_PROJECT_QUOTA_EXCEEDED` を返す
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

**転送プロトコル**

- ファイル操作手段は HTTPS JSON API、`multipart/form-data` upload、GitHub Webhook デプロイに限定する
- FTP は実装しない
- FTPS は実装しない
- SFTP は実装しない
- FTP / FTPS / SFTP 用の設定項目、認証情報、ユーザー、接続管理、転送ログ、JSON ファイル、ディレクトリ、外部ライブラリを追加しない

**圧縮**

- Gzip 圧縮を実装対象とする
- Brotli 圧縮は Rev.90 時点では ASB 本体に実装しない
- `Accept-Encoding: br` を受信しても Brotli 応答へ切り替えない

### 13.8 ドメイン管理詳細

- ドメインは小文字へ正規化して保存する
- ドメインは RFC 1035 に準拠し、各ラベルは 1〜63 文字とする
- ドメイン全体は 253 文字以下とする
- ドメインは最大3階層までとする
- 既に別プロジェクトへ割り当て済みの場合は `ERR_DOMAIN_ALREADY_ASSIGNED` を返す
- プロジェクト削除時は関連ドメインを削除する

Domain追加処理は以下の順序で実行する。

1. 対象 Project の存在を検証する
2. 入力 domain を小文字へ正規化する
3. 正規化後 domain の構文、長さ、階層数を検証する
4. `config/domains.json` を読み込み、構文、`schemaVersion: 1`、必須キー、未知フィールド不在を検証する
5. 同一 domain が同一 Project または別 Project に登録済みでないことを検証する
6. Domain object を `domains[]` に追加する
7. `domain` 昇順で保存する

Domain object の追加時は `isCustom: true` とし、`sslCert` は空文字とする。

Domain一覧は対象 Project に属する Domain のみを返し、`domain` 昇順に固定する。

Domain削除処理は以下の順序で実行する。

1. 対象 Project の存在を検証する
2. 対象 Domain の存在と Project 所属を検証する
3. 無料独自SSL状態を確認する
4. SSL状態が `pending`、`challenge_ready`、`renewing` の場合は `409 Conflict` と `ERR_SSL_OPERATION_CONFLICT` を返す
5. `config/domains.json` から対象 Domain object を削除する
6. `domain` 昇順で保存する

Domain削除時に既存証明書ファイル、秘密鍵ファイル、ACME関連JSON、ACME challenge token を即時削除してはならない。

Domain追加、一覧、削除は、開発リポジトリ内に実行時データ、一時ファイル、ログファイルを作成してはならない。

### 13.9 SSL 管理詳細

Rev.90 時点では、SSL 管理は XServer Static 互換目標として無料独自SSLを提供する。

無料独自SSLは、独自ドメイン単位で有効化し、証明書取得、証明書更新、状態確認を ASB 本体が自動実行する。

ACME は利用者向け機能名ではなく、無料独自SSLを実現する内部実装方式である。

Rev.90 時点の ACME は Let’s Encrypt ACME v2 のみに対応する。

domain validation は HTTP-01 challenge のみに限定する。

実装対象：

- SSL 証明書IDの管理
- 証明書保存先 `certs/` の存在確認
- 証明書ファイルパスの検証
- 証明書有効期限の監視モデル
- 証明書メタデータの JSON 保存
- 無料独自SSLの有効化
- 無料独自SSLの無効化
- SSL状態確認
- Let’s Encrypt ACME v2 client
- ACME account 登録
- ACME account key 生成・保存
- ACME directory 取得
- ACME nonce 管理
- ACME order 作成
- ACME authorization 取得
- HTTP-01 challenge 応答
- ACME finalize
- ACME certificate download
- 証明書自動更新
- 証明書更新スケジューラー
- ACME retry / backoff
- Let’s Encrypt rate limit 配慮
- SSL 証明書生成失敗時の `ERR_SSL_CERT_GENERATION_FAILED`

実装対象外：

- TLS-ALPN-01 challenge
- DNS-01 challenge
- wildcard 証明書
- 複数 CA
- CA 選定
- CA failover
- 任意 ACME directory URL
- DNS provider API 連携
- 手動 TXT 登録
- EAB
- ARI
- OCSP stapling

P3 の SSL状態基盤は、Domain と証明書メタデータの検証境界に限定する。

P3 では Let’s Encrypt との通信、ACME account 登録、ACME order 作成、challenge 応答、証明書取得、証明書自動更新を実行してはならない。

SSL状態確認では、対象 Domain の存在、Project 所属、SSL状態参照、証明書パス、秘密鍵パス、有効期限、証明書本文と秘密鍵本文の対応を検証する。

証明書ファイルの保存先は `storage.basePath/certs/{domain}/fullchain.pem`、秘密鍵ファイルの保存先は `storage.basePath/certs/{domain}/privkey.pem` に固定する。

証明書パスと秘密鍵パスは `storage.basePath` 配下の相対管理対象として扱い、`storage.basePath` 外を参照するパスを許可してはならない。

SSL状態が `issued`、`renewing`、`expired` の場合に証明書ファイルまたは秘密鍵ファイルが存在しない、読み込めない、対応しない、期限情報を検証できない場合は `ERR_SSL_CERT_GENERATION_FAILED` とする。

SSL状態が `disabled`、`pending`、`challenge_ready`、`failed` の場合、証明書ファイルまたは秘密鍵ファイルが存在しないことだけを理由に起動失敗または状態確認失敗としてはならない。

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
- Webhook失敗時の自動リトライは Rev.90 時点では実装しない
- GitHub側からの再送は通常のWebhook受信として扱い、冪等キーで重複判定する

Webhook署名検証は、JSON decode 前のリクエストBody生バイト列に対して行う。

署名検証に失敗したリクエスト、payload形式不正のリクエスト、対象外event、対象外branchは、`config/webhooks.json` に冪等履歴を追加してはならない。

デプロイ対象ファイルを `contents/` へ反映した後は、反映後のファイル集合から `files.json` を再生成する。

Webhookデプロイで生成する `files.json` の各 File object は、`name`、`path`、`size`、`uploadedAt` を必須とし、`path` 昇順で保存する。

Webhookデプロイでは、開発リポジトリ、GitHub payload 内 URL、外部ネットワークをデプロイ先または一時作業先として使用してはならない。

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

バックアップは静的配信復旧に必要な Project 単位データのみを対象とし、`logs/`、`certs/`、`config/auth.json`、`config/acme_*.json`、`config/webhooks.json`、`config/domains.json`、`config/projects.json` を含めてはならない。

atomic rename 後に `config/backups.json` への履歴保存へ失敗した場合は、作成済みtar.gzを削除し、削除失敗時は error log に記録する。

作成済みtar.gzの削除に失敗した場合でも、バックアップ作成APIは成功レスポンスを返してはならない。

**復旧**

- 復旧前にバックアップファイルの存在と SHA-256 ハッシュを検証する
- 復旧前退避先は `storage.basePath/backups/restore-staging/{restoreId}/previous/` とする
- 復旧用展開先は `storage.basePath/backups/restore-staging/{restoreId}/next/` とする
- 復旧対象 Project が存在しない場合は `ERR_PROJECT_NOT_FOUND` を返す
- 同一 `restoreId` の `restore-staging/{restoreId}/` が既に存在する場合は `409 Conflict` と `ERR_BACKUP_RESTORE_CONFLICT` を返す
- 復旧処理は `files.json` と `contents/` のみを置換対象とし、Project定義、Domain定義、SSL証明書、ACME状態、Webhook履歴、ログを置換してはならない
- 復旧成功後に `restore-staging/{restoreId}/` の削除へ失敗した場合は WARN ログへ記録し、復旧成功を取り消してはならない
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
- `/api/monitoring/stats` は管理 HTTPS JSON API として扱い、`X-ASB-Admin-Password` ヘッダーによる単一システム管理者パスワード認証を必須とする
- `/api/monitoring/stats` は request body を受け付けず、body が存在する場合は失敗レスポンスを返す
- `/api/monitoring/stats` の成功レスポンスキーは `cpu`、`memory`、`disk`、`connections`、`requests`、`checkedAt` に固定する
- `cpu` は CPU 使用率を表す `number|null` とし、取得できる場合は 0 以上 100 以下、小数第1位までの数値とする
- `memory` はメモリ使用率を表す `number|null` とし、取得できる場合は 0 以上 100 以下、小数第1位までの数値とする
- `disk` は `storage.basePath` が属するファイルシステムのディスク使用率を表す `number|null` とし、取得できる場合は 0 以上 100 以下、小数第1位までの数値とする
- `connections` は ASB プロセスが把握する現在の HTTP 接続数を表す `number|null` とし、取得できる場合は 0 以上の整数とする
- `requests` は ASB プロセス起動後の累計 HTTP リクエスト数を表す `number|null` とし、取得できる場合は 0 以上の整数とする
- `checkedAt` は監視値を生成した UTC 時刻を RFC3339 秒精度文字列で返し、成功レスポンスでは常に存在する
- `connections` と `requests` は process-local in-memory counter とし、JSON、ログ、キャッシュ、DB、lock file、その他ファイルへ永続化してはならない
- `connections` と `requests` は ASB プロセス再起動により初期化される
- 監視 API はローテーション済みログを読まず、ログファイルから監視値を集計しない
- 監視 API は外部監視サービス、外部メトリクスサービス、外部DB、外部ライブラリに依存してはならない

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

本節は Rev.90 時点の実装契約である。実装者は本節に反する判断をコード側で独自に行ってはならない。

#### 13.14.1 パッケージ境界

実装は以下の Go package 境界に固定する。

| パッケージ | 責務 | 外部副作用 |
|-----------|------|-----------|
| `config` | 設定読み込み、デフォルト適用、起動時検証 | 設定ファイル読み込み |
| `server` | `net/http` サーバー、ルーティング、共通レスポンス | HTTP 入出力 |
| `management` | Project、Domain、SSL 管理 | JSON メタデータ更新 |
| `delivery` | File、静的配信、Webhook | ファイル入出力、JSON メタデータ更新 |
| `data` | Backup、Storage | tar.gz 作成、復旧、JSON メタデータ更新 |
| `system` | Monitoring、Log | ログ読み書き、監視値取得 |

`cmd/asb/main.go` は設定読み込み、依存関係生成、HTTPS サーバー起動、graceful shutdown のみを行う。

各 package は他 package の具象型に直接依存せず、必要な境界は interface で受け渡す。

#### 13.14.2 HTTP ルーティング契約

HTTP ルーティングは Go 標準 `net/http` で実装する。

外部ルーターライブラリは採用しない。

API パスは `/api/` で始まる。

API パス判定は静的ファイル配信より優先する。

静的ファイル配信は API パスに一致しないリクエストのみを対象とする。

API パスは `strings.TrimPrefix` と `/` 分割により解析し、空セグメント、余分な末尾スラッシュ、想定外セグメントは `404 Not Found` とする。

API パス解析では URL path のみを対象とし、query string と fragment をルーティング判定に使用してはならない。

API パスは URL decode 後に判定する。decode に失敗した場合は `400 Bad Request` とする。

API パスに `//`、`.` セグメント、`..` セグメント、NUL 文字、`\` を含む場合は `404 Not Found` とする。

`/api` は定義済み API パスではないため `404 Not Found` とする。

`/api/` は空セグメントを含むため `404 Not Found` とする。

定義済みパスで HTTP メソッドだけが不一致の場合は `405 Method Not Allowed` とし、`Allow` ヘッダーに許可メソッドを設定する。

#### 13.14.3 HTTP リクエスト契約

JSON API は `Content-Type: application/json` を要求する。

`Content-Type` は media type を小文字化して比較し、parameter を除いた値が `application/json` の場合のみ許可する。

`Content-Type` に parameter が付く場合、`charset=utf-8` のみ許可する。

`charset` の値は大文字小文字を区別せず、`utf-8` として比較する。

`charset` 以外の parameter、空 parameter、重複 parameter、quoted charset、複数の `Content-Type` ヘッダーは `400 Bad Request` とする。

Body を要求する JSON API で `Content-Type` が存在しない場合は `400 Bad Request` とする。

Body を持たない API では Body を読み込まない。

Body を要求する API で空 Body の場合は `400 Bad Request` とする。

JSON API の request body 最大サイズは 1MiB とする。

1MiB を超える JSON API request body は `413 Payload Too Large` とし、`ERR_INVALID_REQUEST` を返す。

JSON decode は `json.Decoder` を使用し、`DisallowUnknownFields` を有効にする。

1リクエストにつき JSON 値は1個のみ許可し、後続トークンが存在する場合は `400 Bad Request` とする。

multipart upload は `POST /api/projects/:id/files/upload` のみ許可し、フィールド名は `file` に固定する。

multipart upload の request body 最大サイズは 1GiB + 1MiB とする。

multipart parse 時に file part が複数存在する場合は `400 Bad Request` とする。

multipart の file part が 0 byte の場合は `400 Bad Request` とする。

multipart の file part が 1GiB を超える場合は `413 Payload Too Large` とし、`ERR_PROJECT_QUOTA_EXCEEDED` を返す。

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

管理 HTTPS JSON API のすべてのレスポンスに `X-Request-Id` を設定する。

管理 HTTPS JSON API の `X-Request-Id` はリクエスト受信時に生成した requestId と一致させる。

`Content-Type` を持つ JSON レスポンスでは `application/json; charset=utf-8` のみを返す。

`204 No Content` は Rev.90 時点では使用しない。

`HEAD` と `304 Not Modified` ではレスポンスボディを返してはならない。

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

JSON encode に失敗した場合は一時ファイルを書き込まず、対象操作を失敗扱いとする。

一時ファイル書き込み、file `fsync`、directory `fsync`、atomic rename のいずれかに失敗した場合は、対象操作を失敗扱いとする。

atomic rename 前に失敗した場合は、作成済み一時ファイルの削除を1回だけ試行する。

一時ファイル削除に失敗した場合は、`Operational warning` として error log へ記録する。

atomic rename 後に directory `fsync` が失敗した場合は、対象操作を失敗扱いとし、成功レスポンスを返してはならない。

保存失敗時は成功レスポンスを返してはならない。

#### 13.14.6 起動時検証契約

起動時検証は HTTPS サーバー起動前に完了する。

ASB サーバー起動コマンドは `asb start` とする。

`asb start` は、起動時検証の途中で不足したディレクトリ、JSON ファイル、証明書ファイル、ログファイル、一時ファイルを作成してはならない。

実行時データ領域の初期化コマンドを提供してはならない。

`asb init-runtime` は Rev.90 時点では禁止コマンドであり、実装してはならない。

ASB 本体、install/update、起動時検証、read API、monitoring API、SDK、Web UI、background job は、実行時データ初期化、不足補完、空 JSON 作成、必須 directory 作成を行ってはならない。

実行時データ領域と必須 JSON ファイルは、ASB 起動前に運用者が用意する。

ASB は、既存 JSON ファイルが存在する場合、JSON 構文、`schemaVersion: 1`、必須トップレベルキー、未知フィールド不在を検証し、妥当な場合のみ利用する。

ASB は、既存 JSON ファイルが不正な場合、自動修復または上書きを行わず `ERR_STORAGE_VALIDATION_FAILED` で失敗しなければならない。

起動時検証は以下の順序で実行する。

1. 起動設定ファイル `config.json` を読み込む
2. 設定未知フィールド、型、値範囲を検証する
3. `storage.basePath` を絶対パスへ正規化する
4. `storage.basePath` が開発リポジトリ配下でないことを検証する
5. `storage.basePath/` の存在、directory 種別、読み書き権限を検証する
6. `storage.basePath/config/` の存在、directory 種別、読み書き権限を検証する
7. `storage.basePath/storage/` の存在、directory 種別、読み書き権限を検証する
8. `storage.basePath/storage/projects/` の存在、directory 種別、読み書き権限を検証する
9. `storage.basePath/logs/` の存在、directory 種別、読み書き権限を検証する
10. `storage.basePath/certs/` の存在、directory 種別、読み書き権限を検証する
11. 必須 JSON ファイルの存在、通常ファイル種別、読み書き権限を検証する
12. 必須 JSON ファイルの JSON 構文、`schemaVersion: 1`、必須トップレベルキー、未知フィールド不在を検証する

起動時に存在しなければならない必須 JSON ファイルは以下に限定する。

- `storage.basePath/config/projects.json`
- `storage.basePath/config/domains.json`
- `storage.basePath/config/backups.json`
- `storage.basePath/config/auth.json`
- `storage.basePath/config/webhooks.json`
- `storage.basePath/config/acme_accounts.json`
- `storage.basePath/config/acme_orders.json`
- `storage.basePath/config/acme_authorizations.json`
- `storage.basePath/config/acme_challenges.json`
- `storage.basePath/config/acme_renewals.json`

Project 作成前に `storage.basePath/storage/projects/{projectId}/` および `storage.basePath/storage/projects/{projectId}/files.json` が存在してはならない。

Project 作成 API は、Project 作成処理の一部として `storage.basePath/storage/projects/{projectId}/`、`storage.basePath/storage/projects/{projectId}/contents/`、`storage.basePath/storage/projects/{projectId}/files.json` を作成する。

Project 作成 API は、作成済み Project ディレクトリまたは `files.json` が既に存在する場合、既存ファイルを上書きせず `ERR_STORAGE_VALIDATION_FAILED` で失敗しなければならない。

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

途中失敗時はエラーログを記録し、以下の整合性検証を実行する。

- 更新予定だった JSON ファイルが読み込み可能であること
- 更新済み JSON ファイルが JSON 構文、`schemaVersion`、必須トップレベルキーを満たすこと
- `files.json` に記録された `path` が `contents/` 配下に収まること
- `files.json` に記録されたファイル実体が存在し、通常ファイルであり、`size` が一致すること
- `projects.json` の `used` が `files[]` の `size` 合計と一致すること

整合性検証自体に失敗した場合は `ERR_STORAGE_VALIDATION_FAILED` を error log へ記録する。

#### 13.14.9 保留機能の実装禁止契約

Rev.90 時点では以下を実装してはならない。

- SDK 専用プロトコル
- SDK 専用エンドポイント
- SDK 専用セッション
- WebSocket
- gRPC
- GraphQL
- 独自 TCP プロトコル
- MQTT
- APIキー管理
- 複数ユーザー管理
- ロール・権限分離
- セッション管理
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
- DNS-01 challenge
- TLS-ALPN-01 challenge
- wildcard 証明書
- 複数 CA
- CA 選定
- CA failover
- 任意 ACME directory URL
- DNS provider API 連携
- 手動 TXT 登録
- EAB
- ARI
- OCSP stapling
- `webhook.githubSecret` 未設定時のWebhook署名検証必須化
- Webhook失敗時の自動リトライスケジューラー
- 外部DB
- 外部ストレージ連携
- `.gitignore` を必要とする生成物設計

上記を実装する場合は、先に `ASB-spec.md` を改訂し、確定仕様として昇格させる。

### 13.15 実装詳細固定仕様

本節は Rev.90 時点で実装時に固定する詳細仕様である。

#### 13.15.0 実装基盤一括固定仕様

Rev.90 時点の実装は、管理 HTTPS JSON API、JSON ファイルベース保存、静的配信、GitHub Webhook、無料独自SSL、Backup / Restore、Log / Audit を ASB の基盤機能として扱う。

実装者は、以下の基盤仕様を満たすまで対象機能を完了扱いにしてはならない。

| 基盤 | 固定内容 |
|------|----------|
| 管理 HTTPS JSON API | 開発ローカルと本番環境の両方で HTTPS を必須とし、request、response、error、timestamp、pagination、upload の形式を本書で定義した形式へ統一する |
| 認証 | Webhook を除く管理 API では `X-ASB-Admin-Password` を必須とし、token、session、cookie、API key、Basic、Bearer、JWT、OAuth、OIDC を使用しない |
| JSON 保存 | すべての管理データは `storage.basePath` 配下の JSON ファイルとして保存し、外部DB、SQLite、KVS、外部ストレージを使用しない |
| atomic write | JSON 更新は同一ディレクトリ内の一時ファイルへ書き込み、fsync 後に atomic rename する |
| schemaVersion | 保存 JSON はトップレベルに `schemaVersion` を持ち、Rev.90 時点では整数 `1` のみ許可する |
| unknown field | request JSON、config JSON、保存 JSON の未知フィールドは拒否する |
| 静的配信 | Host 解決、path 正規化、`files.json` 記録ファイル限定配信、`GET` / `HEAD`、`404` / `405`、MIME、ETag、Last-Modified、Gzip を実装する |
| GitHub Webhook | Push event、署名検証、branch filter、冪等キー、deploy lock、atomic publish、失敗記録、retry 非作成を実装する |
| 無料独自SSL | Let’s Encrypt ACME v2、HTTP-01、サブドメイン単位の個別証明書、自動取得、自動更新、失敗状態保存を実装する |
| Backup / Restore | Project 単位の静的配信復旧に必要なデータだけを tar.gz と checksum で扱い、restore 前退避、失敗時復元、metadata JSON を実装する |
| Log / Audit | access log、error log、audit log を JSON Lines とし、requestId、rotation、管理 API 閲覧範囲を固定する |
| 生成禁止 | 開発リポジトリ内に実行時データ、一時ファイル、cache、log、build output、`.gitignore` を生成しない |

管理 HTTPS JSON API の実装単位は以下に固定する。

- request body は JSON object または `multipart/form-data` のみ許可する。
- JSON API の `Content-Type` は `application/json` または `application/json; charset=utf-8` のみ許可する。
- JSON response の `Content-Type` は常に `application/json; charset=utf-8` とする。
- request body 上限、multipart 上限、`limit`、`offset`、path parameter、query parameter は本書の検証規則で拒否または受理を決定する。
- 成功 response は object とし、top-level array、文字列、数値、`null` を返してはならない。
- error response は `error`、`code`、`timestamp`、`httpStatus` の4項目に固定する。
- 管理 API で API ごとの独自 error format を作ってはならない。
- Web UI、SDK、将来計画機能を理由に、管理 API の response 形を分岐してはならない。

JSON 保存実装単位は以下に固定する。

- 保存先は `storage.basePath` 配下に限定する。
- 開発リポジトリ配下を `storage.basePath` に指定した場合は起動失敗とする。
- JSON の整形は deterministic とし、object key 順序、配列ソート順、末尾改行の有無を機能ごとに固定する。
- 複数 JSON を更新する操作は、更新順序、途中失敗時の復元、成功レスポンス可否を機能ごとに固定する。
- migration 対象 JSON と migration 対象外 JSON を明確に分ける。
- downgrade をサポートしない場合でも、サポートしないことを仕様として明記する。

静的配信実装単位は以下に固定する。

- `Host` は Domain 解決専用に使い、未割当 Host は `404 Not Found` とする。
- `Host` の正規化、port除去、末尾dot除去、小文字化、不正判定は Host 境界固定仕様に従う。
- `X-Forwarded-Host`、`Forwarded`、`X-Forwarded-For`、`X-Real-IP` は Domain 解決に使用しない。
- API path と静的配信 path の routing は最初に分離し、`/api/` 配下を静的配信へ fallback してはならない。
- path traversal 判定は URL decode 後、path clean 後、実ファイル path 解決後の各段階で行う。
- `Range` は無視し、`206 Partial Content` を返してはならない。
- Brotli は実装しない。
- Gzip は送信時圧縮のみとし、`.gz`、cache、metadata を作成してはならない。

GitHub Webhook 実装単位は以下に固定する。

- Webhook は GitHub Push event 専用とする。
- remote repository URL を信頼して clone、fetch、pull してはならない。
- `deploy.sourcePath` の既存 local checkout と `after` commit の参照可否だけを信頼境界とする。
- deploy 中は対象 Project 単位で排他し、同一 Project の File 操作、Backup 復旧、別 Webhook deploy と競合させない。
- delivery の重複は `branch + ":" + after` で判定する。
- 失敗時に scheduler、queue、background retry worker を生成してはならない。

無料独自SSL 実装単位は以下に固定する。

- ACME は Let’s Encrypt ACME v2 のみとする。
- challenge は HTTP-01 のみとする。
- wildcard、DNS-01、TLS-ALPN-01、DNS provider API、複数 CA、任意 ACME directory URL は実装しない。
- 証明書は Domain 単位で保存し、wildcard 証明書で複数 Domain を代表させてはならない。
- renewal は Domain 単位で判定し、既存有効証明書を失敗時に削除してはならない。

Backup / Restore 実装単位は以下に固定する。

- Backup 対象は Project の `files.json` と `contents/` に限定する。
- Project 定義、Domain 定義、SSL 証明書、ACME 状態、Webhook 履歴、Log、Auth は Backup 対象外とする。
- Restore は対象 Project の静的配信状態だけを置換する。
- Restore 前に現行 `files.json` と `contents/` を退避し、失敗時は退避から復元する。
- Backup archive は path traversal、absolute path、symlink、device file、unknown entry を拒否する。

Log / Audit 実装単位は以下に固定する。

- access log は API、静的配信、Webhook、ACME challenge handler の HTTP request を対象とする。
- error log は処理失敗、検証失敗、整合性不備、運用警告を対象とする。
- audit log は管理者パスワード変更、Project 作成/削除、Domain 追加/削除、File upload/delete、SSL enable/renew/disable、Backup restore、Webhook deploy 成功/失敗を対象とする。
- log は JSON Lines とし、1行1 JSON object、UTF-8、UTC RFC3339 秒精度に固定する。
- log rotation は `log.maxSize` を超える前後で実行し、欠損または重複が発生しないようにする。
- log API は password 認証対象とし、内部パス、secret、password hash、ACME account key を返してはならない。

#### 13.15.1 API エンドポイント固定表

| API | Method | Path | Body | Query | 成功 | 主な失敗 |
|-----|--------|------|------|-------|------|----------|
| Project 作成 | POST | `/api/projects` | `{"name":string,"quota":number?}` | なし | 201 Project | 400, 409, 500 |
| Project 一覧 | GET | `/api/projects` | なし | なし | 200 `{projects:[]}` | 500 |
| Project 削除 | DELETE | `/api/projects/:id` | なし | なし | 200 `{status,projectId,deletedAt}` | 404, 500 |
| Domain 追加 | POST | `/api/projects/:id/domains` | `{"domain":string}` | なし | 201 Domain | 400, 404, 409, 500 |
| Domain 一覧 | GET | `/api/projects/:id/domains` | なし | なし | 200 `{domains:[]}` | 404, 500 |
| Domain 削除 | DELETE | `/api/projects/:id/domains/:domain` | なし | なし | 200 `{status,projectId,domain,deletedAt}` | 404, 500 |
| 無料独自SSL 有効化 | POST | `/api/projects/:id/domains/:domain/ssl/enable` | なし | なし | 200 SSLStatus | 400, 404, 500 |
| 無料独自SSL 状態確認 | GET | `/api/projects/:id/domains/:domain/ssl` | なし | なし | 200 SSLStatus | 404, 500 |
| 無料独自SSL 更新 | POST | `/api/projects/:id/domains/:domain/ssl/renew` | なし | なし | 200 SSLStatus | 400, 404, 409, 500 |
| 無料独自SSL 無効化 | POST | `/api/projects/:id/domains/:domain/ssl/disable` | なし | なし | 200 SSLStatus | 404, 500 |
| File upload | POST | `/api/projects/:id/files/upload` | multipart `file` | なし | 201 File | 400, 404, 413, 500 |
| File 一覧 | GET | `/api/projects/:id/files` | なし | なし | 200 `{files:[]}` | 404, 500 |
| File 削除 | DELETE | `/api/projects/:id/files/:name` | なし | なし | 200 `{status,projectId,fileName,deletedAt}` | 404, 500 |
| Backup 一覧 | GET | `/api/backups` | なし | なし | 200 `{backups:[]}` | 500 |
| Backup 復旧 | POST | `/api/backups/restore/:id` | なし | なし | 200 `{status,backupId,restoredAt}` | 404, 409, 500 |
| 管理者パスワード変更 | POST | `/api/auth/change-password` | `{"currentPassword":string,"newPassword":string}` | なし | 200 `{status,updatedAt}` | 400, 401, 500 |
| Monitoring | GET | `/api/monitoring/stats` | なし | なし | 200 Monitoring | 500 |
| Access log | GET | `/api/logs/access` | なし | `limit`,`offset` | 200 `{logs:[]}` | 400, 500 |
| Error log | GET | `/api/logs/error` | なし | `limit`,`offset` | 200 `{logs:[]}` | 400, 500 |
| GitHub Webhook | POST | `/api/webhook/github` | GitHub Push JSON | なし | 200 `{status}` | 400, 500 |

管理 API の主な失敗には、上記表に加えて `401 ERR_AUTH_FAILED` および `403 ERR_AUTH_PASSWORD_CHANGE_REQUIRED` を含める。ただし `POST /api/webhook/github` は対象外とする。

`:id` は UUID 形式の文字列のみ許可する。

`:domain` は URL decode 後に小文字正規化し、ドメイン検証を行う。

`:name` は URL decode 後にファイル名検証を行い、パス区切り文字を含む値は拒否する。

`limit` は未指定時 `100`、最小 `1`、最大 `1000` とする。

`offset` は未指定時 `0`、最小 `0` とする。

`limit` または `offset` が整数として解釈できない場合は `400 Bad Request` とする。

`limit` が `1` 未満または `1000` を超える場合は `400 Bad Request` とする。

`offset` が `0` 未満の場合は `400 Bad Request` とする。

#### 13.15.1.1 API 個別実装契約

各 API は以下の request schema、保存先、更新順序、audit log 対象に従う。

| API | Request schema | Validation | 保存先 | 更新順序 | Audit |
|-----|----------------|------------|--------|----------|-------|
| `POST /api/projects` | `name` required string、`quota` optional integer | `name` は Project 名規則、`quota` は `0` より大きく `storage.maxProjectSize` 以下 | `config/projects.json`、`storage/projects/{projectId}/files.json` | request検証、重複確認、Project ID生成、Project directory作成、`contents/`作成、`files.json`作成、`projects.json`保存 | yes |
| `GET /api/projects` | bodyなし | bodyが存在する場合は拒否 | なし | `projects.json`読込、schema検証、sort確認、response生成 | no |
| `DELETE /api/projects/:id` | bodyなし | `:id` UUID、Project存在 | `config/projects.json`、`config/domains.json`、`config/backups.json`、Project directory | Project検証、関連Domain列挙、関連Backup列挙、`files.json`検証、Project directory削除、`domains.json`保存、`backups.json`保存、`projects.json`保存 | yes |
| `POST /api/projects/:id/domains` | `domain` required string | `:id` UUID、Project存在、Domain正規化、Domain未割当 | `config/domains.json` | Project検証、Domain正規化、重複確認、Domain object追加、`domains.json`保存 | yes |
| `GET /api/projects/:id/domains` | bodyなし | `:id` UUID、Project存在 | なし | `domains.json`読込、Project所属filter、domain昇順response生成 | no |
| `DELETE /api/projects/:id/domains/:domain` | bodyなし | Project存在、Domain存在、SSL操作中でない | `config/domains.json` | Project検証、Domain検証、SSL状態確認、Domain object削除、`domains.json`保存 | yes |
| `POST /api/projects/:id/files/upload` | `multipart/form-data`、part名 `file` required | Project存在、file単数、filename/path/size/quota検証 | `storage/projects/{projectId}/contents/`、`files.json`、`projects.json` | request検証、一時file書込、fsync、公開先rename、`files.json`保存、`projects.json`保存 | yes |
| `GET /api/projects/:id/files` | bodyなし | Project存在、`files.json`整合 | なし | `files.json`読込、実体整合検証、name昇順response生成 | no |
| `DELETE /api/projects/:id/files/:name` | bodyなし | Project存在、file存在、path安全 | `contents/`、`files.json`、`projects.json` | file検出、実体削除、`files.json`保存、`projects.json` used保存 | yes |
| `POST /api/projects/:id/domains/:domain/ssl/enable` | bodyなし | Domain存在、状態が `disabled` または `failed` なら開始可能 | `config/acme_*.json`、`certs/{domain}/`、`domains.json` | 状態確認、account/order作成、challenge保存、証明書保存、検証、状態保存 | yes |
| `GET /api/projects/:id/domains/:domain/ssl` | bodyなし | Domain存在 | なし | Domain SSL状態読込、証明書必要時検証、SSLStatus生成 | no |
| `POST /api/projects/:id/domains/:domain/ssl/renew` | bodyなし | Domain存在、状態が `issued` または `expired` | `config/acme_*.json`、`certs/{domain}/`、`domains.json` | 状態確認、renewal作成、challenge保存、新証明書保存、検証、状態保存 | yes |
| `POST /api/projects/:id/domains/:domain/ssl/disable` | bodyなし | Domain存在 | `domains.json` | 状態確認、`disabled`保存、既存証明書は即時削除しない | yes |
| `GET /api/backups` | bodyなし | bodyが存在する場合は拒否 | なし | `backups.json`読込、createdAt降順response生成 | no |
| `POST /api/backups/restore/:id` | bodyなし | Backup存在、status `completed`、checksum一致、archive安全 | Project `files.json`、`contents/` | backup検証、現行退避、archive展開、検証、公開置換、失敗時復元 | yes |
| `POST /api/auth/change-password` | `currentPassword` required string、`newPassword` required string | current一致、new強度、初期password変更必須状態 | `config/auth.json` | request検証、current検証、new hash生成、`auth.json`保存 | yes |
| `GET /api/monitoring/stats` | bodyなし | bodyが存在する場合は拒否 | なし | 実行時JSON/OS情報読込、取得不可値は `null`、response生成 | no |
| `GET /api/logs/access` | bodyなし、query `limit`,`offset` | limit/offset検証、JSON Lines全行decode | なし | access.log読込、全行検証、offset/limit適用 | no |
| `GET /api/logs/error` | bodyなし、query `limit`,`offset` | limit/offset検証、JSON Lines全行decode | なし | error.log読込、全行検証、offset/limit適用 | no |
| `POST /api/webhook/github` | GitHub Push JSON | event、署名、ref、after、sourcePath、Project検証 | `config/webhooks.json`、Project `contents/`、`files.json` | event確認、署名検証、payload検証、冪等確認、tree列挙、atomic publish、`files.json`保存、`webhooks.json`保存 | yes |

bodyなし API に request body が存在する場合は、`400 Bad Request` とし `ERR_INVALID_REQUEST` を返す。

request schema にない JSON field は `400 Bad Request` とし `ERR_UNKNOWN_FIELD` を返す。

保存先が「なし」の API は、成功時に JSON、ファイル、ディレクトリ、ログ以外の実行時データを作成または更新してはならない。

Audit が `yes` の API は、成功、拒否、途中失敗を audit log へ記録する。

Webhook は管理者パスワード認証の対象外だが、署名検証または明示的な secret 未設定方針に従う。

#### 13.15.1.2 API 別失敗条件固定表

各 API は以下の失敗条件と HTTP status / error code の組み合わせを固定する。

| API | 条件 | HTTP | code |
|-----|------|------|------|
| 全 JSON API | JSON 構文不正 | 400 | `ERR_INVALID_JSON` |
| 全 JSON API | request body が空、top-level object 以外、後続 token あり | 400 | `ERR_INVALID_REQUEST` |
| 全 JSON API | request schema に存在しない field | 400 | `ERR_UNKNOWN_FIELD` |
| 全 JSON API | `Content-Type` 不正 | 400 | `ERR_INVALID_REQUEST` |
| 全 JSON API | request body が 1MiB 超過 | 413 | `ERR_INVALID_REQUEST` |
| 全 管理 API | `X-ASB-Admin-Password` 未指定、空文字、不一致 | 401 | `ERR_AUTH_FAILED` |
| 全 管理 API | 初期デフォルトパスワード未変更かつ password change 以外 | 403 | `ERR_AUTH_PASSWORD_CHANGE_REQUIRED` |
| `POST /api/projects` | `name` 不正、`quota` 不正 | 400 | `ERR_INVALID_REQUEST` |
| `POST /api/projects` | Project 名重複 | 409 | `ERR_PROJECT_ALREADY_EXISTS` |
| `POST /api/projects` | Project directory または `files.json` 既存 | 500 | `ERR_STORAGE_VALIDATION_FAILED` |
| `GET /api/projects` | `projects.json` 読込、構文、schema、権限不備 | 500 | `ERR_STORAGE_VALIDATION_FAILED` |
| `DELETE /api/projects/:id` | `:id` 不正 | 400 | `ERR_INVALID_REQUEST` |
| `DELETE /api/projects/:id` | Project 不在 | 404 | `ERR_PROJECT_NOT_FOUND` |
| `DELETE /api/projects/:id` | 関連 JSON または Project directory 削除失敗 | 500 | `ERR_STORAGE_VALIDATION_FAILED` |
| `POST /api/projects/:id/domains` | `:id` または `domain` 不正 | 400 | `ERR_INVALID_REQUEST` |
| `POST /api/projects/:id/domains` | Project 不在 | 404 | `ERR_PROJECT_NOT_FOUND` |
| `POST /api/projects/:id/domains` | Domain 割当済み | 409 | `ERR_DOMAIN_ALREADY_ASSIGNED` |
| `POST /api/projects/:id/domains` | `domains.json` または `projects.json` 保存失敗 | 500 | `ERR_STORAGE_VALIDATION_FAILED` |
| `GET /api/projects/:id/domains` | `:id` 不正 | 400 | `ERR_INVALID_REQUEST` |
| `GET /api/projects/:id/domains` | Project 不在 | 404 | `ERR_PROJECT_NOT_FOUND` |
| `GET /api/projects/:id/domains` | `domains.json` 読込、構文、schema、権限不備 | 500 | `ERR_STORAGE_VALIDATION_FAILED` |
| `DELETE /api/projects/:id/domains/:domain` | `:id` または `:domain` 不正 | 400 | `ERR_INVALID_REQUEST` |
| `DELETE /api/projects/:id/domains/:domain` | Project 不在または Domain 不在 | 404 | `ERR_DOMAIN_NOT_FOUND` |
| `DELETE /api/projects/:id/domains/:domain` | SSL 状態が `pending`、`challenge_ready`、`renewing` | 409 | `ERR_SSL_OPERATION_CONFLICT` |
| `DELETE /api/projects/:id/domains/:domain` | `domains.json` または `projects.json` 保存失敗 | 500 | `ERR_STORAGE_VALIDATION_FAILED` |
| `POST /api/projects/:id/files/upload` | `:id`、multipart、filename、path、file part 不正 | 400 | `ERR_INVALID_REQUEST` |
| `POST /api/projects/:id/files/upload` | Project 不在 | 404 | `ERR_PROJECT_NOT_FOUND` |
| `POST /api/projects/:id/files/upload` | upload 後容量が quota 超過 | 413 | `ERR_PROJECT_QUOTA_EXCEEDED` |
| `POST /api/projects/:id/files/upload` | file 実体保存、rename、metadata 更新失敗 | 500 | `ERR_FILE_UPLOAD_FAILED` |
| `POST /api/projects/:id/files/upload` | `files.json` と `contents/` の整合性不備 | 500 | `ERR_STORAGE_VALIDATION_FAILED` |
| `GET /api/projects/:id/files` | `:id` 不正 | 400 | `ERR_INVALID_REQUEST` |
| `GET /api/projects/:id/files` | Project 不在 | 404 | `ERR_PROJECT_NOT_FOUND` |
| `GET /api/projects/:id/files` | `files.json` と `contents/` の整合性不備 | 500 | `ERR_STORAGE_VALIDATION_FAILED` |
| `DELETE /api/projects/:id/files/:name` | `:id` または `:name` 不正 | 400 | `ERR_INVALID_REQUEST` |
| `DELETE /api/projects/:id/files/:name` | Project 不在 | 404 | `ERR_PROJECT_NOT_FOUND` |
| `DELETE /api/projects/:id/files/:name` | File 不在 | 404 | `ERR_FILE_NOT_FOUND` |
| `DELETE /api/projects/:id/files/:name` | file 削除後の JSON 更新失敗 | 500 | `ERR_STORAGE_VALIDATION_FAILED` |
| SSL API 共通 | `:id` または `:domain` 不正 | 400 | `ERR_INVALID_REQUEST` |
| SSL API 共通 | Project または Domain 不在 | 404 | `ERR_DOMAIN_NOT_FOUND` |
| SSL API 共通 | 証明書本文、秘密鍵、有効期限、path、permission の検証失敗 | 500 | `ERR_SSL_CERT_GENERATION_FAILED` |
| `POST /api/projects/:id/domains/:domain/ssl/renew` | 状態が `disabled`、`pending`、`challenge_ready`、`renewing` | 409 | `ERR_SSL_OPERATION_CONFLICT` |
| SSL enable / renew | ACME account、order、authorization、challenge、certificate 保存失敗 | 500 | `ERR_SSL_CERT_GENERATION_FAILED` |
| `GET /api/backups` | `backups.json` 読込、構文、schema、権限不備 | 500 | `ERR_STORAGE_VALIDATION_FAILED` |
| `POST /api/backups/restore/:id` | `:id` 不正 | 400 | `ERR_INVALID_REQUEST` |
| `POST /api/backups/restore/:id` | Backup 不在 | 404 | `ERR_BACKUP_NOT_FOUND` |
| `POST /api/backups/restore/:id` | Backup status が `completed` 以外、staging 競合 | 409 | `ERR_BACKUP_RESTORE_CONFLICT` |
| `POST /api/backups/restore/:id` | checksum 不一致、archive 不正、展開失敗、復元失敗 | 500 | `ERR_BACKUP_RESTORE_FAILED` |
| `POST /api/auth/change-password` | request schema、password 強度不正 | 400 | `ERR_INVALID_REQUEST` |
| `POST /api/auth/change-password` | `currentPassword` 不一致 | 401 | `ERR_AUTH_FAILED` |
| `POST /api/auth/change-password` | `auth.json` 保存失敗 | 500 | `ERR_STORAGE_VALIDATION_FAILED` |
| `GET /api/monitoring/stats` | 監視値の一部取得不可 | 200 | なし |
| `GET /api/monitoring/stats` | response 生成不能、必須実行時 JSON 読込不能 | 500 | `ERR_INTERNAL` |
| Log API 共通 | `limit` / `offset` 不正 | 400 | `ERR_INVALID_REQUEST` |
| Log API 共通 | ログファイル読込、JSON Lines decode、schema 検証失敗 | 500 | `ERR_LOG_READ_FAILED` |
| `POST /api/webhook/github` | 署名ヘッダー欠落、形式不正、署名不一致 | 401 | `ERR_WEBHOOK_SIGNATURE_INVALID` |
| `POST /api/webhook/github` | JSON 構文不正、必須 payload 欠落 | 400 | `ERR_INVALID_JSON` または `ERR_INVALID_REQUEST` |
| `POST /api/webhook/github` | `deploy.projectId` 空文字 | 500 | `ERR_WEBHOOK_PROJECT_NOT_CONFIGURED` |
| `POST /api/webhook/github` | `deploy.sourcePath`、Git worktree、`after` commit、対象 path 不正 | 500 | `ERR_WEBHOOK_SOURCE_INVALID` |
| `POST /api/webhook/github` | deploy 中の copy、rename、`files.json` 保存、`webhooks.json` 保存失敗 | 500 | `ERR_WEBHOOK_PROCESSING_FAILED` |

静的配信は管理 API ではないため JSON error response を返さない。

静的配信の失敗条件は以下に固定する。

| 条件 | HTTP | body | Log |
|------|------|------|-----|
| 未割当 Host、不正 Host、存在しない path、path traversal、未記録 file | 404 | 空または固定HTML | access log |
| `GET` / `HEAD` 以外 | 405 | 空または固定HTML | access log |
| `files.json` 記録済み file の実体欠落、通常ファイル以外、size 不一致、読込失敗 | 500 | 空または固定HTML | access log と error log |
| Gzip 開始後失敗 | 接続終了 | なし | error log |

#### 13.15.1.3 成功レスポンス禁止条件

以下の条件では、いかなる API も成功レスポンスを返してはならない。

- request の構文、Content-Type、path parameter、query parameter、body schema、unknown field のいずれかが不正。
- Webhook を除く管理 API で認証が成立していない。
- 初期デフォルトパスワードが未変更で、対象 API が `POST /api/auth/change-password` ではない。
- 保存 JSON の読み込み、schema 検証、unknown field 検証、保存前再検証に失敗した。
- JSON encode、一時ファイル書き込み、file fsync、directory fsync、atomic rename に失敗した。
- ファイル実体、証明書、backup archive、restore staging の作成、検証、rename、削除、復元に失敗し、仕様上 success として扱う例外に該当しない。
- audit log 対象操作で、成功レスポンス送信前に必要な audit log 記録へ失敗した。
- access log 書き込みを成功レスポンス送信前に行う仕様の API で、access log 書き込みまたは rotation に失敗した。
- 複数 JSON 更新の最終保存が完了していない。
- 複数ファイル更新の公開状態と保存 JSON の整合性が確定していない。

以下の条件は、処理結果が確定している場合に限り成功レスポンスを維持してよい。

- 成功済み Webhook deploy の `deploy-staging/{deployId}/` 削除失敗。
- 成功済み Backup restore の `restore-staging/{restoreId}/` 削除失敗。
- File overwrite 成功後の旧退避ファイル削除失敗。
- 監視 API の一部 OS 情報取得不可。
- Gzip 圧縮開始前の失敗による未圧縮 `200 OK` fallback。

上記例外は必ず error log に `level: "WARN"`、`code: ""`、`message: "Operational warning"` として記録する。

#### 13.15.2 設定値固定表

| 設定 | 必須 | デフォルト | 許容範囲 | 起動失敗条件 |
|------|------|------------|----------|--------------|
| `server.port` | 任意 | `3000` | `1`-`65535` | 範囲外、数値以外 |
| `server.host` | 任意 | `localhost` | `localhost`、`127.0.0.1`、`0.0.0.0` | 空文字、許可外 |
| `server.tlsCertFile` | 任意 | `/etc/asb/tls/server.crt` | 絶対パス | 相対パス、存在しない、通常ファイルでない |
| `server.tlsKeyFile` | 任意 | `/etc/asb/tls/server.key` | 絶対パス | 相対パス、存在しない、通常ファイルでない、group/world writable |
| `server.shutdownTimeout` | 任意 | `30` | `1`-`300` 秒 | 範囲外、数値以外 |
| `storage.basePath` | 任意 | `/var/asb` | 絶対パス | 相対パス、開発リポジトリ配下 |
| `storage.maxProjectSize` | 任意 | `1073741824` | `104857600`-`1099511627776` | 範囲外、数値以外 |
| `ssl.email` | 任意 | `admin@example.com` | email 形式 | 形式不正 |
| `ssl.renewBefore` | 任意 | `7776000` | `86400`-`15552000` 秒 | 範囲外、数値以外 |
| `ssl.renewCheckInterval` | 任意 | `21600` | `3600`-`86400` 秒 | 範囲外、数値以外 |
| `log.level` | 任意 | `info` | `debug`,`info`,`warn`,`error` | 許可外 |
| `log.format` | 任意 | `json` | `json` | `json` 以外 |
| `log.maxSize` | 任意 | `104857600` | `1048576`-`1073741824` | 範囲外、数値以外 |
| `deploy.projectId` | 任意 | 空文字 | UUID形式または空文字 | UUID形式以外 |
| `deploy.sourcePath` | 任意 | `/srv/asb/source` | 絶対パス | 相対パス、存在しない、Git worktreeでない |
| `deploy.branch` | 任意 | `main` | Git ref name | 空文字、空白文字を含む、`..` を含む |
| `webhook.githubSecret` | 任意 | 空文字 | 文字列 | 文字列以外 |

設定ファイルに未知フィールドがある場合は起動失敗とする。

任意項目が未指定の場合はデフォルト値を適用する。

任意項目の親 object が存在し、子項目だけが未指定の場合は、未指定の子項目にデフォルト値を適用する。

任意項目の親 object 自体が未指定の場合は、親 object 全体に属する全項目へデフォルト値を適用する。

必須トップレベル object 以外の object は追加してはならない。

空文字を許可する設定項目は `deploy.projectId` と `webhook.githubSecret` のみとする。

上記以外の string 設定項目に空文字を指定した場合は起動失敗とする。

数値設定項目は JSON number の整数のみ許可し、文字列数値、小数、指数表記、負数、`null` を許可しない。

boolean、array、object を設定値として要求しない項目に指定した場合は起動失敗とする。

`storage.basePath` と `deploy.sourcePath` は絶対パスを `filepath.Clean` 相当で正規化し、正規化後の値を使用する。

`storage.basePath` は開発リポジトリの絶対パスと同一、またはその配下であってはならない。

`deploy.sourcePath` は開発リポジトリ配下を指定できる。ただし Webhook デプロイ処理は開発リポジトリ内へ実行時データ、一時ファイル、ログファイルを作成してはならない。

起動設定ファイル `config.json` の完全形は以下とする。

```json
{
  "server": {
    "port": 3000,
    "host": "localhost",
    "tlsCertFile": "/etc/asb/tls/server.crt",
    "tlsKeyFile": "/etc/asb/tls/server.key",
    "shutdownTimeout": 30
  },
  "storage": {
    "basePath": "/var/asb",
    "maxProjectSize": 1073741824
  },
  "ssl": {
    "email": "admin@example.com",
    "renewBefore": 7776000,
    "renewCheckInterval": 21600
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

起動設定ファイル `config.json` のトップレベルキーは `server`、`storage`、`ssl`、`log`、`deploy`、`webhook` の6個のみ許可する。

各 object 内の許可キーは `13.15.2 設定値固定表` に記載された設定項目のみとする。

未知のトップレベルキー、未知のネストキー、配列、`null`、型不一致は起動失敗とする。

`server.shutdownTimeout`、`ssl.renewBefore`、`ssl.renewCheckInterval`、`log.maxSize` は秒またはバイトを表す整数として扱い、小数を許可しない。

#### 13.15.3 JSON ファイル固定仕様

| ファイル | 空状態 | 必須トップレベル | 更新主体 | 備考 |
|---------|--------|------------------|----------|------|
| `config/projects.json` | `{"schemaVersion":1,"projects":[]}` | `schemaVersion`, `projects` | Project Service | Project 配列を `createdAt` 昇順で保存 |
| `config/domains.json` | `{"schemaVersion":1,"domains":[]}` | `schemaVersion`, `domains` | Domain Service | `domain` は小文字で保存 |
| `config/backups.json` | `{"schemaVersion":1,"backups":[]}` | `schemaVersion`, `backups` | Backup Service | `createdAt` 降順で保存。`path` は `storage.basePath` からの相対パス |
| `config/auth.json` | `{"schemaVersion":1,"admin":{"passwordHash":"","passwordSalt":"","passwordChanged":false,"updatedAt":""}}` | `schemaVersion`, `admin` | Auth Service | 単一システム管理者パスワード認証状態を保存 |
| `storage/projects/:projectId/files.json` | `{"schemaVersion":1,"files":[]}` | `schemaVersion`, `files` | File Service | `name` 昇順で保存 |
| `config/webhooks.json` | `{"schemaVersion":1,"events":[]}` | `schemaVersion`, `events` | Webhook Service | 冪等キー、処理状態、失敗コードを保存 |
| `config/acme_accounts.json` | `{"schemaVersion":1,"accounts":[]}` | `schemaVersion`, `accounts` | SSL Service | Let’s Encrypt ACME account と account key を保存 |
| `config/acme_orders.json` | `{"schemaVersion":1,"orders":[]}` | `schemaVersion`, `orders` | SSL Service | ACME order 状態を保存 |
| `config/acme_authorizations.json` | `{"schemaVersion":1,"authorizations":[]}` | `schemaVersion`, `authorizations` | SSL Service | ACME authorization 状態を保存 |
| `config/acme_challenges.json` | `{"schemaVersion":1,"challenges":[]}` | `schemaVersion`, `challenges` | SSL Service | HTTP-01 challenge 状態を保存 |
| `config/acme_renewals.json` | `{"schemaVersion":1,"renewals":[]}` | `schemaVersion`, `renewals` | SSL Service | 証明書更新履歴、失敗履歴、次回再試行時刻を保存 |

`config/projects.json`、`config/domains.json`、`config/backups.json`、`config/auth.json`、`config/webhooks.json`、`config/acme_accounts.json`、`config/acme_orders.json`、`config/acme_authorizations.json`、`config/acme_challenges.json`、`config/acme_renewals.json` は起動時に存在していなければならない。

ASB は上記 JSON ファイルを起動時または初期化コマンドで作成してはならない。

`storage/projects/:projectId/files.json` は Project 作成 API によって Project ディレクトリと同時に作成する。

Project 作成 API 以外の処理は、`storage/projects/:projectId/files.json` の初回作成を行ってはならない。

Project 作成 API は `storage/projects/:projectId/`、`storage/projects/:projectId/contents/`、`storage/projects/:projectId/files.json` の新規作成のみを行う。既存 path がある場合は上書きせず失敗する。

マイグレーションは既存 JSON のスキーマ更新のみを行い、不足 JSON の初回作成を行ってはならない。

`config/webhooks.json` は Webhook 冪等性管理に使用する実行時 JSON ファイルであり、外部DBを使用しない。

上記 JSON ファイルのトップレベルには、表に記載された必須トップレベルキー以外を保存してはならない。

`schemaVersion` は整数 `1` のみ許可する。

配列キーの値は配列のみ許可し、`null`、object、文字列、数値を許可しない。

#### 13.15.4 静的配信固定仕様

静的配信は API パスに一致しない HTTP リクエストのうち、`GET` または `HEAD` のみ対象とする。

API パスとは `/api/` で始まるパスを指す。

`GET` または `HEAD` 以外のメソッドで静的配信対象パスへアクセスした場合は `405 Method Not Allowed` とし、`Allow: GET, HEAD` を返す。

#### 13.15.4.1 Host header / reverse proxy / client IP 境界固定仕様

Rev.90 時点では、ASB は HTTP request の接続元、Domain 解決、アクセスログ記録において、Go標準 `net/http` が提供する `Request.Host` と `Request.RemoteAddr` を正とする。

`Host` header は、静的配信、ACME HTTP-01 challenge、Webhook 受信、管理 API routing のうち、Domain 解決が必要な処理でのみ使用する。

管理 HTTPS JSON API の認証、認可、Rate limiting、監視、SDK通信、Web UI通信、保存JSON選択、実行時データ選択に `Host` header を使用してはならない。

ASB は Rev.90 時点では reverse proxy 自体を内製実装せず、reverse proxy 設定ファイルを生成しない。

ASB は Rev.90 時点では以下を信頼してはならない。

- `X-Forwarded-Host`
- `Forwarded`
- `X-Forwarded-For`
- `X-Real-IP`
- `CF-Connecting-IP`
- `True-Client-IP`
- その他の proxy / CDN 由来 header

上記 header を受信しても、Domain 解決、client IP 判定、認証、ログの `remoteAddr`、監視値、Webhook 判定、ACME HTTP-01 challenge 判定に使用してはならない。

上記 header の存在だけを理由に request を拒否してはならない。

`Host` は以下の順序で正規化する。

1. `Request.Host` を取得する
2. 空文字の場合は不正 Host とする
3. ASCII 制御文字、空白文字、`/`、`\`、`@`、`#`、`?` を含む場合は不正 Host とする
4. IPv6 literal の bracket 形式 `[::1]` または `[::1]:port` は Rev.90 時点では Domain 解決対象外として不正 Host とする
5. `host:port` 形式の場合は port 部分を除去する
6. port が存在する場合は 1 以上 65535 以下の10進整数のみ許可する
7. 末尾の `.` を1つだけ除去する
8. 小文字 ASCII へ正規化する
9. Domain validation と同一の label 数、全体長、label 正規表現で検証する
10. `config/domains.json` の `domain` と完全一致する Domain を検索する

port を除去した後の Host が空文字になる場合、不正 Host とする。

複数 Host header を受信した場合は不正 Host として扱う。

不正 Host、未割当 Host、Domain に一致しない Host は、静的配信と ACME HTTP-01 challenge では `404 Not Found` とする。

不正 Host による `404 Not Found` は、path traversal、未記録 file、未割当 Domain と同じ外部向け応答とし、詳細理由をレスポンス本文へ出してはならない。

Webhook 受信では、Host は署名検証、対象 Project 判定、branch 判定、deploy source 判定に使用しない。

ACME HTTP-01 challenge では、正規化後 Host を Domain として扱い、challenge token の探索対象 Domain を決定する。

ACME HTTP-01 challenge では、`X-Forwarded-Host` を challenge 対象 Domain として使用してはならない。

アクセスログの `remoteAddr` は `Request.RemoteAddr` から取得した値を正とする。

`Request.RemoteAddr` が `ip:port` の場合、ログの `remoteAddr` は IP 部分のみを記録する。

`Request.RemoteAddr` が port なし、Unix socket 表記、parse不能、空文字の場合、ログの `remoteAddr` は元の文字列をそのまま記録する。ただし空文字の場合は `unknown` とする。

アクセスログの `remoteAddr` は proxy header 由来の値で上書きしてはならない。

Rev.90 時点では trusted proxy list、proxy subnet、real IP middleware、forwarded header parser、proxy protocol、client IP override を実装してはならない。

Reverse proxy を ASB 前段に置く場合、proxy 側で Host header を ASB へ正しく転送することは運用責務とする。

Reverse proxy 経由で実クライアントIPを保持したい場合でも、Rev.90 時点の ASB は proxy header を信頼せず、`remoteAddr` には ASB から見た接続元を記録する。

Host header / reverse proxy / client IP 境界を理由に、開発リポジトリ内または `storage.basePath` 配下へ proxy 設定JSON、trusted proxy JSON、client IP cache、forwarded header log、proxy状態ファイルを生成してはならない。

#### 13.15.4.2 サーバー実行境界固定仕様

Rev.90 時点では、ASB 本体の HTTP server 実行境界を本節に固定する。

ASB 本体は Go 標準 `net/http` の `http.Server` を使用し、外部 HTTP server framework、外部 router、外部 middleware framework を採用してはならない。

`http.Server` の timeout は以下に固定する。

| 項目 | 値 | 用途 |
|------|----|------|
| `ReadHeaderTimeout` | 10秒 | request header 読み込み上限 |
| `ReadTimeout` | 30秒 | header と body を含む request 読み込み上限 |
| `WriteTimeout` | 60秒 | response 書き込み上限 |
| `IdleTimeout` | 120秒 | keep-alive idle 接続上限 |
| `Shutdown` timeout | `server.shutdownTimeout` 秒 | graceful shutdown 待機上限 |

`server.shutdownTimeout` は起動設定で 1 以上 300 以下の整数秒のみ許可し、既定値は 30 秒とする。

timeout 値を API、Project、Domain、File、Webhook、SSL、Log、Monitoring、SDK、Web UI ごとに分岐してはならない。

timeout 発生時は、response 送信前であれば `500 Internal Server Error` と `ERR_INTERNAL` を返す。

response 送信開始後に timeout、client disconnect、write error が発生した場合、成功レスポンスとして扱ってはならない。

response 送信開始後にエラーレスポンスへ差し替えられない場合は、可能な範囲で error log に `ERR_INTERNAL` を記録し、追加の補完 response を送信しない。

client disconnect は、request context cancellation または response write error として扱う。

client disconnect 発生後は、新規の保存 JSON 更新、実体ファイル公開、backup 完了記録、SSL状態完了記録、Webhook成功記録、ログローテーション補完を開始してはならない。

client disconnect 前に保存状態変更が完了済みの場合は、仕様で定義された整合性検証と error log 記録のみを行い、自動ロールバック可否は各機能の失敗時契約に従う。

panic recovery は `internal/server/middleware.go` で実装する。

panic recovery は ASB process を継続させ、response 未送信であれば `500 Internal Server Error` と `ERR_INTERNAL` を返す。

panic recovery の response body、error log、stderr、stdout に stack trace、内部ファイルパス、環境変数、秘密情報、request body、管理者パスワード、webhook secret、private key、ACME token を出力してはならない。

panic recovery は panic 発生を error log へ `ERR_INTERNAL` として記録する。ただし error log 書き込みに失敗した場合でも、別のログファイル、panic dump、一時ファイル、cache を作成してはならない。

graceful shutdown は `SIGINT` または `SIGTERM` を受信した場合に開始する。

graceful shutdown 開始後は新規接続を受け付けず、処理中 request は `server.shutdownTimeout` まで完了を待つ。

`server.shutdownTimeout` 超過後は処理中 request の context を cancel し、未完了操作を成功扱いしてはならない。

graceful shutdown 中に新規 background job、retry queue、scheduler、timeout state、shutdown state file を作成してはならない。

JSON API の request body 最大サイズは 1MiB とする。

multipart upload の request body 最大サイズは 1GiB + 1MiB とする。

multipart の file part 最大サイズは 1GiB とする。

request body サイズ超過は、JSON API では `413 Payload Too Large` と `ERR_INVALID_REQUEST` を返す。

multipart request 全体が 1GiB + 1MiB を超えた場合は、`413 Payload Too Large` と `ERR_INVALID_REQUEST` を返す。

multipart の file part が 1GiB を超えた場合は、`413 Payload Too Large` と `ERR_PROJECT_QUOTA_EXCEEDED` を返す。

Project quota 超過は、request body サイズ上限を満たした後に判定し、`413 Payload Too Large` と `ERR_PROJECT_QUOTA_EXCEEDED` を返す。

同一 request で request body サイズ超過と Project quota 超過の両方が成立する場合は、request body サイズ超過を優先する。

Rev.90 時点では、ASB 管理 HTTPS JSON API に CORS を実装しない。

ASB 管理 HTTPS JSON API は `Access-Control-Allow-Origin`、`Access-Control-Allow-Methods`、`Access-Control-Allow-Headers`、`Access-Control-Allow-Credentials`、`Access-Control-Max-Age` を返してはならない。

ASB 管理 HTTPS JSON API は CORS preflight 用の `OPTIONS` を許可 method として追加してはならない。

管理 API の定義済み path に `OPTIONS` が送信された場合は `405 Method Not Allowed` と `ERR_METHOD_NOT_ALLOWED` を返す。

`Origin` header の存在だけを理由に request を拒否してはならない。

`Origin` header を認証、認可、Domain 解決、Rate limiting、ログ分類、保存JSON選択、SDK通信、Web UI通信の判断に使用してはならない。

ASB 標準Web UI は Rev.90 時点では、ブラウザの same-origin 制約を前提に、利用者が同一 origin または ASB 外部の reverse proxy 等で接続境界を用意する運用を前提とする。

CORS を将来実装する場合は、許可 Origin、preflight、credential、許可 header、許可 method、設定保存、既定値、テスト条件を `ASB-spec.md` で事前に固定する。

Health Check は `/health` に固定する。

`/health` は管理 API ではなく、`/api/` 配下では提供しない。

`/health` は `GET` のみ許可し、`HEAD`、`POST`、`OPTIONS`、その他 method は `405 Method Not Allowed` と `Allow: GET` を返す。

`/health` は認証不要とし、`X-ASB-Admin-Password` を要求してはならない。

`/health` は `200 OK`、`Content-Type: application/json; charset=utf-8`、`Cache-Control: no-store`、body `{"status":"ok"}` を返す。

`/health` は実行時 JSON、ログファイル、証明書ファイル、Project データ、Domain データ、Backup データ、ACME 状態を読んではならない。

`/health` は readiness、liveness、dependency check、storage validation、monitoring stats を兼ねてはならない。

`/health` を契機に実行時データ、cache、一時ファイル、health check 結果JSON、状態ファイル、ログファイルを生成してはならない。

サーバー実行境界を理由に、開発リポジトリ内または `storage.basePath` 配下へ timeout JSON、CORS JSON、health JSON、request size state、client disconnect state、panic dump、retry queue、scheduler state、cache、一時ファイルを生成してはならない。

対象 Project は以下の順序で決定する。

1. Host header / reverse proxy / client IP 境界固定仕様に従って `Host` を正規化する
2. `config/domains.json` の `domain` と完全一致する Domain を検索する
3. Domain に紐づく `projectId` を配信対象 Project とする

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

静的配信は、対象 Project の `files.json` に記録された `path` のみを配信対象とする。

正規化後の相対パスが `files.json` の `files[].path` と一致しない場合、`contents/` 配下に実ファイルが存在しても `404 Not Found` とする。

`contents/` 配下に `files.json` 未記録の通常ファイルが存在する状態は整合性不備であり、起動時検証または整合性検証で `ERR_STORAGE_VALIDATION_FAILED` として扱う。

ディレクトリ自体は配信しない。正規化後の対象がディレクトリの場合、末尾 `/` の有無にかかわらず `index.html` を探索する。

存在しない静的ファイルは `404 Not Found` とする。

`files.json` にメタデータが存在するが実ファイルが存在しない場合は、整合性不備として `500 Internal Server Error` とし、`ERR_STORAGE_VALIDATION_FAILED` を error log へ記録する。

読み込み権限不足、ファイル情報取得失敗、読み込み途中失敗は `500 Internal Server Error` とする。

静的配信レスポンスの `Content-Type` は、Go 標準ライブラリ `mime.TypeByExtension` による拡張子判定を優先する。

`mime.TypeByExtension` が空文字を返した場合は、ファイル先頭最大512 bytesを読み、Go 標準ライブラリ `http.DetectContentType` で判定する。

空ファイルの場合は `application/octet-stream` とする。

`http.DetectContentType` の結果が空文字の場合は `application/octet-stream` とする。

拡張子判定または内容判定で `text/*` が返り、charset parameter が存在しない場合は `; charset=utf-8` を付与する。

`HEAD` は `GET` と同じヘッダーを返し、レスポンスボディを返してはならない。

`ETag` は `"{size}-{unixModifiedTime}"` 形式の弱い validator とし、レスポンスでは `W/"{size}-{unixModifiedTime}"` として返す。

`Last-Modified` は対象ファイルの更新時刻を HTTP-date 形式で返す。

`Cache-Control` は既定で `public, max-age=60` とする。

`If-None-Match` が `ETag` と一致する場合は `304 Not Modified` を返す。

`If-None-Match` と `If-Modified-Since` の両方が存在する場合は、`If-None-Match` を優先する。

`If-None-Match` が存在せず、`If-Modified-Since` が `Last-Modified` 以降の場合は `304 Not Modified` を返す。

`304 Not Modified` ではレスポンスボディを返してはならない。

`304 Not Modified` では `Content-Type`、`ETag`、`Last-Modified`、`Cache-Control` を返し、`Content-Encoding` を返してはならない。

Range request は Rev.90 時点では実装しない。

`Range` ヘッダーを受信した場合も無視し、通常の `200 OK` または `304 Not Modified` 判定を行う。

Gzip 圧縮済みファイルを返す場合は `Content-Encoding: gzip` を設定する。

Gzip 圧縮済みレスポンスでは `Vary: Accept-Encoding` を返す。

Gzip 圧縮は、`Accept-Encoding` の comma 区切り token に `gzip` が含まれる場合のみ行う。

`Accept-Encoding` token の比較は大文字小文字を区別しない。

`gzip;q=0` は gzip 不許可として扱う。

`gzip` の q 値が省略された場合は許可として扱う。

不正な q 値は gzip 不許可として扱う。

Gzip 圧縮対象は `GET` の `200 OK` レスポンスのみとする。

`HEAD`、`304 Not Modified`、エラーレスポンス、1 byte 未満のファイル、すでに `Content-Encoding` が設定されるレスポンスは Gzip 圧縮対象外とする。

Gzip 圧縮はレスポンス送信時に Go 標準ライブラリ `compress/gzip` で行い、`.gz` ファイル、キャッシュファイル、メタデータを作成してはならない。

Gzip 圧縮開始前に失敗した場合は、未圧縮の `200 OK` として返してよい。

Gzip 圧縮開始後に失敗した場合は、接続を終了し、`ERR_INTERNAL` を error log へ記録する。

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

GitHub Push payload では、トップレベル `ref`、`after`、`repository` object を必須とする。

`repository` object では `clone_url`、`ssh_url`、`html_url` を読み込んでよいが、デプロイ元決定、認証、Git通信には使用してはならない。

`ref` は `refs/heads/{branch}` 形式のみ処理対象とする。

`ref` が `refs/heads/` で始まらない場合は `200 OK` とし、`{"status":"ignored"}` を返す。

`ref` から抽出した branch が `deploy.branch` と完全一致しない場合は `ignored` とする。

`deploy.sourcePath` が存在しない、Git worktreeでない、または `after` commit を参照できない場合は `ERR_WEBHOOK_SOURCE_INVALID` を返す。

`deploy.sourcePath` はディレクトリでなければならない。

`deploy.sourcePath/.git` が存在しない場合でも、`git -C deploy.sourcePath rev-parse --is-inside-work-tree` が成功し、出力が `true` の場合は Git worktree として扱う。

`after` は40文字の16進 SHA-1 形式のみ許可する。

`after` が空文字、すべて `0`、または16進以外を含む場合は `ERR_WEBHOOK_SOURCE_INVALID` とする。

`after` commit の参照可否は `git -C deploy.sourcePath cat-file -e {after}^{commit}` の成功で判定する。

Webhook処理中に `deploy.sourcePath` の branch checkout、fetch、pull、reset、clean、merge、rebase を実行してはならない。

静的コンテンツ反映元は `deploy.sourcePath` の `after` commit 時点のファイルツリーとする。

Rev.90 時点では、Webhookデプロイ時の対象ファイルパスはリポジトリルート配下の全静的ファイルとする。

`.git/`、`.github/`、`AGENTS.md`、`ASB-spec.md`、`ASB-spec.html`、`IMPLEMENTATION_TASKS.md`、`DOCUMENT_INDEX.md`、`README.md` は配信対象から除外する。

Webhook デプロイ対象ファイルは、`git ls-tree -r --name-only {after}` 相当で列挙した通常ファイルに限定する。

symlink、submodule、directory、Git 管理外ファイルはデプロイ対象外とする。

デプロイ対象パスに絶対パス、NUL 文字、`\`、`.` セグメント、`..` セグメント、空白のみのセグメント、先頭 `.` のセグメントが含まれる場合は、Webhook 処理を失敗扱いとし、`ERR_WEBHOOK_SOURCE_INVALID` を返す。

除外ファイル判定は、デプロイ対象パスの正規化後、`contents/` へのコピー前に実行する。

デプロイ先は対象Projectの `storage/projects/:projectId/contents/` 配下に限定する。

Webhookデプロイは開発リポジトリ内へ実行時データ、一時ファイル、ログファイルを作成してはならない。

冪等キーは `branch + ":" + after` とする。

同一冪等キーが `config/webhooks.json` に存在する場合は `200 OK` とし、`{"status":"duplicate"}` を返す。

Webhook 処理完了後に処理状態を `config/webhooks.json` へ保存する。

成功時の `status` は `deployed` とする。

失敗時の `status` は `failed` とし、`errorCode` を保存する。

Webhook失敗時の自動リトライは Rev.90 時点では実装しない。

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

12-14 で使用する一時ディレクトリは、対象 Project の `contents/` と同一ファイルシステム上に作成する。

14 の atomic rename は、既存 `contents/` を `storage/projects/:projectId/deploy-staging/{deployId}/previous-contents/` へ退避した後、新 `contents/` を公開先へ rename する。

新 `contents/` の公開に失敗した場合は、退避済み `previous-contents/` を公開先へ戻す。

新 `contents/` 公開後に 15 の `files.json` 更新へ失敗した場合は、退避済み `previous-contents/` を公開先へ戻してよい。ただし復元に成功した場合でも Webhook 処理は失敗扱いとし、成功レスポンスを返してはならない。

16 の `config/webhooks.json` 保存に失敗した場合は、成功レスポンスを返してはならない。

`deploy-staging/{deployId}/` 削除は、成功レスポンス送信前または失敗レスポンス送信前に1回だけ試行する。

`deploy-staging/{deployId}/` 削除に失敗した場合でも、`contents/` と `files.json` の状態が確定していれば、処理結果は変更しない。

`deploy-staging/{deployId}/` 削除失敗時は、`code` を空文字、`message` を `Operational warning`、`level` を `WARN` として error log へ記録する。

Webhook デプロイで作成する一時ディレクトリ、退避ディレクトリ、比較データは `storage.basePath` 配下に限定し、開発リポジトリ内へ作成してはならない。

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
4. 単一システム管理者認証
5. Project Service / Handler
6. File Service / Handler と静的配信
7. Domain Service / Handler
8. Log / Monitoring
9. Backup / Restore
10. Webhook
11. SSL 管理境界

各段階は `go test ./...` が成功する状態で次へ進む。

### 13.16 入出力契約固定仕様

本節は Rev.90 時点で API、JSON保存、ログ、起動時検証の入出力を固定する仕様である。

#### 13.16.1 共通成功レスポンス契約

成功レスポンスは常に JSON object とする。

成功レスポンスの `Content-Type` は `application/json; charset=utf-8` とする。

成功レスポンス内の日時は UTC RFC3339 秒精度とする。

成功レスポンス内のIDは UUID 文字列とする。

配列レスポンスは対象キーを必ず含め、対象データが空の場合は空配列を返す。

#### 13.16.1.1 管理 API 共通レスポンスヘッダー契約

本節は ASB 管理 HTTPS JSON API の全レスポンスに適用する。

静的コンテンツ配信、ACME HTTP-01 challenge 応答、ヘルスチェックは本節の管理 API 共通レスポンスヘッダー契約の対象外とし、各個別仕様を正とする。

ASB 管理 HTTPS JSON API は、成功レスポンスとエラーレスポンスの両方で以下のヘッダーを返す。

| Header | 値 | 条件 |
|--------|----|------|
| `Content-Type` | `application/json; charset=utf-8` | body を持つ管理 API レスポンスでは常に必須 |
| `X-Request-Id` | requestId | 全管理 API レスポンスで必須 |
| `Cache-Control` | `no-store` | 全管理 API レスポンスで必須 |
| `Allow` | 許可 method の comma + space 区切り | `405 Method Not Allowed` の場合のみ必須 |

管理 API の `X-Request-Id` は、リクエスト受信時に生成した requestId と一致しなければならない。

管理 API の `X-Request-Id` は空文字にしてはならない。

管理 API の `Cache-Control` は `no-store` に固定し、Project、Domain、SSL、File、Backup、Log、Monitoring、Auth の各 API で個別に変更してはならない。

管理 API のレスポンスに `ETag`、`Last-Modified`、`Content-Encoding`、`Vary` を付与してはならない。これらは静的コンテンツ配信専用のヘッダーとして扱う。

管理 API の response body は UTF-8 JSON object とし、末尾改行の有無に依存する仕様を作ってはならない。

`204 No Content` は使用しないため、管理 API の成功レスポンスは必ず JSON body を持つ。

`HEAD` は管理 API の許可 method として定義しない。管理 API に `HEAD` が送信された場合は、対象 path の許可 method に従い `405 Method Not Allowed` とする。

#### 13.16.1.2 管理 API request body 契約

管理 API で request body を受け付ける endpoint は、API 個別実装契約に明記された endpoint のみに限定する。

body なしの endpoint に request body が存在する場合は、`400 Bad Request` と `ERR_INVALID_REQUEST` を返す。

JSON body を受け付ける endpoint では、`Content-Type` は `application/json` または `application/json; charset=utf-8` のみ許可する。

`Content-Type` の charset parameter は `utf-8` のみ許可し、大文字小文字差は区別しない。

`application/json` と `application/json; charset=utf-8` 以外の media type、未知 parameter、複数 charset、空 charset は `400 Bad Request` と `ERR_INVALID_REQUEST` を返す。

JSON API request body は UTF-8 JSON object のみ許可し、top-level array、文字列、数値、真偽値、`null` を許可しない。

JSON decode では未知フィールド、空 body、後続トークン、無効な UTF-8 を拒否し、`400 Bad Request` と `ERR_INVALID_REQUEST` を返す。

request body サイズ超過は `413 Payload Too Large` と `ERR_INVALID_REQUEST` を返す。

multipart upload endpoint は `multipart/form-data` のみ許可し、JSON body 契約の対象外とする。ただし、共通エラーレスポンス、`X-Request-Id`、`Cache-Control: no-store` は維持する。

#### 13.16.1.3 管理 API routing 失敗契約

管理 API path は `/api/` で始まる path のみとする。

`/api` と `/api/` は未定義 API path として扱い、`404 Not Found` と `ERR_NOT_FOUND` を返す。

未定義の `/api/` 配下 path は `404 Not Found` と `ERR_NOT_FOUND` を返す。

定義済み path に対する未対応 method は `405 Method Not Allowed` と `ERR_METHOD_NOT_ALLOWED` を返す。

`405 Method Not Allowed` の `Allow` ヘッダーは、対象 path で許可された method を大文字表記、comma + space 区切り、辞書順で返す。

URL decode 失敗、NUL、`\`、`//`、`.`、`..` を含む API path は、routing 前の不正 path として `400 Bad Request` と `ERR_INVALID_REQUEST` を返す。

query string と fragment は routing 判定に使用しない。fragment は HTTP request に含まれないため、ASB の routing 入力として扱わない。

`/api/` 配下の request は、未定義 path、method 不一致、認証失敗、validation 失敗のいずれの場合も静的コンテンツ配信へ fallback してはならない。

#### 13.16.1.4 timestamp 契約

管理 API の error response `timestamp`、成功レスポンス中の `createdAt`、`updatedAt`、`deletedAt`、`uploadedAt`、`restoredAt`、`checkedAt` は UTC RFC3339 秒精度に固定する。

timestamp 生成は Service に注入された Clock を使用し、Handler または Repository が直接時刻取得してはならない。

Clock が時刻取得に失敗した場合は、対象操作を失敗扱いとし、成功レスポンスを返してはならない。

timestamp は timezone offset 付き表記ではなく、末尾 `Z` の UTC 表記とする。

#### 13.16.1.5 静的配信レスポンスとの境界

静的コンテンツ配信は管理 HTTPS JSON API ではないため、管理 API の JSON response header 契約を適用しない。

静的配信の `Content-Type`、`ETag`、`Last-Modified`、`Cache-Control`、`Content-Encoding`、`Vary`、`304 Not Modified`、`HEAD` の扱いは静的配信仕様を正とする。

静的配信は `X-Request-Id` を返してよいが、管理 API と異なり必須ではない。

静的配信の `404 Not Found` は公開サイト向け応答であり、管理 API の共通エラーレスポンス形式を適用してはならない。

管理 API のレスポンス契約を理由に、静的配信へ JSON error body、`Cache-Control: no-store`、管理 API 用 `Content-Type` を強制してはならない。

#### 13.16.2 API 成功レスポンス固定表

| API | HTTP | 固定レスポンス |
|-----|------|----------------|
| Project 作成 | 201 | `{"id":string,"name":string,"quota":number,"used":number,"domains":[],"createdAt":string}` |
| Project 一覧 | 200 | `{"projects":[Project...]}` |
| Project 削除 | 200 | `{"status":"deleted","projectId":string,"deletedAt":string}` |
| Domain 追加 | 201 | `{"domain":string,"projectId":string,"isCustom":true,"sslCert":string,"createdAt":string}` |
| Domain 一覧 | 200 | `{"domains":[Domain...]}` |
| Domain 削除 | 200 | `{"status":"deleted","projectId":string,"domain":string,"deletedAt":string}` |
| 無料独自SSL 有効化 | 200 | SSLStatus |
| 無料独自SSL 状態確認 | 200 | SSLStatus |
| 無料独自SSL 更新 | 200 | SSLStatus |
| 無料独自SSL 無効化 | 200 | SSLStatus |
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

#### 13.16.3 SSLStatus レスポンス固定仕様

無料独自SSL API は、成功時に必ず SSLStatus object を返す。

SSLStatus object は以下のキーを持つ。

| キー | 型 | 必須 | 説明 |
|------|----|------|------|
| `domain` | string | yes | 小文字正規化済み Domain |
| `projectId` | string | yes | 対象 Project ID |
| `status` | string | yes | `disabled`、`pending`、`challenge_ready`、`issued`、`renewing`、`failed`、`expired` のいずれか |
| `enabled` | boolean | yes | `status` が `disabled` 以外の場合 `true` |
| `issuer` | string | yes | 固定値 `LetsEncrypt` |
| `challenge` | string | yes | 固定値 `http-01` |
| `certificatePath` | string | yes | `issued`、`renewing`、`expired` の場合は `storage.basePath/certs/{domain}/fullchain.pem`、それ以外は空文字 |
| `privateKeyPath` | string | yes | `issued`、`renewing`、`expired` の場合は `storage.basePath/certs/{domain}/privkey.pem`、それ以外は空文字 |
| `expiresAt` | string | yes | 証明書有効期限。未発行または無効化済みの場合は空文字 |
| `renewAfter` | string | yes | 自動更新対象となる UTC RFC3339 時刻。未発行または無効化済みの場合は空文字 |
| `nextRetryAt` | string | yes | retry / backoff 中の場合の次回再試行時刻。それ以外は空文字 |
| `lastErrorCode` | string | yes | 直近失敗の ASB エラーコード。失敗がない場合は空文字 |
| `lastErrorMessage` | string | yes | 直近失敗理由。失敗がない場合は空文字 |
| `updatedAt` | string | yes | UTC RFC3339 |

SSLStatus は上記以外のキーを持ってはならない。

SSLStatus の `status` が `disabled` の場合、`enabled` は `false` とし、`certificatePath`、`privateKeyPath`、`expiresAt`、`renewAfter`、`nextRetryAt`、`lastErrorCode`、`lastErrorMessage` は空文字とする。

SSLStatus の `status` が `pending` または `challenge_ready` の場合、`enabled` は `true` とし、`certificatePath`、`privateKeyPath`、`expiresAt`、`renewAfter` は空文字とする。

SSLStatus の `status` が `issued` の場合、`enabled` は `true` とし、`certificatePath`、`privateKeyPath`、`expiresAt`、`renewAfter` を空文字にしてはならない。

SSLStatus の `status` が `renewing` の場合、`enabled` は `true` とし、既存証明書の `certificatePath`、`privateKeyPath`、`expiresAt`、`renewAfter` を返す。

SSLStatus の `status` が `failed` の場合、`enabled` は `true` とし、`lastErrorCode` と `lastErrorMessage` を空文字にしてはならない。

SSLStatus の `status` が `expired` の場合、`enabled` は `true` とし、期限切れ証明書の `certificatePath`、`privateKeyPath`、`expiresAt` を返す。

`renewAfter` は `expiresAt - ssl.renewBefore` で算出する。

`lastErrorCode` は `13.17.14 エラーコード固定表` に存在する値のみ許可する。

無料独自SSL 有効化 API は、対象 Domain が `disabled` または `failed` の場合、新規取得処理を開始した後の SSLStatus を返す。

無料独自SSL 有効化 API は、対象 Domain が `pending`、`challenge_ready`、`issued`、`renewing` の場合、重複する ACME order を作成せず現在の SSLStatus を返す。

無料独自SSL 更新 API は、対象 Domain が `issued` または `expired` の場合のみ手動更新処理を開始できる。

無料独自SSL 更新 API は、対象 Domain が `disabled`、`pending`、`challenge_ready`、`renewing` の場合 `409 Conflict` とし、`ERR_SSL_OPERATION_CONFLICT` を返す。

無料独自SSL 無効化 API は、対象 Domain の状態を `disabled` に更新した後の SSLStatus を返す。

#### 13.16.4 共通エラーレスポンス契約

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

エラーレスポンスの `error` は以下の固定文言のみ許可する。

| code | error |
|------|-------|
| `ERR_INVALID_JSON` | `Invalid JSON` |
| `ERR_UNKNOWN_FIELD` | `Unknown field` |
| `ERR_INVALID_REQUEST` | `Invalid request` |
| `ERR_AUTH_FAILED` | `Authentication failed` |
| `ERR_AUTH_PASSWORD_CHANGE_REQUIRED` | `Password change required` |
| `ERR_PROJECT_NOT_FOUND` | `Project not found` |
| `ERR_PROJECT_ALREADY_EXISTS` | `Project already exists` |
| `ERR_PROJECT_QUOTA_EXCEEDED` | `Project quota exceeded` |
| `ERR_DOMAIN_NOT_FOUND` | `Domain not found` |
| `ERR_DOMAIN_ALREADY_ASSIGNED` | `Domain already assigned` |
| `ERR_OPERATION_CONFLICT` | `Operation conflict` |
| `ERR_FILE_NOT_FOUND` | `File not found` |
| `ERR_FILE_UPLOAD_FAILED` | `File upload failed` |
| `ERR_STORAGE_VALIDATION_FAILED` | `Storage validation failed` |
| `ERR_BACKUP_NOT_FOUND` | `Backup not found` |
| `ERR_BACKUP_RESTORE_CONFLICT` | `Backup restore conflict` |
| `ERR_BACKUP_RESTORE_FAILED` | `Backup restore failed` |
| `ERR_WEBHOOK_SIGNATURE_INVALID` | `Invalid webhook signature` |
| `ERR_WEBHOOK_PROJECT_NOT_CONFIGURED` | `Webhook project not configured` |
| `ERR_WEBHOOK_SOURCE_INVALID` | `Webhook source invalid` |
| `ERR_WEBHOOK_PROCESSING_FAILED` | `Webhook processing failed` |
| `ERR_SSL_OPERATION_CONFLICT` | `SSL operation conflict` |
| `ERR_SSL_CERT_GENERATION_FAILED` | `SSL certificate validation failed` |
| `ERR_LOG_READ_FAILED` | `Log read failed` |
| `ERR_LOG_WRITE_FAILED` | `Log write failed` |
| `ERR_INTERNAL` | `Internal server error` |

上記表にない `error` 文言を実装してはならない。

#### 13.16.5 保存JSON配列要素スキーマ固定

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

#### 13.16.5.1 保存JSON field 固定表

保存 JSON の配列要素は以下の field のみを持つ。

| Object | Field | 型 | 必須 | Default | Validation |
|--------|-------|----|------|---------|------------|
| Project | `id` | string | yes | なし | UUID v4 |
| Project | `name` | string | yes | なし | `^[A-Za-z0-9_-]{1,255}$` |
| Project | `quota` | number | yes | `storage.maxProjectSize` | integer、1以上、`storage.maxProjectSize` 以下 |
| Project | `used` | number | yes | `0` | integer、0以上、`quota` 以下 |
| Project | `domains` | array | yes | `[]` | string array、domain昇順、`config/domains.json` と整合 |
| Project | `createdAt` | string | yes | 生成時刻 | UTC RFC3339 秒精度 |
| Domain | `domain` | string | yes | なし | 小文字ASCII、2〜3 labels、253 bytes以下 |
| Domain | `projectId` | string | yes | なし | 既存 Project UUID |
| Domain | `isCustom` | boolean | yes | `true` | Rev.90 時点では `true` のみ |
| Domain | `sslCert` | string | yes | `""` | 空文字または SSL certificate ID |
| Domain | `createdAt` | string | yes | 生成時刻 | UTC RFC3339 秒精度 |
| File | `id` | string | yes | なし | UUID v4 |
| File | `projectId` | string | yes | なし | 既存 Project UUID |
| File | `name` | string | yes | なし | File名規則 |
| File | `size` | number | yes | なし | integer、1以上、1GiB以下 |
| File | `path` | string | yes | なし | `contents/` で始まる相対パス |
| File | `uploadedAt` | string | yes | 生成時刻 | UTC RFC3339 秒精度 |
| Backup | `id` | string | yes | なし | UUID v4 |
| Backup | `projectId` | string | yes | なし | 作成時点の Project UUID |
| Backup | `createdAt` | string | yes | 生成時刻 | UTC RFC3339 秒精度 |
| Backup | `size` | number | yes | なし | integer、1以上 |
| Backup | `path` | string | yes | なし | `backups/` 配下相対パス |
| Backup | `sha256` | string | yes | なし | lowercase hex 64 chars |
| Backup | `status` | string | yes | `completed` | `completed`、`failed` |
| WebhookEvent | `key` | string | yes | なし | `{branch}:{after}` |
| WebhookEvent | `branch` | string | yes | なし | deploy.branch規則 |
| WebhookEvent | `after` | string | yes | なし | 40文字 lowercase hex SHA-1 |
| WebhookEvent | `status` | string | yes | なし | `deployed`、`failed` |
| WebhookEvent | `receivedAt` | string | yes | 受信時刻 | UTC RFC3339 秒精度 |
| WebhookEvent | `completedAt` | string | yes | 完了時刻 | UTC RFC3339 秒精度、未完了保存はしない |
| WebhookEvent | `errorCode` | string | yes | `""` | 成功時空文字、失敗時定義済み error code |

Project の `domains` は response 利便性のための重複保持 field とし、`config/domains.json` と整合しなければならない。

Domain 追加または削除時は、`config/domains.json` と `config/projects.json` の Project `domains` を同一操作内で更新する。

Domain 更新の途中で片方の JSON 保存に失敗した場合、成功レスポンスを返してはならない。

File 更新時は、`files.json` の `files[].size` 合計と Project `used` が一致しなければならない。

File upload、overwrite、delete の途中で `files.json` と `projects.json` の整合が取れない場合、成功レスポンスを返してはならない。

Backup `status` が `failed` の要素は復旧対象にしてはならない。

WebhookEvent は処理完了後のみ保存し、`running`、`queued`、`processing` などの中間状態を保存してはならない。

保存 JSON の object key 出力順序は、本節の field 固定表の順序に従う。

#### 13.16.6 保存順序固定

JSON 保存時の配列順序は以下で固定する。

| ファイル | ソート順 |
|---------|----------|
| `config/projects.json` | `createdAt` 昇順、同一時刻の場合は `id` 昇順 |
| `config/domains.json` | `domain` 昇順 |
| `config/backups.json` | `createdAt` 降順、同一時刻の場合は `id` 昇順 |
| `storage/projects/:projectId/files.json` | `name` 昇順 |
| `config/webhooks.json` | `receivedAt` 降順、同一時刻の場合は `key` 昇順 |

#### 13.16.7 起動時検証出力固定

起動時検証は以下の順序で実行する。

1. `--config` で指定された起動設定ファイルの存在確認
2. 起動設定ファイルの通常ファイル確認
3. 起動設定ファイルの読み取り権限確認
4. 起動設定ファイルの JSON 構文検証
5. 起動設定ファイルの未知フィールド拒否
6. 起動設定値の型、範囲、空文字、絶対パス制約検証
7. `storage.basePath` が開発リポジトリ配下でないことの検証
8. `storage.basePath` の存在、directory、読み取り権限確認
9. 必須ディレクトリ存在確認
10. 必須 JSON ファイル存在確認
11. 必須 JSON ファイルの構文とスキーマ検証
12. 必須ログファイル存在確認
13. Project 別ディレクトリと `files.json` 検証
14. SSL 状態付き Domain の証明書検証

必須ディレクトリ存在確認は以下の順序で実行する。

1. `storage.basePath/config/`
2. `storage.basePath/storage/`
3. `storage.basePath/storage/projects/`
4. `storage.basePath/logs/`
5. `storage.basePath/certs/`

必須 JSON ファイル存在確認は以下の順序で実行する。

1. `storage.basePath/config/projects.json`
2. `storage.basePath/config/domains.json`
3. `storage.basePath/config/backups.json`
4. `storage.basePath/config/auth.json`
5. `storage.basePath/config/webhooks.json`
6. `storage.basePath/config/acme_accounts.json`
7. `storage.basePath/config/acme_orders.json`
8. `storage.basePath/config/acme_authorizations.json`
9. `storage.basePath/config/acme_challenges.json`
10. `storage.basePath/config/acme_renewals.json`

必須ログファイル存在確認は以下の順序で実行する。

1. `storage.basePath/logs/access.log`
2. `storage.basePath/logs/error.log`

Project 別ディレクトリと `files.json` は、`config/projects.json` に存在する Project のみ検証対象とする。

ASB は起動時検証の途中で、不足したディレクトリ、JSON ファイル、ログファイル、証明書、ACME challenge file、一時ファイルを作成してはならない。

起動時検証失敗時は HTTPS サーバーを起動せず、終了コード `1` で終了する。

標準エラーには以下の1行のみを出力する。

```text
ASB_STARTUP_ERROR code=<errorCode> message="<message>"
```

起動設定ファイルの構文、未知フィールド、設定値不正は `ERR_INVALID_REQUEST` を使用する。

実行時ディレクトリ、実行時 JSON、ログファイル、権限、スキーマ不正は `ERR_STORAGE_VALIDATION_FAILED` を使用する。

証明書ファイルと秘密鍵の存在、読み込み、対応、有効期限検証に失敗した場合は `ERR_SSL_CERT_GENERATION_FAILED` を使用する。

#### 13.16.8 ログJSON Lines固定

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

アクセスログの `remoteAddr` は Host header / reverse proxy / client IP 境界固定仕様に従い、`Request.RemoteAddr` 由来の値のみを記録する。

アクセスログの `remoteAddr` を `X-Forwarded-For`、`X-Real-IP`、`Forwarded`、CDN由来 header で上書きしてはならない。

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

エラーログの `message` は以下の固定文言のみ許可する。

| code | message |
|------|---------|
| `ERR_INVALID_JSON` | `Invalid JSON detected` |
| `ERR_UNKNOWN_FIELD` | `Unknown field detected` |
| `ERR_INVALID_REQUEST` | `Invalid request rejected` |
| `ERR_PROJECT_NOT_FOUND` | `Project not found` |
| `ERR_PROJECT_ALREADY_EXISTS` | `Project already exists` |
| `ERR_PROJECT_QUOTA_EXCEEDED` | `Project quota exceeded` |
| `ERR_DOMAIN_NOT_FOUND` | `Domain not found` |
| `ERR_DOMAIN_ALREADY_ASSIGNED` | `Domain already assigned` |
| `ERR_OPERATION_CONFLICT` | `Operation conflict` |
| `ERR_FILE_NOT_FOUND` | `File not found` |
| `ERR_FILE_UPLOAD_FAILED` | `File upload failed` |
| `ERR_STORAGE_VALIDATION_FAILED` | `Storage validation failed` |
| `ERR_BACKUP_NOT_FOUND` | `Backup not found` |
| `ERR_BACKUP_RESTORE_CONFLICT` | `Backup restore conflict` |
| `ERR_BACKUP_RESTORE_FAILED` | `Backup restore failed` |
| `ERR_WEBHOOK_SIGNATURE_INVALID` | `Webhook signature invalid` |
| `ERR_WEBHOOK_PROJECT_NOT_CONFIGURED` | `Webhook project not configured` |
| `ERR_WEBHOOK_SOURCE_INVALID` | `Webhook source invalid` |
| `ERR_WEBHOOK_PROCESSING_FAILED` | `Webhook processing failed` |
| `ERR_SSL_CERT_GENERATION_FAILED` | `SSL certificate validation failed` |
| `ERR_LOG_READ_FAILED` | `Log read failed` |
| `ERR_LOG_WRITE_FAILED` | `Log write failed` |
| `ERR_INTERNAL` | `Internal error` |

`code` が空文字の場合、`message` は `Operational warning` のみ許可する。

HTTP リクエスト処理中に発生したエラーでは、エラーログの `requestId` をアクセスログおよびレスポンスヘッダーと一致させる。

リクエスト外のエラーとは、起動時検証、ログローテーション、期限切れログ削除、マイグレーションCLI、install/update スクリプト処理、サーバー shutdown 処理を指す。

上記以外では `requestId` を空文字にしてはならない。

リクエスト外のエラーでは `requestId` は空文字を許可する。

`log.level` は出力する最小レベルを表す。

`DEBUG` は `debug` 設定時のみ出力する。

`INFO` は `debug` または `info` 設定時に出力する。

`WARN` は `debug`、`info`、`warn` 設定時に出力する。

`ERROR` はすべての `log.level` 設定で出力する。

`log.format` は Rev.90 時点では `json` のみ許可する。

通常運用ログを stdout へ出力してはならない。

起動時検証失敗時のみ、stderr へ以下の形式で単一行を出力する。

```text
ASB_STARTUP_ERROR code=<errorCode> message="<message>"
```

#### 13.16.8.1 ログ生成境界

ログファイルは実行時データである。ただし、ログ追記とログローテーションは、仕様で定義された保存状態変更として扱う。

ASB がログ機能で実行してよいファイル操作は以下に限定する。

1. 既存の `storage.basePath/logs/access.log` への append
2. 既存の `storage.basePath/logs/error.log` への append
3. `log.maxSize` 到達後の既存現行ログファイルの `access.log.{unixTime}` または `error.log.{unixTime}` への rename
4. ローテーション直後の新しい `access.log` または `error.log` の作成
5. 期限切れローテーション済みログの削除

上記 4 は、既存現行ログファイルの rename が成功した場合に限り許可する。

ASB は以下を行ってはならない。

- 起動時に `logs/` を作成する
- 起動時に `access.log` または `error.log` を作成する
- Log API、Monitoring API、静的配信、File API、Domain API、SSL状態確認API、Backup一覧APIを契機にログファイルを初回作成する
- ログファイル欠落を検出して自動修復する
- ログファイル破損を検出して切り詰め、上書き、削除、退避、再作成する
- 開発リポジトリ内にログ、一時ログ、ローテーション済みログ、ログ検証結果を作成する

`access.log` または `error.log` が存在しない状態でログ書き込みが必要になった場合、ASB はログファイルを作成せず `ERR_LOG_WRITE_FAILED` とする。

Log API が対象ログファイルの欠落、読み取り不可、JSON Lines 破損を検出した場合、ASB はログファイルを修復せず `ERR_LOG_READ_FAILED` を返す。

ログローテーションは現在ログファイルのサイズが `log.maxSize` 以上になった後、次回書き込み前に実行する。

ローテーションでは現在ログファイルを `access.log.{unixTime}` または `error.log.{unixTime}` へ rename し、新しい `access.log` または `error.log` を作成する。

ローテーション後に新しい現行ログファイルを作成できない場合、ASB は対象ログ書き込みを失敗扱いとし、可能であれば rename 済みログファイルを元の現行ログファイル名へ戻す復旧を1回だけ試行する。

復旧に成功した場合でも、対象ログ書き込みは失敗扱いとする。

復旧に失敗した場合、ASB は追加の補完生成を行わず、`ERR_LOG_WRITE_FAILED` を返す。HTTP レスポンスが送信済みの場合は、標準エラーへ単一行で警告を出力する。

ローテーション済みログの削除対象は、ファイル名末尾の unixTime が現在時刻から7日より古いものに限定する。

期限切れローテーション済みログの削除失敗は、対象リクエストの主処理が確定済みの場合に限り WARN として記録し、成功レスポンスを維持してよい。

ログローテーション失敗時は対象ログ書き込みを失敗扱いとし、HTTP レスポンスが未送信の場合は `500 Internal Server Error` と `ERR_LOG_WRITE_FAILED` を返す。

ログ API は `access.log` または `error.log` の現行ファイルのみを読む。ローテーション済みログは Rev.90 時点ではログ API の対象外とする。

ログ API は対象ログファイルを先頭から読み、JSON Lines を1行ずつ decode する。

空行は壊れたログ行として扱い、`ERR_LOG_READ_FAILED` を返す。

1行でも JSON decode に失敗した場合は `ERR_LOG_READ_FAILED` を返し、部分的な `logs` を成功レスポンスとして返してはならない。

ログ API の `limit` / `offset` は、全行の decode とスキーマ検証が成功した後に適用する。

`offset` がログ件数以上の場合は `200 OK` とし、`logs: []` を返す。

ログ API はログファイルを作成、更新、ローテーション、削除してはならない。

ログには内部ファイルパス、スタックトレース、環境変数、シークレットを含めてはならない。

`storage.basePath/logs/` と現行ログファイルは、ASB 起動前に存在していなければならない。

LogService は、存在しないログディレクトリまたはログファイルを作成してはならない。

LogService は、既存ログファイルへの追記と、既存ログファイルの rotation のみ実行できる。

ログAPI、監視API、静的配信、File API、Domain API、SSL状態確認API、Backup一覧APIは、ログファイルを初回作成する契機として扱ってはならない。

アクセスログ書き込みはレスポンスステータス確定後、レスポンス送信前に実行する。

成功レスポンス送信前にアクセスログ書き込みまたはローテーションへ失敗した場合は、`500 Internal Server Error` と `ERR_LOG_WRITE_FAILED` を返す。

エラーレスポンス生成中に error log 書き込みへ失敗した場合は、レスポンス本文へ内部パスまたは詳細原因を含めず、既定のエラーレスポンスを返す。

監視APIは既存の実行時JSON、プロセス情報、OS情報、`storage.basePath` の状態から値を計算する。

監視APIはメトリクス保存用JSON、履歴JSON、キャッシュファイル、一時ファイル、lock file、監視DB、その他の実行時データを作成、更新、削除してはならない。

監視APIは `connections` と `requests` を process-local in-memory counter として扱い、ファイルへ永続化してはならない。

監視APIはログファイル、ローテーション済みログ、監査ログを監視値の集計元として扱ってはならない。

OS差異により取得できない監視値は `null` とし、取得不可だけを理由に失敗レスポンスへしてはならない。

監視APIの response 生成不能、response serialization 不能、または `storage.basePath` の判定に必要な起動設定読込不能は `500 Internal Server Error` と `ERR_INTERNAL` を返す。

#### 13.16.9 テスト固定項目

Rev.90 の実装では、以下のテストを必須とする。

- 全API成功レスポンスの固定JSONキー検証
- 全APIエラーレスポンスの固定JSONキー検証
- 保存JSONの未知フィールド拒否
- 保存JSONの相対パス保存検証
- 保存JSONのソート順検証
- 起動時検証の順序、終了コード、標準エラー形式検証
- アクセスログとエラーログのJSON Linesフィールド検証

### 13.17 実装境界とファイル操作固定仕様

本節は Rev.90 時点で package 境界、公開 interface、Repository、Storage、複数ファイル更新の実装契約を固定する仕様である。

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
15. 正規化後の相対パスが `files.json` の `files[].path` に記録されていることを検証する
16. 未記録の場合は `404 Not Found` を返す
17. 対象ファイルの存在、通常ファイル、サイズ一致を検証する
18. 記録済みファイルの不整合は `ERR_STORAGE_VALIDATION_FAILED` をログに記録し、`500 Internal Server Error` を返す
19. `Content-Type`、`ETag`、`Last-Modified`、`Cache-Control` を決定する
20. `If-None-Match` を優先し、未指定時のみ `If-Modified-Since` により未変更判定を行う
21. 未変更と判定できる場合は `304 Not Modified` を返す
22. `HEAD` の場合はヘッダーのみを返す
23. `GET` の場合はファイル内容をレスポンスボディとして返す

静的配信では `Range` ヘッダーを無視し、`206 Partial Content` を返してはならない。

静的配信ではディレクトリ一覧を返してはならない。

静的配信では開発リポジトリ内に配信用一時ファイル、キャッシュファイル、実行時データを作成してはならない。

#### 13.17.4 ファイルアップロード処理順序固定

File upload は以下の順序で実行する。

1. URL `:id` を検証する
2. Project の存在を検証する
3. `Content-Type` が `multipart/form-data` であることを検証する
4. multipart field `file` が1つだけ存在することを検証する
5. ファイル名、サイズ、quota を検証する
6. `name` と保存先 `path` の対応を決定し、`contents/` 配下に収まることを検証する
7. 同名ファイルの既存有無により、新規作成または上書き処理へ分岐する
8. 既存 `files.json` を読み込み検証する
9. ファイル実体を `contents/` 配下の一時ファイルへ書き込む
10. 書き込み内容を `fsync` する
11. 一時ファイルを公開先へ atomic rename する
12. `files.json` を更新する
13. `projects.json` の `used` を更新する
14. 成功レスポンスを返す

upload API は `multipart/form-data` 以外を `400 Bad Request` とし、`ERR_INVALID_REQUEST` を返す。

multipart field `file` が存在しない場合、または複数存在する場合は `400 Bad Request` とし、`ERR_INVALID_REQUEST` を返す。

upload file part の filename は必須とする。

filename は `/`、`\`、NUL、`.`、`..`、先頭 `.`, 空白のみを拒否する。

フォルダ階層を含む upload path を扱う場合、各 path segment に同じ検証を適用する。

保存先 `path` は `contents/` で始まる相対パスとし、`files.json` には `storage.basePath` からの絶対パスを保存してはならない。

新規 upload では、同一 `name` または同一 `path` が既に `files.json` に存在する場合、同名ファイル上書き処理として扱う。

新規 upload では、公開先ファイルが存在するにもかかわらず `files.json` に記録がない場合、上書きせず `ERR_STORAGE_VALIDATION_FAILED` を返す。

9-13 の途中で失敗した場合、成功レスポンスを返してはならない。

公開先への rename 後に JSON 更新が失敗した場合は、エラーログを記録し、次回起動時検証または整合性検証で検出できる状態にする。

9-11 の途中で失敗した場合は、作成済みの一時ファイルを削除し、削除に失敗した場合は `code` を空文字、`message` を `Operational warning`、`level` を `WARN` として error log へ記録する。

12 の `files.json` 更新に失敗した場合は、公開先ファイルを削除してよい。ただし削除失敗時でも成功レスポンスを返してはならない。

13 の `projects.json` 更新に失敗した場合は、`files.json` とファイル実体を自動ロールバックしてはならない。

File upload 失敗時に、開発リポジトリ内へ退避ファイル、比較ファイル、復旧用ファイル、一時ファイルを作成してはならない。

#### 13.17.5 ファイル上書き処理順序固定

同名ファイル上書きは以下の順序で実行する。

1. 旧ファイルメタデータを読み込む
2. 旧ファイル実体の存在と通常ファイルであることを検証する
3. 新ファイルを同一 `contents/` 配下の一時ファイルへ書き込む
4. 新ファイルを `fsync` する
5. 旧ファイル実体を同一 `contents/` 配下の退避ファイルへ atomic rename する
6. 新ファイルを公開先へ atomic rename する
7. `files.json` の `size`、`uploadedAt`、`path` を更新する
8. `projects.json` の `used` を差分更新する
9. 退避した旧ファイル実体を削除する

旧ファイルは、新ファイルの atomic rename が成功するまで削除してはならない。

上書き後の `used` は旧サイズを差し引き、新サイズを加算して計算する。

3-6 の途中で失敗した場合は、公開先を旧ファイル実体へ戻す。

7-8 の途中で失敗した場合は、公開先を旧ファイル実体へ戻してよい。ただし復元失敗時でも成功レスポンスを返してはならない。

9 の退避ファイル削除に失敗した場合でも、`files.json` と `projects.json` の更新が完了していれば上書き成功として扱う。

退避ファイル削除失敗時は、`code` を空文字、`message` を `Operational warning`、`level` を `WARN` として error log へ記録する。

ファイル上書き処理で作成する一時ファイルと退避ファイルは、対象 Project の `contents/` 配下に限定し、開発リポジトリ内へ作成してはならない。

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

`projects.json` の `used` は、通常時は操作前後のファイルサイズ差分で更新する。

以下の場合は、対象 Project の `files.json` に記録された `size` の合計から `used` を再計算する。

- ファイル上書き後に `projects.json` の `used` と差分計算結果が一致しない場合
- ファイル削除後に `projects.json` の `used` が負数になる場合
- 起動時検証または整合性検証で `used` と `files[]` の合計が一致しない場合

再計算後の `used` が `quota` を超える場合は、対象操作を失敗扱いとし、`ERR_PROJECT_QUOTA_EXCEEDED` を返す。

`files.json` と実ファイルの不整合は以下で検出する。

- `files[]` に記録された `path` が `contents/` 配下に収まらない
- `files[]` に記録された `path` の実体が存在しない
- `files[]` に記録された `path` の実体が通常ファイルではない
- `files[]` に記録された `size` と実ファイルサイズが一致しない
- `contents/` 配下に `files[]` へ記録されていない通常ファイルが存在する

上記不整合を検出した場合、静的配信では `ERR_STORAGE_VALIDATION_FAILED` を error log へ記録し、対象リクエストを `500 Internal Server Error` とする。

上記不整合を検出した場合、File API の list、upload、overwrite、delete は成功レスポンスを返してはならない。

整合性検証は、不整合を自動修復してはならない。

不整合を検出した場合は、成功レスポンスを返さず、`ERR_STORAGE_VALIDATION_FAILED` を返す。ただし File delete で削除対象の実体のみが存在しない場合は `ERR_FILE_NOT_FOUND` を返す。

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

Backup作成は Project 単位で実行し、対象 Project が存在しない場合は `ERR_PROJECT_NOT_FOUND` を返す。

バックアップtar.gzに `logs/`、`certs/`、他Projectの `contents/` を含めてはならない。

バックアップtar.gzに `config/projects.json`、`config/domains.json`、`config/auth.json`、`config/webhooks.json`、`config/acme_accounts.json`、`config/acme_orders.json`、`config/acme_authorizations.json`、`config/acme_challenges.json`、`config/acme_renewals.json`、`config/migrations.json` を含めてはならない。

バックアップtar.gzには symlink、hardlink、device file、FIFO、socket、絶対パス、`..` セグメント、NUL 文字を含む path を含めてはならない。

バックアップtar.gz内の path 区切り文字は `/` のみ許可し、`\` を含む path を許可しない。

バックアップtar.gzの各エントリは、`files.json` または `contents/` 配下の通常ファイル・ディレクトリに限定する。

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
3. 復旧対象 Project の存在を検証する
4. Backup ファイルの存在を検証する
5. SHA-256 を検証する
6. `restore-staging/{restoreId}/` が存在しないことを検証する
7. `storage.basePath/backups/restore-staging/{restoreId}/previous/` を作成する
8. `storage.basePath/backups/restore-staging/{restoreId}/next/` を作成する
9. 復旧対象Projectの現行 `files.json` と `contents/` を `previous/` へ退避する
10. Backup を `next/` へ展開する
11. 展開後の JSON 構文とスキーマを検証する
12. `next/` の内容を復旧対象へ atomic rename する
13. 成功レスポンスを返す

Backup 展開時は、tar.gz 内の各エントリを展開前に検証する。

展開対象 path が `next/` 配下に収まらない場合、symlink、hardlink、device file、FIFO、socket、絶対パス、`..` セグメント、NUL 文字、`\` を含む場合は展開を中止し、`ERR_BACKUP_RESTORE_FAILED` を返す。

展開後に `next/files.json` と `next/contents/` が存在しない場合は `ERR_BACKUP_RESTORE_FAILED` を返す。

展開後に `next/contents/` 配下の実ファイルと `next/files.json` の整合性検証に失敗した場合は `ERR_BACKUP_RESTORE_FAILED` を返す。

9-12 の途中で失敗した場合、以下の順序で `previous/` から復元する。

1. 復旧対象Projectの `contents/` が存在する場合は `restore-staging/{restoreId}/failed-contents/` へ rename する
2. 復旧対象Projectの `files.json` が存在する場合は `restore-staging/{restoreId}/failed-files.json` へ rename する
3. `previous/files.json` を復旧対象Projectの `files.json` へ atomic rename する
4. `previous/contents/` を復旧対象Projectの `contents/` へ atomic rename する
5. 復元成功または復元失敗を error log へ記録する

復元に失敗した場合は `ERR_BACKUP_RESTORE_FAILED` を返し、成功レスポンスを返してはならない。

復旧対象Projectが存在しない場合は `ERR_PROJECT_NOT_FOUND` を返す。

復旧対象Projectの現行データ退避に失敗した場合は、復旧処理を開始してはならない。

復旧後の `restore-staging/{restoreId}/` 削除は、成功レスポンス送信前に1回だけ試行する。

`restore-staging/{restoreId}/` 削除に失敗した場合でも、復旧対象Projectの `files.json` と `contents/` の復旧が完了していれば復旧成功として扱う。

`restore-staging/{restoreId}/` 削除失敗時は、`code` を空文字、`message` を `Operational warning`、`level` を `WARN` として error log へ記録する。

開発リポジトリ内に復旧用一時ファイル、退避データ、展開データを作成してはならない。

#### 13.17.10 複数JSON更新失敗時契約

複数JSON更新は、操作順序を Service に閉じ込める。

複数JSON更新では、最終JSONの保存が完了するまで成功レスポンスを返してはならない。

途中失敗時は、更新済みJSON名、未更新JSON名、操作名、requestId をエラーログへ記録する。

途中失敗時に自動ロールバックを実装する場合も、ロールバック失敗時は成功扱いにしてはならない。

Rev.90 時点では、複数JSON更新に外部トランザクション機構を導入してはならない。

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

Rev.80 の初期実装では、Go 実装ファイルを以下の構成で作成する。

```text
cmd/asb/main.go
internal/config/
internal/server/
internal/management/
internal/delivery/
internal/data/
internal/system/
```

`cmd/asb/main.go` は、設定読み込み、依存関係生成、HTTPS サーバー起動、signal 受信、graceful shutdown のみを行う。

`internal/config/` は設定構造体、設定読み込み、デフォルト適用、未知フィールド拒否、起動時検証を実装する。

`internal/server/` は `net/http` ルーティング、middleware、requestId 付与、共通 JSON レスポンス、共通エラーレスポンス、HTTP テスト補助を実装する。

`internal/management/` は Project、Domain、SSL 管理境界の Entity、Service interface、Service 実装、Handler を実装する。

`internal/delivery/` は File 管理、静的配信、GitHub Webhook の Entity、Service interface、Service 実装、Handler を実装する。

`internal/data/` は JSON Repository、StorageService、BackupService、atomic save、tar.gz、restore staging を実装する。

`internal/system/` は Clock、IDGenerator、AuthService、LogService、MonitoringService、ログローテーション、監視値取得を実装する。

Go package 名はディレクトリ名と一致させる。

外部公開 API 用 package を追加してはならない。

`internal/` 外へ ASB 本体の実装 package を追加する場合は、先に本仕様を改訂する。

#### 13.17.13 package 内ファイル役割固定

各 package 内のファイル役割は以下に固定する。

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
| `system` | `auth.go` | 単一システム管理者パスワード検証、ハッシュ、パスワード変更 |
| `system` | `log.go` | JSON Lines ログとローテーション |
| `system` | `monitoring.go` | 監視値取得 |

上記ファイル分割は実装開始時の固定構成とする。

責務が増えてファイル分割が必要な場合でも、package 境界と公開 interface を変更してはならない。

#### 13.17.14 エラーコード固定表

Rev.90 の実装では、API と起動時検証が返すエラーコードを以下に固定する。

| code | HTTP | 用途 |
|------|------|------|
| `ERR_INVALID_JSON` | 400 | リクエスト JSON または保存 JSON の構文不正 |
| `ERR_UNKNOWN_FIELD` | 400 | リクエスト JSON、設定 JSON、保存 JSON の未知フィールド |
| `ERR_INVALID_REQUEST` | 400 | Content-Type、Body、query、path parameter の不正 |
| `ERR_AUTH_FAILED` | 401 | システム管理者パスワードが未指定または不正 |
| `ERR_AUTH_PASSWORD_CHANGE_REQUIRED` | 403 | 初期デフォルトパスワードの変更が必要 |
| `ERR_NOT_FOUND` | 404 | 未定義 API path、`/api`、`/api/` |
| `ERR_METHOD_NOT_ALLOWED` | 405 | 定義済み API path に対する未対応 HTTP method |
| `ERR_PROJECT_NOT_FOUND` | 404 | Project が存在しない |
| `ERR_PROJECT_ALREADY_EXISTS` | 409 | Project 名が重複している |
| `ERR_PROJECT_QUOTA_EXCEEDED` | 413 | Project quota を超過する |
| `ERR_DOMAIN_NOT_FOUND` | 404 | Domain が存在しない |
| `ERR_DOMAIN_ALREADY_ASSIGNED` | 409 | Domain が別 Project または同一 Project に割り当て済み |
| `ERR_OPERATION_CONFLICT` | 409 | 対象 resource が別操作中であり排他制御により操作できない |
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
| `ERR_SSL_OPERATION_CONFLICT` | 409 | 無料独自SSL操作が現在状態と競合 |
| `ERR_SSL_CERT_GENERATION_FAILED` | 500 | SSL 証明書管理境界で証明書状態検証に失敗 |
| `ERR_LOG_READ_FAILED` | 500 | ログ API の読み込みまたは JSON Lines 検証に失敗 |
| `ERR_LOG_WRITE_FAILED` | 500 | ログ書き込み、fsync、rotation、保存前検証に失敗 |
| `ERR_INTERNAL` | 500 | 上記に分類できない内部エラー |

`code` は上記表のいずれかでなければならない。

HTTP ステータスは上記表と `13.15.1 API エンドポイント固定表` の範囲で決定する。

保存JSONの起動時検証失敗では、構文不正を `ERR_INVALID_JSON`、未知フィールドを `ERR_UNKNOWN_FIELD`、ディレクトリ・権限・必須ファイル不備を `ERR_STORAGE_VALIDATION_FAILED` とする。

#### 13.17.14.1 HTTP status / error code 選択優先順位

同一 request で複数の失敗条件が成立し得る場合は、以下の順序で最初に検出した失敗を返す。

1. routing 前に判定できる URL decode 失敗、method 不一致、path 不正
2. Webhook を除く管理 API の認証失敗
3. 初期デフォルトパスワード未変更による操作禁止
4. Content-Type 不正
5. request body サイズ超過
6. JSON 構文不正
7. unknown field
8. request schema、path parameter、query parameter の validation 不正
9. 対象 Project、Domain、File、Backup の存在確認失敗
10. 操作競合
11. quota 超過
12. 保存 JSON、ファイル実体、証明書、backup archive、log の読み書き失敗
13. 上記以外の内部エラー

HTTP status の使い分けは以下に固定する。

| HTTP | 使用条件 | 代表 code |
|------|----------|-----------|
| 400 | request の形式、JSON、field、parameter、payload が不正 | `ERR_INVALID_JSON`、`ERR_UNKNOWN_FIELD`、`ERR_INVALID_REQUEST` |
| 401 | 認証入力または Webhook 署名が未指定、不正、不一致 | `ERR_AUTH_FAILED`、`ERR_WEBHOOK_SIGNATURE_INVALID` |
| 403 | 認証自体は検証可能だが初期パスワード変更が必須 | `ERR_AUTH_PASSWORD_CHANGE_REQUIRED` |
| 404 | 指定された Project、Domain、File、Backup が存在しない | `ERR_PROJECT_NOT_FOUND`、`ERR_DOMAIN_NOT_FOUND`、`ERR_FILE_NOT_FOUND`、`ERR_BACKUP_NOT_FOUND` |
| 409 | 既存状態または実行中操作との競合により操作できない | `ERR_PROJECT_ALREADY_EXISTS`、`ERR_DOMAIN_ALREADY_ASSIGNED`、`ERR_OPERATION_CONFLICT`、`ERR_SSL_OPERATION_CONFLICT`、`ERR_BACKUP_RESTORE_CONFLICT` |
| 413 | request body または Project quota が上限を超える | `ERR_INVALID_REQUEST`、`ERR_PROJECT_QUOTA_EXCEEDED` |
| 500 | ASB 内部の永続化、検証、外部コマンド、証明書、ログ処理に失敗 | `ERR_STORAGE_VALIDATION_FAILED`、`ERR_FILE_UPLOAD_FAILED`、`ERR_WEBHOOK_PROCESSING_FAILED`、`ERR_SSL_CERT_GENERATION_FAILED`、`ERR_LOG_READ_FAILED`、`ERR_LOG_WRITE_FAILED`、`ERR_INTERNAL` |

`404` と `409` のどちらも成立する可能性がある場合は、存在確認を先に行い、存在しない対象には `404` を返す。

認証失敗時は、対象 resource の存在有無を確認してはならない。

初期デフォルトパスワード未変更時は、`POST /api/auth/change-password` を除き、対象 resource の存在有無を確認してはならない。

Webhook 署名検証が必要な場合、署名検証に成功するまで request body を JSON decode してはならない。

対象外 Webhook event または対象外 branch は正常応答 `200 OK` とし、error code を返してはならない。

同一 Webhook 冪等キーの重複は正常応答 `200 OK` とし、error code を返してはならない。

静的配信では、path traversal、不正 path、未割当 Host、未記録 file の詳細差異をレスポンス本文に出してはならない。

#### 13.17.14.2 排他制御・整合性検証固定仕様

Rev.90 の実装では、ASB 本体を単一プロセスで動作する前提とし、排他制御はプロセス内 in-memory lock のみで実装する。

lock 状態を JSON、通常ファイル、ディレクトリ、`.gitignore`、外部DB、SQLite、KVS、外部ストレージ、分散 lock service に保存してはならない。

複数プロセス、複数インスタンス、分散 lock、共有 lock file、NFS lock、外部 lock service は Rev.90 時点では実装対象外とする。

同一操作で複数 lock が必要な場合、取得順序は以下に固定する。

1. global startup / migration lock
2. Project lock
3. Domain lock
4. SSL lock
5. File lock
6. Backup / Restore lock
7. Webhook deploy lock
8. Log lock

lock は上記順序で取得し、解放は逆順で行う。

下位順序の lock を取得した後に上位順序の lock を追加取得してはならない。

lock upgrade、循環取得、取得順序の domain 別例外、暗黙的な再取得を実装してはならない。

lock 粒度は以下に固定する。

| 対象 | lock key | lock 種別 | 適用操作 |
|------|----------|-----------|----------|
| 起動時検証 | `global:startup` | exclusive | 起動時検証、必須 JSON / directory 検証 |
| migration | `global:migration` | exclusive | `asb migrate --apply` |
| Project | `project:{projectId}` | shared / exclusive | Project 参照、更新、削除、Project 配下操作 |
| Project 名 | `project-name:{name}` | exclusive | Project 作成時の重複確認と登録 |
| Domain | `domain:{host}` | shared / exclusive | Domain 追加、削除、参照、Host 解決 |
| SSL | `ssl:{host}` | shared / exclusive | 無料独自SSL enable、renew、disable、status |
| File | `file:{projectId}` | shared / exclusive | File upload、delete、list、静的配信 snapshot |
| Backup / Restore | `backup:{projectId}` | shared / exclusive | Backup 作成、Restore |
| Webhook deploy | `webhook:{projectId}` | exclusive | GitHub Webhook deploy |
| Log | `log:{kind}` | shared / exclusive | access log、error log、audit log の write、read、rotation |

同時操作の扱いは以下に固定する。

| 操作A | 操作B | 扱い |
|------|------|------|
| Project delete | 同一 Project 配下の全操作 | 競合。後続操作は `409 ERR_OPERATION_CONFLICT` |
| Project create | 同一 Project 名の Project create | 競合。後続操作は `409 ERR_PROJECT_ALREADY_EXISTS` または `409 ERR_OPERATION_CONFLICT` |
| Domain add/delete | 同一 Domain の add/delete/SSL 操作 | 競合。後続操作は `409 ERR_OPERATION_CONFLICT`、SSL 状態競合は `ERR_SSL_OPERATION_CONFLICT` |
| Domain read | Domain add/delete | read は確定済み snapshot を読む。途中状態を返してはならない |
| SSL enable/renew/disable | 同一 Domain の SSL 操作 | 競合。後続操作は `409 ERR_SSL_OPERATION_CONFLICT` |
| SSL status | SSL enable/renew/disable | status は直近の確定状態を返す。途中状態を成功扱いにしてはならない |
| File upload | 同一 Project の File upload/delete | 競合。後続操作は `409 ERR_OPERATION_CONFLICT` |
| File upload/delete | 同一 Project の静的 GET/HEAD | 静的配信は直近の確定済みファイルを読む。部分ファイルを公開してはならない |
| Webhook deploy | 同一 Project の File upload/delete | 競合。後続操作は `409 ERR_OPERATION_CONFLICT` |
| Webhook deploy | 同一 Project の Backup create/Restore | 競合。後続操作は `409 ERR_OPERATION_CONFLICT` |
| Backup create | 同一 Project の File upload/delete/Webhook deploy/Restore | 競合。後続操作は `409 ERR_OPERATION_CONFLICT` |
| Backup create | 同一 Project の静的 GET/HEAD | 静的配信は継続可。Backup は確定済み snapshot のみを対象にする |
| Restore | 同一 Project の File upload/delete/Webhook deploy/Backup create | 競合。後続操作は `409 ERR_OPERATION_CONFLICT` または `ERR_BACKUP_RESTORE_CONFLICT` |
| Restore | 同一 Project の静的 GET/HEAD | 静的配信は直近の確定済み公開状態を読む。復旧途中の `next/` を公開してはならない |
| Log write | 同一 log kind の rotation | 競合。後続操作は `409 ERR_OPERATION_CONFLICT` |
| Log read | 同一 log kind の write/rotation | read は確定済み JSON Lines snapshot を読む。取得できない場合は `ERR_LOG_READ_FAILED` |

lock 取得は無期限に待機してはならない。

互換性のない実行中操作がある場合は、request を background queue に積まず、即時に失敗させる。

lock 取得失敗時の既定 response は `409 Conflict`、`ERR_OPERATION_CONFLICT`、`Operation conflict` とする。

ただし、SSL の状態競合は `ERR_SSL_OPERATION_CONFLICT`、Backup restore 条件競合は `ERR_BACKUP_RESTORE_CONFLICT`、既存 resource 重複は既存の専用 error code を優先する。

整合性検証の実行タイミングは以下に固定する。

| タイミング | 検証対象 | 失敗時 |
|-----------|----------|--------|
| 起動時 | `config.json`、必須 directory、必須 JSON、schemaVersion、未知 field | 起動失敗。該当 error code を記録 |
| mutation API 実行前 | 対象 Project、Domain、File、SSL、Backup、Log の保存 JSON と実体 | request 失敗。保存状態を変更しない |
| mutation API 永続化後、response 前 | 更新済み JSON、実ファイル、quota、関連 index | 失敗 response。error log 記録 |
| Project delete 前 | Domain、File、Backup、SSL、Log 参照関係 | 不整合があれば削除しない |
| File upload/delete 後 | `files.json`、`contents/`、`projects.used` | 不整合を error log へ記録 |
| Webhook deploy 前 | webhook 設定、署名、対象 Project、source path | 失敗 response。deploy しない |
| Webhook deploy 後 | 公開 contents、`files.json`、Project 使用量 | 不整合を error log へ記録 |
| Backup create 前後 | 対象 Project snapshot、archive、SHA-256 | 失敗 response。破損 archive を有効履歴にしない |
| Restore 前後 | backup archive、`next/`、公開状態、`files.json` | 失敗 response。途中状態を公開しない |
| SSL status / renew | certificate file、metadata、期限、Domain 紐付け | SSL 専用 error code または error log |
| Log API read | 対象 JSON Lines の各行、limit、cursor | `ERR_LOG_READ_FAILED` |

整合性検証は不整合を検出するための処理であり、自動修復処理ではない。

整合性検証は、JSON、静的ファイル、証明書、backup archive、log file、directory を作成、削除、上書き、移動、rename してはならない。

保存状態を変更できる操作は、明示的に実行された API または CLI に限定する。

保存状態を変更できる操作は、Project create/delete、Domain add/delete、File upload/delete、SSL enable/renew/disable、Webhook deploy、Backup create、Restore、Log write/rotation、`asb migrate --apply` のみとする。

実行時データの不足補完、初期化、自動生成を目的とする CLI、API、background job、startup hook、maintenance task を実装してはならない。

整合性検証、startup validation、read API、静的 GET/HEAD、monitoring API は保存状態を変更してはならない。

開発リポジトリ内に実行時データ、lock file、検証結果ファイル、一時ファイル、cache、log、build output、`.gitignore` を生成してはならない。

#### 13.17.15 起動設定ファイル最終固定仕様

起動設定ファイル `config.json` は、ASB process 起動前に運用者が配置する単一 JSON ファイルとする。

ASB は起動設定ファイルを作成、補完、修復、整形、上書き、移動、コピーしてはならない。

起動設定ファイルの探索順序は以下に固定する。

1. `--config` に指定された絶対パス
2. `--config` 未指定時は `/etc/asb/config.json`

`--config` に相対パス、空文字、directory、存在しない path、通常ファイル以外の path が指定された場合は起動失敗とする。

`config.json` は UTF-8 JSON object のみを許可する。top-level array、文字列、数値、真偽値、`null`、空ファイル、複数JSON値を拒否する。

未知 field はすべて拒否する。未知 field 拒否は top-level、`server`、`storage`、`ssl`、`log`、`deploy`、`webhook` の各 object で行う。

設定値は読み込み後に default を適用し、validation を行う。validation 失敗時は起動失敗とする。

起動設定ファイルの固定 field は以下とする。

| field | 型 | 必須 | default | validation |
|-------|----|------|---------|------------|
| `server.port` | int | no | `3000` | 1以上65535以下 |
| `server.host` | string | no | `localhost` | `localhost`、`127.0.0.1`、`::1`、明示IP、明示host名のみ |
| `server.shutdownTimeout` | int | no | `30` | 1以上300以下 |
| `server.tlsCertFile` | string | yes | なし | 絶対パス、通常ファイル、読み取り可能 |
| `server.tlsKeyFile` | string | yes | なし | 絶対パス、通常ファイル、読み取り可能、group/world writable 禁止 |
| `storage.basePath` | string | no | `/var/asb` | 絶対パス、既存directory |
| `storage.maxProjectSize` | number | no | `1073741824` | 1以上1099511627776以下 |
| `ssl.email` | string | yes | なし | 空文字禁止、ASCII、254文字以下、単一 `@` |
| `ssl.renewBefore` | int | no | `7776000` | 86400以上15552000以下 |
| `ssl.renewCheckInterval` | int | no | `21600` | 3600以上604800以下 |
| `log.level` | string | no | `info` | `debug`、`info`、`warn`、`error` のみ |
| `log.format` | string | no | `json` | `json` のみ |
| `log.maxSize` | number | no | `104857600` | 1048576以上1073741824以下 |
| `deploy.projectId` | string | no | 空文字 | 空文字または UUID 形式 |
| `deploy.sourcePath` | string | no | `/srv/asb/source` | 絶対パス |
| `deploy.branch` | string | no | `main` | 空文字禁止、NUL、空白、`..`、先頭 `/` 禁止 |
| `webhook.githubSecret` | string | no | 空文字 | 4096文字以下、ログ出力禁止 |

`server.tlsCertFile`、`server.tlsKeyFile`、`storage.basePath`、`deploy.sourcePath` は、開発リポジトリ内 path を指定してはならない。

`storage.basePath` 配下の必須 directory、必須 JSON、ログファイル、証明書ファイルは起動時に存在検証のみを行い、不足していても作成してはならない。

起動設定ファイル内の secret 相当値は、stdout、stderr、error log、access log、panic recovery、API response、test failure message にそのまま出力してはならない。

secret 相当値は `webhook.githubSecret`、TLS private key path の内容、ACME token、管理者パスワード、将来の認証 secret を含む。

#### 13.17.16 CLI 境界固定仕様

ASB 本体 CLI は以下の subcommand のみを実装対象とする。

| command | 目的 | 保存状態変更 |
|---------|------|--------------|
| `asb start --config <path>` | HTTPS server 起動 | なし |
| `asb version` | version 表示 | なし |
| `asb migrate --storage <path> --from-schema <n> --to-schema <n> --dry-run` | migration 事前検証 | なし |
| `asb migrate --storage <path> --from-schema <n> --to-schema <n> --apply` | migration 実行 | あり |

`asb start` は起動設定読み込み、起動時検証、HTTPS server 起動、signal handling、graceful shutdown のみを行う。

`asb start` は実行時データ初期化、不足補完、空 JSON 作成、directory 作成、ログファイル作成、証明書作成、`.gitignore` 作成を行ってはならない。

`asb version` は stdout に `asb <version>` の1行のみを出力し、stderr、実行時データ、ログ、cache、一時ファイルを生成してはならない。

未定義 subcommand、未定義 option、必須 option 不足、option 型不正は exit code `2` とし、stderr に `ASB_CLI_ERROR code=ERR_INVALID_REQUEST message="Invalid command"` の1行のみを出力する。

以下の CLI は Rev.90 時点では実装禁止とする。

- `asb init`
- `asb init-runtime`
- `asb repair`
- `asb doctor --fix`
- `asb create-config`
- `asb generate-config`
- `asb generate-cert`
- `asb seed`
- `asb dev`
- `asb serve`

CLI の終了コードは以下に固定する。

| exit code | 条件 |
|-----------|------|
| `0` | 正常終了 |
| `1` | 起動時検証、migration、永続化、内部処理の失敗 |
| `2` | CLI 引数、subcommand、option の不正 |

CLI は stderr に stack trace、内部ファイルパス、secret、request body、管理者パスワード、webhook secret、private key、ACME token を出力してはならない。

#### 13.17.17 起動失敗・stderr 固定仕様

起動失敗時は HTTPS server listen を開始してはならない。

起動失敗時は stdout へ何も出力してはならない。

起動失敗時の stderr は以下の単一行形式に固定する。

```text
ASB_STARTUP_ERROR code=<code> message="<message>"
```

`code` は §13.17.14 の error code 固定表に含まれる値のみを使用する。

`message` は固定文言のみを使用し、検証対象 path、secret、JSON断片、内部構造、stack trace を含めてはならない。

起動失敗時は access log、error log、監査ログ、起動状態JSON、検証結果JSON、cache、一時ファイルを作成してはならない。

起動時検証の順序は以下に固定する。

1. CLI 引数検証
2. 起動設定ファイル path 検証
3. 起動設定ファイル JSON decode
4. 起動設定ファイル unknown field 検証
5. 起動設定 default 適用
6. 起動設定値 validation
7. `storage.basePath` directory 検証
8. 必須 runtime directory 検証
9. 必須 runtime JSON 検証
10. 必須ログファイル検証
11. TLS証明書・秘密鍵検証
12. 依存関係生成
13. HTTPS server listen

各段階で失敗した場合は、後続段階を実行してはならない。

#### 13.17.18 ログ出力最終固定仕様

アクセスログとエラーログは UTF-8 JSON Lines とし、1イベントを1行で追記する。

LogService は既存ファイルへの追記と既存ファイルの rotation のみを行う。ログファイルが存在しない場合は作成せず失敗する。

アクセスログの field 順序は以下に固定する。

1. `timestamp`
2. `requestId`
3. `method`
4. `path`
5. `status`
6. `durationMs`
7. `remoteAddr`
8. `userAgent`

エラーログの field 順序は以下に固定する。

1. `timestamp`
2. `requestId`
3. `level`
4. `code`
5. `message`
6. `operation`

ログ出力では以下を必ずマスクまたは出力禁止とする。

- 管理者パスワード
- webhook secret
- private key
- ACME token
- request body
- TLS証明書秘密鍵内容
- 環境変数
- 内部ファイルパス
- stack trace

ログ書き込み、fsync、rotation、JSON encode に失敗した場合、成功レスポンスを返してはならない。

エラーレスポンス生成中に error log 書き込みへ失敗した場合でも、代替ログ、panic dump、cache、一時ファイルを生成してはならない。

ログ API は現行ログファイルを読み取るのみとし、読み取りを理由にログファイル作成、rotation、修復、切り詰め、退避、補助 index 作成を行ってはならない。

#### 13.17.19 package 境界最終固定仕様

ASB 本体の package 境界は `cmd/asb` と `internal/` 配下の固定 package のみに限定する。

`internal/config` は HTTP、Repository、Storage、Log、Domain、Project、SSL、Webhook、Backup の業務判断へ依存してはならない。

`internal/server` は JSON ファイル path、atomic save、Project quota、Domain 永続化、SSL証明書状態、Backup archive の業務判断を持ってはならない。

`internal/management` は HTTP request/response の具体処理、JSONファイルの atomic save 実装、OS signal handling、CLI 引数解析を持ってはならない。

`internal/delivery` は起動設定読み込み、CLI 引数解析、単一システム管理者パスワード変更処理、Backup restore 実装を持ってはならない。

`internal/data` は HTTP status、HTTP header、CLI stdout/stderr、Domain 割当判断、Project quota policy、Webhook署名判断を持ってはならない。

`internal/system` は Project 作成、Domain 割当、File upload、Backup restore、SSL enable の業務フローを持ってはならない。

`cmd/asb/main.go` は各 package の依存関係生成と実行境界のみを扱い、業務判断、JSON schema、HTTP response body 生成を持ってはならない。

package 間の依存方向は以下に固定する。

```text
cmd/asb -> config, server, management, delivery, data, system
server -> management, delivery, system
management -> data, system
delivery -> data, system
data -> system
system -> 標準ライブラリのみ
config -> 標準ライブラリのみ
```

上記以外の package 依存、循環依存、外部公開 package、adapter package、plugin package、legacy package、旧プロジェクト名 package を追加してはならない。

#### 13.17.20 保存・生成・副作用境界固定仕様

ASB の保存状態変更は、仕様で明示された API または CLI の成功条件内でのみ発生させる。

JSON 保存、ファイル実体操作、Backup / Restore、Log rotation、SSL / ACME、Webhook deploy は、それぞれ本節の生成境界を超えてはならない。

JSON 保存の副作用境界は以下に固定する。

- 保存先は `storage.basePath` 配下の仕様定義済み JSON のみに限定する。
- 保存時は既存 JSON 読み込み、schema validation、unknown field 拒否、deterministic output 生成、同一ディレクトリ内一時ファイル書き込み、file `fsync`、atomic rename、directory `fsync` の順に行う。
- 一時ファイルは対象 JSON と同一ディレクトリにのみ作成し、開発リポジトリ内、OS共通一時ディレクトリ、cache directory へ作成してはならない。
- JSON encode、write、file `fsync`、atomic rename、directory `fsync`、保存後 validation のいずれかに失敗した場合は成功レスポンスを返してはならない。
- 保存失敗時に代替 JSON、復旧 JSON、差分 JSON、journal、write-ahead log、補助 index、cache を生成してはならない。

ファイル実体操作の副作用境界は以下に固定する。

- Project 作成 API のみ、対象 Project の `storage/projects/{projectId}/`、`contents/`、`files.json` を新規作成できる。
- File upload / overwrite は対象 Project の `contents/` 配下にのみ一時ファイル、退避ファイル、公開ファイルを作成できる。
- File delete は対象 `contents/` 配下の既存公開ファイルと `files.json`、`projects.json` の更新のみを変更できる。
- File API は未記録ファイルを発見しても自動登録、自動削除、自動修復してはならない。
- 静的配信、File list、Monitoring、Log API、Health Check はファイル実体、`files.json`、`projects.json` を変更してはならない。

Backup / Restore の副作用境界は以下に固定する。

- Backup 作成で生成できるファイルは `storage.basePath/backups/.tmp/backup-{backupId}.tar.gz.tmp` と `storage.basePath/backups/backup-{backupId}.tar.gz` のみに限定する。
- Backup 作成で更新できる JSON は `config/backups.json` のみとする。
- Restore で生成できる directory は `storage.basePath/backups/restore-staging/{restoreId}/previous/` と `storage.basePath/backups/restore-staging/{restoreId}/next/` のみに限定する。
- Restore で変更できる対象は復旧対象 Project の `files.json`、`contents/`、restore staging、必要な error log のみとする。
- Backup / Restore は開発リポジトリ内に archive、checksum、展開データ、退避データ、一時ファイルを作成してはならない。
- Backup / Restore の整合性検証は不整合を自動修復してはならない。

Log rotation の副作用境界は以下に固定する。

- Log rotation は既存の `access.log` または `error.log` を `access.log.{unixTime}` または `error.log.{unixTime}` へ rename し、新しい現行ログファイルを作成する場合に限り許可する。
- 現行ログファイルが存在しない場合、LogService は初回作成せず失敗しなければならない。
- rotation 失敗時に補助ログ、rotation state、index、cache、queue、一時ディレクトリを作成してはならない。
- ログ読み取り、ログ検証、監視、Health Check を理由にログファイルを作成、修復、切り詰め、退避、再生成してはならない。

SSL / ACME の副作用境界は以下に固定する。

- 無料独自SSL enable、renew、disable、ACME 更新処理のみが、仕様定義済み ACME JSON、証明書ファイル、秘密鍵ファイル、HTTP-01 challenge 応答状態を変更できる。
- 起動時、Health Check、Monitoring、Domain list、SSL status read は ACME JSON、証明書ファイル、秘密鍵ファイル、challenge file を生成してはならない。
- HTTP-01 challenge 応答は、仕様で管理された challenge token のみを返し、開発リポジトリ内または公開 `contents/` 配下へ challenge file を生成してはならない。
- DNS-01、TLS-ALPN-01、wildcard、複数CA、DNS provider API を理由に設定、JSON、cache、token、challenge file を生成してはならない。

Webhook deploy の副作用境界は以下に固定する。

- Webhook deploy 成功時に変更できる対象は、対象 Project の `contents/`、`files.json`、`projects.json`、`config/webhooks.json`、必要な access log / error log のみに限定する。
- `deploy.sourcePath` は既存 checkout を読むのみとし、ASB は checkout、clone、pull、build、package install、dependency install、cache 生成を行ってはならない。
- Webhook deploy 失敗時に deploy state file、retry queue、failed payload dump、checkout cache、build cache、一時ログを開発リポジトリ内または `storage.basePath` 配下へ生成してはならない。
- Webhook deploy で一時ファイルが必要な場合は、対象 Project の `contents/` 配下または仕様定義済み staging のみに限定し、成功または失敗時に仕様通り削除または不整合検出対象にする。

開発リポジトリは、いかなる runtime、cache、queue、log、一時ファイル、backup、証明書、challenge、deploy state、migration 作業ファイルの保存先にもしてはならない。

#### 13.17.21 テストファイル配置固定

Rev.90 の実装では、実装 package と同じ責務単位でテストファイルを配置する。

テストファイル名は、対象ファイル名または対象責務名に `_test.go` を付与した名前に固定する。

`internal/config/` は設定読み込み、未知フィールド拒否、デフォルト値、起動時検証をテストする。

`internal/server/` はルーティング、Content-Type、Body decode、成功レスポンス、エラーレスポンス、requestId をテストする。

`internal/management/` は Project、Domain、SSL 管理境界の validation、Service、Handler をテストする。

`internal/delivery/` は File 管理、静的配信、GitHub Webhook の validation、Service、Handler をテストする。

`internal/data/` は JSON Repository、StorageService、BackupService、マイグレーションをテストする。

`internal/system/` は Clock、IDGenerator、AuthService、LogService、MonitoringService をテストする。

禁止機能の非実装確認は、実装 package 横断のテストまたはレビューで確認する。

テストは開発リポジトリ内に実行時データ、ログ、一時ファイル、ビルド成果物を残してはならない。

テストで実行時データ領域が必要な場合は、テストごとに OS の一時ディレクトリを作成し、テスト終了時に削除する。

テスト用 fixture をリポジトリ内に追加する場合は、静的な入力データのみ許可する。

テスト fixture は実行結果、ログ、バックアップ、ビルド成果物、coverage 出力を含んではならない。

テスト fixture として許可するファイルは、JSON 入力例、HTML/CSS/JavaScript/画像の静的配信入力例、Webhook payload 入力例、証明書検証用の固定 PEM 入力例に限定する。

テスト fixture はテスト実行中に更新してはならない。

テスト fixture から生成した出力は、OS の一時ディレクトリ配下にのみ作成し、テスト終了時に削除する。

### 13.18 単一システム管理者認証固定仕様

Rev.90 時点の ASB は単一ユーザー、単一システム管理者モデルとする。

システム管理者は ASB 管理 API の全操作権限を持つ。

Rev.90 時点では、ユーザー一覧、ユーザーID、ロール、権限分離、組織、チーム、テナントを持たない。

管理 API は、`/api/` で始まる HTTPS JSON API のうち `POST /api/webhook/github` を除く API とする。

管理 API は、ASB の標準構成では `server.host=localhost` で待ち受ける。

本番環境で管理 API を外部ネットワークから利用可能にする場合も、ASB 管理 API は HTTPS JSON API として提供する。

ASB 本体は、`server.tlsCertFile` と `server.tlsKeyFile` に指定された証明書ファイルと秘密鍵ファイルを用いて HTTPS 管理 API を起動する。

ASB 本体は、管理 API 用 TLS 証明書ファイルまたは秘密鍵ファイルを開発リポジトリ内へ生成してはならない。

ASB 本体は、管理 API 用 TLS 証明書ファイルまたは秘密鍵ファイルを起動時に自動生成してはならない。

`POST /api/auth/change-password` を除く管理 API は、`X-ASB-Admin-Password` ヘッダーによる単一システム管理者パスワード認証を必須とする。

`X-ASB-Admin-Password` が未指定、空文字、不一致の場合は `401 Unauthorized` とし、`ERR_AUTH_FAILED` を返す。

`POST /api/webhook/github` は GitHub Webhook 署名検証のみを認証境界とし、`X-ASB-Admin-Password` を要求してはならない。

静的コンテンツ配信、ACME HTTP-01 challenge 応答、ヘルスチェックは管理 API ではないため、`X-ASB-Admin-Password` を要求してはならない。

初期デフォルトパスワードは `asb-admin-change-me` とする。

初期デフォルトパスワードは bootstrap 用の一時値であり、通常運用に使用してはならない。

`config/auth.json` の `admin.passwordChanged` が `false` の場合、ASB は `POST /api/auth/change-password` 以外の管理 API を拒否し、`403 Forbidden` と `ERR_AUTH_PASSWORD_CHANGE_REQUIRED` を返す。

管理者パスワード変更 API は `POST /api/auth/change-password` とする。

管理者パスワード変更 API の request body は以下とする。

```json
{
  "currentPassword": "string",
  "newPassword": "string"
}
```

`currentPassword` は現在有効な管理者パスワードと完全一致しなければならない。

管理者パスワード変更 API は `X-ASB-Admin-Password` ヘッダーを要求せず、`currentPassword` を認証入力として扱う。

`newPassword` は以下を満たさなければならない。

- 12文字以上128文字以下
- 初期デフォルトパスワード `asb-admin-change-me` と一致しない
- NUL 文字および制御文字を含まない
- 先頭または末尾に空白文字を含まない

管理者パスワード変更成功時は `config/auth.json` を atomic rename で更新し、`200 OK` と以下を返す。

```json
{
  "status": "password_changed",
  "updatedAt": "2026-09-09T00:00:00Z"
}
```

`config/auth.json` は単一システム管理者認証状態を保存する唯一の JSON ファイルとする。

`config/auth.json` の空状態は以下とする。

```json
{
  "schemaVersion": 1,
  "admin": {
    "passwordHash": "",
    "passwordSalt": "",
    "passwordChanged": false,
    "updatedAt": ""
  }
}
```

平文パスワードを JSON、ログ、標準出力、標準エラー、エラーレスポンスへ出力または保存してはならない。

パスワードハッシュは Go 標準ライブラリのみで実装する。

ハッシュ方式は PBKDF2-HMAC-SHA256 相当処理を `crypto/hmac`、`crypto/sha256`、`crypto/rand`、`crypto/subtle`、`encoding/base64` で内製実装する。

salt は32 bytes、hash は32 bytes、iteration は210000回とする。

`passwordHash` と `passwordSalt` は `base64.RawURLEncoding` で保存する。

パスワード照合は同一方式で導出した hash を `crypto/subtle.ConstantTimeCompare` で比較する。

ASB 本体は Rev.90 時点では以下を実装してはならない。

- APIキー発行
- APIキー保存
- APIキー照合
- APIキー失効
- APIキーローテーション
- APIキー権限スコープ
- APIキー監査履歴
- 複数ユーザー管理
- ロール・権限分離
- 組織管理
- チーム管理
- テナント管理
- セッション管理
- JWT 検証
- OAuth / OIDC 連携
- Basic 認証
- Bearer token 認証
- cookie 認証

`Authorization` ヘッダーまたは `X-API-Key` ヘッダーを受信しても、ASB は認証判断に使用してはならない。

Rev.90 時点では、`Authorization` ヘッダーまたは `X-API-Key` ヘッダーの有無により、成功・失敗・レスポンス内容を変えてはならない。

ASB は APIキー管理、複数ユーザー管理、セッション管理のために以下の JSON ファイル、ディレクトリ、設定項目を作成してはならない。

- `config/api_keys.json`
- `config/users.json`
- `config/sessions.json`
- `config/roles.json`
- `storage/api_keys/`
- `storage/users/`
- `storage/sessions/`
- `auth.users`
- `auth.sessions`
- `apiKey.*`

開発リポジトリ内にも、`storage.basePath` 配下にも、APIキー、複数ユーザー、ロール、セッションを表す実行時データを生成してはならない。

起動設定ファイル `config.json` に認証関連フィールドが存在する場合は、未知フィールドとして起動失敗とする。

APIキー管理または複数ユーザー化を将来実装する場合は、実装前に `ASB-spec.md` を改訂し、以下を最低限確定する。

- ユーザー識別子
- パスワード以外の認証方式を採用するかどうか
- セッションまたは token の要否
- APIキー保存形式
- APIキーのハッシュ化方式
- 発行、失効、ローテーション API
- 権限スコープ
- 管理 API への適用範囲
- SDK との認証連携
- Web UI のロール表示
- 監査ログ
- 単一システム管理者認証からの移行手順

SDK 認証拡張仕様は Rev.80 の対象外とし、実装対象へ昇格する場合は事前に `ASB-spec.md` を改訂する。

---

### 13.19 Rate limiting 固定仕様

Rev.90 時点では、ASB 本体に Rate limiting を実装しない。

Rate limiting とは、送信元IP、Host、Domain、Project、APIキー、ユーザー、HTTPメソッド、URL path、リクエスト数、転送量、同時接続数、時間窓等に基づき、HTTP リクエストの受理、拒否、遅延、または優先度を制御する機能を指す。

Rate limiting は、認証、認可、入力バリデーション、ファイルサイズ上限、プロジェクト容量上限、HTTP timeout、TLS handshake timeout、Webhook署名検証、ACME rate limit handling とは別機能として扱う。

Rev.90 時点の ASB は、Rate limiting を内部保護機構、運用補助機構、互換目標、または将来拡張の下地として部分実装してはならない。

ASB 本体は Rev.90 時点では以下を実装してはならない。

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
- Rate limiting 用レスポンス body
- Rate limiting 用 error code
- Rate limiting 用監査イベント
- Rate limiting 用メトリクス
- Rate limiting 用設定検証

ASB は Rate limiting のために以下の JSON ファイル、ディレクトリ、設定項目を作成してはならない。

- `config/rate_limits.json`
- `config/limits.json`
- `config/traffic_limits.json`
- `config/abuse_limits.json`
- `storage/rate_limits/`
- `storage/counters/`
- `storage/traffic_limits/`
- `storage/abuse_limits/`
- `rateLimit.*`
- `limits.*`
- `trafficLimit.*`
- `abuseLimit.*`

開発リポジトリ内にも、`storage.basePath` 配下にも、Rate limiting の判定状態、集計値、カウンタ、時間窓、送信元別状態を表す実行時データを生成してはならない。

起動設定ファイル `config.json` に Rate limiting 関連フィールドが存在する場合は、未知フィールドとして起動失敗とする。

管理 API、静的配信、Webhook 受信は、Rate limiting の有無により成功・失敗・レスポンス内容を変えてはならない。

Rate limiting を理由に、管理 API、静的配信、Webhook 受信、ACME HTTP-01 challenge 応答、ヘルスチェックのルーティング順序を変更してはならない。

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
- ASB 外部 Rate limiting 構成との責務分界
- 既存 API の互換性
- 既存の Rate limiting なし構成からの移行手順

---

### 13.20 Brotli 圧縮固定仕様

Rev.90 時点では、ASB 本体に Brotli 圧縮を実装しない。

ASB の標準圧縮機能は、Go 標準ライブラリ `compress/gzip` で実装できる Gzip に限定する。

Brotli 圧縮は Go 標準ライブラリに含まれないため、Rev.90 時点では外部ライブラリ例外採用を行わない。

ASB 本体は Rev.90 時点では以下を実装してはならない。

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

起動設定ファイル `config.json` に Brotli 関連フィールドが存在する場合は、未知フィールドとして起動失敗とする。

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

### 13.20.1 HTTP/2 固定仕様

HTTP/2 は ASB互換目標として扱う。

Rev.90 時点では、ASB 本体に HTTP/2 を実装しない。

ASB 本体の管理 API、静的配信、Webhook、ACME HTTP-01 challenge 応答は HTTP/1.1 で提供する。

ASB 本体は Rev.90 時点では以下を実装してはならない。

- HTTP/2 専用設定項目
- h2c
- ALPN の独自制御
- HTTP/2 server push
- HTTP/2 stream priority
- HTTP/2 専用 handler
- HTTP/2 専用 middleware
- HTTP/2 専用ログ項目
- HTTP/2 専用テスト前提

Go 標準 `net/http` の利用により HTTP/2 が暗黙的に有効化されることを避けるため、Rev.90 の実装では `http.Server.TLSNextProto` を空 map に設定し、HTTP/2 自動有効化を無効化する。

起動設定ファイル `config.json` に HTTP/2 関連フィールドが存在する場合は、未知フィールドとして起動失敗とする。

HTTP/2 を将来実装する場合は、実装前に `ASB-spec.md` を改訂し、以下を最低限確定する。

- 適用範囲
- ALPN 設定
- h2c 採否
- TLS 証明書との関係
- 管理 API、静的配信、Webhook、ACME HTTP-01 challenge の扱い
- ログ項目
- テスト条件
- 既存 HTTP/1.1 実装からの移行手順

---

### 13.21 ASB SDK 通信層固定仕様

Rev.90 時点では、公式SDK名称を ASB SDK に固定する。

ASB SDK は、別製品名または別プロジェクト名として分割しない。

ASB SDK は、以下の対応実装を持つ単一の公式SDKとして扱う。

| 対応実装 | 実行環境 | 用途 | 標準ファイル/配置 | 実装方針 |
|---------|----------|------|------------------|----------|
| Browser JavaScript | Browser | ASB 標準Web UI 用通信層 | `webui/asb-sdk.js` | 静的 ES module、ブラウザ Web 標準 API のみ |
| Deno専用 TypeScript | Deno | Deno 利用者向け通信層 | `sdk/deno/asb-sdk.ts` | Deno 専用、Web 標準 API と Deno runtime API の範囲 |
| Go | Go | Go 利用者向け通信層 | `sdk/go/` | Go標準ライブラリ中心 |

ASB SDK は、ASB 管理 HTTPS JSON API の request / response / error / pagination / upload 規約をクライアント側から扱うための共通部品である。

ASB SDK は、ASB 本体を拡張する機構ではなく、ASB 管理 HTTPS JSON API の型付き呼び出し層である。

ASB SDK の各対応実装は、ASB 本体の内部 JSON、Service、Repository、Storage を直接参照または呼び出してはならない。

ASB SDK の各対応実装は、ASB 管理 HTTPS JSON API に存在しない操作を公開 API として提供してはならない。

ASB SDK の各対応実装は、`baseUrl` を必須入力として ASB 管理 HTTPS JSON API の呼び出し先を決定する。

`baseUrl` は `https://` scheme の URL のみ許可する。

`http://`、相対URL、空文字、schemeなしURL、WebSocket URL、独自schemeを `baseUrl` として許可してはならない。

ASB SDK は `baseUrl` 末尾の `/` の有無に依存せず、ASB 管理 API path を単一の `/` で結合する。

ASB SDK の各対応実装の自動 retry 回数は Rev.90 時点では `0` とし、SDK は失敗した HTTP request を自動再送してはならない。

ASB SDK は、SDK 固有の保存データ、設定ファイル、生成ファイル、生成ディレクトリを持たない。

ASB SDK は、単一システム管理者パスワードを各 request の入力として受け取り、`X-ASB-Admin-Password` ヘッダーに設定する。

ASB SDK は、管理者パスワードを SDK 内部の永続状態、設定ファイル、ブラウザストレージ、cookie、セッション、キャッシュへ保存してはならない。

SDK 認証拡張仕様、デスクトップアプリ向けSDK利用、モバイルアプリ向けSDK利用は Rev.90 時点では実装対象外とし、確定仕様へ昇格するまで API、設定項目、JSON、ディレクトリ、外部依存、実行時データを追加してはならない。

ASB SDK の通信規格は、ASB 本体が提供する HTTPS JSON API と同一に固定する。

ASB SDK の公開APIは、ASB 管理 HTTPS JSON API の endpoint 単位に対応する関数または method とする。

ASB SDK の公開API名、引数、戻り値は、ASB 管理 HTTPS JSON API の method、path、request、response、error に対応していなければならない。

ASB SDK は、内部で受信した成功レスポンスJSONとエラーレスポンスJSONを仕様外キー追加なしで返す。

ASB SDK は、通信エラー、timeout、JSON decode失敗、ASB error response を区別できる error 型または error object を提供する。

ASB SDK は、ASB 本体が返した `code`、`message`、`requestId` を破棄、改名、翻訳してはならない。

SDK 通信は、以下の ASB 管理 API 規約に従う。

- HTTP method
- URL path
- `X-ASB-Admin-Password` ヘッダー
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

#### 13.21.1 ASB SDK Browser JavaScript 実装固定仕様

ASB SDK の Browser JavaScript 実装は、ASB 標準Web UI が ASB と通信するために使用する。

ASB SDK の Browser JavaScript 実装の実行環境はブラウザに限定する。

ASB SDK の Browser JavaScript 実装は、以下の Web 標準 API の範囲で実装する。

- `fetch`
- `URL`
- `URLSearchParams`
- `Headers`
- `FormData`
- `Blob`
- `AbortController`
- `Promise`

ASB SDK の Browser JavaScript 実装は、Node.js 実行環境を前提としてはならない。

ASB SDK の Browser JavaScript 実装は、npm 配布、package manager、bundler、transpiler、generated client、外部ライブラリを前提としてはならない。

ASB SDK の Browser JavaScript 実装のファイル形式は、ブラウザで読み込み可能な通常の JavaScript とする。

ASB SDK の Browser JavaScript 実装の標準ファイル名は `asb-sdk.js` とする。

ASB SDK の Browser JavaScript 実装は、ブラウザ標準の ES module として読み込む。

ASB SDK の Browser JavaScript 実装は `webui/asb-sdk.js` に配置する。

ASB SDK の Browser JavaScript 実装は、package 名を持たない。

ASB SDK の Browser JavaScript 実装は、request timeout を `AbortController` で扱う。

ASB SDK の Browser JavaScript 実装は、ブラウザストレージ、cookie、Service Worker、Cache Storage、IndexedDB を SDK 通信用の永続状態として使用してはならない。

ASB SDK の Browser JavaScript 実装は、管理者パスワードを request 単位の引数または呼び出しオプションとして受け取り、request 完了後に参照を保持してはならない。

ASB SDK の Browser JavaScript 実装は、global object へ SDK API を自動登録してはならない。

ASB SDK の Browser JavaScript 実装は、ES module の named export により公開APIを提供する。

#### 13.21.2 ASB SDK Deno専用 TypeScript 実装固定仕様

ASB SDK の Deno専用 TypeScript 実装は、Deno 専用ランタイムで動作する ASB 管理 HTTPS JSON API クライアントとして実装する。

ASB SDK の Deno専用 TypeScript 実装の標準ファイル名は `asb-sdk.ts` とする。

ASB SDK の Deno専用 TypeScript 実装は `sdk/deno/asb-sdk.ts` に配置する。

ASB SDK の Deno専用 TypeScript 実装は、Deno から直接 import できる TypeScript module として実装する。

ASB SDK の Deno専用 TypeScript 実装は、Deno runtime API と Web 標準 API の範囲で実装する。

ASB SDK の Deno専用 TypeScript 実装は、以下を前提としてはならない。

- Node.js
- npm
- package manager
- `package.json`
- `node_modules/`
- `deno.json`
- `deno.lock`
- bundler
- transpiler
- generated client
- 外部ライブラリ

ASB SDK の Deno専用 TypeScript 実装は、Deno 以外の JavaScript runtime での動作を互換目標として扱ってはならない。

ASB SDK の Deno専用 TypeScript 実装は、SDK 専用 JSON ファイル、SDK 専用ディレクトリ、SDK 専用設定項目、SDK 専用実行時データを作成してはならない。

ASB SDK の Deno専用 TypeScript 実装は、HTTP request timeout を `AbortController` で扱う。

ASB SDK の Deno専用 TypeScript 実装は、ASB 管理 HTTPS JSON API の型付き request / response / error を提供する。

ASB SDK の Deno専用 TypeScript 実装は、管理者パスワードを request 単位の引数または呼び出しオプションとして受け取り、ファイル、環境変数、Deno KV、local cache へ保存してはならない。

#### 13.21.3 ASB SDK Go 実装固定仕様

ASB SDK の Go 実装は、Go 利用者向けの ASB 管理 HTTPS JSON API クライアントとして実装する。

ASB SDK の Go 実装は、Go 1.21 以上を前提とする。

ASB SDK の Go 実装は `sdk/go/` 配下に配置する。

ASB SDK の Go 実装は、Go標準ライブラリで実装可能な部分を Go標準ライブラリで実装する。

ASB SDK の Go 実装は、`net/http`、`net/url`、`encoding/json`、`context`、`time`、`mime/multipart` の範囲を中心に実装する。

ASB SDK の Go 実装は、外部HTTP client library、外部JSON library、generated client を前提としてはならない。

ASB SDK の Go 実装は、ASB 本体の `internal/` package を import してはならない。

ASB SDK の Go 実装は、SDK 専用 JSON ファイル、SDK 専用ディレクトリ、SDK 専用設定項目、SDK 専用実行時データを作成してはならない。

ASB SDK の Go 実装は、HTTP request timeout と cancellation を `context.Context` と `http.Client` で扱う。

ASB SDK の Go 実装は、ASB 管理 HTTPS JSON API の型付き request / response / error を提供する。

ASB SDK の Go 実装の package 名は `asb` とする。

ASB SDK の Go 実装は、管理者パスワードを request 単位の引数または呼び出しオプションとして受け取り、ファイル、環境変数、global 変数、cache へ保存してはならない。

ASB SDK の Go 実装で `go.mod` を作成する場合、module path は `github.com/fqwink/Adlaire-Static-Base/sdk/go` に固定する。

ASB SDK の Go 実装の tag は ASB 本体の安定版リリースタグと同一にする。

ASB SDK の Go 実装は Rev.90 時点では外部配布サービスへ登録しない。

ASB SDK の Go 実装は、`go.mod` を作成する場合でも外部 module dependency を追加してはならない。

#### 13.21.4 SDK 共通禁止事項

SDK 通信のために、ASB 本体は Rev.90 時点では以下を実装してはならない。

- SDK 専用 HTTPS API
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
- SDK 専用 browser storage
- SDK 専用 localStorage
- SDK 専用 sessionStorage
- SDK 専用 IndexedDB

ASB 本体は Rev.90 時点では以下の通信方式を SDK 通信として実装してはならない。

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
- `package.json`
- `node_modules/`
- `deno.json`
- `deno.lock`
- `dist/`
- `build/`

開発リポジトリ内にも、`storage.basePath` 配下にも、SDK 通信用のセッション、キャッシュ、handshake 状態、protocol negotiation 状態、client registration 状態を表す実行時データを生成してはならない。

起動設定ファイル `config.json` に SDK 通信関連フィールドが存在する場合は、未知フィールドとして起動失敗とする。

SDK から ASB 管理 API を呼び出す場合、SDK は `X-ASB-Admin-Password` を使用する。

SDK から ASB 管理 API を呼び出す場合でも、Rev.90 時点では `Authorization` ヘッダー、`X-API-Key` ヘッダー、cookie、セッションIDを認証判断に使用してはならない。

ASB 標準Web UI は、ASB SDK の Browser JavaScript 実装を経由して上記の ASB 管理 API 規約に従う。

ASB 標準Web UI は、SDK 専用プロトコル、SDK 専用エンドポイント、SDK 専用セッション、SDK 専用 JSON ファイル、SDK 専用ディレクトリを要求してはならない。

SDK 認証拡張仕様を将来実装する場合は、実装前に `ASB-spec.md` を改訂し、APIキー管理、複数ユーザー化、SDK配布方針との関係を最低限確定する。

#### 13.21.4.1 SDK 認証拡張固定仕様

SDK 認証拡張とは、ASB SDK が `X-ASB-Admin-Password` による単一システム管理者パスワード認証以外の認証方式、認証補助方式、認証状態管理、または認証情報ライフサイクルを扱う機能を指す。

SDK 認証拡張には以下を含む。

- APIキー
- access token
- refresh token
- session ID
- login / logout API
- token refresh API
- client registration
- client secret
- OAuth / OIDC
- JWT
- cookie 認証
- Bearer token 認証
- mTLS client certificate 認証
- 権限 scope
- SDK 内 credential store
- SDK 内 token cache
- SDK 内 session cache

Rev.90 時点では、SDK 認証拡張を実装しない。

Rev.90 時点の ASB SDK は、認証入力として単一システム管理者パスワードのみを request 単位で受け取る。

Rev.90 時点の ASB SDK は、管理者パスワードを `X-ASB-Admin-Password` ヘッダーへ設定する以外の認証処理を行ってはならない。

ASB SDK は Rev.90 時点では以下を公開 API として提供してはならない。

- `login`
- `logout`
- `refreshToken`
- `createSession`
- `deleteSession`
- `createApiKey`
- `deleteApiKey`
- `rotateApiKey`
- `listApiKeys`
- `setToken`
- `setSession`
- `setCredentialStore`

ASB SDK は SDK 認証拡張のために以下を使用してはならない。

- Browser storage
- cookie
- IndexedDB
- Cache Storage
- Service Worker
- Deno KV
- 環境変数
- ローカルファイル
- OS keychain
- global 変数
- package manager 設定

ASB 本体は SDK 認証拡張のために以下を実装してはならない。

- SDK 専用認証 API
- SDK 専用 token 発行 API
- SDK 専用 session API
- SDK 専用 API key API
- SDK client 登録 API
- SDK client secret 検証
- SDK 用権限 scope

SDK 認証拡張のために以下の JSON ファイル、ディレクトリ、設定項目を作成してはならない。

- `config/sdk_auth.json`
- `config/sdk_clients.json`
- `config/sdk_tokens.json`
- `config/sdk_sessions.json`
- `storage/sdk_auth/`
- `storage/sdk_tokens/`
- `storage/sdk_sessions/`
- `sdkAuth.*`
- `sdkClients.*`
- `sdkTokens.*`
- `sdkSessions.*`

開発リポジトリ内にも、`storage.basePath` 配下にも、SDK 認証拡張用の token、session、client registration、credential cache、scope、secret を表す実行時データを生成してはならない。

起動設定ファイル `config.json` に SDK 認証拡張関連フィールドが存在する場合は、未知フィールドとして起動失敗とする。

SDK 認証拡張を将来実装する場合は、実装前に `ASB-spec.md` を改訂し、以下を最低限確定する。

- 認証方式
- 対象 SDK 実装
- ASB 管理 API との関係
- 単一システム管理者認証との併存または置換
- APIキー管理との関係
- 複数ユーザー化との関係
- token または session の保存要否
- JSON保存形式
- 失効、更新、ローテーション
- SDK 公開 API
- Web UI での入力、保持、破棄
- 監査ログ
- migration
- downgrade
- テスト条件

ASB SDK の追加実装詳細を確定する場合は、実装前に `ASB-spec.md` を改訂し、以下を最低限確定する。

- ファイル名
- 読み込み方式
- versioning
- ASB API version との互換性
- エラー型
- retry 方針
- timeout 方針
- upload API の扱い
- 認証仕様
- テスト方法

#### 13.21.5 ASB 標準Web UI 固定仕様

Rev.90 時点では、ASB Web UI を対応必須とする。

ASB Web UI は、ASB 標準Web UIとして扱う。

ASB 標準Web UI は、ASB 本体外の内製管理画面クライアントである。

ASB 標準Web UI は、ブラウザで動作する静的 HTML / CSS / JavaScript アプリケーションとして実装する。

ASB 標準Web UI は、Node.js 実行環境、npm 配布、package manager、bundler、transpiler、外部フレームワーク、外部ライブラリを前提としてはならない。

ASB 標準Web UI は、ブラウザ標準 API と ASB SDK の Browser JavaScript 実装のみを使用する。

ASB 標準Web UI は、ASB SDK の Browser JavaScript 実装である `asb-sdk.js` をブラウザ標準 ES module として読み込む。

ASB 標準Web UI は、ASB SDK の Browser JavaScript 実装を利用して ASB 管理 HTTPS JSON API と通信する。

ASB 標準Web UI は、ASB SDK の Browser JavaScript 実装を経由せずに `fetch` または `XMLHttpRequest` で ASB 管理 API を直接呼び出してはならない。

ASB 標準Web UI は、ASB 本体に内包してはならない。

ASB 標準Web UI は、ASB 本体の内部 JSON、Service、Repository、Storage を直接参照または呼び出してはならない。

ASB 標準Web UI は、ASB 管理 HTTPS JSON API に存在しない操作を前提としてはならない。

ASB 標準Web UI のリポジトリ境界は、ASB 本体と同一リポジトリ内の `webui/` 配下に固定する。

ASB 標準Web UI の入口ファイルは `webui/index.html` とする。

ASB 標準Web UI のスタイルは `webui/styles.css` に配置する。

ASB 標準Web UI の画面制御は `webui/app.js` に配置する。

ASB SDK Browser JavaScript 実装は `webui/asb-sdk.js` として同梱する。

`webui/` は ASB 本体バイナリへ埋め込んではならない。

ASB 本体は `webui/` を起動時に読み込んではならない。

ASB 本体は `webui/` を静的配信対象として自動公開してはならない。

ASB 標準Web UI は、ASB 本体の release artifact とは別 artifact として配布する。

ASB 標準Web UI の配布 artifact は、静的 HTML / CSS / JavaScript と ASB SDK の Browser JavaScript 実装を含むファイル集合とする。

ASB 標準Web UI の配布 artifact は、`node_modules/`、`dist/`、`build/`、`package.json`、`deno.json`、`deno.lock` を含んではならない。

ASB 標準Web UI は build step を持たない。

ASB 標準Web UI の配布 artifact は `webui/` 配下の静的ファイルをそのまま配布可能でなければならない。

ASB 標準Web UI は、初期表示時に `baseUrl` を利用者入力または静的設定値として扱い、永続保存してはならない。

ASB 標準Web UI は、システム管理画面へのアクセスまたは管理操作のたびに、システム管理者パスワード入力を要求する。

ASB 標準Web UI は、入力されたシステム管理者パスワードを ASB SDK の Browser JavaScript 実装へ request 単位で渡す。

ASB 標準Web UI は、システム管理者パスワードをメモリ上で request 完了までの一時値としてのみ扱い、request 完了後に参照を破棄する。

ASB 標準Web UI は、システム管理者パスワードを localStorage、sessionStorage、cookie、IndexedDB、Cache Storage、Service Worker、URL、HTML 属性、ログ、画面表示へ保存または出力してはならない。

ASB 標準Web UI は、`ERR_AUTH_PASSWORD_CHANGE_REQUIRED` を受け取った場合、管理者パスワード変更画面または変更フォームを表示できなければならない。

ASB 標準Web UI を外部ネットワークから利用可能にする場合は、VPN、SSH tunnel、reverse proxy、ファイアウォール、IP制限等のASB外部の運用境界で保護する。

ASB 標準Web UI の画面一覧は以下に固定する。

| 画面 | 目的 | 操作対象 |
|-----|------|----------|
| Dashboard | システム状態、プロジェクト数、ドメイン数、SSL状態、デプロイ状態、ストレージ使用量、直近ログを確認する | Monitoring、Project、Domain、SSL、Deployment、Log |
| Projects | Project の一覧、作成、詳細確認、削除を行う | Project API |
| Domains | Domain の一覧、追加、Project割当、解除、削除を行う | Domain API |
| SSL | 無料独自SSLの有効化、無効化、状態確認、更新を行う | SSL API |
| Files | Project単位のファイル一覧、アップロード、削除を行う | File API |
| Deployments | GitHub Webhookデプロイ結果、重複判定、失敗理由を確認する | Webhook / Deployment 状態 |
| Backups | Backup の作成、一覧、検証、復旧を行う | Backup API |
| Logs | Access log と Error log を確認する | Log API |
| Settings | ASB の読み取り専用設定値、実行時状態、システム管理者パスワード変更を扱う | Config / Monitoring / Auth |

ASB 標準Web UI は、画面ごとに ASB SDK の Browser JavaScript 実装の公開関数のみを呼び出す。

ASB 標準Web UI は、ASB SDK に存在しない操作を UI 操作として提供してはならない。

ASB 標準Web UI は、ASB 管理 HTTPS JSON API が返した `requestId` をエラー表示または詳細表示で確認可能にする。

ASB 標準Web UI は、ユーザー管理 UI、ロール管理 UI、テナント管理 UI、課金 UI、契約管理 UI、FTP / FTPS / SFTP UI を持たない。

ASB 標準Web UI は、ブラウザストレージ、cookie、Service Worker、Cache Storage、IndexedDB を永続状態として使用してはならない。

ASB 標準Web UI は、開発リポジトリ内に `node_modules/`、`dist/`、`build/`、一時ファイル、ログファイル、ビルド成果物を生成してはならない。

ASB 標準Web UI を理由に、ASB 本体へ以下を追加してはならない。

- Web UI 専用 HTTPS API
- Web UI 専用 URL prefix
- Web UI 専用 request body
- Web UI 専用 response body
- Web UI 専用 error format
- Web UI 専用 session
- Web UI 専用 cookie
- Web UI 専用 token
- Web UI 専用 JSON ファイル
- Web UI 専用ディレクトリ
- Web UI 専用実行時データ
- Web UI テンプレート
- Web UI フロントエンドビルド
- Web UI 用外部フレームワーク

外部開発者は、ASB SDK を使用する限り、ASB 標準Web UI をカスタマイズして使用できる。

外部開発者は、ASB SDK を使用する限り、ASB 標準Web UI とは異なる独自Web UIまたは独自フロントエンドを実装できる。

外部開発者の独自Web UIまたは独自フロントエンドでは、外部フロントエンドフレームワーク、bundler、transpiler、package manager、外部ライブラリを採用できる。

外部開発者の独自Web UIまたは独自フロントエンドは、ASB 管理 HTTPS JSON API または ASB SDK の公開仕様に従わなければならない。

外部開発者の独自Web UIまたは独自フロントエンドは、ASB 本体の内部 JSON、Service、Repository、Storage を直接参照または呼び出してはならない。

外部開発者の独自Web UIまたは独自フロントエンドは、Adlaire Group が開発元の公式 ASB、公式 ASB 標準Web UI、公式 ASB SDK、公式仕様の一部として扱わない。

外部開発者は、ASB、ASB 標準Web UI、ASB SDK など、Adlaire Group が開発元の公式プロジェクトに関与できない。

---

### 13.22 無料独自SSL / ACME 固定仕様

Rev.90 時点では、ASB 本体に無料独自SSLを実装する。

無料独自SSLは、XServer Static 互換目標における利用者向け機能名である。

ACME は無料独自SSLを実現する内部実装方式であり、利用者向け機能名として扱わない。

ASB は Let’s Encrypt ACME v2 のみに対応する。

ASB は複数 CA、CA 選定、CA failover、任意 ACME directory URL を実装しない。

domain validation は HTTP-01 challenge のみに限定する。

DNS-01 challenge、TLS-ALPN-01 challenge、wildcard 証明書、DNS provider API 連携、手動 TXT 登録は実装しない。

無料独自SSLは、独自ドメイン単位で有効化する。

無料独自SSLを有効化した Domain は、ASB が証明書取得、証明書保存、証明書更新、更新失敗記録、SSL状態確認を行う。

無料独自SSLの状態は Domain 単位で管理し、以下の値に限定する。

| 状態 | 意味 |
|------|------|
| `disabled` | 無料独自SSLが無効であり、証明書取得対象ではない |
| `pending` | 無料独自SSL有効化要求を受け付け、ACME 処理開始前または処理中である |
| `challenge_ready` | HTTP-01 challenge 応答を公開可能である |
| `issued` | 有効な証明書が保存され、対象 Domain で使用可能である |
| `renewing` | 既存証明書を保持したまま更新処理中である |
| `failed` | 直近の取得または更新が失敗した |
| `expired` | 保存済み証明書の有効期限が切れている |

無料独自SSL有効化 API は、対象 Domain が Project に割り当て済みでない場合 `ERR_DOMAIN_NOT_FOUND` を返す。

無料独自SSL有効化 API は、対象 Domain が `disabled` または `failed` の場合のみ新規取得処理を開始できる。

対象 Domain が `pending`、`challenge_ready`、`issued`、`renewing` の場合、有効化 API は既存状態を返し、重複する ACME order を作成してはならない。

無料独自SSL無効化 API は、対象 Domain の無料独自SSL状態を `disabled` に変更する。

無料独自SSL無効化 API は、既存証明書ファイルを即時削除してはならない。

HTTP-01 challenge 応答は、対象 Domain の `/.well-known/acme-challenge/{token}` で行う。

HTTP-01 challenge 応答は、通常の静的ファイル配信より優先する。

HTTP-01 challenge を成功させるため、対象 Domain の A / AAAA レコードは ASB が応答する公開HTTPサーバーへ到達しなければならない。

HTTP-01 challenge では、外部から port 80 の HTTP リクエストが ASB または ASB 前段のリバースプロキシ経由で ASB の challenge handler へ到達しなければならない。

ASB 前段にリバースプロキシを置く場合、`/.well-known/acme-challenge/` は ASB へ転送しなければならない。

HTTP-01 challenge token は、ACME authorization ごとに生成し、検証完了または失敗後に削除する。

HTTP-01 challenge token は、開発リポジトリ内に生成してはならない。

HTTP-01 challenge token の保存先は `storage.basePath/acme/challenges/` 配下に限定する。

HTTP-01 challenge token ファイルの内容は ACME key authorization 文字列のみとする。

HTTP-01 challenge token は、該当 authorization が `valid`、`invalid`、または `expired` になった時点で削除対象とする。

HTTP-01 challenge token が存在しない場合、challenge handler は通常の静的ファイル探索へ fallback せず `404 Not Found` を返す。

証明書ファイルは `storage.basePath/certs/{domain}/fullchain.pem` と `storage.basePath/certs/{domain}/privkey.pem` に保存する。

`{domain}` は ASB の Domain 正規化ルールで小文字正規化した値を使用する。

証明書ファイル保存先は `storage.basePath` 配下に限定し、開発リポジトリ内に生成してはならない。

ASB は証明書本文と秘密鍵本文の対応確認、有効期限確認を Go 標準ライブラリ `crypto/x509`、`encoding/pem`、`crypto/tls` で行う。

証明書検証に失敗した場合は `ERR_SSL_CERT_GENERATION_FAILED` を返す。

ASB は ACME 内部状態を `storage.basePath` 配下の JSON ファイルとして保存する。

ACME 内部状態の保存先は以下に限定する。

- `config/acme_accounts.json`
- `config/acme_orders.json`
- `config/acme_authorizations.json`
- `config/acme_challenges.json`
- `config/acme_renewals.json`

上記 JSON ファイルは起動時に自動生成してはならない。

上記 JSON ファイルは ASB が初期作成してはならない。

上記 JSON ファイルが存在しない場合、無料独自SSL有効化 API、証明書自動更新、SSL状態確認は不足分を作成せず失敗しなければならない。

無料独自SSL有効化 API は、上記 JSON ファイルを初回作成してはならない。

無料独自SSL有効化 API は、上記 JSON ファイルが存在し、構文、`schemaVersion: 1`、必須トップレベルキー、未知フィールド不在を満たす場合のみ更新できる。

ACME account key は `config/acme_accounts.json` に保存する。

ACME account key は PEM 形式で保存し、保存ファイルの group/world writable を禁止する。

ACME nonce は永続保存しない。

ACME order、authorization、challenge、renewal 履歴は JSON ファイルベースで保存する。

ACME JSON ファイルはすべてトップレベルに `schemaVersion` を持つ。

`config/acme_accounts.json` は以下のキーを持つ。

| キー | 型 | 必須 | 説明 |
|------|----|------|------|
| `schemaVersion` | number | yes | スキーマバージョン |
| `accounts` | array | yes | ACME account 一覧 |

ACME account は以下のキーを持つ。

| キー | 型 | 必須 | 説明 |
|------|----|------|------|
| `id` | string | yes | ASB内部ID |
| `ca` | string | yes | 固定値 `letsencrypt` |
| `directoryUrl` | string | yes | Let’s Encrypt ACME v2 directory URL |
| `accountUrl` | string | yes | ACME account URL |
| `email` | string | no | 登録連絡先 |
| `privateKeyPem` | string | yes | PEM形式の account private key |
| `status` | string | yes | `valid` または `deactivated` |
| `createdAt` | string | yes | UTC RFC3339 |
| `updatedAt` | string | yes | UTC RFC3339 |

`config/acme_orders.json` は以下のキーを持つ。

| キー | 型 | 必須 | 説明 |
|------|----|------|------|
| `schemaVersion` | number | yes | スキーマバージョン |
| `orders` | array | yes | ACME order 一覧 |

ACME order は以下のキーを持つ。

| キー | 型 | 必須 | 説明 |
|------|----|------|------|
| `id` | string | yes | ASB内部ID |
| `domain` | string | yes | 小文字正規化済み Domain |
| `accountId` | string | yes | ACME account ID |
| `orderUrl` | string | yes | ACME order URL |
| `finalizeUrl` | string | yes | finalize URL |
| `certificateUrl` | string | no | certificate URL |
| `status` | string | yes | `pending`、`ready`、`processing`、`valid`、`invalid` のいずれか |
| `expiresAt` | string | no | UTC RFC3339 |
| `createdAt` | string | yes | UTC RFC3339 |
| `updatedAt` | string | yes | UTC RFC3339 |

`config/acme_authorizations.json` は以下のキーを持つ。

| キー | 型 | 必須 | 説明 |
|------|----|------|------|
| `schemaVersion` | number | yes | スキーマバージョン |
| `authorizations` | array | yes | ACME authorization 一覧 |

ACME authorization は以下のキーを持つ。

| キー | 型 | 必須 | 説明 |
|------|----|------|------|
| `id` | string | yes | ASB内部ID |
| `orderId` | string | yes | ACME order ID |
| `domain` | string | yes | 小文字正規化済み Domain |
| `authorizationUrl` | string | yes | ACME authorization URL |
| `status` | string | yes | `pending`、`valid`、`invalid`、`expired`、`deactivated`、`revoked` のいずれか |
| `expiresAt` | string | no | UTC RFC3339 |
| `createdAt` | string | yes | UTC RFC3339 |
| `updatedAt` | string | yes | UTC RFC3339 |

`config/acme_challenges.json` は以下のキーを持つ。

| キー | 型 | 必須 | 説明 |
|------|----|------|------|
| `schemaVersion` | number | yes | スキーマバージョン |
| `challenges` | array | yes | ACME challenge 一覧 |

ACME challenge は以下のキーを持つ。

| キー | 型 | 必須 | 説明 |
|------|----|------|------|
| `id` | string | yes | ASB内部ID |
| `authorizationId` | string | yes | ACME authorization ID |
| `domain` | string | yes | 小文字正規化済み Domain |
| `type` | string | yes | 固定値 `http-01` |
| `challengeUrl` | string | yes | ACME challenge URL |
| `token` | string | yes | HTTP-01 token |
| `keyAuthorizationPath` | string | yes | `storage.basePath/acme/challenges/{domain}/{token}` |
| `status` | string | yes | `pending`、`processing`、`valid`、`invalid` のいずれか |
| `createdAt` | string | yes | UTC RFC3339 |
| `updatedAt` | string | yes | UTC RFC3339 |

`config/acme_renewals.json` は以下のキーを持つ。

| キー | 型 | 必須 | 説明 |
|------|----|------|------|
| `schemaVersion` | number | yes | スキーマバージョン |
| `renewals` | array | yes | 証明書取得・更新履歴 |

ACME renewal は以下のキーを持つ。

| キー | 型 | 必須 | 説明 |
|------|----|------|------|
| `id` | string | yes | ASB内部ID |
| `domain` | string | yes | 小文字正規化済み Domain |
| `kind` | string | yes | `issue` または `renew` |
| `status` | string | yes | `queued`、`running`、`succeeded`、`failed`、`backoff` のいずれか |
| `attemptCount` | number | yes | 試行回数 |
| `lastErrorCode` | string | no | 直近の ASB エラーコード |
| `lastErrorMessage` | string | no | 直近の失敗理由 |
| `nextRetryAt` | string | no | UTC RFC3339 |
| `notBefore` | string | no | 証明書有効開始日時 |
| `notAfter` | string | no | 証明書有効終了日時 |
| `createdAt` | string | yes | UTC RFC3339 |
| `updatedAt` | string | yes | UTC RFC3339 |

無料独自SSLの新規取得状態遷移は以下に固定する。

| 現在状態 | イベント | 次状態 |
|----------|----------|--------|
| `disabled` | 有効化API受理 | `pending` |
| `failed` | 有効化API受理 | `pending` |
| `pending` | HTTP-01 token 配置完了 | `challenge_ready` |
| `challenge_ready` | authorization valid | `pending` |
| `pending` | certificate 保存・検証成功 | `issued` |
| `pending` | 取得失敗 | `failed` |
| `challenge_ready` | challenge 失敗 | `failed` |

証明書更新状態遷移は以下に固定する。

| 現在状態 | イベント | 次状態 |
|----------|----------|--------|
| `issued` | 更新対象判定 | `renewing` |
| `renewing` | HTTP-01 token 配置完了 | `challenge_ready` |
| `challenge_ready` | authorization valid | `renewing` |
| `renewing` | 新証明書保存・検証成功 | `issued` |
| `renewing` | 更新失敗かつ既存証明書有効 | `issued` |
| `renewing` | 更新失敗かつ既存証明書期限切れ | `expired` |
| `issued` | 証明書期限切れ検出 | `expired` |

証明書自動更新は、証明書有効期限の `ssl.renewBefore` 秒前から対象とする。

`ssl.renewBefore` は `86400` 以上 `15552000` 以下の秒数整数のみ許可する。

証明書自動更新は起動時チェックと定期チェックで実行する。

定期チェック間隔は `ssl.renewCheckInterval` で指定する。

`ssl.renewCheckInterval` は `3600` 以上 `86400` 以下の秒数整数のみ許可する。

更新失敗時は retry / backoff を行い、失敗履歴を `config/acme_renewals.json` に保存する。

retry / backoff は Domain 単位で管理する。

retry / backoff は `attemptCount` に基づき、最小 `3600` 秒、最大 `86400` 秒の範囲で次回再試行時刻 `nextRetryAt` を決定する。

`nextRetryAt` より前に同一 Domain の自動更新を再実行してはならない。

更新失敗は既存有効証明書を削除してはならない。

新証明書の保存、証明書検証、秘密鍵権限検証がすべて成功した場合のみ、対象 Domain の SSL 状態を更新する。

Let’s Encrypt rate limit に到達した場合は、`ERR_SSL_CERT_GENERATION_FAILED` を返し、次回再試行可能時刻を renewal 履歴に保存する。

Rev.90 時点で ASB 本体は以下を実装してはならない。

- 複数 CA
- CA 選定
- CA failover
- 任意 ACME directory URL
- DNS-01 challenge
- TLS-ALPN-01 challenge
- wildcard 証明書
- DNS provider API 連携
- 手動 TXT 登録
- EAB
- ARI
- OCSP stapling

---

### 13.23 ASB互換目標固定仕様

Rev.90 時点では、ASB互換目標は将来の到達目標であり、個別の確定仕様へ昇格した項目のみ実装対象とする。

Rev.90 時点では、無料独自SSLのみを XServer Static 互換目標から確定仕様へ昇格する。

ASB互換目標は、XServer Static 等の静的コンテンツ専用ホスティングの利用体験を参考にした ASB 独自の目標である。

ASB互換目標は、外部サービスとの完全互換、API互換、管理画面互換、内部実装互換を意味しない。

ASB互換目標で参照する外部サービスの仕様、挙動、画面、名称、用語、API、設定、保存形式、証明書運用は、`ASB-spec.md` に明記されない限り ASB の確定仕様ではない。

ASB互換目標における「参考」「相当」「目標」は、仕様確定済み事項を識別する語ではない。

ASB互換目標は、ASB の実装を外部サービスへ合わせる指示ではなく、ASB 独自仕様として将来比較可能な利用体験を整理するための境界である。

Rev.90 時点で ASB互換目標に含める対象は以下とする。

- 静的コンテンツ専用ホスティング
- HTML、CSS、JavaScript、画像等の静的ファイル配信
- プロジェクト単位の公開対象管理
- 独自ドメイン割り当て
- 無料独自SSL
- 無料独自SSLの証明書自動取得
- 無料独自SSLの証明書自動更新
- HTTP/2
- GitHub 連携による自動デプロイ
- ファイルアップロード
- フォルダ階層を保持したファイル管理
- SSL更新状態、デプロイ状態、ログの確認

Rev.90 時点で XServer Static互換機能セットとして実装対象に固定する機能は以下とする。

| 機能 | 実装境界 |
|-----|----------|
| 静的コンテンツ配信 | HTML、CSS、JavaScript、画像等の静的ファイル配信、`index.html`、404応答、MIME type、Gzip。 |
| 独自ドメイン | Project へのDomain割り当て、サブドメインの個別Domain管理、Host header によるProject解決。 |
| 無料独自SSL | Let’s Encrypt ACME v2、HTTP-01、証明書自動取得、証明書自動更新、SSL状態確認。 |
| GitHub 自動デプロイ | GitHub Push Webhook、指定branch、ローカルcheckout、冪等キー、デプロイ状態記録。 |
| ファイル管理 | HTTPS JSON API、`multipart/form-data` upload、フォルダ階層保持、上書き、削除、容量制限。 |
| ログ・状態確認 | Access log、Error log、Monitoring API、SSL状態、Webhook処理状態、ストレージ使用量。 |
| バックアップ・復旧 | JSONファイルベースのバックアップ作成、検証、復旧。 |

Rev.90 時点で ASB互換目標に含めない対象は以下とする。

- XServer Static との完全互換
- XServer Static の管理画面再現
- XServer Static の内部実装再現
- 外部サービスのAPI完全互換
- 外部サービスのDNS管理機能
- 外部サービスのCDN機能完全互換
- 外部サービスの課金、契約、アカウント管理
- 外部サービスの SLA / サポート体制
- DNS-01 challenge
- TLS-ALPN-01 challenge
- wildcard 証明書
- 複数 CA
- CA 選定
- CA failover
- 任意 ACME directory URL
- DNS provider API 連携
- 手動 TXT 登録

ASB互換目標に含まれる機能であっても、以下は Rev.90 時点では実装対象ではない。

- HTTP/2 実装詳細
- SDK 外部配布 / npm 配布
- SDK 認証拡張

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

Rev.90 時点では、将来計画、保留事項、検討・調査中事項は実装対象ではない。

本節は、将来計画に含まれる機能を実装対象外として固定する。

ASB 本体は Rev.90 時点では以下を実装してはならない。

- GUI という曖昧カテゴリ
- ASB 本体への Web UI 内包
- デスクトップアプリ
- モバイルアプリ
- 複数ユーザー管理
- ユーザー別権限管理
- マルチテナント
- 課金管理
- 契約管理
- 複数インスタンス管理
- クラスタ管理
- 分散ロック
- NFS 専用連携
- 分散ストレージ専用連携
- 外部ストレージサービス連携
- ログファイル暗号化
- HTTP/2 実装詳細
- FTP
- FTPS
- SFTP

将来計画機能を理由に、ASB 本体は Rev.90 時点では以下を追加、変更、生成してはならない。

- ASB 本体内包 Web UI 用 API
- モバイル専用 API
- テナント用 API
- 課金用 API
- 契約用 API
- 外部ストレージ用 API
- ログ暗号化用 API
- FTP / FTPS / SFTP 用 API
- `ui.*` 設定項目
- `webui.*` 設定項目
- `desktop.*` 設定項目
- `mobile.*` 設定項目
- `tenant.*` 設定項目
- `billing.*` 設定項目
- `nfs.*` 設定項目
- `cluster.*` 設定項目
- `distributedStorage.*` 設定項目
- `externalStorage.*` 設定項目
- `logEncryption.*` 設定項目
- `ftp.*` 設定項目
- `ftps.*` 設定項目
- `sftp.*` 設定項目
- ASB 本体内包 Web UI 用 JSON ファイル
- モバイル用 JSON ファイル
- テナント用 JSON ファイル
- 課金用 JSON ファイル
- 外部ストレージ用 JSON ファイル
- ログ暗号化用 JSON ファイル
- FTP / FTPS / SFTP 用 JSON ファイル
- ASB 本体内包 Web UI 用ディレクトリ
- モバイル用ディレクトリ
- テナント用ディレクトリ
- 課金用ディレクトリ
- 外部ストレージ用ディレクトリ
- ログ暗号化用ディレクトリ
- FTP / FTPS / SFTP 用ディレクトリ
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

#### 13.24.1 複数インスタンス対応固定仕様

複数インスタンス対応とは、複数の ASB process または複数 node が同一の設定、Project、Domain、File、Backup、Log、SSL状態を共有し、同時に管理 API、静的配信、Webhook、Backup、SSL更新を処理する構成を指す。

Rev.90 時点では、ASB 本体に複数インスタンス対応を実装しない。

ASB 本体は Rev.90 時点では以下を実装してはならない。

- node 管理
- leader election
- distributed lock
- cluster membership
- node heartbeat
- node discovery
- cross-node cache invalidation
- cross-node job coordination
- shared deployment queue
- shared ACME renewal queue
- shared backup queue
- active-active 構成制御
- active-standby 構成制御

複数インスタンス対応のために以下の JSON ファイル、ディレクトリ、設定項目を作成してはならない。

- `config/cluster.json`
- `config/nodes.json`
- `config/locks.json`
- `storage/cluster/`
- `storage/nodes/`
- `storage/locks/`
- `cluster.*`
- `node.*`
- `lock.*`

開発リポジトリ内にも、`storage.basePath` 配下にも、node 状態、leader 状態、lock 状態、heartbeat、queue ownership を表す実行時データを生成してはならない。

複数インスタンス対応を将来実装する場合は、実装前に `ASB-spec.md` を改訂し、node 識別子、共有ストレージ、lock 方式、障害時復旧、同時書き込み整合性、Webhook重複処理、ACME更新競合、Backup競合、ログ集約、設定形式、migration、downgrade、テスト条件を最低限確定する。

#### 13.24.2 NFS 連携固定仕様

NFS 連携とは、ASB が Network File System を専用の保存基盤または複数インスタンス用共有ストレージとして認識し、NFS 固有の lock、mount、権限、障害、性能、整合性を扱う機能を指す。

Rev.90 時点では、ASB 本体に NFS 連携を実装しない。

ASB 本体は Rev.90 時点では以下を実装してはならない。

- NFS mount 管理
- NFS lock 制御
- NFS stale handle 検出
- NFS 専用 retry
- NFS 専用 timeout
- NFS 専用 health check
- NFS 専用整合性補正
- NFS 専用エラーコード
- NFS 専用ログ項目

NFS 連携のために以下の JSON ファイル、ディレクトリ、設定項目を作成してはならない。

- `config/nfs.json`
- `storage/nfs/`
- `nfs.*`
- `storage.nfs.*`

開発リポジトリ内にも、`storage.basePath` 配下にも、NFS mount 状態、NFS lock 状態、NFS health check 結果、NFS retry 状態を表す実行時データを生成してはならない。

NFS 連携を将来実装する場合は、実装前に `ASB-spec.md` を改訂し、対応 NFS version、mount 前提、lock 方式、atomic rename 前提、権限、障害時挙動、性能前提、複数インスタンス対応との関係、設定形式、migration、downgrade、テスト条件を最低限確定する。

#### 13.24.3 分散ストレージ連携固定仕様

分散ストレージ連携とは、ASB が複数 node または分散ファイルシステムを保存基盤として扱い、静的ファイル、JSON、Backup、Log、SSL証明書の配置、複製、整合性、障害時復旧を制御する機能を指す。

Rev.90 時点では、ASB 本体に分散ストレージ連携を実装しない。

ASB 本体は Rev.90 時点では以下を実装してはならない。

- distributed storage driver
- replica 管理
- shard 管理
- quorum 制御
- consistency level 制御
- object version conflict 解決
- storage node health check
- replication job
- repair job
- rebalancing
- 分散ストレージ専用 API

分散ストレージ連携のために以下の JSON ファイル、ディレクトリ、設定項目を作成してはならない。

- `config/distributed_storage.json`
- `storage/distributed/`
- `storage/replicas/`
- `storage/shards/`
- `distributedStorage.*`
- `replica.*`
- `shard.*`

開発リポジトリ内にも、`storage.basePath` 配下にも、replica 状態、shard 状態、quorum 状態、repair 状態、rebalancing 状態を表す実行時データを生成してはならない。

分散ストレージ連携を将来実装する場合は、実装前に `ASB-spec.md` を改訂し、保存対象、整合性モデル、書き込み順序、読み取り優先順位、障害時復旧、データ修復、複数インスタンス対応との関係、設定形式、migration、downgrade、テスト条件を最低限確定する。

#### 13.24.4 外部ストレージ連携固定仕様

外部ストレージ連携とは、ASB が S3 互換ストレージ、クラウドストレージ、外部オブジェクトストレージ、外部バックアップサービス等を保存先、バックアップ先、配信元、または復旧元として扱う機能を指す。

Rev.90 時点では、ASB 本体に外部ストレージ連携を実装しない。

ASB 本体は Rev.90 時点では以下を実装してはならない。

- S3 API 連携
- S3 互換 API 連携
- Google Cloud Storage 連携
- Azure Blob Storage 連携
- Dropbox / Google Drive / Box 等のファイルサービス連携
- 外部バックアップ送信
- 外部バックアップ復元
- presigned URL 発行
- 外部ストレージ credential 管理
- 外部ストレージ SDK
- 外部ストレージ専用 API

外部ストレージ連携のために以下の JSON ファイル、ディレクトリ、設定項目を作成してはならない。

- `config/external_storage.json`
- `config/storage_providers.json`
- `storage/external/`
- `storage/providers/`
- `externalStorage.*`
- `storageProvider.*`
- `s3.*`
- `gcs.*`
- `azureBlob.*`

開発リポジトリ内にも、`storage.basePath` 配下にも、外部ストレージ credential、provider 状態、sync 状態、upload queue、download queue、presigned URL 状態を表す実行時データを生成してはならない。

外部ストレージ連携を将来実装する場合は、実装前に `ASB-spec.md` を改訂し、対象 provider、外部SDK採否、credential 保存方式、暗号化要否、保存対象、同期方式、整合性、失敗時再試行、Backupとの関係、設定形式、migration、downgrade、テスト条件を最低限確定する。

#### 13.24.5 ログファイル暗号化固定仕様

ログファイル暗号化とは、ASB が access log、error log、audit log、運用ログ等を保存時に暗号化し、復号、鍵管理、鍵ローテーション、復旧、閲覧制御を扱う機能を指す。

Rev.90 時点では、ASB 本体にログファイル暗号化を実装しない。

ASB 本体は Rev.90 時点では以下を実装してはならない。

- ログ保存時暗号化
- ログ復号 API
- ログ暗号化 key 生成
- ログ暗号化 key 保存
- ログ暗号化 key rotation
- KMS 連携
- 暗号化ログ viewer
- 暗号化ログ migration
- 暗号化ログ専用 error code

ログファイル暗号化のために以下の JSON ファイル、ディレクトリ、設定項目を作成してはならない。

- `config/log_encryption.json`
- `storage/log-keys/`
- `storage/encrypted-logs/`
- `logEncryption.*`
- `kms.*`

開発リポジトリ内にも、`storage.basePath` 配下にも、ログ暗号化 key、復号状態、key rotation 状態、暗号化ログ migration 状態を表す実行時データを生成してはならない。

ログファイル暗号化を将来実装する場合は、実装前に `ASB-spec.md` を改訂し、暗号方式、key 保存方式、key rotation、復号API、閲覧権限、既存ログ移行、Backupとの関係、外部KMS採否、設定形式、migration、downgrade、テスト条件を最低限確定する。

#### 13.24.6 デスクトップアプリ向け SDK 利用固定仕様

デスクトップアプリ向け SDK 利用とは、Windows、macOS、Linux 等のデスクトップアプリが ASB SDK を用いて ASB 管理 HTTPS JSON API と通信する構成を指す。

Rev.90 時点では、デスクトップアプリ向け SDK 利用を実装対象に含めない。

ASB 本体、ASB SDK、ASB 標準Web UI は Rev.90 時点では以下を実装してはならない。

- デスクトップアプリ専用 API
- デスクトップアプリ専用認証
- デスクトップアプリ専用 token
- デスクトップアプリ専用 callback URL
- deep link
- OS keychain 連携
- auto update
- installer 生成
- desktop notification
- tray integration
- native menu
- GUIライブラリ依存

デスクトップアプリ向け SDK 利用のために以下の JSON ファイル、ディレクトリ、設定項目を作成してはならない。

- `config/desktop.json`
- `storage/desktop/`
- `desktop.*`
- `desktopSdk.*`

開発リポジトリ内にも、`storage.basePath` 配下にも、デスクトップアプリ用 token、session、device registration、callback 状態、update 状態を表す実行時データを生成してはならない。

デスクトップアプリ向け SDK 利用を将来実装対象へ昇格する場合は、実装前に `ASB-spec.md` を改訂し、対象OS、配布方式、署名、更新方式、GUIライブラリ採否、SDK実装、認証方式、保存データ、ASB本体との責務分界、外部依存、migration、downgrade、テスト条件を最低限確定する。

#### 13.24.7 モバイルアプリ向け SDK 利用固定仕様

モバイルアプリ向け SDK 利用とは、iOS または Android のモバイルアプリが ASB SDK を用いて ASB 管理 HTTPS JSON API と通信する構成を指す。

Rev.90 時点では、モバイルアプリ向け SDK 利用を実装対象に含めない。

ASB 本体、ASB SDK、ASB 標準Web UI は Rev.90 時点では以下を実装してはならない。

- モバイルアプリ専用 API
- モバイルアプリ専用認証
- モバイルアプリ専用 token
- device registration
- push notification
- biometric authentication
- mobile deep link
- app link / universal link
- mobile offline cache
- mobile sync queue
- app store 配布設定
- モバイル GUIライブラリ依存

モバイルアプリ向け SDK 利用のために以下の JSON ファイル、ディレクトリ、設定項目を作成してはならない。

- `config/mobile.json`
- `storage/mobile/`
- `mobile.*`
- `mobileSdk.*`

開発リポジトリ内にも、`storage.basePath` 配下にも、モバイルアプリ用 token、session、device registration、push notification 状態、offline cache、sync queue を表す実行時データを生成してはならない。

モバイルアプリ向け SDK 利用を将来実装対象へ昇格する場合は、実装前に `ASB-spec.md` を改訂し、対象OS、配布方式、署名、更新方式、GUIライブラリ採否、SDK実装、認証方式、通知、offline cache 採否、保存データ、ASB本体との責務分界、外部依存、migration、downgrade、テスト条件を最低限確定する。

---

### 13.25 外部・旧プロジェクト吸収固定仕様

外部プロジェクトまたは旧プロジェクトの仕様を ASB に吸収する場合は、旧プロジェクト名の別枠を作らない。

外部プロジェクトまたは旧プロジェクトの仕様を、既存の ASB 機能へ無理に混ぜてはならない。

吸収対象の仕様は、ASB の通常機能名へ分解し、Project、File、Domain、SSL、GitHub Webhook、Backup、Log と同じ粒度の ASB 機能として横並びに扱う。

吸収後の機能名は、ASB の正式機能名として扱う。

旧プロジェクト名は、移管元説明、変更履歴、法務・権利・出典確認に必要な記録に限り使用できる。

旧プロジェクト名を、ASB の有効な機能名、製品名、サブシステム名、API名、設定名、JSON名、ディレクトリ名、package 名、SDK名、Web UI名、CLI subcommand 名として使用してはならない。

旧プロジェクト由来仕様を ASB の通常機能へ分解する場合は、以下を満たさなければならない。

- ASB の既存機能へ意味なく統合しない。
- ASB の既存機能と責務が重なる場合は、責務境界を明記する。
- 既存機能へ統合する場合は、既存機能の責務、API、設定、保存JSON、生成物、副作用を変更する理由を明記する。
- 新しい通常機能として扱う場合は、Project や Domain と同じ粒度で機能名を定義する。
- 上位別枠、互換モード、旧名称 module、旧名称 runtime、旧名称 CLI を作らない。
- 実装対象へ昇格するまでは、API、設定項目、保存JSON、生成ファイル、生成ディレクトリ、外部依存、実行時データを追加しない。

#### 13.25.1 Auteur 仕様移管元

Auteur は ASB に仕様のみを吸収する。

Rev.90 時点では、Auteur は独立製品、独立プロジェクト、ASB内の独立サブシステム、外部依存、互換対象、migration 元、別ブランド、有効な ASB 機能名として扱わない。

ASB の正式名称は `Adlaire-Static-Base`、正式略称は `ASB` とする。

Auteur 仕様の移管元は、以下に固定する。

| 項目 | 内容 |
|-----|------|
| 移管元リポジトリ | `https://github.com/fqwink/Auteur` |
| 移管元仕様ファイル | `Auteur_Master_Specification.md` |
| 移管方式 | 仕様のみを ASB 仕様へ吸収する |
| ASB 側正本 | `ASB-spec.md` |

Auteur 仕様は、ASB へ吸収するための一時的な仕様移管元としてのみ扱う。

Auteur 仕様を移管した後の ASB における仕様判断は、`ASB-spec.md` を正とする。

Auteur リポジトリの source code、runtime、CLI、fixture、test、CI、release automation、package、lock file、設定ファイル、生成物は ASB へ移管しない。

Auteur リポジトリ内の `.gitignore`、`deno.json`、TypeScript 実装、fixture、test は、ASB の仕様、実装、生成物、依存関係、開発手順としてコピーしてはならない。

#### 13.25.2 移管元由来仕様の ASB 通常機能分類

移管元由来仕様は、ASB では以下の通常機能名へ分解して扱う。

| 移管元概念 | ASB 通常機能名 | Rev.90 時点の状態 |
|----------------|-------------|------------------|
| Git source of truth | GitHub Webhook / Source Sync | GitHub Webhook は実装対象、Source Sync は将来計画 |
| Markdown / MDX / JSON content | Content Pipeline | 将来計画 |
| JSON Front Matter | Content Pipeline | 将来計画 |
| Content Collections | Content Pipeline | 将来計画 |
| File-based Routing | Site Routing | 将来計画 |
| SSG | Site Rendering | 将来計画 |
| SSR | Site Rendering | 将来計画 |
| Hybrid Rendering | Site Rendering | 将来計画 |
| Islands Architecture | Site Rendering | 将来計画 |
| Request / Response API route | Site Routing | 将来計画 |
| External Content Loader | External Data Integration | 将来計画 |
| Runtime Cache | Runtime Cache | 将来計画 |
| Asset Pipeline | Asset Pipeline | 将来計画 |
| Infrastructure Adapter | 各 ASB 通常機能の実装境界 | 実装境界は確定、汎用 adapter は将来計画 |
| Blog output | Blog | 将来計画 |
| Documentation / Knowledge Base output | Docs | 将来計画 |
| Sitemap | Sitemap | 将来計画 |
| SEO metadata | Site Output | 将来計画 |
| i18n route | Site Output | 将来計画 |
| Ad Slot Rendering | Ad Slot | 将来計画 |
| Database Gateway / Adapter | 外部DB不使用方針により現行対象外 | 将来計画にも未昇格 |
| Database Content Loader | 外部DB不使用方針により現行対象外 | 将来計画にも未昇格 |
| Database Schema Migration | 外部DB不使用方針により現行対象外 | 対象外 |
| Persistent State Store | JSON ファイルベース保存方針 | 外部DB版は対象外 |

#### 13.25.3 ASB へ吸収する技術方針

移管元由来の技術方針のうち、ASB に吸収する方針は以下とする。

- content-first の考え方は、ASB の Content Pipeline、Site Routing、Site Rendering、Site Output、Blog、Docs、Sitemap、Ad Slot の仕様候補へ分解して扱う。
- コンテンツ所有権が利用者側にある考え方は、ASB のセルフホスト、JSON ファイルベース、GitHub Webhook デプロイ、Backup / Restore の仕様へ吸収する。
- 外部 UI framework を採用しない方針は、ASB 標準Web UI の仕様へ吸収する。
- Node.js / npm を採用しない方針は、ASB 本体、ASB 標準Web UI、ASB SDK Browser JavaScript 実装、ASB SDK Deno専用 TypeScript 実装の禁止事項へ吸収する。
- Managed Edge Runtime 非対応方針は、ASB のセルフホスト、VPS / オンプレミス、Linux 単一バイナリ運用の方針へ吸収する。
- Gateway / Adapter / Loader 境界の考え方は、ASB の各通常機能における Handler / Service / Repository / Storage / 外部依存境界へ吸収する。
- DB を標準必須要件にしない方針は、ASB の外部DB不使用、JSON ファイルベース保存方針へ吸収する。
- Realtime Application、Client-side Heavy SPA、Built-in Product Features、Feed Generation、ISR、Edge Runtime Specific Feature、External UI Framework Integration を ASB 本体へ内蔵しない。

#### 13.25.4 ASB 通常機能への分解ルール

移管元由来仕様を ASB へ移す場合は、以下の ASB 通常機能名へ分解する。

| 移管元表現 | ASB 通常機能名 |
|-----------|-----------------|
| Core Product Spec | ASB 仕様 |
| Implementation Spec | ASB 実装タスクまたは非移管対象 |
| Project | ASB Project |
| Content | Content Pipeline |
| Git Sync | GitHub Webhook または Source Sync |
| HTTP Runtime | ASB HTTPS Server または Site Rendering |
| Public API | ASB 管理 HTTPS JSON API または Site Routing |
| Component Runtime | Site Rendering |
| Islands | Site Rendering |
| Asset Pipeline | Asset Pipeline |
| Site Output | Site Output |
| Blog | Blog |
| Documentation / Knowledge Base | Docs |
| Sitemap | Sitemap |
| Ad Slot Rendering | Ad Slot |

ASB の外部公開名、API名、設定名、JSON名、ディレクトリ名、package 名、SDK名、Web UI名、CLI subcommand 名では、上記の ASB 通常機能名のみを使用する。

#### 13.25.5 現行実装対象へ昇格しない移管元由来仕様

Rev.90 時点では、以下の移管元由来仕様を ASB の現行実装対象へ昇格しない。

- Markdown / MDX rendering
- JSON Front Matter parsing
- Content Collections
- Content schema validation
- File-based Routing
- SSG
- SSR
- Hybrid Rendering
- Islands Architecture
- Component Runtime
- Route Manifest
- Build Manifest
- Site generator
- Blog generator
- Docs generator
- Sitemap generator
- SEO metadata generator
- i18n route generator
- Ad Slot
- External Content Loader
- Analytics integration
- Runtime Cache
- Generic Source Sync
- Database Gateway
- Database Adapter
- Database Content Loader
- Database Schema Migration
- Content Metadata Index
- Persistent State Store backed by external DB

上記を実装対象へ昇格する場合は、実装前に `ASB-spec.md` を改訂し、ASB における正式機能名、API、設定項目、保存JSON、生成ファイル、生成ディレクトリ、外部依存、実行時データ配置、開発リポジトリ非生成、migration、downgrade、テスト条件を確定しなければならない。

#### 13.25.6 ASB へ移管しない旧名称・構造

移管元仕様に存在する以下の名称、構造、ファイル、コマンドは、ASB の有効仕様としてコピーしない。

- `auteur.config.json`
- `.auteur/`
- `auteur-project/`
- `src/pages/**/*.astro`
- `src/pages/api/**/*.go`
- `ui/`
- `content/`
- `dist/`
- `dist/static/`
- `dist/server/`
- `dist/manifest.json`
- `.env`
- `deno.json`
- `deno.lock`
- `node_modules/`
- `<auteur-cli>`
- `create`
- `check`
- `build`
- `preview`
- `dev`
- `sync`
- `AUTEUR_*` error code
- Auteur 固有 hydration directive
- Auteur 固有 component syntax

ASB が将来、同種の機能を採用する場合でも、ASB 名称、ASB 設定、ASB JSON、ASB ディレクトリ、ASB CLI subcommand として再設計する。

#### 13.25.7 ASB 通常機能として扱う将来仕様候補

移管元由来のコンテンツ駆動サイト生成機能は、ASB の将来仕様候補として以下の通常機能名に整理する。

| 候補機能 | 説明 | 現行状態 |
|---------|------|---------|
| Content Pipeline | Markdown、MDX、JSON、JSON Front Matter、Content Collections、schema validation、HTML rendering を扱う | 将来計画 |
| Site Routing | ファイルベース route 検出、route path、static output path、404 を扱う | 将来計画 |
| Site Rendering | SSG、SSR、Hybrid Rendering、prerender 指定を扱う | 将来計画 |
| Site Output | Blog、Docs、Sitemap、SEO metadata、i18n route、Ad Slot を扱う | 将来計画 |
| Blog | 記事管理、記事一覧、pagination、draft 管理を扱う | 将来計画 |
| Docs | ドキュメントサイト、階層、前後リンク、sidebar metadata を扱う | 将来計画 |
| Sitemap | `sitemap.xml`、route 出力、hreflang、lastmod を扱う | 将来計画 |
| Ad Slot | 広告枠定義、表示条件、外部広告SDK非内蔵を扱う | 将来計画 |
| Asset Pipeline | image / static asset metadata、copy、加工を扱う | 将来計画 |
| Source Sync | Git provider から content を同期する | 将来計画 |
| External Data Integration | 外部 API、Headless CMS、analytics を content / event 境界へ正規化する | 将来計画 |
| Runtime Cache | 再生成可能な content、loader、asset、route cache を扱う | 将来計画 |

これらは Rev.90 時点では実装対象外であり、既存の Project、File、Domain、SSL、Webhook、Backup、Log、ASB SDK、ASB 標準Web UI の実装を変更する根拠にならない。

#### 13.25.8 Blog / Ad Slot 仕様の扱い

Blog は ASB の通常機能名として扱う。

Rev.90 時点では、Blog は将来計画であり、実装対象ではない。

Blog 仕様を将来実装対象へ昇格する場合は、以下を最低限確定する。

- ASB における記事保存形式
- JSON Front Matter の採否
- Markdown / MDX parser の採否
- slug、author、date、updated、tags、category、excerpt、draft、lang の型と検証
- `/blog`、`/blog/:slug`、`/blog/page/:page` の route 仕様
- draft の公開可否
- sitemap、OG metadata、Twitter Card metadata の扱い
- comment system、search engine、RSS、Atom、newsletter、external SNS SDK の非対応範囲

Ad Slot は ASB の通常機能名として扱う。

Rev.90 時点では、Ad Slot は将来計画であり、実装対象ではない。

Ad Slot を将来実装対象へ昇格する場合は、以下を最低限確定する。

- 広告枠定義の保存形式
- slot ID、size、position、priority、language、route、device、period の型と検証
- static build time injection、request time injection、client side injection の採否
- 外部広告ネットワーク SDK、ユーザー単位行動追跡、realtime bidding、個人情報ターゲティングの非対応範囲
- analytics 連携を採用する場合の Adapter 境界

#### 13.25.9 移管元由来仕様の実装前固定契約

本節は、移管元由来仕様を ASB の正式機能へ昇格する前に確定しなければならない実装契約である。

Rev.90 時点では、本節の項目は実装対象ではない。

移管元由来仕様を ASB へ実装する場合は、仕様改訂時に以下の全項目を機能ごとに固定する。

| 契約項目 | 固定内容 |
|---------|---------|
| 正式機能名 | ASB 名称のみを使用し、Auteur 名を使用しない |
| 責務範囲 | ASB 本体、ASB SDK、ASB 標準Web UI、利用者プロジェクトのどこが責務を持つか |
| 入力 | file、directory、request、JSON、metadata、environment variable、command argument の採否 |
| 出力 | 公開ファイル、manifest、metadata、JSON、log、HTTP response、artifact の採否 |
| 保存形式 | `storage.basePath` 配下へ保存する JSON schema、ソート順、未知フィールド拒否、atomic save 条件 |
| 生成物 | 生成ファイル、生成ディレクトリ、生成タイミング、削除タイミング、再生成可能性 |
| API | 管理 HTTPS JSON API、公開サイト API route、静的配信 route の区別 |
| 設定項目 | 起動設定ファイル `config.json` へ追加する key、型、default、必須性、未知フィールド拒否 |
| CLI | ASB 本体 CLI subcommand の有無、引数、stdout、stderr、終了コード |
| 外部依存 | Go標準ライブラリ、内製実装、例外外部ライブラリのどれを採用するか |
| セキュリティ | path traversal、unsafe URL、HTML injection、script injection、secret leak の拒否条件 |
| 実行時データ | 開発リポジトリ内へ生成しない保証、`storage.basePath` 配下での配置 |
| migration | 既存 JSON schema 変更要否、dry-run、apply、rollback、downgrade |
| テスト | unit、integration、E2E、非生成確認、禁止依存確認、失敗系確認 |

実装対象へ昇格する機能は、上記の契約項目が未確定の状態で API、設定項目、保存JSON、ディレクトリ、package、CLI、SDK public API、Web UI 操作を追加してはならない。

#### 13.25.10 Content Pipeline 昇格時仕様契約

Content Pipeline を将来実装対象へ昇格する場合は、以下を固定する。

- 入力 file type は `.md`、`.mdx`、`.json` を候補とする。
- `.md` は Markdown document として扱う。
- `.mdx` は MDX 採用可否を別途固定するまで実装してはならない。
- `.json` は content data として扱い、top-level object のみ許可する。
- Front Matter は JSON Front Matter のみを候補とし、YAML Front Matter と TOML Front Matter は採用しない。
- JSON Front Matter を採用する場合は、Markdown 本文先頭の JSON object block と本文の区切り文字、parse error、未知 field、空 Front Matter の扱いを固定する。
- Markdown parser、MDX parser、HTML sanitizer は Go 標準ライブラリに存在しないため、内製実装範囲または例外外部ライブラリ採用理由を仕様へ明記するまで導入してはならない。
- content metadata は `title`、`slug`、`date`、`updated`、`tags`、`category`、`draft`、`lang`、`excerpt` を候補とする。
- `title` は 1〜255 bytes の UTF-8 string、`slug` は URL path segment として安全な ASCII string、`date` と `updated` は UTC RFC3339、`tags` は string array、`draft` は boolean、`lang` は BCP 47 形式候補として扱う。
- metadata の default、必須性、正規化、拒否条件は項目ごとに固定する。
- slug 重複は同一 Project、同一 output namespace、同一 locale 内で error とする。
- draft は公開 output へ含めないことを標準案とし、preview へ含める場合は preview 境界を別途固定する。
- 未来日付 content は公開可否を固定するまで公開 output へ含めてはならない。
- unsafe HTML、`script` tag、event handler attribute、`javascript:` URL、`data:` URL は拒否を標準案とする。
- link URL scheme は `http`、`https`、relative path、fragment を許可候補とし、それ以外は拒否候補とする。
- content source directory、schema file、output directory、cache directory は ASB 名称で定義し、Auteur 名を使用しない。
- Content Pipeline の処理結果を保存する場合は、保存JSON名、schemaVersion、atomic save、再生成可否、migration 対象性を固定する。
- Content Pipeline の cache を採用する場合は、cache が再生成可能であること、source of truth でないこと、削除時の復旧方法、開発リポジトリ非生成を固定する。
- Content Pipeline が生成する公開ファイルは、Site Output の責務として扱い、Content Pipeline 単独で `contents/` を直接置換してはならない。
- Content Pipeline を採用しても、ASB 管理 HTTPS JSON API の request / response / error format を変更してはならない。

#### 13.25.11 Site Routing / Site Rendering 昇格時仕様契約

Site Routing と Site Rendering を将来実装対象へ昇格する場合は、以下を固定する。

- route source の配置は ASB 名称の directory として定義し、旧プロジェクト由来の `src/pages/` をそのまま採用してはならない。
- route file extension は `.html`、`.md`、`.json`、将来確定した template extension の候補から個別に固定する。
- route path は小文字化しない。入力元 path の大小文字を保持し、衝突検出は配信環境の filesystem 差異を考慮して固定する。
- `index.html` は directory index として扱う候補とし、`/docs/intro` と `/docs/intro/` の扱いを trailing slash 方針として固定する。
- dynamic route syntax と catch-all route は採用可否を固定するまで実装しない。
- route conflict は static route、dynamic route、catch-all route、generated route の優先順位を固定するまで成功扱いにしてはならない。
- 404 は Project 単位の default 404 と system default 404 の優先順位を固定する。
- redirect を採用する場合は、status code、relative URL、absolute URL、loop detection、最大 hop 数を固定する。
- query string は route match に使わないことを標準案とする。
- fragment は HTTP request に含まれないため route 判定対象外とする。
- SSG は標準候補とする。
- SSR と Hybrid Rendering は、ASB の静的コンテンツ配信ホスティング責務を超える可能性があるため、採用する場合は公開 runtime、sandbox、timeout、state、log、security、Backup 対象性を別途固定する。
- SSR を採用する場合は、管理 HTTPS JSON API との責務分離、公開 route と管理 API の routing 優先順位、認証要否を固定する。
- 公開 API route は Rev.90 時点では実装対象外とし、採用する場合は ASB 管理 HTTPS JSON API と混同しない path prefix、request / response 形式、error 形式、実行権限を固定する。
- middleware は Rev.90 時点では実装対象外とし、採用する場合は適用順序、変更可能な request / response、禁止副作用、timeout を固定する。
- route manifest を採用する場合は、保存先、schemaVersion、必須フィールド、ソート順、再生成可否、migration 対象性を固定する。
- build manifest を採用する場合は、保存先、schemaVersion、asset、route、content metadata の記録範囲を固定する。
- preview server または dev server は ASB 本体 server と混同してはならない。採用する場合は別 artifact、別 process、HTTPS 必須性、生成物配置、終了処理、開発リポジトリ非生成を固定する。

#### 13.25.12 Site Output / Docs / Sitemap 昇格時仕様契約

Site Output、Blog、Docs、Sitemap、Ad Slot を将来実装対象へ昇格する場合は、以下を固定する。

- 出力対象は Blog、Docs、Sitemap、SEO metadata、i18n route、Ad Slot のうち採用するものを個別に固定する。
- 出力 artifact は対象 Project の公開静的ファイルとして扱う候補とし、`contents/` への反映は atomic publish を必須とする。
- 出力 artifact の配置、削除、上書き、rollback、Backup 対象性を固定する。
- Site Output が `contents/` を置換する場合は、Webhook deploy、File upload/delete、Backup restore との排他順序を固定する。
- Blog route は `/blog`、`/blog/:slug`、`/blog/page/:page` を標準候補とする。
- Blog pagination page size は設定化するか固定値にするかを昇格時に確定する。
- Blog の範囲外 page、存在しない slug、draft content は `404 Not Found` を標準案とする。
- Blog metadata は title、date、updated、tags、category、excerpt、lang、draft を候補とする。
- Docs route は階層 path、sidebar metadata、前後リンク、未存在ページ、slug 正規化を固定する。
- Docs sidebar metadata は表示順、階層、title、hidden、previous、next の採否を固定する。
- Sitemap は `/sitemap.xml` を標準候補とし、含める route、除外 route、hreflang、lastmod、更新タイミングを固定する。
- Sitemap の lastmod は content metadata、実ファイル mtime、build time のどれを使うかを固定する。
- SEO metadata は title、description、canonical、OG、Twitter Card の採否、escape、default、未指定時の扱いを固定する。
- i18n route は default locale、locale list、fallback、URL prefix、hreflang、未対応 locale の扱いを固定する。
- Ad Slot は slot ID、size、position、priority、language、route、device、period、rotation、tracking の採否を固定する。
- Ad Slot の injection は static build time injection を標準候補とし、request time injection と client side injection は別途採否を固定するまで実装しない。
- 外部広告ネットワーク SDK、ユーザー単位行動追跡、realtime bidding、個人情報ターゲティングは ASB 本体へ内蔵しない。
- Feed Generation は Rev.90 時点では採用せず、RSS / Atom を実装対象へ昇格する場合は別途仕様改訂を必須とする。

#### 13.25.13 Source Sync / External Data / Cache 昇格時仕様契約

Source Sync、External Data Integration、Runtime Cache を将来実装対象へ昇格する場合は、以下を固定する。

- Source Sync は GitHub Webhook デプロイと重複しない責務範囲を固定する。
- Git provider を増やす場合は、対象 provider、認証方式、secret 保存、webhook 署名検証、retry、冪等性、失敗時ログを固定する。
- 外部 API または Headless CMS を採用する場合は、認証方式、credential 保存、取得頻度、timeout、retry、rate limit 受信時挙動、schema validation、失敗時 fallback を固定する。
- analytics 連携を採用する場合は、収集対象、個人情報非取得、保存形式、保持期間、無効化条件を固定する。
- Runtime Cache は再生成可能データに限定し、source of truth として扱わない。
- Runtime Cache の保存先、key、TTL、invalidation、容量上限、削除時復旧、migration 対象外条件を固定する。
- Runtime Cache を理由に `.gitignore` が必要になる生成物を開発リポジトリ内へ作成してはならない。
- 外部DBを前提とする Persistent State Store は、外部DB不使用方針が変更されるまで実装してはならない。

#### 13.25.14 実装昇格禁止チェック

移管元由来仕様の実装昇格前には、以下をすべて確認する。

- ASB 仕様正本に正式機能名がある。
- Auteur 名を外部公開名、API名、設定名、JSON名、ディレクトリ名、package 名、SDK名、Web UI名、CLI subcommand 名に使用していない。
- `auteur.config.json`、`.auteur/`、`auteur-project/`、`AUTEUR_*` error code を採用していない。
- Node.js、npm、package manager、bundler、transpiler、external UI framework を ASB 公式実装へ導入していない。
- 開発リポジトリ内に実行時データ、cache、一時ファイル、ログ、build output を生成しない。
- `storage.basePath` 配下へ生成するデータは JSON ファイルベース、静的コンテンツ実体、証明書、ログ、backup のいずれかに分類されている。
- ASB 管理 HTTPS JSON API と公開サイト route の責務境界が分離されている。
- ASB SDK と ASB 標準Web UI の責務境界を変更する場合は、事前に該当章を改訂している。

#### 13.25.15 移管元由来仕様の禁止事項

ASB 本体、ASB SDK、ASB 標準Web UI は Rev.90 時点では以下を実装してはならない。

- Auteur 専用 API
- Auteur 専用設定
- Auteur 専用 JSON
- Auteur 専用ディレクトリ
- Auteur 専用 SDK
- Auteur 専用 Web UI
- Auteur 専用 CLI
- Auteur 互換モード
- Auteur migration
- Auteur import
- Auteur export
- Auteur plugin
- Auteur adapter
- Auteur bridge
- Auteur protocol
- Auteur runtime

以下の外部公開名、API名、設定名、JSON名、ディレクトリ名、package 名、SDK名、Web UI名、CLI subcommand 名に `Auteur` または `auteur` を使用してはならない。

- ASB 本体の binary 名
- ASB 本体の API path
- ASB 本体の request / response JSON key
- ASB 本体の error code
- ASB 本体の設定項目
- ASB 本体の保存 JSON
- ASB 本体の実行時ディレクトリ
- ASB SDK の公開 API
- ASB SDK の package / module 名
- ASB 標準Web UI の画面名
- ASB 標準Web UI の保存状態

Auteur 統合吸収のために以下の JSON ファイル、ディレクトリ、設定項目を作成してはならない。

- `config/auteur.json`
- `config/auteur_compat.json`
- `config/auteur_migration.json`
- `storage/auteur/`
- `storage/auteur_compat/`
- `storage/auteur_migration/`
- `auteur.*`
- `auteurCompat.*`
- `auteurMigration.*`

ASB は Auteur を理由に、開発リポジトリ内にも `storage.basePath` 配下にも、Auteur 用の状態、互換状態、移行状態、変換状態、同期状態、キャッシュ、一時ファイル、ログ、メタデータを生成してはならない。

既存文書、設計メモ、会話、外部資料に Auteur 名が存在する場合でも、13.25.1 で固定した移管元仕様以外を ASB の仕様入力元として扱わない。

移管元由来機能を ASB に追加する場合は、旧プロジェクト名を残さず、ASB の通常機能名として `ASB-spec.md` の該当章へ記載する。

移管元由来機能が既存の ASB 機能と責務上重なる場合は、既存機能へ無理に混ぜず、責務境界を明記する。

移管元由来機能を新しい ASB 通常機能として扱う場合は、Project、File、Domain、SSL、GitHub Webhook、Backup、Log と同じ粒度の機能名を定義し、実装前に `ASB-spec.md` を改訂して以下を最低限確定する。

- ASB における正式機能名
- 実装対象範囲
- 非対象範囲
- ASB 本体、ASB SDK、ASB 標準Web UI の責務分界
- API
- request / response
- error code
- 保存 JSON
- 設定項目
- 生成ファイル
- 生成ディレクトリ
- 外部依存の有無
- 開発リポジトリ非生成の保証
- migration の要否
- downgrade
- テスト条件

Auteur という名称は、変更履歴または統合吸収方針の説明に限り使用できる。

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
- Let’s Encrypt ACME v2 実通信（テスト用ACMEサーバーまたはモックで検証）
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

### 15.4 CI / 品質ゲート固定仕様

ASB の標準 CI は GitHub Actions とする。

CI は Pull Request と `main` への反映前確認で実行する。

Rev.90 時点では、CI 方針を仕様として固定する。GitHub Actions workflow ファイルは、Go 実装コード導入後に追加する。

CI workflow を追加する場合は、`.github/workflows/` 配下に配置する。

CI workflow は、ASB 本体、ASB SDK、ASB 標準Web UI、実行時データ、配布 artifact の責務境界を変更する根拠として扱ってはならない。

CI は以下を必須検証項目とする。

- `go test ./...` が成功すること
- `git diff --check` が成功すること
- `.gitignore` が存在しないこと
- 開発リポジトリ内に実行時データ、ログ、一時ファイル、cache、coverage output、build output、release artifact、download 済み asset、`dist/`、`.asb/` が残っていないこと
- 起動失敗、panic recovery、API response、test failure message、ログ出力に secret、内部絶対パス、stack trace、JSON断片が露出しないこと
- テスト fixture は静的入力としてのみ扱い、テスト実行により fixture を作成、更新、削除しないこと
- テスト中に必要な一時データは OS の一時領域のみを使用し、テスト終了時に削除されること
- テスト、CI、coverage、build、release 確認を理由に `.gitignore` を作成しないこと

CI は以下を行ってはならない。

- 開発リポジトリ内への実行時データ生成
- 開発リポジトリ内へのログファイル生成
- 開発リポジトリ内への一時ファイルまたは cache 生成
- 開発リポジトリ内への coverage output 生成
- 開発リポジトリ内へのビルド成果物または release artifact 生成
- `.gitignore` の作成、更新、使用
- 外部DB、SQLite、KVS、外部ストレージ、外部 secret store の導入
- GitHub repository 設定の変更
- Pull Request の自動 merge
- `main` への直接 push

CI でビルド確認を行う場合、成果物は CI runner の一時領域にのみ出力し、開発リポジトリへ残してはならない。

CI で coverage を確認する場合、coverage file を開発リポジトリ内に作成してはならない。coverage 出力が必要な場合は CI runner の一時領域または標準出力に限定する。

CI で E2E テストを行う場合、起動設定ファイル、`storage.basePath`、TLS 証明書、ログファイル、空状態 JSON はテスト用一時領域に事前配置し、ASB が初期化または不足補完してはならない。

CI 失敗は、該当実装フェーズ未完了として扱う。

### 15.5 CI 実体化前提固定仕様

GitHub Actions workflow の実体は、Go 実装コード導入後に追加する。

workflow 実体化時は、以下の job を標準構成とする。

| job | 目的 | 必須確認 |
|-----|------|---------|
| repository-cleanliness | リポジトリ非生成確認 | `.gitignore` 不在、生成物不在、差分不在 |
| test | Go テスト | `go test ./...` |
| format-check | whitespace 確認 | `git diff --check` |
| build-check | ビルド確認 | CI一時領域への amd64 / arm64 build |
| security-output-check | 出力安全性確認 | secret、内部絶対パス、stack trace、JSON断片の非露出 |

CI で許可する書き込み先は以下に限定する。

| 種別 | 許可先 | 開発リポジトリ内保存 |
|-----|--------|--------------------|
| Go test 一時データ | OS一時領域 | 禁止 |
| E2E 用 runtime JSON | OS一時領域に事前配置した `storage.basePath` | 禁止 |
| TLS テスト証明書 | OS一時領域 | 禁止 |
| coverage 出力 | CI runner 一時領域または標準出力 | 禁止 |
| build output | CI runner 一時領域 | 禁止 |
| release artifact 検証用出力 | CI runner 一時領域 | 禁止 |
| log 出力 | CI runner 標準出力またはOS一時領域 | 禁止 |

CI で禁止する生成物は以下に固定する。

- `.gitignore`
- `.asb/`
- `dist/`
- `coverage.out`
- `*.log`
- `*.tmp`
- cache directory
- test result file
- runtime JSON
- release artifact
- build binary
- checksum 作業ファイル
- downloaded asset
- lock file
- external tool state file

通常 build artifact は、CI におけるコンパイル確認のための一時成果物を指す。

release artifact は、GitHub Releases へ添付する配布成果物を指す。

通常 build artifact は release artifact として扱ってはならない。

release artifact を作成する場合は、安定版リリース手順に限定し、開発リポジトリ内へ生成してはならない。

CI で許可する外部通信は以下に限定する。

- GitHub Actions がリポジトリ checkout に必要とする GitHub 通信
- Go toolchain setup に必要な GitHub Actions 標準 action の取得
- Go 標準 toolchain の取得
- `gh` による Pull Request 状態確認

CI は以下の外部通信を行ってはならない。

- Let’s Encrypt 本番 ACME server への通信
- GitHub Webhook 実イベント送信
- 外部DB、外部KVS、外部ストレージへの通信
- DNS provider API への通信
- npm、Deno module registry、外部 package registry への通信
- telemetry、analytics、error reporting service への通信

外部通信を追加する場合は、実装前に `ASB-spec.md` を改訂し、通信先、目的、失敗時挙動、secret 扱い、生成物、テスト条件を確定しなければならない。

CI が失敗した場合、当該 Pull Request または当該実装フェーズは未完了とする。

CI 成功は、Pull Request merge を自動実行してよい根拠ではない。

CI 成功は、GitHub repository 設定を変更してよい根拠ではない。

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
| `config/auth.json` | 単一システム管理者認証スキーマ |
| `config/webhooks.json` | Webhook 冪等キー履歴スキーマ |
| `config/acme_accounts.json` | ACME account スキーマ |
| `config/acme_orders.json` | ACME order スキーマ |
| `config/acme_authorizations.json` | ACME authorization スキーマ |
| `config/acme_challenges.json` | ACME challenge スキーマ |
| `config/acme_renewals.json` | ACME renewal スキーマ |
| `storage/projects/:projectId/files.json` | File メタデータスキーマ |

静的コンテンツ実体、ログファイル、証明書ファイル、ビルド済みバイナリは、Rev.90 時点のマイグレーション対象外とする。

### 16.3 schemaVersion 固定

各実行時 JSON ファイルはトップレベルに `schemaVersion` を持つ。

Rev.90 時点の `schemaVersion` は `1` とする。

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

`--dry-run` は `config/migrations.json` を作成、更新、削除してはならない。

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

10 の `config/migrations.json` 保存に失敗した場合は、マイグレーション全体を失敗扱いとし、可能であれば事前バックアップからロールバックする。

11 の完了ログ記録に失敗した場合でも、1-10 が完了していればマイグレーションは成功扱いとする。

完了ログ記録失敗時は、標準エラーへ単一行で警告を出力する。

### 16.6 migrationHistory 保存形式

マイグレーション履歴は `config/migrations.json` に保存する。

`config/migrations.json` は起動時必須 JSON ファイルではないが、ASB が初回作成してはならない。

`config/migrations.json` は `asb migrate --apply` 実行時にも初回作成してはならない。

存在する場合の配置場所は実行時データ領域に限定し、開発リポジトリ内へ配置してはならない。

`config/migrations.json` の作成は ASB の全実行境界で禁止する。

ASB サーバー起動時、通常の API 処理、静的配信、Webhook、Backup、Log API、SSL 管理境界、`asb migrate --apply` では `config/migrations.json` を作成してはならない。

`config/migrations.json` が存在しない状態で `--apply` を実行する場合は、実行時データ生成禁止により失敗する。

運用者が事前に用意する `config/migrations.json` の空状態は以下に固定する。

```json
{"schemaVersion":1,"migrations":[]}
```

`config/migrations.json` が存在する場合は、JSON 構文、`schemaVersion: 1`、`migrations` 配列、未知フィールド不在を検証してから履歴を追記する。

`config/migrations.json` の作成または更新に失敗した場合は、マイグレーション成功扱いにしてはならない。

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

Rev.90 時点では以下を禁止する。

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

| バージョン | 日付 | 内容 |
|-----------|------|------|
| Rev.90 | 2026-09-09 | READMEを入口文書として位置づけ、仕様正本、HTML版、実装タスクとの参照Rev、責務境界、実装順序要約の整合確認を固定 |
| Rev.89 | 2026-09-09 | 実装フェーズ順序を依存関係に合わせ、単一システム管理者認証をP1へ統合し、ログ・監視、バックアップ・復旧、GitHub Webhook、無料独自SSLの順序を整理 |
| Rev.88 | 2026-09-09 | IMPLEMENTATION_TASKS.md のP0をP0-1〜P0-4へ分割し、P0初回実装とP0全体タスクの混同を解消 |
| Rev.87 | 2026-09-09 | P0初回実装単位を最小Go骨格と `asb version` に限定し、初回で実装しない対象、実装後確認、作成可否境界を固定 |
| Rev.86 | 2026-09-09 | 実装開始条件、仕様不足時の仕様改訂差し戻し、開発リポジトリ非生成確認、Go実装コード導入前のCI workflow先行作成禁止、P0着手前境界を固定 |
| Rev.85 | 2026-09-09 | CI workflow 実体化時の標準 job、許可書き込み先、禁止生成物、通常 build artifact と release artifact の区別、外部通信制限、CI失敗時の扱いを固定 |
| Rev.84 | 2026-09-09 | GitHub Actions を標準 CI とし、Pull Request と main 反映前確認、非生成検査、secret・内部情報非露出、fixture 静的入力、OS一時領域利用、`.gitignore` 非作成を品質ゲートとして固定 |
| Rev.83 | 2026-09-09 | リリース、配布artifact、GitHub Releases、install/update、rollback、systemd、運用者事前責務の境界を実装レベルで一括固定 |
| Rev.82 | 2026-09-09 | JSON保存、ファイル実体操作、Backup / Restore、Log rotation、SSL / ACME、Webhook deploy の保存・生成・副作用境界を実装レベルで一括固定 |
| Rev.81 | 2026-09-09 | 起動設定 `config.json`、CLI subcommand、起動失敗時 stderr、ログ出力、package 依存境界を実装レベルで最終固定 |
| Rev.80 | 2026-09-09 | timeout、request size、panic recovery、client disconnect、graceful shutdown、CORS非実装、Origin非使用、Health Check、サーバー実行境界由来の実行時データ・cache・queue・一時ファイル非生成を実装レベルで固定 |
| Rev.79 | 2026-09-09 | Host header を Domain 解決専用に固定し、reverse proxy header 非信頼、client IP の `Request.RemoteAddr` 由来、access log `remoteAddr`、proxy関連JSON・cache・状態ファイル非生成を実装レベルで固定 |
| Rev.78 | 2026-09-09 | 管理 HTTPS JSON API の共通レスポンスヘッダー、request body 契約、routing 失敗、timestamp、静的配信レスポンスとの境界、`ERR_NOT_FOUND`、`ERR_METHOD_NOT_ALLOWED` を実装レベルで固定 |
| Rev.77 | 2026-09-09 | Monitoring API を読み取り専用・都度計算に固定し、response key、型、単位、取得不可時 `null`、in-memory counter、メトリクスJSON・履歴JSON・cache・一時ファイル・監視DB非生成を実装レベルで固定 |
| Rev.76 | 2026-09-09 | ログファイルの起動時生成禁止、既存ログへの追記、ローテーション時のみの新現行ログ作成、ローテーション失敗時復旧、期限切れログ削除失敗時の扱い、Log API 非生成・非修復を実装レベルで固定 |
| Rev.75 | 2026-09-09 | 起動設定ファイルと `storage.basePath` 配下の実行時 JSON を分離し、運用者による事前配置対象、空状態 JSON、起動時検証順序、ログファイル事前配置、Project 別 files.json 検証境界、起動失敗時エラーコードを実装レベルで固定 |
| Rev.74 | 2026-09-09 | 実行時データの初期化、自動生成、不足補完を禁止し、asb init-runtime を実装対象外へ変更、起動時・install/update・SDK・Web UI による実行時データ生成禁止を固定 |
| Rev.73 | 2026-09-09 | GitHub Releases 配布成果物、checksums.txt 形式、install/update の成功・失敗条件、既存環境上書き禁止、rollback、systemd unit の実装仕様を固定 |
| Rev.72 | 2026-09-09 | Project、Domain、File、SSL、GitHub Webhook、Backup / Restore、Log の排他制御、lock取得順序、同時操作時の競合条件、lock取得失敗時の error、整合性検証タイミング、自動修復禁止方針を固定 |
| Rev.71 | 2026-09-09 | API 別失敗条件、HTTP status、error code、静的配信失敗条件、成功レスポンス禁止条件、HTTP status / error code 選択優先順位を固定し、ログ書き込み失敗エラーを正式化 |
| Rev.70 | 2026-09-09 | Project、Domain、File、SSL、GitHub Webhook、Backup、Log、Auth、Monitoring の API 個別 request schema、validation、保存先、更新順序、audit 対象と、保存 JSON の field 単位スキーマを固定 |
| Rev.69 | 2026-09-09 | 管理 HTTPS JSON API、JSON ファイルベース保存、静的配信、GitHub Webhook、無料独自SSL、Backup / Restore、Log / Audit の実装基盤契約を一括固定し、Content Pipeline、Site Routing、Site Rendering、Site Output、Blog、Docs、Sitemap、Ad Slot の将来昇格時仕様をより具体化 |
| Rev.68 | 2026-09-09 | 外部・旧プロジェクト吸収時は旧名称の別枠を作らず、既存機能へ無理に混ぜず、ASB 通常機能名へ分解して Project、Domain 等と同じ粒度で横並びに扱うポリシーを固定 |
| Rev.67 | 2026-09-09 | Auteur 由来の Content Pipeline、Site Routing、Site Rendering、Site Output、Source Sync、External Data Integration、Runtime Cache を将来昇格する場合の入力、出力、保存形式、生成物、API、設定、CLI、外部依存、実行時データ、migration、テスト条件を実装前契約として具体化 |
| Rev.66 | 2026-09-09 | Auteur リポジトリの `Auteur_Master_Specification.md` を仕様移管元として固定し、仕様のみを ASB 側へ再分類して吸収する方針、非移管対象、将来仕様候補、禁止事項を具体化 |
| Rev.65 | 2026-09-09 | Auteur を ASB に完全統合吸収し、独立製品・独立サブシステム・外部依存・互換対象・migration 元として扱わない方針と専用生成物禁止を仕様化 |
| Rev.64 | 2026-09-09 | 複数インスタンス、NFS、分散ストレージ、外部ストレージ、ログファイル暗号化、デスクトップ/モバイル向けSDK利用を将来計画固定仕様として実装境界まで明文化 |
| Rev.63 | 2026-09-09 | Rate limiting と SDK認証拡張を仕様上の定義、現行非実装範囲、生成禁止対象、将来昇格条件まで明文化 |
| Rev.62 | 2026-09-09 | Rev.62 の実装確定境界を整理し、HTTP/2をASB互換目標かつ現行非実装に固定、SDK認証拡張・デスクトップ/モバイル・Go SDK module path・外部配布の扱いを実装判断可能な粒度へ具体化 |
| Rev.61 | 2026-09-09 | ASBを単一ユーザー・単一システム管理者モデルへ固定し、初期デフォルトパスワード、管理APIの毎回パスワード認証、config/auth.json、SDK/Web UIのパスワード非永続化、複数ユーザー化の将来計画を実装レベルで整理 |
| Rev.60 | 2026-09-09 | クラウドサービス化とウイルススキャンを現行のASB互換対象外、Web UI禁止UI、非実装確認対象から削除し、過去履歴のみへ限定 |
| Rev.59 | 2026-09-09 | ASB SDK と ASB 標準Web UI の実装境界、配置、HTTPS baseUrl、公開API、直接API呼び出し禁止、配布条件を整理 |
| Rev.58 | 2026-09-09 | P3〜P7のドメイン管理、SSL状態基盤、無料独自SSL/ACME、GitHub Webhook、バックアップ・復旧、ログ・監視の実装境界を一括整理 |
| Rev.57 | 2026-09-09 | ファイル管理と静的配信の境界を整理し、files.json記録ファイルのみ配信、未記録ファイルの整合性不備、upload/overwrite/delete、MIME、Gzip、ETag、304、HEADの実装条件を固定 |
| Rev.56 | 2026-09-09 | config/migrations.jsonを起動時必須JSONおよびasb init-runtime初期作成対象から分離し、asb migrate --applyのみ初回作成可能な履歴JSONとして整理 |
| Rev.55 | 2026-09-09 | asb init-runtime、asb start、実行時データ初期化、起動時非生成、Project初期files.json作成境界を実装レベルで整理 |
| Rev.54 | 2026-09-09 | SSLStatusレスポンス、無料独自SSL APIごとの返却条件、SSL操作競合エラーを実装レベルで整理 |
| Rev.53 | 2026-09-09 | 無料独自SSL / Let’s Encrypt ACME v2 / HTTP-01の状態、JSON保存スキーマ、証明書取得・更新状態遷移、retry/backoffを実装レベルで整理 |
| Rev.52 | 2026-09-09 | ASB SDKを単一の公式SDK名称として固定し、Browser JavaScript、Deno専用TypeScript、Goを対応実装として整理 |
| Rev.51 | 2026-09-09 | ASB Web UIをASB標準Web UIとして固定し、webui/配置、別artifact配布、認証なし運用境界、外部開発者の独自UI許可と公式プロジェクト非関与を整理 |
| Rev.50 | 2026-09-09 | ASB SDKをBrowser JavaScript、Deno専用TypeScript、Goの対応実装に拡張し、HTTPS JSON API通信、外部依存禁止、生成物非生成の方針を整理 |
| Rev.49 | 2026-09-09 | ASB Web UIを本体外のブラウザ標準HTML/CSS/JavaScriptアプリとして固定し、画面一覧、SDK経由通信、外部依存禁止、永続状態非使用を整理 |
| Rev.48 | 2026-09-09 | 開発ローカルを含むASB管理APIとSDK通信をHTTPS JSON APIへ統一し、ASB本体がHTTPS管理APIを提供する前提へ修正 |
| Rev.47 | 2026-09-09 | ASB SDKをASB Web UI用のブラウザ専用JavaScript SDKに固定し、Node.js、npm、bundler、外部ライブラリ、SDK専用保存状態を前提にしない方針へ整理 |
| Rev.46 | 2026-09-09 | ASB本体をヘッドレスに固定し、ASB Web UIを本体外の内製管理画面、ASB SDKを管理HTTPS JSON API通信層として位置づけ、デスクトップアプリとモバイルアプリを詳細未定の将来計画へ整理 |
| Rev.45 | 2026-09-09 | READMEの概要・仕様正本バージョン表記とSSL証明書エラー説明を無料独自SSL・SSL証明書管理の確定仕様に合わせて修正 |
| Rev.44 | 2026-09-09 | SSL証明書の技術スタック記述を無料独自SSL / Let’s Encrypt ACME v2 / HTTP-01 の確定仕様に合わせ、旧ACME非実装方針の残存記述を修正 |
| Rev.43 | 2026-09-09 | XServer Static互換機能セットを整理し、静的配信、独自ドメイン、無料独自SSL、GitHub Webhookデプロイ、HTTPS JSON APIファイル管理、ログ・状態確認、バックアップ・復旧を対象として固定 |
| Rev.42 | 2026-09-09 | XServer Static互換目標にFTP/FTPS/SFTP等の転送プロトコル互換を含めず、ASBのファイル操作面をHTTPS JSON API、multipart upload、GitHub Webhookデプロイに限定 |
| Rev.41 | 2026-09-09 | XServer Static互換目標として無料独自SSLを定義し、Let’s Encrypt ACME v2、HTTP-01、証明書自動取得・自動更新を実装対象として固定 |
| Rev.40 | 2026-09-09 | 将来計画、保留事項、実装フェーズ外タスク、将来計画機能固定仕様からクラウドサービス化とウイルススキャンを削除 |
| Rev.39 | 2026-09-09 | README、DOCUMENT_INDEX、IMPLEMENTATION_TASKS、ASB互換目標表現を整備し、仕様正本・HTML生成物・実装タスク・文書索引の役割整合性を固定 |
| Rev.38 | 2026-09-09 | UUID・Project名・File名・保存path・Domain・branchの入力制約、request body上限、multipart上限、レスポンスヘッダー、Gzip条件、Webhook payload/対象ファイル、Backup tar.gz安全検証、JSON保存低レベル失敗、install/update/systemd権限を実装契約として固定 |
| Rev.37 | 2026-09-08 | 設定値の未指定・空文字・型不一致、API パス解析、複数ファイル失敗時整合性検証、ログ API 読み込み、install/update 失敗時復旧、起動時検証順序を実装契約として固定 |
| Rev.36 | 2026-09-08 | File upload、ファイル上書き、Webhook deploy、projects.used 再計算、files.json と実ファイル不整合検出、config/migrations.json 作成条件と保存失敗時の扱いを実装契約として固定 |
| Rev.35 | 2026-09-08 | Content-Type 判定、JSON charset 許可条件、エラーレスポンス error 固定文言、エラーログ message 固定文言、Backup restore 復元手順、restore-staging 削除失敗時の扱い、requestId 空文字許可条件、テスト fixture 許可範囲を実装契約として固定 |
| Rev.34 | 2026-09-08 | エラーコード定義、config/config.json 完全形、schemaVersion 付き runtime JSON 空状態、install/update の arch 判定、Webhook sourcePath 検証、SSL 証明書IDと証明書ファイル検証を実装契約として固定 |
| Rev.33 | 2026-09-08 | ASB-spec.md へのフェーズ詳細記載を禁止し、フェーズ番号、優先度、開発版バージョン、実装順序、実装タスク、フェーズ別完了条件を IMPLEMENTATION_TASKS.md に限定する責務分離を固定 |
| Rev.32 | 2026-09-08 | 実装ファイル構成、package 内ファイル役割、エラーコード固定表、テストファイル配置を追加し、仕様を実装直前の契約粒度へ具体化 |
| Rev.31 | 2026-09-08 | 実装フェーズ単位の開発版バージョン管理を追加し、IMPLEMENTATION_TASKS.md を優先度別フェーズ構成へ再編する方針を固定 |
| Rev.30 | 2026-09-08 | 将来計画、保留事項、検討・調査中事項を Rev.30 時点の実装対象外として固定し、GUI/Web UI/モバイル/クラウド/複数インスタンス/外部ストレージ/ログ暗号化/ウイルススキャン等のAPI・設定・JSON・ディレクトリ・外部依存追加禁止を実装レベルで固定 |
| Rev.29 | 2026-09-08 | ASB互換目標を将来到達目標として実装対象から分離し、XServer Static 相当仕様との関係、含む対象、含めない対象、互換目標を理由にした未確定機能実装禁止を実装レベルで固定 |
| Rev.28 | 2026-09-08 | ACME 実通信を ASB 本体に実装しない方針、SSL管理を証明書メタデータ・配置・検証に限定し、ACME/CA/challenge/自動更新関連データ非生成を実装レベルで固定 |
| Rev.27 | 2026-09-08 | SDK 本体は実装対象外のまま、将来 SDK 通信を ASB 管理 HTTPS JSON API と同一規格に固定し、SDK 専用プロトコル・エンドポイント・実行時データ非生成を実装レベルで固定 |
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
