# zenn-content（匿名連載「壊さない自動化ラボ」Zenn配管）

noteの匿名連載「壊さない自動化ラボ」（https://note.com/kowasanai_lab）の無料回をZennへ転載するための配管。
`articles/` に5本用意済み（すべて `published: false`）。

## Naoyaがやる接続手順（3行）

1. Zennアカウントを（できれば匿名運用用に）作成する
2. このリポジトリをGitHubにpush（未pushなら下記コマンド）→ Zennダッシュボード「GitHubからのデプロイ」でこのリポジトリを連携する
3. 公開したい記事のfront matterを `published: false` → `published: true` に変え、pushすれば自動で公開される（予約投稿は `published_at: "YYYY-MM-DD hh:mm"` をJSTで追加）

```bash
# GH_TOKEN環境変数が使える場合のみ試行済み。無ければ以下を手動実行:
gh repo create kowasanai-lab/zenn-content --public --source=. --push
```

## 中身

- `articles/ep001-ai-fixed-bug-while-sleeping.md` 〜 `ep004-zero-effect-flawed-measure.md`: note連載EP001〜EP004の転載（`type: idea`）
- `articles/gate-lite-deploy-approval-hook.md`: gate-lite（PreToolUseフックで本番デプロイを承認制にする仕組み）の紹介＋導入手順（`type: tech`）

## 注意

- git authorはこのリポジトリローカルで匿名設定済み（`kowasanai-lab <kowasanailab@users.noreply.github.com>`）。本名グローバル設定を継承しない
- Zennの「GitHubからのデプロイ」は連携できるリポジトリが最大2つまで（Zenn仕様・2026-09-11時点）
- 画像を足す場合はリポジトリ直下 `/images` に置き、絶対パス `![](/images/xxx.png)` で参照する（相対パス不可）
