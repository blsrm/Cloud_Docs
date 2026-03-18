# GCS Signed URL – Step‑by‑Step Reference Guide

## Overview
This document explains how to securely share **individual Google Cloud Storage objects** using **Signed URLs** while **Uniform Bucket‑Level Access (UBLA)** is enabled.

This is the **recommended enterprise approach** for object‑level public access without exposing the bucket.

---

## Prerequisites
- GCP Project access
- Cloud Shell or local machine with `gcloud` and `gsutil`
- Required IAM role:
  - `roles/storage.objectAdmin` **or** `roles/storage.admin`

---

## 1. Check Uniform Bucket‑Level Access (UBLA)

```bash
gsutil uniformbucketlevelaccess get gs://vmo2-trg-data-01
```

Expected output:
```text
Enabled: True
```

> ✅ Signed URLs **work with UBLA enabled**.

---

## 2. Why Object‑Level IAM Does Not Work

When UBLA is enabled:
- ❌ Object‑level IAM policies are disabled
- ✅ Bucket‑level IAM + Signed URLs are supported

Error example:
```text
Object policies are disabled for bucket when uniform bucket‑level access is enabled
```

---

## 3. Create Service Account for Signed URLs

```bash
gcloud iam service-accounts create gcs-signedurl-sa   --display-name="GCS Signed URL Service Account"
```

---

## 4. Grant Bucket Read Permission

```bash
gsutil iam ch serviceAccount:gcs-signedurl-sa@$(gcloud config get-value project).iam.gserviceaccount.com:roles/storage.objectViewer  gs://vmo2-trg-data-01
```

✅ Read‑only access (secure)

---

## 5. Create Service Account Key (JSON)

```bash
gcloud iam service-accounts keys create ~/gcs-signedurl-key.json   --iam-account=gcs-signedurl-sa@$(gcloud config get-value project).iam.gserviceaccount.com
```

Verify:
```bash
ls -l ~/gcs-signedurl-key.json
```

---

## 6. Generate Signed URL (24 Hours)

```bash
gsutil signurl -d 24h ~/gcs-signedurl-key.json "gs://vmo2-trg-data-01/VMO2 DevOps Sessions – End-to-End CICD Overview.mp4"
```

Output:
```text
https://storage.googleapis.com/vmo2-trg-data-01/...X-Goog-Signature=...
```

✅ Share this URL publicly
✅ No login required

---

## 7. Generate Signed URL (7 Days)

```bash
gsutil signurl -d 7d ~/gcs-signedurl-key.json "gs://vmo2-trg-data-01/VMO2 DevOps Sessions – End-to-End CICD Overview.mp4"
```

---

## 8. Filename Handling Notes

If the object name contains:
- Spaces
- Special characters (e.g., EN DASH `–`)

✅ Always wrap the path in **double quotes**

---

## 9. Security Best Practices

- ✅ Keep UBLA enabled
- ✅ Use service account keys only for signing
- ✅ Rotate and delete keys when no longer required

### List keys
```bash
gcloud iam service-accounts keys list --iam-account=gcs-signedurl-sa@$(gcloud config get-value project).iam.gserviceaccount.com
```

### Delete key
```bash
gcloud iam service-accounts keys delete KEY_ID --iam-account=gcs-signedurl-sa@$(gcloud config get-value project).iam.gserviceaccount.com
```

---

## 10. Decision Summary

| Requirement | Best Solution |
|------------|---------------|
| Single object public | ✅ Signed URL |
| Temporary access | ✅ Signed URL |
| UBLA enabled | ✅ Supported |
| Production security | ✅ Recommended |

---

## One‑Line Executive Summary
> With Uniform Bucket‑Level Access enabled, individual GCS objects are securely shared using time‑bound Signed URLs generated via a service account, without exposing the bucket publicly.

---

## References
- https://cloud.google.com/storage/docs/uniform-bucket-level-access
- https://cloud.google.com/storage/docs/access-control/signing-urls-with-helpers
