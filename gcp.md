<div align="center">

# ☁️ GCP Cloud Shell Command Reference

### **IAM Custom Roles & Compute Engine CLI Guide**

[![GCP](https://img.shields.io/badge/Google_Cloud-4285F4?style=for-the-badge&logo=google-cloud&logoColor=white)](https://cloud.google.com/)
[![Bash](https://img.shields.io/badge/Bash-121011?style=for-the-badge&logo=gnu-bash&logoColor=white)](https://www.gnu.org/software/bash/)
[![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)](LICENSE)

*A complete, practical guide to using the **gcloud CLI** for everyday GCP tasks.*

---

[Getting Started](#-1-set-up--configure-your-gcp-project) •
[VM Commands](#-2-compute-engine-vm-commands) •
[IAM Roles](#-5-creating-iam-custom-roles) •
[Troubleshooting](#-6-common-errors--fixes)

</div>

---

## 📋 Table of Contents

- [1. Set Up & Configure Your GCP Project](#-1-set-up--configure-your-gcp-project)
- [2. Compute Engine VM Commands](#-2-compute-engine-vm-commands)
- [3. Listing Compute Engine Instances](#-3-listing-compute-engine-instances)
- [4. Understanding gcloud Help Output](#-4-understanding-gcloud-help-output)
- [5. Creating IAM Custom Roles](#-5-creating-iam-custom-roles)
- [6. Common Errors & Fixes](#-6-common-errors--fixes)
- [7. Best Practices](#-7-best-practices)

---

## 🚀 1. Set Up & Configure Your GCP Project

> Whether you're using a **Local Machine** or **Cloud Shell**, the first step is always to configure your project.

### 💻 Option A: From Local Machine (Windows/Mac)

Install [Google Cloud SDK](https://cloud.google.com/sdk/docs/install), then authenticate:

```bash
# Initialize gcloud and set up configuration
gcloud init

# Authenticate with your Google account
gcloud auth login
```

### ☁️ Option B: From Cloud Shell (Recommended)

Cloud Shell comes pre-configured with:
- ✅ Pre-installed `gcloud`, `kubectl`, `terraform`, `python`
- ✅ Permanent **5 GB** home directory at `/home/<username>`
- ✅ No installation required

### ⚙️ Project Configuration Commands

```bash
# Set your default project
gcloud config set project PROJECT_ID

# Verify the active project
gcloud config get-value project

# List all available projects
gcloud projects list
```

---

## 🖥️ 2. Compute Engine VM Commands

> Convert common **UI actions** into powerful **gcloud commands**.

### 🟢 A) Create a VM Instance

**UI Equivalent:** `Compute Engine → VM Instances → Create Instance`

```bash
gcloud compute instances create demo-vm \
  --project=flipkart10com \
  --zone=us-central1-a \
  --machine-type=e2-medium \
  --image-family=debian-12 \
  --image-project=debian-cloud \
  --boot-disk-size=20GB \
  --boot-disk-type=pd-balanced
```

<details>
<summary>📖 <b>What this command does</b></summary>

| Parameter | Description |
|-----------|-------------|
| `demo-vm` | Name of the VM instance |
| `--zone` | Deployment zone (`us-central1-a`) |
| `--machine-type` | VM size (`e2-medium`) |
| `--image-family` | Operating system (Debian 12) |
| `--boot-disk-size` | Disk size (20GB) |
| `--boot-disk-type` | Disk type (Balanced SSD) |

</details>

---

### 🟠 B) Stop a VM Instance

**UI Equivalent:** `VM Instances → Select VM → STOP`

```bash
gcloud compute instances stop demo-vm \
  --project=flipkart10com \
  --zone=us-central1-a
```

> 💡 **Tip:** Stopped VMs don't incur compute charges, only disk storage costs.

---

### 🔵 C) Start a VM Instance

**UI Equivalent:** `VM Instances → Select VM → START`

```bash
gcloud compute instances start demo-vm \
  --project=flipkart10com \
  --zone=us-central1-a
```

---

### 🟣 D) Change Machine Type

**UI Equivalent:** `Edit VM → Machine Configuration → Change machine type`

> ⚠️ **Important:** VM must be **stopped** before changing machine type!

```bash
# Step 1: Stop the VM
gcloud compute instances stop demo-vm \
  --project=flipkart10com \
  --zone=us-central1-a

# Step 2: Change machine type
gcloud compute instances set-machine-type demo-vm \
  --project=flipkart10com \
  --zone=us-central1-a \
  --machine-type=n1-standard-1

# Step 3: Start the VM again
gcloud compute instances start demo-vm \
  --project=flipkart10com \
  --zone=us-central1-a
```

---

### 🔴 E) Delete a VM Instance

**UI Equivalent:** `VM Instances → Select VM → DELETE`

```bash
gcloud compute instances delete demo-vm \
  --project=flipkart10com \
  --zone=us-central1-a \
  --quiet
```

> 💡 The `--quiet` flag skips the confirmation prompt.

---

## 📊 3. Listing Compute Engine Instances

```bash
# List all instances across all zones
gcloud compute instances list

# Filter instances by region
gcloud compute instances list --filter="zone:us-central1*"

# List with specific columns
gcloud compute instances list --format="table(name,zone,status,machineType)"
```

---

## 📚 4. Understanding gcloud Help Output

When you search for gcloud commands, you'll see help output like this:

```
gcloud compute instances list [NAME ...]
   [--regexp=REGEXP]
   [--zones=ZONE,...]
   [--filter=EXPRESSION]
   [--limit=LIMIT]
   [--sort-by=FIELD]
```

| Symbol | Meaning |
|--------|---------|
| `[NAME ...]` | Optional positional arguments |
| `[--flag=VALUE]` | Optional flags with values |
| `--filter` | Filter results |
| `--limit` | Limit number of results |

```bash
# Get help for any command
gcloud compute instances --help
gcloud compute instances create --help
```

---

## 🔐 5. Creating IAM Custom Roles

> Two methods: **YAML-based** (recommended for version control) or **CLI arguments** (quick usage)

### 📄 Method A: YAML-Based Role Creation

**Step 1:** Create the YAML file

```bash
nano role.yaml
```

**Step 2:** Add the role definition

```yaml
title: "Custom VM Creator Role"
description: "Role to create instances without deletion capability"
stage: "GA"
includedPermissions:
  - compute.instances.create
  - compute.instances.list
  - compute.instances.setServiceAccount
  - compute.acceleratorTypes.list
  - compute.disks.create
  - compute.disks.list
  - compute.machineTypes.list
  - compute.networks.get
  - compute.networks.list
  - compute.projects.get
  - compute.regions.list
  - compute.subnetworks.get
  - compute.subnetworks.list
  - compute.subnetworks.use
  - compute.subnetworks.useExternalIp
  - compute.zones.list
```

**Step 3:** Create the role

```bash
gcloud iam roles create vmCreatorRole \
  --file=role.yaml \
  --project=flipkart10com
```

**Step 4:** Verify the role

```bash
# View role details
gcloud iam roles describe vmCreatorRole --project=flipkart10com

# List all custom roles in project
gcloud iam roles list --project=flipkart10com
```

---

### ⚡ Method B: CLI Arguments Only

```bash
gcloud iam roles create vmOperatorBasic \
  --project=flipkart10com \
  --title="VM Operator Basic" \
  --description="Role to start, stop, and list VM instances only" \
  --stage="GA" \
  --permissions="compute.instances.start,compute.instances.stop,compute.instances.list,compute.zones.list"
```

---

### 🔄 Update an Existing Role

```bash
gcloud iam roles update vmOperatorBasic \
  --project=flipkart10com \
  --add-permissions=compute.instances.get,compute.instances.setMetadata
```

---

## ❌ 6. Common Errors & Fixes

| Error | Cause | Solution |
|:------|:------|:---------|
| `Invalid choice: 'role'` | Wrong command syntax | Use `gcloud iam roles create` |
| `API not enabled` | Compute API disabled | `gcloud services enable compute.googleapis.com` |
| `Permission denied` | Insufficient permissions | Need **Owner** or **iam.roleAdmin** role |
| `YAML file not found` | Wrong file path | Verify path with `--file=role.yaml` |
| `Project mismatch` | Wrong project context | Run `gcloud config get-value project` |

---

## ✅ 7. Best Practices

| Practice | Description |
|:---------|:------------|
| 🎯 **Set Default Project** | Always configure project first with `gcloud config set project` |
| 📁 **Use YAML for Roles** | Better version control and team collaboration |
| 🔒 **Least Privilege** | Grant only necessary permissions |
| 📂 **Organize Files** | Store YAML files in `/roles` directory in your repo |
| ☁️ **Use Cloud Shell** | Ideal for learning, training, and quick tasks |

---

## 📁 Suggested Repository Structure

```
gcp-commands/
├── README.md
├── roles/
│   ├── vm-creator-role.yaml
│   ├── vm-operator-role.yaml
│   └── storage-admin-role.yaml
├── scripts/
│   ├── create-vm.sh
│   ├── delete-vm.sh
│   └── setup-project.sh
└── docs/
    └── troubleshooting.md
```

---

## 🔗 Useful Links

- [GCloud CLI Documentation](https://cloud.google.com/sdk/gcloud/reference)
- [Compute Engine Documentation](https://cloud.google.com/compute/docs)
- [IAM Custom Roles Guide](https://cloud.google.com/iam/docs/creating-custom-roles)
- [GCloud Cheat Sheet](https://cloud.google.com/sdk/docs/cheatsheet)

---

<div align="center">

### ⭐ Star this repo if you found it helpful!

**Created with ❤️ by devopsbyrushi**

[![Follow](https://img.shields.io/badge/Follow-devopsbyrushi-blue?style=social&logo=github)](https://github.com/devopsbyrushi)

*DevOps | Cloud | Trainer*

</div>
