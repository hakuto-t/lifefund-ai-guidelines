# Claude Code 環境セットアップ（LIFEFUND社内標準 v1.0）

このファイルをClaude Codeに渡すと、LIFEFUNDのベストプラクティスに基づいた作業環境が自動構築されます。
Anthropic公式ベストプラクティス + LIFEFUND AIガイドラインに準拠。

---

## セットアップ手順

以下のステップを順番に実行してください。各ステップで必要な情報はユーザーに質問してください。

### STEP 1: ヒアリング

まず以下をユーザーに質問してください。一度に全部聞いてOK。

```
Claude Code 環境セットアップを開始します。
以下の情報を教えてください:

1. あなたの名前（Git用）: （例: yamada-taro）
2. メールアドレス（Git用）: （例: taro.y@hakuto-k.jp）
3. GitHubユーザー名: （例: yamada-taro）
4. 所属部署: （例: 経営戦略室、建築事業部）
5. 主に使う言語/フレームワーク: （例: Python, Node.js, HTML, GAS）
6. 作業フォルダのパス: （例: C:\Users\yamada\Desktop\WORK）
7. Pythonのパス: （`where python` または `where py` で確認。不明なら「不明」）
8. デプロイ先: （例: GitHub Pages, なし）
```

### STEP 2: フォルダ構造の作成

作業フォルダに移動し、以下を作成:

```bash
mkdir -p .claude/rules
mkdir -p .claude/skills
mkdir -p _archive
mkdir -p 削除
```

### STEP 3: CLAUDE.md の生成

以下のテンプレートにSTEP 1のヒアリング結果を埋めて、作業フォルダ直下に `CLAUDE.md` を作成。

**注意:**
- `{{...}}` の部分をヒアリング結果で置換する
- デプロイ先が「なし」の場合、デプロイセクション自体を削除
- Pythonが「不明」の場合、Python行を削除
- 言語/フレームワークに応じて環境セクションを調整

```markdown
# プロジェクト共通設定

## 言語
- 日本語で対応すること

## 環境
- Python: `{{Pythonのパス}}`
- Node/npm: グローバルインストール済み
- OS: Windows 11（bash使用）

## Git / GitHub
- user.name: {{名前}}
- user.email: {{メールアドレス}}
- GitHub: {{GitHubユーザー名}}
- 大きいリポジトリのpushは50ファイル/バッチに分割（HTTPS タイムアウト回避）

## デプロイ
- 静的サイト: GitHub Pages（public repo）
- HTMLファイル名は `index.html` にリネーム

## 情報セキュリティ
**IMPORTANT:** LIFEFUND AIガイドラインに基づき、以下を厳守すること
- 個人情報（氏名・住所・電話番号・マイナンバー・口座情報）をAIに入力しない
- 顧客情報・取引先情報を含むデータはマスキングしてから処理する
- パスワード・APIキー・認証トークンをコードにハードコードしない（.envファイルを使用し、.gitignoreに追加）
- 社外秘の経営情報（未公開の財務データ、M&A情報等）をAIに入力しない
- 判断に迷う場合は処理を中断し、ユーザーに確認する

## ファクトチェック必須ルール
**IMPORTANT:** 自分の知識だけで事実情報を断言しない。**必ずWebSearchで裏取りしてから回答する**
- 対象: 外部サービスの仕様・料金・API、ライブラリのバージョン・使い方、技術的な制約・互換性
- 不確かな場合は「確認します」と言ってから調べる。推測で答えない

## AI出力の品質管理
- AI生成物（コード・文書・分析結果）は必ずユーザーが最終確認する前提で出力する
- 「AIが生成した」ことを隠さない。社外提出物にはAI利用の旨を明記する
- 法務・財務・人事に関わる判断をAI単独で行わない。必ず「参考情報」として提示する

## ファイル削除ルール
**IMPORTANT:** Claude Codeにはファイル・フォルダの削除権限がない
- 「削除」とは、ルート直下の `削除/` フォルダに移動することを意味する
- `削除/` フォルダが存在しない場合は、先に作成してから移動する

## フォルダ運用
- 新規プロジェクトはルート直下に作成する（カテゴリフォルダに入れない）
- 完了・休止したプロジェクトは `_archive/` に移動
- プロジェクト一覧は @.claude/rules/project-map.md を参照
```

### STEP 4: .claude/rules/ ファイル群の生成

以下のファイルを全て作成する。

#### 4-1. `.claude/rules/project-map.md`（pathsなし=毎回読込）

```markdown
---
description: プロジェクト一覧と配置ルール
---
# プロジェクト地図

このフォルダ配下のプロジェクト一覧。プロジェクトを探す際はこの地図を参照すること。

| フォルダ | 概要 | Git |
|---|---|---|
| （プロジェクト追加時にここに1行追記する） | | |
```

#### 4-2. `.claude/rules/session-behavior.md`（pathsなし=毎回読込）

```markdown
---
description: セッション管理・チーム・メッセージングの行動ルール
---
# セッション行動ルール

## セッション名リマインド
- セッションのテーマが判明した時点で、まだセッション名が未設定ならセッション名を提案
- 作業内容がわかる簡潔な日本語名（例: `MCP設定修正`、`パワーアップ研修ノート作成`）
- `~/.claude/sessions/` 内の現在のPIDに対応するJSONファイルの `name` フィールドを更新
- 現在のPIDは `$PPID` またはセッションファイル一覧から特定する
- 最初の1回だけ提案し、しつこく繰り返さない

## チームモード（Agent Teams）
- ユーザーが「チームで」「分担して」「並列でやって」「チームモード」などと言ったら、Agent Teamsでタスクを分担する
- チームメイトの役割分担はタスク内容に応じて自動的に決める
- チームメイト数は特に指定がなければ、タスクの規模・性質から最適な人数を自分で判断する
- 作業完了時はリーダーが結果をまとめてユーザーに報告する

## セッション間メッセージング
- ユーザーが「隣に伝えて」「他のセッションに送って」「別ターミナルに共有して」などと言ったら、SendMessageで別セッションにメッセージを送信する
- 送信先のセッション名またはIDが不明な場合は、ユーザーに確認する
```

#### 4-3. `.claude/rules/context-management.md`（pathsなし=毎回読込）

```markdown
---
description: コンテキスト管理の運用ルール（Anthropic公式推奨）
---
# コンテキスト管理

Claudeのコンテキストウィンドウは有限。以下を守って効率的に使うこと。

## 基本原則
- 無関係なタスクの間では `/clear` を使ってコンテキストをリセットする
- 調査・探索タスクはサブエージェント（Agent tool）に委譲し、メインのコンテキストを汚さない
- 2回修正してうまくいかない場合は `/clear` して、改善したプロンプトでやり直す
- `/compact` で重要情報を保持したまま圧縮できる

## やってはいけないこと
- 1つのセッションで無関係な複数タスクを混ぜる（キッチンシンクセッション）
- 範囲を限定せず「調査して」と指示する（スコープを絞るかサブエージェントで）
- 同じ失敗アプローチを繰り返す（コンテキストが汚れて性能劣化する）
```

#### 4-4. `.claude/rules/security.md`（pathsなし=毎回読込）

```markdown
---
description: LIFEFUND AIガイドライン準拠の情報セキュリティルール
---
# 情報セキュリティルール

## データ分類と取り扱い
| 分類 | 例 | AI入力 |
|---|---|---|
| 公開情報 | 公開済みWebサイト、プレスリリース | OK |
| 社内情報 | 社内手順書、議事録、業務マニュアル | 注意して利用可 |
| 機密情報 | 未公開財務データ、経営戦略、M&A情報 | 禁止 |
| 個人情報 | 氏名、住所、電話番号、マイナンバー、口座情報 | 禁止 |
| 認証情報 | パスワード、APIキー、トークン | 禁止 |

## コード上のセキュリティ
- APIキー・パスワードは `.env` ファイルに記載し、`.gitignore` に追加する
- `.env` ファイルをコミットしようとしたら即座に警告する
- config.env、credentials.json 等の認証ファイルもコミット対象外にする
- 顧客データを含むJSON/CSVはリポジトリに含めない

## 判断に迷った場合
- 「この情報をAIに渡してよいか？」と迷ったら、渡さない
- ユーザーに「この情報は社内情報/機密情報に該当しますか？」と確認する
- AI活用推進担当（経営戦略室）に相談することを提案する
```

#### 4-5. 言語/フレームワーク別ルール

ヒアリングで判明した言語に応じて、適切なrulesファイルを**自動判断して**作成する。

**Pythonプロジェクトの場合** → `.claude/rules/python-projects.md`:
```markdown
---
description: Pythonプロジェクトのルール
paths:
  - "**/*.py"
---
- 仮想環境(venv)がある場合はそのPythonを使用
- pip installは仮想環境内で実行
- エンコーディングはUTF-8を明示
- 日本語テキストはターミナルで文字化けするのでファイル経由で扱う
```

**HTML/静的サイトの場合** → `.claude/rules/html-projects.md`:
```markdown
---
description: HTML単体プロジェクトのルール
paths:
  - "**/*.html"
---
- GitHub Pagesデプロイ前提。メインファイルは index.html にリネーム
- ライトテーマ（白背景+インディゴアクセント）のデザインシステムを使用
- 画像はWebP形式（quality=85, method=6）
```

**Node.js/TypeScriptの場合** → `.claude/rules/node-projects.md`:
```markdown
---
description: Node.js/TypeScriptプロジェクトのルール
paths:
  - "**/*.ts"
  - "**/*.tsx"
  - "**/*.js"
  - "**/package.json"
---
- npm ciで依存関係をインストール
- テストはvitest or jestで実行
- node_modulesは.gitignoreに含める
```

**GAS（Google Apps Script）の場合** → `.claude/rules/gas-projects.md`:
```markdown
---
description: Google Apps Scriptプロジェクトのルール
paths:
  - "**/*.gs"
---
- claspでローカル開発・デプロイ
- スプレッドシートIDはハードコードせず変数化する
- APIキーはPropertiesServiceで管理する
```

**Gitリポジトリ共通** → `.claude/rules/git-projects.md`:
```markdown
---
description: Gitリポジトリ共通ルール
---
- コミットメッセージは日本語でも英語でもOK
- 大きいリポジトリのpushは50ファイル/バッチに分割（HTTPSタイムアウト回避）
- .env、credentials.json、config.env 等の認証ファイルは絶対にコミットしない
- .gitignoreに以下を含めること: .env, *.local, node_modules/, __pycache__/, .claude/settings.local.json
```

**判断基準:** ヒアリングの回答に基づいて、該当するものだけ作成する。不要なものは作らない。Gitリポジトリ共通は必ず作成する。

### STEP 5: 検証

以下を実行して結果を報告:

```bash
echo "=== CLAUDE.md ==="
wc -l CLAUDE.md

echo "=== .claude/rules/ ==="
ls .claude/rules/

echo "=== フォルダ構造 ==="
ls -d _archive/ 削除/ .claude/rules/ .claude/skills/ 2>/dev/null

echo "=== CLAUDE.md 行数チェック ==="
lines=$(wc -l < CLAUDE.md)
if [ "$lines" -lt 200 ]; then
  echo "OK: ${lines}行（200行未満）"
else
  echo "WARNING: ${lines}行（200行超過！スリム化が必要）"
fi

echo "=== セキュリティルール存在チェック ==="
if [ -f .claude/rules/security.md ]; then
  echo "OK: security.md あり"
else
  echo "ERROR: security.md がありません！"
fi
```

### STEP 6: 完了報告

以下の形式でユーザーに報告:

```
## セットアップ完了

| 項目 | 状態 |
|---|---|
| CLAUDE.md | ✅ {{行数}}行（200行未満） |
| .claude/rules/ | ✅ {{ファイル数}}ファイル |
| セキュリティルール | ✅ security.md 適用済み |
| コンテキスト管理ルール | ✅ context-management.md 適用済み |
| _archive/ | ✅ 作成済み |
| 削除/ | ✅ 作成済み |

### 作成されたファイル
- CLAUDE.md — 環境設定・セキュリティ・品質管理ルール
- .claude/rules/project-map.md — プロジェクト地図
- .claude/rules/session-behavior.md — セッション行動ルール
- .claude/rules/context-management.md — コンテキスト管理ルール（Anthropic公式準拠）
- .claude/rules/security.md — 情報セキュリティルール（AIガイドライン準拠）
- .claude/rules/git-projects.md — Git共通ルール
- .claude/rules/{{言語別ルール}}.md — 言語固有ルール

### セキュリティ確認事項
以下の点を理解した上で利用を開始してください:
- 個人情報・機密情報はAIに入力しない
- APIキー・パスワードは .env ファイルで管理する
- AI出力は必ず人間が最終確認する
- 判断に迷ったらAI活用推進担当（経営戦略室）に相談

### 日常の使い方Tips
- タスクが変わったら `/clear` でコンテキストをリセット
- 調査は「チームで」と言ってサブエージェントに委任すると効率的
- プロジェクト追加時は `project-map.md` に1行追記
- 休止プロジェクトは `_archive/` に移動
```

---

## 準拠基準

| 基準 | 内容 |
|---|---|
| **Anthropic公式** | code.claude.com/docs/en/best-practices |
| **LIFEFUND AIガイドライン** | v1.2（2026年3月3日施行） |
| **設計書** | Claude Code ベストプラクティス基本設計書 v1.0 |

### 5つの基本原則
1. **地図 > 構造** — フォルダ階層より CLAUDE.md の地図が重要
2. **CLAUDE.md は200行未満** — 超えると指示遵守率が低下
3. **ルールは分離・条件付きロード** — .claude/rules/ で paths 指定
4. **重要ルールに IMPORTANT** — 遵守率が向上
5. **階層は3段まで** — 深いとAI・人間ともにナビゲーション効率低下
