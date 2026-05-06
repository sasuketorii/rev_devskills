# API / Crawler / High-load Reference

## Core stack

- API: Hono or Fastify
- HTTP hot path: `undici` v8.1.0
- Static HTML: Cheerio / parse5 / htmlparser2
- JS-rendered pages: Playwright / Crawlee only when required
- Concurrency: p-limit / p-queue
- Rate limiting: bottleneck / framework-specific rate limit
- Policy: robots-parser, allowlist, audit log

## Critical correction

`undici` and `undici-types` are different packages.

- Runtime package: `undici` v8.1.0 at snapshot.
- Type package: `undici-types` v8.2.0 at snapshot.

Do not copy `undici-types` version into the runtime package register.

## Architecture

```text
authorized job
  -> allowlist/robots/consent check
  -> bounded queue
  -> per-host/account limiter
  -> undici Client/Pool
  -> parser/browser fallback
  -> normalized event
  -> audit sink
```

## Rules

- Do not spawn unbounded promises.
- Do not create a browser per URL.
- Use static parsing first; browser automation is an expensive fallback.
- Always set timeout, max body size, retry budget, and kill switch.
