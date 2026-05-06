# REV DevSkills

REV-C / Sasuke 向けの Codex / Agent Skills と技術ナレッジパックをまとめたリポジトリです。

このリポジトリは、言語・プラットフォームごとの設計判断、依存関係ガバナンス、本番前デプロイガード、課金・セキュリティ・運用事故の点検手順を再利用できる形で管理します。

## Structure

| Directory | Purpose |
|---|---|
| `cloudflare-deploy-guard/` | Cloudflare Workers / Pages / R2 / KV / D1 / Queues などの本番前ガード。 |
| `codex-app-server-guard/` | Codex `app-server` を独自クライアントやサービスへ組み込む前の安全性・課金ガード。 |
| `go-skills-knowledge-pack/` | Go の latest stable 前提のアーキテクチャ・依存関係ナレッジパック。 |
| `payload-cms-deploy-guard/` | Payload CMS の schema / access control / upload / jobs / migration 本番前ガード。 |
| `supabase-deploy-guard/` | Supabase Database / Auth / Storage / Edge Functions / MCP 操作の本番前ガード。 |
| `typescript-skills-knowledge-pack/` | TypeScript / Node.js / Web / AI Agent / Edge / Data 系のナレッジパック。 |

## Naming Policy

- ディレクトリと通常ファイルは `kebab-case` に統一します。
- Codex Skill 規約に合わせ、各スキルの入口は `SKILL.md` のまま維持します。
- パッケージ単位の説明は `README.md`、エージェント向け運用指示は `AGENTS.md` を使います。
- 参照資料は `references/`、実行補助は `scripts/`、貼り付け用プロンプトは `prompts/` に置きます。

## Install

必要なスキルだけ `.agents/skills/` へコピーします。

```bash
mkdir -p .agents/skills
cp -R cloudflare-deploy-guard .agents/skills/
cp -R codex-app-server-guard .agents/skills/
cp -R payload-cms-deploy-guard .agents/skills/
cp -R supabase-deploy-guard .agents/skills/
```

知識パックを Codex Skill として使う場合も同じです。

```bash
cp -R go-skills-knowledge-pack .agents/skills/
cp -R typescript-skills-knowledge-pack .agents/skills/
```

## Usage

各ディレクトリの `README.md` と `SKILL.md` を入口にしてください。デプロイガード系は、対象プラットフォームへ変更を入れる前に静的スキャン、コスト試算、セキュリティ差分、ロールバック、監視、キルスイッチを確認し、`DEPLOY: GO / NO-GO` を出すためのものです。

代表例:

```bash
python .agents/skills/cloudflare-deploy-guard/scripts/cloudflare-static-risk-scan.py . --markdown
python .agents/skills/codex-app-server-guard/scripts/codex-app-server-static-risk-scan.py . --markdown --fail-on high
python .agents/skills/payload-cms-deploy-guard/scripts/payload-cms-static-risk-scan.py . --markdown --fail-on high
python .agents/skills/supabase-deploy-guard/scripts/supabase-static-risk-scan.py . --markdown --fail-on high
```

## Maintenance

- 価格、仕様、API、CLI オプションは変わるため、本番判断の前に公式ドキュメントで再確認します。
- ナレッジパック更新時は `*-update-prompt.md` を使い、公式ソースと release notes を優先します。
- `scripts/` のファイル名や README のコマンド例を変更した場合は、必ず `rg` で古い名前が残っていないか確認します。
