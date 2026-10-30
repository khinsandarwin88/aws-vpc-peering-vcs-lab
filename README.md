# AWS VPC Peering & HCP Terraform VCS-Driven Lab Report

## 1. Executive Summary
This project demonstrates an enterprise-grade Infrastructure-as-Code (IaC) deployment of an intra-region AWS VPC Peering architecture in `eu-west-1` (Ireland). It utilizes a GitOps workflow powered by HCP Terraform VCS-driven runs, a Branch-per-Workspace strategy in a monorepo, OIDC dynamic credentials, AWS Secrets Manager for ephemeral SSH key management, and Open Policy Agent (OPA) for automated compliance enforcement.

## 2. Architecture & Components
* **Cloud Provider:** Amazon Web Services (AWS)
* **Region:** `eu-west-1` (Ireland)
* **Frontend VPC:** `10.0.0.0/16` (Public subnets, NAT gateway, bastion host, and private subnets)
* **Backend VPC:** `192.168.0.0/16` (Private subnets and peered route connections)
* **Orchestration:** HCP Terraform Workspaces linked via VCS
* **Security & Governance:** AWS IAM OIDC, AWS Secrets Manager, and OPA (Open Policy Agent)

## 3. Branch-per-Workspace Strategy & Monorepo Mapping
To maintain strict isolation and independent state management, individual Git branches map directly to designated HCP Terraform workspaces and working directories:

| Workspace Name | Working Directory | Associated Git Branch | Purpose & Managed Resources |
| :--- | :--- | :--- | :--- |
| `fe-network-ws` | `fe-network` | `fe-network` | Frontend VPC, subnets, Internet Gateway, NAT Gateway, Route Tables |
| `be-network-ws` | `be-network` | `be-network` | Backend VPC, subnets, IGW, NAT, and VPC Peering connection |
| `fe-security-ws` | `fe-security` | `fe-security` | Frontend Security Groups and inbound/outbound rules |
| `be-security-ws` | `be-security` | `be-security` | Backend Security Groups and cross-VPC security rules |
| `fe-compute-ws` | `fe-compute` | `fe-compute` | Frontend EC2 instances, public bastion, and ephemeral SSH key binding |
| `be-compute-ws` | `be-compute` | `be-compute` | Backend private EC2 instances and ephemeral SSH key binding |

## 4. Security Implementation
1. **No Static Credentials:** Authentication to AWS is handled via OIDC dynamic credentials, entirely eliminating long-lived static AWS access keys.
2. **Ephemeral Secrets:** SSH private keys are securely generated, stored in AWS Secrets Manager, and read ephemerally solely during instance provisioning.
3. **Policy as Code (OPA):** Open Policy Agent enforces mandatory rules (e.g., blocking public ingress `0.0.0.0/0` on private security groups) to fail and block non-compliant Terraform plans automatically.

## 5. Step-by-Step Operating Procedures (SOP)

### Step 1: Clone the Repository
Begin by cloning the repository to your local development environment:
```bash
git clone [https://github.com/khinsandarwin88/aws-vpc-peering-vcs-lab.git](https://github.com/khinsandarwin88/aws-vpc-peering-vcs-lab.git)
cd aws-vpc-peering-vcs-lab


Step 2: Create and Push Component Branches
Create dedicated branches for each component to maintain the Branch-per-Workspace workflow:

Bash
git checkout main
git checkout -b fe-network
git push origin fe-network
Step 3: Configure HCP Terraform Workspaces
Log in to the HCP Terraform Dashboard.

Create or select your project and set up 6 individual workspaces matching your components.

For each workspace, navigate to Settings -> Version Control to bind your GitHub repository, VCS branch, and working directory.

Step 4: Enforce OPA Policies
Define your OPA Rego security policies within HCP Terraform under organization-managed policies.

Set the enforcement behavior to Mandatory and attach them to the relevant workspaces.

Step 5: Execute the GitOps Deployment Workflow
Make necessary infrastructure code modifications inside a specific component directory.

Commit and push your code updates to the designated feature branch.

Open a Pull Request targeting the main branch to review automated HCP Terraform speculative plans and OPA check results.
