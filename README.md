# `setup-db`

GitHub Action: connect a workflow to a persistent, repo-scoped
[PerSQL](https://persql.com) database — keyless, via GitHub OIDC.
No secret to store; the repo's identity is the credential.

## Usage

```yaml
# .github/workflows/ci.yml
jobs:
  test:
    runs-on: ubuntu-latest
    permissions:
      id-token: write   # required for OIDC
      contents: read
    steps:
      - uses: actions/checkout@v6

      - name: Connect to PerSQL
        id: db
        uses: persql/setup-db-action@v1

      - name: Query
        env:
          PERSQL_TOKEN: ${{ steps.db.outputs.token }}
          PERSQL_URL: ${{ steps.db.outputs.url }}
        run: |
          curl -sf -X POST "$PERSQL_URL/query" \
            -H "Authorization: Bearer $PERSQL_TOKEN" \
            -H 'content-type: application/json' \
            -d '{"sql":"CREATE TABLE IF NOT EXISTS runs (id INTEGER PRIMARY KEY, sha TEXT)"}'
```

## Prerequisites

1. Install the [PerSQL GitHub App](https://github.persql.com) on the
   repository. The installation binds the repo to a PerSQL namespace —
   that's what the OIDC exchange resolves against.
2. Give the job `permissions: id-token: write`. Without it GitHub
   issues no OIDC token and the action fails with a clear error.

## Inputs

| Input | Required | Default | Description |
| --- | --- | --- | --- |
| `api-url` | no | `https://api.persql.com` | PerSQL API base URL. |

## Outputs

| Output | Description |
| --- | --- |
| `token` | Short-lived PerSQL bearer token, scoped to this repo's Actions database (masked in logs via `::add-mask::`). |
| `url` | Base `/v1` URL for the database — `POST {url}/query`. |
| `database` | The provisioned database slug. |
| `namespace` | The PerSQL namespace slug. |

## What you get

Each repository gets one persistent database, provisioned on first
connect and reused on every subsequent run. State written in one
workflow run is there in the next — test matrices, flake counters,
benchmark history, release ledgers.

The token is short-lived and scoped to that single database. Workflow
logs never see it.

For ephemeral per-PR databases instead of one persistent one, use
[`persql/pr-preview-db-action`](https://github.com/persql/pr-preview-db-action).

## Pin to a major version

Tags follow semver. `@v1` follows the latest non-breaking 1.x release
— recommended. Pin to a specific tag (`@v1.0.0`) if you'd rather
upgrade explicitly.

## License

MIT.
