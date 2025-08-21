# Secure Online Boutique - GitOps Repository

GitOps deployment repository for the [Secure Online Boutique application](https://github.com/iamfet/sec-online-boutique-appliation) using ArgoCD, Kustomize, and automated image tag updates. This repository serves as the single source of truth for Kubernetes deployments and configures cluster resources including ArgoCD applications, monitoring stack, network policies, and OPA Gatekeeper policies.

## ✨ Features

- **GitOps Workflow**: ArgoCD-based continuous deployment
- **Kustomize Management**: Environment-specific configurations
- **Automated Updates**: Image tag updates via GitHub Actions
- **Policy Enforcement**: Open Policy Agent (OPA) Gatekeeper policies
- **Security-First**: Multi-layer security with policy enforcement and secret management
- **Service Mesh**: Istio-based traffic management and mTLS encryption
- **Observability**: Prometheus metrics and Grafana dashboards

## 🛠️ Tools and Technologies

| Category | Tool | Purpose |
|----------|------|----------|
| **GitOps** | ArgoCD | Continuous deployment and sync |
| **Configuration** | Kustomize | Environment-specific manifest management |
| **Policy** | OPA Gatekeeper | Security and compliance enforcement |
| **Automation** | GitHub Actions | Automated image tag updates |
| **Monitoring** | Prometheus/Grafana | Observability stack |
| **Service Mesh** | Istio | Traffic management and security |
| **Security** | DefectDojo | Vulnerability management platform |
| **Secrets** | Vault + External Secrets | Secret management and injection |
| **Certificates** | cert-manager | SSL/TLS certificate automation |

## 📁 Repository Structure

```
├── base/                     # Base Kustomize configurations
│   ├── kustomization.yaml
│   ├── adservice.yaml
│   ├── cartservice.yaml
│   ├── frontend.yaml
│   └── ...                   # All 11 microservice manifests
├── overlays/                 # Environment-specific overlays
│   └── dev/                  # Development environment
├── cluster-resources/        # Cluster-wide infrastructure
│   ├── argocd/               # ArgoCD VirtualService configuration
│   ├── cert-manager/         # SSL certificates and cluster issuer
│   ├── external-secret/      # Vault cluster store configuration
│   ├── grafana/              # Grafana dashboards and VirtualService
│   ├── istio/                # Gateway, authorization policies, peer auth
│   ├── opa-gatekeeper/       # Security constraint templates and policies
│   ├── prometheus/           # ServiceMonitors for all components
│   └── vault/                # Vault initialization job
├── defectdojo-app/           # DefectDojo vulnerability management
├── online-boutique-config/   # Application-specific configuration
└── .github/workflows/        # Automation workflows
```

## 🔄 GitOps Workflow

### Deployment Process
```
Application CI/CD → Image Build → Repository Dispatch → Image Tag Update → ArgoCD Sync → Kubernetes Deploy
```

### ArgoCD Applications

| Application | Path | Sync Policy | Purpose |
|-------------|------|-------------|----------|
| **online-boutique** | `overlays/dev` | Automatic | Main application deployment |
| **defectdojo** | `defectdojo-app/` | Automatic | Vulnerability management platform |
| **cluster-resources** | `cluster-resources/` | Automatic | Infrastructure and policy management |

### Automated Image Updates via GitHub Actions

1. **Trigger**: Repository dispatch from application CI/CD pipeline
2. **Update**: GitHub Actions workflow updates image tags in Kustomize files
3. **Commit**: Automated commit with new image versions
4. **Sync**: ArgoCD detects changes and syncs to cluster

## 🏗️ Kustomize Structure

### Base Configuration
Contains common Kubernetes manifests for all microservices:
- Deployments with security contexts
- Services with appropriate ports
- ConfigMaps for application configuration
- Health check configurations

### Environment Overlays

#### Development (dev)
- **Resources**: Optimized CPU/memory limits (64m CPU, 64Mi memory requests)
- **Replicas**: Single instance per service
- **Images**: Automated tag updates via GitHub Actions
- **Networking**: Istio VirtualService with *.fetdevops.com domain
- **Secrets**: Vault integration via External Secrets Operator
- **Monitoring**: Full observability with Prometheus and Grafana

## 🔒 Security Implementation

This repository implements a comprehensive multi-layer security architecture with policy enforcement, network security, secret management, and vulnerability tracking.

### Policy-as-Code Security (OPA Gatekeeper)

#### Constraint Templates and Policies

| Policy | Template | Purpose | Enforcement |
|--------|----------|---------|-------------|
| **Privileged Containers** | `k8spspprivilegedcontainer` | Blocks containers from running in privileged mode | Enforced |
| **NodePort Services** | `k8sblockednodeport` | Prevents NodePort service creation | Enforced |

#### Security Controls
- **Container Security**: Prevents privilege escalation by blocking `securityContext.privileged: true`
- **Network Exposure**: Forces controlled ingress through Istio Gateway by blocking NodePort services
- **Policy Violations**: Real-time enforcement with detailed violation messages
- **Exemption Support**: Image-based exemptions for system components when needed

### Service Mesh Security (Istio)

#### Zero-Trust Network Architecture

| Component | Configuration | Security Feature |
|-----------|---------------|------------------|
| **Gateway** | `fetdevops-gateway` | Wildcard SSL termination (*.fetdevops.com) |
| **Peer Authentication** | `online-boutique-peer-authentication` | Enforces mTLS for all service communication |
| **Authorization Policies** | Service-specific policies | Granular access control per service |

#### Network Security Controls
- **mTLS Encryption**: Automatic mutual TLS between all microservices
- **Authorization Policies**: 
  - ArgoCD access control via `argocd-authorization-policy`
  - DefectDojo access control via `defectdojo-authorization-policy`
  - Frontend access control via `frontend-authorization-policy`
- **Gateway Security**: Centralized ingress with SSL/TLS termination
- **VirtualServices**: Secure routing for ArgoCD, DefectDojo, Grafana, and Frontend

#### VirtualServices and Exposed Endpoints

| Service | VirtualService | Domain | Purpose |
|---------|----------------|--------|----------|
| **ArgoCD** | `argocd-virtual-service` | `argocd.fetdevops.com` | GitOps deployment management UI |
| **DefectDojo** | `defectdojo-virtual-service` | `defectdojo.fetdevops.com` | Vulnerability management platform |
| **Grafana** | `grafana-virtual-service` | `grafana.fetdevops.com` | Monitoring dashboards and visualization |
| **Frontend** | `frontend-virtual-service` | `online.fetdevops.com` | Online Boutique application frontend |

### Secret Management Security

#### Vault Integration
- **External Secrets Operator**: Secure secret injection from HashiCorp Vault
- **Vault Cluster Store**: Centralized secret management configuration
- **No Hardcoded Secrets**: All sensitive data retrieved from Vault at runtime

#### DefectDojo Security Configuration
```yaml
# Secure credential management
existingSecret: defectdojo
postgresql:
  auth:
    existingSecret: defectdojo
    secretKeys:
      adminPasswordKey: DD_DATABASE_PASSWORD
redis:
  auth:
    existingSecret: defectdojo
    existingSecretPasswordKey: DD_REDIS_PASSWORD
```

#### Application Secrets
- **Stripe Integration**: Vault-managed payment credentials via `stripe-vault-external-secret`
- **Database Passwords**: Vault-injected PostgreSQL credentials
- **Redis Authentication**: Vault-managed Redis passwords
- **Admin Credentials**: Secure DefectDojo admin user management

### Container Security

#### Resource Constraints
- **CPU Limits**: Prevents resource exhaustion (64m-500m CPU allocation)
- **Memory Limits**: Controls memory usage (64Mi-1Gi memory allocation)
- **Security Contexts**: Non-root containers enforced via OPA policies

#### Image Security
- **Specific Tags**: No `latest` tags, all images use specific versions
- **Controlled Repositories**: Images from trusted `iamfet/*` namespace
- **Vulnerability Scanning**: Integration with DefectDojo for image assessment

### Certificate Management

#### Automated SSL/TLS
- **cert-manager**: Automated certificate provisioning and renewal
- **Cluster Issuer**: Let's Encrypt integration for SSL certificates
- **Certificate Resources**: Automatic SSL certificate generation for `*.fetdevops.com`

### Vulnerability Management

#### DefectDojo Integration
- **Centralized Tracking**: All security findings aggregated in DefectDojo
- **Automated Reporting**: CI/CD pipeline integration for scan results
- **Risk Assessment**: Vulnerability prioritization and remediation tracking
- **Secure Deployment**: Resource-optimized configuration for personal projects

#### Security Monitoring
- **Prometheus ServiceMonitors**: Security metrics collection for all components
- **Grafana Dashboards**: Security posture visualization
- **Audit Logging**: All configuration changes tracked via Git history

### GitOps Security

#### Infrastructure as Code Security
- **Immutable Infrastructure**: All changes version-controlled and auditable
- **Audit Trail**: Complete deployment history via Git commits
- **Rollback Capability**: Quick reversion to known-good configurations
- **Automated Updates**: Secure image tag updates via repository dispatch

#### Access Control
- **GitHub Secrets**: Secure credential management for automation workflows
- **Repository Dispatch**: Controlled image update mechanism from CI/CD pipelines
- **Branch Protection**: Peer review requirements for configuration changesitoring
- **Prometheus ServiceMonitors**: Security metrics collection for all components
- **Grafana Dashboards**: Security posture visualization
- **Audit Logging**: All configuration changes tracked via Git history

### GitOps Security

#### Infrastructure as Code Security
- **Immutable Infrastructure**: All changes version-controlled and auditable
- **Audit Trail**: Complete deployment history via Git commits
- **Rollback Capability**: Quick reversion to known-good configurations
- **Automated Updates**: Secure image tag updates via repository dispatch

#### Access Control
- **GitHub Secrets**: Secure credential management for automation workflows
- **Repository Dispatch**: Controlled image update mechanism from CI/CD pipelines
- **Branch Protection**: Peer review requirements for configuration changes

## 🚀 Deployment

### Prerequisites

| Requirement | Specification |
|-------------|---------------|
| **EKS Cluster** | v1.28+ with ArgoCD installed |
| **ArgoCD** | v2.8+ with proper RBAC configuration |
| **Istio** | Service mesh for traffic management |
| **OPA Gatekeeper** | Policy enforcement engine |
| **GitHub Secrets** | Repository access tokens configured |

### ArgoCD Application Setup

```bash
# Apply ArgoCD applications
kubectl apply -f applications/

# Verify application status
argocd app list
argocd app get online-boutique
```

### Manual Deployment

```bash
# Deploy application to development
kustomize build overlays/dev | kubectl apply -f -

# Deploy cluster resources
kustomize build cluster-resources | kubectl apply -f -

# Deploy DefectDojo
kustomize build defectdojo-app | kubectl apply -f -

# Verify deployment
kubectl get pods -n online-boutique
kubectl get pods -n defectdojo
```

## 🔧 Configuration Management

### Image Tag Updates

The repository uses GitHub Actions to automatically update image tags when new versions are built:

```yaml
# Triggered by repository dispatch from application CI/CD
name: Update Image Tags
on:
  repository_dispatch:
    types: [update-image]
```

### Configuration Management

The development environment configuration includes:
- **Resource limits and requests**: Optimized for development workloads
- **Replica counts**: Single instance per service
- **Image tags**: Automatically updated via GitHub Actions
- **Networking**: Istio VirtualServices for secure routing
- **Secrets**: Vault-managed credentials via External Secrets Operator

## 📊 Monitoring and Observability

### Metrics Collection
- **Prometheus**: Metrics scraping and storage
- **Grafana**: Visualization and dashboards
- **ArgoCD Metrics**: Deployment and sync status
- **Application Metrics**: Custom business metrics

### Tracing
- **Jaeger**: Distributed tracing via Istio
- **OpenTelemetry**: Instrumentation and trace collection

## 🛠️ Development Workflow

### Making Changes

1. **Fork Repository**: Create personal fork for changes
2. **Create Branch**: Feature branch for modifications
3. **Test Changes**: Validate Kustomize builds locally
4. **Submit PR**: Pull request with detailed description
5. **Review Process**: Automated validation and peer review
6. **Merge**: Automatic deployment to development environment

### Local Testing

```bash
# Validate Kustomize configuration
kustomize build overlays/dev

# Check for YAML syntax errors
yamllint overlays/

# Validate Kubernetes manifests
kubectl apply --dry-run=client -f <(kustomize build overlays/dev)

# Test security policies
kubectl apply --dry-run=client -f <(kustomize build cluster-resources)
```

## 🔄 Automation

### GitHub Actions Workflows

| Workflow | Trigger | Purpose |
|----------|---------|----------|
| **Update Product Catalog Service Tag** | Repository dispatch (`update-productcatalog-tag`) | Automated productcatalogservice image tag updates |

### Required Secrets

- **`ACTIONS_PA_TOKEN`**: GitHub Personal Access Token for repository operations
- **`USER_EMAIL`**: Git user email for automated commits
- **`USER_NAME`**: Git username for automated commits

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make changes following GitOps best practices
4. Test configurations locally
5. Submit a pull request with detailed description

### Best Practices

- **Atomic Changes**: Single logical change per commit
- **Descriptive Commits**: Clear commit messages explaining changes
- **Environment Parity**: Maintain consistency across environments
- **Security First**: Always consider security implications
- **Documentation**: Update documentation for configuration changes

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## 🔗 Related Projects

- [Application Repository](https://github.com/iamfet/sec-online-boutique-appliation) - Microservices source code
- [Infrastructure Repository](https://github.com/iamfet/sec-eks-infra-automation/) - EKS infrastructure automation

---

**Built with ❤️ for GitOps and DevSecOps demonstrations**