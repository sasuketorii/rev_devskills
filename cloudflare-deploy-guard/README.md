# cloudflare-deploy-guard

Codex/Agent Skills用のCloudflareデプロイ前ガードです。

推奨配置:

```bash
mkdir -p .agents/skills
cp -R cloudflare-deploy-guard .agents/skills/
```

使い方:

1. Cloudflareにデプロイ・設定変更する前にこのスキルを呼び出す。
2. 静的スキャンを実行する。
3. Cloudflare MCP/APIで読み取り棚卸しを行う。
4. `DEPLOY: GO` になるまで変更系MCP/APIやdeployを実行しない。

```bash
python .agents/skills/cloudflare-deploy-guard/scripts/cloudflare-static-risk-scan.py . --markdown
```
