# Hub-repo-1 — Reusable GitHub Actions

Central "hub" repo of reusable **composite GitHub Actions** consumed by spoke
infrastructure repositories (e.g. `Spoke-repo/Spoke-repo-2`). Every spoke
workflow references these by tag: `Spoke-repo/Hub-repo-1/<action-path>@v1`.

## Layout

```
Hub-repo-1/
├── onboarding/
│   ├── gha_pipeline_creation_maintainer/action.yml
│   ├── iac_pipeline_creation/action.yml
│   ├── finops_pipeline_creation/action.yml
│   └── templates/                # rendered into new spoke repos
│       ├── CODEOWNERS
│       ├── pipeline_creation_maintainer.yml
│       ├── iac_pipeline.yml.tmpl
│       └── finops_pipeline.yml.tmpl
├── finops/
│   ├── aks/{start_aks_cluster,stop_aks_cluster}/action.yml
│   └── psql/{start_server,stop_server}/action.yml
└── spoke/
    ├── data_factory_module/action.yml
    ├── admin_vm_linux_module/action.yml
    ├── aks_module/action.yml
    ├── app_service_module/action.yml
    ├── web_application_firewall_policy_module/action.yml
    ├── postgresql_flexiserver_pe_module/action.yml
    └── storage_account_module/action.yml
```

## How it's consumed

A spoke repo workflow looks like:

```yaml
- uses: Spoke-repo/Hub-repo-1/spoke/aks_module@v1
  with:
    working_directory: iac/nonprod/aks_sample
    tfvars_path: nonprod/aks_sample/terraform.tfvars
    environment: nonprod
    tf_action: plan
    azure_client_id: ${{ secrets.AZ_CLIENT_ID }}
    azure_tenant_id: ${{ secrets.AZ_TENANT_ID }}
    azure_subscription_id: ${{ secrets.AZ_SUBSCRIPTION_ID }}
```

## Versioning

Tag releases (`v1`, `v1.1`, ...) so spoke repos pin to a stable version rather
than `@main`. Bump the tag whenever an action's `inputs` change in a
breaking way.

## Notes / TODO before production use

- The `finops_pipeline_creation` template currently always points at the
  `aks` start/stop actions — when scaffolding a `psql` schedule, update the
  generated workflow to call `finops/psql/start_server` /
  `finops/psql/stop_server` instead.
- Replace `<your-runner-label>` and `<your-resource-group>` placeholders
  before use.
- Auth is via GitHub OIDC → Azure (`azure/login@v2`); no long-lived secrets
  are stored beyond the three `AZ_*` federated-credential identifiers.
