# eastspring-governance-configs

The **deployment content** for Eastspring's Unity Catalog governance: the Access Matrix workbooks,
the config generated from them, the reference allow-lists and the SQL (mask functions, RLS
allow-list). The engine — the `eastspring-uc-governance` wheel, the notebooks and the jobs — is
built by the code repo's pipeline (8050); the pipelines here download that wheel.

## Layout

A **deployment** is `<domain>-<env>` (e.g. `esg-dev`): one domain (`esg`) in one environment
(`dev` | `sit` | `uat` | `prod`).

```
excels/<deployment>/<workbook>.xlsx   the Access Matrix — business-owned, the input
config/<deployment>/*.yaml            generated from the workbook — never edited by hand
reference/<deployment>.yaml           the allow-list: target_catalog, schemas, groups (+ Entra
                                      object IDs), governed tags, protected set — governance-owned
sql/<deployment>/*.sql                mask UDFs + the RLS allow-list MV — governance-owned
azure-pipelines-platform.yml          provisioning: groups, schemas, governed tags, functions
azure-pipelines.yml                   reconcile: validate, deploy, plan / apply
templates/run-job.yml                 runs a ucgov job and publishes its notebook output
```

Today: `esg-dev` (`target_catalog: hera-dev`).

Everything is per deployment, so a change is rolled out one environment at a time: make it in
`esg-dev`, test it there, then **promote** it by copying the files into `esg-sit`, `esg-uat` and
`esg-prod`. The SQL templates contain no catalog name (it comes from the reference when rendered),
so promoting a SQL change is a plain copy; keep the environments' SQL identical except while a
change is on its way through.

## Onboarding a change (business)

Prerequisites: Python 3.13 (3.10+), the wheel from pipeline 8050 (0.4.0 or later); run every
command from the root of this repo. The example is for `esg-dev`; use your deployment's folder.

<table>
<tr><th></th><th>macOS / Linux (bash / zsh)</th><th>Windows (PowerShell)</th></tr>

<tr><td><b>VirtualEnv</b></td>
<td><pre>
python3 -m venv .venv
source .venv/bin/activate
</pre></td>
<td><pre>
python -m venv .venv
.\.venv\Scripts\Activate.ps1
</pre>
</td></tr>

<tr><td><b>Install Wheel</b></td>
<td><pre>
pip install /path/to/eastspring_uc_governance-0.4.0-py3-none-any.whl
</pre></td>
<td><pre>
pip install &lt;path-to&gt;\eastspring_uc_governance-0.4.0-py3-none-any.whl
</pre></td></tr>

<tr><td><b>Generate yaml from excel</b></td>
<td><pre>
ucgov-generate excels/esg-dev/access-matrix-esg_workshop-v3-dynamic-views.xlsx config/esg-dev --reference reference/esg-dev.yaml --check

ucgov-generate excels/esg-dev/access-matrix-esg_workshop-v3-dynamic-views.xlsx config/esg-dev --reference reference/esg-dev.yaml
</pre></td>
<td><pre>
ucgov-generate excels\esg-dev\access-matrix-esg_workshop-v3-dynamic-views.xlsx config\esg-dev --reference reference\esg-dev.yaml --check

ucgov-generate excels\esg-dev\access-matrix-esg_workshop-v3-dynamic-views.xlsx config\esg-dev --reference reference\esg-dev.yaml
</pre></td></tr>

<tr><td><b>Validate</b></td>
<td><pre>
ucgov-validate config/esg-dev --reference reference/esg-dev.yaml --sql sql/esg-dev --verbose
</pre></td>
<td><pre>
ucgov-validate config\esg-dev --reference reference\esg-dev.yaml --sql sql\esg-dev --verbose
</pre></td></tr>

<tr><td><b>Check before committing</b></td>
<td><pre>
ucgov-generate excels/esg-dev/access-matrix-esg_workshop-v3-dynamic-views.xlsx config/esg-dev --reference reference/esg-dev.yaml --verify
</pre></td>
<td><pre>
ucgov-generate excels\esg-dev\access-matrix-esg_workshop-v3-dynamic-views.xlsx config\esg-dev --reference reference\esg-dev.yaml --verify
</pre></td></tr>
</table>

Then commit the workbook **and** the regenerated `config/<deployment>/` together and open a pull
request into `main`. The pipeline fails if the committed config isn't what the workbook generates.

## Pipelines

| | `azure-pipelines-platform.yml` | `azure-pipelines.yml` |
|---|---|---|
| Runs on | changes under `reference/` or `sql/` (PR + merge), or by hand | every PR into `main` + merge |
| Does | validate → `ucgov-provision groups` / `schemas` / `tags` → deploy reference + SQL → job `ucgov-apply-functions` | `--verify` the config → validate → check groups / schemas / tags exist → deploy config + reference → job `ucgov-reconcile` |

Both take a **mode** run parameter (`plan` | `apply`, default `plan`): pull requests always plan,
apply only runs from `main`. Each run publishes the notebook output as an HTML artifact. Run the
platform pipeline before the first reconcile apply of a deployment (the reconcile checks it).

Each pipeline targets one deployment through its variables `domain` and `envName`, with the
workspace connection from its variable group (`DEV-HERA-REPLATFORM` for `dev`).

## Adding an environment (e.g. `esg-sit`)

1. `excels/esg-sit/` — a copy of the dev workbook with the SIT catalog name, **edited in Excel**
   (it has formulas; scripted edits break them).
2. `reference/esg-sit.yaml` — `target_catalog`, the schemas, and the groups with their Entra object
   IDs for SIT; `rls_entitlement_table` under the SIT catalog.
3. `sql/esg-sit/` — a copy of `sql/esg-dev/` (no edits needed: the catalog comes from the reference).
4. `ucgov-generate excels/esg-sit/<workbook>.xlsx config/esg-sit --reference reference/esg-sit.yaml`, then validate with `--sql sql/esg-sit`.
5. Pipelines for SIT: the same YAML with `envName: sit` and the SIT workspace's variable group.
