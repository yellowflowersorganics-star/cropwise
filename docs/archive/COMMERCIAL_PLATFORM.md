# 🌼 Yellow Flowers SmartFarm Cloud
## Commercial SaaS Platform Architecture & Business Model

**Tagline**: *"Automate, Monitor, and Grow — from One Dashboard."*

---

## 📊 Platform Evolution

### From CropWise → SmartFarm Cloud

| Component | CropWise (Current) | SmartFarm Cloud (Target) | Status |
|:----------|:----------------------|:-------------------------|:-------|
| **Architecture** | Single-tenant | Multi-tenant SaaS | 🔄 Upgrade needed |
| **Authentication** | ✅ JWT-based | ✅ JWT + OAuth2 | ✅ Foundation ready |
| **Data Storage** | PostgreSQL | PostgreSQL + InfluxDB | 🔄 Add time-series |
| **MQTT Security** | Basic auth | TLS + per-tenant isolation | 🔄 Add TLS |
| **Billing** | None | Razorpay/Stripe integration | 🆕 Add module |
| **Multi-zone** | ✅ Supported | ✅ Per-customer zones | ✅ Architecture ready |
| **Dashboard** | ✅ React | ✅ Multi-tenant React | 🔄 Add tenant context |
| **Analytics** | Basic (stub) | AI/ML predictions | 🆕 Build module |
| **Mobile App** | None | React Native | 🆕 New platform |
| **Hardware** | DIY | Branded YF SmartGrow | 🆕 Product design |

---

## 🏗️ Commercial Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    CUSTOMER LAYER (100+ Farms)                  │
│  Each Customer: Organization → Multiple Farms → Multiple Zones  │
└────────────────────────┬────────────────────────────────────────┘
                         │
         ┌───────────────┴────────────────┐
         │                                │
    ┌────▼────┐                     ┌────▼────┐
    │ Farm A  │                     │ Farm B  │
    │ Pune    │                     │ Mumbai  │
    └────┬────┘                     └────┬────┘
         │                                │
         └─────────┬──────────────────────┘
                   │
         ┌─────────▼─────────────────────────────────┐
         │     EDGE LAYER (Per Farm)                 │
         │  ┌─────────────────────────────────┐      │
         │  │  Raspberry Pi / Edge Gateway    │      │
         │  │  - Node-RED Runtime             │      │
         │  │  - Local MQTT (Mosquitto)       │      │
         │  │  - TLS Bridge to Cloud          │      │
         │  │  - Farm ID: farm_<uuid>         │      │
         │  └─────────────────────────────────┘      │
         │           │                                │
         │     ┌─────┴─────┐                         │
         │     │           │                         │
         │  ┌──▼───┐   ┌──▼───┐   ┌──────┐         │
         │  │Zone 1│   │Zone 2│...│Zone N│         │
         │  │ESP32 │   │ESP32 │   │ESP32 │         │
         │  └──────┘   └──────┘   └──────┘         │
         └────────────────┬──────────────────────────┘
                          │ TLS/MQTT
         ┌────────────────▼──────────────────────────┐
         │          CLOUD LAYER (AWS/Azure)          │
         │  ┌──────────────────────────────────┐     │
         │  │  EMQX Cloud MQTT Broker          │     │
         │  │  Topic: yfcloud/<customer_id>/   │     │
         │  │         <farm_id>/<zone_id>/     │     │
         │  └──────────────┬───────────────────┘     │
         │                 │                          │
         │  ┌──────────────▼───────────────────┐     │
         │  │  SmartFarm API (Node.js/FastAPI)│     │
         │  │  - Multi-tenant routing         │     │
         │  │  - JWT Authentication           │     │
         │  │  - Subscription management      │     │
         │  │  - Billing (Razorpay/Stripe)   │     │
         │  └──────────┬──────────────────────┘     │
         │             │                             │
         │  ┌──────────┴────────────┐               │
         │  │                       │               │
         │  ▼                       ▼               │
         │  ┌──────────┐    ┌─────────────┐        │
         │  │InfluxDB  │    │ PostgreSQL  │        │
         │  │Time-     │    │ Users,      │        │
         │  │Series    │    │ Billing,    │        │
         │  │Telemetry │    │ Farms       │        │
         │  └──────────┘    └─────────────┘        │
         │             │                             │
         │  ┌──────────▼───────────────────┐        │
         │  │  Analytics & ML Engine       │        │
         │  │  - Yield predictions         │        │
         │  │  - Energy optimization       │        │
         │  │  - Labour forecasting        │        │
         │  └──────────────────────────────┘        │
         └──────────────────┬───────────────────────┘
                            │
         ┌──────────────────▼───────────────────────┐
         │     USER INTERFACES                      │
         │  ┌────────────┐  ┌────────────┐         │
         │  │ Web Portal │  │ Mobile App │         │
         │  │ React +    │  │ React      │         │
         │  │ Tailwind   │  │ Native     │         │
         │  └────────────┘  └────────────┘         │
         └──────────────────────────────────────────┘
```

---

## 💼 Subscription Tiers

### 🥉 Tier 1: Starter (₹1,500/month)
**Target**: Small farmers, 1 growing room

- ✅ **1 Zone** (up to 1,000 bags)
- ✅ Dashboard + Auto Control
- ✅ Mobile alerts (Telegram/WhatsApp)
- ✅ 15-day incubation timer automation
- ✅ Local data logging only
- ❌ Cloud data sync
- ❌ Historical analytics
- ❌ AI predictions

**Included Hardware** (one-time ₹35,000):
- 1× ESP32 Zone Controller
- 1× SHT31 Sensor
- 1× MH-Z19C CO₂ Sensor
- 1× 4-Channel Relay
- Basic wiring & enclosure

---

### 🥈 Tier 2: Growth (₹3,000/month)
**Target**: Medium farms, multiple rooms

- ✅ **Up to 5 Zones**
- ✅ Cloud data sync (InfluxDB)
- ✅ Web + Mobile Dashboard
- ✅ Monthly performance reports
- ✅ Labour/Yield tracking
- ✅ OTA firmware updates
- ✅ 30-day data retention
- ✅ Email alerts
- ❌ AI insights
- ❌ API access

**Hardware Bundle** (₹1,50,000):
- 5× Zone Controllers
- 1× Raspberry Pi Gateway
- Cloud connectivity setup
- Installation support

---

### 🥇 Tier 3: Enterprise (₹5,000–₹7,000/month)
**Target**: Commercial farms, 10+ zones

- ✅ **Up to 10 Zones** (more on request)
- ✅ Advanced AI insights (Yield & Energy prediction)
- ✅ Remote control access via cloud
- ✅ Data export + API integration
- ✅ 24×7 Support + Custom dashboards
- ✅ White-label option
- ✅ Unlimited data retention
- ✅ Priority OTA updates
- ✅ Custom crop recipes
- ✅ Labour optimization reports

**Hardware Package** (₹3,00,000):
- 10× Premium Zone Controllers
- 2× Raspberry Pi Gateways (redundant)
- Professional installation
- On-site training
- 1-year AMC included

---

## 💰 Business Economics

### Revenue Model

```
Hardware Revenue (One-time):
  Starter: ₹35,000 × 1 zone
  Growth:  ₹1,50,000 × 5 zones
  Enterprise: ₹3,00,000 × 10 zones

Subscription Revenue (Monthly):
  Starter: ₹1,500/month
  Growth:  ₹3,000/month
  Enterprise: ₹5,000–₹7,000/month

Support & AMC (Annual):
  10–15% of hardware cost
  Growth: ₹15,000/year
  Enterprise: ₹30,000/year
```

### Cost Structure (per customer/month)

```
Cloud Hosting:
  MQTT (EMQX): ₹50
  InfluxDB Cloud: ₹80
  API Server: ₹40
  Total: ~₹170/customer/month

Support:
  Email support: Included
  24×7 (Enterprise): Dedicated team

Net Margin:
  Starter: ₹1,500 - ₹170 = ₹1,330 (89%)
  Growth: ₹3,000 - ₹170 = ₹2,830 (94%)
  Enterprise: ₹6,000 - ₹250 = ₹5,750 (96%)
```

### Projected Revenue (100 Customers)

```
Customer Mix:
  50 Starter @ ₹1,500 = ₹75,000/month
  30 Growth @ ₹3,000 = ₹90,000/month
  20 Enterprise @ ₹6,000 = ₹1,20,000/month
  
Monthly Recurring Revenue: ₹2,85,000
Annual MRR: ₹34,20,000

Cloud Costs: ₹25,000/month (₹3,00,000/year)
Net Revenue: ₹31,20,000/year

+ Hardware Sales: ₹50,00,000 first year
Total Year 1: ₹81,20,000
```

---

## 🔐 Multi-Tenant Security Architecture

### Customer Isolation

```
Database Level:
  PostgreSQL: customer_id in every table
  InfluxDB: customer_id as tag
  Row-Level Security (RLS) policies

MQTT Topics:
  yfcloud/<customer_id>/<farm_id>/<zone_id>/telemetry
  yfcloud/<customer_id>/<farm_id>/<zone_id>/command
  
  ACL Rules:
  - Customer A cannot subscribe to Customer B topics
  - Edge devices get credentials per farm

API Authentication:
  JWT Token includes:
    - user_id
    - customer_id (organization)
    - role (admin, operator, viewer)
    - permissions array
  
  Middleware validates:
    - Token signature
    - Token expiry
    - Customer context
    - Resource ownership
```

### Security Layers

```
Layer 1: TLS/SSL
  - All MQTT traffic encrypted (TLS 1.2+)
  - HTTPS only for API
  - Certificate rotation every 90 days

Layer 2: Authentication
  - JWT tokens (15-minute expiry)
  - Refresh tokens (7-day expiry)
  - Device credentials (MQTT username/password)
  - Per-farm API keys

Layer 3: Authorization
  - Role-Based Access Control (RBAC)
  - Roles: SuperAdmin, OrgAdmin, FarmManager, Operator, Viewer
  - Permission checks on every API call

Layer 4: Data Encryption
  - Passwords: bcrypt (12 rounds)
  - Sensitive data: AES-256
  - Database: Encryption at rest

Layer 5: Audit Logging
  - All user actions logged
  - API request logs
  - Device connection logs
  - Billing events
  - Retention: 2 years
```

---

## 📊 Dashboard Features by Tier

### Starter Dashboard
```
Home Page:
  ✓ Live Temperature/Humidity/CO₂ graph
  ✓ Current zone status
  ✓ Incubation timer
  ✓ Basic alerts
  
Batch Management:
  ✓ Start incubation
  ✓ Manual harvest record
  
Settings:
  ✓ Edit thresholds
  ✓ Profile settings
```

### Growth Dashboard
```
All Starter features +

Analytics Page:
  ✓ Yield vs Zone Chart (30 days)
  ✓ Labour Hours tracking
  ✓ Average energy consumption
  ✓ Alerts summary
  
Batch Management:
  ✓ Multi-zone orchestration
  ✓ Batch comparison
  ✓ Historical batches
  
Reports:
  ✓ Monthly performance PDF
  ✓ CSV data export
```

### Enterprise Dashboard
```
All Growth features +

AI Insights:
  ✓ Next flush yield prediction
  ✓ Optimal harvest day
  ✓ Energy optimization suggestions
  ✓ Labour forecasting
  
Advanced Analytics:
  ✓ Custom date ranges
  ✓ Multi-farm comparison
  ✓ Crop recipe optimization
  ✓ Cost per kg analysis
  
Integrations:
  ✓ REST API access
  ✓ Webhook notifications
  ✓ Data warehouse export
  
Management:
  ✓ User management
  ✓ Role assignments
  ✓ Custom dashboards
  ✓ White-label branding
```

---

## 🗄️ Database Schema (Multi-Tenant)

### PostgreSQL Tables

```sql
-- Organizations (Customers)
organizations
  - id (uuid, PK)
  - name
  - subscription_tier (starter/growth/enterprise)
  - subscription_status (active/suspended/cancelled)
  - billing_email
  - created_at

-- Users (linked to organizations)
users
  - id (uuid, PK)
  - organization_id (FK)
  - email
  - role (admin/manager/operator/viewer)
  - created_at

-- Farms (one org can have multiple farms)
farms
  - id (uuid, PK)
  - organization_id (FK)
  - name
  - location
  - created_at

-- Zones (multi-tenant)
zones
  - id (uuid, PK)
  - farm_id (FK)
  - organization_id (FK) -- redundant for quick filtering
  - name
  - active_recipe_id
  - current_stage
  - batch_start_date

-- Devices (multi-tenant)
devices
  - id (uuid, PK)
  - organization_id (FK)
  - farm_id (FK)
  - zone_id (FK)
  - device_id (ESP32 MAC)
  - mqtt_credentials (encrypted)
  - last_seen

-- Subscriptions & Billing
subscriptions
  - id (uuid, PK)
  - organization_id (FK)
  - plan_type
  - status
  - current_period_start
  - current_period_end
  - razorpay_subscription_id
  
invoices
  - id (uuid, PK)
  - organization_id (FK)
  - amount
  - status (paid/pending/failed)
  - razorpay_invoice_id
  - created_at
```

### InfluxDB Measurements

```
Measurement: telemetry
  Tags:
    - customer_id
    - farm_id
    - zone_id
    - device_id
  Fields:
    - temperature
    - humidity
    - co2
    - light_level
  Timestamp: nanosecond precision

Measurement: actuator_states
  Tags:
    - customer_id
    - farm_id
    - zone_id
  Fields:
    - fan_state (boolean)
    - humidifier_state (boolean)
    - heater_state (boolean)
  Timestamp

Measurement: batch_events
  Tags:
    - customer_id
    - farm_id
    - zone_id
    - batch_id
  Fields:
    - event_type (start/harvest/flush)
    - yield_kg
    - labour_hours
  Timestamp
```

---

## 🔄 MQTT Topic Structure (Multi-Tenant)

```
Topic Namespace:
  yfcloud/<customer_id>/<farm_id>/<zone_id>/<message_type>

Examples:
  yfcloud/org_123abc/farm_xyz/zone_a1/telemetry
  yfcloud/org_123abc/farm_xyz/zone_a1/status
  yfcloud/org_123abc/farm_xyz/zone_a1/setpoints
  yfcloud/org_123abc/farm_xyz/zone_a1/command

ACL Rules (EMQX):
  Customer org_123abc can only:
    - Subscribe: yfcloud/org_123abc/#
    - Publish: yfcloud/org_123abc/#
  
  Devices get credentials limited to:
    - Publish: yfcloud/org_123abc/farm_xyz/zone_a1/telemetry
    - Subscribe: yfcloud/org_123abc/farm_xyz/zone_a1/command
```

---

## 🚀 Technical Implementation Phases

### Phase 2A: Multi-Tenancy Foundation (2 months)
**Goal**: Add customer isolation to current CropWise

**Tasks**:
- [ ] Add `organization_id` to all models
- [ ] Implement Row-Level Security (RLS)
- [ ] Update API middleware for tenant context
- [ ] Modify MQTT topics with customer prefix
- [ ] Add organization management UI
- [ ] Create tenant onboarding flow

**Deliverable**: Multi-tenant backend ready

---

### Phase 2B: Billing Integration (1 month)
**Goal**: Implement subscription and payment processing

**Tasks**:
- [ ] Integrate Razorpay SDK
- [ ] Create subscription plans
- [ ] Build billing dashboard
- [ ] Implement webhook handlers
- [ ] Add payment success/failure flows
- [ ] Create invoice generation
- [ ] Add subscription status checks

**Deliverable**: Automated billing system

---

### Phase 3: Time-Series Database (1 month)
**Goal**: Add InfluxDB for high-volume telemetry

**Tasks**:
- [ ] Deploy InfluxDB Cloud
- [ ] Create retention policies
- [ ] Migrate telemetry storage
- [ ] Build historical query APIs
- [ ] Add data aggregation
- [ ] Create backup strategy

**Deliverable**: Scalable telemetry storage

---

### Phase 4: Enhanced Security (2 weeks)
**Goal**: Production-grade security

**Tasks**:
- [ ] Enable TLS for MQTT
- [ ] Implement MQTT ACLs
- [ ] Add rate limiting
- [ ] Implement audit logging
- [ ] Add API key management
- [ ] Security testing & penetration test

**Deliverable**: Enterprise-ready security

---

### Phase 5: Advanced Analytics (2 months)
**Goal**: AI-powered insights

**Tasks**:
- [ ] Collect 6 months of production data
- [ ] Build yield prediction model
- [ ] Create energy optimization model
- [ ] Add labour forecasting
- [ ] Build recommendation engine
- [ ] Create ML API endpoints
- [ ] Add predictions to dashboard

**Deliverable**: AI-powered platform

---

### Phase 6: Mobile App (2 months)
**Goal**: Native mobile experience

**Tasks**:
- [ ] React Native app structure
- [ ] Authentication flow
- [ ] Real-time monitoring screens
- [ ] Push notifications
- [ ] Offline mode
- [ ] App Store submission

**Deliverable**: iOS & Android apps

---

## 🏭 Hardware: YF SmartGrow Controller

### Product Specifications

```
Product Name: YF SmartGrow Zone Controller
Model: YF-ZC-001

Specifications:
  - MCU: ESP32-WROOM-32
  - Sensors: SHT31 (Temp/RH), MH-Z19C (CO₂)
  - Outputs: 4× Relay (10A), 2× PWM
  - Inputs: 4× Digital, 2× Analog
  - Connectivity: WiFi 2.4GHz, MQTT
  - Power: 12V DC, 2A
  - Enclosure: IP65 ABS plastic
  - Display: 2.4" TFT LCD (optional)
  - Dimensions: 150×100×50mm
  - Mounting: DIN rail or wall mount

Features:
  - Pre-configured WiFi (QR code setup)
  - OTA firmware updates
  - Local OLED status display
  - Manual override buttons
  - Status LEDs (Power, WiFi, MQTT)
  - Reset button

Certifications Required:
  - CE (Europe)
  - FCC (US)
  - BIS (India)
```

### Branded Packaging
```
Box Contents:
  ✓ YF SmartGrow Controller
  ✓ SHT31 Sensor with cable
  ✓ MH-Z19C CO₂ Sensor
  ✓ 12V Power adapter
  ✓ Quick start guide
  ✓ QR code for dashboard
  ✓ Mounting hardware
  ✓ 1-year warranty card

Box Design:
  - Yellow Flowers logo
  - Product photo
  - Key features
  - "Cloud-connected precision control for mushrooms"
  - Support: support@yellowflowers.tech
```

---

## 📱 Marketing & Sales Strategy

### Target Markets

**Primary**: Mushroom farmers (Oyster, Button, Shiitake)
- Market size: 10,000+ farms in India
- Pain points: Manual control, inconsistent yields
- Value prop: 30% yield increase, 50% labour reduction

**Secondary**: Vertical farms, greenhouses
- Expanding to lettuce, tomatoes, herbs
- Higher-value crops, premium pricing

### Sales Channels

1. **Direct Sales** (B2B)
   - Field sales team
   - Farm visits & demos
   - Trade shows & exhibitions

2. **Channel Partners**
   - Agriculture equipment dealers
   - Mushroom substrate suppliers
   - Farming cooperatives

3. **Online**
   - Website with demo videos
   - YouTube channel
   - WhatsApp Business

### Pricing Strategy

```
Hardware:
  List Price: ₹45,000/zone
  Channel Partner: ₹35,000 (22% margin)
  Volume Discount: 5+ zones = 15% off

Subscription:
  Annual Prepay: 10% discount
  3-year contract: 20% discount
  Enterprise: Custom pricing

Bundled Offers:
  "Starter Kit": Hardware + 6 months subscription = ₹50,000
  "Farm Bundle": 5 zones + 1 year Growth = ₹1,75,000
```

---

## 🎓 Training & Support

### Customer Onboarding

**Week 1: Installation**
- Hardware delivery
- On-site installation by technician
- WiFi configuration
- Account creation

**Week 2: Training**
- 2-hour dashboard training (online/on-site)
- Incubation process setup
- Alert configuration
- Mobile app walkthrough

**Week 3: Monitoring**
- Daily check-ins via WhatsApp
- Issue resolution
- Fine-tuning thresholds

**Week 4: Handover**
- Final review
- Documentation handover
- Ongoing support activation

### Support Tiers

**Starter**: Email support (24-hour response)
**Growth**: Email + Phone (4-hour response)
**Enterprise**: 24×7 dedicated support + on-site visits

---

## 📊 Success Metrics

### Customer Metrics
- Yield increase: Target 25-30%
- Labour reduction: Target 40-50%
- Energy savings: Target 15-20%
- Customer satisfaction: Target NPS > 50

### Business Metrics
- Monthly Recurring Revenue (MRR)
- Customer Acquisition Cost (CAC)
- Lifetime Value (LTV)
- Churn rate: Target < 5%
- Net Promoter Score (NPS)

### Technical Metrics
- System uptime: Target 99.5%
- Data accuracy: > 98%
- Alert response time: < 5 seconds
- MQTT message delivery: > 99.9%

---

## 🔮 Future Roadmap (12-24 months)

### Q1-Q2 2026: Enhanced Platform
- [ ] Recipe marketplace (farmers share optimized recipes)
- [ ] Community forum
- [ ] Video tutorials
- [ ] Certification program for operators

### Q3-Q4 2026: Ecosystem Expansion
- [ ] Integration with ERP systems
- [ ] Supply chain tracking
- [ ] Marketplace for inputs (substrate, spawn)
- [ ] B2B marketplace for produce

### 2027: International Expansion
- [ ] Launch in Southeast Asia
- [ ] Localization (Thai, Vietnamese, Indonesian)
- [ ] Regional cloud deployments
- [ ] Local partnerships

---

## 💼 Legal & Compliance

### Company Structure
```
Entity: Yellow Flowers Technologies Pvt Ltd
Registration: India (MCA)
GST: Required
MSME: Optional (for benefits)
```

### Intellectual Property
- **Trademark**: "Yellow Flowers SmartFarm Cloud" ™
- **Copyright**: Software code, documentation
- **Patent**: IoT controller logic (optional)
- **Trade Secret**: ML models, algorithms

### Compliance
- Data protection: GDPR (if EU customers), DPDPA (India)
- IoT device: BIS certification
- Cloud security: ISO 27001
- Payment: PCI DSS (via Razorpay)

---

## 📞 Contact & Support

**Yellow Flowers Technologies**
- Website: https://yellowflowers.tech
- Support: support@yellowflowers.tech
- Sales: sales@yellowflowers.tech
- Phone: +91-XXXXX-XXXXX

**Demo Request**: https://yellowflowers.tech/demo

---

**Document Version**: 1.0  
**Last Updated**: November 12, 2025  
**Owner**: Yellow Flowers Technologies

