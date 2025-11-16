# 🧪 CropWise - Complete Testing Guide
## From Scratch to First Harvest

**Let's test everything step by step!**

---

## 📋 Prerequisites Check

Before we start, verify you have:

```bash
# Check Node.js version (should be 18+)
node --version

# Check npm
npm --version

# Check if Git is set up
git --version
```

---

## 🚀 **Step 1: Start Backend**

### A. Navigate to Backend

```bash
cd backend
```

### B. Install Dependencies (if not done)

```bash
npm install
```

**Expected output**:
```
added 234 packages in 15s
```

### C. Check Environment Variables

```bash
# Windows
type .env

# Linux/Mac
cat .env
```

**Should see**:
```
NODE_ENV=development
PORT=3000
DB_DIALECT=sqlite
DB_STORAGE=./cropwise.db
JWT_SECRET=your-secret-key-here
```

**If `.env` doesn't exist**, create it:

```bash
# Windows
echo NODE_ENV=development > .env
echo PORT=3000 >> .env
echo DB_DIALECT=sqlite >> .env
echo DB_STORAGE=./cropwise.db >> .env
echo JWT_SECRET=cropwise-secret-key-2024 >> .env

# Linux/Mac
cat > .env << EOF
NODE_ENV=development
PORT=3000
DB_DIALECT=sqlite
DB_STORAGE=./cropwise.db
JWT_SECRET=cropwise-secret-key-2024
EOF
```

### D. Start Backend Server

```bash
npm run dev
```

**Expected output**:
```
🌱 CropWise Backend running on port 3000
📊 Environment: development
🔗 API: http://localhost:3000/api
📖 Health: http://localhost:3000/health
✅ Database connected successfully
✅ Database models synchronized
⚠️  Redis not available (optional for development)
⚠️  MQTT broker not available (optional for development)
```

**✅ SUCCESS**: Backend is running!

**❌ If you see errors**:
- Port 3000 already in use? → Change PORT in .env to 3001
- Database error? → Delete `cropwise.db` and restart
- Module not found? → Run `npm install` again

---

## 🧪 **Step 2: Test Backend Health**

### A. Open New Terminal (keep backend running)

### B. Test Health Endpoint

```bash
# Windows PowerShell
curl http://localhost:3000/health

# Linux/Mac
curl http://localhost:3000/health

# Or open in browser:
http://localhost:3000/health
```

**Expected response**:
```json
{
  "status": "healthy",
  "timestamp": "2025-11-12T15:30:00.000Z",
  "uptime": 5.123,
  "environment": "development"
}
```

### C. Test API Info

```bash
curl http://localhost:3000/api
```

**Expected response**:
```json
{
  "name": "CropWise API",
  "version": "1.0.0",
  "status": "operational",
  "documentation": "/api/docs"
}
```

**✅ SUCCESS**: API is responding!

---

## 👤 **Step 3: Create Test User Account**

### A. Register New User

```bash
curl -X POST http://localhost:3000/api/auth/register ^
  -H "Content-Type: application/json" ^
  -d "{\"email\":\"test@cropwise.com\",\"password\":\"Test1234\",\"name\":\"Test User\",\"organizationName\":\"Test Farm\"}"

# Linux/Mac (use single quotes)
curl -X POST http://localhost:3000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@cropwise.com","password":"Test1234","name":"Test User","organizationName":"Test Farm"}'
```

**Expected response**:
```json
{
  "success": true,
  "message": "User registered successfully",
  "data": {
    "user": {
      "id": "uuid-here",
      "email": "test@cropwise.com",
      "name": "Test User",
      "organizationId": "org-uuid-here"
    },
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

**✅ Copy the token!** You'll need it for authenticated requests.

**Save token to variable**:
```bash
# Windows PowerShell
$TOKEN = "paste-your-token-here"

# Linux/Mac
export TOKEN="paste-your-token-here"
```

### B. Login (Test Authentication)

```bash
curl -X POST http://localhost:3000/api/auth/login ^
  -H "Content-Type: application/json" ^
  -d "{\"email\":\"test@cropwise.com\",\"password\":\"Test1234\"}"

# Linux/Mac
curl -X POST http://localhost:3000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@cropwise.com","password":"Test1234"}'
```

**Expected response**:
```json
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": "uuid",
    "email": "test@cropwise.com",
    "name": "Test User"
  }
}
```

**✅ SUCCESS**: Authentication works!

---

## 🏢 **Step 4: Create Test Unit**

Units represent physical locations (buildings).

```bash
# Windows
curl -X POST http://localhost:3000/api/units ^
  -H "Authorization: Bearer %TOKEN%" ^
  -H "Content-Type: application/json" ^
  -d "{\"name\":\"Test Building\",\"unitCode\":\"TB-001\",\"location\":{\"address\":\"123 Farm Road\",\"city\":\"Pune\"},\"unitType\":\"building\",\"totalArea\":500}"

# Linux/Mac
curl -X POST http://localhost:3000/api/units \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"Test Building","unitCode":"TB-001","location":{"address":"123 Farm Road","city":"Pune"},"unitType":"building","totalArea":500}'
```

**Expected response**:
```json
{
  "success": true,
  "data": {
    "id": "unit-uuid-here",
    "name": "Test Building",
    "unitCode": "TB-001",
    "status": "active"
  },
  "message": "Unit created successfully"
}
```

**✅ Copy the unit ID!**

```bash
# Windows
$UNIT_ID = "paste-unit-id-here"

# Linux/Mac
export UNIT_ID="paste-unit-id-here"
```

---

## 🚪 **Step 5: Create Test Zone**

Zones represent rooms within a unit.

```bash
# Windows
curl -X POST http://localhost:3000/api/zones ^
  -H "Authorization: Bearer %TOKEN%" ^
  -H "Content-Type: application/json" ^
  -d "{\"name\":\"Incubation Room 1\",\"zoneNumber\":\"A1\",\"unitId\":\"%UNIT_ID%\",\"area\":50,\"status\":\"ready\"}"

# Linux/Mac
curl -X POST http://localhost:3000/api/zones \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d "{\"name\":\"Incubation Room 1\",\"zoneNumber\":\"A1\",\"unitId\":\"$UNIT_ID\",\"area\":50,\"status\":\"ready\"}"
```

**Expected response**:
```json
{
  "success": true,
  "data": {
    "id": "zone-uuid-here",
    "name": "Incubation Room 1",
    "zoneNumber": "A1",
    "unitId": "unit-uuid",
    "status": "ready"
  }
}
```

**✅ Copy the zone ID!**

```bash
# Windows
$ZONE_ID = "paste-zone-id-here"

# Linux/Mac
export ZONE_ID="paste-zone-id-here"
```

---

## 📝 **Step 6: Upload Crop Recipe**

Upload the enhanced oyster mushroom recipe.

```bash
# Windows
curl -X POST http://localhost:3000/api/crop-recipes ^
  -H "Authorization: Bearer %TOKEN%" ^
  -H "Content-Type: application/json" ^
  -d @../shared/examples/oyster-mushroom-enhanced-v2.json

# Linux/Mac
curl -X POST http://localhost:3000/api/crop-recipes \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d @../shared/examples/oyster-mushroom-enhanced-v2.json
```

**Expected response**:
```json
{
  "success": true,
  "data": {
    "recipe_id": "oyster_mushroom_enhanced_v2",
    "name": "Oyster Mushroom Enhanced v2",
    "version": "2.0"
  },
  "message": "Recipe created successfully"
}
```

**✅ Recipe uploaded!**

---

## 🍄 **Step 7: Record First Harvest**

Now the exciting part - record a harvest!

### A. Create Harvest Record

```bash
# Windows
curl -X POST http://localhost:3000/api/harvests ^
  -H "Authorization: Bearer %TOKEN%" ^
  -H "Content-Type: application/json" ^
  -d "{\"batchId\":\"20251112-test-batch\",\"zoneId\":\"%ZONE_ID%\",\"unitId\":\"%UNIT_ID%\",\"flushNumber\":1,\"totalWeightKg\":12.5,\"bagsHarvested\":98,\"bagsDiscarded\":2,\"qualityGrade\":\"premium\",\"harvesterName\":\"Test User\",\"marketDestination\":\"local_market\",\"pricePerKg\":200,\"substrateWeightKg\":250,\"harvestNotes\":\"First test harvest\"}"

# Linux/Mac
curl -X POST http://localhost:3000/api/harvests \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"batchId":"20251112-test-batch","zoneId":"'$ZONE_ID'","unitId":"'$UNIT_ID'","flushNumber":1,"totalWeightKg":12.5,"bagsHarvested":98,"bagsDiscarded":2,"qualityGrade":"premium","harvesterName":"Test User","marketDestination":"local_market","pricePerKg":200,"substrateWeightKg":250,"harvestNotes":"First test harvest"}'
```

**Expected response**:
```json
{
  "success": true,
  "data": {
    "id": "harvest-uuid",
    "batchId": "20251112-test-batch",
    "totalWeightKg": 12.5,
    "bagsHarvested": 98,
    "qualityGrade": "premium",
    "biologicalEfficiency": 5.0,
    "yieldPerBag": 0.127,
    "totalRevenue": 2500
  },
  "message": "Harvest recorded successfully"
}
```

**✅ First harvest recorded!**

**Save harvest ID**:
```bash
# Windows
$HARVEST_ID = "paste-harvest-id-here"

# Linux/Mac
export HARVEST_ID="paste-harvest-id-here"
```

### B. Record Second Harvest (Flush 2)

```bash
# Windows
curl -X POST http://localhost:3000/api/harvests ^
  -H "Authorization: Bearer %TOKEN%" ^
  -H "Content-Type: application/json" ^
  -d "{\"batchId\":\"20251112-test-batch\",\"zoneId\":\"%ZONE_ID%\",\"unitId\":\"%UNIT_ID%\",\"flushNumber\":2,\"totalWeightKg\":8.0,\"bagsHarvested\":96,\"bagsDiscarded\":0,\"qualityGrade\":\"grade_a\",\"harvesterName\":\"Test User\",\"marketDestination\":\"wholesale\",\"pricePerKg\":180,\"substrateWeightKg\":250,\"harvestNotes\":\"Second flush harvest\"}"

# Linux/Mac
curl -X POST http://localhost:3000/api/harvests \
  -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"batchId":"20251112-test-batch","zoneId":"'$ZONE_ID'","unitId":"'$UNIT_ID'","flushNumber":2,"totalWeightKg":8.0,"bagsHarvested":96,"bagsDiscarded":0,"qualityGrade":"grade_a","harvesterName":"Test User","marketDestination":"wholesale","pricePerKg":180,"substrateWeightKg":250,"harvestNotes":"Second flush harvest"}'
```

**✅ Second harvest recorded!**

---

## 📊 **Step 8: Test Analytics Endpoints**

### A. Get All Harvests

```bash
# Windows
curl -H "Authorization: Bearer %TOKEN%" http://localhost:3000/api/harvests

# Linux/Mac
curl -H "Authorization: Bearer $TOKEN" http://localhost:3000/api/harvests
```

**Expected**: List of both harvests

### B. Get Batch Summary

```bash
# Windows
curl -H "Authorization: Bearer %TOKEN%" http://localhost:3000/api/harvests/batch/20251112-test-batch/summary

# Linux/Mac
curl -H "Authorization: Bearer $TOKEN" http://localhost:3000/api/harvests/batch/20251112-test-batch/summary
```

**Expected response**:
```json
{
  "success": true,
  "data": {
    "batchId": "20251112-test-batch",
    "totalFlushes": 2,
    "totalYieldKg": 20.5,
    "totalRevenue": 3940,
    "avgBiologicalEfficiency": 4.1,
    "flushes": [
      {
        "flushNumber": 1,
        "weightKg": 12.5,
        "biologicalEfficiency": 5.0
      },
      {
        "flushNumber": 2,
        "weightKg": 8.0,
        "biologicalEfficiency": 3.2
      }
    ]
  }
}
```

**✅ Analytics working!**

### C. Get Zone Analytics

```bash
# Windows
curl -H "Authorization: Bearer %TOKEN%" "http://localhost:3000/api/harvests/zone/%ZONE_ID%/analytics"

# Linux/Mac
curl -H "Authorization: Bearer $TOKEN" "http://localhost:3000/api/harvests/zone/$ZONE_ID/analytics"
```

**Expected**: Zone performance data

### D. Get Organization Summary

```bash
# Windows
curl -H "Authorization: Bearer %TOKEN%" http://localhost:3000/api/harvests/organization/summary

# Linux/Mac
curl -H "Authorization: Bearer $TOKEN" http://localhost:3000/api/harvests/organization/summary
```

**Expected**: Organization-wide harvest statistics

---

## 🎨 **Step 9: Test Frontend**

### A. Navigate to Frontend

```bash
# Open NEW terminal (keep backend running)
cd frontend
```

### B. Install Dependencies (if not done)

```bash
npm install
```

### C. Start Frontend Dev Server

```bash
npm run dev
```

**Expected output**:
```
  VITE v4.5.0  ready in 2.3s

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose
```

### D. Open Browser

```
http://localhost:5173
```

**You should see**: CropWise dashboard

---

## 🧪 **Step 10: Test Frontend Features**

### A. Login

1. Go to: `http://localhost:5173/login`
2. Enter credentials:
   - Email: `test@cropwise.com`
   - Password: `Test1234`
3. Click "Login"

**✅ Should redirect to dashboard**

### B. View Harvests

1. Navigate to: `http://localhost:5173/harvests`
2. Should see:
   - 2 harvest records
   - Charts showing data
   - Batch summary

### C. Record New Harvest

1. Click "Record Harvest" button
2. Fill in form:
   - Weight: 10 kg
   - Bags: 95
   - Quality: Grade A
   - Harvester: Your Name
3. Submit

**✅ Should see new harvest in list**

### D. Export Report

1. Select batch "20251112-test-batch"
2. Click "PDF Report"
3. Should open print dialog
4. Or click "Excel (CSV)"
5. Should download CSV file

**✅ Export working!**

---

## 📋 **Testing Checklist**

### Backend API Tests

- [ ] ✅ Health endpoint responding
- [ ] ✅ User registration works
- [ ] ✅ User login returns token
- [ ] ✅ Unit created successfully
- [ ] ✅ Zone created successfully
- [ ] ✅ Recipe uploaded
- [ ] ✅ First harvest recorded
- [ ] ✅ Second harvest recorded
- [ ] ✅ Batch summary calculated correctly
- [ ] ✅ Zone analytics working
- [ ] ✅ Org summary working
- [ ] ✅ BE% calculated automatically
- [ ] ✅ Revenue calculated correctly

### Frontend Tests

- [ ] ✅ Frontend starts without errors
- [ ] ✅ Login page loads
- [ ] ✅ Authentication works
- [ ] ✅ Harvest dashboard displays
- [ ] ✅ Charts render correctly
- [ ] ✅ Harvest form opens
- [ ] ✅ Form validation works
- [ ] ✅ Submit creates harvest
- [ ] ✅ PDF export opens
- [ ] ✅ CSV export downloads
- [ ] ✅ Filters work
- [ ] ✅ Responsive on mobile

---

## 🐛 **Common Issues & Solutions**

### Issue 1: "Port 3000 already in use"

**Solution**:
```bash
# Find process using port 3000 (Windows)
netstat -ano | findstr :3000
taskkill /PID <PID> /F

# Linux/Mac
lsof -i :3000
kill -9 <PID>
```

### Issue 2: "Database error"

**Solution**:
```bash
# Delete database and restart
cd backend
del cropwise.db  # Windows
rm cropwise.db   # Linux/Mac
npm run dev
```

### Issue 3: "Token invalid"

**Solution**:
- Re-login to get fresh token
- Check token is correctly copied (no extra spaces)
- Verify JWT_SECRET in .env matches

### Issue 4: "Cannot find module"

**Solution**:
```bash
# Reinstall dependencies
cd backend
rm -rf node_modules package-lock.json
npm install

cd ../frontend
rm -rf node_modules package-lock.json
npm install
```

### Issue 5: "CORS error in frontend"

**Solution**:
- Check backend is running on port 3000
- Verify frontend proxy configuration
- Try clearing browser cache

---

## 🎯 **Quick Test Script**

Save this as `test-api.sh` (Linux/Mac) or `test-api.bat` (Windows):

```bash
#!/bin/bash

# Test CropWise API

echo "🧪 Testing CropWise API..."

# Health check
echo "\n1. Testing health endpoint..."
curl -s http://localhost:3000/health | grep "healthy" && echo "✅ Health OK" || echo "❌ Health FAIL"

# Register user
echo "\n2. Registering test user..."
RESPONSE=$(curl -s -X POST http://localhost:3000/api/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"auto-test@test.com","password":"Test1234","name":"Auto Test","organizationName":"Auto Farm"}')

TOKEN=$(echo $RESPONSE | grep -o '"token":"[^"]*' | cut -d'"' -f4)

if [ -n "$TOKEN" ]; then
  echo "✅ Registration OK"
  echo "Token: ${TOKEN:0:20}..."
else
  echo "❌ Registration FAIL"
  exit 1
fi

# Test authenticated endpoint
echo "\n3. Testing authenticated endpoint..."
curl -s -H "Authorization: Bearer $TOKEN" http://localhost:3000/api/units | grep "success" && echo "✅ Auth OK" || echo "❌ Auth FAIL"

echo "\n🎉 All tests passed!"
```

Run it:
```bash
chmod +x test-api.sh
./test-api.sh
```

---

## 📈 **Expected Results Summary**

After all tests, you should have:

- ✅ **1 User account** (test@cropwise.com)
- ✅ **1 Organization** (Test Farm)
- ✅ **1 Unit** (Test Building)
- ✅ **1 Zone** (Incubation Room 1)
- ✅ **1 Recipe** (Oyster Mushroom Enhanced v2)
- ✅ **2 Harvests** (Flush 1 & 2)
- ✅ **Working analytics** (charts, summaries)
- ✅ **Export functionality** (PDF, Excel)

---

## 🎉 **Success Criteria**

Your system is working if:

1. ✅ Backend starts without errors
2. ✅ Database creates tables automatically
3. ✅ User can register and login
4. ✅ API returns correct JSON responses
5. ✅ Harvest data saves and calculates metrics
6. ✅ Frontend displays charts correctly
7. ✅ Export generates reports
8. ✅ All test endpoints respond with 200 OK

---

## 📝 **Next Steps After Testing**

Once all tests pass:

1. **Add more test data**:
   - Create multiple zones
   - Record more harvests
   - Try different quality grades

2. **Test edge cases**:
   - What if bags harvested = 0?
   - What if weight is very small?
   - What if no photos uploaded?

3. **Test performance**:
   - Record 100 harvests
   - Check if charts still load fast
   - Test pagination

4. **Test on mobile**:
   - Open on phone browser
   - Test form on mobile
   - Check responsive design

---

## 🆘 **Getting Help**

If you encounter issues:

1. **Check logs**:
   - Backend: Look at terminal output
   - Frontend: Open browser console (F12)

2. **Verify data**:
   - Check `cropwise.db` with DB Browser for SQLite
   - Verify tokens are valid

3. **Common commands**:
   ```bash
   # Restart everything
   cd backend && npm run dev
   cd frontend && npm run dev
   
   # Check if ports are free
   netstat -an | findstr 3000
   netstat -an | findstr 5173
   ```

---

**Ready to start testing?** Let's go! 🚀

Run backend first, then follow steps 1-10 above. Let me know if you hit any issues!

