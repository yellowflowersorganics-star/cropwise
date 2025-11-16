# 🚀 CropWise Deployment Test Checklist

Complete guide to test your production deployment.

---

## ✅ Pre-Deployment Checklist

Before deploying, verify all secrets are configured:

### **GitHub Secrets Status**
Visit: https://github.com/yellowflowersorganics-star/cropwise/settings/secrets/actions

- [x] `VITE_API_URL` ✅
- [x] `AWS_ACCESS_KEY_ID` ✅
- [x] `AWS_SECRET_ACCESS_KEY` ✅
- [x] `AWS_REGION` ✅
- [x] `AWS_ACCESS_KEY_ID_PROD` ✅
- [x] `AWS_SECRET_ACCESS_KEY_PROD` ✅
- [x] `JWT_SECRET` ✅
- [x] `SESSION_SECRET` ✅
- [x] `DATABASE_URL` ✅
- [x] `GOOGLE_CLIENT_ID` ✅
- [x] `GOOGLE_CLIENT_SECRET` ✅

**All secrets configured!** ✅

---

## 🧪 STEP 1: Deploy to Development

### **1.1 Push to Develop Branch**

```powershell
cd C:\Users\praghav\cropwise

# Ensure you're on develop branch
git checkout develop

# Pull latest changes
git pull origin develop

# Make a small change to trigger deployment
echo "# Deployment test $(Get-Date)" >> DEPLOYMENT_LOG.md
git add DEPLOYMENT_LOG.md
git commit -m "test: Trigger development deployment"

# Push to trigger GitHub Actions
git push origin develop
```

### **1.2 Monitor Deployment**

1. **Go to**: https://github.com/yellowflowersorganics-star/cropwise/actions
2. **Watch** the workflow run in real-time
3. **Expected duration**: 5-10 minutes

### **1.3 Verify Deployment Success**

Check that all jobs completed:
- ✅ **Lint** - Code quality checks
- ✅ **Test** - Unit tests passed
- ✅ **Build Backend** - Docker image created
- ✅ **Build Frontend** - Static files generated
- ✅ **Security Scan** - No vulnerabilities found
- ✅ **Deploy to DEV** - Deployed to development environment

**Development URL**: https://dev.cropwise.io (or your configured URL)

---

## 🔍 STEP 2: Test Development Environment

### **2.1 Backend Health Check**

```powershell
# Test backend API
curl https://dev-api.cropwise.io/health

# Expected response:
# {
#   "status": "ok",
#   "timestamp": "2025-11-14T...",
#   "database": "connected"
# }
```

### **2.2 Frontend Access**

1. **Open**: https://dev.cropwise.io
2. **Verify**:
   - ✅ Page loads without errors
   - ✅ Login page is visible
   - ✅ "Sign in with Google" button appears
   - ✅ No console errors (F12 → Console)

### **2.3 Database Connection**

```powershell
# Check backend logs for database connection
# In AWS Console → ECS → cropwise-dev-cluster → Tasks → View Logs

# Should see:
# ✅ "Database connected successfully"
# ✅ "Migrations completed"
```

### **2.4 Test User Registration**

1. **Go to**: https://dev.cropwise.io/register
2. **Fill in**:
   - Email: test@example.com
   - Password: TestPassword123!
3. **Submit** registration
4. **Expected**:
   - ✅ User created successfully
   - ✅ JWT token received
   - ✅ Redirected to dashboard

### **2.5 Test Google OAuth**

1. **Go to**: https://dev.cropwise.io/login
2. **Click**: "Sign in with Google"
3. **Expected**:
   - ✅ Redirects to Google login
   - ✅ Shows OAuth consent screen
   - ✅ After approval, redirects back to app
   - ✅ User logged in successfully

**If Google OAuth fails**: See troubleshooting section below

### **2.6 Test Session Persistence**

1. **Login** to the app
2. **Refresh** the page (F5)
3. **Expected**:
   - ✅ Still logged in (session maintained)
   - ✅ No need to login again
   - ✅ SESSION_SECRET is working!

---

## 🚀 STEP 3: Deploy to Production

**Only proceed if development tests passed!**

### **3.1 Merge to Main Branch**

```powershell
cd C:\Users\praghav\cropwise

# Switch to main branch
git checkout main

# Pull latest
git pull origin main

# Merge develop into main
git merge develop

# Review changes
git log --oneline -5

# Push to trigger production deployment
git push origin main
```

### **3.2 Monitor Production Deployment**

1. **Go to**: https://github.com/yellowflowersorganics-star/cropwise/actions
2. **Watch** the production deployment workflow
3. **Note**: Production may require manual approval (security feature)
4. **Duration**: 5-15 minutes

### **3.3 Verify Production Deployment**

All jobs should complete successfully:
- ✅ **Build** - All builds successful
- ✅ **Security Scan** - Passed
- ✅ **Deploy to Production** - Completed
- ✅ **Health Checks** - Passing

**Production URL**: https://www.cropwise.io (or your configured URL)

---

## 🔍 STEP 4: Test Production Environment

### **4.1 Production Health Check**

```powershell
# Test production API
curl https://api.cropwise.io/health

# Expected:
# {
#   "status": "ok",
#   "environment": "production",
#   "database": "connected"
# }
```

### **4.2 Production Frontend**

1. **Open**: https://www.cropwise.io
2. **Verify**:
   - ✅ HTTPS enabled (secure connection)
   - ✅ Page loads quickly
   - ✅ No mixed content warnings
   - ✅ All assets load correctly

### **4.3 Production Database**

**Check CloudWatch Logs**:
1. Go to: https://console.aws.amazon.com/cloudwatch/
2. **Log Groups** → `/ecs/cropwise-prod`
3. **Look for**:
   - ✅ "Connected to database"
   - ✅ No connection errors

**Check RDS Status**:
1. Go to: https://console.aws.amazon.com/rds/
2. **Databases** → `cropwise-production-db`
3. **Verify**:
   - ✅ Status: Available
   - ✅ CPU Utilization < 50%
   - ✅ Database Connections: Active

### **4.4 Production Google OAuth**

⚠️ **IMPORTANT**: Update Google Cloud Console first!

1. Add production URLs to Google Console (see Step 1 above)
2. Test login with Google on production
3. Verify successful authentication

### **4.5 End-to-End Production Test**

**Complete User Journey**:
1. ✅ Visit homepage
2. ✅ Register new account
3. ✅ Verify email (if implemented)
4. ✅ Login with credentials
5. ✅ Logout
6. ✅ Login with Google
7. ✅ Access dashboard
8. ✅ Create/view data
9. ✅ Logout

---

## 📊 STEP 5: Monitor Production

### **5.1 Set Up CloudWatch Alarms**

```powershell
# Create CPU alarm
aws cloudwatch put-metric-alarm `
  --alarm-name cropwise-prod-high-cpu `
  --alarm-description "Alert when CPU > 80%" `
  --metric-name CPUUtilization `
  --namespace AWS/ECS `
  --statistic Average `
  --period 300 `
  --threshold 80 `
  --comparison-operator GreaterThanThreshold
```

### **5.2 Set Up Billing Alert**

1. **Go to**: https://console.aws.amazon.com/billing/home#/budgets
2. **Create Budget**:
   - Type: Cost budget
   - Amount: $50/month (or your budget)
   - Alert threshold: 80%
3. **Add email** for notifications

### **5.3 Monitor Key Metrics**

**Backend (ECS)**:
- CPU Utilization < 70%
- Memory Utilization < 80%
- Task count: Running

**Database (RDS)**:
- CPU Utilization < 60%
- Connections: < 80% of max
- Free Storage Space: > 20%

**Frontend (CloudFront)**:
- 4xx Error Rate: < 1%
- 5xx Error Rate: < 0.1%
- Cache Hit Rate: > 80%

---

## 🚨 Troubleshooting

### **Issue 1: Deployment Failed**

**Check GitHub Actions Logs**:
1. Go to failed workflow
2. Click on failed job
3. Expand error section
4. Look for specific error

**Common Errors**:

**A. AWS Credentials Invalid**
```
Error: The security token included in the request is invalid
```
**Solution**: Verify AWS secrets in GitHub are correct

**B. Docker Build Failed**
```
Error: failed to solve with frontend dockerfile.v0
```
**Solution**: Check Dockerfile syntax, rebuild locally first

**C. Database Connection Failed**
```
Error: Connection refused to database
```
**Solution**: 
- Verify DATABASE_URL is correct
- Check RDS security group allows ECS connections
- Verify RDS is in same VPC as ECS

---

### **Issue 2: Google OAuth Not Working**

**Error**: "redirect_uri_mismatch"

**Solution**:
1. Go to: https://console.cloud.google.com/apis/credentials
2. Click your OAuth Client ID
3. Verify redirect URIs match exactly:
   ```
   https://api.cropwise.io/api/auth/google/callback
   https://www.cropwise.io/auth/google/callback
   ```
4. No typos, no trailing slashes
5. Save and wait 5 minutes

**Error**: "Access blocked: This app's request is invalid"

**Solution**:
1. Go to OAuth Consent Screen
2. Add your email as test user
3. Verify scopes include email and profile

---

### **Issue 3: Session Not Persisting**

**Symptoms**: User logged out on page refresh

**Solution**:
1. Verify SESSION_SECRET is in GitHub Secrets
2. Check browser console for cookie errors
3. Ensure cookies are enabled
4. For HTTPS: verify secure cookie settings

---

### **Issue 4: Database Migration Failed**

**Error**: "relation does not exist"

**Solution**:
```powershell
# Connect to production database
psql $env:DATABASE_URL

# Check if tables exist
\dt

# If no tables, run migrations manually
cd backend
npm run migration:run
```

---

## ✅ Success Criteria

Your deployment is successful if:

### **Development Environment**
- ✅ GitHub Actions workflow completes without errors
- ✅ Backend health check returns 200 OK
- ✅ Frontend loads without errors
- ✅ Database connection successful
- ✅ Can register new user
- ✅ Can login with credentials
- ✅ Session persists after refresh

### **Production Environment**
- ✅ All development checks pass
- ✅ HTTPS enabled
- ✅ Google OAuth works
- ✅ No console errors
- ✅ CloudWatch metrics healthy
- ✅ RDS metrics healthy
- ✅ No billing alerts

---

## 📈 Performance Benchmarks

Your app should meet these benchmarks:

### **Response Times**
- Homepage: < 1 second
- API calls: < 500ms
- Database queries: < 100ms

### **Availability**
- Uptime: > 99.5%
- Error rate: < 0.5%

### **Resource Usage**
- Backend CPU: < 50% average
- Backend Memory: < 70%
- Database CPU: < 40%
- Database Connections: < 50

---

## 🎯 Post-Deployment Tasks

After successful deployment:

### **Immediate (Today)**
- [ ] Test all critical user flows
- [ ] Verify Google OAuth works
- [ ] Check CloudWatch logs for errors
- [ ] Set up billing alerts
- [ ] Save production URLs in documentation

### **This Week**
- [ ] Set up monitoring dashboards
- [ ] Configure CloudWatch alarms
- [ ] Test backup and restore procedures
- [ ] Document deployment process
- [ ] Set up Slack notifications (optional)

### **This Month**
- [ ] Performance optimization
- [ ] Load testing
- [ ] Security audit
- [ ] Cost optimization review
- [ ] Scale testing

---

## 📚 Additional Resources

- **GitHub Actions**: https://github.com/yellowflowersorganics-star/cropwise/actions
- **AWS Console**: https://console.aws.amazon.com
- **CloudWatch Logs**: https://console.aws.amazon.com/cloudwatch/
- **RDS Console**: https://console.aws.amazon.com/rds/
- **Google Cloud Console**: https://console.cloud.google.com

---

## 🎉 Congratulations!

If all tests pass, you've successfully deployed CropWise to production! 🚀

**Your architecture**:
```
Users
  ↓
CloudFront CDN (Frontend)
  ↓
Application Load Balancer
  ↓
ECS Fargate (Backend)
  ↓
RDS PostgreSQL (Database)
```

**All managed by AWS, deployed via GitHub Actions!**

---

**Created**: November 14, 2025  
**Status**: Ready for Production Deployment ✅

