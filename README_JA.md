# Playful Proxy API Panel (PPAP)

[English](README.md) | [中文](README_CN.md) | 日本語

**PPAP は、自ホスト向けの CLIProxyAPI 互換 fork です。管理パネル、永続化される使用量分析、コスト推定、Codex 向け model alias をまとめて提供します。**

[`router-for-me/CLIProxyAPI`](https://github.com/router-for-me/CLIProxyAPI) の OpenAI/Gemini/Claude/Codex 互換 proxy surface を保ちながら、日常運用で必要になる usage snapshot、request/cost metrics、backend と同じ tag で配布される管理パネル、より安全な thinking strength alias を追加しています。

素の CLIProxyAPI が必要なら上流を使ってください。  
使用量、latency、cache hit、Codex strength routing をローカルで見たいなら PPAP を使ってください。

## PPAP の違い

- **使用量分析を内蔵**: `/v0/management/usage`、import/export、local snapshot persistence、cache hit rate、first-byte latency、average latency、TPS、token breakdown、model/API rollup。
- **パネルと backend を同時 release**: frontend source は [`web/management-panel`](web/management-panel) にあり、各 release に同じ tag で build された `management.html` が含まれます。
- **Codex を主要 workflow として扱う**: OpenAI Codex OAuth、GPT model routing、Spark pricing estimate、thinking strength alias をこの fork で保守します。
- **thinking strength の書き方を統一**: `model(high)` と `model-high` の両方に対応し、`low`、`medium`、`high`、`xhigh` を扱います。explicit alias と exact model name が優先されます。
- **上流互換を維持**: 競合しない上流更新は取り込みます。Redis usage queue retention も含まれ、PPAP の usage persistence は維持されます。

## Core Features

- OpenAI/Gemini/Claude/Codex-compatible API endpoints
- OAuth login for OpenAI Codex and Claude Code
- Streaming and non-streaming responses
- Function calling, tools, and multimodal input
- Multi-account routing and load balancing
- Gemini CLI, AI Studio Build, Claude Code, OpenAI Codex, and Amp CLI support
- OpenAI-compatible upstream providers such as OpenRouter through config
- Reusable Go SDK

## Quick Start

この repository の [latest Release](https://github.com/daishuge/playful-proxy-api-panel/releases/latest) から platform に合う archive をダウンロードし、展開して local config で起動します。

```bash
cp config.example.yaml config.yaml
./cli-proxy-api -config ./config.yaml
```

Default HTTP port は `8317` です。

Docker で自ホストする場合は、この repository から build してください。

```bash
git clone https://github.com/daishuge/playful-proxy-api-panel.git
cd playful-proxy-api-panel
cp config.example.yaml config.yaml
mkdir -p auths logs
docker compose up -d --build
```

`config.yaml`、`.env`、OAuth files、API keys、auth directories、logs、generated stores は git に commit しないでください。

## Configuration Notes

[`config.example.yaml`](config.example.yaml) から始めてください。PPAP でよく使う設定:

- `usage-statistics-enabled`: built-in usage snapshot を有効化。
- `usage-statistics-path`: snapshot file の保存先を指定。
- `redis-usage-queue-retention-seconds`: Redis usage queue retention を調整。
- `oauth-model-alias`: friendly model alias を定義し、legacy config style も維持。

thinking levels を宣言している model では、PPAP は次のような aliases を自動で公開できます。

```text
gpt-5.3-codex-spark-low
gpt-5.3-codex-spark-medium
gpt-5.3-codex-spark-high
gpt-5.3-codex-spark-xhigh
```

従来の parentheses style も使えます。

```text
gpt-5.3-codex-spark(high)
```

## Codex Spark Pricing

PPAP pricing data には `gpt-5.3-codex-spark` が含まれています。公式 preview pricing が安定するまでは、`gpt-5.3-codex` の推定 rate を使います。

References:

- [Introducing GPT-5.3-Codex-Spark](https://openai.com/index/introducing-gpt-5-3-codex-spark/)
- [Codex rate card](https://help.openai.com/en/articles/11369540-codex-rate-card)
- [OpenAI API pricing](https://openai.com/api/pricing/)

## Management

- Management panel source: [`web/management-panel`](web/management-panel)
- Management API docs: [help.router-for.me/management/api](https://help.router-for.me/management/api)
- Usage endpoints: `/v0/management/usage`, `/v0/management/usage/export`, `/v0/management/usage/import`
- Amp CLI guide: [help.router-for.me/agent-client/amp-cli.html](https://help.router-for.me/agent-client/amp-cli.html)

Release asset の `management.html` は backend binaries と同じ tag から build されます。

## SDK And Docs

- SDK usage: [docs/sdk-usage.md](docs/sdk-usage.md)
- Advanced executors and translators: [docs/sdk-advanced.md](docs/sdk-advanced.md)
- Access: [docs/sdk-access.md](docs/sdk-access.md)
- Watcher: [docs/sdk-watcher.md](docs/sdk-watcher.md)
- Custom provider example: [`examples/custom-provider`](examples/custom-provider)

## License

MIT. See [LICENSE](LICENSE).
