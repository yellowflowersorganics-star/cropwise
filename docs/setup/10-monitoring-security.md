# 🔒 Step 10: Monitoring & Security

Final step: Set up comprehensive monitoring, alerts, and security for production.

**Prerequisites**: [Step 9 (Production Deployment)](09-production-deployment.md) completed

**Time Required**: 30-45 minutes

---

## 📋 What You'll Accomplish

By the end of this guide, you'll have:
- ✅ CloudWatch monitoring and dashboards
- ✅ Alarms for critical metrics
- ✅ Log aggregation and analysis
- ✅ Security best practices implemented
- ✅ Backup and disaster recovery configured
- ✅ Cost monitoring and optimization
- ✅ Incident response procedures

---

## 🛠️ Step-by-Step Instructions

### 1. CloudWatch Monitoring

#### **1.1 Create CloudWatch Dashboard**

1. Go to CloudWatch → **Dashboards**
2. Click **Create dashboard**
3. Name: `CropWise-Production-Dashboard`

Add widgets for:

**Backend Metrics**:
- ECS CPU Utilization
- ECS Memory Utilization
- ECS Task Count
- ALB Target Response Time
- ALB Request Count
- ALB 4XX/5XX Errors

**Database Metrics**:
- RDS CPU Utilization
- RDS Database Connections
- RDS Read/Write Latency
- RDS Free Storage Space

**Frontend Metrics**:
- CloudFront Request Count
- CloudFront Error Rate
- CloudFront Cache Hit Rate

#### **1.2 Create Alarms**

**High CPU Alarm**:
```bash
aws cloudwatch put-metric-alarm \
  --alarm-name CropWise-Prod-High-CPU \
  --alarm-description "CPU > 80% for 5 minutes" \
  --metric-name CPUUtilization \
  --namespace AWS/ECS \
  --statistic Average \
  --period 300 \
  --evaluation-periods 1 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=ServiceName,Value=cropwise-backend-prod-service \
                Name=ClusterName,Value=cropwise-prod-cluster \
  --alarm-actions arn:aws:sns:ap-south-1:ACCOUNT_ID:cropwise-alerts
```

**High Memory Alarm**:
```bash
aws cloudwatch put-metric-alarm \
  --alarm-name CropWise-Prod-High-Memory \
  --alarm-description "Memory > 80% for 5 minutes" \
  --metric-name MemoryUtilization \
  --namespace AWS/ECS \
  --statistic Average \
  --period 300 \
  --evaluation-periods 1 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=ServiceName,Value=cropwise-backend-prod-service \
                Name=ClusterName,Value=cropwise-prod-cluster \
  --alarm-actions arn:aws:sns:ap-south-1:ACCOUNT_ID:cropwise-alerts
```

**Database Alarms**:
```bash
# High database CPU
aws cloudwatch put-metric-alarm \
  --alarm-name CropWise-Prod-DB-High-CPU \
  --alarm-description "RDS CPU > 80%" \
  --metric-name CPUUtilization \
  --namespace AWS/RDS \
  --statistic Average \
  --period 300 \
  --evaluation-periods 2 \
  --threshold 80 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=DBInstanceIdentifier,Value=cropwise-prod-db \
  --alarm-actions arn:aws:sns:ap-south-1:ACCOUNT_ID:cropwise-alerts

# Low storage space
aws cloudwatch put-metric-alarm \
  --alarm-name CropWise-Prod-DB-Low-Storage \
  --alarm-description "RDS Free Storage < 5 GB" \
  --metric-name FreeStorageSpace \
  --namespace AWS/RDS \
  --statistic Average \
  --period 300 \
  --evaluation-periods 1 \
  --threshold 5000000000 \
  --comparison-operator LessThanThreshold \
  --dimensions Name=DBInstanceIdentifier,Value=cropwise-prod-db \
  --alarm-actions arn:aws:sns:ap-south-1:ACCOUNT_ID:cropwise-alerts
```

**Error Rate Alarm**:
```bash
aws cloudwatch put-metric-alarm \
  --alarm-name CropWise-Prod-High-Errors \
  --alarm-description "5XX errors > 10 in 5 minutes" \
  --metric-name HTTPCode_Target_5XX_Count \
  --namespace AWS/ApplicationELB \
  --statistic Sum \
  --period 300 \
  --evaluation-periods 1 \
  --threshold 10 \
  --comparison-operator GreaterThanThreshold \
  --dimensions Name=LoadBalancer,Value=app/cropwise-prod-alb/XXXXX \
  --alarm-actions arn:aws:sns:ap-south-1:ACCOUNT_ID:cropwise-alerts
```

---

### 2. Log Management

#### **2.1 Enable CloudWatch Logs Insights**

1. Go to CloudWatch → **Logs** → **Insights**
2. Select log groups:
   - `/ecs/cropwise-backend-prod`
   - `/aws/rds/instance/cropwise-prod-db/error`
   - `/aws/elasticloadbalancing/app/cropwise-prod-alb`

**Useful Queries**:

**Find errors**:
```
fields @timestamp, @message
| filter @message like /ERROR/
| sort @timestamp desc
| limit 100
```

**API response times**:
```
fields @timestamp, duration
| filter @message like /Request completed/
| stats avg(duration), max(duration), min(duration) by bin(5m)
```

**Failed logins**:
```
fields @timestamp, @message
| filter @message like /Login failed/
| stats count() by bin(1h)
```

#### **2.2 Set Up Log Retention**

```bash
# Backend logs - keep for 30 days
aws logs put-retention-policy \
  --log-group-name /ecs/cropwise-backend-prod \
  --retention-in-days 30

# Database logs - keep for 7 days
aws logs put-retention-policy \
  --log-group-name /aws/rds/instance/cropwise-prod-db/error \
  --retention-in-days 7
```

---

### 3. Security Configuration

#### **3.1 Enable AWS WAF**

1. Go to **WAF & Shield** → **Web ACLs**
2. Click **Create web ACL**
3. Name: `cropwise-prod-waf`
4. Resource type: CloudFront distribution
5. Add rule groups:
   - ✅ Core rule set (OWASP Top 10)
   - ✅ Known bad inputs
   - ✅ SQL injection
   - ✅ Rate limiting (2000 requests/5min per IP)

6. Associate with:
   - Production CloudFront distribution
   - Production ALB

#### **3.2 Enable AWS Shield**

Free tier is automatic. For DDoS protection:

```bash
# Enable Shield Standard (free)
aws shield describe-subscription

# Optional: Upgrade to Shield Advanced ($3000/month)
# Provides 24/7 DDoS response team and cost protection
```

#### **3.3 Enable GuardDuty**

1. Go to **GuardDuty**
2. Click **Get Started**
3. Enable GuardDuty
4. Configure notifications to SNS topic

```bash
aws guardduty create-detector --enable
```

#### **3.4 Enable Security Hub**

```bash
# Enable Security Hub
aws securityhub enable-security-hub

# Enable security standards
aws securityhub batch-enable-standards \
  --standards-subscription-requests \
    StandardsArn=arn:aws:securityhub:::ruleset/cis-aws-foundations-benchmark/v/1.2.0
```

#### **3.5 Secrets Rotation**

1. Go to **AWS Secrets Manager**
2. Store production secrets:
   - Database credentials
   - JWT secret
   - Google OAuth credentials
   - Twilio credentials

3. Enable automatic rotation:
   - Database password: Every 30 days
   - API keys: Every 90 days

```bash
# Store database credentials
aws secretsmanager create-secret \
  --name cropwise/prod/database \
  --description "Production database credentials" \
  --secret-string file://db-secret.json

# Enable rotation
aws secretsmanager rotate-secret \
  --secret-id cropwise/prod/database \
  --rotation-rules AutomaticallyAfterDays=30
```

---

### 4. Backup & Disaster Recovery

#### **4.1 RDS Automated Backups**

Already configured in Step 5, verify:

```bash
aws rds describe-db-instances \
  --db-instance-identifier cropwise-prod-db \
  --query 'DBInstances[0].[BackupRetentionPeriod,PreferredBackupWindow]'

# Should return: [7, "03:00-04:00"]
```

#### **4.2 Create Manual Snapshot**

```bash
# Create snapshot before major changes
aws rds create-db-snapshot \
  --db-instance-identifier cropwise-prod-db \
  --db-snapshot-identifier cropwise-prod-pre-migration-$(date +%Y%m%d)
```

#### **4.3 S3 Versioning**

Already enabled. Verify:

```bash
aws s3api get-bucket-versioning \
  --bucket cropwise-prod-frontend

# Should return: "Status": "Enabled"
```

#### **4.4 Disaster Recovery Plan**

**RTO (Recovery Time Objective)**: 4 hours  
**RPO (Recovery Point Objective)**: 5 minutes

**Recovery Procedures**:

1. **Database Failure**:
   ```bash
   # Promote read replica to master
   aws rds promote-read-replica \
     --db-instance-identifier cropwise-prod-db-read-replica
   
   # Update backend with new endpoint
   # Redeploy
   ```

2. **Application Failure**:
   ```bash
   # Rollback to previous version
   aws ecs update-service \
     --cluster cropwise-prod-cluster \
     --service cropwise-backend-prod-service \
     --task-definition cropwise-backend-prod:PREVIOUS_REVISION \
     --force-new-deployment
   ```

3. **Frontend Failure**:
   ```bash
   # Rollback S3 to previous version
   aws s3 sync s3://cropwise-prod-frontend-backup s3://cropwise-prod-frontend
   
   # Invalidate CloudFront
   aws cloudfront create-invalidation \
     --distribution-id EXXXXXXXXX \
     --paths "/*"
   ```

---

### 5. Cost Monitoring

#### **5.1 Cost Allocation Tags**

Tag all resources:

```bash
# Tag ECS cluster
aws ecs tag-resource \
  --resource-arn arn:aws:ecs:ap-south-1:ACCOUNT_ID:cluster/cropwise-prod-cluster \
  --tags key=Project,value=CropWise key=Environment,value=Production key=CostCenter,value=Engineering

# Tag RDS
aws rds add-tags-to-resource \
  --resource-name arn:aws:rds:ap-south-1:ACCOUNT_ID:db:cropwise-prod-db \
  --tags Key=Project,Value=CropWise Key=Environment,Value=Production
```

#### **5.2 Enable Cost Anomaly Detection**

1. Go to **Cost Management** → **Cost Anomaly Detection**
2. Click **Create monitor**
3. Name: `CropWise-Cost-Monitor`
4. Alert threshold: $50 (adjust as needed)
5. SNS topic: `cropwise-cost-alerts`

#### **5.3 Review Cost Explorer**

Weekly review:
1. Go to **Cost Explorer**
2. Filter by tag: `Project=CropWise`
3. Review trends
4. Identify optimization opportunities

---

### 6. Incident Response

#### **6.1 Create Runbooks**

Document procedures for common incidents:

**High CPU/Memory**:
1. Check CloudWatch logs for errors
2. Review recent deployments
3. Scale up ECS tasks if needed
4. Roll back if necessary

**Database Issues**:
1. Check RDS performance insights
2. Review slow query logs
3. Optimize queries
4. Scale up instance if needed

**5XX Errors**:
1. Check ECS task health
2. Review application logs
3. Check database connectivity
4. Verify external service availability

#### **6.2 On-Call Rotation**

Set up PagerDuty or similar:
1. Create escalation policies
2. Set up on-call schedule
3. Configure SNS to notify PagerDuty
4. Test alerting

#### **6.3 Post-Mortem Template**

After incidents, create post-mortem:
- What happened?
- Root cause analysis
- Timeline of events
- Resolution steps
- Preventive measures
- Action items

---

## ✅ Final Verification Checklist

Ensure everything is configured:

**Monitoring**:
- [ ] CloudWatch dashboard created
- [ ] All critical alarms configured
- [ ] SNS topic for alerts
- [ ] Log Insights queries saved
- [ ] Log retention policies set

**Security**:
- [ ] AWS WAF enabled
- [ ] GuardDuty enabled
- [ ] Security Hub enabled
- [ ] Secrets in Secrets Manager
- [ ] Secrets rotation enabled
- [ ] MFA on all accounts

**Backup & DR**:
- [ ] RDS automated backups enabled
- [ ] Manual snapshots taken
- [ ] S3 versioning enabled
- [ ] DR procedures documented
- [ ] Recovery tested (in staging)

**Cost Management**:
- [ ] Resources tagged
- [ ] Cost anomaly detection enabled
- [ ] Budgets configured
- [ ] Weekly cost review scheduled

**Operations**:
- [ ] Runbooks documented
- [ ] On-call rotation set up
- [ ] Team trained
- [ ] Post-mortem template ready

---

## 🎉 Congratulations!

You've successfully completed the full CropWise setup!

**What You've Accomplished**:
- ✅ GitHub repository with proper workflow
- ✅ Local development environment
- ✅ AWS account and infrastructure
- ✅ Three environments (dev, staging, prod)
- ✅ Automated CI/CD pipelines
- ✅ Authentication and integrations
- ✅ Production deployment with custom domain
- ✅ Comprehensive monitoring and security
- ✅ Backup and disaster recovery
- ✅ Cost optimization

---

## 📚 Next Steps

Now that setup is complete, focus on:

1. **Development**: [Developer Guide](../development/DEVELOPER_GUIDE.md)
2. **Operations**: [Admin Guide](../operations/ADMIN_GUIDE.md)
3. **Feature Development**: Start building!

---

## 🆘 Support

If you need help:
- Check [Troubleshooting Guide](../deployment/TROUBLESHOOTING.md)
- Review [FAQ](../reference/FAQ.md)
- Contact the team

---

**Last Updated**: November 16, 2025

**You're all set! Happy farming! 🌱**

