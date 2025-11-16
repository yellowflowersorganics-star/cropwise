# 📱 WhatsApp & SMS Notification Setup Guide

Complete guide to enable WhatsApp and SMS notifications in CropWise using Twilio.

---

## 🎯 Overview

CropWise now supports **multi-channel notifications**:
- 📱 **WhatsApp** - Rich formatted messages
- 💬 **SMS** - Fallback text messages  
- 📧 **Email** - (Coming soon)
- 🔔 **In-App** - Real-time alerts

---

## 📋 Prerequisites

1. **Twilio Account** (Free trial available)
2. **WhatsApp Business** (via Twilio Sandbox for testing)
3. **Phone Numbers** for employees (with country codes)

---

## 🚀 Step-by-Step Setup

### **Step 1: Create Twilio Account**

1. Go to [https://www.twilio.com/try-twilio](https://www.twilio.com/try-twilio)
2. Sign up for a free trial account
3. Verify your email and phone number
4. Get **$15 free credit** for testing

### **Step 2: Get Your Credentials**

After signup, you'll see your dashboard:

1. **Account SID**: `ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`
2. **Auth Token**: Click to reveal (keep this secret!)
3. Copy both values

### **Step 3: Setup WhatsApp Sandbox (For Testing)**

1. In Twilio Console, go to **Messaging** → **Try it out** → **Send a WhatsApp message**
2. You'll see a sandbox number like: `whatsapp:+14155238886`
3. Follow instructions to **join sandbox**:
   - Send "join <your-code>" to the WhatsApp number
   - Example: `join apple-orange`
4. Your phone is now connected!

### **Step 4: Get SMS Phone Number (Optional)**

1. Go to **Phone Numbers** → **Manage** → **Buy a number**
2. Select your country
3. Choose a number (free trial gives you 1 number)
4. Buy the number (uses your $15 credit)

### **Step 5: Configure Environment Variables**

Add these to your `backend/.env` file:

```env
# Twilio Configuration
TWILIO_ACCOUNT_SID=ACxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
TWILIO_AUTH_TOKEN=your_auth_token_here
TWILIO_WHATSAPP_NUMBER=whatsapp:+14155238886
TWILIO_SMS_NUMBER=+1234567890

# Timezone for scheduled tasks
TIMEZONE=Asia/Kolkata
```

**Important Notes:**
- WhatsApp number format: `whatsapp:+14155238886` (include `whatsapp:` prefix)
- SMS number format: `+1234567890` (just the number with country code)
- Use your timezone (e.g., `Asia/Kolkata`, `America/New_York`, `Europe/London`)

### **Step 6: Restart Backend Server**

```bash
cd backend
npm run dev
```

You should see:
```
✅ Twilio service initialized
✅ Scheduled tasks initialized successfully
  - Daily summary: 7:00 AM
  - Inventory check: Every 6 hours
  - Batch milestones: Every 12 hours
  - Task reminders: Every 15 minutes
  - Alert cleanup: 3:00 AM daily
```

---

## 🧪 Testing the Integration

### **Test 1: Check System Status**

```bash
curl -X GET http://localhost:3000/api/notifications/system/status \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

Expected response:
```json
{
  "success": true,
  "twilio": {
    "enabled": true,
    "whatsappConfigured": true,
    "smsConfigured": true,
    "accountSid": "ACxxxxxxxx..."
  },
  "scheduler": {
    "dailySummary": { "running": true },
    "inventoryCheck": { "running": true },
    "batchMilestones": { "running": true },
    "taskReminders": { "running": true },
    "cleanupAlerts": { "running": true }
  }
}
```

### **Test 2: Send Test WhatsApp**

```bash
curl -X POST http://localhost:3000/api/notifications/send/whatsapp \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -d '{
    "to": "+919876543210",
    "message": "🌱 Hello from CropWise! This is a test WhatsApp message."
  }'
```

### **Test 3: Send Test SMS**

```bash
curl -X POST http://localhost:3000/api/notifications/send/sms \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -d '{
    "to": "+919876543210",
    "message": "Hello from CropWise! This is a test SMS."
  }'
```

### **Test 4: Trigger Daily Summary Manually**

```bash
curl -X POST http://localhost:3000/api/notifications/send/daily-summary \
  -H "Authorization: Bearer YOUR_JWT_TOKEN"
```

---

## 📱 Employee Phone Number Setup

For WhatsApp/SMS to work, employees need phone numbers:

### **Add WhatsApp Number to Employee**

1. Go to **Employees** page
2. Edit an employee
3. Add **Phone** (e.g., `+919876543210`)
4. Add **WhatsApp Number** (e.g., `+919876543210`)
5. Save

**Important:**
- Always use **country code** (e.g., +91 for India, +1 for USA)
- No spaces or dashes (use `+919876543210` not `+91 98765 43210`)
- Employee must have joined WhatsApp sandbox (for testing)

---

## 🔔 Automated Notifications

### **Daily Summary (7:00 AM)**

Every morning at 7:00 AM, employees receive:

```
🌱 Good Morning, Rajesh!

Here are your tasks for today:

1. 🚨 [URGENT] Purchase wheat straw
   ⏰ Due: 10:00 AM
2. 📌 [MEDIUM] Harvest Zone A
   ⏰ Due: 2:00 PM
3. 📝 [LOW] Daily inspection

Total: 3 tasks

Have a productive day! 🚀

- CropWise
```

### **Task Notifications (Real-Time)**

When a task is assigned:

```
📌 New Task Assigned

Hello Rajesh,

Task: Purchase 500kg wheat straw
Priority: URGENT
Due: Today at 10:00 AM
Details: Contact supplier and arrange delivery

Please complete this task on time.

- CropWise
```

### **Environmental Alerts (Real-Time)**

When sensors detect issues:

```
⚠️ HIGH ALERT

Rajesh, attention required!

Environmental Alert: Zone A

Temperature is 25.5% above maximum (32.5°C > 26°C)

Take action: View Zone

- CropWise
```

### **Inventory Alerts (Every 6 Hours)**

When stock is low:

```
⚠️ MEDIUM ALERT

Low Stock Alert

Wheat Straw is running low. Current: 50 kg, Minimum: 200 kg

Take action: Restock Now

- CropWise
```

---

## ⚙️ Customizing Notification Preferences

Users can customize their notification settings:

### **Via Frontend (Settings Page)**

1. Go to **Settings** → **Notifications** tab
2. Toggle channels:
   - ✅ In-App Notifications
   - ✅ WhatsApp Notifications
   - ✅ SMS Notifications
   - ☐ Email Notifications
3. Configure quiet hours (e.g., 10 PM - 7 AM)
4. Set severity threshold (Low, Medium, High, Critical)
5. Enable daily digest

### **Quiet Hours**

No notifications will be sent during quiet hours (except critical alerts).

Example:
- Start: 22:00 (10 PM)
- End: 07:00 (7 AM)

---

## 🔄 Scheduled Jobs

| Job | Frequency | Description |
|-----|-----------|-------------|
| Daily Summary | 7:00 AM | Send task list to all active employees |
| Inventory Check | Every 6 hours | Check low stock and send alerts |
| Batch Milestones | Every 12 hours | Check harvest readiness, stage transitions |
| Task Reminders | Every 15 minutes | Send reminders for upcoming tasks |
| Alert Cleanup | 3:00 AM | Delete old dismissed alerts (30+ days) |

### **Manually Trigger Jobs (For Testing)**

```bash
# Trigger daily summary
POST /api/notifications/trigger/dailySummary

# Trigger inventory check
POST /api/notifications/trigger/inventoryCheck

# Trigger batch milestones
POST /api/notifications/trigger/batchMilestones

# Trigger task reminders
POST /api/notifications/trigger/taskReminders

# Trigger alert cleanup
POST /api/notifications/trigger/cleanupAlerts
```

---

## 💰 Twilio Pricing (After Free Trial)

### **Free Trial**
- $15 credit
- ~500 WhatsApp messages
- ~500 SMS messages

### **Production Pricing (India)**
- WhatsApp: ₹0.25 per message (~$0.003)
- SMS: ₹0.50 per message (~$0.006)
- Very affordable for farms!

### **Recommended Monthly Budget**

For a farm with 10 employees:
- Daily summaries: 10 employees × 30 days = 300 messages/month
- Task notifications: ~200 messages/month
- Alert notifications: ~100 messages/month

**Total: ~600 messages/month = ₹150 ($2) per month** ✅ Very affordable!

---

## 🚀 Production Setup (Moving from Sandbox)

### **Step 1: Upgrade Twilio Account**
1. Add payment method
2. Get account approved

### **Step 2: Request WhatsApp Business API Access**
1. Go to Twilio Console → WhatsApp → Request Access
2. Provide business details
3. Wait for approval (1-3 days)

### **Step 3: Get WhatsApp Business Number**
1. Buy a dedicated WhatsApp number
2. Configure webhooks
3. Update `TWILIO_WHATSAPP_NUMBER` in `.env`

### **Step 4: Update Employee Numbers**
1. No sandbox join needed in production!
2. Employees just need WhatsApp installed
3. Messages will be received automatically

---

## 🛠️ Troubleshooting

### **Problem: "Twilio not enabled"**
**Solution:**
- Check `.env` file has correct credentials
- Restart backend server
- Verify Account SID and Auth Token are correct

### **Problem: "WhatsApp message failed"**
**Solution:**
- Ensure phone number has country code (e.g., `+919876543210`)
- Check employee joined WhatsApp sandbox (for testing)
- Verify sandbox is still active (expires after 3 days of inactivity)

### **Problem: "SMS not delivered"**
**Solution:**
- Verify you have an SMS-enabled Twilio number
- Check phone number format (no spaces or dashes)
- Ensure number is verified in Twilio (for trial accounts)

### **Problem: "Scheduled tasks not running"**
**Solution:**
- Check server logs for scheduler errors
- Verify timezone is set correctly in `.env`
- Test manually: `POST /api/notifications/trigger/dailySummary`

---

## 📊 Monitoring & Logs

### **View Logs**

```bash
# Backend logs
cd backend
npm run dev

# Look for:
✅ WhatsApp sent to +919876543210: SM...
✅ SMS sent to +919876543210: SM...
✅ Daily summaries sent to 5 employees
⏰ Running daily summary task...
```

### **Twilio Dashboard**

- View all sent messages: [Twilio Console → Messaging](https://console.twilio.com/us1/develop/sms/logs)
- Track delivery status
- Monitor costs

---

## ✅ Quick Start Checklist

- [ ] Create Twilio account
- [ ] Get Account SID and Auth Token
- [ ] Join WhatsApp Sandbox
- [ ] Add credentials to `.env`
- [ ] Restart backend server
- [ ] Add employee phone numbers
- [ ] Test WhatsApp message
- [ ] Test SMS message
- [ ] Test daily summary
- [ ] Configure notification preferences

---

## 🎉 You're All Set!

Your CropWise is now equipped with professional WhatsApp & SMS notifications!

**Next Steps:**
- Add all employee phone numbers
- Customize notification preferences
- Monitor message delivery
- Upgrade to production WhatsApp API when ready

**Questions?** Check the Twilio docs or contact support! 🚀🌱

