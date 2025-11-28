# 📚 Ebook Landing Page - CI/CD Lab

[![PR Validation](https://github.com/Tagomata/ebook-app-frontend/actions/workflows/pr-validation.yml/badge.svg)](https://github.com/Tagomata/ebook-app-frontend/actions/workflows/pr-validation.yml)
[![Deploy to S3](https://github.com/Tagomata/ebook-app-frontend/actions/workflows/deploy-to-s3.yml/badge.svg)](https://github.com/Tagomata/ebook-app-frontend/actions/workflows/deploy-to-s3.yml)

A **DevOps laboratory project** demonstrating enterprise-grade CI/CD pipeline implementation, infrastructure as code, and security best practices for static website deployment.

> **📌 Portfolio Project**: This repository showcases professional software engineering practices including automated testing, security scanning, OIDC authentication, and zero-downtime deployments to AWS.

---

## 🎯 Project Context

This is a **learning laboratory** focused on implementing production-ready DevOps practices. The static website template serves as the deployment artifact to demonstrate:

- ✅ Automated CI/CD pipelines
- ✅ Infrastructure as Code with Terraform
- ✅ Security-first DevSecOps approach
- ✅ Cloud-native architecture on AWS

### 📦 Project Components

| Component | Repository | Purpose |
|-----------|-----------|---------|
| **Frontend** (this repo) | [ebook-app-frontend](https://github.com/Tagomata/ebook-app-frontend) | Static website + CI/CD pipelines |
| **Infrastructure** | [ebook-app](https://github.com/Tagomata/ebook-app) | Terraform IaC for AWS resources |
| **Original Template** | [Ebook by pravinmishraaws](https://github.com/pravinmishraaws/Ebook.git) | Static website template (base) |

> **Note**: The static website template was adapted from [pravinmishraaws/Ebook](https://github.com/pravinmishraaws/Ebook.git) and used as an example for this CI/CD deployment laboratory.

---

## 🌟 What This Project Demonstrates

### CI/CD & Automation
- ✅ **GitHub Actions Workflows** — Parallel job execution for fast feedback
- ✅ **Automated Validation** — HTML, security, links, and linting on every PR
- ✅ **Automated Deployment** — Zero-touch deployment to AWS S3 + CloudFront
- ✅ **Branch Protection** — Enforced code review and status checks

### Security (DevSecOps)
- ✅ **OIDC Authentication** — No long-lived AWS credentials stored
- ✅ **Secrets Scanning** — Gitleaks detects accidentally committed credentials
- ✅ **Vulnerability Scanning** — Trivy scans for known CVEs
- ✅ **SARIF Integration** — Security results in GitHub Security tab

### Infrastructure & Cloud
- ✅ **Infrastructure as Code** — Terraform manages all AWS resources
- ✅ **S3 Static Hosting** — Cost-effective, scalable hosting
- ✅ **CloudFront CDN** — Global content delivery with edge caching
- ✅ **Optimized Performance** — Cache headers by file type

---

## 🏗️ Architecture Overview

### CI/CD Pipeline Flow

```
┌─────────────────────────────────────────────┐
│         DEVELOPER: Opens Pull Request       │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│    Workflow: PR Validation (4 jobs/parallel)│
├─────────────────────────────────────────────┤
│  ✅ validate-html      → W3C standards       │
│  ✅ security-scan      → Gitleaks + Trivy    │
│  ✅ check-links        → Broken link detect  │
│  ✅ lint-code          → ESLint validation   │
└─────────────────────────────────────────────┘
                    ↓
            All checks pass? ✅
                    ↓
┌─────────────────────────────────────────────┐
│       MERGE TO MAIN (protected branch)      │
└─────────────────────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────┐
│      Workflow: Deploy to S3 + CloudFront    │
├─────────────────────────────────────────────┤
│  🔐 Step 1: AWS OIDC Authentication         │
│  📦 Step 2: S3 Sync (incremental)           │
│  ⚡ Step 3: Cache Headers Optimization      │
│  🌐 Step 4: CloudFront Invalidation         │
└─────────────────────────────────────────────┘
                    ↓
            🌐 LIVE IN PRODUCTION
```

### Infrastructure Architecture

```
┌──────────────────────────────────────────────────┐
│             User (Global Access)                 │
└──────────────────┬───────────────────────────────┘
                   │ HTTPS
                   ↓
┌──────────────────────────────────────────────────┐
│          CloudFront CDN (Edge Locations)         │
│        Distribution: E3RFO2ZKJEH7S               │
│    • Cache optimization per file type            │
│    • HTTPS only, Gzip compression                │
└──────────────────┬───────────────────────────────┘
                   │
                   ↓
┌──────────────────────────────────────────────────┐
│         S3 Static Website Hosting                │
│       Bucket: ebook-app-prod-website             │
│    • Versioning enabled                          │
│    • Public read access via CloudFront           │
└──────────────────────────────────────────────────┘
                   ↑
                   │ Sync files
┌──────────────────────────────────────────────────┐
│          GitHub Actions (CI/CD)                  │
│    ↔ AWS IAM Role (OIDC Trust)                   │
│      github-actions-ebook-deploy-role            │
│    • Temporary credentials (1-hour)              │
│    • Least privilege permissions                 │
└──────────────────────────────────────────────────┘
                   ↑
                   │ Infrastructure
┌──────────────────────────────────────────────────┐
│          Terraform (IaC Repository)              │
│    github.com/Tagomata/ebook-app                 │
│    • S3 bucket configuration                     │
│    • CloudFront distribution                     │
│    • IAM roles and policies                      │
│    • OIDC provider setup                         │
└──────────────────────────────────────────────────┘
```

---

## 🔒 Security Implementation

### Zero-Trust Authentication (OIDC)

This project uses **OpenID Connect (OIDC)** instead of static AWS credentials:

```yaml
# ✅ SECURE: No credentials stored
- uses: aws-actions/configure-aws-credentials@v4
  with:
      role-to-assume: ${{ vars.AWS_ROLE_ARN }}
      aws-region: us-east-2

# ❌ INSECURE: Never use this approach
- env:
    AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
    AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

**Benefits:**
- 🔐 No long-lived credentials in GitHub
- ⏱️ Temporary tokens (1-hour validity)
- 🎯 Granular trust policy (repo + branch specific)
- 📊 Full audit trail in AWS CloudTrail

### Trust Policy Configuration

```json
{
  "Condition": {
    "StringEquals": {
      "token.actions.githubusercontent.com:sub": "repo:Tagomata/ebook-app-frontend:ref:refs/heads/main"
    }
  }
}
```

**This means:**
- ✅ Only this repository can assume the role
- ✅ Only the `main` branch can deploy
- ❌ Forks cannot assume the role
- ❌ Pull requests cannot deploy

### Security Scanning

| Tool | Purpose | Runs On |
|------|---------|---------|
| **Gitleaks** | Detects secrets (API keys, passwords) | Every PR |
| **Trivy** | Scans for vulnerabilities (CVEs) | Every PR |
| **html5validator** | W3C standards compliance | Every PR |
| **Lychee** | Broken link detection | Every PR |

---

## ⚡ Performance Optimizations

### Cache Header Strategy

Optimized cache headers by file type for maximum performance:

```yaml
HTML files:   1 hour   → max-age=3600, must-revalidate
CSS/JS files: 7 days   → max-age=604800, immutable
Images:       30 days  → max-age=2592000, immutable
Fonts:        1 year   → max-age=31536000, immutable
```

**Why this matters:**
- **HTML** changes frequently → short cache, always revalidate
- **CSS/JS** with `immutable` → browser never revalidates (faster)
- **Images/Fonts** rarely change → long cache reduces bandwidth

### CDN Configuration
- **Edge Locations**: Global distribution via CloudFront
- **Automatic Invalidation**: Cache cleared on every deployment
- **Compression**: Gzip enabled for text assets
- **HTTPS Only**: Secure content delivery

---

## 🛠️ Technology Stack

### Frontend Technologies
![HTML5](https://img.shields.io/badge/HTML5-E34F26?style=flat&logo=html5&logoColor=white)
![CSS3](https://img.shields.io/badge/CSS3-1572B6?style=flat&logo=css3&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat&logo=javascript&logoColor=black)
![Bootstrap](https://img.shields.io/badge/Bootstrap-7952B3?style=flat&logo=bootstrap&logoColor=white)
![jQuery](https://img.shields.io/badge/jQuery-0769AD?style=flat&logo=jquery&logoColor=white)

### DevOps & Cloud
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat&logo=github-actions&logoColor=white)
![AWS](https://img.shields.io/badge/AWS-232F3E?style=flat&logo=amazon-aws)
![Terraform](https://img.shields.io/badge/Terraform-7B42BC?style=flat&logo=terraform&logoColor=white)
![S3](https://img.shields.io/badge/Amazon_S3-569A31?style=flat&logo=amazon-s3&logoColor=white)
![CloudFront](https://img.shields.io/badge/CloudFront-232F3E?style=flat&logo=amazon-aws)

### Security & Quality Tools
![Gitleaks](https://img.shields.io/badge/Gitleaks-000000?style=flat&logo=git&logoColor=white)
![Trivy](https://img.shields.io/badge/Trivy-1904DA?style=flat&logo=aqua&logoColor=white)
![ESLint](https://img.shields.io/badge/ESLint-4B32C3?style=flat&logo=eslint&logoColor=white)

---

## 🚀 Getting Started

### Prerequisites
- Git
- A modern web browser
- (Optional) Local web server for development

### Local Development

```bash
# Clone the repository
git clone https://github.com/Tagomata/ebook-app-frontend.git
cd ebook-app-frontend

# Option 1: Open directly in browser
open index.html

# Option 2: Use Python simple server (recommended)
python -m http.server 8000
# Visit: http://localhost:8000

# Option 3: Use Node.js http-server
npx http-server -p 8000
```

### Development Workflow

1. **Create a feature branch**
   ```bash
   git checkout -b feature/my-feature
   ```

2. **Make your changes**
   - Edit HTML/CSS/JS files
   - Test locally in browser

3. **Commit and push**
   ```bash
   git add .
   git commit -m "feat: add new feature"
   git push origin feature/my-feature
   ```

4. **Open a Pull Request**
   - Go to GitHub and create PR
   - Wait for 4 validation checks to pass:
     - ✅ HTML validation
     - ✅ Security scanning
     - ✅ Link checking
     - ✅ JavaScript linting

5. **Merge to main** (requires approval)
   - Automated deployment triggers
   - Changes live in ~60 seconds

---

## 🔧 Infrastructure Setup

This project requires AWS infrastructure managed in a **separate repository**.

### Infrastructure Repository

📦 **[Tagomata/ebook-app](https://github.com/Tagomata/ebook-app)**

Contains Terraform code for:
- S3 bucket configuration
- CloudFront distribution
- IAM roles and policies
- OIDC provider setup

### Required AWS Resources

| Resource | Name/ID | Purpose |
|----------|---------|---------|
| S3 Bucket | `ebook-app-prod-website` | Static website hosting |
| CloudFront | `E3RFO2ZKJEH7S` | CDN for global delivery |
| IAM Role | `github-actions-ebook-deploy-role` | OIDC authentication |
| OIDC Provider | `token.actions.githubusercontent.com` | GitHub Actions trust |

### GitHub Variables Configuration

The following variables are configured in GitHub:

```
Settings → Secrets and variables → Actions → Variables
```

| Variable | Value | Purpose |
|----------|-------|---------|
| `AWS_REGION` | `us-east-2` | AWS region for deployment |
| `AWS_ROLE_ARN` | `arn:aws:iam::579897422919:role/...` | IAM role for OIDC |
| `S3_BUCKET_NAME` | `ebook-app-prod-website` | Target S3 bucket |
| `CLOUDFRONT_DISTRIBUTION_ID` | `E3RFO2ZKJEH7S` | CloudFront for invalidation |

### Syncing Infrastructure Changes

If infrastructure is recreated or modified:

1. **Update Terraform** in [ebook-app](https://github.com/Tagomata/ebook-app) repo
   ```bash
   cd ebook-app
   terraform apply
   ```

2. **Update GitHub Variables** in this repo
   - Copy new values from Terraform outputs
   - Update variables in GitHub UI

3. **Test deployment**
   - Trigger workflow manually or push to main

---

## 📊 CI/CD Pipeline Details

### Workflow 1: PR Validation

**File**: `.github/workflows/pr-validation.yml`
**Trigger**: Every pull request to `main`
**Duration**: ~40 seconds

**Jobs** (run in parallel):

```yaml
validate-html:
  - Validates HTML against W3C standards
  - Ignores third-party libraries (Bootstrap)
  - Fails on syntax errors

security-scan:
  - Gitleaks: Scans for secrets in code and git history
  - Trivy: Scans for vulnerabilities
  - Uploads SARIF results to GitHub Security tab

check-links:
  - Lychee: Checks all links in HTML
  - Detects 404s, timeouts, broken anchors
  - Auto-creates GitHub issue if links are broken

lint-code:
  - ESLint: Validates JavaScript code quality
  - Only lints custom code (not jQuery/Bootstrap)
  - Enforces consistent coding standards
```

---

### Workflow 2: Deploy to S3

**File**: `.github/workflows/deploy-to-s3.yml`
**Trigger**: Push to `main` or manual
**Duration**: ~60 seconds

**Permissions**:
```yaml
permissions:
  id-token: write  # Required for OIDC
  contents: read   # Read repository code
```

**Steps**:

1. **Checkout code** (`actions/checkout@v4`)
2. **Configure AWS credentials** via OIDC
   - No secrets needed
   - Temporary credentials (1 hour)
3. **Sync files to S3**
   - Excludes: `.git/`, `.github/`, `*.md`
   - Only uploads changed files
   - Deletes removed files
4. **Set cache headers**
   - HTML: 1 hour
   - CSS/JS: 7 days
   - Images: 30 days
   - Fonts: 1 year
5. **Invalidate CloudFront cache**
   - Path: `/*` (all files)
   - Ensures users see latest version
6. **Generate deployment summary**
   - Shows commit SHA, user, bucket, region


---

## 🎯 Best Practices Demonstrated

### DevOps
✅ Infrastructure as Code (Terraform)
✅ GitOps workflow
✅ Automated deployments
✅ Parallel CI/CD execution
✅ Status badges for visibility

### Security
✅ No hardcoded credentials
✅ OIDC authentication (zero-trust)
✅ Automated vulnerability scanning
✅ Branch protection rules
✅ SARIF security reporting
✅ Least privilege IAM policies

### Code Quality
✅ W3C HTML5 validation
✅ JavaScript linting (ESLint)
✅ Broken link detection
✅ Consistent formatting

### Performance
✅ Optimized cache headers
✅ CloudFront CDN
✅ Minified assets
✅ Gzip compression
✅ Incremental deployments

---

## 🤝 Contributing

This is a learning/portfolio project, but contributions are welcome!

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Ensure all validation checks pass
5. Submit a pull request

**Note**: All PRs require:
- ✅ HTML validation passing
- ✅ Security scans passing
- ✅ No broken links
- ✅ JavaScript linting passing

---


## 📝 Credits & Attribution

### Original Template
- **Source**: [pravinmishraaws/Ebook](https://github.com/pravinmishraaws/Ebook.git)
- **Template by**: TemplateMo
- **Images**: FreePik

### CI/CD Implementation
- **Author**: Santiago ([@Tagomata](https://github.com/Tagomata))
- **Purpose**: DevOps learning laboratory
- **Focus**: Enterprise-grade CI/CD practices

### Infrastructure
- **Repository**: [Tagomata/ebook-app](https://github.com/Tagomata/ebook-app)
- **Technology**: Terraform (Infrastructure as Code)

---

## 📚 Learning Outcomes

This project demonstrates understanding of:

- ✅ **GitHub Actions**: Workflows, jobs, steps, matrix strategies
- ✅ **AWS Services**: S3, CloudFront, IAM, STS
- ✅ **Security**: OIDC, secrets scanning, vulnerability management
- ✅ **DevOps**: CI/CD pipelines, GitOps, automation
- ✅ **IaC**: Terraform for cloud infrastructure
- ✅ **Performance**: CDN, caching strategies, optimization

---

## 🔗 Related Links

- 🏗️ **Infrastructure Repo**: [github.com/Tagomata/ebook-app](https://github.com/Tagomata/ebook-app)
- 📄 **Original Template**: [github.com/pravinmishraaws/Ebook](https://github.com/pravinmishraaws/Ebook.git)
- 🌐 **Live Site**: [Add your CloudFront URL here]

---

## 📞 Contact

**Santiago** - [@Tagomata](https://github.com/Tagomata)

Questions or feedback? Feel free to:
- Open an issue
- Submit a pull request
- Star the repo ⭐

---

<div align="center">

**🚀 Built with GitHub Actions, AWS, and Terraform**

*A DevOps laboratory demonstrating enterprise CI/CD practices*

⭐ **Star this repo if you find it useful for learning!**

</div>
