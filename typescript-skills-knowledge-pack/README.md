# TypeScriptSkills Knowledge Pack

- **Version**: `v0.1.1-strict`
- **Snapshot date**: 2026-05-06 JST
- **Scope**: REV-C Inc. / Sasuke の TypeScript 技術体系。RustSkills の思想を TypeScript / Node.js / Bun / Deno / Web / AI Agent / E2EE / Data Platform へ移植する。
- **Policy**: 原則は **latest stable**。ただし Node.js は「本番 latest LTS + CI latest Current」、pnpm は Node engine と registry dist-tag の差分を明示して運用する。

## Files

```text
typescript-skills-knowledge-pack/
  README.md
  AGENTS.md
  SKILL.md
  typescript-skills-master.md
  typescript-skills-sources.md
  typescript-skills-update-prompt.md
  typescript-skills-pack-audit-2026-05-06-v0.1.1.md
  references/
    runtime-toolchain.md
    frontend-fullstack.md
    api-crawler-highload.md
    edge-runtime-workers.md
    ai-agent-realtime.md
    crypto-e2ee-security.md
    data-state-search.md
    performance-hotpath.md
    testing-release-quality.md
    observability-governance.md
```

## Baseline

- Production runtime: **Node.js latest LTS**
- Compatibility/R&D runtime: **Node.js latest Current**, Bun, Deno
- Package manager: **pnpm latest stable compatible with production Node baseline**
- Language: TypeScript latest stable
- Web: React / Next.js / React Router / Vite / TanStack / Tailwind
- API: Hono / Fastify / Undici / OpenAPI contract generation
- Browser automation: Playwright / Crawlee / Cheerio / parse5
- Edge: Cloudflare Workers / Wrangler / Miniflare where required
- AI: OpenAI SDK / OpenAI Agents SDK / Vercel AI SDK / LangGraph / MCP
- Web Builder: Yjs / Liveblocks / tldraw / Lexical / dnd-kit / Floating UI / Radix
- Data: Drizzle / Kysely / Postgres / Redis / BullMQ / Qdrant / LanceDB
- Observability: pino / OpenTelemetry / Prometheus / Sentry / web-vitals
- Release quality: tsd / publint / @arethetypeswrong/cli / dependency-cruiser / knip
- Supply chain: pnpm minimumReleaseAge, OSV-Scanner, npm audit, Renovate, trusted publishing/provenance

## How to use

1. `SKILL.md` を Codex / Claude Code / ChatGPT Skills の入口として読む。
2. 詳細判断は `typescript-skills-master.md` と `references/*` を参照する。
3. 更新時は `typescript-skills-update-prompt.md` を ChatGPT / Claude / Gemini Deep Research に貼り、公式ソース中心に再調査する。
4. package version は latest stable を基本にし、pre/rc/canary/next は R&D 隔離する。
5. 実repoでは必ず lockfile、audit、typecheck、lint、test、E2E、bundle/bench を通す。

## Safety

Crawler、form sender、browser automation、AI agent、E2EEは、許可済み業務・同意済みデータ処理・自社管理対象・正当なQA/負荷試験に限定する。外部対象には必ず rate limit、allowlist、audit、kill switch を設計する。

## v0.1.1 strict audit delta

v0.1.0 は思想として強かったが、厳密監査で以下を修正した。

- `README.md` と `references/` の構造不一致を解消。
- `undici v8.2.0` を `undici v8.1.0` に修正し、`undici-types v8.2.0` との取り違えを明記。
- `@swc/core v1.15.33` を `@swc/core v1.15.32` に修正し、`@swc/wasm v1.15.33` との取り違えを明記。
- pnpm は Node 24 LTS 前提なら v11.0.6 を標準候補にし、npm `latest` dist-tag が v10系を指す可能性を注意事項に昇格。
- `@types/node` は npm latest だけでなく、本番 Node major に合わせるルールを追加。
- `tsd` / `publint` / `@arethetypeswrong/cli` / `web-vitals` / `happy-dom` / `wrangler` / `miniflare` を追加。
