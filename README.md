# Project-7-GitOps: Kubernetes Deployment Manifests

> 📋 **For complete project overview and context, see [project-7-overview](https://github.com/adamlevi87/project-7-overview)**

This repository contains the GitOps deployment configurations and ArgoCD application manifests for Project-7, implementing a complete continuous deployment pipeline using the GitOps methodology.

## Purpose & Architecture

This GitOps repository serves as the **single source of truth** for all Kubernetes deployment configurations, implementing a complete separation of concerns between application code and deployment manifests. ArgoCD continuously monitors this repository and automatically syncs any changes to the target Kubernetes clusters.

### Repository Structure

```
├── apps/                        # ArgoCD Application manifests
│   └── frontend/
│       └── application.yaml     # Frontend application definition
├── manifests/                   # Kubernetes deployment manifests  
│   └── frontend/
│       ├── app-values.yaml      # Static application configuration
│       ├── infra-values.yaml    # Infrastructure values (managed by Terraform)
│       └── digest-values.yaml   # Container image digests (managed by CI/CD)
└── reference_only/              # Reference templates and documentation
    ├── project/                 # ArgoCD project templates
    └── app_of_apps.yaml         # App-of-apps pattern template
```

## GitOps Workflow

### **Multi-Source ArgoCD Configuration**
Applications use ArgoCD's multi-source capability to separate concerns:
- **Application Repository** ([project-7-app](https://github.com/adamlevi87/project-7-app)) - Contains Helm charts and application code
- **GitOps Repository** (this repo) - Contains environment-specific values and deployment configuration

### **Automated Value Management**
Different types of values are managed through different automation:

#### **Infrastructure Values** (`infra-values.yaml`)
- **Managed by**: [project-7-tf](https://github.com/adamlevi87/project-7-tf) Terraform workflows
- **Contains**: ECR URLs, IAM roles, ingress configuration, security groups
- **Update trigger**: Infrastructure changes via Terraform deployment

#### **Container Digests** (`digest-values.yaml`)  
- **Managed by**: [project-7-app](https://github.com/adamlevi87/project-7-app) CI/CD workflows
- **Contains**: SHA256 digests for container images
- **Update trigger**: Application deployments via CI/CD pipeline

#### **Application Configuration** (`app-values.yaml`)
- **Managed by**: Manual Git commits  
- **Contains**: Static application settings, feature flags, environment-specific config
- **Update trigger**: Manual changes through Git workflow

## Repository Initialization

This repository is **automatically bootstrapped** by the Terraform infrastructure when `bootstrap_mode=true` is used in the [project-7-tf](https://github.com/adamlevi87/project-7-tf) deployment.

### Bootstrap Process
1. **Terraform detects** empty or missing GitOps repository
2. **Generates all required manifests** using infrastructure data
3. **Creates initial PR** with complete ArgoCD configuration
4. **Auto-merges PR** (for dev/staging) or awaits manual review (for production)

### Automated Setup Includes:
- **ArgoCD Project** definition for proper RBAC and source repository access
- **Application manifests** with multi-source configuration
- **Infrastructure values** with current ECR repositories, IAM roles, and networking
- **Template files** for reference and documentation

## Branch Strategy

### **Environment Branches**
- **`dev`** - Development environment deployments
- **`staging`** - Staging environment deployments  
- **`main`** - Production environment deployments

### **Automated PR Management**
- **Infrastructure updates** create PRs targeting appropriate environment branch
- **Container deployments** create PRs with new image digests
- **Auto-merge** for dev/staging environments
- **Manual approval** required for production deployments

## ArgoCD Integration

### **App-of-Apps Pattern**
The repository implements the App-of-Apps pattern where a root application manages all other applications, providing:
- **Centralized management** of multiple applications
- **Dependency ordering** through sync waves
- **Consistent configuration** across all applications

### **Automated Sync Policies**
Applications are configured with:
- **Auto-sync enabled** for continuous deployment
- **Self-healing** to automatically fix configuration drift  
- **Auto-pruning** to remove resources no longer defined in Git
- **Retry policies** with exponential backoff for failed deployments

### **Multi-Source Benefits**
- **Separation of concerns** between application code and deployment configuration
- **Independent versioning** of Helm charts and environment-specific values
- **Shared value files** across multiple applications
- **Simplified rollbacks** by reverting specific value changes

## Operational Workflows

### **Standard Deployment Flow**
```
Application Change → CI/CD Build → ECR Push → GitOps PR → ArgoCD Sync → Kubernetes Deployment
```

### **Infrastructure Update Flow**  
```
Terraform Apply → Infrastructure Values Update → GitOps PR → ArgoCD Sync → Configuration Update
```

### **Rollback Process**
```
Identify Previous Digest → Manual GitOps PR → Merge → ArgoCD Sync → Previous Version Deployed
```

## Security & Compliance

### **GitOps Security Model**
- **Read-only cluster access** for ArgoCD
- **Git-based audit trail** for all deployment changes
- **RBAC integration** with GitHub teams and organizations
- **Signed commits** and PR-based change approval

### **Image Security**
- **Digest-based deployments** prevent tag-based attacks
- **Cosign signature verification** for supply chain security
- **SBOM attestation** for vulnerability tracking
- **Immutable image references** through SHA256 digests

---

*This GitOps repository implements enterprise-grade continuous deployment practices, providing automated delivery with proper security controls, audit trails, and operational safeguards for production Kubernetes environments.*
