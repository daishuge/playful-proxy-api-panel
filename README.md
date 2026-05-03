# Playful Proxy API Panel (PPAP)

English | [中文](README_CN.md) | [日本語](README_JA.md)

**PPAP is a self-hosted, upstream-compatible CLIProxyAPI fork with a built-in management panel, persistent usage analytics, and Codex-focused model ergonomics.**

It keeps the familiar OpenAI/Gemini/Claude/Codex-compatible proxy surface from [`router-for-me/CLIProxyAPI`](https://github.com/router-for-me/CLIProxyAPI), then adds the pieces that matter when you run it every day: usage snapshots, cost estimates, panel assets released with the backend, and safer thinking-strength aliases.

Use upstream CLIProxyAPI when you want the vanilla project. Use PPAP when you want the same proxy style with more local visibility and a tighter operations loop.

## What Makes PPAP Different

- **Usage analytics built in**: restored `/v0/management/usage`, import/export endpoints, persistent local snapshots, cache hit rate, first-byte latency, average latency, TPS, token breakdowns, and per-model/per-API rollups.
- **Panel and backend released together**: the management panel source lives in [`web/management-panel`](web/management-panel), and each release ships the matching `management.html`.
- **Codex is treated as a primary workflow**: OpenAI Codex OAuth, GPT model routing, Spark pricing estimation, and thinking-strength aliases are maintained in this fork.
- **Thinking aliases are predictable**: both `model(high)` and `model-high` work for `low`, `medium`, `high`, and `xhigh`; explicit aliases and exact model names stay higher priority.
- **Upstream compatibility is still the baseline**: upstream fixes are merged where they do not conflict with PPAP behavior. Recent Redis usage queue retention support is included.

## Core Features

- OpenAI/Gemini/Claude/Codex-compatible API endpoints for CLI models
- OAuth login for OpenAI Codex and Claude Code
- Streaming and non-streaming responses
- Function calling/tools and multimodal input
- Multi-account routing and load balancing
- Gemini CLI, AI Studio Build, Claude Code, OpenAI Codex, and Amp CLI support
- OpenAI-compatible upstream providers such as OpenRouter through config
- Reusable Go SDK for embedding the proxy

## Quick Start

Download the [latest PPAP release](https://github.com/daishuge/playful-proxy-api-panel/releases/latest), extract the archive for your platform, then start with a local config file:

```bash
cp config.example.yaml config.yaml
./cli-proxy-api -config ./config.yaml
```

The default HTTP port is `8317`.

For Docker self-hosting, build from this repository so the container contains PPAP-specific code:

```bash
git clone https://github.com/daishuge/playful-proxy-api-panel.git
cd playful-proxy-api-panel
cp config.example.yaml config.yaml
mkdir -p auths logs
docker compose up -d --build
```

Keep `config.yaml`, `.env`, OAuth files, API keys, auth directories, logs, and generated stores out of git.

## Configuration Notes

Start from [`config.example.yaml`](config.example.yaml). The most useful PPAP-specific settings are:

- `usage-statistics-enabled`: enable built-in usage snapshots.
- `usage-statistics-path`: optionally move the usage snapshot away from the config directory.
- `redis-usage-queue-retention-seconds`: tune Redis usage queue retention when Redis usage queueing is enabled.
- `oauth-model-alias`: define friendly model aliases while preserving old config compatibility.

For models that declare thinking levels, PPAP can expose automatic aliases such as:

```text
gpt-5.3-codex-spark-low
gpt-5.3-codex-spark-medium
gpt-5.3-codex-spark-high
gpt-5.3-codex-spark-xhigh
```

The older parenthesized style still works:

```text
gpt-5.3-codex-spark(high)
```

## Codex Spark Pricing

`gpt-5.3-codex-spark` is included in PPAP pricing data for local usage-cost estimation. Until official preview pricing settles, PPAP temporarily estimates it with the `gpt-5.3-codex` rate.

References:

- [Introducing GPT-5.3-Codex-Spark](https://openai.com/index/introducing-gpt-5-3-codex-spark/)
- [Codex rate card](https://help.openai.com/en/articles/11369540-codex-rate-card)
- [OpenAI API pricing](https://openai.com/api/pricing/)

## Management

- Management panel source: [`web/management-panel`](web/management-panel)
- Management API docs: [help.router-for.me/management/api](https://help.router-for.me/management/api)
- Usage endpoints: `/v0/management/usage`, `/v0/management/usage/export`, `/v0/management/usage/import`
- Amp CLI guide: [help.router-for.me/agent-client/amp-cli.html](https://help.router-for.me/agent-client/amp-cli.html)

The release asset `management.html` is built from the same tag as the backend binaries, so a running PPAP instance can point its panel updater at this repository.

## SDK And Docs

- SDK usage: [docs/sdk-usage.md](docs/sdk-usage.md)
- Advanced executors and translators: [docs/sdk-advanced.md](docs/sdk-advanced.md)
- Access: [docs/sdk-access.md](docs/sdk-access.md)
- Watcher: [docs/sdk-watcher.md](docs/sdk-watcher.md)
- Custom provider example: [`examples/custom-provider`](examples/custom-provider)

## License

MIT. See [LICENSE](LICENSE).
