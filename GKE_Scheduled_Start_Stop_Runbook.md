
# GKE Scheduled Start/Stop – Runbook

## Overview
This runbook documents the end‑to‑end setup, troubleshooting, and validation steps for **GKE scheduled scale‑up/scale‑down to zero** using:
- GKE Cluster Autoscaler (min=0)
- Cloud Run Jobs (start / stop)
- Cloud Scheduler
- Terraform (infrastructure)

This setup is designed for **DEV / non‑prod** clusters to reduce cost by stopping nodes during nights and weekends.

---

## Architecture Summary

- **GKE cluster**: Regional (`europe-west2`)
- **Node pool**: Autoscaling enabled (min=0, max=6)
- **Automation**:
  - Cloud Scheduler → triggers Cloud Run Jobs
  - Cloud Run Job → executes `gcloud container clusters resize`
- **IAM model**:
  - Terraform SA: creates infra only (no IAM mutations)
  - Cloud Run Job SA: resizes GKE node pool
  - Scheduler Invoker SA: invokes Cloud Run Jobs

---

## Prerequisites

- Existing VPC and Subnet
- User‑managed secondary ranges for Pods & Services
- Terraform executed from GCE VM using default Compute SA

---

## Secondary IP Ranges (User‑Managed)

Create reusable secondary ranges (avoid GKE‑reserved ranges):

```bash
gcloud compute networks subnets update vmo2-gcp-network-trg-1-subnet-app   --region europe-west2   --add-secondary-ranges   gke-dev-pods=10.168.0.0/14,gke-dev-services=172.20.0.0/20
```

Verify:
```bash
gcloud compute networks subnets describe vmo2-gcp-network-trg-1-subnet-app   --region europe-west2   --format="yaml(secondaryIpRanges)"
```

---

## Terraform – Key Configuration

### Node Pool Autoscaling
```hcl
autoscaling {
  min_node_count = 0
  max_node_count = 6
}
```

> Note: `minNodeCount=0` is not shown in `gcloud` output because 0 is default.

### Cloud Run Job Command (example – STOP)
```bash
gcloud container clusters resize gke-vmo2-trg-dev-cluster   --node-pool np-app   --num-nodes 0   --region europe-west2   --project gci-tcsvmo2gcp-pjpc-01nl165324   --quiet
```

---

## IAM – Final Working Model

### 1. Cloud Run Job Runtime SA

Service Account:
```
gke-resizer-sa@<PROJECT>.iam.gserviceaccount.com
```

Required role (non‑conditional):
```
roles/container.admin
```

Grant (Admin‑only, one time):
```bash
gcloud projects add-iam-policy-binding <PROJECT_ID>   --member="serviceAccount:gke-resizer-sa@<PROJECT>.iam.gserviceaccount.com"   --role="roles/container.admin"
```

⚠️ Important: Previous conditional role was expired and caused runtime failures.

---

### 2. Cloud Scheduler → Cloud Run Job Invocation

Invoker SA:
```
scheduler-invoker-sa@<PROJECT>.iam.gserviceaccount.com
```

Grant per job:
```bash
gcloud run jobs add-iam-policy-binding gke-nodepool-start   --region europe-west1   --member="serviceAccount:scheduler-invoker-sa@<PROJECT>.iam.gserviceaccount.com"   --role="roles/run.invoker"


gcloud run jobs add-iam-policy-binding gke-nodepool-stop   --region europe-west1   --member="serviceAccount:scheduler-invoker-sa@<PROJECT>.iam.gserviceaccount.com"   --role="roles/run.invoker"
```

---

## Validation & Operations

### Check Node Pool Autoscaling
```bash
gcloud container node-pools describe np-app   --cluster gke-vmo2-trg-dev-cluster   --region europe-west2   --format="yaml(autoscaling,status,management)"
```

### Check Actual Nodes (Source of Truth)
```bash
kubectl get nodes
```

> Note: `nodeCount` is not shown for autoscaled pools; Kubernetes API is authoritative.

---

## Why Nodes May Remain After STOP

Even after scaling node pool to 0:
- GKE system DaemonSets (DNS, logging, metrics, CSI) must terminate cleanly
- Autoscaler uses **eventual consistency**
- 1–2 nodes may remain for **10–20 minutes**

✅ This is expected behavior.

### Correct Success Check
Check Managed Instance Group size:
```bash
gcloud compute instance-groups managed list   --filter="name~gke-gke-vmo2-trg-dev-cluster-np-app"   --regions europe-west2


gcloud compute instance-groups managed describe <MIG_NAME>   --region europe-west2   --format="value(targetSize)"
```

✅ `targetSize = 0` confirms scale‑to‑zero success.

---

## Scheduler Jobs

### List Jobs
```bash
gcloud scheduler jobs list --location europe-west1
```

### Manually Trigger (for testing)
```bash
gcloud scheduler jobs run gke-stop-10pm  --location europe-west1
gcloud scheduler jobs run gke-start-10am --location europe-west1
```

---

## Key Learnings / Best Practices

- Never reuse **GKE‑created** secondary ranges
- Avoid IAM changes in Terraform when using restricted SAs
- Always check for **conditional IAM bindings**
- Autoscaler scale‑to‑zero is **eventual**, not immediate
- MIG target size is the best validation signal

---

## Final Status

✅ Cluster stable
✅ Autoscaling working
✅ Scheduler + Cloud Run automation functional
✅ Cost‑optimized DEV GKE setup complete

---

_End of Runbook_
