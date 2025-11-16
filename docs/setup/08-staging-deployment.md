# 🎭 Step 8: Staging Environment Deployment

Complete guide to deploy CropWise to AWS Staging environment.

**Prerequisites**: Steps 1-7 completed

**Time Required**: 30-45 minutes

---

## 📋 What You'll Accomplish

By the end of this guide, you'll have:
- ✅ Staging AWS infrastructure created
- ✅ Staging database configured
- ✅ GitHub Actions deploying to staging automatically
- ✅ Staging environment tested and verified
- ✅ Staging URL accessible

---

## 🛠️ Step-by-Step Instructions

### 1. Create Staging AWS Resources

#### **1.1 Follow Development Infrastructure Guide**

Use the same steps as [Step 4: AWS Infrastructure (Development)](04-aws-infrastructure-dev.md), but with **staging** naming:

- ECR Repository: `cropwise-backend:stage` (already exists from dev setup)
- ECS Cluster: `cropwise-stage-cluster`
- Task Definition: `cropwise-backend-stage`
- ECS Service: `cropwise-backend-stage-service`
- Load Balancer: `cropwise-stage-alb`
- S3 Bucket: `cropwise-stage-frontend`
- CloudFront: `Staging` distribution

**Key Differences**:
- Use `stage` instead of `dev` in all resource names
- Use staging database URL (from Step 5)
- Higher resources (optional):
  - Task CPU: 512 (vs 256 for dev)
  - Task Memory: 1024 MB (vs 512 MB for dev)

---

### 2. Configure Staging Database

#### **2.1 Verify Staging RDS Instance**

From [Step 5: AWS Database Setup](05-aws-database-setup.md), verify you have:

```
Database Identifier: cropwise-stage-db
Database Name: cropwise_stage
Username: cropwise_admin
Endpoint: cropwise-stage-db.xxxxx.ap-south-1.rds.amazonaws.com
Port: 5432
```

#### **2.2 Run Migrations on Staging**

```bash
# Set staging database URL
export DATABASE_URL=postgresql://cropwise_admin:YOUR_PASSWORD@cropwise-stage-db.xxxxx.ap-south-1.rds.amazonaws.com:5432/cropwise_stage

# Run migrations
cd backend
npm run migrate

# Seed with test data
npm run seed:staging
```

---

### 3. Configure GitHub Actions for Staging

#### **3.1 Verify GitHub Secrets**

Ensure these secrets are set in GitHub (from Step 1):

**Staging-Specific Secrets**:
```
BACKEND_URL_STAGE=http://cropwise-stage-alb-XXX.ap-south-1.elb.amazonaws.com
FRONTEND_URL_STAGE=https://dXXXXXXXXXX.cloudfront.net
S3_BUCKET_STAGE=cropwise-stage-frontend
CLOUDFRONT_DISTRIBUTION_ID_STAGE=EXXXXXXXXX
ECS_CLUSTER_STAGE=cropwise-stage-cluster
ECS_SERVICE_STAGE=cropwise-backend-stage-service
ECS_TASK_DEFINITION_STAGE=cropwise-backend-stage
DATABASE_URL_STAGE=postgresql://...
```

#### **3.2 Create Staging Deployment Workflow**

Create `.github/workflows/deploy-staging.yml`:

```yaml
name: Deploy to Staging

on:
  push:
    branches: [staging]
  pull_request:
    branches: [staging]
    types: [closed]
  workflow_dispatch:

jobs:
  deploy-backend:
    name: Deploy Backend to Staging
    runs-on: ubuntu-latest
    if: github.event_name == 'push' || (github.event_name == 'pull_request' && github.event.pull_request.merged == true)
    
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
      
      - name: Build, tag, and push image to Amazon ECR
        env:
          ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          ECR_REPOSITORY: cropwise-backend
          IMAGE_TAG: stage
        run: |
          docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG ./backend
          docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
      
      - name: Update ECS service
        run: |
          aws ecs update-service \
            --cluster ${{ secrets.ECS_CLUSTER_STAGE }} \
            --service ${{ secrets.ECS_SERVICE_STAGE }} \
            --force-new-deployment \
            --region ap-south-1

  deploy-frontend:
    name: Deploy Frontend to Staging
    runs-on: ubuntu-latest
    if: github.event_name == 'push' || (github.event_name == 'pull_request' && github.event.pull_request.merged == true)
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
          cache-dependency-path: frontend/package-lock.json
      
      - name: Install dependencies
        working-directory: ./frontend
        run: npm ci
      
      - name: Build frontend
        working-directory: ./frontend
        env:
          VITE_API_URL: ${{ secrets.BACKEND_URL_STAGE }}
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
          aws s3 sync frontend/dist s3://${{ secrets.S3_BUCKET_STAGE }} --delete
      
      - name: Invalidate CloudFront cache
        run: |
          aws cloudfront create-invalidation \
            --distribution-id ${{ secrets.CLOUDFRONT_DISTRIBUTION_ID_STAGE }} \
            --paths "/*"

  run-tests:
    name: Run E2E Tests on Staging
    needs: [deploy-backend, deploy-frontend]
    runs-on: ubuntu-latest
    
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run E2E tests
        env:
          TEST_URL: ${{ secrets.FRONTEND_URL_STAGE }}
          API_URL: ${{ secrets.BACKEND_URL_STAGE }}
        run: npm run test:e2e
      
      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: e2e-test-results
          path: test-results/

  notify:
    name: Send Deployment Notification
    needs: [run-tests]
    runs-on: ubuntu-latest
    if: always()
    
    steps:
      - name: Send Slack notification
        uses: 8398a7/action-slack@v3
        with:
          status: ${{ job.status }}
          text: 'Staging deployment completed!'
          webhook_url: ${{ secrets.SLACK_WEBHOOK_URL }}
        if: env.SLACK_WEBHOOK_URL != ''
```

---

### 4. Test Staging Deployment

#### **4.1 Push to Staging Branch**

```bash
# Create PR from develop to staging
git checkout develop
git pull origin develop

# Create staging branch if it doesn't exist
git checkout -b staging
git push origin staging

# Or merge develop into staging
git checkout staging
git merge develop
git push origin staging
```

#### **4.2 Monitor GitHub Actions**

1. Go to your repository → **Actions** tab
2. Watch the "Deploy to Staging" workflow
3. Check for any errors
4. Wait for all jobs to complete (5-10 minutes)

---

### 5. Verify Staging Deployment

#### **5.1 Health Check**

```bash
# Backend health check
curl http://cropwise-stage-alb-XXX.ap-south-1.elb.amazonaws.com/health

# Expected response:
# {"status":"ok","environment":"staging","timestamp":"..."}
```

#### **5.2 Frontend Access**

1. Open your staging CloudFront URL in browser:
   ```
   https://dXXXXXXXXXX.cloudfront.net
   ```

2. You should see the CropWise login page

#### **5.3 Test Core Features**

**Manual Testing Checklist**:
- [ ] Login with Google OAuth
- [ ] Create a farm
- [ ] Add a crop batch
- [ ] Create a task
- [ ] View dashboard
- [ ] Check notifications
- [ ] Test API endpoints
- [ ] Verify database connection

---

### 6. Configure Custom Domain (Optional)

#### **6.1 Using Route 53**

1. Go to Route 53 → **Hosted zones**
2. Select your domain
3. Create record:
   - **Record name**: `staging.cropwise.com`
   - **Record type**: A (Alias)
   - **Route traffic to**: Alias to CloudFront distribution
   - **Choose distribution**: Select your staging CloudFront

4. Update CloudFront:
   - Add alternate domain name (CNAME): `staging.cropwise.com`
   - Request ACM certificate for `staging.cropwise.com`
   - Update CloudFront distribution

#### **6.2 Update Environment Variables**

```bash
# Update in GitHub Secrets
FRONTEND_URL_STAGE=https://staging.cropwise.com
BACKEND_URL_STAGE=https://api-staging.cropwise.com
```

---

### 7. Set Up Monitoring

#### **7.1 CloudWatch Alarms**

Create alarms for staging:

```bash
# CPU utilization alarm
aws cloudwatch put-metric-alarm \
  --alarm-name cropwise-stage-high-cpu \
  --alarm-description "CPU utilization > 80% for staging" \
  --metric-name CPUUtilization \
  --namespace AWS/ECS \
  --statistic Average \
  --period 300 \
  --evaluation-periods 2 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=ServiceName,Value=cropwise-backend-stage-service \
                Name=ClusterName,Value=cropwise-stage-cluster

# Memory utilization alarm
aws cloudwatch put-metric-alarm \
  --alarm-name cropwise-stage-high-memory \
  --alarm-description "Memory utilization > 80% for staging" \
  --metric-name MemoryUtilization \
  --namespace AWS/ECS \
  --statistic Average \
  --period 300 \
  --evaluation-periods 2 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=ServiceName,Value=cropwise-backend-stage-service \
                Name=ClusterName,Value=cropwise-stage-cluster
```

#### **7.2 View Logs**

```bash
# Backend logs
aws logs tail /ecs/cropwise-backend-stage --follow

# Access logs
aws logs tail cropwise-stage-alb-logs --follow
```

---

## ✅ Verification Checklist

Before moving to production, ensure:

- [ ] Staging infrastructure created
- [ ] Staging database configured and migrated
- [ ] GitHub Actions deploy-staging.yml working
- [ ] Backend health endpoint returns OK
- [ ] Frontend accessible via CloudFront URL
- [ ] Google OAuth login works
- [ ] All core features tested manually
- [ ] E2E tests passing
- [ ] CloudWatch alarms configured
- [ ] Logs accessible and readable
- [ ] No errors in staging environment
- [ ] Performance acceptable

---

## 🎉 What's Next?

Your staging environment is ready and tested!

**Next Step**: [Step 9: Production Deployment](09-production-deployment.md)

---

## 📚 Related Documentation

- [Deployment Guide](../deployment/DEPLOYMENT_GUIDE.md) - Full deployment guide
- [Troubleshooting](../deployment/TROUBLESHOOTING.md) - Common issues
- [Monitoring Guide](../operations/MONITORING_GUIDE.md) - Monitoring best practices

---

**Last Updated**: November 16, 2025

