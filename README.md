# CLIProxyAPI Global Usage Build

This workspace contains the local source checkout and custom build used on 2026-05-02 to restore CLIProxyAPI global usage statistics for Cameron's Raspberry Pi CPA service.

## Scope

- Base: `router-for-me/CLIProxyAPI` `v6.10.0`
- Custom branch: `cameron/global-usage-persist`
- Custom version: `6.10.0-cameron.1`
- Remote service: `/home/daishuge/project/cliproxyapi` on `m.daishuge.win`

## Change

The upstream `v6.10.0` commit `18bb9c31` removed usage tracking and logging functionality. This local build reverts that removal so `/v0/management/usage`, `/v0/management/usage/export`, and `/v0/management/usage/import` are available again.

The remote `config.yaml` keeps `usage-statistics-enabled: true` and now pins the current management panel with `remote-management.disable-auto-update-panel: true` so a later panel auto-update does not silently remove the visible usage surface again.

## Validation

- `go test ./internal/usage ./internal/api/handlers/management ./internal/redisqueue ./test -run 'Usage|APIKeyUsage|Config|Management'`
- Local smoke on `127.0.0.1:18317`: `/usage` import/read succeeded and `usage-statistics-enabled` persisted across restart.
- RPi canary on `127.0.0.1:18317`: `/v1/models` returned `200`; a `gpt-5.5-low-fast` chat request returned `200` and usage totals increased to one request.
- RPi production on `127.0.0.1:8317`: service active, `/v1/models` returned `200`, and a `gpt-5.5-low-fast` chat request returned `200`.

Full `go test ./...` was also run. It still has upstream-unrelated failures in `internal/registry` and `internal/runtime/executor`; the restored usage/management paths passed.
