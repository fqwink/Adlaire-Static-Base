# ASB 実装タスクリスト

## 1. 運用ルール

`ASB-spec.md` は、Adlaire-Static-Base（ASB）のマスター仕様書正本である。

本ファイルは、`ASB-spec.md` に基づいて実装タスクを管理する。

参照仕様バージョン: `ASB-spec.md Rev.39`

`ASB-spec.md` で仕様確定済みの事項のみを実装タスクとして扱う。

`ASB-spec.md` と本ファイルが矛盾する場合は、`ASB-spec.md` を正とする。

`ASB-spec.md` に記載のない内容を、実装確定タスクとして扱ってはならない。

実装はフェーズ単位で進める。

フェーズ番号、優先度、開発版バージョン、実装順序、実装タスク、フェーズ別完了条件は本ファイルで管理する。

`ASB-spec.md` に P0、P1、P2 などのフェーズ詳細を記載してはならない。

各実装フェーズには、開発版バージョン `v0.N` を1つ対応させる。

本ファイルのフェーズ番号、優先度、開発版バージョン、実装順序、実装タスク、フェーズ別完了条件は、`ASB-spec.md` へ逆流させない。

本ファイルに記載された実装フェーズ管理は、`ASB-spec.md` の仕様範囲、API、設定項目、JSON保存形式、生成ファイル、生成ディレクトリを追加または変更する根拠ではない。

フェーズ完了時は、該当フェーズの完了条件をすべて満たし、`go test ./...`、`git diff --check`、`.gitignore` 不在確認、開発リポジトリ内生成物確認を行う。

完了したフェーズは `13. 実装済みフェーズ` へ移動する。

`ASB-spec.md` の仕様改訂によりタスク内容が変わる場合は、本ファイルも正本仕様に合わせて更新する。

## 2. 実装フェーズ一覧

| 優先度 | フェーズ | バージョン | 実装単位 | 状態 |
|-------|---------|-----------|----------|------|
| 最高 | P0 | v0.1 | 基盤 | 未着手 |
| 最高 | P1 | v0.2 | API・JSON・起動検証 | 未着手 |
| 高 | P2 | v0.3 | 静的配信・ファイル管理 | 未着手 |
| 高 | P3 | v0.4 | ドメイン・SSL管理境界 | 未着手 |
| 高 | P4 | v0.5 | GitHub Webhook デプロイ | 未着手 |
| 中 | P5 | v0.6 | バックアップ・復旧 | 未着手 |
| 中 | P6 | v0.7 | ログ・監視 | 未着手 |
| 中 | P7 | v0.8 | マイグレーション | 未着手 |
| 低 | P8 | v0.9 | 配布・install/update | 未着手 |
| 低 | P9 | v0.10 | 禁止機能・非実装確認 | 未着手 |

## 3. P0 / v0.1 / 基盤

優先度: 最高

目的: ASB 本体を Go 単一バイナリとして実装できる最小構造を確定する。

### 実装タスク

- Go 1.21 以上による ASB 本体の単一バイナリ基盤を構築する。
- `go.mod` を作成する。
- `cmd/asb/main.go` を作成する。
- `cmd/asb/main.go` は設定読み込み、依存関係生成、HTTPサーバー起動、signal受信、graceful shutdown のみに限定する。
- `config`、`server`、`management`、`delivery`、`data`、`system` の package 境界を作成する。
- `internal/config/`、`internal/server/`、`internal/management/`、`internal/delivery/`、`internal/data/`、`internal/system/` を作成する。
- Management Domain、Delivery Domain、Data Domain、System Domain の基本ディレクトリ構成を作成する。
- Handler、Service、Entity の層構造を実装する。
- Handler が Service interface のみに依存する構造にする。
- Handler が JSON ファイルを直接読み書きしない構造にする。
- Service が Repository、Storage、Clock、IDGenerator、LogService interface に依存する構造にする。
- Entity がファイル入出力、HTTP 入出力、時刻取得、ID生成を行わない構造にする。
- Entity は保存形式とレスポンス形式の型定義のみを持つ構造にする。
- ドメイン間の接続を `cmd/asb/main.go` で一元管理する。
- 責務間の循環依存を禁止する。
- `config.Loader`、`server.Router`、`server.Responder`、`management.ProjectService`、`management.DomainService`、`management.SSLService`、`delivery.FileService`、`delivery.StaticService`、`delivery.WebhookService`、`data.JSONRepository`、`data.StorageService`、`data.BackupService`、`system.LogService`、`system.MonitoringService`、`system.Clock`、`system.IDGenerator` の公開 interface 境界を整備する。
- Project、File、Backup の UUID 形式 ID 生成を `crypto/rand` で実装する。
- UUID を RFC 4122 version 4、小文字16進、ハイフン付き36文字として生成・検証し、`crypto/rand` 失敗時は `ERR_INTERNAL` とする。
- Clock 注入により時刻取得を行い、時刻取得失敗時の `ERR_INTERNAL` をテスト可能にする。
- 外部ルーターライブラリを使わず、Go標準 `net/http` でルーティングする。
- `internal/system/id_test.go` を作成する。
- `internal/system/clock_test.go` を作成する。

### 完了条件

- `go test ./...` が成功する。
- package 間の循環依存がない。
- Handler が永続化層へ直接依存していない。
- `.gitignore` が存在しない。
- 開発リポジトリ内に実行時データ、ログ、一時ファイル、ビルド成果物が残っていない。

### 非対象

- API の全実装
- 静的配信
- Webhook
- バックアップ
- マイグレーション
- 配布成果物生成

## 4. P1 / v0.2 / API・JSON・起動検証

優先度: 最高

目的: HTTP JSON API、設定、起動時検証、JSON保存の共通契約を実装する。

### 実装タスク

- `config/config.json` の読み込み、デフォルト値適用、起動時バリデーションを実装する。
- `internal/config/config.go`、`internal/config/loader.go`、`internal/config/validate.go` を実装する。
- `internal/server/router.go`、`internal/server/response.go`、`internal/server/middleware.go` を実装する。
- `internal/data/json_repository.go` を実装する。
- `server.port`、`server.host`、`server.shutdownTimeout`、`storage.basePath`、`storage.maxProjectSize`、`ssl.email`、`ssl.renewBefore`、`log.level`、`log.format`、`log.maxSize` の起動時バリデーションを実装する。
- 任意設定項目の未指定時、親 object 未指定時、空文字、型不一致、数値の小数・指数表記・負数・`null` 拒否を仕様通り実装する。
- `storage.basePath` と `deploy.sourcePath` の絶対パス正規化、開発リポジトリ配下判定、Git worktree 判定を実装する。
- 設定未知フィールド検出時の起動失敗を実装する。
- 起動時に設定済み実行時データ領域の存在確認と権限検証を実装する。
- 起動時に実行時データ用のディレクトリまたはファイルを自動生成しない。
- 開発リポジトリを実行時データ保存先として扱わない。
- `storage.basePath` 配下の `config/`、`storage/`、`storage/projects/`、`logs/`、`certs/` の順序付き存在検証を実装する。
- `config/projects.json`、`config/domains.json`、`config/backups.json`、`config/webhooks.json`、各 Project の `storage/projects/:projectId/files.json` の順序付き起動時存在検証を実装する。
- 必須 JSON ファイルの構文検証、必須フィールド検証、未知フィールド拒否を実装する。
- Go標準 `net/http` によるHTTPサーバーを実装する。
- グレースフルシャットダウンと `shutdownTimeout` を実装する。
- API パスを静的ファイル配信より優先して判定する。
- API パス解析では URL path のみを使用し、query string と fragment をルーティング判定に使わない。
- API パスの URL decode 失敗、`//`、`.`、`..`、NUL、`\`、`/api`、`/api/` の拒否を実装する。
- API レスポンスの `application/json; charset=utf-8` 統一を実装する。
- JSON API の `Content-Type: application/json` 要求を実装する。
- JSON API request body 最大サイズ 1MiB と超過時 `413` / `ERR_INVALID_REQUEST` を実装する。
- charset 付き `Content-Type: application/json` を許可する。
- JSON decode で未知フィールド拒否、空 Body 拒否、後続トークン拒否を実装する。
- 未定義ルート `404 Not Found` と未対応メソッド `405 Method Not Allowed` を実装する。
- `405 Method Not Allowed` では `Allow` ヘッダーを返す。
- 共通JSONレスポンスと共通エラーレスポンス形式を実装する。
- エラーレスポンスに `error`、`code`、`timestamp`、`httpStatus` を含める。
- エラーレスポンスの `httpStatus` と実際の HTTP ステータスを一致させる。
- エラーレスポンスの `timestamp` を UTC RFC3339 秒精度に固定する。
- エラーレスポンスに内部ファイルパス、スタックトレース、機密値を含めない。
- すべてのHTTPレスポンスで `X-Request-Id` を返し、JSONレスポンスの `Content-Type` を固定する。
- `HEAD` と `304 Not Modified` でレスポンスボディを返さないことを共通レスポンス層で保証する。
- `204 No Content` を使用しない。
- 配列レスポンスは対象データが空でも空配列を返す。
- URL パラメータ `:id`、`:domain`、`:name` の URL decode、正規化、バリデーションを実装する。
- `:id` は Rev.39 の UUID 正規表現に一致する値のみ許可する。
- `:domain` は小文字正規化後、label数、全体長、label正規表現、末尾 `.` 除去を仕様通り検証する。
- `:name` は長さ、NUL、パス区切り、`.`、`..`、先頭 `.`、空白のみを仕様通り拒否する。
- JSON ファイル更新時の読み込み検証、保存前再検証、同一ファイル排他書き込みを実装する。
- JSON ファイル保存では同一ディレクトリ内の一時ファイル、`fsync`、atomic rename による置換を実装する。
- JSON encode、file fsync、directory fsync、atomic rename の失敗時に成功レスポンスを返さない。
- `data.JSONRepository` は JSON 読み込み、スキーマ検証、排他、atomic save のみに限定する。
- `data.JSONRepository` が HTTP ステータス、HTTP リクエスト、HTTP レスポンスを扱わないことを実装する。
- `data.StorageService` は `storage.basePath` 配下のファイル実体操作のみに限定する。
- `data.StorageService` が Project、Domain、Webhook の業務判断を行わないことを実装する。
- 一時ファイルを開発リポジトリ内へ作成しない。
- 保存失敗時に成功レスポンスを返さない。
- 複数JSON更新では最終JSONの保存完了まで成功レスポンスを返さない。
- 複数JSON更新の途中失敗時に更新済みJSON名、未更新JSON名、操作名、requestId をエラーログへ記録する。
- 複数ファイル更新の途中失敗時に、更新予定JSON、更新済みJSON、`files.json` path、実ファイル、`projects.used` の整合性検証を実装する。
- 整合性検証失敗時は `ERR_STORAGE_VALIDATION_FAILED` を error log へ記録する。
- Rev.39 時点では複数JSON更新に外部トランザクション機構を導入しない。
- Rev.39 時点の API エンドポイント固定表に記載されたメソッド、パス、成功ステータス、失敗コードを実装する。
- Rev.39 API 成功レスポンス固定表に記載された JSON キーと型を実装する。
- プロジェクト作成 API `POST /api/projects` を実装する。
- プロジェクト一覧 API `GET /api/projects` を実装する。
- プロジェクト削除 API `DELETE /api/projects/:id` を実装する。
- プロジェクト名、quota、ID、作成日時のバリデーションを実装する。
- `config/projects.json` によるプロジェクト情報の永続化を実装する。
- `config/projects.json` の `projects[]` スキーマ、`createdAt` 昇順、同一時刻時 `id` 昇順を実装する。
- Project削除処理順序を Project検証、関連Domain列挙、関連Backup列挙、`files.json`検証、`contents/`削除、`files.json`削除、`domains.json`更新、`backups.json`更新、`projects.json`更新、成功応答の順に固定して実装する。
- Project削除を best effort 成功扱いにしない。
- プロジェクト名重複を `ERR_PROJECT_ALREADY_EXISTS` として扱う。
- 存在しないプロジェクト参照を `ERR_PROJECT_NOT_FOUND` として扱う。
- `internal/config/loader_test.go`、`internal/config/validate_test.go`、`internal/server/response_test.go`、`internal/data/json_repository_test.go` を作成する。

### 完了条件

- 起動時検証順序、終了コード、stderr 形式のテストが成功する。
- 全API成功レスポンスの固定JSONキー検証テストが成功する。
- 全APIエラーレスポンスの固定JSONキー検証テストが成功する。
- 保存JSONの未知フィールド拒否、相対パス保存、ソート順検証テストが成功する。
- Project API の作成、一覧、削除、重複、存在なし参照のテストが成功する。
- `go test ./...` が成功する。
- `.gitignore` が存在しない。
- 開発リポジトリ内に実行時データ、ログ、一時ファイル、ビルド成果物が残っていない。

### 非対象

- ファイルアップロード
- 静的配信
- ドメインAPI
- SSL証明書実通信
- Webhook
- バックアップ

## 5. P2 / v0.3 / 静的配信・ファイル管理

優先度: 高

目的: 静的コンテンツ配信とファイル管理APIを実装する。

### 実装タスク

- ファイルアップロード API `POST /api/projects/:id/files/upload` を実装する。
- `internal/delivery/file.go`、`internal/delivery/static.go`、`internal/delivery/handler.go` を実装する。
- `internal/data/storage.go` を実装する。
- ファイル一覧 API `GET /api/projects/:id/files` を実装する。
- ファイル削除 API `DELETE /api/projects/:id/files/:name` を実装する。
- `storage/projects/:projectId/files.json` によるファイルメタデータ管理を実装する。
- `storage/projects/:projectId/files.json` の `files[]` スキーマ、`name` 昇順、相対パス保存を実装する。
- フォルダ階層を保持した静的ファイル管理を実装する。
- プロジェクト quota に基づく容量制限を実装する。
- ファイル名、サイズ、パス区切り文字禁止のバリデーションを実装する。
- multipart アップロードのフィールド名を `file` に固定する。
- multipart upload の request body 最大サイズ 1GiB + 1MiB、file part 複数拒否、0 byte 拒否、1GiB超過時 `413` を実装する。
- File `path` は `contents/` で始まる相対パスのみ許可し、絶対パス、NUL、`\`、`.`、`..`、空白のみセグメントを拒否する。
- ファイルアップロード処理順序を URL検証、Project検証、multipart検証、ファイル検証、既存JSON検証、一時ファイル書込、fsync、atomic rename、`files.json`更新、`projects.json`更新、成功応答の順に固定して実装する。
- ファイル削除処理順序を URL検証、Project検証、`files.json`検出、ファイル実体削除、`files.json`更新、`projects.json` used更新、成功応答の順に固定して実装する。
- 同名ファイル上書き時の使用容量再計算を実装する。
- 同名ファイル上書き時は旧メタデータ読み込み、旧ファイル実体検証、新ファイル一時書込、fsync、旧ファイル退避、新ファイル公開、`files.json`更新、`projects.json` used差分更新、旧ファイル退避削除の順で実装する。
- 旧ファイルは新ファイルの atomic rename 成功まで削除しない。
- File upload / overwrite 失敗時に成功レスポンスを返さず、仕様に従って一時ファイル、退避ファイル、公開済みファイルの削除または復元を行う。
- `projects.used` は通常時差分更新とし、不一致、負数、整合性検証時は `files[]` の `size` 合計から再計算する。
- `files.json` と `contents/` の不整合検出を実装し、未記録ファイル、実体欠落、通常ファイル以外、サイズ不一致、配信ルート外 path を失敗扱いにする。
- Host ヘッダーから Domain を解決する。
- Domain から Project を解決する。
- URL path を静的ファイルパスへ変換する。
- `GET` と `HEAD` の静的配信を実装する。
- ディレクトリURLでは `index.html` を解決する。
- path traversal を拒否する。
- 隠しセグメントを拒否する。
- 配信ルート外参照を拒否する。
- Content-Type を Go標準ライブラリで判定する。
- Gzip による静的ファイル圧縮を実装する。
- Gzip は `Accept-Encoding` に `gzip` token があり `q=0` でない `GET 200 OK` のみ対象にする。
- Gzip 圧縮では `.gz`、キャッシュ、メタデータを生成せず、圧縮開始後失敗時は接続終了と `ERR_INTERNAL` ログ記録を実装する。
- `HEAD` ではレスポンスボディを返さない。
- `ETag` を `W/"{size}-{unixModifiedTime}"` 形式で返す。
- `Last-Modified` を HTTP-date 形式で返す。
- `Cache-Control` を既定で `public, max-age=60` とする。
- `If-None-Match` と `If-Modified-Since` による `304 Not Modified` を実装する。
- Range request は Rev.39 時点では実装せず、`Range` ヘッダーを無視して `206 Partial Content` を返さない。
- `Accept-Encoding: br` では Brotli 応答を返さない。
- Brotli 用の `.br`、キャッシュ、一時ファイル、メタデータを開発リポジトリ内にも `storage.basePath` 配下にも生成しない。
- 静的配信でディレクトリ一覧を返さない。
- 静的配信で開発リポジトリ内に配信用一時ファイル、キャッシュファイル、実行時データを作成しない。
- `internal/delivery/file_test.go`、`internal/delivery/static_test.go`、`internal/data/storage_test.go` を作成する。

### 完了条件

- File API の upload、list、delete、overwrite のテストが成功する。
- 静的配信の Host 解決、`GET`、`HEAD`、`405`、`index.html` 解決、path traversal 拒否、隠しセグメント拒否、Content-Type、ETag、Last-Modified、Cache-Control、304、Range無視、Gzip のテストが成功する。
- Brotli 用外部ライブラリ、middleware、precompress、`.br`、キャッシュ、設定項目、メタデータを生成しないテストが成功する。
- `go test ./...` が成功する。
- `.gitignore` が存在しない。
- 開発リポジトリ内に配信用一時ファイル、キャッシュ、実行時データ、ビルド成果物が残っていない。

### 非対象

- Brotli 圧縮
- Range request
- 外部CDN連携
- ウイルススキャン

## 6. P3 / v0.4 / ドメイン・SSL管理境界

優先度: 高

目的: ドメイン管理とSSL証明書管理境界を、ACME実通信なしで実装する。

### 実装タスク

- ドメイン追加 API `POST /api/projects/:id/domains` を実装する。
- `internal/management/domain.go`、`internal/management/ssl.go`、`internal/management/handler.go` を実装する。
- ドメイン一覧 API `GET /api/projects/:id/domains` を実装する。
- ドメイン削除 API `DELETE /api/projects/:id/domains/:domain` を実装する。
- `config/domains.json` によるドメイン情報の永続化を実装する。
- `config/domains.json` の `domains[]` スキーマと `domain` 昇順を実装する。
- RFC 1035 準拠のドメインバリデーションを実装する。
- ドメインの小文字正規化、253文字以下、最大3階層制限を実装する。
- ドメイン重複割り当てを `ERR_DOMAIN_ALREADY_ASSIGNED` として扱う。
- SSL証明書管理境界を実装する。
- 設定済み証明書保存先 `certs/` の検証と管理を実装する。
- SSL証明書ID、証明書メタデータ、証明書ファイルパス検証、有効期限監視モデル、失敗エラー `ERR_SSL_CERT_GENERATION_FAILED` を実装する。
- SSL更新状態は手動配置または ASB 外部運用の結果として確認できる管理モデルに限定する。
- ACME による証明書取得・更新を ASB互換目標として扱うが、Rev.39 時点では実通信を実装しない。
- 証明書ファイルそのものを ASB 本体で生成、取得、更新、削除、失効しない。
- ACME client、ACME account 登録、ACME account key 生成/保存、ACME directory 取得、ACME nonce 取得、ACME order 作成、ACME authorization 取得、ACME challenge 応答、ACME finalize、ACME certificate download、ACME revoke を実装しない。
- DNS-01 challenge、HTTP-01 challenge、TLS-ALPN-01 challenge、wildcard 証明書自動取得、複数 CA 連携、CA 選定、証明書自動更新、証明書更新スケジューラー、challenge 状態管理、ACME retry、ACME rate limit 回避を実装しない。
- `config/acme.json`、`config/ca.json`、`config/acme_accounts.json`、`config/acme_orders.json`、`config/acme_authorizations.json`、`config/acme_challenges.json` を作成しない。
- `storage/acme/`、`storage/acme/accounts/`、`storage/acme/orders/`、`storage/acme/challenges/`、`storage/certs/acme/` を作成しない。
- `acme.*`、`ca.*`、`ssl.acme.*` 設定項目を定義しない。
- `config/config.json` に ACME 関連フィールドまたは CA 選定関連フィールドが存在する場合は、未知フィールドとして起動失敗させる。
- `internal/management/domain_test.go`、`internal/management/ssl_test.go` を作成する。

### 完了条件

- Domain API の追加、一覧、削除、重複、存在なし参照のテストが成功する。
- 証明書ファイルが手動配置または ASB 外部運用で配置された前提で、ASB が存在、パス、有効期限のみを検証するテストが成功する。
- ACME client、challenge、CA連携、証明書自動更新、ACME 関連 JSON ファイルまたはディレクトリを生成しないテストが成功する。
- `config/config.json` に ACME 関連フィールドまたは CA 選定関連フィールドが存在する場合に未知フィールドとして起動失敗するテストが成功する。
- `go test ./...` が成功する。
- `.gitignore` が存在しない。

### 非対象

- ACME 実通信
- CA 選定
- ワイルドカード証明書自動取得
- 証明書自動更新
- TLS 終端

## 7. P4 / v0.5 / GitHub Webhook デプロイ

優先度: 高

目的: ローカルcheckoutを唯一のデプロイ元とする GitHub Push Webhook デプロイを実装する。

### 実装タスク

- GitHub Webhook API `POST /api/webhook/github` を実装する。
- `internal/delivery/webhook.go` を実装する。
- GitHub Push イベントの検出を実装する。
- `X-GitHub-Event` が `push` 以外の場合は `200 OK` と `{"status":"ignored"}` を返す。
- 対象ブランチ設定と対象外ブランチの成功扱い無視を実装する。
- 対象外ブランチの場合は `200 OK` と `{"status":"ignored"}` を返す。
- 同一冪等キー受信時は `200 OK` と `{"status":"duplicate"}` を返す。
- `webhook.githubSecret` 未設定時はWebhook署名検証を行わない。
- `webhook.githubSecret` 設定時は `X-Hub-Signature-256` を必須にする。
- Webhook署名をGo標準ライブラリ `crypto/hmac` と `crypto/sha256` で検証する。
- Webhook署名比較を `hmac.Equal` で実装する。
- 署名なし、不正形式、不一致を `401 Unauthorized` と `ERR_WEBHOOK_SIGNATURE_INVALID` で拒否する。
- GitHub Push payload の `repository.clone_url`、`repository.ssh_url`、`repository.html_url` をデプロイ元に使わない。
- GitHub Push payload の `ref`、`after`、`repository` object 必須検証を実装する。
- `ref` は `refs/heads/{branch}` のみ処理し、それ以外は `ignored` とする。
- `ref` から抽出した branch と `deploy.branch` の完全一致判定を実装する。
- `deploy.projectId` をWebhookデプロイ先Projectとして扱う。
- `deploy.projectId` 未設定時は `ERR_WEBHOOK_PROJECT_NOT_CONFIGURED` を返す。
- `deploy.sourcePath` のローカルcheckoutを唯一のデプロイ元として扱う。
- Webhook処理時にネットワーク越しのGit clone、fetch、pullを行わない。
- `deploy.sourcePath` の存在、Git worktree、`after` commit 参照可否を検証する。
- `deploy.sourcePath` 不正時は `ERR_WEBHOOK_SOURCE_INVALID` を返す。
- 指定ブランチの自動デプロイ処理を実装する。
- Webhookデプロイ対象から `.git/`、`.github/`、主要仕様・管理ドキュメントを除外する。
- Webhookデプロイ対象を `after` commit の通常ファイルに限定し、symlink、submodule、directory、Git管理外ファイルを除外する。
- Webhookデプロイ対象pathの絶対パス、NUL、`\`、`.`、`..`、空白のみセグメント、先頭 `.` セグメントを拒否する。
- Webhookデプロイ先を対象Projectの `storage/projects/:projectId/contents/` 配下に限定する。
- Webhookデプロイを一時ディレクトリ作成、静的ファイルコピー、fsync、atomic rename、`files.json`更新、`webhooks.json`保存の順に実装する。
- Webhookデプロイでは既存 `contents/` を `deploy-staging/{deployId}/previous-contents/` へ退避し、新 `contents/` 公開失敗または `files.json` 更新失敗時に仕様に従って復元する。
- `deploy-staging/{deployId}/` 削除失敗時は `Operational warning` を記録し、確定済み処理結果を変更しない。
- デプロイ状態を確認できる管理モデルを実装する。
- Webhook処理成功・失敗ログを実装する。
- Webhook 処理の冪等性方針を仕様に従って実装する。
- `config/webhooks.json` による Webhook 冪等キー履歴保存を実装する。
- `config/webhooks.json` の `events[]` スキーマ、`status`、`completedAt`、`errorCode`、`receivedAt` 降順、同一時刻時 `key` 昇順を実装する。
- Webhook失敗時は `failed` と `errorCode` を `config/webhooks.json` に保存する。
- Webhook失敗時の自動リトライスケジューラーを実装しない。
- `internal/delivery/webhook_test.go` を作成する。

### 完了条件

- GitHub Push payload の正常系、対象外イベント、対象外ブランチ、重複イベント、署名不正、sourcePath不正のテストが成功する。
- Webhook処理時にネットワーク越しのGit clone、fetch、pullを行わないテストまたはレビューが完了する。
- Webhookデプロイ処理順序テストが成功する。
- Webhook失敗時の自動リトライが存在しないことを確認する。
- `go test ./...` が成功する。
- `.gitignore` が存在しない。

### 非対象

- GitHub API 連携
- ネットワーク越しのGit操作
- 自動リトライスケジューラー
- GitHub Actions 実行

## 8. P5 / v0.6 / バックアップ・復旧

優先度: 中

目的: JSONファイルベースの実行時データを対象に、tar.gzバックアップと復旧を実装する。

### 実装タスク

- バックアップ一覧 API `GET /api/backups` を実装する。
- `internal/data/backup.go` を実装する。
- バックアップ復旧 API `POST /api/backups/restore/:id` を実装する。
- `config/backups.json` によるバックアップ履歴管理を実装する。
- `config/backups.json` の `backups[]` スキーマ、`createdAt` 降順、同一時刻時 `id` 昇順を実装する。
- tar.gz 形式のバックアップ作成を実装する。
- Backup作成処理順序を Project検証、対象データ読み込み検証、`storage.basePath/backups/` 書き込み検証、tar.gz一時作成、SHA-256計算、atomic rename、`backups.json` へ `status: "completed"` 保存、成功応答の順に固定して実装する。
- SHA-256 ハッシュによるバックアップ整合性検証を実装する。
- 復旧前の既存データ退避を実装する。
- 復旧後の整合性確認を実装する。
- Backup復旧処理順序を Backup履歴検証、`status: "completed"` 検証、Backupファイル存在検証、SHA-256検証、`previous/` と `next/` 作成、現行データ退避、展開、展開後JSON検証、atomic rename、成功応答の順に固定して実装する。
- Backup復旧途中失敗時は可能な限り退避領域から復元し、復元失敗時は `ERR_BACKUP_RESTORE_FAILED` を返す。
- バックアップ保存先を `storage.basePath/backups/` に固定する。
- バックアップファイル名を `backup-{backupId}.tar.gz` に固定する。
- `config/backups.json` の `path` を `storage.basePath` からの相対パスとして保存する。
- `config/backups.json` の `status` を `completed` または `failed` として扱う。
- バックアップtar.gzには対象Projectの `files.json` と `contents/` のみを含める。
- バックアップtar.gzに `logs/`、`certs/`、他Projectの `contents/` を含めない。
- バックアップtar.gz内の symlink、hardlink、device、FIFO、socket、絶対パス、`..`、NUL、`\` を拒否する。
- Backup restore 展開時は各tarエントリを展開前に検証し、`next/` 配下外へ出るpathを拒否する。
- Backup restore 展開後に `next/files.json`、`next/contents/`、実ファイルと `files.json` の整合性を検証する。
- バックアップ作成用一時tarを `storage.basePath/backups/.tmp/` 配下に限定する。
- atomic rename 後に履歴保存へ失敗した場合は、作成済みtar.gzを削除する。
- バックアップ保存先を別障害領域へ複製する作業をASB外の運用責務として扱う。
- 外部ストレージ連携を Rev.39 時点では実装対象外として扱う。
- Backup復旧前退避先を `storage.basePath/backups/restore-staging/{restoreId}/previous/` に固定する。
- Backup復旧用展開先を `storage.basePath/backups/restore-staging/{restoreId}/next/` に固定する。
- Backup履歴の `status` が `completed` でない場合は復旧を拒否する。
- 復旧対象Projectの現行データ退避に失敗した場合は、復旧処理を開始しない。
- 復旧途中失敗時は `previous/` から復元する。
- `restore-staging/{restoreId}/` 削除失敗は WARN ログに記録する。
- 開発リポジトリ内にバックアップ、一時tar、checksum、復旧用一時ファイル、退避データ、展開データを作成しない。
- `internal/data/backup_test.go` を作成する。

### 完了条件

- Backup作成、Backup復旧、途中失敗、復元失敗、履歴保存失敗のテストが成功する。
- バックアップtar.gz構成とSHA-256検証テストが成功する。
- 外部ストレージ連携が存在しないことを確認する。
- 開発リポジトリ内にバックアップ関連生成物が残っていない。
- `go test ./...` が成功する。
- `.gitignore` が存在しない。

### 非対象

- 外部ストレージ連携
- 別障害領域への自動複製
- ログファイルと証明書ファイルのバックアップ

## 9. P6 / v0.7 / ログ・監視

優先度: 中

目的: JSON Linesログ、ログAPI、監視API、ログローテーションを実装する。

### 実装タスク

- ログ保存先を `storage.basePath/logs/` に固定する。
- `internal/system/log.go`、`internal/system/monitoring.go` を実装する。
- アクセスログを `storage.basePath/logs/access.log` に保存する。
- エラーログを `storage.basePath/logs/error.log` に保存する。
- アクセスログとエラーログを UTF-8 JSON Lines で出力する。
- アクセスログの固定フィールドと順序を実装する。
- エラーログの固定フィールドと順序を実装する。
- `requestId` をリクエスト受信時に生成し、`X-Request-Id`、アクセスログ、エラーログへ同一値で引き回す。
- `log.level` による DEBUG、INFO、WARN、ERROR の出力条件を実装する。
- 通常運用ログを stdout へ出力しない。
- 起動時検証失敗時のみ stderr へ `ASB_STARTUP_ERROR code=... message="..."` 形式で単一行を出力する。
- `log.maxSize` 到達後、次回書き込み前にログローテーションする。
- ローテーション後ファイル名を `access.log.{unixTime}` または `error.log.{unixTime}` に固定する。
- ローテーション済みログは7日経過後に削除対象とする。
- ログローテーションおよび期限切れ削除失敗を WARN として記録する。
- アクセスログ API `GET /api/logs/access` を実装する。
- エラーログ API `GET /api/logs/error` を実装する。
- ログ API は現行 `access.log` / `error.log` のみを新しい順に返す。
- `limit` と `offset` によるログ取得を実装する。
- `limit` はデフォルト100、最小1、最大1000を実装する。
- `offset` はデフォルト0、最小0を実装する。
- 壊れた JSON 行を検出した場合、ログ API は `500 Internal Server Error` を返す。
- ログ API は全行の decode とスキーマ検証が成功した後に `limit` / `offset` を適用する。
- ログ API で空行または壊れた JSON 行を検出した場合は部分成功を返さず、`ERR_LOG_READ_FAILED` を返す。
- `offset` がログ件数以上の場合は `200 OK` と `logs: []` を返す。
- ログ API ではログファイルを作成、更新、ローテーション、削除しない。
- システムステータス API `GET /api/monitoring/stats` を実装する。
- CPU、メモリ、ディスク、接続数、リクエスト数の監視値取得を実装する。
- OS 依存で取得できない監視値は `null` として成功レスポンスに含める。
- 開発リポジトリ内にログファイルを作成しない。
- `internal/system/log_test.go`、`internal/system/monitoring_test.go` を作成する。

### 完了条件

- アクセスログとエラーログのJSON Linesフィールド、フィールド順、requestId一致、stdout非出力、stderr起動失敗出力、ローテーションのテストが成功する。
- ログ API の新しい順、壊れたJSON行検出、現行ログのみ対象のテストが成功する。
- Monitoring API の成功レスポンスと取得不可項目 `null` のテストが成功する。
- 開発リポジトリ内にログファイルが残っていない。
- `go test ./...` が成功する。
- `.gitignore` が存在しない。

### 非対象

- ログファイル暗号化
- 外部監視サービス連携
- ローテーション済みログAPI

## 10. P7 / v0.8 / マイグレーション

優先度: 中

目的: 実行時 JSON ファイルのスキーマ移行を、明示コマンドのみで実装する。

### 実装タスク

- マイグレーション対象を `config/projects.json`、`config/domains.json`、`config/backups.json`、`config/webhooks.json`、`storage/projects/:projectId/files.json` に限定する。
- `internal/data/migration_test.go` を作成する。
- 各実行時 JSON ファイルのトップレベル `schemaVersion` を実装する。
- `schemaVersion` 未指定の JSON ファイルを `0` として扱う。
- 未対応 `schemaVersion` 検出時の起動失敗を実装する。
- 起動時の自動マイグレーションを禁止する。
- `asb migrate --storage /var/asb --from-schema 0 --to-schema 1 --dry-run` を実装する。
- `asb migrate --storage /var/asb --from-schema 0 --to-schema 1 --apply` を実装する。
- `--dry-run` と `--apply` の同時指定を拒否する。
- `--dry-run` では実行時 JSON ファイルを変更しない。
- `--dry-run` では `config/migrations.json` を作成、更新、削除しない。
- `--apply` では事前検証、事前バックアップ、変換後JSON生成、再検証、atomic rename、履歴保存、完了ログ記録の順に実装する。
- マイグレーション作業ファイル、一時ファイル、退避ファイル、履歴ファイルを開発リポジトリ内に作成しない。
- `config/migrations.json` の `schemaVersion`、`migrations[]`、`status`、`backupPath` スキーマを実装する。
- `config/migrations.json` は `asb migrate --apply` 実行時のみ作成し、起動時、API、静的配信、Webhook、Backup、Log API、SSL 管理境界では作成しない。
- `config/migrations.json` 作成または更新失敗時はマイグレーション成功扱いにしない。
- マイグレーション失敗時の事前バックアップからのロールバックを実装する。
- ロールバック失敗時に標準エラー、エラーログ、`config/migrations.json` へ `failed` として記録する。
- 外部DBマイグレーション、外部トランザクション機構、外部マイグレーションフレームワークを導入しない。

### 完了条件

- `schemaVersion` なしを `0` として扱うテストが成功する。
- 未対応 `schemaVersion` の起動失敗テストが成功する。
- `--dry-run` が実行時 JSON ファイルを変更しないテストが成功する。
- `--apply` が対象 JSON ファイルを atomic rename で更新するテストが成功する。
- 事前バックアップ作成、途中失敗時の成功禁止、ロールバック成功、ロールバック失敗時の `failed` 記録テストが成功する。
- 開発リポジトリ内に移行作業ファイルを作成しないテストが成功する。
- `go test ./...` が成功する。
- `.gitignore` が存在しない。

### 非対象

- 起動時の自動マイグレーション
- 外部DBマイグレーション
- 外部マイグレーションフレームワーク

## 11. P8 / v0.9 / 配布・install/update

優先度: 低

目的: 安定版リリース、配布成果物、インストール、アップデートの手順を実装する。

### 実装タスク

- 安定版リリース判定基準を実装手順として固定する。
- GitHub Releases を標準配布先として扱う。
- リリースタグを安定版バージョンと同一文字列にする。
- `asb-linux-amd64-vX.Y`、`asb-linux-arm64-vX.Y`、`checksums.txt` を標準配布成果物として生成する。
- `checksums.txt` のSHA-256形式と配布ファイル名一致を検証する。
- Linux amd64 と Linux arm64 の標準ビルドコマンドを実装する。
- ビルド成果物を開発リポジトリへ残さないリリース手順を実装する。
- `install.sh --version vX.Y --arch amd64|arm64` を実装する。
- `update.sh --version vX.Y --arch amd64|arm64` を実装する。
- `scripts/install_test.sh`、`scripts/update_test.sh` を作成する。
- `latest` 指定、自動最新版選択、未指定バージョンでの install/update 実行を拒否する。
- install/update でダウンロード失敗、checksum不一致、`--version` 不一致、systemd操作失敗を成功扱いしない。
- `install.sh` は既存 `/usr/local/bin/asb` が存在する場合に上書きせず失敗する。
- `install.sh` は checksum 検証、`--version` 出力確認、配置、`asb.service` 配置、`daemon-reload`、`enable`、`start` の順に実装する。
- `install.sh` / `update.sh` が配置する `/usr/local/bin/asb` と `/usr/local/bin/asb.previous` の owner、group、mode を仕様通り固定する。
- `update.sh` は checksum 検証と `--version` 出力確認が完了するまで、既存サービス停止、既存バイナリ退避、バイナリ置換を実行しない。
- `update.sh` は `/usr/local/bin/asb` 不在、または `/usr/local/bin/asb.previous` 既存の場合に失敗する。
- `update.sh` の起動失敗時に `/usr/local/bin/asb.previous` から復旧を試行する。
- `update.sh` は復旧に成功した場合でも終了コード `1` で失敗する。
- `asb.service` を `/etc/systemd/system/asb.service` 向けの固定仕様で提供する。
- `asb.service` の owner、group、mode と、`asb` system user のログイン不可・homeなし作成を実装する。
- `asb.service` の `ExecStart=/usr/local/bin/asb --config /etc/asb/config.json`、`Restart=on-failure`、`NoNewPrivileges=true` を実装する。

### 完了条件

- Linux amd64 と Linux arm64 のビルドが成功する。
- 配布成果物名と `checksums.txt` の整合テストが成功する。
- install/update の正常系、失敗系、`latest` 拒否、`--version` 不一致、checksum不一致、復旧処理のテストが成功する。
- ビルド成果物が開発リポジトリに残っていない。
- `go test ./...` が成功する。
- `.gitignore` が存在しない。

### 非対象

- 安定版リリースの実行
- Pull Request merge
- GitHub リポジトリ設定変更

## 12. P9 / v0.10 / 禁止機能・非実装確認

優先度: 低

目的: Rev.39 時点で実装対象外の機能が混入していないことを確認する。

### 実装タスク

- ASB互換目標を将来の到達目標として扱い、Rev.39 時点の実装対象として扱わない。
- `internal/asb_forbidden_test.go` を作成する。
- ASB互換目標に含まれることを、未確定機能の実装根拠として扱わない。
- ASB互換目標を理由に `.gitignore`、外部DB、未承認外部ライブラリ、未承認外部サービス連携、開発リポジトリ内実行時データ、起動時自動生成、ビルド成果物自動生成を追加しない。
- ASB互換目標を理由に APIキー管理、ユーザー認証、Rate limiting、Brotli圧縮、ACME実通信、CA選定、SDK本体、SDK専用通信を実装しない。
- 将来計画、保留事項、検討・調査中事項を Rev.39 時点の実装対象として扱わない。
- GUI、Web UI、デスクトップアプリ、モバイルアプリ、クラウドサービス化、SaaS基盤、ユーザー管理、マルチテナント、課金管理、契約管理、複数インスタンス管理、クラスタ管理、分散ロック、NFS専用連携、分散ストレージ専用連携、外部ストレージサービス連携、ウイルススキャン、ログファイル暗号化、HTTP/2実装詳細を実装しない。
- 将来計画機能を理由に UI用API、モバイル専用API、クラウド用API、テナント用API、課金用API、契約用API、外部ストレージ用API、ウイルススキャン用API、ログ暗号化用APIを追加しない。
- 将来計画機能を理由に `ui.*`、`webui.*`、`desktop.*`、`mobile.*`、`cloud.*`、`tenant.*`、`billing.*`、`nfs.*`、`cluster.*`、`distributedStorage.*`、`externalStorage.*`、`virusScan.*`、`logEncryption.*` 設定項目を追加しない。
- 将来計画機能を理由に UI用JSON、モバイル用JSON、クラウド用JSON、テナント用JSON、課金用JSON、外部ストレージ用JSON、ウイルススキャン用JSON、ログ暗号化用JSONを追加しない。
- 将来計画機能を理由に UI用ディレクトリ、モバイル用ディレクトリ、クラウド用ディレクトリ、テナント用ディレクトリ、課金用ディレクトリ、外部ストレージ用ディレクトリ、ウイルススキャン用ディレクトリ、ログ暗号化用ディレクトリを追加しない。
- 管理 API が `Authorization` ヘッダーまたは `X-API-Key` ヘッダーの有無でレスポンスを変えないことをテストする。
- APIキー、ユーザー、セッション、認証状態を表す JSON ファイルまたはディレクトリを生成しないことをテストする。
- `config/config.json` に認証関連フィールドが存在する場合に未知フィールドとして起動失敗することをテストする。
- Rate limiting 用 middleware、制限アルゴリズム、永続カウンタ、設定項目、JSON ファイルまたはディレクトリを生成しないことをテストする。
- 管理 API、静的配信、Webhook 受信が Rate limiting 関連条件でレスポンスを変えないことをテストする。
- `config/config.json` に Rate limiting 関連フィールドが存在する場合に未知フィールドとして起動失敗することをテストする。
- SDK 通信が ASB 管理 HTTP JSON API と同一規格であることをテストする。
- SDK 専用プロトコル、SDK 専用エンドポイント、SDK 専用セッション、SDK 専用 JSON ファイルまたはディレクトリを生成しないことをテストする。
- `config/config.json` に SDK 通信関連フィールドが存在する場合に未知フィールドとして起動失敗することをテストする。
- 将来計画機能を理由に未確定 API、設定項目、JSONファイル、ディレクトリ、外部依存が追加されていないことをテストまたはレビューで確認する。
- 将来計画、保留事項、検討・調査中事項が個別確定仕様なしに実装対象へ昇格していないことを確認する。

### 完了条件

- 禁止機能の非生成テストまたはレビューが完了する。
- 認証、Rate limiting、SDK、ACME、Brotli、将来計画機能が実装されていないことを確認する。
- `.gitignore` が存在しない。
- 開発リポジトリ内に実行時データ、ログ、一時ファイル、ビルド成果物が残っていない。
- `go test ./...` が成功する。

### 非対象

- 禁止機能の実装
- 未確定タスクの実装
- 将来計画機能の仕様昇格

## 13. 実装済みフェーズ

現時点ではなし。

## 14. 実装フェーズ外の仕様未確定タスク

以下は Rev.39 時点では実装フェーズに含めない。

- ACME protocol 対応範囲、CA選定、複数CA、challenge方式、account key 保護、DNS provider連携、retry、rate limit、テスト方法、失敗時挙動を確定する。
- SDK 本体、SDK 配布方針、SDK 認証仕様を確定する。
- SSL証明書自動更新の実通信とスケジューリング仕様を確定する。
- HTTP/2 実装詳細を実装対象へ昇格する場合の API、設定項目、テスト条件を仕様改訂で確定する。
- GUI、デスクトップアプリ、Web UI を実装対象へ昇格する場合のリポジトリ境界、API、設定項目、外部依存、生成物を仕様改訂で確定する。
- モバイルアプリを実装対象へ昇格する場合の API、認証、配布、設定項目、外部依存を仕様改訂で確定する。
- クラウドサービス化を実装対象へ昇格する場合のテナント、認証、課金、契約、アカウント管理、保存JSON、外部依存を仕様改訂で確定する。
- 複数インスタンス対応、NFS連携、分散ストレージ連携を実装対象へ昇格する場合のロック、整合性、障害時挙動、設定項目、外部依存を仕様改訂で確定する。
- 外部ストレージサービス統合を実装対象へ昇格する場合のAPI、認証、保存JSON、バックアップ整合性、外部SDK採否を仕様改訂で確定する。
- ログファイル暗号化を実装対象へ昇格する場合の鍵管理、暗号化形式、復号API、外部KMS採否、移行手順を仕様改訂で確定する。
- ウイルススキャンを実装対象へ昇格する場合のスキャン方式、隔離、削除、外部API採否、保存JSON、テスト条件を仕様改訂で確定する。
