# 👤 CropWise - User Guide

**Version:** 1.0.0  
**Last Updated:** November 2025

Welcome to CropWise! This guide will help you get started with managing your smart farm.

---

## 📋 Table of Contents

1. [Getting Started](#getting-started)
2. [Dashboard Overview](#dashboard-overview)
3. [Managing Farms](#managing-farms)
4. [Managing Zones](#managing-zones)
5. [Growing Recipes](#growing-recipes)
6. [Batch Management](#batch-management)
7. [Harvest Recording](#harvest-recording)
8. [Inventory Management](#inventory-management)
9. [Task Management](#task-management)
10. [Employee Management](#employee-management)
11. [Quality Control](#quality-control)
12. [Analytics & Reports](#analytics--reports)
13. [IoT Devices](#iot-devices)
14. [Notifications](#notifications)
15. [Settings](#settings)
16. [Mobile App](#mobile-app)
17. [FAQs](#faqs)

---

## 🚀 Getting Started

### Creating Your Account

1. **Visit** https://www.cropwise.io
2. **Click** "Sign Up" or "Get Started"
3. **Choose** sign-up method:
   - Email & Password
   - Google Account (recommended)
4. **Fill in** your details:
   - First Name
   - Last Name
   - Organization Name
   - Email Address
   - Password (if not using Google)
5. **Verify** your email address
6. **Complete** your profile

### First Login

1. **Go to** https://www.cropwise.io/login
2. **Sign in** with your credentials or Google
3. **Tour** the dashboard (first-time users)
4. **Complete** the setup wizard:
   - Create your first farm
   - Set up zones
   - Configure notifications

---

## 📊 Dashboard Overview

Your dashboard provides a real-time overview of your farming operation.

### Key Metrics

**Active Batches**
- Number of currently growing batches
- Click to view details

**Total Zones**
- Number of growing zones
- Active vs. maintenance status

**Monthly Yield**
- Total harvest in kg/lbs this month
- Comparison to previous month

**Monthly Revenue**
- Total sales this month
- Profit margins

**Quality Score**
- Average quality rating across batches
- Grade distribution (A, B, C)

**Pending Tasks**
- Number of tasks due today
- Overdue tasks highlighted

### Quick Actions

- ➕ **Start New Batch**: Begin a new growing cycle
- 📦 **Record Harvest**: Log harvest yields
- ✅ **Complete Task**: Mark tasks as done
- 📋 **Add Inventory**: Update stock levels
- 🔔 **View Alerts**: Check notifications

### Recent Activity

- Latest harvests
- Recent task completions
- Inventory transactions
- System alerts

---

## 🏡 Managing Farms

### Creating a Farm

1. **Navigate to** Farms → Add New Farm
2. **Enter details**:
   - **Name**: e.g., "North Greenhouse"
   - **Type**: Indoor, Outdoor, Greenhouse, Vertical
   - **Area**: Size of your farm
   - **Unit**: sq ft, sq m, acres
   - **Location**: Address or GPS coordinates
   - **Description**: Optional notes
3. **Click** Save

### Viewing Farm Details

1. **Click** on any farm card
2. **View**:
   - Total zones
   - Active batches
   - Historical yield data
   - Revenue statistics
3. **Tabs**:
   - Overview
   - Zones
   - Batches
   - Analytics

### Editing a Farm

1. **Go to** Farm Details
2. **Click** Edit button (pencil icon)
3. **Update** information
4. **Save** changes

### Deleting a Farm

⚠️ **Warning**: This cannot be undone!

1. **Go to** Farm Details
2. **Click** Delete button (trash icon)
3. **Confirm** deletion
4. **Note**: Cannot delete if zones exist

---

## 📍 Managing Zones

Zones are growing areas within your farm (e.g., grow rooms, shelves, sections).

### Creating a Zone

1. **Navigate to** Zones → Add New Zone
2. **Select** parent farm
3. **Enter details**:
   - **Name**: e.g., "Zone A - Incubation"
   - **Area**: Size in sq ft/m
   - **Environment Type**: Controlled, Semi-controlled, Outdoor
   - **Max Capacity**: Optional (in kg or units)
   - **Description**: Optional notes
4. **Click** Save

### Zone Status

- **🟢 Active**: Currently in use
- **🟡 Idle**: Not in use, ready to start
- **🔴 Maintenance**: Under maintenance
- **⚪ Inactive**: Temporarily disabled

### Changing Zone Status

1. **Go to** Zone Details
2. **Click** Status dropdown
3. **Select** new status
4. **Add** notes (optional)
5. **Save**

### Zone Environmental Monitoring

**Real-time Data** (if IoT sensors connected):
- 🌡️ Temperature
- 💧 Humidity
- 🌫️ CO₂ Levels
- 💡 Light Intensity

**Charts**:
- 24-hour history
- 7-day trends
- Alert thresholds

---

## 📖 Growing Recipes

Recipes define the environmental conditions and stages for growing specific crops.

### Creating a Recipe

1. **Navigate to** Recipes → Add New Recipe
2. **Enter basic info**:
   - **Name**: e.g., "Oyster Mushroom - Standard"
   - **Crop Type**: Oyster Mushroom, Shiitake, Lettuce, etc.
   - **Description**: Growing method details
3. **Add Stages**:

#### Example Stage: Incubation
- **Name**: Incubation
- **Duration**: 15 days
- **Temperature**: 20-24°C (68-75°F)
- **Humidity**: 85-95%
- **CO₂**: <5000 ppm
- **Light**: 0 hours/day
- **Instructions**: "Maintain high humidity, no light"

#### Example Stage: Fruiting
- **Name**: Fruiting
- **Duration**: 10 days
- **Temperature**: 16-20°C (61-68°F)
- **Humidity**: 90-95%
- **CO₂**: <1000 ppm
- **Light**: 12 hours/day
- **Instructions**: "Reduce temperature, introduce light"

4. **Add more stages** as needed
5. **Save** recipe

### Using Templates

Pre-built recipes available:
- Oyster Mushroom (Standard)
- Shiitake Mushroom (Premium)
- King Oyster Mushroom
- Lion's Mane Mushroom
- Button Mushroom
- Lettuce (Hydroponic)
- Herbs (Basil, Cilantro)

**To use**:
1. **Browse** Recipe Templates
2. **Click** "Use Template"
3. **Customize** as needed
4. **Save** as your own

---

## 📦 Batch Management

Batches represent individual growing cycles from start to harvest.

### Starting a New Batch

1. **Navigate to** Batches → Start New Batch
2. **Select**:
   - **Zone**: Where you're growing
   - **Recipe**: Growing method to follow
3. **Enter details**:
   - **Substrate Type**: Straw, sawdust, coco coir, etc.
   - **Substrate Amount**: kg/lbs
   - **Spawn Amount**: kg/lbs
   - **Number of Units**: Bags, blocks, trays
   - **Start Date**: Usually today
4. **Add notes**: Optional
5. **Click** Start Batch

### Batch Status Tracking

**Status Lifecycle**:
1. **🟢 In Progress**: Currently growing
2. **⏸️ Paused**: Temporarily stopped
3. **✅ Completed**: Finished and harvested
4. **❌ Failed**: Did not produce results
5. **📦 Archived**: Stored for records

### Monitoring Batch Progress

**Overview Panel**:
- Days elapsed / Total duration
- Current stage
- Progress bar (%)
- Expected harvest date
- Environmental conditions

**Stage Transitions**:
- Automatic progression based on duration
- Manual approval option (for critical stages)
- Checklist for manual tasks

**Example Transition**:
```
Incubation (Day 15) → Awaiting Manager Approval
☐ Check for full colonization
☐ Make cuts on bags
☐ Move to fruiting chamber

[Approve] [Decline]
```

### Completing a Batch

1. **Go to** Batch Details
2. **Click** "Complete Batch"
3. **Enter final details**:
   - Total yield (calculated from harvests)
   - Quality notes
   - Success/failure reason
4. **Confirm** completion

---

## 🍄 Harvest Recording

Track yield, quality, and profitability for each harvest.

### Recording a Harvest

1. **Navigate to** Harvests → Record New Harvest
2. **Select** batch
3. **Enter details**:
   - **Flush Number**: 1, 2, 3, etc. (for mushrooms)
   - **Yield**: kg/lbs harvested
   - **Grade**: A (Premium), B (Standard), C (Budget)
   - **Harvest Date**: Usually today
   - **Notes**: Observations
4. **Add photos**: Optional (for quality documentation)
5. **Save**

### Understanding Flushes

**For Mushroom Farming**:
- **Flush 1**: First harvest (usually highest yield)
- **Flush 2**: Second harvest (7-10 days after first)
- **Flush 3**: Third harvest (diminishing returns)

**Yield Expectations**:
- Flush 1: 60-70% of total
- Flush 2: 25-30% of total
- Flush 3: 5-10% of total

### Quality Grading

| Grade | Description | Price Multiplier |
|-------|-------------|-----------------|
| **A** | Premium: uniform, no defects | 1.3x |
| **B** | Standard: good quality | 1.0x |
| **C** | Budget: minor defects, ok to sell | 0.7x |
| **D** | Waste: cannot sell | 0x |

### Bio-Efficiency Calculation

**Formula**: (Total Yield / Substrate Weight) × 100

**Example**:
- Substrate: 50 kg
- Total Yield: 120 kg
- Bio-Efficiency: (120/50) × 100 = **240%**

**Benchmarks**:
- Oyster Mushroom: 100-150%
- Shiitake: 80-120%
- King Oyster: 120-180%

---

## 📦 Inventory Management

Track substrate, spawn, supplies, and finished products.

### Adding Inventory Items

1. **Navigate to** Inventory → Add Item
2. **Enter details**:
   - **Name**: e.g., "Straw Substrate"
   - **Category**: Substrate, Spawn, Supplies, Packaging
   - **Current Quantity**: Amount in stock
   - **Unit**: kg, lbs, pieces, boxes
   - **Unit Cost**: Price per unit
   - **Low Stock Threshold**: Minimum before alert
   - **Supplier**: Vendor name
3. **Save**

### Recording Transactions

**Types**:
- **➕ Add**: New stock received
- **➖ Remove**: Stock used in production
- **✏️ Adjust**: Correction or audit

**Example - Using Substrate**:
1. **Go to** Inventory → Straw Substrate
2. **Click** "Record Transaction"
3. **Select** "Remove"
4. **Enter**:
   - Quantity: 50 kg
   - Reason: "Used for Batch B-2024-015"
   - Link to batch (optional)
5. **Save**

### Low Stock Alerts

**Automatic Notifications** when:
- Quantity drops below threshold
- Critical items (spawn, substrate)
- Email + in-app alert

**Managing Alerts**:
1. **Review** alerts dashboard
2. **Order** from supplier
3. **Mark** as "Ordered"
4. **Update** when received

---

## ✅ Task Management

Organize daily operations and team assignments.

### Creating a Task

1. **Navigate to** Tasks → Add New Task
2. **Enter details**:
   - **Title**: "Check Zone A humidity"
   - **Description**: Detailed instructions
   - **Priority**: High, Medium, Low
   - **Due Date**: When it needs to be done
   - **Assigned To**: Team member
   - **Related Zone**: Optional link
   - **Related Batch**: Optional link
3. **Add checklist** (optional):
   - ☐ Sub-task 1
   - ☐ Sub-task 2
4. **Set recurrence** (optional):
   - Daily, Weekly, Monthly
   - Custom schedule
5. **Save**

### Task Status

- **📝 Pending**: Not started
- **🔄 In Progress**: Currently working
- **✅ Completed**: Finished
- **❌ Cancelled**: No longer needed

### Completing a Task

1. **Go to** Task Details
2. **Check off** checklist items
3. **Add** completion notes
4. **Click** "Mark Complete"
5. **Optional**: Add photos

### Daily Task List

**Filter by**:
- Due today
- Assigned to me
- Priority
- Zone/Batch

**Sort by**:
- Due date
- Priority
- Created date

---

## 👥 Employee Management

Manage your team, roles, and permissions.

### Adding an Employee

1. **Navigate to** Employees → Add New Employee
2. **Enter details**:
   - **First Name**
   - **Last Name**
   - **Email** (for login)
   - **Phone**
   - **Department**: Operations, Sales, Admin
   - **Role**: Farm Manager, Technician, Worker
   - **Hire Date**
   - **Hourly Rate** (optional, for labor tracking)
3. **Set permissions**:
   - View only
   - Edit
   - Admin
4. **Send** invitation email
5. **Save**

### Departments

- **Operations**: Daily farm work
- **Sales**: Customer management
- **Procurement**: Inventory & supplies
- **Quality Control**: Inspections
- **Administration**: Management

### Roles & Permissions

| Role | Permissions |
|------|-------------|
| **Owner** | Full access, billing |
| **Admin** | All features except billing |
| **Farm Manager** | Batches, harvests, tasks, employees |
| **Technician** | Tasks, harvests, quality checks |
| **Worker** | View only, complete assigned tasks |

### Tracking Work Hours

**Clock In/Out**:
1. **Employee** clicks "Clock In" on dashboard
2. **System** records start time
3. **Employee** clicks "Clock Out" when done
4. **System** calculates hours worked

**View Work Logs**:
- Daily hours by employee
- Weekly summaries
- Labor costs per batch

---

## 🔬 Quality Control

Ensure consistent product quality through inspections.

### Performing Quality Inspection

1. **Navigate to** Quality → New Inspection
2. **Select** batch
3. **Choose** standard:
   - Visual inspection
   - Size/weight check
   - Contamination check
   - Customer requirements
4. **Record observations**:
   - Pass/Fail criteria
   - Defects found
   - Sample size tested
5. **Add photos**
6. **Generate** report
7. **Save**

### Quality Standards

**Create Custom Standards**:
1. **Go to** Quality → Standards
2. **Click** "Add Standard"
3. **Define criteria**:
   - Minimum cap size
   - Color requirements
   - No bruising/damage
   - Freshness indicators
4. **Set** acceptable limits
5. **Save**

### Defect Tracking

**Common Defects**:
- Contamination (green mold, bacteria)
- Irregular shape
- Discoloration
- Pest damage
- Insufficient size

**Recording Defects**:
1. **During** quality inspection
2. **Take** photos
3. **Note** quantity affected
4. **Determine** corrective action
5. **Link** to batch for tracking

---

## 📈 Analytics & Reports

Make data-driven decisions with comprehensive analytics.

### Dashboard Analytics

**Overview Metrics**:
- Total yield (monthly, quarterly, yearly)
- Revenue & profitability
- Success rate (%)
- Average cycle time
- Quality distribution

**Charts Available**:
- Yield trends (line chart)
- Quality distribution (pie chart)
- Revenue vs. cost (bar chart)
- Batch performance (comparison)
- Zone utilization (%)

### Yield Analytics

**View**:
- Yield per batch
- Yield per zone
- Yield per recipe
- Yield trends over time
- Bio-efficiency comparison

**Export Options**:
- CSV
- Excel
- PDF Report

### Financial Analytics

**Reports**:
- **Profitability Summary**
  - Total revenue
  - Total costs
  - Net profit
  - Profit margin (%)
  - ROI (%)

- **Cost Breakdown**
  - Substrate costs
  - Spawn costs
  - Labor costs
  - Utilities
  - Packaging
  - Overhead

- **Revenue Analysis**
  - Sales by crop type
  - Sales by customer
  - Sales by grade
  - Average price per kg

### Custom Reports

1. **Navigate to** Reports → Custom Report
2. **Select** metrics
3. **Choose** date range
4. **Apply** filters
5. **Generate** report
6. **Export** or share

---

## 🤖 IoT Devices

Connect and manage your ESP32 sensors and controllers.

### Device Setup

**See detailed guide**: [IoT Setup Guide](IOT_INTEGRATION_GUIDE.md)

**Quick Overview**:
1. **Purchase** ESP32 boards and sensors
2. **Flash** firmware (provided)
3. **Configure** WiFi credentials
4. **Register** device in CropWise
5. **Assign** to zone
6. **Start** monitoring

### Viewing Sensor Data

1. **Navigate to** Devices or Zone Details
2. **View** real-time data:
   - Temperature
   - Humidity
   - CO₂
   - Light level
3. **Check** historical charts
4. **Set** alert thresholds

### Equipment Control

**If you have relays/actuators connected**:
- **Fans**: Turn on/off, set speed
- **Humidifiers**: Auto/manual control
- **Heaters**: Temperature setpoint
- **Lights**: Schedule, intensity
- **Irrigation**: Watering schedules

**Control Methods**:
- Manual override
- Automatic (recipe-based)
- Scheduled
- Threshold-triggered

### Troubleshooting Devices

**Device Offline**:
- Check WiFi connection
- Verify power supply
- Restart device
- Check MQTT broker

**Incorrect Readings**:
- Calibrate sensors
- Check sensor placement
- Replace faulty sensor
- Verify wiring

---

## 🔔 Notifications

Stay informed with multi-channel alerts.

### Notification Types

1. **Environmental Alerts**
   - Temperature out of range
   - Humidity too low/high
   - CO₂ exceeded threshold

2. **Operational Alerts**
   - Batch stage transition pending
   - Harvest due
   - Task overdue

3. **Inventory Alerts**
   - Low stock warning
   - Item out of stock

4. **System Alerts**
   - Device offline
   - Sensor error
   - System maintenance

### Notification Channels

- **📱 In-App**: Real-time dashboard notifications
- **📧 Email**: Digest or immediate
- **📲 SMS**: Critical alerts only
- **💬 WhatsApp**: (if configured)
- **🔔 Push**: Mobile app notifications

### Managing Preferences

1. **Navigate to** Settings → Notifications
2. **Enable/Disable** by type
3. **Choose** delivery methods
4. **Set** quiet hours
5. **Configure** digest frequency
6. **Save** preferences

---

## ⚙️ Settings

### Account Settings

- Update profile information
- Change password
- Enable 2FA (recommended)
- Linked accounts (Google)
- Session management

### Organization Settings

- Company name
- Logo upload
- Contact information
- Billing details
- Subscription plan

### System Preferences

- **Units**: Metric (kg, °C) or Imperial (lbs, °F)
- **Timezone**: Auto-detect or manual
- **Language**: English (more coming soon)
- **Currency**: USD, EUR, INR, etc.
- **Date Format**: MM/DD/YYYY or DD/MM/YYYY

### Notification Settings

- Email preferences
- SMS alerts
- Quiet hours (10 PM - 7 AM)
- Alert priorities

### Integrations

- Google Calendar
- Slack
- WhatsApp Business
- Zapier (webhook)
- API access tokens

---

## 📱 Mobile App

CropWise works great on mobile browsers and has a dedicated app (coming soon).

### Mobile Web App

**Access**: https://www.cropwise.io on mobile browser

**Features**:
- ✅ Responsive design
- ✅ Touch-optimized interface
- ✅ Camera access for photos
- ✅ Push notifications
- ✅ Offline viewing (limited)

**Add to Home Screen**:

**iOS**:
1. Open in Safari
2. Tap Share button
3. Select "Add to Home Screen"
4. Confirm

**Android**:
1. Open in Chrome
2. Tap menu (3 dots)
3. Select "Add to Home Screen"
4. Confirm

### React Native App (Coming Soon)

**Features**:
- Native performance
- Offline mode
- Camera integration
- Barcode scanning
- GPS location tracking
- Background notifications

**Beta Access**: Contact support@cropwise.io

---

## ❓ FAQs

### General Questions

**Q: Is CropWise free?**
A: We offer a free tier (up to 2 farms, 10 zones, 5 devices). Paid plans start at $49/month.

**Q: Can I use it without IoT devices?**
A: Yes! Manual data entry works fine. IoT is optional but recommended.

**Q: Is my data secure?**
A: Yes. We use bank-level encryption, secure AWS hosting, and regular backups.

**Q: Can multiple users access the same account?**
A: Yes. Add team members with different permission levels.

**Q: Do you offer training?**
A: Yes. Free onboarding webinar + video tutorials. Premium support available.

### Technical Questions

**Q: What sensors do you support?**
A: DHT22, BME280, MH-Z19, BH1750, and more. See [IoT Guide](IOT_INTEGRATION_GUIDE.md).

**Q: Can I export my data?**
A: Yes. CSV, Excel, and JSON export available.

**Q: Do you have an API?**
A: Yes. RESTful API with full documentation. See [API Docs](/api-docs).

**Q: What databases do you use?**
A: PostgreSQL (production), SQLite (development). Fully backed up.

**Q: Is there a mobile app?**
A: Mobile web app available now. Native app coming in Q1 2025.

### Troubleshooting

**Q: I forgot my password**
A: Click "Forgot Password" on login page. Check your email for reset link.

**Q: My device shows as offline**
A: Check WiFi, power, and MQTT broker status. See [Troubleshooting](TROUBLESHOOTING.md).

**Q: Data isn't updating**
A: Refresh page (Ctrl+R). Check internet connection. Clear cache if persistent.

**Q: I can't see my team member's tasks**
A: Check permissions. Only Admins and Managers see all tasks.

**Q: How do I delete my account?**
A: Contact support@cropwise.io. Export your data first.

---

## 📞 Support

### Getting Help

- **📚 Documentation**: [docs.cropwise.io](https://docs.cropwise.io)
- **💬 Live Chat**: Available on website (Mon-Fri, 9 AM - 5 PM EST)
- **📧 Email**: support@cropwise.io
- **📱 Phone**: +1-555-CROPWISE (Business hours)
- **🐛 Bug Reports**: [GitHub Issues](https://github.com/yellowflowersorganics-star/cropwise/issues)

### Community

- **Forum**: [community.cropwise.io](https://community.cropwise.io)
- **Facebook Group**: CropWise Users
- **Discord**: [discord.gg/cropwise](https://discord.gg/cropwise)
- **YouTube**: Tutorials and tips

### Training & Webinars

**Free Onboarding**: Book at [cropwise.io/training](https://cropwise.io/training)

**Topics**:
- Getting Started (30 min)
- Advanced Features (1 hour)
- IoT Setup Workshop (2 hours)
- Monthly Q&A Sessions

---

## 🎓 Best Practices

### For Maximum Yield

1. **Follow recipes carefully** - Don't skip stages
2. **Monitor environmental conditions** - Stay within ranges
3. **Maintain cleanliness** - Prevent contamination
4. **Track everything** - Data = insights
5. **Regular quality checks** - Catch issues early

### For Team Efficiency

1. **Assign tasks clearly** - Use descriptions and checklists
2. **Set up recurring tasks** - Automate routine work
3. **Use mobile app** - Check tasks on the go
4. **Enable notifications** - Don't miss critical alerts
5. **Review analytics weekly** - Identify improvements

### For Profitability

1. **Track all costs** - Labor, materials, utilities
2. **Grade harvests accurately** - Price appropriately
3. **Optimize bio-efficiency** - Improve substrate usage
4. **Reduce waste** - Monitor closely
5. **Plan ahead** - Use analytics for forecasting

---

## 🔄 Updates & Changelog

**Current Version**: 1.0.0

**Recent Updates**:
- Added hierarchical IoT architecture
- Improved mobile responsiveness
- Enhanced quality control features
- New profitability analytics

**Coming Soon**:
- Native mobile app (iOS/Android)
- AI-powered yield predictions
- Advanced reporting dashboard
- Marketplace integration

**See**: [CHANGELOG.md](../CHANGELOG.md) for full history

---

**Happy Farming! 🌱**

*Got questions? We're here to help: support@cropwise.io*

