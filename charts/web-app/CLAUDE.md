# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Helm chart for deploying boatload-llc platform web apps (React frontend + Go API), on GKE or on-prem (`google.gke: false`). Published to `https://boatload-llc.github.io/helm-charts` via chart-releaser on merge to `main`.

## Dev Workflow

```bash
make test               # lint + render diff + unit tests (full suite)
make unit               # helm-unittest only (fast feedback)
make template           # render to test-output.yaml, git diff to review changes
make lint               # helm lint only
helm template test . -f test-values.yaml -n test-ns   # render to stdout for inspection
```

`make test` is the primary validation loop. Unit tests live in `tests/*_test.yaml` and use [helm-unittest](https://github.com/helm-unittest/helm-unittest).

## Versioning

Bump `version` in `Chart.yaml` for every change. On-prem apps get the chart version from on-prem-admin's `apps/charts/app-set/values.yaml` `chart.version`.

## Required Values

Every deployment must supply:
- `lifecycle` — e.g. `dev`, `stage`, `prod`, `lswingz-42`
- `app_code` — e.g. `luckyswingz`
- `domain` — e.g. `luckyswingz.com`
- `gke_project_id`, `db_project_id` — GCP project IDs for GKE cluster and Cloud SQL
- `google.app_project` (set it; the `{app_code}-app-project` fallback is the retired legacy project)

## Adding an Integration

Add an `enabled: false` default in `values.yaml`, gate the ExternalSecret entry in `external-secrets.yaml`, and gate the env vars in `api-dep.yaml` (and `nginx-dep.yaml` if the frontend needs it).

For new integrations that don't need a dedicated feature flag, use `external_secrets.extra_keys: [my-key]` — the chart adds the entry automatically with the prefix applied. No chart change needed. Secrets are only rendered when `enabled: true` and `local: false`.

## Ephemeral Environments

All `remoteRef.key` values are prefixed with `external_secrets.secret_prefix` (defaults `""`). For ephemeral envs, set `secret_prefix: "lswingz-42-"` so secrets resolve to `lswingz-42-django-key` etc. without touching permanent secrets.
