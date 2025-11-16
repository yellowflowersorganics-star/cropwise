# 🔬 Quality Control System - Complete Guide

## Overview

The Quality Control (QC) system provides comprehensive inspection tracking, defect management, quality standards, and compliance reporting for mushroom production.

---

## 📊 Database Models

### 1. **QualityCheck**
Records individual quality inspections with detailed results.

**Key Fields:**
- `checkType`: harvest, pre-harvest, post-harvest, packaging, storage, substrate, environmental
- `overallGrade`: A+, A, B, C, Reject
- `passStatus`: pass, fail, conditional, pending
- `qualityScore`: 0-100 score
- `sampleSize`: Quantity inspected
- `defectCount`: Number of defects found
- `defectRate`: Percentage of defects
- `inspectorName`: Name of inspector
- `status`: submitted, under_review, approved, rejected

**Associations:**
- Belongs to User (owner, reviewer)
- Belongs to Zone, Batch, Harvest, Farm
- Has many Defects

### 2. **Defect**
Tracks individual defects found during quality checks.

**Key Fields:**
- `defectType`: discoloration, contamination, damage, undersized, oversized, deformity, spots, bruising, decay, pest
- `severity`: minor, major, critical
- `category`: appearance, size, color, texture, contamination, packaging
- `affectedQuantity`: Amount affected
- `rootCause`: Why it occurred
- `marketability`: saleable, discounted, reject, waste
- `immediateAction`: Actions taken
- `correctiveAction`: How to fix
- `preventiveAction`: How to prevent
- `actionStatus`: pending, in_progress, completed

**Associations:**
- Belongs to QualityCheck
- Belongs to User (owner, responsible person, verified by)

### 3. **QualityStandard**
Defines quality criteria, grading systems, and compliance standards.

**Key Fields:**
- `name`: Standard name
- `code`: Unique standard code (auto-generated)
- `standardType`: product, process, environmental, safety, hygiene, regulatory
- `category`: appearance, size, color, texture, contamination, packaging, labeling, weight
- `cropType`: Applicable crop type (optional)
- `criteria`: JSON object defining quality criteria
- `gradingSystem`: JSON object defining grading rules
- `version`: Version number
- `status`: draft, active, archived
- `isMandatory`: Whether standard is mandatory
- `isPublic`: Whether standard is visible to all users
- `complianceRate`: Percentage of compliance

**Associations:**
- Belongs to User (owner, approver)
- Belongs to Organization

---

## 🔌 Backend API

### **Quality Checks**

#### Get All Quality Checks
```
GET /api/quality/checks
Query Params: checkType, overallGrade, passStatus, status, zoneId, batchId, farmId, page, limit
Response: { checks: [], pagination: {} }
```

#### Get Quality Check by ID
```
GET /api/quality/checks/:id
Response: { check details with defects }
```

#### Create Quality Check
```
POST /api/quality/checks
Body: {
  checkType, category, farmId?, zoneId?, batchId?, standardId, sampleSize,
  samplingMethod, result, overallScore?, notes?, correctiveActions?
}
Response: { created check }
```

#### Update Quality Check
```
PUT /api/quality/checks/:id
Body: { fields to update }
Response: { updated check }
```

#### Delete Quality Check
```
DELETE /api/quality/checks/:id
Response: { message }
```

#### Review Quality Check
```
POST /api/quality/checks/:id/review
Body: { status: 'approved' | 'rejected', reviewNotes }
Response: { updated check }
```

### **Defects**

#### Get All Defects
```
GET /api/quality/defects
Query Params: qualityCheckId, defectType, severity, actionStatus, page, limit
Response: { defects: [], pagination: {} }
```

#### Add Defect to Quality Check
```
POST /api/quality/checks/:qualityCheckId/defects
Body: {
  type, severity, description, location?, affectedQuantity?,
  rootCause?, correctiveAction?
}
Response: { created defect }
```

#### Update Defect
```
PUT /api/quality/defects/:id
Body: { fields to update }
Response: { updated defect }
```

#### Delete Defect
```
DELETE /api/quality/defects/:id
Response: { message }
```

### **Analytics**

#### Get Quality Statistics
```
GET /api/quality/stats
Query Params: days (7, 30, 90, 365), zoneId?, farmId?
Response: {
  totalChecks, passedChecks, failedChecks, pendingChecks,
  totalDefects, criticalDefects, byStatus: [], byGrade: []
}
```

#### Get Defect Analysis
```
GET /api/quality/defects/analysis
Query Params: days, zoneId?, farmId?
Response: {
  byType: [], bySeverity: [], byCategory: [],
  topDefects: [], trendData: []
}
```

#### Get Compliance Report
```
GET /api/quality/compliance
Query Params: days, standardId?
Response: {
  overallCompliance, totalStandards, byStandard: [],
  complianceHistory: []
}
```

### **Quality Standards**

#### Get All Standards
```
GET /api/quality/standards
Query Params: standardType, category, status, cropType, isMandatory, page, limit
Response: { standards: [], pagination: {} }
```

#### Get Active Standards
```
GET /api/quality/standards/active
Query Params: cropType?
Response: [ active standards ]
```

#### Get Standard by ID
```
GET /api/quality/standards/:id
Response: { standard details }
```

#### Create Standard
```
POST /api/quality/standards
Body: {
  name, code?, standardType, category, cropType?, description?,
  status, isMandatory, isPublic
}
Response: { created standard }
```

#### Update Standard
```
PUT /api/quality/standards/:id
Body: { fields to update }
Response: { updated standard }
```

#### Delete Standard
```
DELETE /api/quality/standards/:id
Response: { message }
```

#### Approve Standard
```
POST /api/quality/standards/:id/approve
Response: { approved standard }
```

#### Duplicate Standard
```
POST /api/quality/standards/:id/duplicate
Response: { duplicated standard }
```

---

## 💻 Frontend Pages

### 1. **Quality Dashboard** (`/quality`)
- Overview statistics (inspections, pass rate, defects, pending reviews)
- Pie chart: Inspection results distribution
- Bar chart: Defects by type
- Defects by severity breakdown
- Compliance report with progress bars
- Quick action cards

**Key Components:**
- Date range filter (7, 30, 90, 365 days)
- Real-time statistics
- Recharts visualizations
- Navigation to inspection and standards pages

### 2. **Quality Inspection** (`/quality/inspection`)
- New inspection form
- Farm/Zone/Batch selection with cascading dropdowns
- Quality standard selection (from active standards)
- Sample size and sampling method
- Overall score and result
- Notes and corrective actions
- Dynamic defect form
  - Add multiple defects
  - Each defect: type, severity, description, location, affected quantity, root cause, corrective action
- Photo upload support (ready for implementation)

**Features:**
- Form validation
- Auto-save drafts (ready for implementation)
- Real-time defect calculations
- Submit and save

### 3. **Quality Standards** (`/quality/standards`)
- List all quality standards
- Filter by type, category, status
- Create/Edit/Delete standards
- Approve draft standards
- Duplicate standards (for creating variants)
- Status badges (draft, active, archived)
- Mandatory and public indicators

**Standard Form:**
- Name, code (auto-generated)
- Standard type and category
- Crop type (optional)
- Description
- Status (draft, active, archived)
- Mandatory checkbox
- Public checkbox

---

## 🎯 Inspection Types

| Type | Description | When to Use |
|------|-------------|-------------|
| **Harvest** | Quality check during harvesting | At harvest time |
| **Pre-Harvest** | Inspection before harvest readiness | 1-2 days before harvest |
| **Post-Harvest** | Quality check after harvesting | After harvest, before packaging |
| **Packaging** | Packaging quality and standards | During packaging process |
| **Storage** | Storage conditions and product quality | Daily/weekly storage checks |
| **Substrate** | Substrate quality and contamination | Before inoculation |
| **Environmental** | Growing environment conditions | Daily/weekly |

---

## 📊 Grading System

### **Overall Grades**
- **A+**: Premium quality, no defects, exceeds standards
- **A**: Excellent quality, minor defects acceptable
- **B**: Good quality, some defects, saleable
- **C**: Acceptable quality, significant defects, discounted
- **Reject**: Does not meet minimum standards, not saleable

### **Pass Status**
- **Pass**: Meets all quality standards
- **Fail**: Does not meet quality standards
- **Conditional**: Meets some standards, requires follow-up
- **Pending**: Awaiting review/additional testing

---

## 🐛 Defect Types

### **Appearance**
- Discoloration
- Spots
- Bruising
- Deformity

### **Contamination**
- Bacterial
- Fungal
- Pest damage
- Foreign matter

### **Physical**
- Undersized
- Oversized
- Physical damage
- Broken/crushed

### **Quality**
- Texture issues
- Decay
- Moisture problems
- Off-odor

---

## 🚨 Severity Levels

| Severity | Description | Action Required |
|----------|-------------|-----------------|
| **Minor** | Cosmetic issues, still saleable | Monitor, no immediate action |
| **Major** | Affects quality, may reduce price | Corrective action recommended |
| **Critical** | Safety/quality risk, not saleable | Immediate action required |

---

## 📈 Compliance Tracking

The system automatically tracks:
- **Overall compliance rate**: Percentage of checks passing standards
- **Standard-specific compliance**: Compliance rate per quality standard
- **Trend analysis**: Compliance over time
- **Non-compliance alerts**: Automatic alerts for failed checks

---

## 🔄 Complete Workflow

### **1. Create Quality Standard**
1. Go to `/quality/standards`
2. Click "New Standard"
3. Fill in standard details:
   - Name: "Premium Oyster Mushroom Standard"
   - Type: Product
   - Category: Appearance
   - Crop Type: Oyster Mushroom
   - Status: Draft
   - Mandatory: Yes
4. Save standard
5. Approve standard to make it active

### **2. Perform Quality Inspection**
1. Go to `/quality/inspection`
2. Select inspection type (e.g., Harvest)
3. Select farm, zone, and batch
4. Select applicable quality standard
5. Enter sample size and method
6. Perform inspection and enter overall score
7. Add defects if found:
   - Type, severity, description
   - Affected quantity
   - Root cause and corrective action
8. Enter notes and recommendations
9. Submit inspection

### **3. Review and Analyze**
1. Go to `/quality` dashboard
2. View statistics and charts
3. Filter by date range
4. Identify trends and issues
5. Review defect analysis
6. Check compliance report
7. Take corrective actions as needed

---

## 🎯 Best Practices

### **Inspection Frequency**
- **Pre-Harvest**: Daily for last 3 days
- **Harvest**: Every batch
- **Post-Harvest**: Every batch
- **Storage**: Daily or weekly
- **Packaging**: Spot checks (10% sample)

### **Sample Sizes**
- **Small batches (<5kg)**: 100% inspection
- **Medium batches (5-20kg)**: 30-50% sample
- **Large batches (>20kg)**: 10-20% sample
- Use random or systematic sampling

### **Documentation**
- Take photos of defects
- Record environmental conditions
- Document corrective actions
- Track recurring issues
- Maintain inspection logs

### **Follow-Up**
- Set follow-up dates for conditional passes
- Verify corrective actions
- Re-inspect after corrective measures
- Track improvement over time

---

## 📊 Reporting

### **Available Reports**
1. **Quality Statistics**: Overall inspection metrics
2. **Defect Analysis**: Defect patterns and trends
3. **Compliance Report**: Standards compliance tracking
4. **Batch Quality**: Quality performance by batch
5. **Zone Performance**: Quality metrics by zone
6. **Trend Analysis**: Quality trends over time

### **Export Options**
(Ready for implementation)
- PDF reports
- Excel spreadsheets
- CSV data export
- Chart images

---

## 🔐 Access Control

### **Roles & Permissions**
- **Inspector**: Can create and submit inspections
- **Reviewer**: Can review and approve inspections
- **Admin**: Full access to all quality features
- **Farmer**: View own farm's quality data

### **Data Isolation**
- Users see only their own quality data
- Organization-level data sharing (optional)
- Public standards visible to all users

---

## 🚀 Advanced Features (Ready for Enhancement)

### **Photo/Video Upload**
- Defect documentation
- Before/after comparisons
- Training materials

### **Mobile App**
- On-site inspections
- Offline data entry
- Photo capture
- GPS location tagging

### **AI/ML Integration**
- Automated defect detection (computer vision)
- Quality prediction
- Trend forecasting
- Root cause analysis

### **Integrations**
- Export to ERP systems
- Certification body reporting
- Customer quality reports
- Supply chain traceability

---

## 📚 Example Use Case

**Scenario**: Oyster Mushroom Harvest Inspection

1. **Create Standard**
   - Name: "Premium Oyster Mushroom - Grade A"
   - Criteria: Size 5-10cm, No spots, Fresh appearance
   - Grading: A+ (perfect), A (minor defects), B (saleable), Reject

2. **Perform Inspection**
   - Zone: Zone A1
   - Batch: A1-20241113-001
   - Sample: 5kg out of 50kg harvest
   - Method: Random sampling
   - Result: Overall Score 87/100

3. **Record Defects**
   - Defect 1:
     - Type: Discoloration
     - Severity: Minor
     - Quantity: 0.2kg
     - Location: Cap edges
     - Root Cause: Excess humidity in final stage
     - Action: Adjust humidity for next batch

4. **Analysis**
   - 96% pass rate for the week
   - Humidity-related defects trending up
   - Corrective action: Recalibrate humidity sensors

---

## 💡 Tips for Success

1. **Train Inspectors**: Ensure consistent grading
2. **Use Photos**: Visual documentation helps
3. **Track Trends**: Monitor patterns over time
4. **Act Quickly**: Address issues immediately
5. **Continuous Improvement**: Use data to improve processes
6. **Customer Feedback**: Integrate customer quality feedback
7. **Regular Audits**: Verify inspection accuracy
8. **Standard Updates**: Keep standards current

---

## 🎉 System Status

**✅ Fully Implemented:**
- Complete database models
- Full CRUD API endpoints
- Quality Dashboard with analytics
- Quality Inspection form
- Quality Standards management
- Defect tracking
- Statistics and charts
- Compliance reporting
- Navigation integration

**🚀 Ready for Enhancement:**
- Photo/video upload
- Mobile app integration
- Export/PDF reports
- Email notifications for failed checks
- Automated compliance alerts
- Customer portal for quality reports

---

## 📞 Support

For questions or issues with the Quality Control system:
- Check the documentation above
- Review API responses for error details
- Consult backend logs for debugging
- Refer to frontend console for client-side errors

---

**Quality Control is now fully operational in CropWise! 🎊**

Start by creating quality standards, then perform inspections, and analyze your quality metrics! 🔬📊

