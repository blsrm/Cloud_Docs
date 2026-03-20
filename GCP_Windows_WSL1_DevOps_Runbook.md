# DevOps Toolchain + WSL (Windows Server 2022 on GCE) — Consolidated Runbook & Justifications

**Owner:** Murugavel Ramachandran  
**Scope:** Commands, scripts, troubleshooting, and enterprise rationale captured from the full conversation for future reuse.

---

## 1) Environment Summary (What we are working with)

- **Windows Server 2022 VM on Google Compute Engine (GCE)**.
- **WSL distro:** Ubuntu (observed **Ubuntu 22.04.5 LTS**).
- **WSL mode in your case:** **WSL1** (you explicitly confirmed WSL2 is not supported in your current usage).

### Key implication of WSL1
- WSL1 is a syscall translation layer (no full Linux kernel), so some Linux service patterns (systemd, Docker daemon, local Kubernetes runtimes) are not a good fit.
- **systemd enablement guidance** applies to WSL2 (it’s supported there via `/etc/wsl.conf`). citeturn22search1

---

## 2) Why WSL2 failed earlier (and how we validated nested virtualization on GCP)

### Symptom (Windows VM)
- WSL2 install failed with: `HCS_E_HYPERV_NOT_INSTALLED`.

### Root cause (in GCE)
- On cloud VMs, WSL2 requires the guest hypervisor to run; that needs **nested virtualization** on the VM host.

### Enterprise reality
- Many enterprise projects do not have a **default** VPC network; VM creates fail unless `--network` and `--subnet` are explicitly specified.

---

## 3) GCP Commands — Nested Virtualization + VM Creation (Authoritative)

### 3.1 List Networks and Subnets (avoid “default network not found”)

```bash
gcloud compute networks list

gcloud compute networks subnets list --regions=europe-west2
```

### 3.2 Test nested virtualization support (Debian test VM)

```bash
gcloud compute instances create test-nested-check \
  --project=gci-tcsvmo2gcp-pjpc-01nl165324 \
  --zone=europe-west2-a \
  --machine-type=n2-standard-4 \
  --enable-nested-virtualization \
  --network=vmo2-gcp-network-trg-1-vpc \
  --subnet=vmo2-gcp-network-trg-1-subnet-app \
  --image-family=debian-12 \
  --image-project=debian-cloud
```

### 3.3 Create Windows Server 2022 VM (WSL2-ready, nested virt enabled)

```bash
gcloud compute instances create vm02-gcp-shared-windows-jumphost-wsl2-02 \
  --project=gci-tcsvmo2gcp-pjpc-01nl165324 \
  --zone=europe-west2-a \
  --machine-type=n2-standard-8 \
  --enable-nested-virtualization \
  --network=vmo2-gcp-network-trg-1-vpc \
  --subnet=vmo2-gcp-network-trg-1-subnet-app \
  --image-family=windows-2022 \
  --image-project=windows-cloud \
  --boot-disk-size=120GB \
  --boot-disk-type=pd-ssd
```

### 3.4 Reset Windows password (RDP)

```bash
gcloud compute reset-windows-password vm02-gcp-shared-windows-jumphost-wsl2-02 \
  --zone=europe-west2-a
```

---

## 4) Docker CLI vs Docker Daemon (Concept + Enterprise Guidance)

### What is Docker CLI?
- The **Docker CLI** (`docker`) is the **client** tool that sends requests to a Docker API endpoint.
- By itself, it does not build/run containers; it needs a server/engine to execute operations.

### What is Docker daemon?
- The **Docker daemon** (`dockerd`) is the **engine/service** that performs image builds, runs containers, manages networks/volumes.

### Is Docker CLI alone sufficient to build images?
- **No**—`docker build` requires a daemon/engine endpoint.
- Enterprise approach: build images in **CI/CD** (e.g., Cloud Build) to ensure reproducibility and auditability.

### Practical note for your environment
- In WSL, the Docker install script itself warns about WSL and recommends Docker Desktop on Windows. The official Docker Desktop docs describe the **WSL2 backend** as the supported approach. citeturn22search7

---

## 5) Rancher Desktop in WSL — What is possible?

### Can you install Rancher Desktop inside WSL Ubuntu?
- **Not recommended / generally not the intended model.** Rancher Desktop is a desktop app on Windows/macOS/Linux.

### How Rancher Desktop integrates with WSL (supported model)
- Rancher Desktop can expose kubeconfig into WSL distros using the **WSL Integrations** setting, enabling `kubectl` inside WSL to talk to the Rancher Desktop Kubernetes cluster. citeturn22search13

### Rancher Desktop on Windows depends on WSL2 virtualization layer
- Rancher Desktop’s Windows backend design is based on a **WSL2 VM-style backend**. citeturn22search15

**Conclusion for your case:** Since your Ubuntu is running in **WSL1**, Rancher Desktop integration benefits are limited; local Kubernetes on the same machine is not the right fit.

---

## 6) Enterprise Recommendation (Developer Productivity vs Operational Overhead)

### Recommended enterprise delivery model (you already have this)
- Developers commit to GitLab main branch.
- CI/CD (Cloud Build) builds container images.
- Deploy to DEV GKE.

**Why this is a strong enterprise approach:**
- Standardized builds (no “works on my machine”).
- Centralized security scanning, signing, SBOM, compliance.
- Reproducible deployments and easier audit.

### Do all developers need local Docker + local Kubernetes?
- **No.**
- Allow local Docker/Kubernetes only for:
  - teams building Helm charts/operators/CRDs,
  - developers needing fast local feedback loops,
  - offline development needs.

**For platform/infra roles** (your role), local Kubernetes is usually not required; focus is on GKE infra, clusters, networking, IAM, CI/CD, and shared platform capabilities.

---

## 7) WSL2 enablement steps (Reference) — only if/when WSL2 is used

> These steps are included for completeness; systemd support and Docker Desktop integrations typically assume WSL2. citeturn22search1turn22search7

### 7.1 Enable WSL features (PowerShell Admin)

```powershell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
wsl --set-default-version 2
```

### 7.2 Install Ubuntu 24.04

```powershell
wsl --install -d Ubuntu-24.04
```

---

## 8) WSL1 Optimized Tool Install — Practical Commands (What you needed)

You validated installed/missing tools and found many missing:
- Missing: `yarn`, `tsc`, `ts-node`, `next`, `newman`, `apigeelint`, `kubectl`, `helm`, `k9s`, `istioctl`, `psql`, `hurl`, `glab`, `gcloud`

### 8.1 Fix NVM loading in WSL when using `set -u` (nounset)

> **Why:** `set -u` can break scripts that reference unset variables; nvm can hit this with `PROVIDED_VERSION`.

Run in your shell:

```bash
set +u
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && . "$NVM_DIR/bash_completion"
set -u
```

Validate:

```bash
type -a nvm
nvm --version
node -v
```

### 8.2 Install Node global tools (yarn/typescript/ts-node/next/newman/apigeelint)

```bash
npm install -g yarn typescript ts-node next newman apigeelint
```

Verify:

```bash
yarn -v
tsc -v
ts-node -v
next -v
newman -v
apigeelint -v || true
```

### 8.3 Install kubectl

```bash
curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
rm -f kubectl
kubectl version --client
```

### 8.4 Install helm

```bash
curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```

### 8.5 Install k9s

```bash
curl -sS https://webinstall.dev/k9s | bash
export PATH="$HOME/.local/bin:$PATH"
k9s version
```

### 8.6 Install istioctl

```bash
curl -L https://istio.io/downloadIstio | sh -
sudo mv istio-*/bin/istioctl /usr/local/bin/
rm -rf istio-*
istioctl version --remote=false
```

### 8.7 Install PostgreSQL client (psql)

```bash
sudo apt-get update -y
sudo apt-get install -y postgresql-client
psql --version
```

### 8.8 Install Hurl

```bash
sudo apt-get update -y
sudo apt-get install -y hurl
hurl --version
```

### 8.9 Install glab (GitLab CLI)

```bash
curl -fsSL https://gitlab.com/gitlab-org/cli/-/raw/main/scripts/install.sh | sudo bash
glab --version
```

### 8.10 Install gcloud (Google Cloud CLI)

```bash
curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg \
  | sudo gpg --dearmor -o /usr/share/keyrings/cloud.google.gpg

echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" \
  | sudo tee /etc/apt/sources.list.d/google-cloud-sdk.list >/dev/null

sudo apt-get update -y
sudo apt-get install -y google-cloud-cli
gcloud version
```

---

## 9) Validation & Post-check

### 9.1 Tool presence check loop

```bash
for c in git make curl openssl node npm npx yarn tsc ts-node next newman apigeelint kubectl helm k9s istioctl psql hurl glab gcloud; do
  printf "%-12s : " "$c"
  command -v "$c" >/dev/null 2>&1 && echo "OK" || echo "MISSING"
done
```

### 9.2 Kubernetes client sanity

```bash
kubectl version --client
helm version
k9s version
```

### 9.3 Cloud CLI sanity

```bash
gcloud version
gcloud auth list
```

---

## 10) Notes & Decisions Log (Justification)

### 10.1 Why we avoid local Docker/Kubernetes inside your WSL1 environment
- WSL1 is not designed for systemd-managed daemons and container runtimes in the same way as a full Linux kernel.
- Docker Desktop’s recommended Windows integration uses a **WSL2 backend**, which you don’t have in the WSL1 scenario. citeturn22search7

### 10.2 Preferred enterprise approach
- Use CI/CD for image builds and deployments (Cloud Build → Artifact Registry → GKE).
- Local dev stacks (Rancher Desktop/Tilt) remain optional for developer convenience, not a mandatory baseline.

### 10.3 Rancher Desktop placement
- Install **Rancher Desktop on Windows host** (where applicable) and enable WSL integrations to expose kubeconfig into WSL distros. citeturn22search13
- Rancher Desktop’s Windows backend is based on WSL2 virtualization architecture. citeturn22search15

---

## References
- Microsoft: **systemd support in WSL** (WSL2-focused configuration using `/etc/wsl.conf`). citeturn22search1
- Docker Docs: **Docker Desktop WSL2 backend** (recommended integration model). citeturn22search7
- Rancher Desktop Docs: **WSL integrations** for sharing kubeconfig into WSL distros. citeturn22search13
- Rancher Desktop architecture note: **WSL2 backend implementation** design. citeturn22search15
