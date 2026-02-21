# 🔒 Azure Cloud Security — Defense in Depth with Network Security Groups

## 📐 Architecture Overview

```
                        ┌─────────────────┐
                        │    Internet      │
                        │   (Untrusted)    │
                        └────────┬────────┘
                                 │
                    ✅ HTTP/HTTPS (80, 443)
                    ✅ SSH from Admin IP only
                                 │
┌────────────────────────────────▼──────────────────────────────────┐
│                        Azure Cloud                                 │
│                                                                    │
│   ┌────────────────────────────────────────────────────────────┐  │
│   │              CloudSecLab-VNet (10.0.0.0/16)                │  │
│   │                                                            │  │
│   │   ┌─────────────────────┐    ┌─────────────────────────┐  │  │
│   │   │   WebTier-Subnet    │    │    DataTier-Subnet       │  │  │
│   │   │   (10.0.1.0/24)     │    │    (10.0.2.0/24)         │  │  │
│   │   │                     │    │                          │  │  │
│   │   │  ┌───────────────┐  │    │  ┌───────────────────┐  │  │  │
│   │   │  │ WebServer-VM  │  │MySQL│  │   DBServer-VM     │  │  │  │
│   │   │  │ Ubuntu 22.04  ├──┼────┼──►   Ubuntu 22.04    │  │  │  │
│   │   │  │ Apache HTTP   │  │3306│  │   MySQL Database   │  │  │  │
│   │   │  │ 10.0.1.4      │  │    │  │   10.0.2.4         │  │  │  │
│   │   │  │ Public IP ✅  │  │    │  │   No Public IP ❌  │  │  │  │
│   │   │  └───────────────┘  │    │  └───────────────────┘  │  │  │
│   │   │                     │    │                          │  │  │
│   │   │  WebTier-NSG        │    │  DataTier-NSG            │  │  │
│   │   │  ✅ HTTP (80)       │    │  ✅ MySQL from WebTier   │  │  │
│   │   │  ✅ HTTPS (443)     │    │  ✅ SSH from WebTier     │  │  │
│   │   │  ✅ SSH (My IP)     │    │  ❌ Deny All Others      │  │  │
│   │   │  ❌ Deny All        │    │                          │  │  │
│   │   └─────────────────────┘    └─────────────────────────┘  │  │
│   └────────────────────────────────────────────────────────────┘  │
└───────────────────────────────────────────────────────────────────┘
```

---

## 🎯 Objectives

- Design and deploy a secure two-tier cloud infrastructure on Microsoft Azure
- Implement Network Security Groups (NSGs) with granular inbound and outbound rules
- Apply the **principle of least privilege** across all network access controls
- Demonstrate **defense-in-depth** by layering multiple security controls
- Isolate the database tier from the public internet using private-only networking
- Validate security controls by testing both allowed and denied traffic flows

---

## 🛡️ Security Principles Applied

| Principle | Implementation |
|---|---|
| **Least Privilege** | Each NSG rule grants only the minimum access required — no blanket allow-all rules |
| **Defense in Depth** | Multiple security layers: NSGs at subnet level, no public IP on DB, port restrictions |
| **Network Segmentation** | Web and database tiers isolated in separate subnets with independent NSG policies |
| **Zero Direct DB Access** | Database server has no public IP — unreachable from internet under any circumstance |
| **Admin Access Control** | SSH restricted to a specific administrator IP address only |

---

## 🏗️ Infrastructure Components

### Resource Group
| Setting | Value |
|---|---|
| Name | `CloudSecLab-RG` |
| Purpose | Logical container for all lab resources — delete the RG to remove everything |

### Virtual Network
| Setting | Value |
|---|---|
| Name | `CloudSecLab-VNet` |
| Address Space | `10.0.0.0/16` (65,536 IPs) |

### Subnets
| Subnet | CIDR | Purpose |
|---|---|---|
| `WebTier-Subnet` | `10.0.1.0/24` | Hosts the public-facing web server |
| `DataTier-Subnet` | `10.0.2.0/24` | Hosts the private database server |

### Virtual Machines
| VM | OS | Role | Private IP | Public IP |
|---|---|---|---|---|
| `WebServer-VM` | Ubuntu 22.04 LTS | Apache Web Server | `10.0.1.4` | ✅ Assigned |
| `DBServer-VM` | Ubuntu 22.04 LTS | MySQL Database | `10.0.2.4` | ❌ None |

---

## 🔥 Network Security Group Rules

### WebTier-NSG
| Priority | Rule Name | Port | Protocol | Source | Action |
|---|---|---|---|---|---|
| 100 | `Allow-HTTP-Internet` | 80 | TCP | Any | ✅ Allow |
| 110 | `Allow-HTTPS-Internet` | 443 | TCP | Any | ✅ Allow |
| 120 | `Allow-SSH-MyIP` | 22 | TCP | Admin IP only | ✅ Allow |
| 4000 | `Deny-All-Inbound` | * | Any | Any | ❌ Deny |

### DataTier-NSG
| Priority | Rule Name | Port | Protocol | Source | Action |
|---|---|---|---|---|---|
| 100 | `Allow-MySQL-WebTier` | 3306 | TCP | `10.0.1.0/24` | ✅ Allow |
| 110 | `Allow-SSH-WebTier` | 22 | TCP | `10.0.1.0/24` | ✅ Allow |
| 4000 | `Deny-All-Inbound` | * | Any | Any | ❌ Deny |

---

## 🧪 Security Validation Tests

| Test | From | To | Expected | Result |
|---|---|---|---|---|
| HTTP access | Internet | WebServer-VM:80 | ✅ Success | Apache default page served |
| HTTPS access | Internet | WebServer-VM:443 | ✅ Success | Port open, ready for SSL cert |
| SSH from admin IP | Admin machine | WebServer-VM:22 | ✅ Success | Shell access granted |
| SSH from unknown IP | Any other IP | WebServer-VM:22 | ❌ Blocked | Connection timed out |
| MySQL from web tier | WebServer-VM | DBServer-VM:3306 | ✅ Success | Internal connection allowed |
| Direct DB access | Internet | DBServer-VM | ❌ Blocked | No public IP — unreachable |

---

## 📁 Repository Structure

```
azure-cloud-security-lab/
│
├── README.md
├── docs/
│   └── Cloud_Security-lab01.docx
├── diagrams/
│   └── cloud-security-lab-diagram.svg
├── screenshots/
│   ├── 01-resource-group.png
│   ├── 02-vnet-subnets.png
│   ├── 03-webserver-vm.png
│   ├── 04-dbserver-vm.png
│   ├── 05-webtier-nsg-rules.png
│   ├── 06-datatier-nsg-rules.png
│   ├── 07-ssh-test.png
│   └── 08-http-test.png
└── configs/
    ├── webtier-nsg-rules.json
    └── datatier-nsg-rules.json
```

---

## 🚀 How to Reproduce This Lab

### Prerequisites
- Active Microsoft Azure subscription (free trial works)
- Basic understanding of networking concepts (IP, subnets, TCP/IP ports)
- Azure CLI or access to Azure Portal

### Steps
1. Create Resource Group `CloudSecLab-RG`
2. Create Virtual Network `CloudSecLab-VNet` with address space `10.0.0.0/16`
3. Create `WebTier-Subnet` (`10.0.1.0/24`) and `DataTier-Subnet` (`10.0.2.0/24`)
4. Deploy `WebServer-VM` (Ubuntu 22.04) into `WebTier-Subnet` with a public IP
5. Deploy `DBServer-VM` (Ubuntu 22.04) into `DataTier-Subnet` with **no public IP**
6. Create and configure `WebTier-NSG` with rules defined above — associate to `WebTier-Subnet`
7. Create and configure `DataTier-NSG` with rules defined above — associate to `DataTier-Subnet`
8. Install Apache on `WebServer-VM` and MySQL on `DBServer-VM`
9. Run validation tests to confirm security controls are working

> ⚠️ Remember to delete `CloudSecLab-RG` after completing the lab to avoid ongoing Azure charges.

---

## 💡 Key Learnings

- Azure NSGs operate at both the **subnet level** and **NIC level** — when both are present, traffic must be permitted by both to flow through
- Assigning **no public IP** to a VM is the most effective way to prevent direct internet access — NSG rules alone are not sufficient if a public IP exists
- **Priority order matters** in NSG rules — lower numbers are evaluated first and the first match wins
- A `/32` CIDR notation restricts access to a single specific IP address — essential for locking down administrative SSH access
- **Defense in depth** means no single control is relied upon alone — multiple overlapping layers ensure that if one fails, others still protect the system

---

## 🛠️ Technologies Used

- Microsoft Azure (Virtual Network, VMs, NSGs, Public IP)
- Ubuntu Server 22.04 LTS
- Apache HTTP Server
- MySQL Database
- Azure Cloud Shell
- Bash / Linux CLI

---

## 👤 Author

**Waqas Salman**
Postgraduate Certificate — Cybersecurity & AI | Humber College
MSc Computer Science | Middlesex University

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-blue?style=flat&logo=linkedin)](https://linkedin.com/in/waqas-salman)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-black?style=flat&logo=github)](https://github.com/waqassalman)

---

*This lab was completed as part of the Cybersecurity & AI postgraduate program at Humber College, Toronto, ON.*
