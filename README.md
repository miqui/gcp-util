# gcp-util

A collection of self-contained utilities for provisioning and managing Google Cloud Platform resources from the command line. Each tool lives in its own subdirectory with its own README, scripts, and diagrams.

## Contents

| Directory | Description |
| --- | --- |
| [`gke-script/`](gke-script/) | One-shot provisioning and teardown of a development GKE cluster: dedicated VPC + subnet, Cloud NAT, private autoscaling nodes, IP-restricted control plane access, and an in-region Artifact Registry — plus a companion teardown script that removes everything in dependency order. |

## Conventions

- Each subdirectory is standalone: scripts take configuration via environment variables and are safe to re-run (idempotent).
- Teardown/cleanup companions are provided alongside provisioning scripts wherever possible.
- See each subdirectory's README for prerequisites, configuration, cost estimates, and troubleshooting.

## Requirements

Utilities here generally assume the [`gcloud` CLI](https://cloud.google.com/sdk) is installed and authenticated (`gcloud auth login`), a GCP project with billing attached, and sufficient IAM permissions on that project — details per tool in its README.
