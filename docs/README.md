# 📚 CropWise Documentation

**Welcome to the CropWise documentation hub!**

This directory contains all documentation organized by purpose and workflow.

---

## 🚀 Quick Start

**👉 New to CropWise?** Start here: **[0-START-HERE.md](../0-START-HERE.md)**

This master guide provides a clear, sequential path from zero to production deployment.

---

## 📁 Documentation Structure

### **1. Setup Guides** (`setup/`)

**Sequential guides for initial project setup** (follow in order):

| Step | Guide | Purpose | Time |
|------|-------|---------|------|
| 1 | [GitHub Setup](setup/01-github-setup.md) | Repository, branches, secrets | 30 min |
| 2 | [Local Development](setup/02-local-development.md) | Dev environment setup | 45 min |
| 3 | [AWS Account Setup](setup/03-aws-account-setup.md) | AWS account configuration | 20 min |
| 4 | [AWS Infrastructure (Dev)](setup/04-aws-infrastructure-dev.md) | Development AWS resources | 60 min |
| 5 | [AWS Database Setup](setup/05-aws-database-setup.md) | RDS PostgreSQL for all envs | 30 min |
| 6 | [Google OAuth Setup](setup/06-google-oauth-setup.md) | Authentication integration | 20 min |
| 7 | [Communication Setup](setup/07-communication-setup.md) | Twilio, Slack, WhatsApp | 30 min |
| 8 | [Staging Deployment](setup/08-staging-deployment.md) | Staging environment | 45 min |
| 9 | [Production Deployment](setup/09-production-deployment.md) | Production environment | 60 min |
| 10 | [Monitoring & Security](setup/10-monitoring-security.md) | Monitoring and hardening | 45 min |

**Total Time**: 4-6 hours for complete setup

---

### **2. Development** (`development/`)

**Daily development guides and workflows**:

- **[Developer Guide](development/DEVELOPER_GUIDE.md)** - Complete development workflow
- **[Git Workflow](development/GIT_WORKFLOW.md)** - Branching, PRs, merging
- **[Testing Guide](development/TESTING_GUIDE.md)** - Unit, integration, E2E tests
- **[Contributing](development/CONTRIBUTING.md)** - How to contribute code
- **[Team Onboarding](development/TEAM_ONBOARDING.md)** - New developer onboarding

---

### **3. Deployment** (`deployment/`)

**Deployment and CI/CD guides**:

- **[Deployment Guide](deployment/DEPLOYMENT_GUIDE.md)** - Complete deployment workflow
- **[CI/CD Setup Guide](deployment/CICD_SETUP_GUIDE.md)** - GitHub Actions setup
- **[Release Process](deployment/RELEASE_PROCESS.md)** - Release management
- **[Troubleshooting](deployment/TROUBLESHOOTING.md)** - Common issues and fixes

---

### **4. Operations** (`operations/`)

**Operations and maintenance**:

- **[Admin Guide](operations/ADMIN_GUIDE.md)** - System administration
- **[Security Guide](operations/SECURITY_GUIDE.md)** - Security best practices
- **[Secrets Management](operations/SECRETS_MANAGEMENT_GUIDE.md)** - Managing secrets

---

### **5. Features** (`features/`)

**Feature-specific documentation**:

#### **IoT Features** (`features/iot/`)
- [IoT Integration Guide](features/iot/IOT_INTEGRATION_GUIDE.md)
- [ESP32 Zone Controller Guide](features/iot/ESP32_ZONE_CONTROLLER_GUIDE.md)
- [ESP-NOW Setup Guide](features/iot/ESP_NOW_SETUP_GUIDE.md)
- [IoT Architecture V2](features/iot/IOT_ARCHITECTURE_V2.md)

#### **Management Features** (`features/management/`)
- [Task Management Guide](features/management/TASK_MANAGEMENT_GUIDE.md)
- [Inventory Management Guide](features/management/INVENTORY_MANAGEMENT_GUIDE.md)
- [Labor Cost Tracking Guide](features/management/LABOR_COST_TRACKING_GUIDE.md)
- [Quality Control Guide](features/management/QUALITY_CONTROL_GUIDE.md)

#### **Integration Features** (`features/integrations/`)
- [WhatsApp & SMS Setup](features/integrations/WHATSAPP_SMS_SETUP.md)
- [Google OAuth Checklist](features/integrations/GOOGLE_OAUTH_CHECKLIST.md)

---

### **6. Reference** (`reference/`)

**Quick references and lookup guides**:

- **[Quick Reference](reference/QUICK_REFERENCE.md)** - Command cheat sheet
- **[API Reference](reference/API_REFERENCE.md)** - API documentation
- **[FAQ](reference/FAQ.md)** - Frequently asked questions
- **[User Guide](reference/USER_GUIDE.md)** - End-user documentation

---

### **7. Architecture** (`architecture/`)

**System architecture documentation**:

- **[Architecture Overview](architecture/ARCHITECTURE_OVERVIEW.md)** - System design
- **[Database Schema](architecture/DATABASE_SCHEMA.md)** - Database structure
- **[Edge Gateway](architecture/EDGE_GATEWAY.md)** - Edge computing architecture

---

### **8. Archive** (`archive/`)

Historical and deprecated documentation (for reference only).

---

## 🎯 Common Scenarios

### **"I'm setting up the project from scratch"**
→ Start with **[0-START-HERE.md](../0-START-HERE.md)** and follow steps 1-10

### **"I'm a new developer joining the team"**
→ Read **[Team Onboarding](development/TEAM_ONBOARDING.md)**

### **"I need to deploy a hotfix"**
→ Follow **[Release Process](deployment/RELEASE_PROCESS.md)** (hotfix section)

### **"The application is down"**
→ Check **[Troubleshooting](deployment/TROUBLESHOOTING.md)**

### **"I need to add IoT device support"**
→ Read **[IoT Integration Guide](features/iot/IOT_INTEGRATION_GUIDE.md)**

### **"I need a quick command reference"**
→ Check **[Quick Reference](reference/QUICK_REFERENCE.md)**

### **"I want to understand the system architecture"**
→ Read **[Architecture Overview](architecture/ARCHITECTURE_OVERVIEW.md)**

---

## 📊 Documentation Status

| Category | Guides | Status |
|----------|--------|--------|
| Setup | 10 | ✅ Complete |
| Development | 5 | ✅ Complete |
| Deployment | 4 | ✅ Complete |
| Operations | 3 | ✅ Complete |
| Features | 11 | ✅ Complete |
| Reference | 4 | ✅ Complete |
| Architecture | 3 | ✅ Complete |

**Last Updated**: November 16, 2025

---

## 🤝 Contributing to Documentation

Found an error or want to improve the docs?

1. Create an issue with label `documentation`
2. Make changes in a feature branch
3. Submit PR with clear description
4. Tag reviewers

See **[Contributing Guide](development/CONTRIBUTING.md)** for details.

---

## 📝 Documentation Standards

- Use Markdown for all docs
- Include table of contents for long docs
- Add code examples where applicable
- Keep guides focused and actionable
- Update last modified date
- Cross-reference related docs

---

## 🆘 Need Help?

If you can't find what you're looking for:

1. Check the **[FAQ](reference/FAQ.md)**
2. Search the docs (Ctrl/Cmd + F)
3. Ask in team Slack channel
4. Create a GitHub issue

---

**Happy farming! 🌱**
