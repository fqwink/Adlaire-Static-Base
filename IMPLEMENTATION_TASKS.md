# ASB 実装タスクリスト

## 1. 運用ルール

`ASB-spec.md` は、Adlaire-Static-Base（ASB）のマスター仕様書正本である。

本ファイルは、`ASB-spec.md` に基づいて実装タスクを管理する。

参照仕様バージョン: `ASB-spec.md Rev.18`

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
- Rev.18 時点では複数JSON更新に外部トランザクション機構を導入しない。
- Rev.18 時点の API エンドポイント固定表に記載されたメソッド、パス、成功ステータス、失敗コードを実装する。
- Rev.18 API 成功レスポンス固定表に記載された JSON キーと型を実装する。
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
- 未割当 Host は `404 Not Found` とする。
- `/` は `index.html` として解決する。
- 静的ファイル配信は `storage/projects/:projectId/contents/` 配下に限定する。
- `..`、絶対パス、URL decode 後に配信ルート外へ出るパスを拒否する。
- 静的配信の `Content-Type` は Go 標準ライブラリで判定し、不明時は `application/octet-stream` とする。
- アクセスログとエラーログのJSON出力を実装する。
- アクセスログの JSON Lines 固定フィールドを実装する。
- エラーログの JSON Lines 固定フィールドを実装する。
- ログレベル DEBUG、INFO、WARN、ERROR を実装する。
- ログ保持期間7日間の方針を実装または運用仕様として整理する。
- `go test ./...` によるユニットテスト基盤を整備する。
- Project API、File API、起動時検証、404/405、共通エラーレスポンス、保留機能未実装のテストを整備する。
- JSON 保存の排他、atomic rename、保存失敗時挙動のテストを整備する。
- 静的配信の Host 解決、`index.html` 解決、path traversal 拒否、Content-Type、Gzip のテストを整備する。
- API 固定表の全エンドポイント、URL パラメータ検証、ログ `limit` / `offset` 境界値のテストを整備する。
- 全API成功レスポンスの固定JSONキー検証テストを整備する。
- 全APIエラーレスポンスの固定JSONキー検証テストを整備する。
- 保存JSONの未知フィールド拒否、相対パス保存、ソート順検証テストを整備する。
- 起動時検証の順序、終了コード、標準エラー形式検証テストを整備する。
- アクセスログとエラーログのJSON Linesフィールド検証テストを整備する。
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
- `config/webhooks.json` の `events[]` スキーマ、`receivedAt` 降順、同一時刻時 `key` 昇順を実装する。
- `X-GitHub-Event` が `push` 以外の場合は `200 OK` と `{"status":"ignored"}` を返す。
- 対象外ブランチの場合は `200 OK` と `{"status":"ignored"}` を返す。
- 同一冪等キー受信時は `200 OK` と `{"status":"duplicate"}` を返す。
- Webhook 処理成功後にのみ冪等キーを保存する。
- バックアップ一覧 API `GET /api/backups` を実装する。
- バックアップ復旧 API `POST /api/backups/restore/:id` を実装する。
- `config/backups.json` によるバックアップ履歴管理を実装する。
- `config/backups.json` の `backups[]` スキーマ、`createdAt` 降順、同一時刻時 `id` 昇順を実装する。
- tar.gz 形式のバックアップ作成を実装する。
- Backup作成処理順序を Project検証、対象データ読み込み検証、tar.gz一時作成、SHA-256計算、atomic rename、`backups.json`保存、成功応答の順に固定して実装する。
- SHA-256 ハッシュによるバックアップ整合性検証を実装する。
- 復旧前の既存データ退避を実装する。
- 復旧後の整合性確認を実装する。
- Backup復旧処理順序を Backup履歴検証、Backupファイル存在検証、SHA-256検証、現行データ退避、展開、展開後JSON検証、atomic rename、成功応答の順に固定して実装する。
- Backup復旧途中失敗時は可能な限り退避領域から復元し、復元失敗時は `ERR_BACKUP_RESTORE_FAILED` を返す。
- システムステータス API `GET /api/monitoring/stats` を実装する。
- CPU、メモリ、ディスク、接続数、リクエスト数の監視値取得を実装する。
- OS 依存で取得できない監視値は `null` として成功レスポンスに含める。
- アクセスログ API `GET /api/logs/access` を実装する。
- エラーログ API `GET /api/logs/error` を実装する。
- `limit` と `offset` によるログ取得を実装する。
- `limit` はデフォルト100、最小1、最大1000を実装する。
- `offset` はデフォルト0、最小0を実装する。
- 統合テストで全APIエンドポイントのリクエスト・レスポンス仕様を検証する。
- E2E テスト `tests/e2e.sh` を整備する。

## 4. 低優先度

- `install.sh` を実装する。
- `update.sh` を実装する。
- `asb.service` のsystemdユニットを提供する。
- Linux amd64 向けビルド手順を自動化する。
- Linux arm64 向けビルド手順を自動化する。
- HTTP/2 対応方針を Go 標準ライブラリで実装可能な範囲として整理する。
- GitHub Releases 等での配布手順を整理する。
- リリースバイナリのSHA-256チェックサム生成を実装する。
- GUI、デスクトップアプリ、Web UI の将来計画を管理する。
- モバイルアプリ化の将来計画を管理する。
- クラウドサービス化の将来計画を管理する。
- 複数インスタンス対応に向けたNFSまたは分散ストレージ調査を管理する。
- 外部ストレージサービス統合を将来候補として管理する。
- ログファイル暗号化を将来候補として管理する。

## 5. 仕様未確定タスク

- ACME クライアント内製実装、CA選定、テスト方法、失敗時挙動を確定する。
- APIキー管理の採用可否と実装範囲を確定する。
- SDK通信規格と配布方針を確定する。
- Rate limiting の採用可否と実装範囲を確定する。
- GitHub Webhook の署名検証、デプロイ元取得方式を確定する。
- バックアップ保存先を別障害領域へ配置する具体要件を確定する。
- 安定版リリース判定基準をリリースポリシーとして確定する。
- Webhook失敗時の自動リトライスケジュール仕様を確定する。
- Brotli 圧縮を採用する場合の外部ライブラリ例外採用可否を確定する。
- SSL証明書自動更新の実通信とスケジューリング仕様を確定する。
- Rev.18 の保留機能実装禁止契約に反する実装が入らないことを確認する。
- マイグレーションの `schemaVersion`、`--dry-run`、`--apply`、事前バックアップ、途中失敗、ロールバック、開発リポジトリ非生成のテストを整備する。

## 6. 実装済みリスト

現時点ではなし。
