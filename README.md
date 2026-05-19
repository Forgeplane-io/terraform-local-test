# Terraform Local Test

A small deterministic Terraform fixture for testing real Terraform execution with the `hashicorp/local` provider.

The configuration writes a generated config file, README, and sensitive env file to a local output directory. It is intended for smoke-testing Terraform plan/apply flows in Forgeplane and similar IaC runners.

## Requirements

- Terraform `>= 1.0`
- Network access to download the `hashicorp/local` provider on first init

## Usage

Initialize providers:

```sh
terraform init
```

Apply with the default values from `terraform.tfvars`:

```sh
terraform apply -auto-approve
```

Or write output to a temporary directory:

```sh
OUT_DIR=$(mktemp -d)
terraform apply -auto-approve -var="output_dir=$OUT_DIR"
```

Check idempotence after apply:

```sh
terraform plan -detailed-exitcode -var="output_dir=$OUT_DIR"
```

A deterministic fixture should return exit code `0` after a successful apply.

## Variables

| Variable | Default | Description |
| --- | --- | --- |
| `output_dir` | `/tmp/forgeplane-test` | Directory where generated files are written. |
| `environment` | `dev` | Environment label written into generated content. |
| `app_name` | `forgeplane` | Application name written into generated content. |
| `replicas` | `2` | Replica count written into generated content. |
| `generated_at` | `2026-01-01T00:00:00Z` | Stable timestamp-like value for deterministic output. |

## Determinism note

This fixture intentionally avoids Terraform functions such as `timestamp()`, `uuid()`, and random providers in resource content. Those values change between plans and make follow-up plans report changes even when Terraform and the runner are working correctly.

Use this repository for idempotent real-Terraform tests. Use a separate explicit always-diff fixture when you need to verify drift/change reporting behavior.
