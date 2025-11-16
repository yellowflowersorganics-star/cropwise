# 🔧 CropWise - Administrator Guide

**Version:** 1.0.0  
**Last Updated:** November 2025

This guide is for system administrators responsible for deploying, configuring, and maintaining CropWise.

---

## 📋 Table of Contents

1. [System Requirements](#system-requirements)
2. [Installation](#installation)
3. [Configuration](#configuration)
4. [Database Management](#database-management)
5. [User Management](#user-management)
6. [Security](#security)
7. [Backup & Recovery](#backup--recovery)
8. [Monitoring](#monitoring)
9. [Performance Tuning](#performance-tuning)
10. [Troubleshooting](#troubleshooting)
11. [Maintenance](#maintenance)
12. [Scaling](#scaling)

---

## 💻 System Requirements

### Minimum Requirements (Development)

**Backend Server**:
- CPU: 2 cores
- RAM: 4 GB
- Storage: 20 GB SSD
- OS: Ubuntu 20.04+ / Windows Server 2019+ / macOS 11+
- Node.js: 18.x or higher
- PostgreSQL: 15.x or SQLite 3.x

**Frontend Server** (optional, can be served by backend):
- CPU: 1 core
- RAM: 2 GB
- Storage: 5 GB SSD
- Web Server: Nginx / Apache

**Network**:
- Bandwidth: 10 Mbps minimum
- Latency: <100ms to users

### Recommended Requirements (Production)

**Backend Server**:
- CPU: 4 cores / 8 threads
- RAM: 16 GB
- Storage: 100 GB SSD (NVMe preferred)
- OS: Ubuntu 22.04 LTS
- Load Balancer: AWS ALB / Nginx

**Database Server**:
- CPU: 4 cores
- RAM: 16 GB
- Storage: 250 GB SSD (+ backup storage)
- PostgreSQL: 15.4 or higher
- Multi-AZ deployment (AWS RDS recommended)

**Cache Server**:
- Redis: 7.x
- RAM: 4 GB
- Dedicated instance recommended

**MQTT Broker** (for IoT):
- Eclipse Mosquitto 2.x
- CPU: 2 cores
- RAM: 2 GB

---

## 📦 Installation

### Option 1: Quick Start (Docker Compose)

```bash
# Clone repository
git clone https://github.com/yellowflowersorganics-star/cropwise.git
cd cropwise

# Copy environment files
cp backend/.env.example backend/.env
cp frontend/.env.example frontend/.env

# Edit configuration
nano backend/.env

# Start services
docker-compose up -d

# Check status
docker-compose ps

# View logs
docker-compose logs -f
```

**Services Started**:
- Backend API: http://localhost:3000
- Frontend: http://localhost:8080
- PostgreSQL: localhost:5432
- Redis: localhost:6379
- Mosquitto MQTT: localhost:1883

### Option 2: Manual Installation

#### Backend Setup

```bash
# Navigate to backend
cd backend

# Install dependencies
npm install

# Configure environment
cp .env.example .env
nano .env

# Run database migrations
npm run migrate

# Seed initial data (optional)
npm run seed

# Start server
npm run dev        # Development
npm start          # Production
```

#### Frontend Setup

```bash
# Navigate to frontend
cd frontend

# Install dependencies
npm install

# Configure environment
cp .env.example .env
nano .env

# Build for production
npm run build

# Serve with Nginx or use dev server
npm run dev
```

### Option 3: AWS Deployment

See [AWS Deployment Guide](DEPLOY_TO_AWS_NOW.md) for complete instructions.

---

## ⚙️ Configuration

### Environment Variables

#### Backend Configuration

**File**: `backend/.env`

```bash
# Environment
NODE_ENV=production
PORT=3000

# Database
DB_DIALECT=postgres
DB_HOST=your-db-host.rds.amazonaws.com
DB_PORT=5432
DB_NAME=cropwise_db
DB_USER=cropwise_admin
DB_PASSWORD=your_secure_password

# Security
JWT_SECRET=your_64_character_secret_key_here_minimum_32_chars
JWT_EXPIRES_IN=7d

# Redis
REDIS_HOST=your-redis-host.cache.amazonaws.com
REDIS_PORT=6379
REDIS_PASSWORD=your_redis_password
REDIS_TLS=true

# MQTT Broker
MQTT_BROKER_URL=mqtt://your-mqtt-broker:1883
MQTT_USERNAME=cropwise
MQTT_PASSWORD=secure_mqtt_password

# Google OAuth
GOOGLE_CLIENT_ID=your_google_client_id.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=your_google_client_secret
GOOGLE_CALLBACK_URL=https://api.yourdomain.com/api/auth/google/callback

# Twilio (SMS/WhatsApp)
TWILIO_ACCOUNT_SID=your_twilio_account_sid
TWILIO_AUTH_TOKEN=your_twilio_auth_token
TWILIO_PHONE_NUMBER=+1234567890
TWILIO_WHATSAPP_NUMBER=+14155238886

# Frontend URL
FRONTEND_URL=https://www.yourdomain.com

# AWS (optional)
AWS_REGION=us-east-1
AWS_ACCESS_KEY_ID=your_aws_key
AWS_SECRET_ACCESS_KEY=your_aws_secret
S3_BUCKET_NAME=cropwise-uploads

# Email (AWS SES)
SMTP_HOST=email-smtp.us-east-1.amazonaws.com
SMTP_PORT=587
SMTP_USER=your_smtp_username
SMTP_PASS=your_smtp_password
EMAIL_FROM=noreply@yourdomain.com

# Logging
LOG_LEVEL=info
LOG_FILE_PATH=logs/

# Rate Limiting
RATE_LIMIT_WINDOW_MS=900000
RATE_LIMIT_MAX_REQUESTS=100

# Security
CORS_ORIGIN=https://www.yourdomain.com
```

#### Frontend Configuration

**File**: `frontend/.env`

```bash
# API URL
VITE_API_URL=https://api.yourdomain.com

# Google OAuth (optional)
VITE_GOOGLE_CLIENT_ID=your_google_client_id.apps.googleusercontent.com

# Feature Flags
VITE_ENABLE_IOT=true
VITE_ENABLE_ANALYTICS=true
```

### Database Configuration

#### PostgreSQL Setup

```bash
# Connect to PostgreSQL
psql -h your-db-host -U postgres

# Create database
CREATE DATABASE cropwise_db;

# Create user
CREATE USER cropwise_admin WITH PASSWORD 'your_password';

# Grant privileges
GRANT ALL PRIVILEGES ON DATABASE cropwise_db TO cropwise_admin;

# Enable extensions (for future AI/ML features)
\c cropwise_db
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pg_trgm";
```

#### Database Migrations

```bash
# Run all migrations
cd backend
npm run migrate

# Rollback last migration
npm run migrate:undo

# Create new migration
npx sequelize-cli migration:generate --name add-new-feature
```

### Web Server Configuration

#### Nginx Configuration

**File**: `/etc/nginx/sites-available/cropwise`

```nginx
# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;
    return 301 https://$server_name$request_uri;
}

# HTTPS Server
server {
    listen 443 ssl http2;
    server_name yourdomain.com www.yourdomain.com;

    # SSL Certificates
    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;

    # SSL Configuration
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;

    # Security Headers
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # Frontend (React app)
    location / {
        root /var/www/cropwise/frontend/dist;
        try_files $uri $uri/ /index.html;
        
        # Cache static assets
        location ~* \.(js|css|png|jpg|jpeg|gif|svg|ico|woff|woff2|ttf|eot)$ {
            expires 1y;
            add_header Cache-Control "public, immutable";
        }
    }

    # Backend API
    location /api {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Timeouts
        proxy_connect_timeout 60s;
        proxy_send_timeout 60s;
        proxy_read_timeout 60s;
    }

    # WebSocket for real-time updates (if implemented)
    location /socket.io {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "Upgrade";
        proxy_set_header Host $host;
    }

    # Health check endpoint
    location /health {
        proxy_pass http://localhost:3000/health;
        access_log off;
    }

    # Logs
    access_log /var/log/nginx/cropwise_access.log;
    error_log /var/log/nginx/cropwise_error.log;
}
```

**Enable site**:
```bash
sudo ln -s /etc/nginx/sites-available/cropwise /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

---

## 🗄️ Database Management

### Backups

#### Automated PostgreSQL Backups

**Create backup script**: `/opt/scripts/backup-cropwise-db.sh`

```bash
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups/cropwise"
DB_NAME="cropwise_db"
DB_USER="cropwise_admin"
DB_HOST="your-db-host"

# Create backup
pg_dump -h $DB_HOST -U $DB_USER -F c -b -v -f "${BACKUP_DIR}/cropwise_${DATE}.backup" $DB_NAME

# Compress
gzip "${BACKUP_DIR}/cropwise_${DATE}.backup"

# Upload to S3 (optional)
aws s3 cp "${BACKUP_DIR}/cropwise_${DATE}.backup.gz" s3://your-backup-bucket/database/

# Keep only last 30 days locally
find $BACKUP_DIR -type f -mtime +30 -delete

# Log
echo "$(date): Backup completed - cropwise_${DATE}.backup.gz" >> /var/log/cropwise-backup.log
```

**Make executable**:
```bash
chmod +x /opt/scripts/backup-cropwise-db.sh
```

**Schedule with cron**: `crontab -e`
```bash
# Daily backup at 2 AM
0 2 * * * /opt/scripts/backup-cropwise-db.sh
```

#### Restore from Backup

```bash
# Restore from local backup
pg_restore -h your-db-host -U cropwise_admin -d cropwise_db -c /backups/cropwise/cropwise_20241114_020000.backup

# Or from S3
aws s3 cp s3://your-backup-bucket/database/cropwise_20241114_020000.backup.gz ./
gunzip cropwise_20241114_020000.backup.gz
pg_restore -h your-db-host -U cropwise_admin -d cropwise_db -c cropwise_20241114_020000.backup
```

### Database Maintenance

#### Vacuum and Analyze

```bash
# Connect to database
psql -h your-db-host -U cropwise_admin -d cropwise_db

# Vacuum (reclaim storage)
VACUUM VERBOSE;

# Analyze (update statistics)
ANALYZE VERBOSE;

# Full vacuum (locks tables, do during maintenance window)
VACUUM FULL VERBOSE;
```

**Automate with cron**:
```bash
# Weekly vacuum on Sunday at 3 AM
0 3 * * 0 psql -h your-db-host -U cropwise_admin -d cropwise_db -c "VACUUM ANALYZE;"
```

#### Index Maintenance

```sql
-- Check for missing indexes
SELECT
  schemaname,
  tablename,
  attname,
  n_distinct,
  correlation
FROM pg_stats
WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY abs(correlation) DESC;

-- Rebuild indexes
REINDEX TABLE batches;
REINDEX TABLE telemetries;

-- Monitor index usage
SELECT
  schemaname,
  tablename,
  indexname,
  idx_scan,
  idx_tup_read,
  idx_tup_fetch
FROM pg_stat_user_indexes
ORDER BY idx_scan ASC;
```

---

## 👥 User Management

### Creating Admin User

```bash
# Via command line (development)
cd backend
node scripts/create-admin.js

# Or via database
psql -h your-db-host -U cropwise_admin -d cropwise_db

INSERT INTO users (email, password_hash, first_name, last_name, role, verified)
VALUES (
  'admin@yourdomain.com',
  '$2a$10$hashed_password_here',
  'Admin',
  'User',
  'admin',
  true
);
```

### Resetting User Password

```bash
# Via script
node scripts/reset-password.js user@example.com

# Or manually update hash in database
psql -h your-db-host -U cropwise_admin -d cropwise_db

UPDATE users
SET password_hash = '$2a$10$new_hashed_password'
WHERE email = 'user@example.com';
```

### Managing Subscriptions

```sql
-- View active subscriptions
SELECT u.email, s.plan, s.status, s.expires_at
FROM users u
JOIN subscriptions s ON u.id = s.user_id
WHERE s.status = 'active';

-- Upgrade user plan
UPDATE subscriptions
SET plan = 'professional', expires_at = expires_at + INTERVAL '1 year'
WHERE user_id = 123;

-- Cancel subscription (soft delete)
UPDATE subscriptions
SET status = 'cancelled', cancelled_at = NOW()
WHERE user_id = 123;
```

---

## 🔒 Security

### SSL/TLS Certificates

#### Let's Encrypt (Free)

```bash
# Install Certbot
sudo apt install certbot python3-certbot-nginx

# Obtain certificate
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Auto-renewal (already set up by certbot)
sudo certbot renew --dry-run
```

### Firewall Configuration

```bash
# UFW (Ubuntu)
sudo ufw allow 22/tcp      # SSH
sudo ufw allow 80/tcp      # HTTP
sudo ufw allow 443/tcp     # HTTPS
sudo ufw allow 1883/tcp    # MQTT (if public)
sudo ufw enable

# Check status
sudo ufw status verbose
```

### Security Headers

Already configured in Nginx. Verify:

```bash
curl -I https://yourdomain.com | grep -i security
```

### JWT Secret Rotation

```bash
# Generate new secret
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"

# Update environment variable
nano backend/.env
# Update JWT_SECRET

# Restart backend
pm2 restart cropwise-backend

# Note: This will invalidate all existing sessions
```

### Regular Security Audits

```bash
# Check for npm vulnerabilities
cd backend
npm audit

cd frontend
npm audit

# Fix vulnerabilities
npm audit fix

# Update dependencies
npm update
```

---

## 💾 Backup & Recovery

### Complete System Backup

**Backup Script**: `/opt/scripts/full-backup.sh`

```bash
#!/bin/bash
DATE=$(date +%Y%m%d)
BACKUP_ROOT="/backups/cropwise"

# 1. Database backup
pg_dump -h your-db-host -U cropwise_admin -F c cropwise_db > "${BACKUP_ROOT}/db_${DATE}.backup"

# 2. Application code (if not in git)
tar -czf "${BACKUP_ROOT}/app_${DATE}.tar.gz" /var/www/cropwise

# 3. Environment files
tar -czf "${BACKUP_ROOT}/env_${DATE}.tar.gz" /var/www/cropwise/backend/.env /var/www/cropwise/frontend/.env

# 4. Uploaded files (if any)
tar -czf "${BACKUP_ROOT}/uploads_${DATE}.tar.gz" /var/www/cropwise/uploads

# 5. Logs
tar -czf "${BACKUP_ROOT}/logs_${DATE}.tar.gz" /var/www/cropwise/backend/logs

# 6. Upload to S3
aws s3 sync $BACKUP_ROOT s3://your-backup-bucket/cropwise/${DATE}/

echo "Backup completed: ${DATE}"
```

### Disaster Recovery Plan

**Recovery Time Objective (RTO)**: 4 hours  
**Recovery Point Objective (RPO)**: 24 hours

**Steps**:
1. Provision new servers (use Infrastructure as Code)
2. Restore database from latest backup
3. Deploy application from git
4. Restore environment files
5. Configure SSL certificates
6. Update DNS records
7. Verify all services
8. Monitor for issues

---

## 📊 Monitoring

### Application Monitoring

#### PM2 Process Manager

```bash
# Install PM2
npm install -g pm2

# Start application
cd /var/www/cropwise/backend
pm2 start src/index.js --name cropwise-backend

# Auto-start on boot
pm2 startup
pm2 save

# Monitor
pm2 monit
pm2 logs cropwise-backend
pm2 status
```

#### Health Checks

```bash
# Backend health
curl http://localhost:3000/health

# Expected response:
# {"status":"healthy","timestamp":"2024-11-14T10:30:00.000Z","uptime":12345}

# Monitor script
watch -n 30 'curl -s http://localhost:3000/health | jq'
```

### Database Monitoring

```sql
-- Active connections
SELECT count(*) FROM pg_stat_activity;

-- Long running queries
SELECT pid, now() - query_start AS duration, query
FROM pg_stat_activity
WHERE state = 'active' AND now() - query_start > interval '5 minutes';

-- Database size
SELECT pg_size_pretty(pg_database_size('cropwise_db'));

-- Table sizes
SELECT schemaname, tablename, pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename))
FROM pg_tables
WHERE schemaname NOT IN ('pg_catalog', 'information_schema')
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC
LIMIT 10;
```

### Log Management

```bash
# View backend logs
tail -f /var/www/cropwise/backend/logs/combined.log

# View error logs
tail -f /var/www/cropwise/backend/logs/error.log

# View Nginx logs
tail -f /var/log/nginx/cropwise_access.log
tail -f /var/log/nginx/cropwise_error.log

# Rotate logs (configure logrotate)
sudo nano /etc/logrotate.d/cropwise
```

**Logrotate Configuration**:
```
/var/www/cropwise/backend/logs/*.log {
    daily
    rotate 30
    compress
    delaycompress
    missingok
    notifempty
    create 0640 www-data www-data
    sharedscripts
    postrotate
        pm2 reloadLogs
    endscript
}
```

---

## ⚡ Performance Tuning

### Database Optimization

**PostgreSQL Configuration**: `/etc/postgresql/15/main/postgresql.conf`

```ini
# Memory Settings
shared_buffers = 4GB
effective_cache_size = 12GB
maintenance_work_mem = 1GB
work_mem = 64MB

# Connection Settings
max_connections = 200

# Query Tuning
random_page_cost = 1.1  # For SSD
effective_io_concurrency = 200

# Logging
log_min_duration_statement = 1000  # Log slow queries (>1s)
```

Restart PostgreSQL:
```bash
sudo systemctl restart postgresql
```

### Application Optimization

#### Enable Compression

Already configured in backend (`compression` middleware).

#### Redis Caching

```javascript
// Implement caching for expensive queries
const getCachedFarms = async (userId) => {
  const cacheKey = `farms:${userId}`;
  
  // Check cache
  let farms = await redis.get(cacheKey);
  if (farms) {
    return JSON.parse(farms);
  }
  
  // Query database
  farms = await Farm.findAll({ where: { user_id: userId } });
  
  // Cache for 5 minutes
  await redis.setex(cacheKey, 300, JSON.stringify(farms));
  
  return farms;
};
```

### Frontend Optimization

```bash
# Build optimized production bundle
cd frontend
npm run build

# Analyze bundle size
npx vite-bundle-visualizer

# Enable gzip in Nginx (already configured)
gzip on;
gzip_types text/plain text/css application/json application/javascript;
```

---

## 🔧 Troubleshooting

### Common Issues

#### Backend Won't Start

```bash
# Check logs
pm2 logs cropwise-backend

# Common causes:
# 1. Port already in use
sudo lsof -i :3000
sudo kill -9 <PID>

# 2. Database connection failure
psql -h your-db-host -U cropwise_admin -d cropwise_db

# 3. Missing environment variables
cat backend/.env

# 4. Node modules issues
cd backend
rm -rf node_modules package-lock.json
npm install
```

#### Database Connection Errors

```bash
# Test connection
psql -h your-db-host -U cropwise_admin -d cropwise_db

# Check PostgreSQL status
sudo systemctl status postgresql

# View PostgreSQL logs
sudo tail -f /var/log/postgresql/postgresql-15-main.log

# Check firewall
sudo ufw status
```

#### High Memory Usage

```bash
# Check memory
free -h
htop

# Check process memory
ps aux --sort=-%mem | head

# Restart services if needed
pm2 restart cropwise-backend
sudo systemctl restart postgresql
sudo systemctl restart redis
```

#### Slow Queries

```sql
-- Enable query logging
ALTER SYSTEM SET log_min_duration_statement = 1000;
SELECT pg_reload_conf();

-- View slow queries
SELECT query, calls, total_time, mean_time
FROM pg_stat_statements
ORDER BY mean_time DESC
LIMIT 10;

-- Add indexes for slow queries
CREATE INDEX idx_batches_zone_status ON batches(zone_id, status);
```

---

## 🔄 Maintenance

### Regular Maintenance Tasks

#### Daily
- [ ] Check application health
- [ ] Monitor error logs
- [ ] Verify backups completed
- [ ] Check disk space

#### Weekly
- [ ] Review slow queries
- [ ] Update security patches
- [ ] Check SSL certificate expiry
- [ ] Vacuum database

#### Monthly
- [ ] Update dependencies
- [ ] Review user feedback
- [ ] Capacity planning
- [ ] Security audit

### Update Procedure

```bash
# 1. Backup everything
/opt/scripts/full-backup.sh

# 2. Pull latest code
cd /var/www/cropwise
git pull origin main

# 3. Update dependencies
cd backend && npm install
cd ../frontend && npm install

# 4. Run migrations
cd backend
npm run migrate

# 5. Build frontend
cd ../frontend
npm run build

# 6. Restart services
pm2 restart cropwise-backend
sudo systemctl reload nginx

# 7. Verify
curl http://localhost:3000/health
```

---

## 📈 Scaling

### Horizontal Scaling

**Load Balancer Setup** (AWS ALB or Nginx):

```nginx
upstream cropwise_backend {
    least_conn;
    server backend1.local:3000;
    server backend2.local:3000;
    server backend3.local:3000;
}

server {
    listen 80;
    location /api {
        proxy_pass http://cropwise_backend;
    }
}
```

### Vertical Scaling

**Upgrade server resources**:
- Increase CPU cores
- Add more RAM
- Upgrade to faster SSD
- Adjust PostgreSQL settings accordingly

### Database Scaling

**Read Replicas**:
```javascript
const sequelize = new Sequelize({
  replication: {
    read: [
      { host: 'read-replica-1.rds.amazonaws.com', username: 'cropwise_admin', password: 'password' },
      { host: 'read-replica-2.rds.amazonaws.com', username: 'cropwise_admin', password: 'password' }
    ],
    write: { host: 'primary.rds.amazonaws.com', username: 'cropwise_admin', password: 'password' }
  }
});
```

---

## 📞 Support

**Technical Support**: admins@cropwise.io  
**Emergency Hotline**: +1-555-EMERGENCY  
**Documentation**: https://docs.cropwise.io  

---

**System Status**: https://status.cropwise.io

*Last reviewed: November 2025*

