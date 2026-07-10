# Neua-Labs/.github

Org-level GitHub configuration: the public org profile (`profile/README.md`)
and **reusable workflows** shared by every Neua satellite app repo.

## Reusable workflows

Per [ADR-0002](https://github.com/Neua-Labs/neua-studio/blob/main/docs/adr/0002-fleet-repo-topology.md),
satellite repos (jury-day, neua-fit-app, her-crew, and future graduates) stop
copy-pasting near-identical CI/deploy YAML and instead call these:

| Workflow | Purpose |
|---|---|
| [`reusable-app-ci.yml`](.github/workflows/reusable-app-ci.yml) | PR/push CI: frontend build+test, optional `functions/` type-check+bundle+test, optional Playwright e2e |
| [`reusable-firebase-deploy.yml`](.github/workflows/reusable-firebase-deploy.yml) | Manual (`workflow_dispatch`) Firebase deploy with a `--only` target |
| [`reusable-framework-bump.yml`](.github/workflows/reusable-framework-bump.yml) | Weekly bump of `@neua-labs/*` to latest + a grouped PR (release-train follower) |

Each satellite adds a thin caller (≤15 lines). Example CI caller:

```yaml
name: CI
on: [pull_request, push]
jobs:
  ci:
    uses: Neua-Labs/.github/.github/workflows/reusable-app-ci.yml@main
    with:
      has-functions: true
      run-e2e: true
    secrets:
      NEUA_PACKAGES_TOKEN: ${{ secrets.NEUA_PACKAGES_TOKEN }}
```

Required repo secrets: `NEUA_PACKAGES_TOKEN` (read:packages PAT for the private
`@neua-labs/*` registry) and, for deploys, `FIREBASE_SERVICE_ACCOUNT`.

Pin callers to `@main` so fixes to shared CI propagate without a satellite
commit (the ADR-0002 propagation hierarchy).
