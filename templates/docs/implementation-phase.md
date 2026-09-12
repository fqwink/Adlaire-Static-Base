# 実装フェーズテンプレート

## Pn / v0.N / 実装単位名

優先度:

目的:

### 実装タスク

- 実装するタスクを列挙する。
- `ASB-spec.md` で確定済みの仕様のみを対象にする。
- 仕様に存在しない内容をタスク化しない。

### 完了条件

- `go test ./...` が成功する。
- `git diff --check` が成功する。
- `.gitignore` が存在しない。
- 開発リポジトリ内に実行時データ、ログ、一時ファイル、cache、coverage output、build output、release artifact、download 済み asset、`dist/`、`.asb/` が存在しない。
- 対象仕様、API、JSON、禁止事項が `ASB-spec.md` と一致する。

### 非対象

- このフェーズで実装しない事項を列挙する。
- 将来計画、保留事項、検討事項を含めない。

### 生成禁止確認

- `.gitignore`
- `.asb/`
- `dist/`
- `coverage.out`
- `*.log`
- `*.tmp`
- runtime JSON
- build binary
- release artifact
