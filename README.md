<div align="center">

# REV DevSkills

**An opinionated, production-grade Agent Skills arsenal — for shipping with Codex & Claude Code without bleeding.**

*Latest stable. Cost-aware. Security-architectural.*

![snapshot](https://img.shields.io/badge/snapshot-2026--05--06-black)
![rust](https://img.shields.io/badge/rust-latest%20stable%20policy-orange)
![go](https://img.shields.io/badge/go-1.26.2-00ADD8)
![node](https://img.shields.io/badge/node-24%20LTS-339933)
![verdict](https://img.shields.io/badge/verdict-DEPLOY%3A%20GO%20%2F%20NO--GO-red)
![author](https://img.shields.io/badge/by-鳥居佐助%20%2F%20REV--C%20Asia-111)

[Why](#why) · [What This Is](#what-this-is) · [Skills](#whats-inside) · [Deploy Guards](#deploy-guards) · [Knowledge Packs](#knowledge-packs) · [Philosophy](#core-philosophy) · [Author](#author)

</div>

---

> 8 Skills. 4 production deploy guards. 3 language knowledge packs. 1 repo hygiene guard. 132 static-scan rules. 60+ SQL audit checks. 80+ Rust crates / 120+ npm package entries / 40+ Go modules — knowledge packs は version register、snapshot 日付、自己採点付き。
>
> **READMEではなく、防具であり、武器であり、履歴書です。**

これは REV-C Asia CEO 兼 Developer の **鳥居 佐助 (Sasuke Torii)** が、Codex と Claude Code に本番権限を渡しながら積み上げてきた判断の結晶を、AI エージェントが読める形に圧縮したアーカイブです。Cloudflare の課金が一晩で吹き飛びかける夜、Supabase の RLS が一行抜けていた朝、Codex `app-server` の起動オプションが安全境界を曖昧にしていたコミット、Payload CMS の Local API が public route から呼ばれていたプルリク — そのひとつひとつから逆算して、「次に同じことを踏まないために、エージェントが何を読んでから動くべきか」を 8 つの Skill にまとめてあります。便利スクリプト集ではなく、本番にエージェントを置くということを真剣に扱うための土台です。

対象は **CCTeam / Social Psychometrics CRM / 高負荷送信基盤 / Leptos Web Builder / TUI AI Agent / E2EE Infrastructure** — REV-C Asia の実プロダクト群を支えるために書かれ、運用されています。

## What This Is

- SasukeTorii の開発スタイルを Codex / Claude Code に渡すための Skill pack。
- 言語 / runtime / platform ごとの「採用する技術」「避ける技術」「本番に出す前の確認」をまとめた knowledge pack。
- 高負荷 API、crawler、browser automation、AI agent、E2EE、data/search、edge runtime、CMS、database、deploy guard を扱うための実戦メモ。
- ただの checklist ではなく、agent が実装・レビュー・調査・更新の判断に使う前提知識。

## Why

エージェントに「本番にデプロイして」「migration を流して」「Edge Function を作って」と依頼したとき、人間がやれば気づく以下の事故が、エージェント任せだと素通りします。

- Bot 一回で R2 egress + Images Transformations が数千ドル単位で爆発する
- `anon` キーで `select('*')` が全テーブルを舐める RLS 設定漏れ
- Codex `app-server` を WebSocket 認証なしで露出し、tool 経由で任意コマンド実行
- Payload CMS の `read: () => true` を本番投入し、GraphQL 深堀りで関連テーブル全走査
- Spend Cap が効かない **対象外項目**（Compute / Branching / Read Replica / Custom Domain / Disk IOPS / IPv4 / Log Drains / MFA Phone / PITR）で月末に請求書が爆発
- Rust で `pre / alpha / beta / rc / dev` や **yanked version** を「最新安定版」として採用してしまう
- Go / TypeScript で「古い v0.x の情報」や、`undici` / `undici-types`、`@swc/core` / `@swc/wasm` のような **隣接パッケージのバージョン取り違え** で依存関係を pin してしまう

REV DevSkills は、これらを「人間の経験と勘」ではなく **静的スキャン・SQL 監査・コスト試算スクリプト・リスクマトリクス・チェックリスト** で捕まえます。すべてのデプロイガードは初期判定 `DEPLOY: NO-GO` から始まり、公式ドキュメントを読んだだけでは GO に切り替わりません。

## What's Inside

| カテゴリ | Skill | 役割 | スナップショット |
|---|---|---|---|
| Repo Hygiene Guard | [`.agents/skills/naming-normalization-guard/`](.agents/skills/naming-normalization-guard/) | Skill / knowledge pack 追加・リネーム時の規約、参照更新、stale name check を agent に徹底させる | 規約 |
| Deploy Guard | [`cloudflare-deploy-guard/`](cloudflare-deploy-guard/) | Workers / Pages / R2 / KV / D1 / Queues / Images で課金・Bot・cache・deploy 事故を潰す | 2026-05-06 |
| Deploy Guard | [`codex-app-server-guard/`](codex-app-server-guard/) | Codex `app-server` を独自 UI / 社内 tool / agent 基盤へ組み込む前に auth / approval / sandbox / usage を点検する | 2026-05-06 |
| Knowledge Pack | [`go-skills-knowledge-pack/`](go-skills-knowledge-pack/) | Go で高負荷 API / crawler / service daemon / CLI / workflow / security-sensitive backend を作るための判断基準 | 2026-05-06 / v0.1.1 |
| Deploy Guard | [`payload-cms-deploy-guard/`](payload-cms-deploy-guard/) | Payload CMS の access control / Local API / GraphQL / uploads / jobs / migration / storage cost を本番前に潰す | 2026-05-06 |
| Knowledge Pack | [`rust-skills-knowledge-pack/`](rust-skills-knowledge-pack/) | Rust で async worker / Leptos+WASM / TUI+audio agent / E2EE / hot path / data+search を焼き切るための判断基準 | 2026-04-29 / v0.1.2 |
| Deploy Guard | [`supabase-deploy-guard/`](supabase-deploy-guard/) | Supabase の RLS / GRANT / Auth / Storage / Realtime / Edge Functions / MCP 操作 / branch+add-on 課金を点検する | 2026-05-06 |
| Knowledge Pack | [`typescript-skills-knowledge-pack/`](typescript-skills-knowledge-pack/) | TypeScript / Node.js / Web / AI Agent / Edge / Data / crawler / E2EE の設計と dependency governance | 2026-05-06 / v0.1.2 |

### Coverage Matrix

|                                    | Cost | Security | Outage | Design | Hygiene |
|------------------------------------|:----:|:--------:|:------:|:------:|:-------:|
| `naming-normalization-guard`       |      |          |        |        | ✅      |
| `cloudflare-deploy-guard`          | ✅   | ✅       | ✅     |        |         |
| `codex-app-server-guard`           | ✅   | ✅       | ✅     | ✅     |         |
| `payload-cms-deploy-guard`         | ✅   | ✅       | ✅     |        |         |
| `supabase-deploy-guard`            | ✅   | ✅       | ✅     |        |         |
| `rust-skills-knowledge-pack`       |      | ✅       |        | ✅     |         |
| `go-skills-knowledge-pack`         |      | ✅       |        | ✅     |         |
| `typescript-skills-knowledge-pack` |      | ✅       |        | ✅     |         |

## Core Philosophy

8 つの Skill すべてを横断する非交渉ルール。

- **Latest stable first** — 原則は最新安定版。ただし `prerelease / canary / experimental / v0.x / yanked` は R&D / Watch として扱う。snapshot 日付と version を必ず明記する。
- **Production before aesthetics** — 見た目や抽象化より、実運用で壊れない境界、rollback、監視、kill switch を優先する。
- **Language has lanes** — TypeScript は web / edge / agent glue、Go は backend / CLI / durable service、Rust は低レイヤ・暗号・hot path、platform rules は Cloudflare / Supabase / Payload の制約を優先する。
- **Cost is a bug surface** — Bot / crawler / retry / loop / 画像変換 / storage egress / LLM token / DB IO は常に事故要因として扱う。課金メーターは「ユーザー数」ではなく「触れる回数」で爆発する。
- **Security is architectural** — secret / PII / 心理データ / voice payload / key material / service role / Local API bypass をログ・client・public env へ出さない。
- **Agent output must be governed** — Codex / Claude Code に任せる範囲を広げても、approval / sandbox / tool side effect / MCP 操作 / 本番権限は明示的に制御する。デプロイガードの初期値は常に `DEPLOY: NO-GO`。

> **倫理ライン (Big Five / Psychometrics)**: 推定値は確率・仮説として扱う。説明可能性と人間レビューを設ける。センシティブ属性推定や差別的意思決定を避ける。同意、保存期間、削除要求、監査を明示する。

## How Agents Should Use This

1. まず対象に合う `SKILL.md` を読む。
2. 言語系 knowledge pack では `*-master.md` を**単一の真実源**として読む。`SKILL.md` は入口であり、完全な version register ではない。
3. 分野別の詳細が必要なら `references/` を読む。`references/` は master から切り出した補助資料として扱う。
4. dependency や version を更新するなら `*-sources.md` と公式 release notes を確認する。
5. 本番へ変更する前に deploy guard 系 Skill で `DEPLOY: GO / NO-GO` を出す。
6. 未確認の pricing / quota / API behavior / security boundary は古い記憶で決めない。

## Deploy Guards

4 つの Deploy Guard は、対象サービスごとの違いを残しつつ、同じゲート構造で動きます。

1. **静的リスクスキャン** — `*-static-risk-scan.py --markdown --fail-on high`。リポジトリのコード・設定ファイルを走査し、危険パターンを Markdown / JSON で出力（4 ガード合計 **132 ルール**）
2. **動的監査** — DB の RLS / GRANT / index、本番ホストの CORS / GraphQL 到達性、`app-server` の stdio smoke test と WebSocket 露出設定の点検
3. **コストシナリオ試算** — `expected` / `10x` / bot-abuse / crawler / bug-loop / migration-failure など、guard ごとのリスクに合わせて月次コストを試算
4. **リスクマトリクス + チェックリスト** — Critical blocker と High risk を明文化、判定基準を統一フォーマットで管理
5. **`DEPLOY: GO / NO-GO` 判定** — 出力フォーマット固定（対象 / 変更概要 / 課金対象棚卸し / ブロッカー / コスト試算 / セキュリティ差分 / キルスイッチ / ロールバック / 監視 / 残余リスク）

> **初期判定は常に `DEPLOY: NO-GO`。公式ドキュメントを読んだだけでは GO にしません。実際の transport / auth / approval / sandbox / workspace 分離 / tool 権限 / usage 上限 / logs / kill switch を確認してから初めて GO にします。** — `codex-app-server-guard/SKILL.md`

### cloudflare-deploy-guard

Cloudflare は低コストで始めやすい代わりに、課金は **「ユーザー数」ではなく「課金メーターに触れる回数」** で爆発します。小規模サイトでも、画像変換 URL / KV list / DO write / Queue loop / D1 scan / R2 GET/List / Worker route 過大 / AI crawler / 外部 API 再試行で高額化する。検査項目、ブロッカー、コストシナリオで、Bot 一回が数千ドルに化ける時限爆弾を投入前に止めます。

- `scripts/cloudflare-static-risk-scan.py` — wrangler.toml / next.config / Worker コードの走査（秘密値は出力しない）
- `references/cloudflare-cost-security-checklist.md` / `cloudflare-risk-matrix.md` / `cloudflare-deploy-report-template.md`

### codex-app-server-guard

OpenAI Codex `app-server` を独自クライアント・WebSocket・MCP 統合・AI Agent 基盤として使う前のセキュリティゲート。**即 NO-GO 条件を 7 カテゴリ**（Transport / exposure、Auth / credentials、Approval / side effects、Sandbox / command execution、Tools / apps / MCP / skills、Process lifecycle / cost、Data handling / logs）に分割。approval UI に何を表示すべきか、process TTL、tool result の redact など、実装レベルの判定基準まで降りています。

- `scripts/codex-app-server-static-risk-scan.py` — config / コード走査
- `scripts/codex-app-server-smoke-test.py` — stdio 接続テスト
- `scripts/codex-app-server-launch-guard.py` — 起動コマンドの危険引数検出（non-loopback WebSocket / auth 欠落 / raw token / danger mode 等）
- `scripts/codex-app-server-cost-estimator.py` — turns / tokens / モード別の課金試算
- `references/codex-safe-config-template.toml` ほか

### payload-cms-deploy-guard

Postgres / MongoDB / SQLite の 3 アダプタを別ライン監査。最重要ルールには **「Local API は access control をデフォルトでスキップする — ユーザー起点の処理では必ず `user` を渡し、`overrideAccess: false` を明示する」** 「`maxDepth` は最小化する — relationship/upload の再帰、循環参照、深い populate は DB/CPU/メモリ/egress を急増させる」「ephemeral filesystem のホストでは Upload をローカル保存で本番運用しない」など、運用したことがある人にしか書けない条文が並びます。

- `scripts/payload-cms-static-risk-scan.py`
- `scripts/payload-cms-postgres-audit.sql` / `payload-cms-mongo-audit.js`
- `scripts/payload-cms-runtime-probe.py` — CORS / GraphQL 到達性プローブ
- `scripts/payload-cms-cost-scenario-estimator.py`
- `references/payload-cms-deploy-report-template.md` ほか

### supabase-deploy-guard

Hard blocker を「課金 × セキュリティ」で明示。同梱する SQL 監査ファイル群が「RLS が有効」と「RLS で正しく絞られている」を別物として点検します。`migration-failure` を 5 番目のコストシナリオに加える独自視点。

- `scripts/supabase-static-risk-scan.py`
- `scripts/supabase-db-audit.sql` / `supabase-inventory-audit.sql`
- `scripts/supabase-cost-scenario-estimator.py`
- `references/supabase-mcp-playbook.md` / `supabase-mcp-runbook.md` / `supabase-sql-audit-queries.md` ほか

## Knowledge Packs

「最新安定版を使え」と言うのは簡単ですが、実際の難所は **どのモジュールの、どのバージョンを、どの組み合わせで pin するか**。Knowledge Pack はその答えを 1 ファイル 1 分野で固定したスナップショットです。`pre / alpha / beta / rc / dev` および **yanked version** は最新安定版として扱いません。

そして、**この監査は片道ではありません**。TypeScript v0.1.1-strict で「`undici v8.2.0` は誤りで `v8.1.0` が正」「`@swc/core v1.15.33` は誤りで `v1.15.32` が正」と訂正していたものが、v0.1.2 の同日 npm registry 再確認で **`undici v8.2.0` / `@swc/core v1.15.33` こそが現行の公式 latest だった** と巻き戻った経緯があります。重要なのは、`undici` と `undici-types`、`@swc/core` と `@swc/wasm` のような **隣接パッケージで version が等しくなる罠** を避ける運用ルールが残ったこと。集大成の精度はこの自己訂正の往復で担保されています。

### rust-skills-knowledge-pack

REV-C の主戦場 — **高負荷送信基盤 / Leptos Web Builder / TUI AI Agent / E2EE Infrastructure / Social Psychometrics CRM** — を Rust の最新安定版で支えるためのナレッジパック (v0.1.2 / 2026-04-29)。**1,000 行超の master file**、80+ crate に `A001`〜`U010` の ID を振った Crate Register、12 の feature flag、5 つの実装プレイブックで構成。

設計の核は **「標準レーンを強くしつつ、ホットパスだけ別レーンで焼き切る」**。

```text
Standard lane:
  tokio / reqwest / tower / leptos / ratatui / cpal / rustls / sqlx / surrealdb / tracing
Limit-break lane:
  bytes / hyper / governor / ringbuf / rkyv / simd-json / tikv-jemallocator
  chacha20poly1305 / ed25519-dalek / qdrant-client / tantivy / lancedb
R&D lane:
  graviola / tokio-uring / monoio / distx / hpke-rs
```

非交渉ルールから一例:

> **callback 内・lock 内・hot path 内でやってはいけないこと** — `cpal` callback 内での alloc、mutex、JSON parse、HTTP 送信、ログ大量出力。`DashMap` guard 保持中の `.await`。`Mutex` guard 保持中の DB アクセス、ネットワーク I/O、重い format 処理。request ごとの `reqwest::Client::new()`。`tokio::spawn` の無制限連打。大量データへの `fetch_all` / `collect()` の早すぎる呼び出し。**暗号 nonce の再利用、鍵用途混同、秘密鍵の Debug 出力。**

> **速さは p50/p95/p99、RSS、allocations/op、WASM size、callback duration、handshake latency、vector search latency で判断する。**

5 ドメイン: 超高負荷クローラー & フォーム送信基盤 / Leptos+WASM フルスタック Web ビルダー / TUI+生音声+LLM インサイドセールス AI エージェント / 秘匿 E2EE インフラ / AI/Data/Search 基盤。10 レーン: Async I/O / Crawler+Form / Browser CDP / Extreme Linux I/O (tokio-uring / monoio) / Leptos+WASM / TUI Realtime Audio / E2EE+Crypto / AI+Data+Search / Memory+Performance / Observability+Update Governance。

旧監査では `ringbuf 0.4.9` が yanked であることを docs.rs と crates.io の差分から特定し `0.4.8` に切り戻すような、**実測しないと出てこない補正**が監査ログに残されています。現行 v0.1.2 の master では再確認後の採用値として `ringbuf 0.5.0` を置いています。

### go-skills-knowledge-pack

Go 1.26.2 を起点に、**12 分野 × 採用レベル 5 段階（Core / Adopt / Watch / R&D / Hold）× 40+ module register**。`net/http` / `chi` / `connect-go` / `pgx/v5` / `sqlc` / `ent` / `openai-go/v3` / MCP go-sdk v1.6.0 / `cobra` / `bubbletea/v2` / `slog` / OTel / Prometheus / NATS。Go module の semantic import versioning 上、`v0.x` は「最新タグ」でも stable ではないため Core ではなく Adopt / Watch / R&D として扱う、というポリシーを明記。

```go
tr := &http.Transport{
    MaxIdleConns:        2048,
    MaxIdleConnsPerHost: 64,
    MaxConnsPerHost:     128,
    IdleConnTimeout:     90 * time.Second,
    TLSHandshakeTimeout: 10 * time.Second,
}
client := &http.Client{Transport: tr, Timeout: 30 * time.Second}
```

抽象論ではなく即コピペできる Transport tuning まで降りています。「Treat browser automation as escalation, not default crawling.」のような原則も全分野で一貫。

12 ファイルのリファレンス（runtime-toolchain / api-backend-rpc / highload-crawler-browser / data-persistence-search / ai-agents-mcp / crypto-security-e2ee / messaging-workflows / tui-cli-audio / performance-memory / observability-governance / testing-release / multiskill-interop）+ master + update-prompt + sources + audit log。

### typescript-skills-knowledge-pack

Node 24 LTS / pnpm 11.0.6 / TypeScript 最新を起点に、**10 分野 × 120+ npm package entries** (v0.1.2 / 2026-05-06)。React 19 / Next.js / TanStack Router / Tailwind v4 / Hono / Fastify / Undici v8.2.0 / Crawlee / Playwright / Cloudflare Workers / OpenAI SDK / AI SDK / LangGraph / MCP / jose / `@noble/*` / Drizzle / Kysely / BullMQ / Vitest / Biome / pino / OTel / Sentry。

`pnpm` は npm `latest` (v10.x) と `latest-11` (v11.0.6) が並立することを把握し、`packageManager` を明示的に pin する運用。`@types/node` は production の Node メジャーに合わせ、blind に latest を使わない。

> **TypeScript の限界を認める** — 低遅延音声、暗号鍵ホットパス、SIMD/zero-copy は Rust sidecar / N-API / WASM に逃がす。

— TS Skill のオーナーが「TS では無理な領域」を明示しているからこそ、RustSkills との境界が信頼できます。`v0.1.2` は v0.1.1-strict のガバナンスモデルを保ちつつ、master-first 構造への正規化と同日 registry 再確認を反映。

## Multi-Skill Interop

8 つの Skill を AI エージェントに同時にロードしたとき、衝突は次の優先順位で解決します。

1. **Repo hygiene** — `naming-normalization-guard` は新規追加 / リネーム時に最初に走る
2. **Security / compliance / secrets** — platform-specific official docs と `AGENTS.md` が言語選好を override
3. **Cloudflare runtime behavior** — CloudflareSkills が言語固有仮定を override
4. **Supabase product behavior** — SupabaseSkills が言語固有 client 仮定を override
5. **Language implementation details** — RustSkills / TypeScriptSkills / GoSkills が割り当てられた runtime boundary 内で適用
6. **Performance claims** — 採用前にベンチマーク or production metric が必要

集大成は単なる束ではなく、**co-loadable な統一ガバナンス体系** として設計されています。

## Repo Standards

### Knowledge Pack Standard (master-first)

```text
<language>-skills-knowledge-pack/
  README.md                     # 人間向け概要
  AGENTS.md                     # repo に置く AI 運用指示
  SKILL.md                      # Codex / Claude / ChatGPT Skills 入口
  <language>-skills-master.md   # 単一の真実源
  <language>-skills-sources.md  # 公式ソース・registry 確認先
  <language>-skills-update-prompt.md
  <language>-skills-pack-audit-YYYY-MM-DD-vX.Y.Z.md
  references/                   # master から分野別に切り出した詳細
```

監査ファイルは版付きの `*-pack-audit-YYYY-MM-DD-vX.Y.Z.md` を**現行監査**として扱います。無版または旧 version の audit は履歴互換で残してよいが、現在の採用判断には使いません。

### Deploy Guard Target Standard (gate-first)

```text
<service>-deploy-guard/
  README.md
  SKILL.md
  prompts/
  references/
    source-links.md                 # or <service>-source-links.md
    <service>-cost-security-checklist.md / <service>-checklist.md
    <service>-risk-matrix.md
    <service>-deploy-report-template.md / <service>-report-template.md
  scripts/
```

## Numbers

| | |
|---|---|
| Skills (合計) | **8** (Deploy Guard 4 + Knowledge Pack 3 + Repo Hygiene 1) |
| Rust crates pinned | **80+** (`A001`〜`U010` の Decision Records) |
| npm package entries | **120+** (Runtime / Frontend / API / AI / Crypto / Data / Testing) |
| Go modules pinned | **40+** (Core / Adopt / Watch / R&D / Hold) |
| Cost scenarios | guard ごとに `expected` / `10x` / bot-abuse / crawler / bug-loop / migration-failure 等を使い分け |
| Performance metrics | **8** 軸 (p50/p95/p99 / RSS / allocations/op / WASM size / callback duration / handshake latency / vector search latency) |
| Self-audit roundtrips | TS は v0.1.0 → v0.1.1-strict → v0.1.2 で版を重ね、誤訂正は再確認で巻き戻している |

## Install

必要なスキルだけ `.agents/skills/` にコピーします。

```bash
mkdir -p .agents/skills
cp -R cloudflare-deploy-guard          .agents/skills/
cp -R codex-app-server-guard           .agents/skills/
cp -R payload-cms-deploy-guard         .agents/skills/
cp -R supabase-deploy-guard            .agents/skills/
cp -R rust-skills-knowledge-pack       .agents/skills/
cp -R go-skills-knowledge-pack         .agents/skills/
cp -R typescript-skills-knowledge-pack .agents/skills/
```

`naming-normalization-guard` はリポジトリ運用 Skill のため、すでに `.agents/skills/naming-normalization-guard/` に配置されています。

## Usage

各ディレクトリの `README.md` と `SKILL.md` を入口にしてください。言語系の実判断では `*-master.md` を正本として読みます。Deploy Guard 系の典型的な実行は次のとおりです。

```bash
python .agents/skills/cloudflare-deploy-guard/scripts/cloudflare-static-risk-scan.py . --markdown
python .agents/skills/codex-app-server-guard/scripts/codex-app-server-static-risk-scan.py . --markdown --fail-on high
python .agents/skills/payload-cms-deploy-guard/scripts/payload-cms-static-risk-scan.py . --markdown --fail-on high
python .agents/skills/supabase-deploy-guard/scripts/supabase-static-risk-scan.py . --markdown --fail-on high
```

エージェントには、これらの出力に加えて DB 監査 / コスト試算 / チェックリストをすべて通したうえで `DEPLOY: GO / NO-GO` を出すことを要求します。`NO-GO` のときは原因と修正案を返すまでデプロイを止めます。

### Verdict format

```text
DEPLOY: GO
DEPLOY: NO-GO  reason=rls-missing-on-public-table  table=user_psych_profiles
```

## Snapshot Philosophy

知識は腐ります。だから日付を打ちます。

すべてのナレッジパックは `*-pack-audit-YYYY-MM-DD-vX.Y.Z.md` という監査ファイルを同梱し、自分で自分のドキュメントを採点します。価格・仕様・API・CLI オプションは変わるため、本番判断の前に必ず公式ドキュメントで再確認します。

更新時は各パックの `*-update-prompt.md` を Deep Research に渡し、公式ソースと release notes を起点に diff・audit・snapshot します。**訂正そのものを訂正することがある** — TypeScript v0.1.2 が示したとおり、registry 再確認は片道ではありません。

## Naming Policy

- ディレクトリと通常ファイルは `kebab-case`
- Codex Skill 規約に従い、各スキルの入口は `SKILL.md`
- パッケージ単位の説明は `README.md`、エージェント向け運用指示は `AGENTS.md`
- 参照資料は `references/`、実行補助は `scripts/`、貼り付け用プロンプトは `prompts/`
- リネーム・追加時は `naming-normalization-guard` を必ず通す

## Maintenance

- 価格 / 仕様 / API / CLI オプションは変わるため、本番判断の前に必ず公式ドキュメントで再確認する
- ナレッジパック更新時は `*-update-prompt.md` を使い、公式ソースと release notes を優先する
- `scripts/` のファイル名や README のコマンド例を変更したら、`rg` で旧名が残っていないか必ず確認する（`naming-normalization-guard` の役割）
- Skills は「一般論」ではなく **SasukeTorii の開発判断を圧縮したもの** として更新する。実プロジェクトで得た失敗、勝ち筋、言語ハック、agent への指示の癖をここへ戻す

## Author

**鳥居 佐助 (Sasuke Torii)** — REV-C Asia CEO 兼 Developer。Tokyo。

日本発で、世界基準のエンジニアリングをやりたい人間です。本リポジトリは CCTeam / Social Psychometrics CRM / 高負荷送信基盤 / Leptos Web Builder / TUI AI Agent / E2EE Infrastructure を支えるために実プロジェクトから抽出したもので、私が普段下している判断を AI エージェントに渡せる形に圧縮してあります。採用・パートナーシップ・技術相談はリポジトリ Issues か GitHub プロフィール経由で。

## Acknowledgements

巨人の肩に乗って判断を下しています。— Anthropic Claude Code / OpenAI Codex / Cloudflare / Supabase / Payload CMS / Rust Foundation / Go team / TC39 / Node.js project / OpenJS Foundation / pnpm / Astral (uv, ruff) / tokio-rs / leptos-rs / `@noble/*` / jose / docs.rs / crates.io / RustSec / OSV.

独自判断はすべて鳥居佐助の責任です。

## License

MIT License. See [`LICENSE`](LICENSE).
