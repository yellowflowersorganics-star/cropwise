# 🔄 CropWise → Yellow Flowers SmartFarm Cloud
## Migration & Implementation Plan

**Current Status**: CropWise v1.0 (Single-tenant foundation)  
**Target**: Yellow Flowers SmartFarm Cloud (Multi-tenant SaaS)  
**Timeline**: 6-9 months to full commercial launch

---

## 📊 Gap Analysis

### What We Have (CropWise)
✅ Backend API with Node.js/Express  
✅ PostgreSQL database  
✅ JWT authentication  
✅ Farm & Zone models  
✅ Crop recipe system  
✅ MQTT service  
✅ ESP32 firmware  
✅ React dashboard  
✅ Recipe execution engine  

### What We Need (SmartFarm Cloud)
🔄 Multi-tenant data architecture  
🆕 Organization/Customer model  
🆕 Subscription & billing system  
🔄 MQTT topic namespacing  
🔄 Enhanced security (TLS, ACL)  
🆕 InfluxDB time-series storage  
🆕 Analytics & ML module  
🆕 Mobile app  
🆕 Branded hardware  

---

## 🚀 Implementation Roadmap

### Phase 2A: Multi-Tenancy (Weeks 1-8)

#### Week 1-2: Database Schema Update
- [ ] Add `Organization` model
- [ ] Add `organization_id` to all existing models
- [ ] Create migrations
- [ ] Implement Row-Level Security
- [ ] Update all queries with tenant context

#### Week 3-4: API Multi-Tenant Support
- [ ] Update authentication middleware
- [ ] Add organization context to req object
- [ ] Update all controllers with tenant filtering
- [ ] Add organization management APIs
- [ ] Create organization onboarding flow

#### Week 5-6: MQTT Topic Restructuring
- [ ] Update topic structure with customer_id prefix
- [ ] Modify ESP32 firmware for new topics
- [ ] Update backend MQTT service
- [ ] Add MQTT ACL configuration
- [ ] Test multi-tenant isolation

#### Week 7-8: Frontend Multi-Tenant Updates
- [ ] Add organization selector
- [ ] Update all API calls with org context
- [ ] Add org management UI
- [ ] Test tenant isolation
- [ ] Performance testing

**Milestone**: ✅ Multi-tenant platform ready for testing

---

### Phase 2B: Billing Integration (Weeks 9-12)

#### Week 9: Subscription Model
- [ ] Create `Subscription` model
- [ ] Create `SubscriptionPlan` model
- [ ] Define plan tiers (Starter/Growth/Enterprise)
- [ ] Implement plan limits checking
- [ ] Add subscription status middleware

#### Week 10: Razorpay Integration
- [ ] Set up Razorpay account
- [ ] Integrate Razorpay SDK
- [ ] Create subscription plans in Razorpay
- [ ] Implement webhook handlers
- [ ] Test payment flows

#### Week 11: Billing Dashboard
- [ ] Create billing page UI
- [ ] Add plan selection interface
- [ ] Implement payment button
- [ ] Add invoice display
- [ ] Create usage tracking UI

#### Week 12: Testing & Polish
- [ ] Test all payment scenarios
- [ ] Test plan upgrades/downgrades
- [ ] Test subscription cancellation
- [ ] Add billing notifications
- [ ] Documentation

**Milestone**: ✅ Automated billing system operational

---

### Phase 3: Time-Series Database (Weeks 13-16)

#### Week 13: InfluxDB Setup
- [ ] Set up InfluxDB Cloud account
- [ ] Create organization and buckets
- [ ] Define retention policies
- [ ] Set up authentication tokens
- [ ] Configure backups

#### Week 14: Data Migration
- [ ] Create InfluxDB service module
- [ ] Migrate telemetry writes to InfluxDB
- [ ] Keep PostgreSQL for metadata
- [ ] Update MQTT handler
- [ ] Test write performance

#### Week 15: Query APIs
- [ ] Create historical data APIs
- [ ] Add aggregation queries
- [ ] Implement downsampling
- [ ] Add export functionality
- [ ] Performance optimization

#### Week 16: Dashboard Integration
- [ ] Update charts to use InfluxDB
- [ ] Add time range selectors
- [ ] Implement real-time updates
- [ ] Test with large datasets
- [ ] Documentation

**Milestone**: ✅ Scalable telemetry storage

---

### Phase 4: Enhanced Security (Weeks 17-18)

#### Week 17: MQTT Security
- [ ] Enable TLS on MQTT broker
- [ ] Create per-customer certificates
- [ ] Implement ACL rules
- [ ] Update ESP32 firmware for TLS
- [ ] Test certificate rotation

#### Week 18: API Security
- [ ] Add rate limiting
- [ ] Implement audit logging
- [ ] Add API key management
- [ ] Security penetration testing
- [ ] Documentation updates

**Milestone**: ✅ Enterprise-grade security

---

### Phase 5: Analytics & ML (Weeks 19-26)

#### Weeks 19-22: Data Collection
- [ ] Deploy to 5-10 pilot farms
- [ ] Collect 3 months of production data
- [ ] Clean and validate data
- [ ] Create training datasets

#### Weeks 23-24: Model Development
- [ ] Build yield prediction model
- [ ] Create energy optimization model
- [ ] Develop labour forecasting
- [ ] Train and validate models

#### Weeks 25-26: Integration
- [ ] Create ML API service
- [ ] Add prediction endpoints
- [ ] Update dashboard with insights
- [ ] Test accuracy
- [ ] Documentation

**Milestone**: ✅ AI-powered insights

---

### Phase 6: Mobile App (Weeks 27-34)

#### Weeks 27-28: App Foundation
- [ ] Set up React Native project
- [ ] Create navigation structure
- [ ] Implement authentication
- [ ] Design app screens

#### Weeks 29-31: Core Features
- [ ] Real-time monitoring screens
- [ ] Batch management
- [ ] Alerts and notifications
- [ ] Settings and profile

#### Weeks 32-33: Testing
- [ ] iOS testing
- [ ] Android testing
- [ ] Performance optimization
- [ ] Bug fixes

#### Week 34: Launch
- [ ] App Store submission
- [ ] Play Store submission
- [ ] Marketing materials
- [ ] User documentation

**Milestone**: ✅ Mobile apps launched

---

## 🛠️ Technical Implementation Details

### 1. Multi-Tenant Database Schema

```sql
-- Step 1: Add Organizations table
CREATE TABLE organizations (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    name VARCHAR(255) NOT NULL,
    subdomain VARCHAR(100) UNIQUE,
    subscription_tier VARCHAR(50) DEFAULT 'starter',
    subscription_status VARCHAR(50) DEFAULT 'trial',
    max_zones INTEGER DEFAULT 1,
    billing_email VARCHAR(255),
    razorpay_customer_id VARCHAR(255),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Step 2: Add organization_id to existing tables
ALTER TABLE users ADD COLUMN organization_id UUID REFERENCES organizations(id);
ALTER TABLE farms ADD COLUMN organization_id UUID REFERENCES organizations(id);
ALTER TABLE zones ADD COLUMN organization_id UUID REFERENCES organizations(id);
ALTER TABLE devices ADD COLUMN organization_id UUID REFERENCES organizations(id);

-- Step 3: Create indexes
CREATE INDEX idx_users_org ON users(organization_id);
CREATE INDEX idx_farms_org ON farms(organization_id);
CREATE INDEX idx_zones_org ON zones(organization_id);

-- Step 4: Enable Row-Level Security
ALTER TABLE users ENABLE ROW LEVEL SECURITY;
ALTER TABLE farms ENABLE ROW LEVEL SECURITY;
ALTER TABLE zones ENABLE ROW LEVEL SECURITY;

-- Step 5: Create RLS policies
CREATE POLICY tenant_isolation ON users
    USING (organization_id = current_setting('app.current_org')::UUID);

CREATE POLICY tenant_isolation ON farms
    USING (organization_id = current_setting('app.current_org')::UUID);
```

### 2. Subscription Models

```javascript
// backend/src/models/Organization.js
const Organization = sequelize.define('Organization', {
  id: { type: DataTypes.UUID, primaryKey: true },
  name: DataTypes.STRING,
  subscriptionTier: {
    type: DataTypes.ENUM('starter', 'growth', 'enterprise'),
    defaultValue: 'starter'
  },
  subscriptionStatus: {
    type: DataTypes.ENUM('trial', 'active', 'suspended', 'cancelled'),
    defaultValue: 'trial'
  },
  maxZones: { type: DataTypes.INTEGER, defaultValue: 1 },
  razorpayCustomerId: DataTypes.STRING,
  razorpaySubscriptionId: DataTypes.STRING
});

// backend/src/models/Subscription.js
const Subscription = sequelize.define('Subscription', {
  id: { type: DataTypes.UUID, primaryKey: true },
  organizationId: DataTypes.UUID,
  planType: DataTypes.ENUM('starter', 'growth', 'enterprise'),
  status: DataTypes.ENUM('active', 'cancelled', 'past_due'),
  currentPeriodStart: DataTypes.DATE,
  currentPeriodEnd: DataTypes.DATE,
  cancelAtPeriodEnd: DataTypes.BOOLEAN
});
```

### 3. MQTT Topic Structure

```javascript
// Old (Single-tenant):
cropwise/{deviceId}/telemetry

// New (Multi-tenant):
yfcloud/{customerId}/{farmId}/{zoneId}/telemetry

// Example:
yfcloud/org_abc123/farm_xyz789/zone_a1/telemetry
yfcloud/org_abc123/farm_xyz789/zone_a1/status
yfcloud/org_abc123/farm_xyz789/zone_a1/command

// MQTT ACL (EMQX format):
{allow, {user, "org_abc123_device_01"}, publish, ["yfcloud/org_abc123/#"]}.
{allow, {user, "org_abc123_device_01"}, subscribe, ["yfcloud/org_abc123/#"]}.
{deny, all}.
```

### 4. Middleware Updates

```javascript
// backend/src/middleware/tenantContext.js
const tenantContext = async (req, res, next) => {
  // Extract organization from JWT token
  const organizationId = req.user.organizationId;
  
  // Set tenant context for database queries
  await sequelize.query(
    `SET LOCAL app.current_org = '${organizationId}'`
  );
  
  req.organizationId = organizationId;
  next();
};

// backend/src/middleware/subscriptionCheck.js
const checkSubscriptionLimits = async (req, res, next) => {
  const org = await Organization.findByPk(req.organizationId);
  
  if (org.subscriptionStatus !== 'active') {
    return res.status(403).json({
      error: 'Subscription inactive',
      message: 'Please renew your subscription'
    });
  }
  
  // Check zone limits
  const zoneCount = await Zone.count({
    where: { organizationId: req.organizationId }
  });
  
  if (zoneCount >= org.maxZones) {
    return res.status(403).json({
      error: 'Zone limit reached',
      message: `Your plan allows ${org.maxZones} zones. Upgrade to add more.`
    });
  }
  
  next();
};
```

---

## 🧪 Testing Strategy

### Unit Tests
```javascript
describe('Multi-tenant isolation', () => {
  it('should not allow cross-tenant data access', async () => {
    const org1User = createUser({ organizationId: 'org1' });
    const org2Farm = createFarm({ organizationId: 'org2' });
    
    const result = await org1User.getFarms();
    expect(result).not.toContain(org2Farm);
  });
});

describe('Subscription limits', () => {
  it('should enforce zone limits per plan', async () => {
    const starterOrg = { maxZones: 1 };
    const response = await createZone(starterOrg, { count: 2 });
    expect(response.status).toBe(403);
  });
});
```

### Integration Tests
- Multi-tenant API access
- MQTT topic isolation
- Billing webhooks
- Payment processing
- Plan upgrades

### Load Tests
- 100 concurrent customers
- 1000 zones sending telemetry
- Database query performance
- MQTT message throughput

---

## 📋 Deployment Checklist

### Pre-Launch
- [ ] All tests passing
- [ ] Security audit complete
- [ ] Performance benchmarks met
- [ ] Documentation updated
- [ ] Support team trained
- [ ] Billing system tested
- [ ] Backup/restore tested

### Infrastructure
- [ ] Production database (PostgreSQL)
- [ ] InfluxDB Cloud account
- [ ] MQTT broker (EMQX Cloud)
- [ ] Redis for caching
- [ ] CDN for static assets
- [ ] SSL certificates
- [ ] Monitoring (Sentry, DataDog)

### Legal
- [ ] Terms of Service
- [ ] Privacy Policy
- [ ] Data Processing Agreement
- [ ] Service Level Agreement
- [ ] Refund policy

---

## 📈 Success Criteria

### Technical
- ✅ System uptime > 99.5%
- ✅ API response time < 200ms (p95)
- ✅ Zero data leakage between tenants
- ✅ MQTT delivery > 99.9%
- ✅ All security tests passed

### Business
- ✅ 10 pilot customers onboarded
- ✅ Payment processing working
- ✅ Customer satisfaction > 4/5
- ✅ Zero critical bugs in production
- ✅ Support response time < 4 hours

---

## 🆘 Rollback Plan

If critical issues arise:

1. **Stop new signups** - Pause marketing
2. **Assess impact** - How many customers affected?
3. **Communicate** - Email all customers immediately
4. **Fix or rollback** - Deploy hotfix or revert
5. **Monitor** - 24-hour monitoring period
6. **Post-mortem** - Document what went wrong

---

## 📞 Support During Migration

**Technical Lead**: Available 24×7 during launch week  
**Escalation Path**: Email → Phone → On-site  
**Documentation**: All changes documented in real-time  
**Customer Communication**: Weekly updates during migration

---

**Migration Plan Version**: 1.0  
**Last Updated**: November 12, 2025  
**Owner**: Yellow Flowers Technologies  
**Status**: Ready to begin Phase 2A

