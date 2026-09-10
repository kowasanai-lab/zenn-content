---
title: "Claude Codeの本番デプロイを、承認なしでは実行できなくする（PreToolUseフック・gate-lite）"
emoji: "🚧"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["claudecode", "ai", "セキュリティ", "hook", "devops"]
published: true
---

Claude Code（または同様のAIコーディングエージェント）に本番環境の操作を任せていると、いつか必ず考えることになる。

「型チェックもレビューも済んでいないのに、`vercel --prod` や破壊的な `DELETE` 文が、そのまま実行されてしまわないか」

「気をつける」「プロンプトに注意書きを書く」は対策にならない。エージェントは文脈を見失うし、指示を読み飛ばすこともある。人間だって同じミスをする。必要なのは、**エージェントの意思とは無関係に、コマンド実行そのものを止める仕組み**だ。

その最小構成として作ったのが `gate-lite` という2ファイル完結のフックで、コードごと無料で公開している。

https://github.com/kowasanai-lab/gate-lite

## 何をするツールか

Claude CodeのPreToolUse hookとして動く、Pythonスクリプト1本（`gate.py`）と、承認済みマーカーを発行するスクリプト1本（`grant.py`）だけの構成。設定はJSON1箇所への追記だけで完了する。

止める対象は2種類。

| ゲート | 検知するコマンドの例 | 解除に必要なもの |
|---|---|---|
| `deploy` | `vercel ... --prod`, `firebase deploy` | 有効な承認マーカー（既定30分） |
| `data` | `psql`/`supabase`/`mongosh` 経由の破壊的SQL（`UPDATE`/`DELETE`/`DROP`/`TRUNCATE`/`ALTER`）、`firebase firestore:delete`、`supabase db reset`/`push` | 有効な承認マーカー（既定30分） |

マーカーが無い、期限切れ、あるいは**別プロジェクトで発行されたもの**であれば、該当コマンドは実行前に拒否される。「プロジェクトAのデプロイチェックに合格した」という事実だけで「プロジェクトBの本番デプロイ」が素通りしてしまう事故を、発行時のディレクトリ（gitリポジトリ単位）を照合することで防いでいる。

`grep "vercel --prod" README.md` のような読み取り専用コマンドに本番コマンド名の文字列が含まれるだけでは誤発火しない一方、`sh`/`eval`/`xargs`・コマンド置換のように実行内容を静的に判定できない構文が混ざっている場合は、安全側に倒して全文を判定する。マーカー自体も `touch` や Write/Edit ツールでの直書きでは作れないようにしてあり、`grant.py` を通した発行しか受け付けない。

## 導入手順（3ステップ）

1. **配置**: `gate-lite/` ディレクトリごとプロジェクト内に置く（例: `.claude/gate-lite/`）。
2. **登録**: `.claude/settings.json`（またはユーザー共通の `~/.claude/settings.json`）の `hooks.PreToolUse` に、`matcher: "Bash"` で `python3 /absolute/path/to/gate-lite/gate.py` を1行追加する。
3. **動作確認**: 付属のテストを流す。

```bash
cd gate-lite
python3 tests/test_gate_readonly.py   # 誤爆しないか・本物は止まるか（18ケース）
python3 tests/test_gate_scope.py      # マーカーのプロジェクト越境防止（5ケース）
```

すべて `PASS` になれば導入完了。あとは、デプロイ前チェック（型チェック・lint・プレビュー確認など）やDB操作前の対象確認が終わった手順の最後に、次のコマンドで承認マーカーを発行する。

```bash
python3 grant.py deploy "型チェック✓ lint✓ プレビューで実操作確認✓"
python3 grant.py data   "SELECT対象確認✓ 件数一致✓ バックアップ取得✓"
```

根拠（合格の理由）は10文字以上必須にしてあり、空欄や雑な理由では発行できない。発行内容はログに残るので、後から棚卸しもできる。

## このパッケージに含まれないもの

- SEO/MEOなど外部スクリプトに依存するサブゲートは含まない（自己完結にするため意図的に除外）
- マーカー発行の承認プロセスそのもの（何をチェックすれば合格とするか）はプロジェクトごとに決める前提。gate-liteが担うのは「承認を経ずに実行されるのを防ぐ」until部分だけ

Bash内部の判定ロジック（コマンド分割・セグメント判定・スコープ照合の実装詳細）を含む全実装の解説は、有料の実装編に譲る。

https://note.com/kowasanai_lab/n/n97c49424fb76
