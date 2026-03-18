# WSL2 + Ubuntu 24.04 on GCP Windows VM — Runbook (Commands + Scripts)

> **Purpose**: Future reference for enabling **WSL2 + Ubuntu 24.04** on a **Windows VM running in Google Cloud (GCE)** using **nested virtualization**, plus the **Ubuntu 24.04 DevOps toolchain install** (single script) and all key troubleshooting commands.
>
> **Context (what we solved)**:
> - WSL2 failed inside a GCP Windows VM with: `HCS_E_HYPERV_NOT_INSTALLED`
> - Root cause: nested virtualization not enabled / wrong VM family / missing VPC network in command.
> - Fix: Create an **N2** VM with `--enable-nested-virtualization` and correct `--network/--subnet`.

---

## 1) Quick Compatibility (Mac vs Ubuntu 24.04)

All these tools are supported on Ubuntu 24.04 and commonly used for DevOps/Kubernetes:

- Rancher Desktop, VS Code, Tilt, Git, npm, make, nvm, helm, kubectl, nerdctl, curl, k9s, yarn,
  apigeelint, psql, openssl, newman, npx, tsc, istioctl, node, next, ts-node, glab, Hurl, gcloud.

Common Linux-specific gotchas:
- Add your user to the `docker` group (no `sudo docker`)
- Increase `inotify` watch limits for Node/Tilt/Next file watching
- Prefer working under Linux FS (not `/mnt/c`) when using WSL

---

## 2) Ubuntu 24.04 — Full DevOps Toolchain Install (Single Bash Script)

### 2.1 How to run

```bash
nano full-devops-install-ubuntu24.sh
# paste the script below
chmod +x full-devops-install-ubuntu24.sh
./full-devops-install-ubuntu24.sh
```

> **Note**: Docker group changes require logout/login (or reboot).

### 2.2 Full install script

```bash
#!/usr/bin/env bash
set -euo pipefail

# ==========================
# Full DevOps Toolchain Install for Ubuntu 24.04 LTS
# Installs:
# Rancher Desktop, VS Code, Tilt, Git, make, nvm, node, npm, npx, yarn,
# typescript (tsc), ts-node, next, helm, kubectl, nerdctl, curl, k9s,
# apigeelint, psql, openssl, newman, istioctl, glab, hurl, gcloud
# ==========================

LOG_PREFIX="[DEVOPS-INSTALL]"
NVM_VERSION="v0.39.7"
INOTIFY_WATCHES="524288"

log() { echo -e "${LOG_PREFIX} $*"; }
warn() { echo -e "${LOG_PREFIX} \033[33mWARN:\033[0m $*"; }
err() { echo -e "${LOG_PREFIX} \033[31mERROR:\033[0m $*"; }

need_cmd() {
  command -v "$1" >/dev/null 2>&1
}

require_sudo() {
  if ! sudo -n true 2>/dev/null; then
    log "Sudo permission required. You may be prompted for password..."
  fi
}

ensure_ubuntu() {
  if [[ ! -f /etc/os-release ]]; then
    err "/etc/os-release not found. Unsupported OS."
    exit 1
  fi
  . /etc/os-release
  if [[ "${ID:-}" != "ubuntu" ]]; then
    err "This script is intended for Ubuntu. Detected ID=${ID:-unknown}"
    exit 1
  fi
  log "Detected OS: ${PRETTY_NAME}"
}

apt_install() {
  require_sudo
  sudo apt-get update -y
  sudo apt-get install -y "$@"
}

add_sysctl_inotify() {
  require_sudo
  local conf="/etc/sysctl.conf"
  if grep -q "^fs.inotify.max_user_watches=" "$conf" 2>/dev/null; then
    sudo sed -i "s/^fs.inotify.max_user_watches=.*/fs.inotify.max_user_watches=${INOTIFY_WATCHES}/" "$conf"
  else
    echo "fs.inotify.max_user_watches=${INOTIFY_WATCHES}" | sudo tee -a "$conf" >/dev/null
  fi
  sudo sysctl -p >/dev/null || true
  log "Inotify max_user_watches set to ${INOTIFY_WATCHES}"
}

install_core_packages() {
  log "Installing core packages..."
  apt_install \
    ca-certificates curl gnupg lsb-release \
    build-essential unzip zip software-properties-common \
    git make openssl
}

install_vscode() {
  if need_cmd code; then
    log "VS Code already installed. Skipping."
    return
  fi

  log "Installing VS Code..."
  require_sudo

  sudo mkdir -p /etc/apt/keyrings
  curl -fsSL https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor | sudo tee /etc/apt/keyrings/packages.microsoft.gpg >/dev/null

  echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" \
    | sudo tee /etc/apt/sources.list.d/vscode.list >/dev/null

  sudo apt-get update -y
  sudo apt-get install -y code
  log "VS Code installed."
}

install_docker() {
  if need_cmd docker; then
    log "Docker already installed. Skipping Docker engine install."
  else
    log "Installing Docker (official get.docker.com script)..."
    require_sudo
    curl -fsSL https://get.docker.com | sudo sh
  fi

  if groups "$USER" | grep -q "\bdocker\b"; then
    log "User '$USER' already in docker group."
  else
    require_sudo
    sudo usermod -aG docker "$USER"
    warn "Added '$USER' to docker group. You must LOG OUT and LOG IN (or reboot) for it to fully apply."
  fi
}

install_nvm_node() {
  if [[ -d "${HOME}/.nvm" ]]; then
    log "nvm already present at ~/.nvm. Skipping nvm install."
  else
    log "Installing nvm (${NVM_VERSION})..."
    curl -fsSL "https://raw.githubusercontent.com/nvm-sh/nvm/${NVM_VERSION}/install.sh" | bash
  fi

  export NVM_DIR="${HOME}/.nvm"
  [[ -s "${NVM_DIR}/nvm.sh" ]] && source "${NVM_DIR}/nvm.sh"
  [[ -s "${NVM_DIR}/bash_completion" ]] && source "${NVM_DIR}/bash_completion"

  log "Installing Node.js LTS via nvm..."
  nvm install --lts
  nvm use --lts

  log "Node version: $(node -v)"
  log "npm version:  $(npm -v)"
  log "npx version:  $(npx -v)"
}

install_node_global_tools() {
  log "Installing global Node tools (yarn, typescript, ts-node, next, newman, apigeelint)..."
  npm install -g yarn typescript ts-node next newman apigeelint
}

install_kubectl() {
  if need_cmd kubectl; then
    log "kubectl already installed. Skipping."
    return
  fi

  log "Installing kubectl..."
  curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
  require_sudo
  sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
  rm -f kubectl
  log "kubectl installed."
}

install_helm() {
  if need_cmd helm; then
    log "helm already installed. Skipping."
    return
  fi
  log "Installing Helm..."
  curl -fsSL https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
}

install_tilt() {
  if need_cmd tilt; then
    log "Tilt already installed. Skipping."
    return
  fi
  log "Installing Tilt..."
  curl -fsSL https://raw.githubusercontent.com/tilt-dev/tilt/master/scripts/install.sh | bash
}

install_k9s() {
  if need_cmd k9s; then
    log "k9s already installed. Skipping."
    return
  fi
  log "Installing k9s (via webinstall)..."
  curl -sS https://webinstall.dev/k9s | bash

  if [[ ":$PATH:" != *":${HOME}/.local/bin:"* ]]; then
    export PATH="${HOME}/.local/bin:$PATH"
  fi
}

install_istioctl() {
  if need_cmd istioctl; then
    log "istioctl already installed. Skipping."
    return
  fi

  log "Installing istioctl..."
  curl -L https://istio.io/downloadIstio | sh -
  require_sudo
  local ISTIOCTL_BIN
  ISTIOCTL_BIN="$(ls -d istio-* 2>/dev/null | tail -n 1)/bin/istioctl"
  if [[ -f "$ISTIOCTL_BIN" ]]; then
    sudo mv "$ISTIOCTL_BIN" /usr/local/bin/istioctl
    rm -rf "$(dirname "$(dirname "$ISTIOCTL_BIN")")" || true
  fi
}

install_nerdctl() {
  if need_cmd nerdctl; then
    log "nerdctl already installed. Skipping."
    return
  fi
  log "Installing nerdctl..."
  apt_install nerdctl
}

install_psql_client() {
  if need_cmd psql; then
    log "psql already installed. Skipping."
    return
  fi
  log "Installing PostgreSQL client (psql)..."
  apt_install postgresql-client
}

install_hurl() {
  if need_cmd hurl; then
    log "Hurl already installed. Skipping."
    return
  fi
  log "Installing Hurl..."
  apt_install hurl
}

install_glab() {
  if need_cmd glab; then
    log "glab already installed. Skipping."
    return
  fi
  log "Installing glab (GitLab CLI)..."
  require_sudo
  curl -fsSL https://gitlab.com/gitlab-org/cli/-/raw/main/scripts/install.sh | sudo bash
}

install_gcloud() {
  if need_cmd gcloud; then
    log "gcloud already installed. Skipping."
    return
  fi

  log "Installing Google Cloud SDK (gcloud)..."
  require_sudo

  curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg \
    | sudo gpg --dearmor -o /usr/share/keyrings/cloud.google.gpg

  echo "deb [signed-by=/usr/share/keyrings/cloud.google.gpg] https://packages.cloud.google.com/apt cloud-sdk main" \
    | sudo tee /etc/apt/sources.list.d/google-cloud-sdk.list >/dev/null

  sudo apt-get update -y
  sudo apt-get install -y google-cloud-cli
}

install_rancher_desktop() {
  if dpkg -l | grep -q rancher-desktop; then
    log "Rancher Desktop already installed. Skipping."
    return
  fi

  log "Installing Rancher Desktop (.deb from latest release)..."
  local DEB="rancher-desktop-linux-amd64.deb"
  curl -L -o "${DEB}" "https://github.com/rancher-sandbox/rancher-desktop/releases/latest/download/${DEB}"
  require_sudo
  sudo apt-get install -y "./${DEB}"
  rm -f "${DEB}"
}

final_checks() {
  log "Applying recommended system tweaks..."
  add_sysctl_inotify

  log "Done ✅"
  warn "If Docker group was just added, please LOG OUT / LOG IN (or reboot)."
}

main() {
  ensure_ubuntu
  install_core_packages
  install_vscode
  install_docker
  install_nvm_node
  install_node_global_tools
  install_kubectl
  install_helm
  install_tilt
  install_k9s
  install_istioctl
  install_nerdctl
  install_psql_client
  install_hurl
  install_glab
  install_gcloud
  install_rancher_desktop
  final_checks
}

main "$@"
```

---

## 3) Windows (GCP VM) — WSL2 + Ubuntu 24.04 Detailed Steps

### 3.1 In Windows VM: verify virtualization is available

**Task Manager → Performance → CPU** should show:
- `Virtualization: Enabled`

Or PowerShell:

```powershell
systeminfo | findstr /i "Virtualization Hyper-V"
```

Expected after nested virtualization is enabled:
- You should *not* get the old error during WSL install.

### 3.2 Enable required Windows optional features

Run PowerShell as Administrator:

```powershell
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
```

Reboot the VM.

### 3.3 Set WSL2 default + install Ubuntu 24.04

```powershell
wsl --set-default-version 2
wsl --install -d Ubuntu-24.04
```

### 3.4 First-time Ubuntu steps

```bash
sudo apt update && sudo apt upgrade -y
uname -a
```

Verify WSL version:

```powershell
wsl -l -v
```

Expect Ubuntu distro version `2`.

---

## 4) WSL2 troubleshooting commands (when WSL2 fails)

### 4.1 Confirm required Windows features state

```powershell
dism.exe /online /get-featureinfo /featurename:VirtualMachinePlatform
dism.exe /online /get-featureinfo /featurename:Microsoft-Windows-Subsystem-Linux
```

### 4.2 Ensure default WSL version is 2

```powershell
wsl --set-default-version 2
wsl --status
```

### 4.3 If WSL is stuck

```powershell
wsl --shutdown
```

### 4.4 Hypervisor launch type (rare)

```powershell
bcdedit /enum {current} | findstr /i hypervisorlaunchtype
bcdedit /set hypervisorlaunchtype auto
```

---

## 5) GCP — Nested Virtualization + VM Creation (Authoritative Commands)

### 5.1 Why nested virtualization was needed

Running Windows inside GCE is already virtualized.
WSL2 needs a lightweight VM (Hyper-V).
Therefore you must:
- Use a supported machine family (e.g., **N2**)
- Enable nested virtualization **at creation time**

### 5.2 Test nested virtualization with a Linux VM (Debian)

> **Note**: In many enterprise projects, the `default` VPC does not exist; you must specify `--network` and `--subnet`.

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

### 5.3 Create Windows Server 2022 VM with nested virtualization (WSL2-ready)

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

### 5.4 Reset Windows password (RDP access)

```bash
gcloud compute reset-windows-password vm02-gcp-shared-windows-jumphost-wsl2-02 \
  --zone=europe-west2-a
```

### 5.5 List networks / subnets (to avoid “default network not found”)

```bash
gcloud compute networks list

gcloud compute networks subnets list --regions=europe-west2
```

---

## 6) Notes & Best Practices

### 6.1 Disk size warning during Windows VM creation

If you see:

> Disk size is larger than image size … might need to resize root partition

Usually **Windows auto-expands**. If not, use **Disk Management → Extend Volume**.

### 6.2 Best practice for DevOps

Even if WSL2 works, for heavy Kubernetes/CI workloads the simplest model is often:
- **Ubuntu 24.04 Linux VM directly** on GCP
- Use **VS Code Remote SSH**

But if you require Windows jumphost + Linux tools, your current setup (Windows + nested virt + WSL2) is valid.

---

## 7) Quick “All-in-one” validation checklist

### Windows (PowerShell Admin)

```powershell
wsl --status
systeminfo | findstr /i "Virtualization Hyper-V"
```

### Ubuntu (WSL)

```bash
uname -a
node -v
npm -v
kubectl version --client
helm version
gcloud version
```

---

*Document generated for Murugavel Ramachandran — for internal reuse and future reference.*
