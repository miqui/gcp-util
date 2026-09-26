# Deploy Fixes Log

Fixes applied to `gke-deploy.sh` during the `PROJECT_ID=k8s-dev-412419` deploy run on 2026-09-26.

## 1. Public IP detection returned IPv6

**Symptom**

```text
ERROR: could not detect public IP (got: '2600:1700:3211:1400:78d8:6992:715e:6ae')
```

**Cause**

`curl -s ifconfig.me` resolved over IPv6 on a dual-stack network, but the validation regex only accepts IPv4, and `--master-authorized-networks` expects an IPv4 CIDR here.

**Fix**

Force IPv4 in the operator IP detection:

```diff
-MY_IP=$(curl -s --max-time 10 ifconfig.me || true)
+MY_IP=$(curl -4 -s --max-time 10 ifconfig.me || true)
```

## 2. Cluster create rejected secondary ranges without IP aliasing

**Symptom**

```text
ERROR: (gcloud.container.clusters.create) Cannot specify --cluster-secondary-range-name without --enable-ip-alias.
```

**Cause**

`--cluster-secondary-range-name` / `--services-secondary-range-name` require alias IP ranges to be explicitly enabled with this gcloud version.

**Fix**

Added `--enable-ip-alias` to `gcloud container clusters create`:

```diff
     --network="$VPC" \
     --subnetwork="$SUBNET" \
+    --enable-ip-alias \
     --cluster-secondary-range-name=pods \
```

## 3. `--enable-image-streaming` no longer a valid flag

**Symptom**

```text
ERROR: (gcloud.artifacts.repositories.create) unrecognized arguments: --enable-image-streaming
```

**Cause**

The flag was removed from `gcloud artifacts repositories create`; image streaming is now enabled by default for Artifact Registry Docker repositories, so no opt-in is needed.

**Fix**

Removed the flag from the repo creation command:

```diff
   gcloud artifacts repositories create "$REPO" \
     --repository-format=docker \
-    --location="$REGION" \
-    --enable-image-streaming
+    --location="$REGION"
```

## 4. Missing `gke-gcloud-auth-plugin` (environment, not script)

**Symptom**

```text
CRITICAL: ACTION REQUIRED: gke-gcloud-auth-plugin, which is needed for continued use of kubectl, was not found or is not executable.
```

**Cause**

The plugin required for `kubectl` GKE authentication was not installed locally. Without it, the script's `kubectl get nodes` / `cluster-info` verification step fails.

**Fix**

Installed the gcloud component (one-time, local machine):

```sh
gcloud components install gke-gcloud-auth-plugin --quiet
```

## Notes from the run

- The script is idempotent: re-running skips already-created resources (VPC, subnet, router, NAT, cluster).
- gcloud backed up a malformed `~/.kube/config` to `~/.kube/config.2026-09-26T14-05-44Z.34611.00.backup` and recreated it with only the `dev-cluster` context. Merge any other contexts back from the backup if needed.
- Final state: `dev-cluster` RUNNING in `us-central1-a` (v1.35.6-gke.1250000, 2 nodes, autoscaling 2..5, private nodes), `api-images` Artifact Registry repo created with node SA pull access.
