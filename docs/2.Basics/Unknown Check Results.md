---
layout: default
published: true
title: Unknown Check Results
nav_order: 7
---

# Unknown Check Results

Some policies can return **UNKNOWN** when they cannot determine pass or fail—for example when a value depends on a variable or a Terraform plan field is not yet known. Checkov now includes these results in the report so you can see when and where they occur.

## What appears in the report

- **Summary:** The scan summary includes an **Unknown checks** count.
- **CLI:** Unknown results are listed (in yellow when not using `--quiet`) with check ID, resource, file, and guide link.
- **JSON:** The report includes a top-level `unknown_checks` array and `summary.unknown`.
- **Other formats:** SARIF, CSV, JUnit XML, and platform uploads include unknown checks where applicable.

## Example CLI output

```
terraform_plan scan results:

Passed checks: 485, Failed checks: 55, Skipped checks: 0, Unknown checks: 132

Check: CKV_AWS_140: "Ensure that RDS global clusters are encrypted"
	UNKNOWN for resource: module.db_module.module.global_cluster.aws_rds_global_cluster.global
	File: /plan.json:5109-5157
	Guide: https://docs.prismacloud.io/...
...
```

Unknown results are grouped with the rest of the checks and can be inspected to decide whether to resolve variables, adjust the plan, or accept the uncertainty.

## Terraform plan: `EVAL_TF_PLAN_AFTER_UNKNOWN`

For Terraform plan scans, the optional env var **`EVAL_TF_PLAN_AFTER_UNKNOWN`** enables use of the plan’s `after_unknown` section to try to resolve some values. When enabled:

- Some checks that would be **UNKNOWN** may become **PASSED** or **FAILED**.
- In other cases, a value that was previously known in the plan can be treated as unknown, so a check may move from **FAILED** to **UNKNOWN**.

Example (sanitized; counts will vary by plan):

| | Without env var | With `EVAL_TF_PLAN_AFTER_UNKNOWN=True` |
|---|------------------|----------------------------------------|
| Passed | 485 | 485 |
| Failed | 55 | 54 |
| Unknown | 132 | 133 |

Here, one check (e.g. an encryption check on `aws_rds_global_cluster`) moved from FAILED to UNKNOWN because with after_unknown evaluation its value is treated as unresolved.

To enable:

```bash
export EVAL_TF_PLAN_AFTER_UNKNOWN=True
checkov -f plan.json --framework terraform_plan
```

## JSON output

With `--output json` you get:

- `results.unknown_checks`: list of records (same shape as `failed_checks` / `passed_checks`, with `check_result.result: "UNKNOWN"`).
- `summary.unknown`: count of unknown checks.

Example (abbreviated):

```json
{
  "summary": {
    "passed": 485,
    "failed": 55,
    "skipped": 0,
    "unknown": 132
  },
  "results": {
    "passed_checks": [...],
    "failed_checks": [...],
    "skipped_checks": [...],
    "unknown_checks": [
      {
        "check_id": "CKV_AWS_140",
        "check_name": "Ensure that RDS global clusters are encrypted",
        "resource": "module.db_module.module.global_cluster.aws_rds_global_cluster.global",
        "file_path": "/plan.json",
        "check_result": { "result": "UNKNOWN" }
      }
    ]
  }
}
```

## Exit code

UNKNOWN results do **not** affect the exit code. Only **failed** checks (and your `--soft-fail` / `--hard-fail-on` settings) determine a non-zero exit code.
