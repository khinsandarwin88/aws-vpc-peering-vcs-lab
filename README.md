# Multi-Workspace VPC Peering Lab with HCP Terraform, OPA, and GitHub Branches

Welcome to our Terraform project! This lab demonstrates how to manage a multi-tier AWS network using **HCP Terraform**, **Open Policy Agent (OPA)** for security, and a **Branch-per-Workspace** workflow in GitHub.

---

## 1. Project Overview & Architecture

We have set up two isolated AWS VPCs connected securely through VPC Peering in the `eu-west-1` region:
* **Frontend VPC:** `10.0.0.0/16`
* **Backend VPC:** `192.168.0.0/16`

Instead of keeping everything in one single branch, we use a **Branch-per-Workspace** approach where each HCP Terraform workspace is linked to its own specific Git branch.

---

## 2. Workspace & Branch Mapping

Here is how our workspaces are connected to GitHub branches and directories:

| Order | Workspace Name | GitHub Branch | Directory | What it deploys |
| :--- | :--- | :--- | :--- | :--- |
| 1 | `fe-network-ws` | `fe-network` | `fe-network/` | Frontend VPC, Subnets, IGW, NAT |
| 2 | `be-network-ws` | `be-network` | `be-network/` | Backend VPC, Subnets, Peering |
| 3 | `fe-security-ws` | `fe-security` | `fe-security/` | Frontend Security Groups |
| 4 | `be-security-ws` | `be-security` | `be-security/` | Backend Security Groups + Cross-VPC rules |
| 5 | `fe-compute-ws` | `fe-compute` | `fe-compute/` | Frontend Bastion and Private Instances |
| 6 | `be-compute-ws` | `be-compute` | `be-compute/` | Backend Private Instances |

---

## 3. Security & Policy as Code (OPA)

To keep our infrastructure safe, we integrated **Open Policy Agent (OPA)** rules:
* **No Public SSH:** Blocking `0.0.0.0/0` ingress on Port 22.
* **Subnet Limits:** Keeping the subnet counts safe and controlled.
* **Guardrail Enforcement:** If a rule is violated (like opening SSH to the internet), HCP Terraform will automatically fail the plan step and block the deployment.

---

## 4. Step-by-Step Workflow Guide

To make changes or test this project, follow these simple steps:

1. **Switch to your component branch:**
   ```bash
   git checkout fe-network
   ```
2. **Make your changes** inside the respective folder (e.g., `fe-network/`).
3. **Add and commit your changes:**
   ```bash
   git add fe-network/
   git commit -m "Update frontend network settings"
   ```
4. **Push to GitHub:**
   ```bash
   git origin fe-network
   ```
5. **Check HCP Terraform:** 
   * Go to your HCP Terraform dashboard.
   * Open `fe-network-ws` to see the automated plan, OPA check, and apply your changes.

---

## 5. Clean Up (Destroy Order)

When you are finished with the lab, destroy your resources in reverse order to avoid dependency errors:
1. `be-compute-ws`
2. `fe-compute-ws`
3. `be-security-ws`
4. `fe-security-ws`
5. `be-network-ws`
6. `fe-network-ws`