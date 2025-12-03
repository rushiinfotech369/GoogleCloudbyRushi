# 🌐 Google Cloud VPC, Subnets, Firewall & VM Networking

> **A beginner-friendly guide to GCP networking fundamentals**  
> Created by: **DevOpsByRushi**

---

## 📚 Table of Contents

- [VPC Basics](#-1-vpc-basics)
- [Firewall Concepts](#-2-firewall-concepts)
- [Hands-On: Create VPC, Subnets & VMs](#-3-hands-on-create-vpc-subnets--vms)
- [Firewall Rules](#-4-firewall-rules-vpc-level)
- [VM-to-VM Communication](#-5-vm-to-vm-communication)
- [Firewall Rule vs Policy](#-6-firewall-rule-vs-firewall-policy)
- [Architecture Diagram](#-7-architecture-diagram)
- [Tag-Based Firewall](#-8-vm-level-firewall-using-tags)

---

## 🌐 1. VPC Basics

### Default VPC

Google Cloud provides a **Default VPC** with:
- Auto-created subnets in all regions
- Default firewall rules allowing SSH (22), RDP (3389), ICMP (ping), and internal VM communication

> ✅ VMs in default VPC can be accessed from the internet immediately.

### Custom VPC

When you create your own VPC:
- No default firewall rules
- No default subnets

> ⚠️ VM access fails until firewall rules are explicitly created.

---

## 🔥 2. Firewall Concepts

| Term | Meaning |
|------|---------|
| **Ingress** | Traffic entering VM (e.g., SSH, HTTP) |
| **Egress** | Traffic leaving VM (e.g., outbound requests) |

Firewalls can be applied at:
- **VPC Level** — applies to all VMs in the network
- **VM Level** — using network tags (covered in Section 8)

---

## 🚀 3. Hands-On: Create VPC, Subnets & VMs

> 💡 CLI is preferred over UI because the console UI changes frequently.

### 3.1 Create Custom VPC

```bash
gcloud compute networks create dev-vpc \
  --subnet-mode=custom
```

### 3.2 Create Subnets

**US Subnet (Application VM)**
```bash
gcloud compute networks subnets create dev-subnet-us \
  --network=dev-vpc \
  --region=us-central1 \
  --range=10.10.0.0/24
```

**Asia Subnet (Database VM)**
```bash
gcloud compute networks subnets create dev-subnet-asia \
  --network=dev-vpc \
  --region=asia-southeast1 \
  --range=10.20.0.0/24
```

**Secure Subnet (Admin VM)**
```bash
gcloud compute networks subnets create dev-subnet-secure \
  --network=dev-vpc \
  --region=asia-southeast1 \
  --range=10.30.0.0/24
```

### 3.3 Create VMs

**Application VM (US)**
```bash
gcloud compute instances create app-server-us \
  --zone=us-central1-a \
  --machine-type=e2-micro \
  --subnet=dev-subnet-us
```

**Database VM (Asia)**
```bash
gcloud compute instances create db-server-asia \
  --zone=asia-southeast1-a \
  --machine-type=e2-micro \
  --subnet=dev-subnet-asia
```

**Secure Admin VM**
```bash
gcloud compute instances create secure-admin-vm \
  --zone=asia-southeast1-b \
  --machine-type=e2-micro \
  --subnet=dev-subnet-secure
```

---

## 🔐 4. Firewall Rules (VPC Level)

**Allow SSH (Port 22)**
```bash
gcloud compute firewall-rules create allow-ssh \
  --network=dev-vpc \
  --direction=INGRESS \
  --priority=1000 \
  --action=ALLOW \
  --rules=tcp:22 \
  --source-ranges=0.0.0.0/0
```

**Allow ICMP (Ping)**
```bash
gcloud compute firewall-rules create allow-icmp \
  --network=dev-vpc \
  --direction=INGRESS \
  --priority=1000 \
  --action=ALLOW \
  --rules=icmp \
  --source-ranges=0.0.0.0/0
```

---

## 🔄 5. VM-to-VM Communication

VMs inside the same VPC can communicate using **private IPs**, even across different regions.

```bash
# From app-server-us, ping the database server
ping <db-server-asia-private-ip>
```

---

## 🛡 6. Firewall Rule vs Firewall Policy

| Firewall Rule | Firewall Policy |
|---------------|-----------------|
| Applies to a single VPC | Applies to Organization/Folder |
| Local network rule | Centralized company-wide rule |
| Created by DevOps team | Created by Security team |

**Quick Summary:**
- `Firewall Rule` = Local VPC rule
- `Firewall Policy` = Organization-wide rule

---

## 📊 7. Architecture Diagram

```
                            ┌─────────────────────┐
                            │       dev-vpc       │
                            └─────────────────────┘
                                      │
            ┌─────────────────────────┼─────────────────────────┐
            │                         │                         │
            ▼                         ▼                         ▼
┌───────────────────┐     ┌───────────────────┐     ┌───────────────────┐
│   dev-subnet-us   │     │  dev-subnet-asia  │     │ dev-subnet-secure │
│   10.10.0.0/24    │     │   10.20.0.0/24    │     │   10.30.0.0/24    │
│   (us-central1)   │     │ (asia-southeast1) │     │ (asia-southeast1) │
└─────────┬─────────┘     └─────────┬─────────┘     └─────────┬─────────┘
          │                         │                         │
          ▼                         ▼                         ▼
┌───────────────────┐     ┌───────────────────┐     ┌───────────────────┐
│   app-server-us   │◄───►│  db-server-asia   │     │  secure-admin-vm  │
│  (Application)    │     │    (Database)     │     │     (Admin)       │
└───────────────────┘     └───────────────────┘     └───────────────────┘
```

---

## 🔖 8. VM-Level Firewall Using Tags

Tags allow you to apply firewall rules to specific VMs only.

**Step 1: Add tag to the VM**
```bash
gcloud compute instances add-tags secure-admin-vm \
  --zone=asia-southeast1-b \
  --tags=restricted-access
```

**Step 2: Create firewall rule targeting the tag**
```bash
gcloud compute firewall-rules create allow-ssh-restricted \
  --network=dev-vpc \
  --direction=INGRESS \
  --priority=1000 \
  --action=ALLOW \
  --rules=tcp:22 \
  --source-ranges=0.0.0.0/0 \
  --target-tags=restricted-access
```

> ✅ Only `secure-admin-vm` can be accessed via SSH.  
> ✅ Other VMs remain protected.

---

## 📝 Key Takeaways

| Best Practice | Why |
|---------------|-----|
| Always create firewall rules for SSH/ICMP in custom VPCs | Custom VPCs have no default rules |
| Use meaningful naming conventions | Easier to manage (e.g., `dev-vpc`, `app-server-us`) |
| Use tags to restrict VM access | More granular security control |
| Prefer CLI over UI | UI changes frequently; CLI is consistent |

---

## ✨ Author

**DevOps By Rushi**  
*Training • YouTube • Cloud • DevOps • Automation*

---

## 📄 License

This project is for educational purposes. Feel free to use and share!
