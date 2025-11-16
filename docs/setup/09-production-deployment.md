# 🚀 Step 9: Production Deployment

Complete guide to deploy CropWise to AWS Production environment.

**Prerequisites**: [Step 8 (Staging Deployment)](08-staging-deployment.md) completed and tested

**Time Required**: 1 hour

**⚠️ Important**: Production deployment requires careful testing and approval process.

---

## 📋 What You'll Accomplish

By the end of this guide, you'll have:
- ✅ Production AWS infrastructure created
- ✅ Production database configured with backups
- ✅ Custom domain with SSL/HTTPS
- ✅ GitHub Actions deploying to production (with approval)
- ✅ Production monitoring and alerts
- ✅ Production-ready security configuration

---

## 🛠️ Step-by-Step Instructions

### 1. Create Production AWS Resources

Follow the same process as staging, but with production-grade configuration:

#### **1.1 ECR Repository** (Already exists)

Use `cropwise-backend:prod` tag for production images.

#### **1.2 ECS Cluster**

- **Name**: `cropwise-prod-cluster`
- **Capacity provider**: AWS Fargate

#### **1.3 Task Definition**

- **Family**: `cropwise-backend-prod`
- **CPU**: 1024 (1 vCPU)
- **Memory**: 2048 MB (2 GB)
- **Container**:
  - Image: `123456789012.dkr.ecr.ap-south-1.amazonaws.com/cropwise-backend:prod`
  - Port: 3000
  - Environment: `production`

**Key Production Settings**:
- ✅ Enable CloudWatch logging
- ✅ Enable X-Ray tracing
- ✅ Enable health checks
- ✅ Set resource limits

#### **1.4 Application Load Balancer**

- **Name**: `cropwise-prod-alb`
- **Scheme**: Internet-facing
- **Network**: All 3 availability zones
  - ✅ ap-south-1a
  - ✅ ap-south-1b
  - ✅ ap-south-1c
- **Security Group**: `cropwise-prod-alb-sg`
  - HTTPS (443): 0.0.0.0/0
  - HTTP (80): 0.0.0.0/0 (redirect to HTTPS)

#### **1.5 Target Group**

- **Name**: `cropwise-prod-tg`
- **Target type**: IP
- **Protocol**: HTTP
- **Port**: 3000
- **Health check**:
  - Path: `/health`
  - Interval: 30 seconds
  - Timeout: 5 seconds
  - Healthy threshold: 2
  - Unhealthy threshold: 3

#### **1.6 ECS Service**

- **Service name**: `cropwise-backend-prod-service`
- **Desired tasks**: 2 (for high availability)
- **Min tasks**: 2
- **Max tasks**: 10
- **Load balancer**: `cropwise-prod-alb`
- **Auto-scaling**: Enable

**Auto-Scaling Policy**:
```json
{
  "TargetValue": 75.0,
  "PredefinedMetricType": "ECSServiceAverageCPUUtilization",
  "ScaleInCooldown": 300,
  "ScaleOutCooldown": 60
}
```

#### **1.7 S3 + CloudFront**

- **S3 Bucket**: `cropwise-prod-frontend`
- **Versioning**: ✅ Enabled (for rollback)
- **Logging**: ✅ Enabled
- **CloudFront Distribution**:
  - **Price Class**: Use all edge locations (best performance)
  - **SSL Certificate**: Custom ACM certificate
  - **Caching**: Optimized for SPA
  - **WAF**: Enable AWS WAF

---

### 2. Configure Custom Domain

#### **2.1 Request SSL Certificate**

1. Go to **AWS Certificate Manager** (ACM)
2. Ensure region: **US East (N. Virginia) - us-east-1**
   - (CloudFront requires certificates in us-east-1)
3. Click **Request a certificate**
4. **Domain names**:
   - `cropwise.com`
   - `*.cropwise.com` (wildcard)
   - `www.cropwise.com`

5. **Validation method**: DNS validation
6. Click **Request**
7. Add CNAME records to your DNS (follow AWS instructions)
8. Wait for validation (5-30 minutes)

#### **2.2 Configure CloudFront**

1. Edit your production CloudFront distribution
2. **Alternate domain names (CNAMEs)**:
   - `cropwise.com`
   - `www.cropwise.com`
3. **SSL Certificate**: Select your ACM certificate
4. **Supported HTTP versions**: HTTP/2, HTTP/3
5. **Security policy**: TLSv1.2_2021

6. Save changes

#### **2.3 Configure Route 53**

1. Go to Route 53 → **Hosted zones**
2. Select `cropwise.com`
3. Create records:

**Frontend (A Record - Alias)**:
- **Record name**: `cropwise.com`
- **Record type**: A
- **Alias**: Yes
- **Route traffic to**: CloudFront distribution
- **Distribution**: Select production CloudFront

**WWW Redirect (A Record - Alias)**:
- **Record name**: `www.cropwise.com`
- **Record type**: A
- **Alias**: Yes
- **Route traffic to**: CloudFront distribution

**Backend API (A Record - Alias)**:
- **Record name**: `api.cropwise.com`
- **Record type**: A
- **Alias**: Yes
- **Route traffic to**: Application Load Balancer
- **ALB**: Select `cropwise-prod-alb`

**Verify DNS propagation**:
```bash
nslookup cropwise.com
nslookup api.cropwise.com
```

---

### 3. Configure Production Database

#### **3.1 Verify RDS Configuration**

From [Step 5](05-aws-database-setup.md), ensure production database has:

- ✅ Multi-AZ deployment
- ✅ Automated backups (7-35 days retention)
- ✅ Backup window: 3:00-4:00 AM (low traffic time)
- ✅ Maintenance window: Sunday 4:00-5:00 AM
- ✅ Delete protection: Enabled
- ✅ Encryption at rest: Enabled
- ✅ Performance Insights: Enabled
- ✅ Enhanced monitoring: Enabled

#### **3.2 Create Read Replica** (Optional but recommended)

```bash
aws rds create-db-instance-read-replica \
  --db-instance-identifier cropwise-prod-db-read-replica \
  --source-db-instance-identifier cropwise-prod-db \
  --db-instance-class db.t3.small \
  --availability-zone ap-south-1b \
  --publicly-accessible false
```

#### **3.3 Run Production Migrations**

```bash
# Set production database URL (from secure location)
export DATABASE_URL=postgresql://cropwise_admin:STRONG_PASSWORD@cropwise-prod-db.xxxxx.ap-south-1.rds.amazonaws.com:5432/cropwise_prod

# Run migrations
cd backend
npm run migrate

# DO NOT seed production with test data!
# npm run seed  # ❌ NEVER run this on production
```

---

### 4. Configure GitHub Actions for Production

#### **4.1 Update GitHub Secrets**

Add production-specific secrets:

```
BACKEND_URL_PROD=https://api.cropwise.com
FRONTEND_URL_PROD=https://cropwise.com
S3_BUCKET_PROD=cropwise-prod-frontend
CLOUDFRONT_DISTRIBUTION_ID_PROD=EXXXXXXXXX
ECS_CLUSTER_PROD=cropwise-prod-cluster
ECS_SERVICE_PROD=cropwise-backend-prod-service
ECS_TASK_DEFINITION_PROD=cropwise-backend-prod
DATABASE_URL_PROD=postgresql://...
```

#### **4.2 Create Production Deployment Workflow**

Create `.github/workflows/deploy-production.yml`:

```yaml
name: Deploy to Production

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  deploy-backend:
    name: Deploy Backend to Production
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://api.cropwise.com
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-south-1
      
      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v1
      
      - name: Build, tag, and push image
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          ECR_REPOSITORY: cropwise-backend
        run: |
          # Tag with version and prod
          VERSION=$(cat backend/package.json | jq -r '.version')
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:v$VERSION ./backend
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:prod ./backend
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:v$VERSION
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:prod
      
      - name: Update ECS service
        run: |
          aws ecs update-service \
            --cluster ${{ secrets.ECS_CLUSTER_PROD }} \
            --service ${{ secrets.ECS_SERVICE_PROD }} \
            --force-new-deployment \
            --region ap-south-1
      
      - name: Wait for deployment to complete
        run: |
          aws ecs wait services-stable \
            --cluster ${{ secrets.ECS_CLUSTER_PROD }} \
            --services ${{ secrets.ECS_SERVICE_PROD }} \
            --region ap-south-1

  deploy-frontend:
    name: Deploy Frontend to Production
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://cropwise.com
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install dependencies
        working-directory: ./frontend
        run: npm ci
      
      - name: Build frontend
        working-directory: ./frontend
        env:
          VITE_API_URL: ${{ secrets.BACKEND_URL_PROD }}
          VITE_GOOGLE_CLIENT_ID: ${{ secrets.GOOGLE_CLIENT_ID }}
        run: npm run build
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: ap-south-1
      
      - name: Deploy to S3
        run: |
          aws s3 sync frontend/dist s3://${{ secrets.S3_BUCKET_PROD }} --delete
      
      - name: Invalidate CloudFront cache
        run: |
          aws cloudfront create-invalidation \
            --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID_PROD }} \
            --paths "/*"

  smoke-tests:
    name: Run Smoke Tests
    needs: [deploy-backend, deploy-frontend]
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Health check
        run: |
          STATUS=$(curl -s -o /dev/null -w "%{http_code}" https://api.cropwise.com/health)
          if [ $STATUS -ne 200 ]; then
            echo "Health check failed with status $STATUS"
            exit 1
          fi
      
      - name: Frontend check
        run: |
          STATUS=$(curl -s -o /dev/null -w "%{http_code}" https://cropwise.com)
          if [ $STATUS -ne 200 ]; then
            echo "Frontend check failed with status $STATUS"
            exit 1
          fi

  notify:
    name: Send Deployment Notification
    needs: [smoke-tests]
    runs-on: ubuntu-latest
    if: always()
    
    steps:
      - name: Send Slack notification
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: '🚀 Production deployment completed!'
          fields: repo,message,commit,author
          webhook_url: ${{ secrets.SLACK_WEBHOOK_URL }}
```

#### **4.3 Configure GitHub Environment Protection**

1. Go to **Settings** → **Environments**
2. Click **production**
3. Enable **Required reviewers**:
   - Add 2-3 team members
4. Enable **Wait timer**: 5 minutes (safety delay)
5. Save rules

---

### 5. Deploy to Production

#### **5.1 Create Production Release**

```bash
# Ensure you're on main branch
git checkout main
git pull origin main

# Merge staging into main (after thorough testing)
git merge staging

# Create release tag
git tag -a v1.0.0 -m "Production release v1.0.0"
git push origin v1.0.0

# Push to main
git push origin main
```

#### **5.2 Approve Deployment**

1. Go to **Actions** tab in GitHub
2. Find "Deploy to Production" workflow
3. Review the deployment request
4. **Required reviewers approve**
5. Wait for 5-minute safety delay
6. Deployment proceeds automatically

#### **5.3 Monitor Deployment**

Watch the deployment in real-time:
```bash
# Backend ECS service
aws ecs describe-services \
  --cluster cropwise-prod-cluster \
  --services cropwise-backend-prod-service

# CloudWatch logs
aws logs tail /ecs/cropwise-backend-prod --follow
```

---

### 6. Verify Production Deployment

#### **6.1 Health Checks**

```bash
# Backend
curl https://api.cropwise.com/health

# Frontend
curl https://cropwise.com

# API test
curl https://api.cropwise.com/api/health
```

#### **6.2 Manual Testing**

**Critical Path Testing**:
- [ ] Navigate to https://cropwise.com
- [ ] Login with Google OAuth
- [ ] Create a farm
- [ ] Add crop batch
- [ ] Create task
- [ ] View dashboard
- [ ] Test notifications
- [ ] Verify data persistence
- [ ] Test on mobile device

#### **6.3 Performance Testing**

```bash
# Load test (use k6 or similar)
k6 run load-test.js

# Check response times
curl -w "@curl-format.txt" -s https://api.cropwise.com/health
```

---

## ✅ Verification Checklist

Before announcing go-live:

- [ ] Production infrastructure created
- [ ] Custom domain configured with SSL
- [ ] Database configured with backups
- [ ] GitHub Actions with approval process working
- [ ] All health checks passing
- [ ] Core features tested manually
- [ ] Performance acceptable (< 2s page load)
- [ ] Mobile responsiveness verified
- [ ] Security scan completed
- [ ] Monitoring and alerts active
- [ ] Backup and recovery tested
- [ ] Team trained on production procedures

---

## 🎉 What's Next?

Your production environment is live!

**Next Step**: [Step 10: Monitoring & Security](10-monitoring-security.md)

---

**Last Updated**: November 16, 2025

