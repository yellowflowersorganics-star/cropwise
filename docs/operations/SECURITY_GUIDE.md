# 🔐 CropWise - Security Guide

Best practices and security measures for CropWise.

---

## 🎯 Security Overview

CropWise implements multiple layers of security to protect your data and systems:

- **Authentication**: JWT tokens + Google OAuth 2.0
- **Authorization**: Role-based access control (RBAC)
- **Encryption**: TLS/HTTPS for data in transit, AES for data at rest
- **Input Validation**: Protection against SQL injection, XSS, CSRF
- **Rate Limiting**: DDoS protection
- **Security Headers**: HSTS, CSP, X-Frame-Options
- **Regular Updates**: Security patches and dependency updates

---

## 🔑 Authentication

### JWT (JSON Web Tokens)

**How it works**:
1. User logs in with email/password
2. Server validates credentials
3. Server generates JWT token (signed with secret key)
4. Client stores token (localStorage or httpOnly cookie)
5. Client sends token with every API request
6. Server verifies token signature and expiration

**Best Practices**:

```bash
# Use strong JWT secret (minimum 32 characters)
JWT_SECRET=$(node -e "console.log(require('crypto').randomBytes(32).toString('hex'))")

# Set appropriate expiration
JWT_EXPIRES_IN=7d  # 7 days

# Rotate JWT secret periodically (every 90 days)
```

**Token Security**:
- ✅ Tokens are signed with HMAC-SHA256
- ✅ Tokens expire after 7 days (configurable)
- ✅ Refresh tokens for long-lived sessions
- ❌ Never expose JWT_SECRET in client code
- ❌ Never commit secrets to Git

### Google OAuth 2.0

**Setup**:
1. Create OAuth credentials in [Google Cloud Console](https://console.cloud.google.com/)
2. Configure authorized redirect URIs
3. Store credentials securely

```bash
# Environment variables
GOOGLE_CLIENT_ID=your_client_id.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=your_client_secret
GOOGLE_CALLBACK_URL=https://yourdomain.com/api/auth/google/callback
```

**Security Benefits**:
- ✅ No password storage (Google handles it)
- ✅ 2FA support (if enabled on Google account)
- ✅ Automatic security updates from Google
- ✅ Federated identity management

---

## 🛡️ Authorization (RBAC)

### User Roles

| Role | Permissions |
|------|-------------|
| **Owner** | Full access + billing management |
| **Admin** | All features except billing |
| **Farm Manager** | Manage batches, harvests, tasks, employees |
| **Technician** | View data, complete tasks, record harvests |
| **Worker** | View assigned tasks, clock in/out |
| **Read-Only** | View-only access (reports, analytics) |

### Permission Checks

```javascript
// Middleware example
const requireRole = (roles) => {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Unauthorized' });
    }
    
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    
    next();
  };
};

// Usage
router.delete('/api/farms/:id', requireRole(['owner', 'admin']), deleteFarm);
```

### Best Practices

- ✅ Follow principle of least privilege
- ✅ Regularly audit user permissions
- ✅ Remove inactive users
- ✅ Use separate accounts for automation
- ✅ Log all permission changes

---

## 🔒 Data Encryption

### In Transit (TLS/HTTPS)

**SSL/TLS Certificate**:
```bash
# Use Let's Encrypt (free)
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# Auto-renewal
sudo certbot renew --dry-run
```

**Nginx Configuration**:
```nginx
server {
    listen 443 ssl http2;
    
    # SSL certificates
    ssl_certificate /etc/letsencrypt/live/yourdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/yourdomain.com/privkey.pem;
    
    # SSL protocols and ciphers
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    ssl_prefer_server_ciphers on;
    
    # HSTS
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
}
```

### At Rest (Database)

**PostgreSQL Encryption**:
```bash
# Enable encryption at rest (AWS RDS)
aws rds modify-db-instance \
    --db-instance-identifier cropwise-db \
    --storage-encrypted \
    --apply-immediately
```

**Password Hashing**:
```javascript
// Use bcrypt for password hashing
const bcrypt = require('bcrypt');

// Hash password (10 salt rounds)
const hashedPassword = await bcrypt.hash(password, 10);

// Verify password
const isValid = await bcrypt.compare(password, hashedPassword);
```

---

## 🚫 Input Validation & Sanitization

### SQL Injection Prevention

**Use parameterized queries**:

```javascript
// ✅ GOOD - Parameterized query (Sequelize)
const batches = await Batch.findAll({
  where: { zone_id: req.params.zoneId }
});

// ❌ BAD - String concatenation
const query = `SELECT * FROM batches WHERE zone_id = ${req.params.zoneId}`;
```

### XSS Prevention

**Sanitize user input**:
```javascript
const xss = require('xss');

// Sanitize HTML input
const cleanDescription = xss(req.body.description);

// Or use Content Security Policy (CSP)
app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'", "'unsafe-inline'"],
    styleSrc: ["'self'", "'unsafe-inline'"],
    imgSrc: ["'self'", 'data:', 'https:'],
  }
}));
```

**React automatically escapes**:
```jsx
// React escapes by default - safe from XSS
<div>{user.description}</div>

// Dangerous - only use if you trust the source
<div dangerouslySetInnerHTML={{ __html: trustedHTML }} />
```

### CSRF Protection

```javascript
// Use CSRF tokens
const csrf = require('csurf');
const csrfProtection = csrf({ cookie: true });

app.use(csrfProtection);

// Include token in forms
<input type="hidden" name="_csrf" value="{{ csrfToken }}" />
```

---

## 🚦 Rate Limiting

**Prevent brute force and DDoS**:

```javascript
const rateLimit = require('express-rate-limit');

// General API rate limiting
const apiLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 minutes
  max: 100, // Max 100 requests per window
  message: 'Too many requests, please try again later.'
});

app.use('/api/', apiLimiter);

// Stricter limits for authentication
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5, // Max 5 login attempts per window
  skipSuccessfulRequests: true
});

app.use('/api/auth/login', authLimiter);
```

---

## 🔍 Security Headers

```javascript
const helmet = require('helmet');

app.use(helmet()); // Default security headers

// Or customize
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'"],
    }
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true
  }
}));
```

**Headers added**:
- `Strict-Transport-Security`: Force HTTPS
- `X-Frame-Options`: Prevent clickjacking
- `X-Content-Type-Options`: Prevent MIME sniffing
- `X-XSS-Protection`: Enable browser XSS filter
- `Content-Security-Policy`: Control resource loading

---

## 📝 Security Logging

**Log security events**:

```javascript
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ filename: 'logs/security.log' })
  ]
});

// Log authentication attempts
logger.info('Login attempt', { 
  email: email, 
  ip: req.ip, 
  success: true 
});

// Log permission violations
logger.warn('Unauthorized access attempt', {
  user: req.user.id,
  resource: '/api/admin/users',
  ip: req.ip
});
```

**Monitor for**:
- Failed login attempts
- Permission violations
- Unusual API usage patterns
- Database errors
- System errors

---

## 🛡️ Firewall Configuration

### UFW (Ubuntu)

```bash
# Enable firewall
sudo ufw enable

# Allow SSH
sudo ufw allow 22/tcp

# Allow HTTP/HTTPS
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp

# Allow PostgreSQL (only from specific IPs)
sudo ufw allow from 10.0.0.0/8 to any port 5432

# Deny all other traffic
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Check status
sudo ufw status verbose
```

---

## 🔐 Secrets Management

### Environment Variables

**Never commit secrets to Git**:

```bash
# ✅ GOOD - Use .env file (add to .gitignore)
DB_PASSWORD=secure_password
JWT_SECRET=random_64_char_string

# ❌ BAD - Hardcoded in code
const dbPassword = "secure_password";
```

### AWS Secrets Manager (Production)

```bash
# Store secret
aws secretsmanager create-secret \
    --name cropwise/database/password \
    --secret-string "super_secure_password"

# Retrieve in application
const AWS = require('aws-sdk');
const secretsManager = new AWS.SecretsManager();

const secret = await secretsManager.getSecretValue({ 
  SecretId: 'cropwise/database/password' 
}).promise();

const dbPassword = JSON.parse(secret.SecretString).password;
```

---

## 🔄 Regular Updates

### Dependency Updates

```bash
# Check for outdated packages
npm outdated

# Update dependencies
npm update

# Security audit
npm audit

# Fix vulnerabilities
npm audit fix
```

### System Updates

```bash
# Ubuntu
sudo apt update
sudo apt upgrade

# Restart services
sudo systemctl restart cropwise-backend
sudo systemctl restart postgresql
sudo systemctl restart nginx
```

---

## 🚨 Security Monitoring

### Intrusion Detection

**Fail2Ban** (block repeated failed logins):

```bash
# Install
sudo apt install fail2ban

# Configure
sudo nano /etc/fail2ban/jail.local
```

Add:
```ini
[sshd]
enabled = true
port = 22
maxretry = 3
bantime = 3600

[nginx-limit-req]
enabled = true
port = 80,443
maxretry = 5
```

### Log Monitoring

**Logwatch** (daily security reports):

```bash
# Install
sudo apt install logwatch

# Send daily reports
sudo logwatch --output mail --mailto admin@yourdomain.com --detail high
```

---

## 🔑 Two-Factor Authentication (2FA)

**Coming in v1.1**:
- TOTP (Time-based One-Time Password)
- SMS verification
- Email verification
- Hardware keys (YubiKey)

**Current workaround**: Use Google OAuth (supports 2FA)

---

## 🐛 Vulnerability Disclosure

### Reporting Security Issues

**Found a security vulnerability?**

**DO NOT** create a public GitHub issue.

Instead:
1. Email: **security@cropwise.io**
2. Include:
   - Description of vulnerability
   - Steps to reproduce
   - Potential impact
   - Suggested fix (optional)

**We commit to**:
- Acknowledge within 24 hours
- Provide status update within 7 days
- Credit you in security advisory (if desired)
- Fix critical issues within 48 hours

---

## ✅ Security Checklist

### Pre-Deployment

- [ ] Change all default passwords
- [ ] Generate new JWT secret
- [ ] Enable HTTPS/TLS
- [ ] Configure firewall
- [ ] Set up automated backups
- [ ] Enable security headers
- [ ] Configure rate limiting
- [ ] Set up logging
- [ ] Run security audit (`npm audit`)
- [ ] Review user permissions

### Post-Deployment

- [ ] Monitor logs daily
- [ ] Update dependencies monthly
- [ ] Review user access quarterly
- [ ] Rotate secrets annually
- [ ] Perform penetration testing
- [ ] Security training for team

### Ongoing

- [ ] Subscribe to security advisories
- [ ] Keep software updated
- [ ] Monitor for unusual activity
- [ ] Regular backups
- [ ] Disaster recovery testing

---

## 📚 Additional Resources

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [Node.js Security Best Practices](https://nodejs.org/en/docs/guides/security/)
- [PostgreSQL Security](https://www.postgresql.org/docs/current/security.html)
- [AWS Security Best Practices](https://aws.amazon.com/security/best-practices/)

---

## 📞 Security Contact

- **Email**: security@cropwise.io
- **PGP Key**: [Download](https://cropwise.io/security/pgp-key.asc)
- **Bug Bounty**: Coming soon

---

**Stay Secure! 🔐**

*Last updated: November 2025*

