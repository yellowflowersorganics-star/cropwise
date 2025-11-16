# ❓ CropWise - Frequently Asked Questions

Common questions and answers about CropWise.

---

## 📋 General Questions

### What is CropWise?

CropWise is an open-source, IoT-powered farm management platform designed for controlled-environment agriculture. It helps farmers manage their operations through real-time monitoring, batch tracking, harvest recording, analytics, and automated environmental control.

### Is CropWise really free?

Yes! CropWise is 100% open-source under the MIT License. You can:
- ✅ Use it commercially
- ✅ Modify the source code
- ✅ Self-host on your own servers
- ✅ Deploy without any fees

Optional cloud hosting services are available for a fee, but the software itself is free forever.

### What types of crops can I grow with CropWise?

CropWise is designed for controlled-environment agriculture and works well with:
- **Mushrooms** (oyster, shiitake, lion's mane, etc.) ✅ Fully supported
- **Leafy greens** (lettuce, spinach, herbs) ✅ Fully supported
- **Vegetables** (tomatoes, peppers, cucumbers) ✅ Fully supported
- **Vertical farming** (any crop) ✅ Fully supported
- **Research crops** ✅ Fully supported

The system is crop-agnostic — you define the environmental parameters for your specific growing method.

### Do I need to be a programmer to use it?

**No!** CropWise has a user-friendly web interface that requires no programming knowledge. However, if you want to:
- Customize the system
- Add new features
- Integrate with other tools

Some programming knowledge (JavaScript/Node.js) will be helpful.

### Can I use it without IoT devices?

**Yes!** You can use CropWise for:
- Farm and zone management
- Batch tracking
- Harvest recording
- Inventory management
- Task management
- Employee management
- Analytics and reporting

IoT devices are **optional** but highly recommended for automated environmental monitoring and control.

---

## 💻 Technical Questions

### What are the system requirements?

**Minimum** (Small farm, <5 zones):
- 2 CPU cores
- 4 GB RAM
- 20 GB storage
- Ubuntu 20.04+ or Windows Server

**Recommended** (Commercial operation):
- 4 CPU cores
- 16 GB RAM
- 100 GB SSD
- Ubuntu 22.04 LTS

See [Installation Guide](INSTALLATION.md) for details.

### What database does it use?

- **Development**: SQLite (no setup required)
- **Production**: PostgreSQL 15+ (recommended)

PostgreSQL is preferred for production because it:
- Handles concurrent users better
- Supports advanced features (JSON, full-text search)
- Integrates with AI/ML tools (pgvector, TimescaleDB)
- Scales to millions of records

### Can it run on Raspberry Pi?

**Yes, but with limitations:**
- Raspberry Pi 4 (8GB): ✅ Small farms (<5 zones)
- Raspberry Pi 5 (8GB): ✅ Medium farms (<15 zones)
- Raspberry Pi as MQTT gateway: ✅ Recommended use

For commercial operations, we recommend a dedicated server or cloud hosting.

### Does it work on Windows?

**Yes!** CropWise runs on:
- ✅ Windows 10/11
- ✅ Windows Server 2019+
- ✅ Ubuntu/Debian Linux
- ✅ macOS 11+

See [Installation Guide](INSTALLATION.md) for platform-specific instructions.

### What IoT devices are supported?

**Microcontrollers:**
- ✅ ESP32 (recommended)
- ✅ ESP8266
- ✅ Raspberry Pi Pico W
- ✅ Arduino with WiFi shield

**Sensors:**
- ✅ DHT22 (temperature, humidity)
- ✅ BME280 (temp, humidity, pressure)
- ✅ MH-Z19 / MQ-135 (CO₂)
- ✅ BH1750 (light intensity)
- ✅ Soil moisture sensors
- ✅ Water flow sensors
- ✅ pH / EC sensors

See [IoT Integration Guide](IOT_INTEGRATION_GUIDE.md) for complete list.

### Can I integrate with existing systems?

**Yes!** CropWise provides:
- **REST API**: Full API access
- **Webhooks**: Real-time event notifications
- **CSV Export**: Bulk data export
- **MQTT**: Standard IoT protocol
- **Zapier**: Connect to 5,000+ apps (coming soon)

See [API Documentation](/api-docs) for details.

---

## 💰 Cost & Pricing

### How much does it cost to run?

**Self-hosted** (one-time + ongoing):
- Server hardware: $500-2,000 (one-time)
- Electricity: ~$10-30/month
- Internet: Your existing connection
- **Total**: ~$10-30/month after initial investment

**Cloud-hosted on AWS** (monthly):
- Small farm: $50-80/month
- Medium farm: $100-150/month
- Large farm: $200-400/month

See [AWS Deployment Guide](DEPLOY_TO_AWS_NOW.md) for detailed cost breakdown.

### Do I need to pay for support?

**No!** Community support is free:
- Documentation
- GitHub Issues
- Community Forum
- Discord/Slack

**Paid support** is available for:
- Priority email support
- Phone support
- Custom development
- Training sessions
- SLA guarantees

### What's the difference between self-hosted and cloud-hosted?

| Feature | Self-Hosted | Cloud-Hosted |
|---------|-------------|--------------|
| **Cost** | Hardware + electricity | Monthly subscription |
| **Control** | Full control | Managed service |
| **Updates** | Manual | Automatic |
| **Backups** | You manage | Automatic |
| **Scaling** | Manual | Auto-scaling |
| **Support** | Community | Priority support |

---

## 🔐 Security & Privacy

### Is my data secure?

**Yes!** CropWise uses:
- **Encryption**: All data encrypted at rest and in transit (HTTPS/TLS)
- **Authentication**: JWT tokens + Google OAuth
- **Authorization**: Role-based access control (RBAC)
- **Backups**: Automated daily backups
- **Compliance**: GDPR-ready

See [Security Guide](SECURITY_GUIDE.md) for details.

### Where is my data stored?

**Self-hosted**: On your own server (you control location)

**Cloud-hosted**: 
- Default: AWS US-East-1 (Virginia)
- Optional: EU (Frankfurt), Asia (Singapore)
- You can request specific regions

### Can I export my data?

**Yes!** You can export:
- All data as CSV/Excel
- Database backups (PostgreSQL dumps)
- Sensor data (JSON/CSV)
- Reports (PDF)

**No lock-in** — you own your data.

### Who can see my data?

**Self-hosted**: Only people you grant access

**Cloud-hosted**: 
- Your team members (based on permissions)
- CropWise team (only for support, with your permission)
- **Never sold to third parties**

---

## 🤖 IoT & Hardware

### How many sensors do I need?

**Minimum** (per zone):
- 1x Temperature/Humidity sensor (DHT22): $5
- 1x ESP32 microcontroller: $8
- **Total**: ~$15/zone

**Recommended** (per zone):
- 1x ESP32 Master: $8
- 2x ESP32 Slaves: $16
- 1x DHT22 (temp/humidity): $5
- 1x BH1750 (light): $3
- 1x MQ-135 (CO₂): $5
- 1x Relay module (4-channel): $5
- **Total**: ~$42/zone

See [IoT Setup Guide](ESP_NOW_SETUP_GUIDE.md) for shopping list.

### Do I need a Raspberry Pi?

**Optional but recommended** for:
- MQTT broker (gateway)
- Local data processing
- Offline operation
- Multiple zones (>5)

**Alternatives**:
- Run MQTT broker on your main server
- Use cloud MQTT service (AWS IoT, HiveMQ)

### Can I use Arduino instead of ESP32?

**Yes**, but ESP32 is recommended because:
- Built-in WiFi + Bluetooth
- More memory (520KB RAM)
- Faster processor (240 MHz)
- Lower cost (~$5-8)
- Better community support

Arduino requires additional WiFi shield ($15-30).

### How reliable is the IoT system?

**Very reliable** with proper setup:
- Auto-reconnection if WiFi drops
- Offline buffering (stores data locally)
- Watchdog timers (auto-restart on crash)
- Redundant sensors (backup sensors)

**Uptime**: Typically 99%+ with good WiFi.

---

## 📊 Features & Functionality

### Can I manage multiple farms?

**Yes!** CropWise supports:
- Unlimited farms
- Unlimited zones per farm
- Unlimited batches
- Multi-location management
- Separate user permissions per farm

### Does it support multiple users?

**Yes!** Features include:
- Unlimited users (team members)
- Role-based access control (Owner, Admin, Manager, Worker)
- Department management
- Task assignments
- Individual dashboards

### Can I get notifications on my phone?

**Yes!** Notifications via:
- 📧 Email
- 📲 SMS (via Twilio)
- 💬 WhatsApp (via Twilio)
- 📱 Push notifications (mobile app, coming soon)
- 🔔 In-app notifications

### Does it have a mobile app?

**Currently**: Mobile-responsive web app (works in mobile browser)

**Coming Q1 2025**: Native apps for iOS and Android

### Can I customize the dashboard?

**Currently**: Standard dashboard layout

**Coming Q2 2025**: 
- Drag-and-drop widgets
- Custom KPIs
- Personalized views
- Dark mode

### Does it track profitability?

**Yes!** Complete financial tracking:
- Revenue tracking (sales)
- Cost tracking (materials, labor, utilities)
- Profit per batch
- ROI calculations
- Profit margins
- Batch comparisons

---

## 🔄 Updates & Maintenance

### How often is it updated?

**Release schedule**:
- **Major releases**: Every 6-12 months (v1.0, v2.0)
- **Minor releases**: Every 2-4 weeks (v1.1, v1.2)
- **Patches**: As needed (v1.1.1, v1.1.2)

**Security updates**: Released immediately when needed.

### Do I need to update manually?

**Self-hosted**: Yes, but it's simple:
```bash
git pull origin main
npm install
npm run migrate
pm2 restart
```

**Cloud-hosted**: Automatic updates (zero downtime).

### Will updates break my data?

**No!** We follow these practices:
- Database migrations (automated)
- Backward compatibility
- Rollback scripts
- Testing on staging first
- Release notes for breaking changes

**Always backup before major updates.**

### How do I get notified of updates?

- **GitHub**: Watch the repository (Releases only)
- **Email**: Subscribe to our newsletter
- **RSS**: [blog.cropwise.io/feed](https://blog.cropwise.io/feed)
- **Discord**: #announcements channel

---

## 🆘 Support & Community

### Where can I get help?

**Free support:**
- 📚 [Documentation](https://docs.cropwise.io)
- 💬 [Community Forum](https://community.cropwise.io)
- 🐛 [GitHub Issues](https://github.com/yellowflowersorganics-star/cropwise/issues)
- 💬 [Discord](https://discord.gg/cropwise)

**Paid support:**
- 📧 Email: support@cropwise.io
- 📱 Phone: +1-555-CROPWISE
- 💼 Enterprise: SLA with <1 hour response time

### Can I hire someone to set it up for me?

**Yes!** Options:
1. **Official setup service**: $299 one-time fee
2. **Certified partners**: See [Partners page](https://cropwise.io/partners)
3. **Hire a freelancer**: Post on Upwork, Fiverr
4. **Community help**: Ask in forum (may be free!)

### Is there a community?

**Yes!** Join us:
- **Discord**: [discord.gg/cropwise](https://discord.gg/cropwise) - 500+ members
- **Forum**: [community.cropwise.io](https://community.cropwise.io)
- **Facebook**: CropWise Users Group
- **Reddit**: r/CropWiseOS (coming soon)

### Can I contribute to the project?

**Absolutely!** We welcome:
- 🐛 Bug reports
- ✨ Feature suggestions
- 🔧 Code contributions
- 📝 Documentation improvements
- 🌍 Translations
- 💰 Donations/sponsorship

See [Contributing Guide](../CONTRIBUTING.md).

---

## 🚀 Advanced Questions

### Can it scale to 100+ zones?

**Yes!** CropWise scales to:
- 1,000+ zones (with proper hardware)
- 10,000+ batches
- Millions of sensor readings
- 100+ concurrent users

**Requirements for large scale**:
- PostgreSQL with read replicas
- Redis caching
- Load balancer
- Auto-scaling (AWS ECS/Kubernetes)

See [Admin Guide](ADMIN_GUIDE.md) for scaling strategies.

### Can I use it for research?

**Yes!** Perfect for:
- University research projects
- Agricultural R&D
- A/B testing growing methods
- Publication-ready data
- Grant-funded projects

**Academia perks**:
- Free cloud hosting (contact us)
- Citation-ready documentation
- Data export for analysis

### Does it support AI/ML?

**Yes!** The architecture is AI-ready:
- PostgreSQL + pgvector (vector similarity)
- TimescaleDB (time-series optimization)
- Nvidia Jetson support (edge AI)
- Python integration
- TensorFlow/PyTorch compatible

**Coming in Q2 2025**:
- Yield prediction models
- Disease detection (computer vision)
- Automated optimization

### Can I white-label it?

**Yes!** Under MIT License you can:
- Rebrand the interface
- Use your logo and colors
- Host under your domain
- Offer as a service
- **No attribution required** (but appreciated!)

### Is there an enterprise version?

**No separate version** — same codebase for everyone!

**Enterprise features** (available now):
- Priority support
- SLA guarantees
- Custom development
- Training sessions
- White-glove onboarding
- Dedicated account manager

**Contact**: enterprise@cropwise.io

---

## 🌍 Language & Localization

### Is it available in my language?

**Currently**: English only

**Coming soon**:
- Spanish
- Portuguese
- French
- German
- Chinese
- Hindi

Want to help translate? See [Contributing Guide](../CONTRIBUTING.md).

### Can I change the units?

**Yes!** Supports:
- **Temperature**: Celsius (°C) or Fahrenheit (°F)
- **Weight**: Kilograms (kg) or Pounds (lbs)
- **Area**: Square meters (m²) or Square feet (ft²)
- **Volume**: Liters (L) or Gallons (gal)
- **Currency**: USD, EUR, INR, GBP, etc.

Configure in Settings → Preferences.

---

## 🔗 Integration Questions

### Does it work with QuickBooks?

**Coming Q2 2025**: Direct integration with:
- QuickBooks
- Xero
- FreshBooks

**Currently**: Export financial data as CSV and import manually.

### Can I connect to my weather station?

**Yes!** Supports:
- MQTT weather stations
- HTTP weather APIs
- CSV import

**Coming soon**: Direct integration with popular weather services.

### Does it integrate with e-commerce?

**Coming Q3 2025**: 
- WooCommerce
- Shopify
- Magento

**Currently**: Use API to build custom integration.

---

## 📞 Still Have Questions?

Can't find your answer? We're here to help!

- **Email**: support@cropwise.io
- **Live Chat**: [cropwise.io](https://cropwise.io) (Mon-Fri, 9 AM - 5 PM EST)
- **Schedule a call**: [calendly.com/cropwise](https://calendly.com/cropwise)
- **Forum**: [community.cropwise.io](https://community.cropwise.io)

---

**Updated**: November 2025  
**Version**: 1.0.0

