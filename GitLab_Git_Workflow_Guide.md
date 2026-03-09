# GitLab Workflow & Useful Git Commands

This document summarizes the **end-to-end Git workflow** used in this project, including
- cloning and pulling code
- creating folders/files
- committing and pushing changes
- working with **protected branches** using **feature branches + Merge Requests**
- understanding `origin` and `-u`
- adding approvals and process steps in GitLab

---

## Repository
```
http://gitlab.tcsvmo2trg.com/consumer/frontend/app1.git
```

---

## 1. One-time Git Configuration
Set your name and email (required for commits):

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@company.com"
```

Verify:
```bash
git config --global --list
```

---

## 2. Clone the Repository (First Time)
This downloads the repository to your local machine:

```bash
git clone http://gitlab.tcsvmo2trg.com/consumer/frontend/app1.git
cd app1
```

---

## 3. Pull Latest Changes (Daily Use)
Always pull before starting work:

```bash
git checkout main
git pull origin main
```

---

## 4. Create Folder and Files

### Create a folder
```bash
mkdir frontend
```

### Create a file
```bash
touch frontend/readme.md
```

Edit the file:
```bash
nano frontend/readme.md
```

---

## 5. Stage (Add) Changes

Add a specific file:
```bash
git add frontend/readme.md
```

Add a full folder:
```bash
git add frontend/
```

Add everything:
```bash
git add .
```

Check status:
```bash
git status
```

---

## 6. Commit Changes
Commit saves your changes locally:

```bash
git commit -m "Add initial frontend structure"
```

---

## 7. Protected Branch Issue (main)

If you see this error:
```
You are not allowed to push code to protected branches
```

It means **`main` is protected** and you must use a **feature branch + Merge Request**.

---

## 8. Feature Branch Workflow (Recommended)

### Create a feature branch
```bash
git checkout -b feature/add-frontend-initial
```

### Push feature branch
```bash
git push -u origin feature/add-frontend-initial
```

---

## 9. What does this command mean?

```bash
git push -u origin feature/add-frontend-initial
```

- **origin** → alias (name) for the remote GitLab repository URL
- **feature/add-frontend-initial** → branch being pushed
- **-u** → sets upstream tracking so future commands can be:

```bash
git push
git pull
```

Check remotes:
```bash
git remote -v
```

---

## 10. Merge Request (MR) Process

### Create Merge Request
1. GitLab → Project → Merge Requests
2. New Merge Request
3. Source: `feature/add-frontend-initial`
4. Target: `main`

### Recommended MR Description Template
```text
Purpose:
- Initial frontend setup

Changes:
- Added frontend folders and files

Impact:
- No production impact

Testing:
- Local testing completed

Approvals Required:
- Frontend Lead
- DevOps Lead
```

---

## 11. Adding Approvals

### A. MR-Level (Manual)
- Add **Reviewers / Approvers** while creating MR

### B. Project-Level (Maintainer only)
Path:
```
Settings → Merge Requests
```
Options:
- Required approvals count
- Prevent self-approval
- Require approval before merge

### C. Approval Rules
Path:
```
Settings → Merge Requests → Approval rules
```
Define:
- Who must approve
- How many approvals
- Which branches (e.g., main)

---

## 12. CODEOWNERS (Automatic Approvals)

Create file:
```
CODEOWNERS
```

Example:
```text
/frontend/   @frontend-lead
/devops/    @devops-team
```

Effect:
- GitLab automatically requests approval when matching files change

---

## 13. Adding Process Steps

### A. Merge Request Template
File:
```
.gitlab/merge_request_templates/default.md
```

Example:
```md
## Validation Steps
- [ ] Code reviewed
- [ ] Build successful
- [ ] No secrets committed
- [ ] Naming standards followed
```

---

## 14. CI/CD Steps (Optional)

Defined in `.gitlab-ci.yml`:

```yaml
stages:
  - build
  - test

build_frontend:
  stage: build
  script:
    - npm install
    - npm run build
```

---

## 15. Useful Git Commands

Check branch:
```bash
git branch
```

Show tracking:
```bash
git branch -vv
```

View recent commits:
```bash
git log --oneline -5
```

---

## 16. Standard Daily Workflow

```bash
git pull
git checkout -b feature/my-change
# make changes
git add .
git commit -m "Describe change"
git push -u origin feature/my-change
# create Merge Request
```

---

✅ This workflow ensures:
- No direct pushes to protected branches
- Mandatory reviews and approvals
- Full audit and compliance tracking

