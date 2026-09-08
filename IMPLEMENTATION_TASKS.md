# ASB 実装タスクリスト

## 1. 運用ルール

`ASB-spec.md` は、Adlaire-Static-Base（ASB）のマスター仕様書正本である。

本ファイルは、`ASB-spec.md` に基づいて実装タスクを管理する。

参照仕様バージョン: `ASB-spec.md Rev.13`

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
- ASB互換目標の静的コンテンツ専用ホスティング機能目標を実装基準として扱う。
- ASB 独自仕様であるセルフホスト、Go単一バイナリ、JSONファイルベース、外部DB不使用を実装制約として扱う。
- 起動時に設定済み実行時データ領域の存在確認と権限検証を実装する。
- 実行時データ領域が存在しない、または権限が不足する場合は起動失敗とする。
- 起動時に実行時データ用のディレクトリまたはファイルを自動生成しないことを実装する。
- 開発リポジトリを実行時データ保存先として扱わないことを実装する。
- `config/config.json` の読み込み、デフォルト値適用、起動時バリデーションを実装する。
- `storage.basePath` 配下の `config/`、`storage/`、`logs/`、`certs/` の存在検証を実装する。
- 必要な JSON ファイルの存在確認、構文検証、必須フィールド検証を実装する。
- `server.port`、`server.host`、`shutdownTimeout`、`storage.maxProjectSize`、`ssl.renewBefore`、`log.level` の起動時バリデーションを実装する。
- Go標準 `net/http` によるHTTPサーバーを実装する。
- グレースフルシャットダウンと `shutdownTimeout` を実装する。
- API レスポンスの `application/json; charset=utf-8` 統一を実装する。
- JSON API の `Content-Type: application/json` 要求を実装する。
- 未定義ルート `404 Not Found` と未対応メソッド `405 Method Not Allowed` を実装する。
- 共通JSONレスポンスと共通エラーレスポンス形式を実装する。
- エラーレスポンスに `error`、`code`、`timestamp`、`httpStatus` を含める。
- エラーレスポンスに内部ファイルパス、スタックトレース、機密値を含めない。
- JSON の未知フィールド拒否、空 Body 拒否、UTC RFC3339 日時保存を実装する。
- Project、File、Backup の UUID 形式 ID 生成を `crypto/rand` で実装する。
- JSON ファイル更新時の読み込み検証、保存前再検証、同一ファイル排他書き込みを実装する。
- プロジェクト作成 API `POST /api/projects` を実装する。
- プロジェクト一覧 API `GET /api/projects` を実装する。
- プロジェクト削除 API `DELETE /api/projects/:id` を実装する。
- プロジェクト名、quota、ID、作成日時のバリデーションを実装する。
- `config/projects.json` によるプロジェクト情報の永続化を実装する。
- プロジェクト名重複を `ERR_PROJECT_ALREADY_EXISTS` として扱う。
- 存在しないプロジェクト参照を `ERR_PROJECT_NOT_FOUND` として扱う。
- ファイルアップロード API `POST /api/projects/:id/files/upload` を実装する。
- ファイル一覧 API `GET /api/projects/:id/files` を実装する。
- ファイル削除 API `DELETE /api/projects/:id/files/:name` を実装する。
- `storage/projects/:projectId/files.json` によるファイルメタデータ管理を実装する。
- フォルダ階層を保持した静的ファイル管理を実装する。
- プロジェクト quota に基づく容量制限を実装する。
- ファイル名、サイズ、パス区切り文字禁止のバリデーションを実装する。
- multipart アップロードのフィールド名を `file` に固定する。
- 同名ファイル上書き時の使用容量再計算を実装する。
- Gzip による静的ファイル圧縮を実装する。
- アクセスログとエラーログのJSON出力を実装する。
- ログレベル DEBUG、INFO、WARN、ERROR を実装する。
- ログ保持期間7日間の方針を実装または運用仕様として整理する。
- `go test ./...` によるユニットテスト基盤を整備する。
- Project API、File API、起動時検証、404/405、共通エラーレスポンス、保留機能未実装のテストを整備する。

## 3. 中優先度

- ドメイン追加 API `POST /api/projects/:id/domains` を実装する。
- ドメイン一覧 API `GET /api/projects/:id/domains` を実装する。
- ドメイン削除 API `DELETE /api/projects/:id/domains/:domain` を実装する。
- `config/domains.json` によるドメイン情報の永続化を実装する。
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
- バックアップ一覧 API `GET /api/backups` を実装する。
- バックアップ復旧 API `POST /api/backups/restore/:id` を実装する。
- `config/backups.json` によるバックアップ履歴管理を実装する。
- tar.gz 形式のバックアップ作成を実装する。
- SHA-256 ハッシュによるバックアップ整合性検証を実装する。
- 復旧前の既存データ退避を実装する。
- 復旧後の整合性確認を実装する。
- システムステータス API `GET /api/monitoring/stats` を実装する。
- CPU、メモリ、ディスク、接続数、リクエスト数の監視値取得を実装する。
- OS 依存で取得できない監視値は `null` として成功レスポンスに含める。
- アクセスログ API `GET /api/logs/access` を実装する。
- エラーログ API `GET /api/logs/error` を実装する。
- `limit` と `offset` によるログ取得を実装する。
- `limit` のデフォルト100、最大1000を実装する。
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

## 6. 実装済みリスト

現時点ではなし。
