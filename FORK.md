# MyFreeApp fork of grafana/mcp-grafana

This fork exists for one reason: `run_panel_query` upstream cannot execute
panels backed by a Microsoft SQL Server datasource, and every MyFreeApp
dashboard is backed by one.

It is meant to be **temporary**. Delete it once both patches are released
upstream and go back to `grafana/mcp-grafana`.

## What the `myfreeapp` branch carries on top of upstream

| Commit | Upstream PR | What it fixes |
| --- | --- | --- |
| `fix(run_panel_query): resolve constant and textbox variables from query` | [#1041](https://github.com/grafana/mcp-grafana/pull/1041) | `constant` and `textbox` variables resolved to nothing, so a bare `$name` reached the datasource and SQL Server rejected it as a syntax error. Affected 6 of our 23 dashboards. |
| `feat(run_panel_query): support MSSQL panels` | [#1042](https://github.com/grafana/mcp-grafana/pull/1042), closes [#999](https://github.com/grafana/mcp-grafana/issues/999) | MSSQL was missing from the datasource allowlist. It is sqlds-based like BigQuery, so the BigQuery executor was generalised rather than duplicated. |

Both are upstreamed as separate PRs because #1041 is an independent bug fix
that has nothing to do with MSSQL.

## Building

`azure-pipelines.yml` is manual-only. It runs the upstream unit suite in a
`golang` container (no Go toolchain needed on the agent), then builds and
pushes with the upstream `Dockerfile` to:

```
us-docker.pkg.dev/prj-myfreeapp-artifacts-shared/ar-myfreeapp-shared-us/mcp-grafana
```

tagged with the build ID and `latest`. Take the build-ID tag and set it on the
`mcp-grafana` container of `myfreeapp-grafana-mcp` in `myfreeapp-infrastructure`
— avoid `latest` there, so a rollback is a tag change rather than a rebuild.

## Keeping up with upstream

```bash
git remote add upstream https://github.com/grafana/mcp-grafana.git
git fetch upstream
git rebase upstream/main    # on the myfreeapp branch
```

If a rebase drops a commit because it merged upstream, that is the signal to
start removing this fork.
