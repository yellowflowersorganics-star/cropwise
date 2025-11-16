# ⚡ Quick Reference

**Quick commands and snippets for CropWise developers**

---

## 🚀 Quick Start Commands

```bash
# Start development environment
cd backend && npm run dev &
cd frontend && npm run dev

# Start with Docker
docker-compose up -d

# Run tests
npm test

# Build for production
npm run build
```

---

## 📦 NPM Scripts

### Backend

```bash
npm start              # Start production server
npm run dev           # Start development server with nodemon
npm test              # Run tests with Jest
npm run test:watch    # Run tests in watch mode
npm run lint          # Run ESLint
npm run lint:fix      # Fix ESLint errors
npm run migrate       # Run database migrations
npm run migrate:undo  # Undo last migration
npm run seed          # Seed database with test data
```

### Frontend

```bash
npm run dev           # Start Vite dev server
npm run build         # Build for production
npm run preview       # Preview production build
npm test              # Run Vitest tests
npm run lint          # Run ESLint
npm run lint:fix      # Fix ESLint errors
```

---

## 🔀 Git Commands

### Daily Workflow

```bash
# Start new feature
git checkout develop
git pull origin develop
git checkout -b feature/my-feature

# Commit changes
git add .
git commit -m "feat(scope): description"

# Keep branch updated
git fetch origin
git rebase origin/develop

# Push changes
git push origin feature/my-feature

# After PR merge, cleanup
git checkout develop
git pull origin develop
git branch -d feature/my-feature
```

### Commit Message Format

```bash
# Format
<type>(<scope>): <subject>

# Types
feat      # New feature
fix       # Bug fix
docs      # Documentation
style     # Formatting
refactor  # Code restructuring
perf      # Performance improvement
test      # Adding tests
chore     # Build/tooling
ci        # CI/CD changes

# Examples
git commit -m "feat(batches): add QR code generation"
git commit -m "fix(harvest): prevent negative weight values"
git commit -m "docs(api): update endpoint documentation"
```

### Branch Management

```bash
# List branches
git branch -a

# Delete local branch
git branch -d feature/my-feature

# Delete remote branch
git push origin --delete feature/my-feature

# Rename branch
git branch -m old-name new-name

# Switch to branch
git checkout branch-name
git switch branch-name  # Git 2.23+
```

### Stashing

```bash
# Save work in progress
git stash

# List stashes
git stash list

# Apply latest stash
git stash pop

# Apply specific stash
git stash apply stash@{0}

# Drop stash
git stash drop stash@{0}
```

---

## 🗄️ Database Commands

### PostgreSQL

```bash
# Connect to database
psql -U cropwise -d cropwise

# Common psql commands
\l                    # List databases
\c cropwise          # Connect to database
\dt                   # List tables
\d batches            # Describe table
\du                   # List users
\q                    # Quit

# Backup database
pg_dump -U cropwise -d cropwise -F c -f backup.dump

# Restore database
pg_restore -U cropwise -d cropwise -F c backup.dump

# Create database
createdb -U postgres cropwise

# Drop database
dropdb -U postgres cropwise
```

### Sequelize Migrations

```bash
# Create new migration
npx sequelize-cli migration:generate --name migration-name

# Run all migrations
npx sequelize-cli db:migrate

# Undo last migration
npx sequelize-cli db:migrate:undo

# Undo all migrations
npx sequelize-cli db:migrate:undo:all

# Check migration status
npx sequelize-cli db:migrate:status

# Create seeder
npx sequelize-cli seed:generate --name demo-data

# Run all seeders
npx sequelize-cli db:seed:all

# Undo all seeders
npx sequelize-cli db:seed:undo:all
```

---

## 🐳 Docker Commands

### Basic Commands

```bash
# Start all services
docker-compose up -d

# View logs
docker-compose logs -f
docker-compose logs -f backend  # Specific service

# Stop all services
docker-compose down

# Stop and remove volumes
docker-compose down -v

# Restart service
docker-compose restart backend

# Rebuild service
docker-compose up -d --build backend

# Execute command in container
docker-compose exec backend npm run migrate
docker-compose exec postgres psql -U cropwise

# View running containers
docker ps

# View all containers
docker ps -a

# Remove all stopped containers
docker container prune

# Remove all unused images
docker image prune -a
```

### Build and Push Images

```bash
# Build image
docker build -t cropwise/backend:latest ./backend

# Tag image
docker tag cropwise/backend:latest cropwise/backend:v1.0.0

# Push to registry
docker push cropwise/backend:latest

# Pull image
docker pull cropwise/backend:latest

# Login to Docker Hub
docker login

# Login to AWS ECR
aws ecr get-login-password --region ap-south-1 | \
  docker login --username AWS --password-stdin \
  123456789012.dkr.ecr.ap-south-1.amazonaws.com
```

---

## ☁️ AWS CLI Commands

### ECS

```bash
# List clusters
aws ecs list-clusters

# Describe cluster
aws ecs describe-clusters --cluster cropwise-production

# List services
aws ecs list-services --cluster cropwise-production

# Describe service
aws ecs describe-services \
  --cluster cropwise-production \
  --services cropwise-backend

# Update service (deploy new version)
aws ecs update-service \
  --cluster cropwise-production \
  --service cropwise-backend \
  --force-new-deployment

# View running tasks
aws ecs list-tasks --cluster cropwise-production

# Describe task
aws ecs describe-tasks \
  --cluster cropwise-production \
  --tasks TASK_ARN

# Stop task
aws ecs stop-task \
  --cluster cropwise-production \
  --task TASK_ARN
```

### RDS

```bash
# List DB instances
aws rds describe-db-instances

# Describe specific instance
aws rds describe-db-instances \
  --db-instance-identifier cropwise-db

# Create snapshot
aws rds create-db-snapshot \
  --db-instance-identifier cropwise-db \
  --db-snapshot-identifier backup-$(date +%Y%m%d)

# List snapshots
aws rds describe-db-snapshots \
  --db-instance-identifier cropwise-db

# Restore from snapshot
aws rds restore-db-instance-from-db-snapshot \
  --db-instance-identifier cropwise-db-restored \
  --db-snapshot-identifier backup-20250115

# Modify DB instance
aws rds modify-db-instance \
  --db-instance-identifier cropwise-db \
  --db-instance-class db.t3.large \
  --apply-immediately
```

### S3

```bash
# List buckets
aws s3 ls

# List bucket contents
aws s3 ls s3://cropwise-frontend-prod

# Sync directory to S3
aws s3 sync ./dist s3://cropwise-frontend-prod --delete

# Copy file to S3
aws s3 cp file.txt s3://cropwise-frontend-prod/

# Download file from S3
aws s3 cp s3://cropwise-frontend-prod/file.txt ./

# Remove file
aws s3 rm s3://cropwise-frontend-prod/file.txt

# Make bucket public
aws s3api put-bucket-policy \
  --bucket cropwise-frontend-prod \
  --policy file://policy.json
```

### CloudWatch Logs

```bash
# List log groups
aws logs describe-log-groups

# View logs (tail)
aws logs tail /ecs/cropwise-backend --follow

# View logs with time range
aws logs tail /ecs/cropwise-backend \
  --since 1h \
  --follow

# Filter logs
aws logs filter-log-events \
  --log-group-name /ecs/cropwise-backend \
  --filter-pattern "ERROR"

# Create log stream
aws logs create-log-stream \
  --log-group-name /ecs/cropwise-backend \
  --log-stream-name my-stream
```

### Secrets Manager

```bash
# Create secret
aws secretsmanager create-secret \
  --name cropwise/db-password \
  --secret-string "secure_password"

# Get secret
aws secretsmanager get-secret-value \
  --secret-id cropwise/db-password

# Update secret
aws secretsmanager update-secret \
  --secret-id cropwise/db-password \
  --secret-string "new_password"

# Delete secret
aws secretsmanager delete-secret \
  --secret-id cropwise/db-password \
  --force-delete-without-recovery
```

---

## 🧪 Testing Commands

### Backend Testing (Jest)

```bash
# Run all tests
npm test

# Run with coverage
npm test -- --coverage

# Run specific test file
npm test -- batch.test.js

# Run tests matching pattern
npm test -- --testNamePattern="create batch"

# Run tests in watch mode
npm test -- --watch

# Run tests with verbose output
npm test -- --verbose

# Update snapshots
npm test -- --updateSnapshot
```

### Frontend Testing (Vitest)

```bash
# Run tests
npm test

# Run with UI
npm test -- --ui

# Run with coverage
npm test -- --coverage

# Run specific test
npm test -- BatchCard.test.jsx

# Run in watch mode
npm test -- --watch
```

### API Testing (cURL)

```bash
# GET request
curl -X GET http://localhost:8080/api/batches

# GET with auth token
curl -X GET http://localhost:8080/api/batches \
  -H "Authorization: Bearer YOUR_TOKEN"

# POST request
curl -X POST http://localhost:8080/api/batches \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{"batchCode":"BATCH-001","farmId":1}'

# PUT request
curl -X PUT http://localhost:8080/api/batches/1 \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -d '{"status":"completed"}'

# DELETE request
curl -X DELETE http://localhost:8080/api/batches/1 \
  -H "Authorization: Bearer YOUR_TOKEN"

# Pretty print JSON response
curl http://localhost:8080/api/batches | jq '.'
```

---

## 🔍 Debugging Commands

### Node.js Debugging

```bash
# Start with debugger
node --inspect src/index.js

# Start with breakpoint
node --inspect-brk src/index.js

# Debug with nodemon
nodemon --inspect src/index.js
```

### Check Logs

```bash
# Backend logs
tail -f backend/logs/combined.log
tail -f backend/logs/error.log

# Docker logs
docker-compose logs -f backend
docker-compose logs -f --tail=100 backend

# Filter logs with grep
docker-compose logs backend | grep ERROR

# System logs
journalctl -u cropwise-backend -f
```

### Network Debugging

```bash
# Check if port is in use
lsof -i :8080
netstat -tuln | grep 8080

# Test connectivity
telnet localhost 8080
nc -zv localhost 8080

# Test API endpoint
curl -I http://localhost:8080/health

# DNS lookup
nslookup cropwise.io
dig cropwise.io
```

### Process Management

```bash
# Find process by port
lsof -i :8080

# Kill process
kill -9 PID

# View all Node processes
ps aux | grep node

# Monitor system resources
htop
top
```

---

## 📊 Monitoring Commands

### System Monitoring

```bash
# Disk usage
df -h
du -sh *

# Memory usage
free -h

# CPU usage
top
htop

# Docker stats
docker stats

# Check service status
systemctl status cropwise-backend

# View service logs
journalctl -u cropwise-backend -f
```

### Database Monitoring

```bash
# PostgreSQL connections
SELECT * FROM pg_stat_activity;

# Database size
SELECT pg_size_pretty(pg_database_size('cropwise'));

# Table sizes
SELECT 
  schemaname,
  tablename,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename))
FROM pg_tables 
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;

# Long running queries
SELECT 
  pid,
  now() - query_start as duration,
  query
FROM pg_stat_activity
WHERE state = 'active'
ORDER BY duration DESC;

# Kill query
SELECT pg_cancel_backend(PID);
SELECT pg_terminate_backend(PID);
```

---

## 🔐 Security Commands

### Generate Secrets

```bash
# Generate random password
openssl rand -base64 32

# Generate UUID
uuidgen

# Generate JWT secret
node -e "console.log(require('crypto').randomBytes(64).toString('hex'))"

# Hash password (bcrypt)
node -e "console.log(require('bcryptjs').hashSync('password', 10))"
```

### SSL/TLS

```bash
# Generate self-signed certificate
openssl req -x509 -newkey rsa:4096 \
  -keyout key.pem -out cert.pem \
  -days 365 -nodes

# View certificate details
openssl x509 -in cert.pem -text -noout

# Test SSL connection
openssl s_client -connect cropwise.io:443

# Check certificate expiry
echo | openssl s_client -servername cropwise.io \
  -connect cropwise.io:443 2>/dev/null | \
  openssl x509 -noout -dates
```

---

## 📝 Common Tasks

### Add New API Endpoint

```bash
# 1. Create model
# Edit: backend/src/models/ModelName.js

# 2. Create migration
npx sequelize-cli migration:generate --name create-model-table

# 3. Run migration
npm run migrate

# 4. Create controller
# Edit: backend/src/controllers/modelController.js

# 5. Create routes
# Edit: backend/src/routes/model.routes.js

# 6. Register routes
# Edit: backend/src/index.js

# 7. Test
curl http://localhost:8080/api/model
```

### Add New React Page

```bash
# 1. Create page component
# Create: frontend/src/pages/MyPage.jsx

# 2. Add route
# Edit: frontend/src/App.jsx

# 3. Create API service
# Edit: frontend/src/services/myService.js

# 4. Test
npm run dev
# Visit: http://localhost:5173/my-page
```

### Deploy to Production

```bash
# 1. Merge to main
git checkout main
git merge develop
git push origin main

# 2. Tag release
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin v1.0.0

# 3. GitHub Actions deploys automatically
# Or manually:
./scripts/aws-deploy.sh

# 4. Verify deployment
curl https://api.cropwise.io/health
```

---

## 🆘 Troubleshooting Quick Fixes

### Port Already in Use

```bash
# Find process
lsof -i :8080

# Kill process
kill -9 PID

# Or change port in .env
PORT=8081
```

### Database Connection Failed

```bash
# Check PostgreSQL is running
pg_isready

# Restart PostgreSQL
sudo systemctl restart postgresql

# Check credentials in .env
# Verify DB_HOST, DB_PORT, DB_USER, DB_PASSWORD
```

### Module Not Found

```bash
# Clear and reinstall
rm -rf node_modules package-lock.json
npm install

# Clear npm cache
npm cache clean --force
```

### Docker Build Failed

```bash
# Clear Docker cache
docker system prune -a

# Rebuild without cache
docker-compose build --no-cache

# Remove all containers and volumes
docker-compose down -v
docker system prune -a --volumes
```

### Git Merge Conflict

```bash
# View conflicts
git status

# Edit files to resolve conflicts
# Look for <<<<<<< HEAD markers

# After resolving
git add .
git commit -m "resolve: merge conflicts"

# Or abort merge
git merge --abort
```

---

## 📚 Useful Links

### Documentation
- [Developer Guide](./DEVELOPER_GUIDE.md)
- [Deployment Guide](./DEPLOYMENT_GUIDE.md)
- [API Documentation](http://localhost:8080/api-docs)
- [Git Workflow](./GIT_WORKFLOW.md)

### External Resources
- [Node.js Docs](https://nodejs.org/docs)
- [React Docs](https://react.dev)
- [PostgreSQL Manual](https://www.postgresql.org/docs)
- [AWS CLI Reference](https://awscli.amazonaws.com/v2/documentation/api/latest/index.html)

---

## 🔖 Keyboard Shortcuts

### VS Code

```
Ctrl+P          # Quick Open
Ctrl+Shift+P    # Command Palette
Ctrl+`          # Toggle Terminal
Ctrl+B          # Toggle Sidebar
Ctrl+/          # Toggle Comment
Ctrl+Shift+F    # Find in Files
F12             # Go to Definition
Alt+Shift+F     # Format Document
Ctrl+D          # Add Selection to Next Match
```

### Chrome DevTools

```
F12             # Open DevTools
Ctrl+Shift+C    # Inspect Element
Ctrl+Shift+J    # Console
Ctrl+Shift+I    # DevTools
Ctrl+R          # Reload
Ctrl+Shift+R    # Hard Reload
```

### Terminal

```
Ctrl+C          # Stop Process
Ctrl+Z          # Suspend Process
Ctrl+L          # Clear Screen
Ctrl+R          # Search History
Ctrl+A          # Start of Line
Ctrl+E          # End of Line
Ctrl+U          # Clear Line
Tab             # Auto-complete
```

---

**Last Updated**: November 2024  
**Version**: 1.0.0

Need more help? Check the [Developer Guide](./DEVELOPER_GUIDE.md) or ask in #dev-help channel!
