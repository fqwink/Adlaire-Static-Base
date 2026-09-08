# ASB 実装タスクリスト

## 1. 運用ルール

`ASB-spec.md` は、Adlaire-Static-Base（ASB）のマスター仕様書正本である。

本ファイルは、`ASB-spec.md` に基づいて実装タスクを管理する。

参照仕様バージョン: `ASB-spec.md Rev.27`

`ASB-spec.md` で仕様確定済みの事項を実装タスクとしてリスト化する。

`ASB-spec.md` と本ファイルが矛盾する場合は、`ASB-spec.md` を正とする。

`ASB-spec.md` に記載のない内容を、実装確定タスクとして扱ってはならない。

実装が完了したタスクは、未実装リストから削除し、`6. 実装済みリスト` へ移動する。

`ASB-spec.md` の仕様改訂によりタスク内容が変わる場合は、本ファイルも正本仕様に合わせて更新する。

## 2. 高優先度

- Go 1.21 以上による ASB 本体の単一バイナリ基盤を構築する。
- `main.go`、`go.mod`、`README.md` の基本プロジェクトファイルを作成する。
- Management Domain、Delivery Domain、Data Domain、System Domain の基本ディレクトリ構成を作成する。
- 各責務を Handler、Service、Entity の層構造で実装できる境界を整備する。
- ドメイン間の接続を `main.go` で一元管理する。
- 責務間の循環依存を禁止する構成にする。
- `config`、`server`、`management`、`delivery`、`data`、`system` の package 境界を整備する。
- `config.Loader`、`server.Router`、`server.Responder`、`management.ProjectService`、`management.DomainService`、`management.SSLService`、`delivery.FileService`、`delivery.StaticService`、`delivery.WebhookService`、`data.JSONRepository`、`data.StorageService`、`data.BackupService`、`system.LogService`、`system.MonitoringService`、`system.Clock`、`system.IDGenerator` の公開 interface 境界を整備する。
- `main.go` は設定読み込み、依存関係生成、HTTPサーバー起動、graceful shutdown のみに限定する。
- 他 package の具象型生成を `main.go` の依存関係生成処理に限定する。
- Handler、Service、Entity の責務分離を実装する。
- Handler が JSON ファイルを直接読み書きしない構造にする。
- Handler が Service interface のみに依存する構造にする。
- Service が Repository、Storage、Clock、IDGenerator、LogService interface に依存する構造にする。
- Entity がファイル入出力、HTTP 入出力、時刻取得、ID生成を行わない構造にする。
- Entity は保存形式とレスポンス形式の型定義のみを持つ構造にする。
- ASB互換目標の静的コンテンツ専用ホスティング機能目標を実装基準として扱う。
- ASB 独自仕様であるセルフホスト、Go単一バイナリ、JSONファイルベース、外部DB不使用を実装制約として扱う。
- 起動時に設定済み実行時データ領域の存在確認と権限検証を実装する。
- 実行時データ領域が存在しない、または権限が不足する場合は起動失敗とする。
- 起動時に実行時データ用のディレクトリまたはファイルを自動生成しないことを実装する。
- 開発リポジトリを実行時データ保存先として扱わないことを実装する。
- `config/config.json` の読み込み、デフォルト値適用、起動時バリデーションを実装する。
- `storage.basePath` 配下の `config/`、`storage/`、`logs/`、`certs/` の存在検証を実装する。
- 必要な JSON ファイルの存在確認、構文検証、必須フィールド検証を実装する。
- `config/projects.json`、`config/domains.json`、`config/backups.json`、`config/webhooks.json`、`storage/projects/:projectId/files.json` の起動時存在検証を実装する。
- `server.port`、`server.host`、`server.shutdownTimeout`、`storage.basePath`、`storage.maxProjectSize`、`ssl.email`、`ssl.renewBefore`、`log.level`、`log.format`、`log.maxSize` の起動時バリデーションを実装する。
- 設定未知フィールド検出時の起動失敗を実装する。
- Go標準 `net/http` によるHTTPサーバーを実装する。
- グレースフルシャットダウンと `shutdownTimeout` を実装する。
- API レスポンスの `application/json; charset=utf-8` 統一を実装する。
- JSON API の `Content-Type: application/json` 要求を実装する。
- charset 付き `Content-Type: application/json` を許可する。
- JSON decode で未知フィールド拒否と後続トークン拒否を実装する。
- 未定義ルート `404 Not Found` と未対応メソッド `405 Method Not Allowed` を実装する。
- `405 Method Not Allowed` では `Allow` ヘッダーを返す。
- API パスを静的ファイル配信より優先して判定する。
- 外部ルーターライブラリを使わず、Go標準 `net/http` でルーティングする。
- 共通JSONレスポンスと共通エラーレスポンス形式を実装する。
- エラーレスポンスに `error`、`code`、`timestamp`、`httpStatus` を含める。
- エラーレスポンスの `httpStatus` と実際の HTTP ステータスを一致させる。
- エラーレスポンスの `timestamp` を UTC RFC3339 秒精度に固定する。
- エラーレスポンスに内部ファイルパス、スタックトレース、機密値を含めない。
- JSON の未知フィールド拒否、空 Body 拒否、UTC RFC3339 日時保存を実装する。
- Project、File、Backup の UUID 形式 ID 生成を `crypto/rand` で実装する。
- JSON ファイル更新時の読み込み検証、保存前再検証、同一ファイル排他書き込みを実装する。
- JSON ファイル保存では同一ディレクトリ内の一時ファイル、`fsync`、atomic rename による置換を実装する。
- `data.JSONRepository` は JSON 読み込み、スキーマ検証、排他、atomic save のみに限定する。
- `data.JSONRepository` が HTTP ステータス、HTTP リクエスト、HTTP レスポンスを扱わないことを実装する。
- `data.StorageService` は `storage.basePath` 配下のファイル実体操作のみに限定する。
- `data.StorageService` が Project、Domain、Webhook の業務判断を行わないことを実装する。
- 一時ファイルを開発リポジトリ内へ作成しないことを実装する。
- 保存失敗時に成功レスポンスを返さないことを実装する。
- 複数JSON更新では最終JSONの保存完了まで成功レスポンスを返さないことを実装する。
- 複数JSON更新の途中失敗時に更新済みJSON名、未更新JSON名、操作名、requestId をエラーログへ記録する。
- Rev.27 時点では複数JSON更新に外部トランザクション機構を導入しない。
- Rev.27 時点の API エンドポイント固定表に記載されたメソッド、パス、成功ステータス、失敗コードを実装する。
- Rev.27 API 成功レスポンス固定表に記載された JSON キーと型を実装する。
- 管理 API は Rev.27 時点では認証なしとして実装する。
- 管理 API で `Authorization` ヘッダーと `X-API-Key` ヘッダーを認証判断に使用しないことを実装する。
- 管理 API で `Authorization` ヘッダーまたは `X-API-Key` ヘッダーの有無により、成功・失敗・レスポンス内容を変えないことを実装する。
- APIキー発行、APIキー保存、APIキー照合、APIキー失効、APIキーローテーション、APIキー権限スコープ、APIキー監査履歴を実装しないことを確認する。
- ユーザー認証、セッション管理、JWT 検証、OAuth / OIDC 連携、Basic 認証、Bearer token 認証、認証 middleware を実装しないことを確認する。
- `config/api_keys.json`、`config/users.json`、`config/sessions.json`、`storage/api_keys/`、`storage/users/` を作成しないことを実装する。
- `auth.*`、`apiKey.*` 設定項目を定義しないことを実装する。
- `config/config.json` に認証関連フィールドが存在する場合は、未知フィールドとして起動失敗させる。
- 管理 API の本番公開時保護は ASB 外部のリバースプロキシ、ファイアウォール、VPN、SSH tunnel、IP制限等の運用境界として扱う。
- Rate limiting は Rev.27 時点では ASB 本体に実装しないことを確認する。
- Rate limiting middleware、IP別制限、Host別制限、Domain別制限、Project別制限、API別制限、Webhook別制限、静的配信別制限を実装しないことを確認する。
- token bucket、leaky bucket、sliding window counter、fixed window counter、同時接続数制限、転送量制限を実装しないことを確認する。
- `429 Too Many Requests` と `Retry-After` ヘッダーを Rate limiting 用に返さないことを確認する。
- `config/rate_limits.json`、`config/limits.json`、`storage/rate_limits/`、`storage/counters/` を作成しないことを実装する。
- `rateLimit.*`、`limits.*` 設定項目を定義しないことを実装する。
- `config/config.json` に Rate limiting 関連フィールドが存在する場合は、未知フィールドとして起動失敗させる。
- 管理 API、静的配信、Webhook 受信が Rate limiting の有無により成功・失敗・レスポンス内容を変えないことを実装する。
- Rate limiting の本番対応は ASB 外部のリバースプロキシ、WAF、CDN、ファイアウォール、ロードバランサ等の運用境界として扱う。
- Brotli 圧縮は Rev.27 時点では ASB 本体に実装しないことを確認する。
- 圧縮機能は Go 標準ライブラリ `compress/gzip` による Gzip を標準対象として実装する。
- `Accept-Encoding: br` を受信しても Brotli 応答へ切り替えないことを実装する。
- `Accept-Encoding` に `gzip` と `br` の両方が含まれる場合でも、圧縮応答を返す場合は Gzip のみを使用する。
- `Accept-Encoding` に `br` のみが含まれる場合は Brotli 圧縮を行わず、未圧縮応答または既存 Gzip 仕様に従った応答のみを返す。
- Brotli 圧縮、Brotli 展開、Brotli 用外部ライブラリ、Brotli 用 middleware、Brotli 用 precompress 処理、Brotli 用動的圧縮、Brotli 用事前圧縮を実装しないことを確認する。
- `.br` ファイル自動生成、`.br` ファイル自動削除、Brotli 用キャッシュ生成、Brotli 用キャッシュ削除、Brotli 用 Content negotiation、Brotli 用 `Content-Encoding: br` 返却を実装しないことを確認する。
- `*.br`、`storage/brotli/`、`storage/cache/brotli/` を作成しないことを実装する。
- `brotli.*`、`compression.brotli.*` 設定項目を定義しないことを実装する。
- `config/config.json` に Brotli 関連フィールドが存在する場合は、未知フィールドとして起動失敗させる。
- SDK 本体は Rev.27 時点では ASB 本体に実装しないことを確認する。
- SDK 通信規格は ASB 管理 HTTP JSON API と同一として扱う。
- SDK 通信で、現行 API の HTTP method、URL path、query parameter、path parameter、request JSON body、multipart upload、success response JSON、error response JSON、HTTP status code、error code、UTC RFC3339 timestamp、pagination、static file upload 規約を使用する。
- SDK 専用 HTTP API、SDK 専用 URL prefix、SDK 専用 request body、SDK 専用 response body、SDK 専用 error format、SDK 専用 pagination、SDK 専用 upload protocol を実装しないことを確認する。
- SDK 専用 session、SDK 専用 token、SDK 専用 cookie、SDK 専用 handshake、SDK 専用 protocol negotiation、SDK 専用 version negotiation を実装しないことを確認する。
- WebSocket、gRPC、GraphQL、独自 TCP プロトコル、UDP、MQTT、AMQP、Server-Sent Events、long polling を SDK 通信として実装しないことを確認する。
- `config/sdk.json`、`config/sdk_clients.json`、`config/sdk_sessions.json`、`storage/sdk/`、`storage/sdk_sessions/` を作成しないことを実装する。
- `sdk.*`、`sdkAuth.*` 設定項目を定義しないことを実装する。
- `config/config.json` に SDK 通信関連フィールドが存在する場合は、未知フィールドとして起動失敗させる。
- SDK から ASB 管理 API を呼び出す場合でも、`Authorization` ヘッダー、`X-API-Key` ヘッダー、cookie、セッションIDを認証判断に使用しないことを確認する。
- 配列レスポンスは対象データが空でも空配列を返す。
- URL パラメータ `:id`、`:domain`、`:name` の URL decode、正規化、バリデーションを実装する。
- `:id` は UUID 形式のみ許可する。
- `:domain` は小文字正規化後にドメイン仕様で検証する。
- `:name` はパス区切り文字を含む値を拒否する。
- プロジェクト作成 API `POST /api/projects` を実装する。
- プロジェクト一覧 API `GET /api/projects` を実装する。
- プロジェクト削除 API `DELETE /api/projects/:id` を実装する。
- プロジェクト名、quota、ID、作成日時のバリデーションを実装する。
- `config/projects.json` によるプロジェクト情報の永続化を実装する。
- `config/projects.json` の `projects[]` スキーマ、`createdAt` 昇順、同一時刻時 `id` 昇順を実装する。
- Project削除処理順序を Project検証、関連Domain列挙、関連Backup列挙、`files.json`検証、`contents/`削除、`files.json`削除、`domains.json`更新、`backups.json`更新、`projects.json`更新、成功応答の順に固定して実装する。
- Project削除を best effort 成功扱いにしないことを実装する。
- プロジェクト名重複を `ERR_PROJECT_ALREADY_EXISTS` として扱う。
- 存在しないプロジェクト参照を `ERR_PROJECT_NOT_FOUND` として扱う。
- ファイルアップロード API `POST /api/projects/:id/files/upload` を実装する。
- ファイルアップロード処理順序を URL検証、Project検証、multipart検証、ファイル検証、既存JSON検証、一時ファイル書込、fsync、atomic rename、`files.json`更新、`projects.json`更新、成功応答の順に固定して実装する。
- ファイル一覧 API `GET /api/projects/:id/files` を実装する。
- ファイル削除 API `DELETE /api/projects/:id/files/:name` を実装する。
- ファイル削除処理順序を URL検証、Project検証、`files.json`検出、ファイル実体削除、`files.json`更新、`projects.json` used更新、成功応答の順に固定して実装する。
- `storage/projects/:projectId/files.json` によるファイルメタデータ管理を実装する。
- `storage/projects/:projectId/files.json` の `files[]` スキーマ、`name` 昇順、相対パス保存を実装する。
- フォルダ階層を保持した静的ファイル管理を実装する。
- プロジェクト quota に基づく容量制限を実装する。
- ファイル名、サイズ、パス区切り文字禁止のバリデーションを実装する。
- multipart アップロードのフィールド名を `file` に固定する。
- 同名ファイル上書き時の使用容量再計算を実装する。
- 同名ファイル上書き時は旧メタデータ読み込み、新ファイル一時書込、fsync、atomic rename、`files.json`更新、`projects.json` used差分更新の順で実装する。
- 旧ファイルは新ファイルの atomic rename 成功まで削除しないことを実装する。
- Gzip による静的ファイル圧縮を実装する。
- Host ヘッダーと `config/domains.json` の対応に基づく静的配信を実装する。
- Host の port、末尾 `.`, 大文字小文字を正規化してから Domain と完全一致照合する。
- 未割当 Host は `404 Not Found` とする。
- Domain が存在しても対象 Project が存在しない場合は `500 Internal Server Error` とし、`ERR_STORAGE_VALIDATION_FAILED` をログへ記録する。
- 静的配信対象パスで `GET` / `HEAD` 以外のメソッドは `405 Method Not Allowed` とし、`Allow: GET, HEAD` を返す。
- `/` は `index.html` として解決する。
- 末尾 `/` のディレクトリパスは `index.html` として解決する。
- 静的ファイル配信は `storage.basePath/storage/projects/:projectId/contents/` 配下に限定する。
- `..`、絶対パス、NUL、隠しセグメント、URL decode 後に配信ルート外へ出るパスを `404 Not Found` として拒否する。
- 静的配信の `Content-Type` は Go 標準ライブラリの拡張子判定、先頭512 bytes判定の順で判定し、不明時は `application/octet-stream` とする。
- `HEAD` は `GET` と同じヘッダーを返し、レスポンスボディを返さない。
- `ETag` を `W/"{size}-{unixModifiedTime}"` 形式で返す。
- `Last-Modified` を HTTP-date 形式で返す。
- `Cache-Control` を既定で `public, max-age=60` とする。
- `If-None-Match` と `If-Modified-Since` による `304 Not Modified` を実装する。
- Range request は Rev.27 時点では実装せず、`Range` ヘッダーを無視して `206 Partial Content` を返さない。
- `Accept-Encoding: br` では Brotli 応答を返さない。
- Brotli 用の `.br`、キャッシュ、一時ファイル、メタデータを開発リポジトリ内にも `storage.basePath` 配下にも生成しない。
- 静的配信でディレクトリ一覧を返さない。
- 静的配信で開発リポジトリ内に配信用一時ファイル、キャッシュファイル、実行時データを作成しない。
- ログ保存先を `storage.basePath/logs/` に固定する。
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
- ログ API は現行 `access.log` / `error.log` のみを新しい順に返す。
- 壊れた JSON 行を検出した場合、ログ API は `500 Internal Server Error` を返す。
- 開発リポジトリ内にログファイルを作成しない。
- `go test ./...` によるユニットテスト基盤を整備する。
- Project API、File API、起動時検証、404/405、共通エラーレスポンス、保留機能未実装のテストを整備する。
- JSON 保存の排他、atomic rename、保存失敗時挙動のテストを整備する。
- 静的配信の Host 解決、`GET` / `HEAD` / `405`、`index.html` 解決、path traversal 拒否、隠しセグメント拒否、Content-Type、ETag、Last-Modified、Cache-Control、304、Range無視、Gzip のテストを整備する。
- API 固定表の全エンドポイント、URL パラメータ検証、ログ `limit` / `offset` 境界値のテストを整備する。
- ログ API の新しい順、壊れたJSON行検出、現行ログのみ対象のテストを整備する。
- 全API成功レスポンスの固定JSONキー検証テストを整備する。
- 全APIエラーレスポンスの固定JSONキー検証テストを整備する。
- 保存JSONの未知フィールド拒否、相対パス保存、ソート順検証テストを整備する。
- 起動時検証の順序、終了コード、標準エラー形式検証テストを整備する。
- アクセスログとエラーログのJSON Linesフィールド、フィールド順、requestId一致、stdout非出力、stderr起動失敗出力、ローテーションのテストを整備する。
- Unit、Handler、Repository、Storage、Integration、Startup の最低テスト分類を整備する。
- Project削除、File upload、File overwrite、File delete、Backup作成、Backup復旧の処理順序テストを整備する。
- Handler が永続化層へ直接依存しないことをコード構造で確認する。
- マイグレーション対象を `config/projects.json`、`config/domains.json`、`config/backups.json`、`config/webhooks.json`、`storage/projects/:projectId/files.json` に限定する。
- 各実行時 JSON ファイルのトップレベル `schemaVersion` を実装する。
- `schemaVersion` 未指定の JSON ファイルを `0` として扱う。
- 未対応 `schemaVersion` 検出時の起動失敗を実装する。
- 起動時の自動マイグレーションを禁止する。
- `asb migrate --storage /var/asb --from-schema 0 --to-schema 1 --dry-run` を実装する。
- `asb migrate --storage /var/asb --from-schema 0 --to-schema 1 --apply` を実装する。
- `--dry-run` と `--apply` の同時指定を拒否する。
- `--dry-run` では実行時 JSON ファイルを変更しないことを実装する。
- `--apply` では事前検証、事前バックアップ、変換後JSON生成、再検証、atomic rename、履歴保存、完了ログ記録の順に実装する。
- マイグレーション作業ファイル、一時ファイル、退避ファイル、履歴ファイルを開発リポジトリ内に作成しないことを実装する。
- `config/migrations.json` の `schemaVersion`、`migrations[]`、`status`、`backupPath` スキーマを実装する。
- マイグレーション失敗時の事前バックアップからのロールバックを実装する。
- ロールバック失敗時に標準エラー、エラーログ、`config/migrations.json` へ `failed` として記録する。
- 外部DBマイグレーション、外部トランザクション機構、外部マイグレーションフレームワークを導入しない。
- 安定版リリース判定基準を実装手順として固定する。
- GitHub Releases を標準配布先として扱う。
- リリースタグを安定版バージョンと同一文字列にする。
- `asb-linux-amd64-vX.Y`、`asb-linux-arm64-vX.Y`、`checksums.txt` を標準配布成果物として生成する。
- `checksums.txt` のSHA-256形式と配布ファイル名一致を検証する。
- Linux amd64 と Linux arm64 の標準ビルドコマンドを実装する。
- ビルド成果物を開発リポジトリへ残さないリリース手順を実装する。
- `install.sh --version vX.Y --arch amd64|arm64` を実装する。
- `update.sh --version vX.Y --arch amd64|arm64` を実装する。
- `latest` 指定、自動最新版選択、未指定バージョンでの install/update 実行を拒否する。
- install/update でダウンロード失敗、checksum不一致、`--version` 不一致、systemd操作失敗を成功扱いしない。
- `update.sh` の起動失敗時に `/usr/local/bin/asb.previous` から復旧を試行する。
- `asb.service` を `/etc/systemd/system/asb.service` 向けの固定仕様で提供する。
- `asb.service` の `ExecStart=/usr/local/bin/asb --config /etc/asb/config.json`、`Restart=on-failure`、`NoNewPrivileges=true` を実装する。

## 3. 中優先度

- ドメイン追加 API `POST /api/projects/:id/domains` を実装する。
- ドメイン一覧 API `GET /api/projects/:id/domains` を実装する。
- ドメイン削除 API `DELETE /api/projects/:id/domains/:domain` を実装する。
- `config/domains.json` によるドメイン情報の永続化を実装する。
- `config/domains.json` の `domains[]` スキーマと `domain` 昇順を実装する。
- RFC 1035 準拠のドメインバリデーションを実装する。
- ドメインの小文字正規化、253文字以下、最大3階層制限を実装する。
- ドメイン重複割り当てを `ERR_DOMAIN_ALREADY_ASSIGNED` として扱う。
- SSL証明書管理境界を実装する。
- ACME による証明書取得・更新を ASB互換目標として扱う。
- 設定済み証明書保存先 `certs/` の検証と管理を実装する。
- SSL証明書ID、有効期限監視モデル、失敗エラー `ERR_SSL_CERT_GENERATION_FAILED` を実装する。
- SSL更新状態を確認できる管理モデルを実装する。
- GitHub Webhook API `POST /api/webhook/github` を実装する。
- GitHub Push イベントの検出を実装する。
- 対象ブランチ設定と対象外ブランチの成功扱い無視を実装する。
- 指定ブランチの自動デプロイ処理を実装する。
- デプロイ状態を確認できる管理モデルを実装する。
- Webhook処理成功・失敗ログを実装する。
- Webhook 処理の冪等性方針を仕様に従って実装する。
- `config/webhooks.json` による Webhook 冪等キー履歴保存を実装する。
- `config/webhooks.json` の `events[]` スキーマ、`status`、`completedAt`、`errorCode`、`receivedAt` 降順、同一時刻時 `key` 昇順を実装する。
- `X-GitHub-Event` が `push` 以外の場合は `200 OK` と `{"status":"ignored"}` を返す。
- 対象外ブランチの場合は `200 OK` と `{"status":"ignored"}` を返す。
- 同一冪等キー受信時は `200 OK` と `{"status":"duplicate"}` を返す。
- `webhook.githubSecret` 未設定時はWebhook署名検証を行わない。
- `webhook.githubSecret` 設定時は `X-Hub-Signature-256` を必須にする。
- Webhook署名をGo標準ライブラリ `crypto/hmac` と `crypto/sha256` で検証する。
- Webhook署名比較を `hmac.Equal` で実装する。
- 署名なし、不正形式、不一致を `401 Unauthorized` と `ERR_WEBHOOK_SIGNATURE_INVALID` で拒否する。
- GitHub Push payload の `repository.clone_url`、`repository.ssh_url`、`repository.html_url` をデプロイ元に使わない。
- `deploy.projectId` をWebhookデプロイ先Projectとして扱う。
- `deploy.projectId` 未設定時は `ERR_WEBHOOK_PROJECT_NOT_CONFIGURED` を返す。
- `deploy.sourcePath` のローカルcheckoutを唯一のデプロイ元として扱う。
- Webhook処理時にネットワーク越しのGit clone、fetch、pullを行わない。
- `deploy.sourcePath` の存在、Git worktree、`after` commit 参照可否を検証する。
- `deploy.sourcePath` 不正時は `ERR_WEBHOOK_SOURCE_INVALID` を返す。
- Webhookデプロイ対象から `.git/`、`.github/`、主要仕様・管理ドキュメントを除外する。
- Webhookデプロイ先を対象Projectの `storage/projects/:projectId/contents/` 配下に限定する。
- Webhookデプロイを一時ディレクトリ作成、静的ファイルコピー、fsync、atomic rename、`files.json`更新、`webhooks.json`保存の順に実装する。
- Webhook失敗時は `failed` と `errorCode` を `config/webhooks.json` に保存する。
- Webhook失敗時の自動リトライスケジューラーを実装しない。
- Webhook 処理完了後に処理状態を保存する。
- バックアップ一覧 API `GET /api/backups` を実装する。
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
- バックアップ作成用一時tarを `storage.basePath/backups/.tmp/` 配下に限定する。
- atomic rename 後に履歴保存へ失敗した場合は、作成済みtar.gzを削除する。
- バックアップ保存先を別障害領域へ複製する作業をASB外の運用責務として扱う。
- 外部ストレージ連携を Rev.27 時点では実装対象外として扱う。
- Backup復旧前退避先を `storage.basePath/backups/restore-staging/{restoreId}/previous/` に固定する。
- Backup復旧用展開先を `storage.basePath/backups/restore-staging/{restoreId}/next/` に固定する。
- Backup履歴の `status` が `completed` でない場合は復旧を拒否する。
- 復旧対象Projectの現行データ退避に失敗した場合は、復旧処理を開始しない。
- 復旧途中失敗時は `previous/` から復元する。
- `restore-staging/{restoreId}/` 削除失敗は WARN ログに記録する。
- 開発リポジトリ内にバックアップ、一時tar、checksum、復旧用一時ファイル、退避データ、展開データを作成しない。
- システムステータス API `GET /api/monitoring/stats` を実装する。
- CPU、メモリ、ディスク、接続数、リクエスト数の監視値取得を実装する。
- OS 依存で取得できない監視値は `null` として成功レスポンスに含める。
- アクセスログ API `GET /api/logs/access` を実装する。
- エラーログ API `GET /api/logs/error` を実装する。
- `limit` と `offset` によるログ取得を実装する。
- `limit` はデフォルト100、最小1、最大1000を実装する。
- `offset` はデフォルト0、最小0を実装する。
- 管理 API が `Authorization` ヘッダーまたは `X-API-Key` ヘッダーの有無でレスポンスを変えないことをテストする。
- APIキー、ユーザー、セッション、認証状態を表す JSON ファイルまたはディレクトリを生成しないことをテストする。
- `config/config.json` に認証関連フィールドが存在する場合に未知フィールドとして起動失敗することをテストする。
- Rate limiting 用 middleware、制限アルゴリズム、永続カウンタ、設定項目、JSON ファイルまたはディレクトリを生成しないことをテストする。
- 管理 API、静的配信、Webhook 受信が Rate limiting 関連条件でレスポンスを変えないことをテストする。
- `config/config.json` に Rate limiting 関連フィールドが存在する場合に未知フィールドとして起動失敗することをテストする。
- `Accept-Encoding: br` と `Accept-Encoding: gzip, br` の静的配信レスポンスをテストする。
- Brotli 用外部ライブラリ、middleware、precompress、`.br`、キャッシュ、設定項目、メタデータを生成しないことをテストする。
- `config/config.json` に Brotli 関連フィールドが存在する場合に未知フィールドとして起動失敗することをテストする。
- SDK 通信が ASB 管理 HTTP JSON API と同一規格であることをテストする。
- SDK 専用プロトコル、SDK 専用エンドポイント、SDK 専用セッション、SDK 専用 JSON ファイルまたはディレクトリを生成しないことをテストする。
- `config/config.json` に SDK 通信関連フィールドが存在する場合に未知フィールドとして起動失敗することをテストする。
- 統合テストで全APIエンドポイントのリクエスト・レスポンス仕様を検証する。
- E2E テスト `tests/e2e.sh` を整備する。

## 4. 低優先度

- 配布成果物生成をリリース作業手順として自動化する。
- HTTP/2 対応方針を Go 標準ライブラリで実装可能な範囲として整理する。
- GUI、デスクトップアプリ、Web UI の将来計画を管理する。
- モバイルアプリ化の将来計画を管理する。
- クラウドサービス化の将来計画を管理する。
- 複数インスタンス対応に向けたNFSまたは分散ストレージ調査を管理する。
- 外部ストレージサービス統合を将来候補として管理する。
- ログファイル暗号化を将来候補として管理する。

## 5. 仕様未確定タスク

- ACME クライアント内製実装、CA選定、テスト方法、失敗時挙動を確定する。
- SDK 本体、SDK 配布方針、SDK 認証仕様を確定する。
- SSL証明書自動更新の実通信とスケジューリング仕様を確定する。
- Rev.27 の保留機能実装禁止契約に反する実装が入らないことを確認する。
- マイグレーションの `schemaVersion`、`--dry-run`、`--apply`、事前バックアップ、途中失敗、ロールバック、開発リポジトリ非生成のテストを整備する。

## 6. 実装済みリスト

現時点ではなし。
