# 🔧 CropWise - Troubleshooting Guide

Quick solutions to common issues.

---

## 📋 Quick Diagnosis

### System Health Check

```bash
# Backend health
curl http://localhost:3000/health

# Database connection
psql -h localhost -U cropwise_admin -d cropwise_db -c "SELECT 1;"

# Redis connection
redis-cli ping

# MQTT broker
mosquitto_sub -h localhost -t 'test' -C 1
```

---

## 🚨 Common Issues

### 1. Backend Won't Start

**Symptoms**: Server fails to start, error in logs

#### Solution A: Port Already in Use
```bash
# Find process using port 3000
sudo lsof -i :3000

# Kill the process
sudo kill -9 <PID>

# Or change port in .env
PORT=3001
```

#### Solution B: Database Connection Failed
```bash
# Check database is running
sudo systemctl status postgresql

# Test connection
psql -h localhost -U cropwise_admin -d cropwise_db

# Check credentials in .env
cat backend/.env | grep DB_
```

#### Solution C: Missing Dependencies
```bash
cd backend
rm -rf node_modules package-lock.json
npm install
npm start
```

---

### 2. Frontend Shows "Network Error"

**Symptoms**: API calls fail, console shows CORS errors

#### Solution A: Backend Not Running
```bash
# Check backend status
curl http://localhost:3000/health

# Start backend
cd backend
npm start
```

#### Solution B: Incorrect API URL
```javascript
// Check frontend/.env
VITE_API_URL=http://localhost:3000  // Should match backend

// Or frontend/src/services/*.service.js
const API_BASE_URL = import.meta.env.VITE_API_URL || 'http://localhost:3000';
```

#### Solution C: CORS Configuration
```javascript
// backend/src/index.js
app.use(cors({
  origin: ['http://localhost:8080', 'http://localhost:5173'],
  credentials: true
}));
```

---

### 3. Database Errors

#### Error: "relation does not exist"

**Cause**: Migrations not run

**Solution**:
```bash
cd backend
npm run migrate
```

#### Error: "password authentication failed"

**Cause**: Wrong credentials

**Solution**:
```bash
# Check .env file
cat backend/.env | grep DB_PASSWORD

# Reset PostgreSQL password
sudo -u postgres psql
ALTER USER cropwise_admin WITH PASSWORD 'new_password';
\q

# Update .env
DB_PASSWORD=new_password
```

#### Error: "too many connections"

**Cause**: Connection pool exhausted

**Solution**:
```javascript
// backend/src/config/database.js
pool: {
  max: 20,  // Increase from 10
  min: 5,
  acquire: 30000,
  idle: 10000
}
```

---

### 4. IoT Device Issues

#### Device Shows as "Offline"

**Diagnostics**:
```bash
# Check MQTT broker
sudo systemctl status mosquitto

# Test MQTT connection
mosquitto_sub -h localhost -p 1883 -t 'cropwise/#' -v

# Check device logs (if ESP32 connected via USB)
screen /dev/ttyUSB0 115200
```

**Solutions**:
1. **WiFi Connection**:
   - Verify SSID and password in ESP32 firmware
   - Check signal strength
   - Restart router

2. **MQTT Broker**:
   ```bash
   sudo systemctl restart mosquitto
   sudo journalctl -u mosquitto -f
   ```

3. **Firewall**:
   ```bash
   sudo ufw allow 1883/tcp
   ```

#### Sensor Readings Incorrect

**Solutions**:
1. **Calibrate Sensors**:
   ```cpp
   // In ESP32 firmware
   dht.begin();
   delay(2000);  // Wait for sensor to stabilize
   ```

2. **Check Wiring**:
   - Verify VCC, GND, DATA pins
   - Ensure proper voltage (3.3V or 5V)
   - Check for loose connections

3. **Replace Faulty Sensor**:
   - Test with known good sensor
   - Check sensor datasheet for specs

---

### 5. Authentication Issues

#### "Invalid credentials" on Login

**Solutions**:
1. **Reset Password**:
   ```bash
   node backend/scripts/reset-password.js user@example.com
   ```

2. **Check JWT Secret**:
   ```bash
   # Must be consistent across all backend instances
   cat backend/.env | grep JWT_SECRET
   ```

#### Google OAuth Not Working

**Error**: "redirect_uri_mismatch"

**Solution**:
```bash
# 1. Go to Google Cloud Console
# 2. APIs & Services → Credentials
# 3. Update Authorized redirect URIs:
https://yourdomain.com/api/auth/google/callback

# 4. Update backend/.env
GOOGLE_CALLBACK_URL=https://yourdomain.com/api/auth/google/callback
```

#### Session Expires Too Quickly

**Solution**:
```bash
# Update JWT expiration
JWT_EXPIRES_IN=7d  # or 30d for longer sessions

# Restart backend
pm2 restart cropwise-backend
```

---

### 6. Performance Issues

#### Slow Page Load

**Diagnostics**:
```bash
# Check server resources
htop
free -h
df -h

# Check database performance
psql -d cropwise_db -c "SELECT pg_stat_statements_reset();"
# Run slow query
psql -d cropwise_db -c "SELECT query, calls, total_time FROM pg_stat_statements ORDER BY total_time DESC LIMIT 10;"
```

**Solutions**:
1. **Enable Redis Caching**:
   ```bash
   # Check Redis is running
   redis-cli ping

   # Verify connection in .env
   REDIS_HOST=localhost
   REDIS_PORT=6379
   ```

2. **Add Database Indexes**:
   ```sql
   CREATE INDEX idx_batches_zone_status ON batches(zone_id, status);
   CREATE INDEX idx_harvests_batch ON harvests(batch_id);
   CREATE INDEX idx_telemetry_zone_time ON telemetries(zone_id, timestamp);
   ```

3. **Optimize Queries**:
   ```javascript
   // Add pagination
   const batches = await Batch.findAll({
     limit: 20,
     offset: (page - 1) * 20,
     order: [['created_at', 'DESC']]
   });
   ```

#### High Memory Usage

**Solution**:
```bash
# Restart services
pm2 restart cropwise-backend
sudo systemctl restart postgresql

# Increase swap space
sudo fallocate -l 4G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile

# Make permanent
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

---

### 7. Build/Deployment Issues

#### npm install Fails

**Error**: "EACCES: permission denied"

**Solution**:
```bash
# Fix npm permissions
sudo chown -R $USER:$USER ~/.npm
sudo chown -R $USER:$USER node_modules

# Or use nvm (recommended)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install 18
nvm use 18
```

#### Frontend Build Fails

**Error**: "JavaScript heap out of memory"

**Solution**:
```bash
# Increase Node.js memory
export NODE_OPTIONS="--max-old-space-size=4096"
npm run build

# Or in package.json
"scripts": {
  "build": "NODE_OPTIONS='--max-old-space-size=4096' vite build"
}
```

#### Docker Container Won't Start

**Solution**:
```bash
# Check logs
docker logs cropwise-backend

# Rebuild without cache
docker-compose build --no-cache
docker-compose up -d

# Check disk space
df -h
```

---

### 8. Data Issues

#### Missing Data After Update

**Solution**:
```bash
# Rollback migration
cd backend
npm run migrate:undo

# Or restore from backup
psql -h localhost -U cropwise_admin -d cropwise_db < /backups/cropwise_db.sql
```

#### Duplicate Records

**Solution**:
```sql
-- Find duplicates
SELECT batch_number, COUNT(*)
FROM batches
GROUP BY batch_number
HAVING COUNT(*) > 1;

-- Delete duplicates (keep first)
DELETE FROM batches
WHERE id NOT IN (
  SELECT MIN(id)
  FROM batches
  GROUP BY batch_number
);
```

#### Data Not Syncing

**Solution**:
```bash
# Check WebSocket connection (if using Socket.IO)
# In browser console:
socket.connected

# Force page refresh
Ctrl + Shift + R  # Windows/Linux
Cmd + Shift + R   # Mac

# Clear browser cache
```

---

### 9. Email/SMS Not Sending

#### Emails Not Received

**Diagnostics**:
```bash
# Check logs
tail -f backend/logs/combined.log | grep email

# Test SMTP connection
telnet email-smtp.us-east-1.amazonaws.com 587
```

**Solutions**:
1. **Verify SMTP Settings**:
   ```bash
   SMTP_HOST=email-smtp.us-east-1.amazonaws.com
   SMTP_PORT=587
   SMTP_USER=your_smtp_username
   SMTP_PASS=your_smtp_password
   EMAIL_FROM=noreply@yourdomain.com
   ```

2. **Check Spam Folder**

3. **Verify Domain Authentication** (SPF, DKIM, DMARC)

#### SMS/WhatsApp Not Sending

**Solution**:
```bash
# Verify Twilio credentials
TWILIO_ACCOUNT_SID=AC...
TWILIO_AUTH_TOKEN=your_token
TWILIO_PHONE_NUMBER=+1234567890

# Test API
curl -X POST https://api.twilio.com/2010-04-01/Accounts/$TWILIO_ACCOUNT_SID/Messages.json \
  --data-urlencode "To=+1234567890" \
  --data-urlencode "From=$TWILIO_PHONE_NUMBER" \
  --data-urlencode "Body=Test message" \
  -u $TWILIO_ACCOUNT_SID:$TWILIO_AUTH_TOKEN
```

---

### 10. Chart/Dashboard Not Loading

**Symptoms**: Blank dashboard, charts not rendering

**Solutions**:
1. **Check API Response**:
   ```javascript
   // Open browser console (F12)
   // Check Network tab for failed requests
   ```

2. **Clear Cache**:
   ```bash
   # Hard refresh
   Ctrl + F5  # Windows/Linux
   Cmd + Shift + R  # Mac
   ```

3. **Check Data Format**:
   ```javascript
   // Ensure data matches expected format
   console.log(data);
   ```

---

## 🔍 Debugging Tools

### Backend Debugging

```bash
# Run with debug logging
cd backend
DEBUG=* npm start

# Or use Node.js debugger
node --inspect src/index.js

# Then open Chrome and go to:
# chrome://inspect
```

### Frontend Debugging

```javascript
// Enable React DevTools
// Install: https://react.dev/learn/react-developer-tools

// Enable verbose logging
localStorage.setItem('debug', '*');

// View Zustand state
console.log(useAuthStore.getState());
```

### Database Debugging

```sql
-- Enable query logging
SET log_statement = 'all';

-- View active queries
SELECT pid, query, state, query_start
FROM pg_stat_activity
WHERE state != 'idle';

-- Kill long-running query
SELECT pg_terminate_backend(pid);
```

---

## 📊 Log Files

### Important Log Locations

```bash
# Backend application logs
/var/www/cropwise/backend/logs/combined.log
/var/www/cropwise/backend/logs/error.log

# PM2 logs
~/.pm2/logs/cropwise-backend-out.log
~/.pm2/logs/cropwise-backend-error.log

# Nginx logs
/var/log/nginx/cropwise_access.log
/var/log/nginx/cropwise_error.log

# PostgreSQL logs
/var/log/postgresql/postgresql-15-main.log

# System logs
/var/log/syslog
journalctl -u cropwise-backend
```

### Log Analysis

```bash
# Count errors in last hour
grep -c "ERROR" backend/logs/error.log

# Find specific error
grep "Database connection" backend/logs/error.log

# Monitor logs in real-time
tail -f backend/logs/combined.log

# Search logs with timestamps
awk '/2024-11-14 10:00/,/2024-11-14 11:00/' backend/logs/combined.log
```

---

## 🆘 Getting Help

### Before Contacting Support

Gather this information:

1. **System Info**:
   ```bash
   node --version
   npm --version
   psql --version
   uname -a
   ```

2. **Error Messages**:
   - Exact error text
   - Screenshots
   - Browser console output

3. **Steps to Reproduce**:
   - What were you doing?
   - Can you reproduce it?
   - Does it happen every time?

4. **Recent Changes**:
   - Did you update anything?
   - New environment?
   - Configuration changes?

### Contact Support

- **Email**: support@cropwise.io
- **GitHub Issues**: https://github.com/yellowflowersorganics-star/cropwise/issues
- **Community Forum**: https://community.cropwise.io
- **Emergency**: +1-555-CROPWISE

---

## 📚 Additional Resources

- [User Guide](USER_GUIDE.md)
- [Admin Guide](ADMIN_GUIDE.md)
- [API Documentation](/api-docs)
- [Git Workflow](GIT_WORKFLOW.md)
- [AWS Deployment](DEPLOY_TO_AWS_NOW.md)

---

**Still stuck? We're here to help!**  
*support@cropwise.io*

