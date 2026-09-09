# ASB 実装タスクリスト

## 1. 運用ルール

`ASB-spec.md` は、Adlaire-Static-Base（ASB）のマスター仕様書正本である。

本ファイルは、`ASB-spec.md` に基づいて実装タスクを管理する。

参照仕様バージョン: `ASB-spec.md Rev.68`

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
| 高 | P3 | v0.4 | ドメイン管理・SSL状態基盤 | 未着手 |
| 高 | P4 | v0.5 | 無料独自SSL / Let’s Encrypt ACME v2 | 未着手 |
| 高 | P5 | v0.6 | GitHub Webhook デプロイ | 未着手 |
| 中 | P6 | v0.7 | バックアップ・復旧 | 未着手 |
| 中 | P7 | v0.8 | ログ・監視 | 未着手 |
| 中 | P8 | v0.9 | マイグレーション | 未着手 |
| 低 | P9 | v0.10 | 配布・install/update | 未着手 |
| 低 | P10 | v0.11 | ASB SDK | 未着手 |
| 低 | P11 | v0.12 | ASB 標準Web UI | 未着手 |
| 低 | P12 | v0.13 | 禁止機能・非実装確認 | 未着手 |
| 高 | P13 | v0.14 | 単一システム管理者認証 | 未着手 |

## 3. P0 / v0.1 / 基盤

優先度: 最高

目的: ASB 本体を Go 単一バイナリとして実装できる最小構造を確定する。

### 実装タスク

- Go 1.21 以上による ASB 本体の単一バイナリ基盤を構築する。
- `go.mod` を作成する。
- `cmd/asb/main.go` を作成する。
- `cmd/asb/main.go` は設定読み込み、依存関係生成、HTTPSサーバー起動、signal受信、graceful shutdown のみに限定する。
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
- `config.Loader`、`server.Router`、`server.Responder`、`management.ProjectService`、`management.DomainService`、`management.SSLService`、`delivery.FileService`、`delivery.StaticService`、`delivery.WebhookService`、`data.JSONRepository`、`data.StorageService`、`data.BackupService`、`system.AuthService`、`system.LogService`、`system.MonitoringService`、`system.Clock`、`system.IDGenerator` の公開 interface 境界を整備する。
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

目的: HTTPS JSON API、設定、起動時検証、JSON保存の共通契約を実装する。

### 実装タスク

- `config/config.json` の読み込み、デフォルト値適用、起動時バリデーションを実装する。
- `internal/config/config.go`、`internal/config/loader.go`、`internal/config/validate.go` を実装する。
- `internal/server/router.go`、`internal/server/response.go`、`internal/server/middleware.go` を実装する。
- `internal/data/json_repository.go` を実装する。
- `asb init-runtime` コマンドを実装する。
- `asb start` コマンドを実装し、起動時に不足ディレクトリ、JSON、証明書、ログ、一時ファイルを作成しないことを保証する。
- `server.port`、`server.host`、`server.tlsCertFile`、`server.tlsKeyFile`、`server.shutdownTimeout`、`storage.basePath`、`storage.maxProjectSize`、`ssl.email`、`ssl.renewBefore`、`ssl.renewCheckInterval`、`log.level`、`log.format`、`log.maxSize` の起動時バリデーションを実装する。
- 任意設定項目の未指定時、親 object 未指定時、空文字、型不一致、数値の小数・指数表記・負数・`null` 拒否を仕様通り実装する。
- `storage.basePath` と `deploy.sourcePath` の絶対パス正規化、開発リポジトリ配下判定、Git worktree 判定を実装する。
- 設定未知フィールド検出時の起動失敗を実装する。
- 起動時に設定済み実行時データ領域の存在確認と権限検証を実装する。
- 起動時に実行時データ用のディレクトリまたはファイルを自動生成しない。
- 開発リポジトリを実行時データ保存先として扱わない。
- `asb init-runtime` は `storage.basePath` が開発リポジトリ配下の場合に初期化を開始せず `ERR_STORAGE_VALIDATION_FAILED` で失敗する。
- `asb init-runtime` は作成予定パスがすべて `storage.basePath` 配下に収まることを検証する。
- `asb init-runtime` は `storage.basePath/`、`config/`、`storage/`、`storage/projects/`、`logs/`、`certs/` を仕様順序で作成または検証する。
- `asb init-runtime` は `config/projects.json`、`config/domains.json`、`config/backups.json`、`config/auth.json`、`config/webhooks.json`、`config/acme_accounts.json`、`config/acme_orders.json`、`config/acme_authorizations.json`、`config/acme_challenges.json`、`config/acme_renewals.json` を空状態で作成する。
- `asb init-runtime` は `storage/projects/:projectId/files.json` を作成しない。
- `asb init-runtime` は既存 JSON が妥当な場合は変更せず、既存 JSON が不正な場合は自動修復または上書きを行わない。
- `asb init-runtime` は既存ファイルまたはディレクトリを上書き、削除、移動、truncate しない。
- `asb init-runtime` は必須 JSON を同一ディレクトリ内の一時ファイル、`fsync`、atomic rename で作成する。
- `asb init-runtime` 成功時は stdout へ `ASB_RUNTIME_INITIALIZED storageBasePath="..."` を単一行出力し、終了コード `0` とする。
- `asb init-runtime` 失敗時は stderr へ `ASB_RUNTIME_INIT_ERROR code=... message="..."` を単一行出力し、終了コード `1` とする。
- `storage.basePath` 配下の `config/`、`storage/`、`storage/projects/`、`logs/`、`certs/` の順序付き存在検証を実装する。
- `config/projects.json`、`config/domains.json`、`config/backups.json`、`config/auth.json`、`config/webhooks.json`、`config/acme_accounts.json`、`config/acme_orders.json`、`config/acme_authorizations.json`、`config/acme_challenges.json`、`config/acme_renewals.json`、各 Project の `storage/projects/:projectId/files.json` の順序付き起動時存在検証を実装する。
- 必須 JSON ファイルの構文検証、必須フィールド検証、未知フィールド拒否を実装する。
- `server.tlsCertFile` と `server.tlsKeyFile` の絶対パス、存在、通常ファイル、秘密鍵ファイルの group/world writable 禁止を検証する。
- 管理 API 用 TLS 証明書ファイルまたは秘密鍵ファイルを開発リポジトリ内へ生成しない。
- 管理 API 用 TLS 証明書ファイルまたは秘密鍵ファイルを起動時に自動生成しない。
- Go標準 `net/http` によるHTTPSサーバーを実装する。
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
- `:id` は Rev.68 の UUID 正規表現に一致する値のみ許可する。
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
- Rev.68 時点では複数JSON更新に外部トランザクション機構を導入しない。
- Rev.68 時点の API エンドポイント固定表に記載されたメソッド、パス、成功ステータス、失敗コードを実装する。
- Rev.68 API 成功レスポンス固定表に記載された JSON キーと型を実装する。
- プロジェクト作成 API `POST /api/projects` を実装する。
- プロジェクト一覧 API `GET /api/projects` を実装する。
- プロジェクト削除 API `DELETE /api/projects/:id` を実装する。
- プロジェクト名、quota、ID、作成日時のバリデーションを実装する。
- `config/projects.json` によるプロジェクト情報の永続化を実装する。
- `config/projects.json` の `projects[]` スキーマ、`createdAt` 昇順、同一時刻時 `id` 昇順を実装する。
- Project 作成 API は `storage/projects/{projectId}/`、`storage/projects/{projectId}/contents/`、`storage/projects/{projectId}/files.json` を作成する。
- Project 作成 API は既存 Project ディレクトリまたは `files.json` が存在する場合、上書きせず `ERR_STORAGE_VALIDATION_FAILED` で失敗する。
- Project削除処理順序を Project検証、関連Domain列挙、関連Backup列挙、`files.json`検証、`contents/`削除、`files.json`削除、`domains.json`更新、`backups.json`更新、`projects.json`更新、成功応答の順に固定して実装する。
- Project削除を best effort 成功扱いにしない。
- プロジェクト名重複を `ERR_PROJECT_ALREADY_EXISTS` として扱う。
- 存在しないプロジェクト参照を `ERR_PROJECT_NOT_FOUND` として扱う。
- `internal/config/loader_test.go`、`internal/config/validate_test.go`、`internal/server/response_test.go`、`internal/data/json_repository_test.go` を作成する。

### 完了条件

- 起動時検証順序、終了コード、stderr 形式のテストが成功する。
- `asb init-runtime` の作成順序、空 JSON、stdout、終了コードのテストが成功する。
- `asb init-runtime` が既存妥当 JSON を変更しないテストが成功する。
- `asb init-runtime` が既存不正 JSON を修復または上書きしないテストが成功する。
- `asb init-runtime` が開発リポジトリ配下に実行時データを作成しないテストが成功する。
- `asb start` が不足ディレクトリまたは不足 JSON を自動生成せず起動失敗するテストが成功する。
- Project 作成 API が Project ディレクトリ、`contents/`、`files.json` を作成し、既存ファイルを上書きしないテストが成功する。
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
- 無料独自SSL
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
- upload API は `multipart/form-data` 以外を `400 Bad Request` とし、`ERR_INVALID_REQUEST` を返す。
- multipart アップロードのフィールド名を `file` に固定する。
- multipart upload の request body 最大サイズ 1GiB + 1MiB、file part 複数拒否、0 byte 拒否、1GiB超過時 `413` を実装する。
- upload file part の filename 必須、`/`、`\`、NUL、`.`、`..`、先頭 `.`, 空白のみ拒否を実装する。
- フォルダ階層を含む upload path は各 path segment に filename と同じ検証を適用する。
- File `path` は `contents/` で始まる相対パスのみ許可し、絶対パス、NUL、`\`、`.`、`..`、先頭 `.`, 空白のみセグメントを拒否する。
- ファイルアップロード処理順序を URL検証、Project検証、multipart検証、ファイル検証、既存JSON検証、一時ファイル書込、fsync、atomic rename、`files.json`更新、`projects.json`更新、成功応答の順に固定して実装する。
- 新規 upload で同一 `name` または同一 `path` が既に `files.json` に存在する場合は、同名ファイル上書き処理として扱う。
- 新規 upload で公開先ファイルが存在するが `files.json` に記録がない場合は、上書きせず `ERR_STORAGE_VALIDATION_FAILED` を返す。
- ファイル操作手段を HTTPS JSON API、`multipart/form-data` upload、GitHub Webhook デプロイに限定する。
- FTP、FTPS、SFTP を実装しない。
- FTP / FTPS / SFTP 用のユーザー、認証、接続管理、転送ログ、設定項目、JSONファイル、ディレクトリ、外部ライブラリを追加しない。
- ファイル削除処理順序を URL検証、Project検証、`files.json`検出、ファイル実体削除、`files.json`更新、`projects.json` used更新、成功応答の順に固定して実装する。
- 同名ファイル上書き時の使用容量再計算を実装する。
- 同名ファイル上書き時は旧メタデータ読み込み、旧ファイル実体検証、新ファイル一時書込、fsync、旧ファイル退避、新ファイル公開、`files.json`更新、`projects.json` used差分更新、旧ファイル退避削除の順で実装する。
- 旧ファイルは新ファイルの atomic rename 成功まで削除しない。
- File upload / overwrite 失敗時に成功レスポンスを返さず、仕様に従って一時ファイル、退避ファイル、公開済みファイルの削除または復元を行う。
- `projects.used` は通常時差分更新とし、不一致、負数、整合性検証時は `files[]` の `size` 合計から再計算する。
- `files.json` と `contents/` の不整合検出を実装し、未記録ファイル、実体欠落、通常ファイル以外、サイズ不一致、配信ルート外 path を失敗扱いにする。
- 不整合検出時、整合性検証は自動修復しない。
- 不整合検出時、File API の list、upload、overwrite、delete は成功レスポンスを返さない。
- Host ヘッダーから Domain を解決する。
- Domain から Project を解決する。
- URL path を静的ファイルパスへ変換する。
- `GET` と `HEAD` の静的配信を実装する。
- ディレクトリURLでは `index.html` を解決する。
- path traversal を拒否する。
- 隠しセグメントを拒否する。
- 配信ルート外参照を拒否する。
- Content-Type を Go標準ライブラリで判定する。
- 静的配信は `files.json` に記録された `path` のみを配信対象とし、未記録ファイルは実体が存在しても `404 Not Found` とする。
- `files.json` 記録済みファイルの実体欠落、通常ファイル以外、サイズ不一致、配信ルート外 path は `ERR_STORAGE_VALIDATION_FAILED` を error log へ記録し `500 Internal Server Error` とする。
- Gzip による静的ファイル圧縮を実装する。
- Gzip は `Accept-Encoding` に `gzip` token があり `q=0` でない `GET 200 OK` のみ対象にする。
- Gzip 圧縮済みレスポンスでは `Vary: Accept-Encoding` を返す。
- Gzip の不正な q 値は gzip 不許可として扱う。
- Gzip 圧縮では `.gz`、キャッシュ、メタデータを生成せず、圧縮開始後失敗時は接続終了と `ERR_INTERNAL` ログ記録を実装する。
- `HEAD` ではレスポンスボディを返さない。
- `ETag` を `W/"{size}-{unixModifiedTime}"` 形式で返す。
- `Last-Modified` を HTTP-date 形式で返す。
- `Cache-Control` を既定で `public, max-age=60` とする。
- `If-None-Match` と `If-Modified-Since` による `304 Not Modified` を実装し、両方が存在する場合は `If-None-Match` を優先する。
- `304 Not Modified` では `Content-Type`、`ETag`、`Last-Modified`、`Cache-Control` を返し、`Content-Encoding` を返さない。
- Range request は Rev.68 時点では実装せず、`Range` ヘッダーを無視して `206 Partial Content` を返さない。
- `Accept-Encoding: br` では Brotli 応答を返さない。
- Brotli 用の `.br`、キャッシュ、一時ファイル、メタデータを開発リポジトリ内にも `storage.basePath` 配下にも生成しない。
- 静的配信でディレクトリ一覧を返さない。
- 静的配信で開発リポジトリ内に配信用一時ファイル、キャッシュファイル、実行時データを作成しない。
- `internal/delivery/file_test.go`、`internal/delivery/static_test.go`、`internal/data/storage_test.go` を作成する。

### 完了条件

- File API の upload、list、delete、overwrite のテストが成功する。
- upload API の Content-Type、file field欠落、file field複数、filename、path segment、未記録公開先ファイル拒否のテストが成功する。
- `files.json` と `contents/` の不整合で File API が成功レスポンスを返さず、自動修復しないテストが成功する。
- 静的配信の Host 解決、`GET`、`HEAD`、`405`、`index.html` 解決、path traversal 拒否、隠しセグメント拒否、Content-Type、ETag、Last-Modified、Cache-Control、304、Range無視、Gzip のテストが成功する。
- 静的配信が `files.json` 未記録ファイルを配信せず、記録済みファイル不整合を `500` とするテストが成功する。
- `304 Not Modified` の `If-None-Match` 優先、`Content-Encoding` 不在、`HEAD` body不在、Gzip `Vary`、不正 q 値のテストが成功する。
- Brotli 用外部ライブラリ、middleware、precompress、`.br`、キャッシュ、設定項目、メタデータを生成しないテストが成功する。
- `go test ./...` が成功する。
- `.gitignore` が存在しない。
- 開発リポジトリ内に配信用一時ファイル、キャッシュ、実行時データ、ビルド成果物が残っていない。

### 非対象

- Brotli 圧縮
- Range request
- 外部CDN連携

## 6. P3 / v0.4 / ドメイン管理・SSL状態基盤

優先度: 高

目的: ドメイン管理と無料独自SSLの状態管理基盤を実装する。

### 実装タスク

- ドメイン追加 API `POST /api/projects/:id/domains` を実装する。
- `internal/management/domain.go`、`internal/management/ssl.go`、`internal/management/handler.go` を実装する。
- ドメイン一覧 API `GET /api/projects/:id/domains` を実装する。
- ドメイン削除 API `DELETE /api/projects/:id/domains/:domain` を実装する。
- `config/domains.json` によるドメイン情報の永続化を実装する。
- `config/domains.json` の `domains[]` スキーマと `domain` 昇順を実装する。
- RFC 1035 準拠のドメインバリデーションを実装する。
- ドメインの小文字正規化、253文字以下、最大3階層制限を実装する。
- Domain追加処理を Project存在検証、domain小文字正規化、domain検証、`domains.json` 検証、重複検証、Domain object追加、`domain` 昇順保存の順に固定して実装する。
- Domain object 追加時の `isCustom: true` と `sslCert: ""` を実装する。
- Domain一覧を対象Project所属のみ、`domain` 昇順に固定して実装する。
- ドメイン重複割り当てを `ERR_DOMAIN_ALREADY_ASSIGNED` として扱う。
- Domain削除処理を Project存在検証、Domain存在とProject所属検証、SSL状態確認、削除競合確認、Domain object削除、`domain` 昇順保存の順に固定して実装する。
- SSL状態が `pending`、`challenge_ready`、`renewing` の Domain削除は `409 Conflict` と `ERR_SSL_OPERATION_CONFLICT` を返す。
- Domain削除時に既存証明書ファイル、秘密鍵ファイル、ACME関連JSON、ACME challenge token を即時削除しない。
- 証明書保存先 `certs/` の存在確認と権限検証を実装する。
- SSL証明書ID、証明書メタデータ、証明書ファイルパス検証、有効期限監視モデル、失敗エラー `ERR_SSL_CERT_GENERATION_FAILED` を実装する。
- `config/domains.json` の Domain 要素に SSL状態参照を保存できる構造を実装する。
- 証明書本文と秘密鍵本文の対応確認、有効期限確認、秘密鍵権限検証を Go 標準ライブラリで実装する。
- SSL状態確認に必要な Entity、Service interface、Repository境界を実装する。
- P3 の SSL状態基盤を Domain と証明書メタデータの検証境界に限定する。
- 証明書ファイル保存先を `storage.basePath/certs/{domain}/fullchain.pem`、秘密鍵保存先を `storage.basePath/certs/{domain}/privkey.pem` に固定する。
- 証明書パスと秘密鍵パスが `storage.basePath` 外を参照しない検証を実装する。
- SSL状態が `issued`、`renewing`、`expired` の場合、証明書または秘密鍵の不在、読込不可、対応不一致、期限検証不可を `ERR_SSL_CERT_GENERATION_FAILED` として扱う。
- SSL状態が `disabled`、`pending`、`challenge_ready`、`failed` の場合、証明書ファイル不在だけを理由に失敗しない。
- このフェーズでは Let’s Encrypt との実通信を行わない。
- このフェーズでは証明書取得、証明書自動更新、ACME account 登録、order 作成、challenge 応答を実装しない。
- Domain API と SSL状態確認API が開発リポジトリ内に実行時データ、一時ファイル、ログファイルを作成しない実装にする。
- `internal/management/domain_test.go`、`internal/management/ssl_test.go` を作成する。

### 完了条件

- Domain API の追加、一覧、削除、重複、存在なし参照のテストが成功する。
- Domain追加処理順序、Domain一覧ソート、Domain削除時のSSL操作競合、証明書即時削除なしのテストが成功する。
- 証明書ファイルの存在、パス、有効期限、秘密鍵権限、証明書と秘密鍵の対応確認テストが成功する。
- SSL状態別の証明書不在許容条件と失敗条件のテストが成功する。
- Let’s Encrypt 実通信、ACME account 登録、order 作成、challenge 応答、証明書自動更新が実装されていないことを確認する。
- `go test ./...` が成功する。
- `.gitignore` が存在しない。

### 非対象

- 無料独自SSLの有効化 API
- 無料独自SSLの証明書取得
- 無料独自SSLの証明書自動更新
- Let’s Encrypt ACME v2 client
- HTTP-01 challenge 応答
- DNS-01 challenge
- TLS-ALPN-01 challenge
- wildcard 証明書
- DNS provider API 連携
- 複数 CA
- CA 選定
- CA failover

## 7. P4 / v0.5 / 無料独自SSL / Let’s Encrypt ACME v2

優先度: 高

目的: XServer Static互換目標の無料独自SSLを、Let’s Encrypt ACME v2 と HTTP-01 に限定して実装する。

### 実装タスク

- 無料独自SSLを実装する。
- 無料独自SSL 有効化 API `POST /api/projects/:id/domains/:domain/ssl/enable` を実装する。
- 無料独自SSL 状態確認 API `GET /api/projects/:id/domains/:domain/ssl` を実装する。
- 無料独自SSL 更新 API `POST /api/projects/:id/domains/:domain/ssl/renew` を実装する。
- 無料独自SSL 無効化 API `POST /api/projects/:id/domains/:domain/ssl/disable` を実装する。
- 無料独自SSL API の成功レスポンスを SSLStatus object に統一する。
- SSLStatus object の `domain`、`projectId`、`status`、`enabled`、`issuer`、`challenge`、`certificatePath`、`privateKeyPath`、`expiresAt`、`renewAfter`、`nextRetryAt`、`lastErrorCode`、`lastErrorMessage`、`updatedAt` を実装する。
- SSLStatus object に仕様外キーを含めない。
- SSLStatus の `issuer` は `LetsEncrypt`、`challenge` は `http-01` に固定する。
- SSLStatus の `status` ごとに、証明書パス、有効期限、更新予定、次回再試行時刻、失敗理由の空文字条件を仕様通りに実装する。
- `renewAfter` を `expiresAt - ssl.renewBefore` で算出する。
- 無料独自SSL 更新 API は対象 Domain が `disabled`、`pending`、`challenge_ready`、`renewing` の場合に `409 Conflict` と `ERR_SSL_OPERATION_CONFLICT` を返す。
- 無料独自SSLの有効化、無効化、状態確認、証明書取得、証明書自動更新を実装する。
- Domain 単位の無料独自SSL状態 `disabled`、`pending`、`challenge_ready`、`issued`、`renewing`、`failed`、`expired` を実装する。
- 有効化 API は対象 Domain が `disabled` または `failed` の場合のみ新規 ACME order を作成する。
- 有効化 API は対象 Domain が `pending`、`challenge_ready`、`issued`、`renewing` の場合に重複 ACME order を作成せず既存状態を返す。
- 無効化 API は無料独自SSL状態を `disabled` に変更し、既存証明書ファイルを即時削除しない。
- Let’s Encrypt ACME v2 client を実装する。
- ACME account 登録、account key 生成・保存、directory 取得、nonce 管理、order 作成、authorization 取得、HTTP-01 challenge 応答、finalize、certificate download を実装する。
- HTTP-01 challenge 応答を通常の静的ファイル配信より優先する。
- `/.well-known/acme-challenge/{token}` を ASB の challenge handler で応答する。
- HTTP-01 challenge handler は token が存在しない場合に通常の静的ファイル探索へ fallback せず `404 Not Found` を返す。
- ASB 前段にリバースプロキシを置く場合、`/.well-known/acme-challenge/` が ASB へ転送される構成を前提として検証する。
- HTTP-01 challenge token を `storage.basePath/acme/challenges/{domain}/{token}` に保存し、開発リポジトリ内に生成しない。
- HTTP-01 challenge token の内容を ACME key authorization 文字列のみにする。
- authorization が `valid`、`invalid`、`expired` になった場合に HTTP-01 challenge token を削除対象にする。
- 証明書ファイルを `storage.basePath/certs/{domain}/fullchain.pem` と `storage.basePath/certs/{domain}/privkey.pem` に保存する。
- 証明書本文と秘密鍵本文の対応確認、有効期限確認を Go 標準ライブラリ `crypto/x509`、`encoding/pem`、`crypto/tls` で実装する。
- ACME 内部状態を `storage.basePath` 配下の `config/acme_accounts.json`、`config/acme_orders.json`、`config/acme_authorizations.json`、`config/acme_challenges.json`、`config/acme_renewals.json` に保存する。
- 無料独自SSL有効化 API は ACME JSON ファイルを初回作成せず、既存ACME JSONの構文、`schemaVersion: 1`、必須トップレベルキー、未知フィールド不在を検証した後に更新する。
- `config/acme_accounts.json` の `schemaVersion`、`accounts` と account の `id`、`ca`、`directoryUrl`、`accountUrl`、`email`、`privateKeyPem`、`status`、`createdAt`、`updatedAt` を実装する。
- `config/acme_orders.json` の `schemaVersion`、`orders` と order の `id`、`domain`、`accountId`、`orderUrl`、`finalizeUrl`、`certificateUrl`、`status`、`expiresAt`、`createdAt`、`updatedAt` を実装する。
- `config/acme_authorizations.json` の `schemaVersion`、`authorizations` と authorization の `id`、`orderId`、`domain`、`authorizationUrl`、`status`、`expiresAt`、`createdAt`、`updatedAt` を実装する。
- `config/acme_challenges.json` の `schemaVersion`、`challenges` と challenge の `id`、`authorizationId`、`domain`、`type`、`challengeUrl`、`token`、`keyAuthorizationPath`、`status`、`createdAt`、`updatedAt` を実装する。
- `config/acme_renewals.json` の `schemaVersion`、`renewals` と renewal の `id`、`domain`、`kind`、`status`、`attemptCount`、`lastErrorCode`、`lastErrorMessage`、`nextRetryAt`、`notBefore`、`notAfter`、`createdAt`、`updatedAt` を実装する。
- 無料独自SSLの新規取得状態遷移を `disabled` / `failed` から `pending`、`challenge_ready`、`issued` または `failed` へ進める実装にする。
- 証明書更新状態遷移を `issued` から `renewing`、`challenge_ready`、`issued`、`expired` へ進める実装にする。
- 証明書自動更新スケジューラー、`ssl.renewBefore`、`ssl.renewCheckInterval`、retry / backoff、Let’s Encrypt rate limit 配慮を実装する。
- 証明書自動更新は、証明書有効期限の `ssl.renewBefore` 秒前から対象にする。
- `ssl.renewBefore` は `86400` 以上 `15552000` 以下の秒数整数のみ許可する。
- `ssl.renewCheckInterval` は `3600` 以上 `86400` 以下の秒数整数のみ許可する。
- retry / backoff は Domain 単位で管理し、`attemptCount` に基づき最小 `3600` 秒、最大 `86400` 秒の範囲で `nextRetryAt` を決定する。
- `nextRetryAt` より前に同一 Domain の自動更新を再実行しない。
- 複数 CA、CA 選定、CA failover、任意 ACME directory URL を実装しない。
- DNS-01 challenge、TLS-ALPN-01 challenge、wildcard 証明書、DNS provider API 連携、手動 TXT 登録、EAB、ARI、OCSP stapling を実装しない。
- `internal/management/acme_test.go`、`internal/management/ssl_acme_test.go` を作成する。

### 完了条件

- 無料独自SSLの有効化、無効化、状態確認、証明書取得、証明書自動更新のテストが成功する。
- 無料独自SSL 有効化、状態確認、更新、無効化 API が SSLStatus object を仕様通り返す。
- SSLStatus の状態別空文字条件、`enabled`、`issuer`、`challenge`、`renewAfter`、`nextRetryAt`、`lastErrorCode` が仕様通りである。
- 無料独自SSL 更新 API の競合状態で `409 Conflict` と `ERR_SSL_OPERATION_CONFLICT` が返る。
- Let’s Encrypt ACME v2 のHTTP-01 challengeフローをテスト用ACMEサーバーまたはモックで検証する。
- ACME account、order、authorization、challenge、renewal の JSON 保存スキーマが仕様通りである。
- 無料独自SSL有効化 API が ACME JSON ファイルを初回作成しないことを確認する。
- 新規取得、更新、失敗、backoff、期限切れの状態遷移が仕様通りである。
- `nextRetryAt` より前に自動更新が再実行されない。
- HTTP-01 challenge token が通常静的ファイル配信へ fallback しない。
- 既存有効証明書が更新失敗で削除されない。
- DNS-01、TLS-ALPN-01、wildcard、DNS provider API、複数CA、CA選定、CA failoverが実装されていないことを確認する。
- `go test ./...` が成功する。
- `.gitignore` が存在しない。

### 非対象

- DNS-01 challenge
- TLS-ALPN-01 challenge
- wildcard 証明書
- DNS provider API 連携
- 複数 CA
- CA 選定
- CA failover

## 8. P5 / v0.6 / GitHub Webhook デプロイ

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
- Webhook署名検証はJSON decode前のリクエストBody生バイト列に対して実行する。
- 署名なし、不正形式、不一致を `401 Unauthorized` と `ERR_WEBHOOK_SIGNATURE_INVALID` で拒否する。
- 署名検証失敗、payload形式不正、対象外event、対象外branchでは `config/webhooks.json` に冪等履歴を追加しない。
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
- Webhookデプロイ反映後のファイル集合から `files.json` を再生成する。
- Webhookデプロイで生成する File object の必須キーを `name`、`path`、`size`、`uploadedAt` に固定し、`path` 昇順で保存する。
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
- 署名検証前JSON decode禁止、対象外event/branchの冪等履歴非作成、`files.json` 再生成内容とソート順のテストが成功する。
- `go test ./...` が成功する。
- `.gitignore` が存在しない。

### 非対象

- GitHub API 連携
- ネットワーク越しのGit操作
- 自動リトライスケジューラー
- GitHub Actions 実行

## 9. P6 / v0.7 / バックアップ・復旧

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
- バックアップtar.gzに `config/projects.json`、`config/domains.json`、`config/webhooks.json`、`config/acme_*.json`、`config/migrations.json` を含めない。
- バックアップtar.gz内の symlink、hardlink、device、FIFO、socket、絶対パス、`..`、NUL、`\` を拒否する。
- Backup restore 展開時は各tarエントリを展開前に検証し、`next/` 配下外へ出るpathを拒否する。
- Backup restore 展開後に `next/files.json`、`next/contents/`、実ファイルと `files.json` の整合性を検証する。
- バックアップ作成用一時tarを `storage.basePath/backups/.tmp/` 配下に限定する。
- atomic rename 後に履歴保存へ失敗した場合は、作成済みtar.gzを削除する。
- 作成済みtar.gzの削除に失敗した場合でも、バックアップ作成APIは成功レスポンスを返さない。
- バックアップ保存先を別障害領域へ複製する作業をASB外の運用責務として扱う。
- 外部ストレージ連携を Rev.68 時点では実装対象外として扱う。
- Backup復旧前退避先を `storage.basePath/backups/restore-staging/{restoreId}/previous/` に固定する。
- Backup復旧用展開先を `storage.basePath/backups/restore-staging/{restoreId}/next/` に固定する。
- Backup履歴の `status` が `completed` でない場合は復旧を拒否する。
- 復旧対象Projectが存在しない場合は `ERR_PROJECT_NOT_FOUND` を返す。
- 同一 `restoreId` の `restore-staging/{restoreId}/` が既に存在する場合は `409 Conflict` と `ERR_BACKUP_RESTORE_CONFLICT` を返す。
- 復旧対象Projectの現行データ退避に失敗した場合は、復旧処理を開始しない。
- 復旧途中失敗時は `previous/` から復元する。
- 復旧処理では `files.json` と `contents/` のみを置換し、Project定義、Domain定義、SSL証明書、ACME状態、Webhook履歴、ログを置換しない。
- `restore-staging/{restoreId}/` 削除失敗は WARN ログに記録する。
- 開発リポジトリ内にバックアップ、一時tar、checksum、復旧用一時ファイル、退避データ、展開データを作成しない。
- `internal/data/backup_test.go` を作成する。

### 完了条件

- Backup作成、Backup復旧、途中失敗、復元失敗、履歴保存失敗のテストが成功する。
- Project不在、restore staging競合、バックアップ対象外ファイル混入禁止、復旧時の非対象データ非置換のテストが成功する。
- バックアップtar.gz構成とSHA-256検証テストが成功する。
- 外部ストレージ連携が存在しないことを確認する。
- 開発リポジトリ内にバックアップ関連生成物が残っていない。
- `go test ./...` が成功する。
- `.gitignore` が存在しない。

### 非対象

- 外部ストレージ連携
- 別障害領域への自動複製
- ログファイルと証明書ファイルのバックアップ

## 10. P7 / v0.8 / ログ・監視

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
- `storage.basePath/logs/` と現行ログファイルは、LogService の書き込み処理または起動時検証で明示的に必要な場合のみ作成する。
- ログAPI、監視API、静的配信、File API、Domain API、SSL状態確認API、Backup一覧APIではログファイルを初回作成しない。
- アクセスログ書き込みはレスポンスステータス確定後、レスポンス送信前に実行する。
- 成功レスポンス送信前にアクセスログ書き込みまたはローテーションへ失敗した場合は `500 Internal Server Error` と `ERR_LOG_WRITE_FAILED` を返す。
- エラーレスポンス生成中の error log 書き込み失敗時は、内部パスや詳細原因をレスポンス本文に含めない。
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
- 監視APIは既存の実行時JSON、プロセス情報、OS情報、`storage.basePath` の状態から値を計算する。
- 監視APIではメトリクス保存用JSON、キャッシュファイル、一時ファイルを作成しない。
- 開発リポジトリ内にログファイルを作成しない。
- `internal/system/log_test.go`、`internal/system/monitoring_test.go` を作成する。

### 完了条件

- アクセスログとエラーログのJSON Linesフィールド、フィールド順、requestId一致、stdout非出力、stderr起動失敗出力、ローテーションのテストが成功する。
- ログ API の新しい順、壊れたJSON行検出、現行ログのみ対象のテストが成功する。
- Monitoring API の成功レスポンスと取得不可項目 `null` のテストが成功する。
- ログAPI/監視APIがログファイル、メトリクスJSON、キャッシュ、一時ファイルを作成しないテストが成功する。
- 成功レスポンス送信前のアクセスログ書き込み失敗が `ERR_LOG_WRITE_FAILED` になるテストが成功する。
- 開発リポジトリ内にログファイルが残っていない。
- `go test ./...` が成功する。
- `.gitignore` が存在しない。

### 非対象

- ログファイル暗号化
- 外部監視サービス連携
- ローテーション済みログAPI

## 11. P8 / v0.9 / マイグレーション

優先度: 中

目的: 実行時 JSON ファイルのスキーマ移行を、明示コマンドのみで実装する。

### 実装タスク

- マイグレーション対象を `config/projects.json`、`config/domains.json`、`config/backups.json`、`config/auth.json`、`config/webhooks.json`、`config/acme_accounts.json`、`config/acme_orders.json`、`config/acme_authorizations.json`、`config/acme_challenges.json`、`config/acme_renewals.json`、`storage/projects/:projectId/files.json` に限定する。
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
- `config/migrations.json` は起動時必須 JSON ファイルとして扱わない。
- `config/migrations.json` は `asb init-runtime` の初期作成対象に含めない。
- `config/migrations.json` は `asb migrate --apply` 実行時のみ初回作成し、起動時、API、静的配信、Webhook、Backup、Log API、SSL 管理境界では作成しない。
- `config/migrations.json` 作成または更新失敗時はマイグレーション成功扱いにしない。
- マイグレーション失敗時の事前バックアップからのロールバックを実装する。
- ロールバック失敗時に標準エラー、エラーログ、`config/migrations.json` へ `failed` として記録する。
- 外部DBマイグレーション、外部トランザクション機構、外部マイグレーションフレームワークを導入しない。

### 完了条件

- `schemaVersion` なしを `0` として扱うテストが成功する。
- 未対応 `schemaVersion` の起動失敗テストが成功する。
- `--dry-run` が実行時 JSON ファイルを変更しないテストが成功する。
- `--apply` が対象 JSON ファイルを atomic rename で更新するテストが成功する。
- `config/migrations.json` が `asb init-runtime` で作成されないテストが成功する。
- `config/migrations.json` が `asb migrate --apply` 実行時のみ初回作成されるテストが成功する。
- 事前バックアップ作成、途中失敗時の成功禁止、ロールバック成功、ロールバック失敗時の `failed` 記録テストが成功する。
- 開発リポジトリ内に移行作業ファイルを作成しないテストが成功する。
- `go test ./...` が成功する。
- `.gitignore` が存在しない。

### 非対象

- 起動時の自動マイグレーション
- 外部DBマイグレーション
- 外部マイグレーションフレームワーク

## 12. P9 / v0.10 / 配布・install/update

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

## 13. P10 / v0.11 / ASB SDK

優先度: 低

目的: ASB 管理 HTTPS JSON API を呼び出すための、ASB SDK とその Browser JavaScript、Deno専用 TypeScript、Go 対応実装を実装する。

### 実装タスク

- ASB SDK を単一の公式SDKとして実装する。
- ASB SDK を別製品名または別プロジェクト名として分割しない。
- ASB SDK を ASB 管理 HTTPS JSON API の型付き呼び出し層として実装し、ASB 本体拡張機構として扱わない。
- ASB SDK の Browser JavaScript 実装をブラウザ専用 JavaScript 実装として実装する。
- ASB SDK の Browser JavaScript 実装の標準ファイルを `webui/asb-sdk.js` として実装する。
- ASB SDK の Browser JavaScript 実装をブラウザ標準 ES module として読み込める形式にする。
- ASB SDK の Browser JavaScript 実装は named export で公開APIを提供し、global object へ自動登録しない。
- ASB SDK の Browser JavaScript 実装は package 名を持たない構成にする。
- ASB SDK の Browser JavaScript 実装は Web 標準 API の `fetch`、`URL`、`URLSearchParams`、`Headers`、`FormData`、`Blob`、`AbortController`、`Promise` の範囲で実装する。
- ASB SDK の Deno専用 TypeScript 実装を Deno 専用ランタイムの TypeScript module として実装する。
- ASB SDK の Deno専用 TypeScript 実装の標準ファイルを `sdk/deno/asb-sdk.ts` として実装する。
- ASB SDK の Deno専用 TypeScript 実装は Deno runtime API と Web 標準 API の範囲で実装する。
- ASB SDK の Deno専用 TypeScript 実装は Node.js、npm、package manager、`package.json`、`node_modules/`、`deno.json`、`deno.lock`、bundler、transpiler、generated client、外部ライブラリを前提にしない。
- ASB SDK の Go 実装を Go 1.21 以上の ASB 管理 HTTPS JSON API クライアント実装として実装する。
- ASB SDK の Go 実装を `sdk/go/` 配下に配置し、package 名を `asb` とする。
- ASB SDK の Go 実装で `go.mod` を作成する場合、module path を `github.com/fqwink/Adlaire-Static-Base/sdk/go` に固定する。
- ASB SDK の Go 実装の tag を ASB 本体の安定版リリースタグと同一にする。
- ASB SDK の Go 実装を Rev.68 時点では外部配布サービスへ登録しない。
- ASB SDK の Go 実装は `net/http`、`net/url`、`encoding/json`、`context`、`time`、`mime/multipart` を中心に Go標準ライブラリで実装する。
- ASB SDK の Go 実装は ASB 本体の `internal/` package を import しない。
- ASB SDK の Go 実装は外部HTTP client library、外部JSON library、generated client を前提にしない。
- ASB SDK の Go 実装で `go.mod` を作成する場合でも外部 module dependency を追加しない。
- ASB SDK の各対応実装は ASB 管理 HTTPS JSON API の method、path、query parameter、path parameter、request JSON、multipart upload、success response JSON、error response JSON、HTTP status code、error code、pagination、UTC RFC3339 timestamp を扱う。
- ASB SDK の各対応実装は `baseUrl`、timeout、request cancellation、JSON request、JSON response、multipart upload、error response の扱いを提供する。
- ASB SDK の各対応実装は `baseUrl` に `https://` scheme の URL のみを許可する。
- ASB SDK の各対応実装は `http://`、相対URL、空文字、schemeなしURL、WebSocket URL、独自schemeを `baseUrl` として拒否する。
- ASB SDK の各対応実装は `baseUrl` 末尾の `/` の有無に依存せず、ASB 管理 API path を単一の `/` で結合する。
- ASB SDK の各対応実装は、管理API呼び出し時に単一システム管理者パスワードを request 単位の引数または呼び出しオプションとして受け取る。
- ASB SDK の各対応実装は、管理API呼び出し時に `X-ASB-Admin-Password` ヘッダーを設定する。
- ASB SDK の各対応実装は、管理者パスワードをファイル、環境変数、ブラウザストレージ、cookie、セッション、global変数、cache、SDK専用実行時データへ保存しない。
- ASB SDK の公開APIを ASB 管理 HTTPS JSON API の endpoint 単位に対応する関数または method として実装する。
- ASB SDK の公開API名、引数、戻り値を ASB 管理 HTTPS JSON API の method、path、request、response、error に対応させる。
- ASB SDK の各対応実装の自動 retry 回数は `0` とし、失敗した HTTP request を自動再送しない。
- ASB SDK の各対応実装は ASB 本体の内部 JSON、Service、Repository、Storage を直接参照または呼び出さない。
- ASB SDK は成功レスポンスJSONとエラーレスポンスJSONを仕様外キー追加なしで返す。
- ASB SDK は通信エラー、timeout、JSON decode失敗、ASB error response を区別できる error 型または error object を提供する。
- ASB SDK は ASB 本体が返した `code`、`message`、`requestId` を破棄、改名、翻訳しない。
- SDK 専用 HTTPS API、SDK 専用 URL prefix、SDK 専用 request body、SDK 専用 response body、SDK 専用 error format、SDK 専用 pagination、SDK 専用 upload protocol を作らない。
- SDK 専用 session、token、cookie、handshake、protocol negotiation、version negotiation を作らない。
- SDK 専用 JSON ファイル、SDK 専用ディレクトリ、SDK 専用設定項目、SDK 専用実行時データを作らない。
- ASB SDK の Browser JavaScript 実装はブラウザストレージ、cookie、Service Worker、Cache Storage、IndexedDB を SDK 通信用の永続状態として使用しない。
- `package.json`、`node_modules/`、`deno.json`、`deno.lock`、`dist/`、`build/` を SDK 実装理由で追加しない。
- SDK のユニットテストまたはレビューで、Node.js、npm、bundler、外部ライブラリへの依存がないことを確認する。

### 完了条件

- ASB SDK の Browser JavaScript、Deno専用 TypeScript、Go 対応実装が実装されている。
- ASB SDK の各対応実装が ASB 管理 HTTPS JSON API と同一規格で通信する。
- `baseUrl` の HTTPS 限定、不正scheme拒否、path結合のテストが成功する。
- SDK公開APIが ASB 管理 HTTPS JSON API endpoint と対応していることを確認する。
- SDK error 型または error object が通信エラー、timeout、JSON decode失敗、ASB error response を区別できる。
- SDK が ASB error response の `code`、`message`、`requestId` を保持する。
- SDK が `X-ASB-Admin-Password` を request 単位で送信し、管理者パスワードを永続保存しない。
- SDK 専用 API、SDK 専用保存データ、SDK 専用実行時データが存在しない。
- ASB SDK の Browser JavaScript 実装と Deno専用 TypeScript 実装が Node.js、npm、package manager、bundler、外部ライブラリを前提にしていない。
- ASB SDK の Deno専用 TypeScript 実装が Deno 専用である。
- ASB SDK の Go 実装が ASB 本体の `internal/` package を import していない。
- ASB SDK の Go 実装の module path と tag 方針が仕様通りである。
- ASB SDK の Browser JavaScript 実装が global object へ自動登録されていない。
- ASB SDK の Go 実装が外部 module dependency を追加していない。
- ASB SDK が SDK認証拡張用の login、logout、refreshToken、session、API key、credential store API を公開していない。
- `.gitignore` が存在しない。
- 開発リポジトリ内に実行時データ、ログ、一時ファイル、ビルド成果物が残っていない。
- `go test ./...` が成功する。

### 非対象

- ASB Web UI 本体の実装
- Node.js 対応
- npm 配布
- Deno 以外の TypeScript runtime 対応
- bundler 前提の配布
- SDK 認証拡張仕様
- デスクトップアプリ向け SDK 対応
- モバイルアプリ向け SDK 対応

## 14. P11 / v0.12 / ASB 標準Web UI

優先度: 低

目的: ASB 本体外の内製標準管理画面クライアントとして、ブラウザ標準 HTML / CSS / JavaScript の ASB 標準Web UI を実装する。

### 実装タスク

- ASB 標準Web UI を `webui/` 配下に配置する。
- ASB 標準Web UI の入口ファイルを `webui/index.html` とする。
- ASB 標準Web UI のスタイルを `webui/styles.css` に配置する。
- ASB 標準Web UI の画面制御を `webui/app.js` に配置する。
- ASB SDK Browser JavaScript 実装を `webui/asb-sdk.js` として同梱する。
- ASB 標準Web UI を静的 HTML / CSS / JavaScript アプリケーションとして実装する。
- ASB 標準Web UI は build step を持たず、`webui/` 配下の静的ファイルをそのまま配布可能にする。
- ASB 標準Web UI は ASB 本体に内包しない。
- ASB 標準Web UI は ASB 本体バイナリへ埋め込まない。
- ASB 本体は `webui/` を起動時に読み込まない。
- ASB 本体は `webui/` を静的配信対象として自動公開しない。
- ASB 標準Web UI は ASB 本体の release artifact とは別 artifact として配布する。
- ASB 標準Web UI の配布 artifact は静的 HTML / CSS / JavaScript と ASB SDK の Browser JavaScript 実装を含むファイル集合にする。
- ASB 標準Web UI はブラウザ標準 API と ASB SDK の Browser JavaScript 実装のみを使用する。
- ASB 標準Web UI は `asb-sdk.js` をブラウザ標準 ES module として読み込む。
- ASB 標準Web UI は ASB SDK の Browser JavaScript 実装経由で ASB 管理 HTTPS JSON API のみを呼び出す。
- ASB 標準Web UI は ASB SDK を経由せずに `fetch` または `XMLHttpRequest` で ASB 管理 API を直接呼び出さない。
- ASB 標準Web UI は ASB 本体の内部 JSON、Service、Repository、Storage を直接参照または呼び出さない。
- ASB 標準Web UI は ASB SDK に存在しない操作を UI 操作として提供しない。
- ASB 標準Web UI は初期表示時に `baseUrl` を利用者入力または静的設定値として扱い、永続保存しない。
- ASB 標準Web UI は ASB 管理 HTTPS JSON API が返した `requestId` をエラー表示または詳細表示で確認可能にする。
- Dashboard 画面を実装し、システム状態、プロジェクト数、ドメイン数、SSL状態、デプロイ状態、ストレージ使用量、直近ログを表示する。
- Projects 画面を実装し、Project の一覧、作成、詳細確認、削除を行う。
- Domains 画面を実装し、Domain の一覧、追加、Project割当、解除、削除を行う。
- SSL 画面を実装し、無料独自SSLの有効化、無効化、状態確認、更新を行う。
- Files 画面を実装し、Project単位のファイル一覧、アップロード、削除を行う。
- Deployments 画面を実装し、GitHub Webhookデプロイ結果、重複判定、失敗理由を表示する。
- Backups 画面を実装し、Backup の作成、一覧、検証、復旧を行う。
- Logs 画面を実装し、Access log と Error log を表示する。
- Settings 画面を実装し、ASB の読み取り専用設定値、実行時状態、システム管理者パスワード変更を扱う。
- システム管理画面へのアクセスまたは管理操作のたびに、システム管理者パスワード入力を要求する。
- 入力されたシステム管理者パスワードを ASB SDK の Browser JavaScript 実装へ request 単位で渡す。
- システム管理者パスワードをメモリ上で request 完了までの一時値としてのみ扱い、request 完了後に参照を破棄する。
- `ERR_AUTH_PASSWORD_CHANGE_REQUIRED` を受け取った場合、管理者パスワード変更画面または変更フォームを表示する。
- Node.js 実行環境、npm 配布、package manager、bundler、transpiler、外部フレームワーク、外部ライブラリを前提にしない。
- ユーザー管理 UI、ロール管理 UI、テナント管理 UI、課金 UI、契約管理 UI、FTP / FTPS / SFTP UI を実装しない。
- ブラウザストレージ、cookie、Service Worker、Cache Storage、IndexedDB を永続状態として使用しない。
- `package.json`、`node_modules/`、`deno.json`、`deno.lock`、`dist/`、`build/`、一時ファイル、ログファイル、ビルド成果物を生成しない。
- 外部ネットワークから利用可能にする場合は、VPN、SSH tunnel、reverse proxy、ファイアウォール、IP制限等のASB外部の運用境界で保護する。
- 外部開発者の独自Web UIまたは独自フロントエンドは、公式 ASB 標準Web UI の実装タスクとして扱わない。

### 完了条件

- Dashboard、Projects、Domains、SSL、Files、Deployments、Backups、Logs、Settings の各画面が実装されている。
- 全画面が ASB SDK の Browser JavaScript 実装経由で ASB 管理 HTTPS JSON API のみを呼び出す。
- Web UI が `fetch` または `XMLHttpRequest` で ASB 管理 API を直接呼び出していないことを確認する。
- Web UI が ASB SDK に存在しない操作を提供していないことを確認する。
- Web UI の `baseUrl` が永続保存されないことを確認する。
- Web UI がシステム管理者パスワードを毎回入力として扱い、ブラウザストレージ、cookie、IndexedDB、Cache Storage、Service Worker、URL、ログ、画面表示へ保存または出力しないことを確認する。
- Web UI が `ERR_AUTH_PASSWORD_CHANGE_REQUIRED` を受け取った場合に管理者パスワード変更を実行できる。
- Web UI がエラー時に `requestId` を確認可能にする。
- ASB 標準Web UI が `webui/` 配下に配置されている。
- `webui/index.html`、`webui/styles.css`、`webui/app.js`、`webui/asb-sdk.js` が存在する。
- ASB 標準Web UI が ASB 本体 release artifact とは別 artifact として配布できる。
- ASB 本体に Web UI 画面、テンプレート、フロントエンドビルド、Web UI 専用保存 JSON、Web UI 専用実行時データが追加されていない。
- Web UI が build step を持たず、`webui/` 配下の静的ファイルだけで配布できる。
- Node.js、npm、bundler、外部フレームワーク、外部ライブラリを前提にしていない。
- `.gitignore` が存在しない。
- 開発リポジトリ内に実行時データ、ログ、一時ファイル、ビルド成果物が残っていない。
- `go test ./...` が成功する。

### 非対象

- ASB 本体への Web UI 内包
- Web UI 専用 HTTPS API
- Web UI 専用保存 JSON
- Web UI 専用実行時データ
- Node.js 対応
- npm 配布
- ASB 標準Web UI 本体を TypeScript で実装すること
- bundler 前提の配布
- 外部フレームワーク採用
- 外部ライブラリ採用
- 外部開発者の独自Web UIまたは独自フロントエンドの実装

## 15. P12 / v0.13 / 禁止機能・非実装確認

優先度: 低

目的: Rev.68 時点で実装対象外の機能が混入していないことを確認する。

### 実装タスク

- 未昇格のASB互換目標を将来の到達目標として扱い、Rev.68 時点の実装対象として扱わない。
- `internal/asb_forbidden_test.go` を作成する。
- XServer Static互換機能セットの実装対象が、静的配信、独自ドメイン、無料独自SSL、GitHub Webhookデプロイ、HTTPS JSON APIによるファイル管理、ログ・状態確認、バックアップ・復旧に限定されていることを確認する。
- XServer Static互換機能セットを理由に、XServer Static完全互換、管理画面再現、内部実装再現、DNS管理、DNS provider API、DNS-01、wildcard、複数CA、CDN完全互換、課金・契約・アカウント管理を追加しない。
- ASB互換目標に含まれることを、未昇格機能の実装根拠として扱わない。
- ASB互換目標を理由に `.gitignore`、外部DB、未承認外部ライブラリ、未承認外部サービス連携、開発リポジトリ内実行時データ、起動時自動生成、ビルド成果物自動生成を追加しない。
- ASB互換目標を理由に APIキー管理、複数ユーザー管理、Rate limiting、Brotli圧縮、HTTP/2、CA選定、SDK専用通信を実装しない。
- HTTP/2 が暗黙的に有効化されないよう、Rev.68 の実装では `http.Server.TLSNextProto` を空 map に設定する。
- HTTP/2 専用設定項目、h2c、ALPN独自制御、server push、stream priority、専用handler、専用middleware、専用ログ項目を実装しない。
- 将来計画、保留事項、検討・調査中事項を Rev.68 時点の実装対象として扱わない。
- GUIという曖昧カテゴリ、ASB本体へのWeb UI内包、デスクトップアプリ、モバイルアプリ、複数ユーザー管理、ユーザー別権限管理、マルチテナント、課金管理、契約管理、複数インスタンス管理、クラスタ管理、分散ロック、NFS専用連携、分散ストレージ専用連携、外部ストレージサービス連携、ログファイル暗号化、HTTP/2実装詳細、FTP、FTPS、SFTPをASB本体に実装しない。
- 将来計画機能または転送プロトコル互換を理由に ASB本体内包Web UI用API、モバイル専用API、テナント用API、課金用API、契約用API、外部ストレージ用API、ログ暗号化用API、FTP / FTPS / SFTP 用 APIを追加しない。
- 将来計画機能または転送プロトコル互換を理由に `ui.*`、`webui.*`、`desktop.*`、`mobile.*`、`tenant.*`、`billing.*`、`nfs.*`、`cluster.*`、`distributedStorage.*`、`externalStorage.*`、`logEncryption.*`、`ftp.*`、`ftps.*`、`sftp.*` 設定項目を追加しない。
- 将来計画機能または転送プロトコル互換を理由に ASB本体内包Web UI用JSON、モバイル用JSON、テナント用JSON、課金用JSON、外部ストレージ用JSON、ログ暗号化用JSON、FTP / FTPS / SFTP 用 JSONを追加しない。
- 将来計画機能または転送プロトコル互換を理由に ASB本体内包Web UI用ディレクトリ、モバイル用ディレクトリ、テナント用ディレクトリ、課金用ディレクトリ、外部ストレージ用ディレクトリ、ログ暗号化用ディレクトリ、FTP / FTPS / SFTP 用ディレクトリを追加しない。
- 複数インスタンス対応を理由に node 管理、leader election、distributed lock、cluster membership、node heartbeat、shared queue を実装しない。
- NFS 連携を理由に NFS mount 管理、NFS lock 制御、NFS stale handle 検出、NFS 専用 retry、NFS 専用 health check を実装しない。
- 分散ストレージ連携を理由に distributed storage driver、replica 管理、shard 管理、quorum 制御、repair job、rebalancing を実装しない。
- 外部ストレージ連携を理由に S3、S3互換API、GCS、Azure Blob、Dropbox、Google Drive、Box、presigned URL、外部ストレージ credential 管理、外部SDK を実装しない。
- ログファイル暗号化を理由にログ保存時暗号化、ログ復号 API、key 生成、key 保存、key rotation、KMS連携、暗号化ログ viewer を実装しない。
- デスクトップアプリ向けSDK利用を理由に専用API、専用認証、専用token、callback URL、deep link、OS keychain、auto update、installer、desktop notification、tray integration、native menu、GUIライブラリ依存を追加しない。
- モバイルアプリ向けSDK利用を理由に専用API、専用認証、専用token、device registration、push notification、biometric authentication、mobile deep link、offline cache、sync queue、app store 配布設定、モバイルGUIライブラリ依存を追加しない。
- 旧プロジェクト仕様の吸収を理由に、旧プロジェクト専用 API、設定、JSON、ディレクトリ、SDK、Web UI、CLI、互換モード、migration、import、export、plugin、adapter、bridge、protocol、runtime を追加しない。
- 外部公開名、API名、設定名、JSON名、ディレクトリ名、package 名、SDK名、Web UI名、CLI subcommand 名に `Auteur` または `auteur` を使用しない。
- Auteur リポジトリ `https://github.com/fqwink/Auteur` の `Auteur_Master_Specification.md` は仕様移管元としてのみ扱い、source code、runtime、CLI、fixture、test、CI、release automation、package、lock file、設定ファイル、生成物を ASB へ移管しない。
- Auteur リポジトリ内の `.gitignore`、`deno.json`、TypeScript 実装、fixture、test が ASB の仕様、実装、生成物、依存関係、開発手順としてコピーされていないことを確認する。
- `auteur.config.json`、`.auteur/`、`auteur-project/`、`src/pages/**/*.astro`、`src/pages/api/**/*.go`、`ui/`、`content/`、`dist/`、`.env`、`deno.json`、`deno.lock`、`AUTEUR_*` error code、Auteur 固有 hydration directive、Auteur 固有 component syntax が ASB の有効仕様として追加されていないことを確認する。
- Content Pipeline、Site Routing、Site Rendering、Site Output、Blog、Docs、Sitemap、Ad Slot、Asset Pipeline、Source Sync、External Data Integration、Runtime Cache、Database Gateway、Database Adapter が Rev.68 時点の実装対象へ昇格していないことを確認する。
- 管理 API が `Authorization` ヘッダーまたは `X-API-Key` ヘッダーの有無でレスポンスを変えないことをテストする。
- APIキー、複数ユーザー、ロール、セッションを表す JSON ファイルまたはディレクトリを生成しないことをテストする。
- `config/config.json` に認証関連フィールドが存在する場合に未知フィールドとして起動失敗することをテストする。
- Rate limiting 用 middleware、制限アルゴリズム、永続カウンタ、設定項目、JSON ファイルまたはディレクトリを生成しないことをテストする。
- 管理 API、静的配信、Webhook 受信が Rate limiting 関連条件でレスポンスを変えないことをテストする。
- `config/config.json` に Rate limiting 関連フィールドが存在する場合に未知フィールドとして起動失敗することをテストする。
- Rate limiting を理由に `429 Too Many Requests`、`Retry-After`、Rate limiting 用 error code、監査イベント、メトリクスが返却または生成されないことをテストする。
- Rate limiting を理由に管理 API、静的配信、Webhook 受信、ACME HTTP-01 challenge 応答、ヘルスチェックのルーティング順序が変わらないことを確認する。
- HTTP/2 が無効化され、HTTP/2 関連設定項目が未知フィールドとして起動失敗することをテストする。
- SDK 通信が ASB 管理 HTTPS JSON API と同一規格であることをテストする。
- ASB 標準Web UI が ASB SDK の Browser JavaScript 実装を経由して ASB 管理 HTTPS JSON API と通信する設計になっていることを確認する。
- ASB 標準Web UI が ASB 本体の内部 JSON、Service、Repository、Storage を直接参照しないことを確認する。
- ASB 本体に Web UI 画面、Web UI テンプレート、Web UI フロントエンドビルド、Web UI 専用保存 JSON、Web UI 専用実行時データが追加されていないことを確認する。
- SDK 専用プロトコル、SDK 専用エンドポイント、SDK 専用セッション、SDK 専用 JSON ファイルまたはディレクトリを生成しないことをテストする。
- `config/config.json` に SDK 通信関連フィールドが存在する場合に未知フィールドとして起動失敗することをテストする。
- SDK認証拡張用の token、session、client registration、credential cache、scope、secret を表す JSON ファイル、ディレクトリ、設定項目、実行時データが生成されないことをテストする。
- ASB SDK が単一システム管理者パスワード以外の認証入力を保持、更新、ローテーション、永続化しないことを確認する。
- `config/config.json` に SDK認証拡張関連フィールドが存在する場合に未知フィールドとして起動失敗することをテストする。
- 複数インスタンス、NFS、分散ストレージ、外部ストレージ、ログ暗号化、デスクトップアプリ、モバイルアプリ用の設定項目、JSON ファイル、ディレクトリ、実行時データが生成されないことをテストする。
- `config/config.json` に `cluster.*`、`node.*`、`lock.*`、`nfs.*`、`distributedStorage.*`、`externalStorage.*`、`logEncryption.*`、`desktop.*`、`desktopSdk.*`、`mobile.*`、`mobileSdk.*` 相当の関連フィールドが存在する場合に未知フィールドとして起動失敗することをテストする。
- `config/auteur.json`、`config/auteur_compat.json`、`config/auteur_migration.json`、`storage/auteur/`、`storage/auteur_compat/`、`storage/auteur_migration/`、`auteur.*`、`auteurCompat.*`、`auteurMigration.*` が追加または生成されないことをテストする。
- 旧プロジェクト名が実装入力元、互換対象、migration 元、独立サブシステム名、ASB の有効な通常機能名として使用されていないことをレビューで確認する。ただし、`ASB-spec.md` で固定した仕様移管元の記録と変更履歴は除く。
- 将来計画機能を理由に未昇格 API、設定項目、JSONファイル、ディレクトリ、外部依存が追加されていないことをテストまたはレビューで確認する。
- 将来計画、保留事項、検討・調査中事項が個別確定仕様なしに実装対象へ昇格していないことを確認する。
- XServer Static互換機能セット外の機能が、個別確定仕様なしに実装対象へ昇格していないことを確認する。

### 完了条件

- 禁止機能の非生成テストまたはレビューが完了する。
- 複数ユーザー管理、APIキー管理、セッション管理、Rate limiting、HTTP/2、SDK専用通信、Brotli、ASB本体へのWeb UI内包、将来計画機能が実装されていないことを確認する。
- `.gitignore` が存在しない。
- 開発リポジトリ内に実行時データ、ログ、一時ファイル、ビルド成果物が残っていない。
- `go test ./...` が成功する。

### 非対象

- 禁止機能の実装
- 未確定タスクの実装
- 将来計画機能の仕様昇格

## 16. P13 / v0.14 / 単一システム管理者認証

優先度: 高

目的: ASB 管理 HTTPS JSON API に、単一システム管理者パスワード認証を実装する。

### 実装タスク

- `internal/system/auth.go` を作成し、単一システム管理者認証、パスワード検証、パスワード変更、ハッシュ処理を実装する。
- `internal/system/auth_test.go` を作成する。
- `config/auth.json` の `schemaVersion`、`admin.passwordHash`、`admin.passwordSalt`、`admin.passwordChanged`、`admin.updatedAt` を実装する。
- `config/auth.json` の空状態を `{"schemaVersion":1,"admin":{"passwordHash":"","passwordSalt":"","passwordChanged":false,"updatedAt":""}}` に固定する。
- 初期デフォルトパスワードを `asb-admin-change-me` として扱う。
- 初期デフォルトパスワードが未変更の場合、`POST /api/auth/change-password` 以外の管理 API を `403 Forbidden`、`ERR_AUTH_PASSWORD_CHANGE_REQUIRED` で拒否する。
- `POST /api/auth/change-password` を実装する。
- 管理者パスワード変更 API は `currentPassword` と `newPassword` を受け取り、現在パスワード照合後に `config/auth.json` を atomic rename で更新する。
- `newPassword` は12文字以上128文字以下、初期デフォルトパスワード不一致、NUL文字および制御文字なし、先頭末尾空白なしを検証する。
- `POST /api/auth/change-password` を除く管理 API に `X-ASB-Admin-Password` ヘッダー検証 middleware を適用する。
- `X-ASB-Admin-Password` が未指定、空文字、不一致の場合は `401 Unauthorized`、`ERR_AUTH_FAILED` を返す。
- `POST /api/webhook/github` には `X-ASB-Admin-Password` を要求せず、GitHub Webhook 署名検証のみを認証境界とする。
- 静的コンテンツ配信、ACME HTTP-01 challenge 応答、ヘルスチェックには `X-ASB-Admin-Password` を要求しない。
- パスワードハッシュは Go 標準ライブラリのみで実装する。
- PBKDF2-HMAC-SHA256 相当処理を `crypto/hmac`、`crypto/sha256`、`crypto/rand`、`crypto/subtle`、`encoding/base64` で内製実装する。
- salt は32 bytes、hash は32 bytes、iteration は210000回とする。
- `passwordHash` と `passwordSalt` は `base64.RawURLEncoding` で保存する。
- パスワード照合は `crypto/subtle.ConstantTimeCompare` で行う。
- 平文パスワードを JSON、ログ、標準出力、標準エラー、エラーレスポンスへ保存または出力しない。
- APIキー、複数ユーザー、ロール、権限分離、組織、チーム、テナント、セッション、JWT、OAuth/OIDC、Basic認証、Bearer token、cookie認証を実装しない。
- `Authorization` ヘッダーまたは `X-API-Key` ヘッダーを認証判断に使用しない。

### 完了条件

- `config/auth.json` の空状態、読み込み、未知フィールド拒否、atomic save のテストが成功する。
- 初期デフォルトパスワードで管理者パスワード変更 API のみ実行できるテストが成功する。
- `X-ASB-Admin-Password` 未指定、空文字、不一致で `ERR_AUTH_FAILED` を返すテストが成功する。
- 管理者パスワード変更 API が `X-ASB-Admin-Password` ヘッダーを要求せず、`currentPassword` を認証入力として扱うテストが成功する。
- 管理者パスワード変更後、変更後パスワードのみが有効になるテストが成功する。
- Webhook、静的配信、ACME HTTP-01 challenge、ヘルスチェックが管理者パスワードを要求しないテストが成功する。
- 平文パスワードが JSON、ログ、標準出力、標準エラー、エラーレスポンスへ出力されないことを確認する。
- APIキー、複数ユーザー、ロール、セッション、token、cookie 認証の実装が追加されていないことを確認する。
- `go test ./...` が成功する。
- `.gitignore` が存在しない。
- 開発リポジトリ内に実行時データ、ログ、一時ファイル、ビルド成果物が残っていない。

### 非対象

- 複数ユーザー管理
- ユーザー別権限管理
- APIキー管理
- セッション管理
- JWT / OAuth / OIDC / Basic / Bearer token / cookie 認証
- SDK 認証拡張仕様

## 17. 実装済みフェーズ

現時点ではなし。

## 18. 実装フェーズ外の昇格待ちタスク

以下は Rev.68 時点では実装フェーズに含めない。

- SDK認証拡張を実装対象へ昇格する場合の認証方式、対象SDK実装、ASB管理APIとの関係、単一システム管理者認証との併存または置換、APIキー管理、複数ユーザー化、保存JSON、公開API、Web UI、監査ログ、migration、downgrade、テスト条件を仕様改訂で確定する。
- 移管元由来の Content Pipeline を実装対象へ昇格する場合は、`.md`、`.mdx`、`.json`、JSON Front Matter、content metadata、parser / sanitizer 採否、source directory、schema file、output directory、cache directory、slug 重複、draft、unsafe HTML、link URL scheme、保存JSON、cache、migration、downgrade、テスト条件を仕様改訂で確定する。
- 移管元由来の Site Routing、Site Rendering を実装対象へ昇格する場合は、route source、route file extension、dynamic route、catch-all route、route conflict、`index.html`、trailing slash、404、redirect、method handling、SSG、SSR、Hybrid Rendering、公開 API route、middleware、route manifest、build manifest、preview/dev server、HTTPS 必須性、生成物配置、テスト条件を仕様改訂で確定する。
- 移管元由来の Site Output、Blog、Docs、Sitemap、Ad Slot を実装対象へ昇格する場合は、採否、出力 artifact、atomic publish、rollback、Backup 対象性、route、metadata、外部SDK非内蔵、tracking 非対応、テスト条件を仕様改訂で確定する。
- 移管元由来の Source Sync、External Data Integration、Runtime Cache を実装対象へ昇格する場合は、GitHub Webhook デプロイとの責務分離、対象 provider、認証方式、secret 保存、署名検証、retry、冪等性、timeout、schema validation、fallback、個人情報非取得、cache key、TTL、invalidation、容量上限、migration 対象性、テスト条件を仕様改訂で確定する。
- 移管元由来仕様を実装対象へ昇格する場合は、ASB 通常機能名のみを使い、旧プロジェクト専用名、互換モード、旧プロジェクト CLI、旧プロジェクト runtime、`auteur.config.json`、`.auteur/`、`auteur-project/`、`AUTEUR_*` error code を作らないことを仕様改訂で確定する。
- 複数インスタンス対応を実装対象へ昇格する場合の node 識別子、共有ストレージ、lock方式、障害時復旧、同時書き込み整合性、Webhook重複処理、ACME更新競合、Backup競合、ログ集約、設定形式、migration、downgrade、テスト条件を仕様改訂で確定する。
- NFS連携を実装対象へ昇格する場合の対応NFS version、mount前提、lock方式、atomic rename前提、権限、障害時挙動、性能前提、複数インスタンス対応との関係、設定形式、migration、downgrade、テスト条件を仕様改訂で確定する。
- 分散ストレージ連携を実装対象へ昇格する場合の保存対象、整合性モデル、書き込み順序、読み取り優先順位、障害時復旧、データ修復、複数インスタンス対応との関係、設定形式、migration、downgrade、テスト条件を仕様改訂で確定する。
- 外部ストレージ連携を実装対象へ昇格する場合の対象provider、外部SDK採否、credential保存方式、暗号化要否、保存対象、同期方式、整合性、失敗時再試行、Backupとの関係、設定形式、migration、downgrade、テスト条件を仕様改訂で確定する。
- ログファイル暗号化を実装対象へ昇格する場合の暗号方式、key保存方式、key rotation、復号API、閲覧権限、既存ログ移行、Backupとの関係、外部KMS採否、設定形式、migration、downgrade、テスト条件を仕様改訂で確定する。
- デスクトップアプリ向けSDK利用を実装対象へ昇格する場合の対象OS、配布方式、署名、更新方式、GUIライブラリ採否、SDK実装、認証方式、保存データ、ASB本体との責務分界、外部依存、migration、downgrade、テスト条件を仕様改訂で確定する。
- モバイルアプリ向けSDK利用を実装対象へ昇格する場合の対象OS、配布方式、署名、更新方式、GUIライブラリ採否、SDK実装、認証方式、通知、offline cache採否、保存データ、ASB本体との責務分界、外部依存、migration、downgrade、テスト条件を仕様改訂で確定する。
- HTTP/2 を実装対象へ昇格する場合の API、設定項目、ALPN、h2c採否、テスト条件を仕様改訂で確定する。
