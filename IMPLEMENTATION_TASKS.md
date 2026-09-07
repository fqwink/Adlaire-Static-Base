# ASB 実装タスクリスト

## 1. 運用ルール

`ASB-spec.md` は、Adlaire-Static-Base（ASB）のマスター仕様書正本である。

本ファイルは、`ASB-spec.md` に基づいて実装タスクを管理する。

参照仕様バージョン: `ASB-spec.md Rev.1.7`

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
- 起動時に `.asb/config/`、`.asb/storage/`、`.asb/logs/`、`.asb/certs/` を自動生成する。
- `config/config.json` の読み込み、デフォルト値適用、起動時バリデーションを実装する。
- Go標準 `net/http` によるHTTPサーバーを実装する。
- グレースフルシャットダウンと `shutdownTimeout` を実装する。
- 共通JSONレスポンスと共通エラーレスポンス形式を実装する。
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
- プロジェクト quota に基づく容量制限を実装する。
- ファイル名、サイズ、パス区切り文字禁止のバリデーションを実装する。
- 同名ファイル上書き時の使用容量再計算を実装する。
- Gzip による静的ファイル圧縮を実装する。
- Brotli 対応方針を仕様の外部依存ポリシーと整合する形で確定する。
- アクセスログとエラーログのJSON出力を実装する。
- ログレベル DEBUG、INFO、WARN、ERROR を実装する。
- ログ保持期間7日間の方針を実装または運用仕様として整理する。
- `go test ./...` によるユニットテスト基盤を整備する。

## 3. 中優先度

- ドメイン追加 API `POST /api/projects/:id/domains` を実装する。
- ドメイン一覧 API `GET /api/projects/:id/domains` を実装する。
- ドメイン削除 API `DELETE /api/projects/:id/domains/:domain` を実装する。
- `config/domains.json` によるドメイン情報の永続化を実装する。
- RFC 1035 準拠のドメインバリデーションを実装する。
- ドメイン重複割り当てを `ERR_DOMAIN_ALREADY_ASSIGNED` として扱う。
- Let's Encrypt 連携用のSSL管理境界を実装する。
- ACME クライアント内製実装の詳細設計を確定する。
- 証明書保存先 `.asb/certs/` の管理を実装する。
- SSL証明書自動更新のスケジューリングを実装する。
- GitHub Webhook API `POST /api/webhook/github` を実装する。
- GitHub Push イベントの検出を実装する。
- 指定ブランチの自動デプロイ処理を実装する。
- Webhook処理成功・失敗ログを実装する。
- Webhook失敗時の自動リトライスケジュール仕様を確定する。
- バックアップ一覧 API `GET /api/backups` を実装する。
- バックアップ復旧 API `POST /api/backups/restore/:id` を実装する。
- `config/backups.json` によるバックアップ履歴管理を実装する。
- tar.gz 形式のバックアップ作成を実装する。
- SHA-256 ハッシュによるバックアップ整合性検証を実装する。
- 復旧前の既存データ退避を実装する。
- 復旧後の整合性確認を実装する。
- システムステータス API `GET /api/monitoring/stats` を実装する。
- CPU、メモリ、ディスク、接続数、リクエスト数の監視値取得を実装する。
- アクセスログ API `GET /api/logs/access` を実装する。
- エラーログ API `GET /api/logs/error` を実装する。
- `limit` と `offset` によるログ取得を実装する。
- 統合テストで全APIエンドポイントのリクエスト・レスポンス仕様を検証する。
- E2E テスト `tests/e2e.sh` を整備する。

## 4. 低優先度

- `install.sh` を実装する。
- `update.sh` を実装する。
- `asb.service` のsystemdユニットを提供する。
- Linux amd64 向けビルド手順を自動化する。
- Linux arm64 向けビルド手順を自動化する。
- GitHub Releases 等での配布手順を整理する。
- リリースバイナリのSHA-256チェックサム生成を実装する。
- GUI、デスクトップアプリ、Web UI の将来計画を管理する。
- モバイルアプリ化の将来計画を管理する。
- クラウドサービス化の将来計画を管理する。
- 複数インスタンス対応に向けたNFSまたは分散ストレージ調査を管理する。
- 外部ストレージサービス統合を将来候補として管理する。
- ログファイル暗号化を将来候補として管理する。

## 5. 仕様未確定タスク

- Brotli 圧縮をGo標準ライブラリのみで扱うか、外部ライブラリ例外として扱うかを確定する。
- ACME クライアント内製実装の詳細仕様、テスト方法、失敗時挙動を確定する。
- APIキー管理を現行仕様で実装対象に含めるか、保留事項として扱うかを確定する。
- Rate limiting を現行仕様で実装対象に含めるか、保留事項として扱うかを確定する。
- GitHub Webhook の署名検証、対象ブランチ設定、デプロイ元取得方式を確定する。
- バックアップ保存先を別障害領域へ配置する具体要件を確定する。
- 安定版リリース判定基準をリリースポリシーとして確定する。
- 章番号 `13` の欠番を維持するか、仕様改訂で補正するかを確定する。

## 6. 実装済みリスト

現時点ではなし。
