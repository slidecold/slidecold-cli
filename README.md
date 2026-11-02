# SlideCold CLI

[![ci](https://github.com/slidecold/slidecold-cli/actions/workflows/ci.yml/badge.svg)](https://github.com/slidecold/slidecold-cli/actions/workflows/ci.yml)

`slidecold` is the command-line interface for the [SlideCold](https://slidecold.com) API (Instagram cold-DM outreach).
It has a command for every API operation, generated from the operation table of the
[Go SDK](https://github.com/slidecold/slidecold-go).

## Install

**Release binaries.** Download the archive for your OS and architecture (Linux, macOS, Windows; amd64, arm64) from
the [latest release](https://github.com/slidecold/slidecold-cli/releases/latest), extract it, and put `slidecold` on
your `PATH`. Every archive and `checksums.txt` carries signed build provenance; verify a download with the
[GitHub CLI](https://cli.github.com):

```sh
gh attestation verify slidecold_0.1.0_darwin_arm64.tar.gz --repo slidecold/slidecold-cli
```

**Go** (>= 1.22):

```sh
go install github.com/slidecold/slidecold-cli/cmd/slidecold@latest
```

## Log in

Create an API key (`cs_...`) at <https://slidecold.com/settings>, then:

```sh
slidecold login                 # prompts for the key, checks it, and saves it
slidecold login --key cs_...    # non-interactive
slidecold logout                # removes the saved key
```

The key is looked up in this order: `--api-key`, the `SLIDECOLD_API_KEY` environment variable, then the config file.
The base URL resolves the same way (`--base-url`, `SLIDECOLD_BASE_URL`, config `baseUrl`), defaulting to
`https://slidecold.com/api/v1`.

The config file `{"apiKey": "cs_...", "baseUrl": "..."}` is shared with the SlideCold SDKs (Go, Node, Python), so
after `slidecold login` they find the key too. It is written with `0600` permissions at:

| Condition | Path |
|---|---|
| `XDG_CONFIG_HOME` is set | `$XDG_CONFIG_HOME/slidecold/config.json` |
| `~/.config/slidecold/` exists (any OS) | `~/.config/slidecold/config.json` |
| Windows | `%AppData%\slidecold\config.json` |
| Linux, macOS | `~/.config/slidecold/config.json` |

## Usage

```text
slidecold <resource> <action> [args] [--param value] [--data JSON|@file|@-] [--file path]
```

```sh
slidecold campaigns list --folder-id root
slidecold campaigns stats cmp_123
slidecold campaigns create --data @campaign.json
slidecold campaigns update cmp_123 --data '{"status":"PAUSED"}'
slidecold audiences export aud_123 > members.csv  # raw CSV
slidecold audiences import-members aud_123 --file members.csv
slidecold conversations list --status replied --limit 50
slidecold conversations send-photo conv_123 --file photo.jpg
slidecold accounts login-status job_123
slidecold version
```

Resources are `campaigns`, `audiences`, `conversations`, `accounts`, `proxies` and `folders`. `slidecold --help`,
`slidecold campaigns --help` and `slidecold campaigns create --help` (with endpoint, required scope and the body's
JSON Schema) document the rest. Output is pretty-printed JSON (raw CSV for exports); errors go to stderr with exit
code 1.

## Development

```sh
go vet ./... && go test -race ./...
go run github.com/goreleaser/goreleaser/v2@latest check
```

To build against a local checkout of the SDK, put both repos side by side and create a `go.work` (gitignored):

```sh
go work init . && go work edit -replace github.com/slidecold/slidecold-go=../slidecold-go
```

Releasing: push a `vX.Y.Z` tag. GoReleaser builds the archives and the release, then GitHub attests them. The
Homebrew cask is published to `slidecold/homebrew-tap` only once that repo and the `HOMEBREW_TAP_TOKEN` secret exist
(see `.goreleaser.yaml`).

## License

MIT
