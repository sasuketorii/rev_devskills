# GoSkills Knowledge Pack v0.1.1-strict

REV-C / Sasuke 向けの Go 技術体系ナレッジパックです。RustSkills / TypeScriptSkills と同じ設計思想で、Goの最新安定版・公式ドキュメント・実運用CIを前提に構成しています。

## Files

- `go-skills-master.md` — GoSkills全体のMaster。
- `SKILL.md` — Codex / Claude系Agent Skills向けの軽量Skillハブ。
- `go-skills-update-prompt.md` — ChatGPT / Claude / Gemini Deep Researchで更新するためのプロンプト。
- `go-skills-sources.md` — 公式ドキュメント・pkg.go.dev・GitHub Release中心のソースインデックス。
- `AGENTS.md` — Go repoに置くAIエージェント向け運用指示。
- `go-skills-pack-audit-2026-05-06.md`
- `go-skills-pack-audit-2026-05-06-v0.1.1.md` — 初期監査レポート。
- `references/` — 分野別の詳細ガイド。

## Core principle

GoSkillsは「全部最新安定版を使う」を前提にします。ただしGo moduleのv0.xは、最新タグであってもsemantic import versioning上はstable扱いしません。Core / Adopt / Watch / R&D / Hold を分けて、実repoの `go.mod` と `go.sum` で検証します。

## Recommended first commands

```bash
go version
go env GOVERSION
go list -m -u -json all
go mod tidy
go mod verify
go test ./...
go test -race ./...
go vet ./...
govulncheck ./...
gosec ./...
golangci-lint run
staticcheck ./...
```

## Best next step

実際のGoプロジェクトrepoにこのPackを置いた後、`go.mod` / `go.sum` / CI / benchmark / pprof 結果を突き合わせて `v0.1.1-strict` に更新してください。


## v0.1.1-strict audit note

v0.1.1-strict fixes strict fact-check issues from v0.1.0-strict:

- Corrected Go module paths: `go.uber.org/zap`, `go.uber.org/ratelimit`.
- Updated versions: `github.com/klauspost/compress v1.18.6`, `github.com/qdrant/go-client v1.17.1`, `golangci-lint v2.12.1`.
- Added `references/multiskill-interop.md` for Cloudflare / Supabase / Rust / TypeScript / Go co-loading.
