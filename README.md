# AWS VPC Peering & HCP Terraform VCS-Driven Lab Report

## 1. Executive Summary

This project demonstrates an enterprise-grade Infrastructure-as-Code (IaC) deployment of an intra-region AWS VPC Peering architecture in `eu-west-1` (Ireland). It uses a GitOps workflow powered by HCP Terraform VCS-driven runs, a Branch-per-Workspace strategy in a monorepo, OIDC dynamic credentials, AWS Secrets Manager for ephemeral SSH key management, and Open Policy Agent (OPA) for automated compliance enforcement.

## 2. Architecture & Components

- **Cloud Provider:** Amazon Web Services (AWS)
- **Region:** `eu-west-1` (Ireland)
- **Frontend VPC:** `10.0.0.0/16` (public subnets, NAT gateway, bastion host, and private subnets)
- **Backend VPC:** `192.168.0.0/16` (private subnets and peered route connections)
- **Orchestration:** HCP Terraform workspaces linked via VCS
- **Security & Governance:** AWS IAM OIDC, AWS Secrets Manager, and OPA (Open Policy Agent)

## 3. Branch-per-Workspace Strategy & Monorepo Mapping

To maintain strict isolation and independent state management, each Git branch maps directly to a designated HCP Terraform workspace and working directory:

| Workspace Name | Working Directory | Associated Git Branch | Purpose & Managed Resources |
| :--- | :--- | :--- | :--- |
| `fe-network-ws` | `fe-network` | `fe-network` | Frontend VPC, subnets, Internet Gateway, NAT Gateway, route tables |
| `be-network-ws` | `be-network` | `be-network` | Backend VPC, subnets, IGW, NAT, and VPC Peering connection |
| `fe-security-ws` | `fe-security` | `fe-security` | Frontend security groups and inbound/outbound rules |
| `be-security-ws` | `be-security` | `be-security` | Backend security groups and cross-VPC security rules |
| `fe-compute-ws` | `fe-compute` | `fe-compute` | Frontend EC2 instances, public bastion, and ephemeral SSH key binding |
| `be-compute-ws` | `be-compute` | `be-compute` | Backend private EC2 instances and ephemeral SSH key binding |

## 4. Security Implementation

1. **No Static Credentials:** Authentication to AWS is handled via OIDC dynamic credentials, eliminating long-lived static AWS access keys.
2. **Ephemeral Secrets:** SSH private keys are securely generated, stored in AWS Secrets Manager, and read ephemerally only during instance provisioning.
3. **Policy as Code (OPA):** Open Policy Agent enforces mandatory rules (for example, blocking public ingress `0.0.0.0/0` on private security groups) so that non-compliant Terraform plans fail and are blocked automatically.

## 5. Step-by-Step Operating Procedures (SOP)

### Step 1: Clone the Repository

Clone the repository to your local development environment:

```bash
git clone https://github.com/khinsandarwin88/aws-vpc-peering-vcs-lab.git
cd aws-vpc-peering-vcs-lab
```

### Step 2: Create and Push Component Branches

Create a dedicated branch for each component to maintain the Branch-per-Workspace workflow. For a single branch:

```bash
git checkout main
git checkout -b fe-network
git push -u origin fe-network
```

To create all six branches at once:

```bash
git checkout main
for b in fe-network fe-security fe-compute be-network be-security be-compute; do
  git checkout -b "$b" main
  git push -u origin "$b"
  git checkout main
done
```

### Step 3: Configure HCP Terraform Workspaces

1. Log in to the HCP Terraform dashboard.
2. Create or select your project and set up 6 individual workspaces matching your components.
3. For each workspace, go to **Settings → Version Control** and set:
   - **VCS branch:** the matching branch (for example, `fe-network`)
   - **Terraform working directory:** the matching folder (for example, `fe-network`)
   - **VCS trigger:** only trigger runs when files in specified paths change (for example, `fe-network/**/*`)

### Step 4: Enforce OPA Policies

1. Define your OPA Rego security policies (for example, `network-guardrails.rego`) in HCP Terraform under organization-managed policies.
2. Set the enforcement behavior to **Mandatory**.
3. Attach the policy set to the relevant workspaces (`fe-network-ws` and `fe-security-ws`).

### Step 5: Execute the GitOps Deployment Workflow

1. Make the necessary infrastructure code changes inside a specific component directory.
2. Commit and push your changes to the designated component branch.
3. Open a Pull Request targeting the `main` branch to review the automated HCP Terraform speculative plan and OPA check results.

### Step 6: Verify and Sync the Local Repository with Remote

If you hit remote divergence or need to reconcile local changes before pushing:

```bash
git add README.md
git commit -m "Update complete README file"
git pull origin main --rebase
git push origin main
```
