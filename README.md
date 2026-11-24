> This is a demo project
# Dify Terraform Provider (Unofficial)

<p align="center">
  <img src="img/cover.png" alt="Dify Terraform Provider — AI apps, credentials, plugins & keys as code">
</p>

[![Tests](https://github.com/wylswz/dify-unofficial-terraform-provider/actions/workflows/test.yml/badge.svg)](https://github.com/wylswz/dify-unofficial-terraform-provider/actions/workflows/test.yml)
[![Go Report Card](https://goreportcard.com/badge/github.com/wylswz/dify-unofficial-terraform-provider)](https://goreportcard.com/report/github.com/wylswz/dify-unofficial-terraform-provider)
[![License: MIT](https://img.shields.io/badge/License-MIT-brightgreen.svg)](LICENSE)

A Terraform provider for managing [Dify](https://dify.ai) applications, datasets, model and tool provider credentials, plugin installs, and app API keys as infrastructure-as-code.

The provider talks to Dify's user-scoped OpenAPI (`/openapi/v1`) and authenticates with a **Personal Access Token (PAT)**.

## Requirements

- [Terraform](https://www.terraform.io/downloads.html) >= 1.0
- [Go](https://go.dev/dl/) — see `go.mod` for the required version (to build the provider)
- A Dify instance exposing the `/openapi/v1` API. The provider is acceptance-tested
  against upstream [langgenius/dify](https://github.com/langgenius/dify) at the
  commit pinned in [`scripts/dify-commit`](scripts/dify-commit).

## Personal Access Token

Create a PAT in the Dify console under **Account Settings > Personal Access Tokens**. The token needs access to the workspace you want to manage. Set it via `DIFY_TOKEN` or a sensitive Terraform variable rather than hardcoding it. Resource secrets can still be present in Terraform state and plan files; treat both as sensitive.

## Installing the Provider

Until the provider is published on the Terraform Registry, install a release binary into the local plugin mirror — this one-liner detects your OS/arch (Linux/macOS, amd64/arm64):

```bash
VERSION=0.0.1; VERSION=${VERSION#v}; OS=$(uname -s | tr '[:upper:]' '[:lower:]'); ARCH=$(uname -m | sed 's/x86_64/amd64/;s/aarch64/arm64/'); \
  curl -sL "https://github.com/wylswz/dify-unofficial-terraform-provider/releases/download/v${VERSION}/terraform-provider-dify_${VERSION}_${OS}_${ARCH}.zip" -o /tmp/dify-tf.zip \
  && mkdir -p "$HOME/.terraform.d/plugins/registry.terraform.io/wylswz/dify/${VERSION}/${OS}_${ARCH}" \
  && unzip -o /tmp/dify-tf.zip -d "$HOME/.terraform.d/plugins/registry.terraform.io/wylswz/dify/${VERSION}/${OS}_${ARCH}" \
  && rm /tmp/dify-tf.zip
```

Or build from source with `make install-local` (see [Developing the Provider](#developing-the-provider)).

## Quick Start

```hcl
terraform {
  required_providers {
    dify = {
      source  = "registry.terraform.io/wylswz/dify"
      version = "0.0.1"
    }
  }
}

provider "dify" {
  host         = "https://dify.example.com"
  token        = var.dify_token
  workspace_id = var.dify_workspace_id
}
```

## Documentation and Examples

- [Provider configuration](docs/index.md)
- [Resource reference](docs/resources/) and [data source reference](docs/data-sources/)
- [Complete example](examples/complete/) for credentials, tools, and an agent workflow
- [Knowledge base example](examples/knowledge-base/) for datasets, documents, and retrieval

## Developing the Provider

```bash
make build      # compile the provider
make install    # install into $GOBIN for dev_overrides
make test       # unit tests (httptest-based, no Dify needed)
make testacc    # acceptance tests against a live Dify (needs TF_ACC=1 + DIFY_*)

# or end-to-end against a real upstream Dify stack pinned to scripts/dify-commit:
bash scripts/dify-acceptance.sh [path/to/local/dify-repo]
make lint       # golangci-lint
make generate   # terraform fmt examples + regenerate docs/ via tfplugindocs
```

See [`CONTRIBUTING.md`](CONTRIBUTING.md) for the full development workflow, and [`CHANGELOG.md`](CHANGELOG.md) for release notes. Releases are automated with GoReleaser — see [`.github/workflows/release.yml`](.github/workflows/release.yml).

## License

See [LICENSE](LICENSE) for the repository's MIT license text. Files with an
explicit SPDX notice retain the license stated in that notice (currently
`MPL-2.0`; the full MPL-2.0 text is included in [LICENSES/MPL-2.0.txt](LICENSES/MPL-2.0.txt)).
