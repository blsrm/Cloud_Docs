# Runbook: Self‑Hosted GitLab CE on GCP + SendGrid SMTP + Cloud Build (GitLab) + GKE Deploy

> **Purpose**: A consolidated, copy‑paste friendly runbook of the commands and configurations we used to get:
> 1) GitLab Omnibus running on Ubuntu,
> 2) outbound email via SendGrid SMTP,
> 3) GitLab behind GCP HTTPS Load Balancer (TLS termination),
> 4) Cloud Build ↔ self‑managed GitLab connection with KMS,
> 5) Cloud Build triggers + multi‑image Docker builds + deploy to GKE.

---

## 0) References (official)

- GitLab Omnibus SMTP settings: https://docs.gitlab.com/omnibus/settings/smtp/ citeturn6search37  
- Google Cloud: sending email from Compute Engine (ports 587/465 allowed; port 25 blocked externally): https://docs.cloud.google.com/compute/docs/tutorials/sending-mail citeturn9search47  
- SendGrid Email Activity event meanings (Processed, Delivered, Dropped, Deferred, etc.): https://www.twilio.com/docs/sendgrid/ui/analytics-and-reporting/email-activity citeturn9search41  
- SendGrid “Processing” status troubleshooting / account review: https://support.sendgrid.com/hc/en-us/articles/31236228514587-Emails-Stuck-in-Processing-Status citeturn9search42  
- SendGrid “Account Under Review” states: https://www.twilio.com/docs/sendgrid/ui/account-and-settings/account-under-review citeturn9search53  
- Cloud Build substitutions (built‑in vs user‑defined): https://docs.cloud.google.com/build/docs/configuring-builds/substitute-variable-values citeturn23search10  
- Cloud Build escaping/substitution gotcha (use `$$` for env vars in steps): https://stackoverflow.com/questions/56800591/cloud-build-substitutions-in-substitutions-section citeturn23search13

---

## 1) GitLab Omnibus quick install (Ubuntu)

> Omnibus is the recommended package‑based install for self‑managed GitLab.

### 1.1 Update and install dependencies

```bash
sudo apt update
sudo apt upgrade -y

sudo apt install -y curl openssh-server ca-certificates postfix tzdata perl
```

### 1.2 Add GitLab CE repository and install

```bash
curl -sS https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.deb.sh | sudo bash

# Set your external URL (HTTP or HTTPS depending on your reverse proxy strategy)
sudo EXTERNAL_URL="http://gitlab.example.com" apt install -y gitlab-ce

sudo gitlab-ctl reconfigure
```

### 1.3 Useful service commands

```bash
sudo gitlab-ctl status
sudo gitlab-ctl restart
sudo gitlab-ctl tail
```

---

## 2) SendGrid SMTP for GitLab (recommended on GCP)

### 2.1 Why SendGrid (GCP networking note)

- Google Cloud allows outbound SMTP to **ports 587/465**, while **port 25 to external IPs is blocked** by default. citeturn9search47

### 2.2 SendGrid prerequisites

1) Create SendGrid account
2) Complete **Domain Authentication** (SPF/DKIM) (recommended for deliverability)
3) Create an API key with **Mail Send** permissions

### 2.3 Configure GitLab SMTP (`/etc/gitlab/gitlab.rb`)

> This is the GitLab‑official SMTP configuration pattern. citeturn6search37

```ruby
# /etc/gitlab/gitlab.rb

gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtp.sendgrid.net"
gitlab_rails['smtp_port'] = 587
gitlab_rails['smtp_user_name'] = "apikey"               # SendGrid uses the literal username "apikey"
gitlab_rails['smtp_password'] = "SG.xxxxx..."          # your SendGrid API key

gitlab_rails['smtp_domain'] = "tcsvmo2trg.com"         # your sending domain
gitlab_rails['smtp_authentication'] = "login"
gitlab_rails['smtp_enable_starttls_auto'] = true
gitlab_rails['smtp_openssl_verify_mode'] = 'peer'

# Align From/Reply-To to authenticated domain (improves deliverability)
gitlab_rails['gitlab_email_from'] = 'noreply@tcsvmo2trg.com'
gitlab_rails['gitlab_email_reply_to'] = 'noreply@tcsvmo2trg.com'
```

Apply:

```bash
sudo gitlab-ctl reconfigure
sudo gitlab-ctl restart
```

### 2.4 Test SMTP from GitLab

```bash
sudo gitlab-rails console -e production
```

```ruby
# Confirm GitLab is using SMTP
ActionMailer::Base.delivery_method
ActionMailer::Base.smtp_settings

# Send a test email
Notify.test_email('you@example.com', 'SMTP Test', 'Hello from GitLab').deliver_now
```

### 2.5 If GitLab says “sent” but no mail received

1) Check **SendGrid Email Activity** events (Processed vs Delivered vs Dropped, etc.). citeturn9search41  
2) If status is **Processing** for long time, check if account is **Under Review / Suspended**. citeturn9search42turn9search53

---

## 3) GitLab behind GCP HTTPS Load Balancer (TLS termination)

### 3.1 Recommended pattern

- **TLS terminates at the GCP HTTPS Load Balancer (frontend 443).**
- LB forwards **HTTP (80)** to GitLab backend.
- **Disable GitLab Let’s Encrypt** inside the VM (HTTP‑01 challenges commonly fail behind LBs / reverse proxies).

### 3.2 GitLab reverse proxy awareness (typical settings)

```ruby
external_url "https://gitlab.tcsvmo2trg.com"

letsencrypt['enable'] = false

nginx['listen_port'] = 80
nginx['listen_https'] = false

nginx['proxy_set_headers'] = {
  "X-Forwarded-Proto" => "https",
  "X-Forwarded-Ssl"   => "on"
}

# Adjust as needed to include your LB/proxy CIDRs
# gitlab_rails['trusted_proxies'] = ['10.0.0.0/8','172.16.0.0/12','192.168.0.0/16']
```

Apply:

```bash
sudo gitlab-ctl reconfigure
sudo gitlab-ctl restart
```

---

## 4) Cloud Build ↔ Self‑Managed GitLab: KMS + IAM + Tokens

### 4.1 Enable required APIs

```bash
gcloud services enable cloudkms.googleapis.com secretmanager.googleapis.com cloudbuild.googleapis.com
```

### 4.2 Important: KMS key location must match connection region

- Cloud Build host connections are **regional**, and the selected KMS key must be in the **same region** (example: `europe-west2`).

### 4.3 Create KMS KeyRing + Key (regional)

```bash
# Example region: europe-west2
KMS_LOCATION="europe-west2"
KEYRING="cloudbuild-gitlab-kr-ew2"
KEY="cloudbuild-gitlab-key"

gcloud kms keyrings create "$KEYRING" --location="$KMS_LOCATION"
gcloud kms keys create "$KEY" --location="$KMS_LOCATION" --keyring="$KEYRING" --purpose=encryption
```

### 4.4 Grant KMS permissions to service agents (Cloud Build + Secret Manager)

```bash
PROJECT_ID=$(gcloud config get-value project)
PROJECT_NUMBER=$(gcloud projects describe "$PROJECT_ID" --format="value(projectNumber)")

# Secret Manager service agent
gcloud kms keys add-iam-policy-binding "$KEY" \
  --location="$KMS_LOCATION" --keyring="$KEYRING" \
  --member="serviceAccount:service-${PROJECT_NUMBER}@gcp-sa-secretmanager.iam.gserviceaccount.com" \
  --role="roles/cloudkms.cryptoKeyEncrypterDecrypter"

# Cloud Build service agent
gcloud kms keys add-iam-policy-binding "$KEY" \
  --location="$KMS_LOCATION" --keyring="$KEYRING" \
  --member="serviceAccount:service-${PROJECT_NUMBER}@gcp-sa-cloudbuild.iam.gserviceaccount.com" \
  --role="roles/cloudkms.cryptoKeyEncrypterDecrypter"

# Verify
gcloud kms keys get-iam-policy "$KEY" --location="$KMS_LOCATION" --keyring="$KEYRING"
```

### 4.5 GitLab tokens required by Cloud Build

Create a dedicated GitLab **service user** (recommended) and add it as **Maintainer** to at least one project/group.

- **API access token**: scope `api` (full API) 
- **Read API access token**: scope `read_api`

> Cloud Build validates the authorizer token by checking it can access at least one repository; role **Maintainer+** typically required.

### 4.6 GitLab Group / Project / Membership quick steps (UI)

1) **Create group**: *Groups → New group* 
2) **Create project** inside the group: *Group → New project* 
3) **Create service user**: *Admin Area → Users → New user* 
4) **Add user to group**: *Group → Members → Invite → Role = Maintainer*

---

## 5) Cloud Build triggers (GitLab)

### 5.1 Create trigger (Console)

1) Cloud Build → **Triggers** → **Create trigger**
2) Source: **GitLab (self-managed host connection)**
3) Event: **Push to branch**
4) Branch regex: `^main$` (or your branch)
5) Config: **cloudbuild.yaml** (repo root)

---

## 6) Multi‑Dockerfile mono‑repo: build & push 2 images + deploy to GKE

### 6.1 Naming recommendation

Use hierarchical names (keeps images grouped):

- `gcr.io/$PROJECT_ID/sample-app/frontend:$COMMIT_SHA`
- `gcr.io/$PROJECT_ID/sample-app/backend:$COMMIT_SHA`

> In Container Registry, repositories are created implicitly when you push the first image.

### 6.2 Kubernetes manifests (placeholders)

In `k8s/frontend-deployment.yaml`:

```yaml
image: FRONTEND_IMAGE
```

In `k8s/backend-deployment.yaml`:

```yaml
image: BACKEND_IMAGE
```

### 6.3 Important Cloud Build substitution rule (why you hit FRONTEND_IMG error)

Cloud Build applies a substitutions pass to `$VARS` in build configs. Built‑in substitutions include `$PROJECT_ID`, `$COMMIT_SHA`, etc. citeturn23search10  
If you want bash to expand environment variables inside a step script, use `$$VAR` to escape Cloud Build substitution. citeturn23search13

### 6.4 Working `cloudbuild.yaml` (build + push + deploy)

```yaml
substitutions:
  _REGION: "europe-west2"
  _CLUSTER: "gke-vmo2-trg-test-cluster"
  _NAMESPACE: "default"

steps:
  - name: "gcr.io/cloud-builders/docker"
    id: "build-frontend"
    args:
      - "build"
      - "-t"
      - "gcr.io/$PROJECT_ID/sample-app/frontend:$COMMIT_SHA"
      - "-f"
      - "frontend/Dockerfile"
      - "frontend"

  - name: "gcr.io/cloud-builders/docker"
    id: "build-backend"
    args:
      - "build"
      - "-t"
      - "gcr.io/$PROJECT_ID/sample-app/backend:$COMMIT_SHA"
      - "-f"
      - "backend/Dockerfile"
      - "backend"

  - name: "gcr.io/cloud-builders/docker"
    id: "push-frontend"
    args:
      - "push"
      - "gcr.io/$PROJECT_ID/sample-app/frontend:$COMMIT_SHA"

  - name: "gcr.io/cloud-builders/docker"
    id: "push-backend"
    args:
      - "push"
      - "gcr.io/$PROJECT_ID/sample-app/backend:$COMMIT_SHA"

  - name: "gcr.io/cloud-builders/gcloud"
    id: "get-gke-creds"
    args:
      - "container"
      - "clusters"
      - "get-credentials"
      - "${_CLUSTER}"
      - "--region"
      - "${_REGION}"
      - "--project"
      - "$PROJECT_ID"

  - name: "gcr.io/cloud-builders/kubectl"
    id: "deploy"
    entrypoint: "bash"
    env:
      - "CLOUDSDK_COMPUTE_REGION=${_REGION}"
      - "CLOUDSDK_CONTAINER_CLUSTER=${_CLUSTER}"
    args:
      - "-c"
      - |
        set -euo pipefail

        FRONTEND_IMG="gcr.io/$PROJECT_ID/sample-app/frontend:$COMMIT_SHA"
        BACKEND_IMG="gcr.io/$PROJECT_ID/sample-app/backend:$COMMIT_SHA"

        echo "Deploying:"
        echo "  FRONTEND: $$FRONTEND_IMG"
        echo "  BACKEND : $$BACKEND_IMG"

        sed -i "s|FRONTEND_IMAGE|$$FRONTEND_IMG|g" k8s/frontend-deployment.yaml
        sed -i "s|BACKEND_IMAGE|$$BACKEND_IMG|g"  k8s/backend-deployment.yaml

        kubectl apply -n "${_NAMESPACE}" -f k8s/frontend-deployment.yaml
        kubectl apply -n "${_NAMESPACE}" -f k8s/backend-deployment.yaml

images:
  - "gcr.io/$PROJECT_ID/sample-app/frontend:$COMMIT_SHA"
  - "gcr.io/$PROJECT_ID/sample-app/backend:$COMMIT_SHA"

options:
  logging: CLOUD_LOGGING_ONLY
```

---

## 7) Common troubleshooting quick checks

### 7.1 Cloud Build build fails before running steps

- Usually substitutions parsing error: validate custom substitutions start with `_` and escape bash variables as `$$VAR`. citeturn23search10turn23search13

### 7.2 GKE deploy failures

- If you see IAM errors (cannot get credentials), grant required GKE permissions to the Cloud Build service account.
- If you see Kubernetes `Forbidden`, create a ClusterRoleBinding for the Cloud Build identity to allow apply.

### 7.3 SendGrid emails not arriving

- Use SendGrid Email Activity to see whether an email is only **Processed** or actually **Delivered/Dropped/Deferred**. citeturn9search41
- If stuck in **Processing**, check account **Under Review** banner/ticket and resolve compliance. citeturn9search42turn9search53

---

## 8) Handy one‑liners

### 8.1 Test SendGrid SMTP connectivity from VM

```bash
nc -vz smtp.sendgrid.net 587
nc -vz smtp.sendgrid.net 465
```

### 8.2 Confirm GCP outbound SMTP ports guidance

- Ports 587/465 allowed; port 25 blocked externally by default. citeturn9search47

---

*End of runbook.*
