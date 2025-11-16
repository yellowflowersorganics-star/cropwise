# 🔧 Registration Error - Quick Fix Guide

## Problem
You're seeing "Registration failed" when trying to create an account.

## ✅ What I Just Fixed

I've updated the backend to use **SQLite** instead of PostgreSQL, so you don't need to install any database! The backend will now:

1. ✅ Use SQLite (no PostgreSQL needed)
2. ✅ Auto-create database tables
3. ✅ Work without Redis or MQTT for testing
4. ✅ Auto-install sqlite3 driver

## 🚀 How to Start the Backend

### Option 1: Using npm (Recommended)

```bash
# In the backend folder
cd backend
npm run dev
```

You should see:
```
✅ Database connected successfully
✅ Database models synchronized
🌱 CropWise Backend running on port 3000
📊 Environment: development
🔗 API: http://localhost:3000/api
```

### Option 2: If you see errors

1. **Stop any running backend** (Ctrl+C in the terminal)

2. **Clear and restart**:
```bash
cd backend
rm -f cropwise.db  # Remove old database
npm run dev
```

3. **Check the console** for any errors

## 🧪 Test the Backend

Open a new terminal and test:

```bash
# Test health endpoint
curl http://localhost:3000/health

# Should return:
# {"status":"healthy","timestamp":"...","uptime":...}
```

## 🎨 Start the Frontend

In a **separate terminal**:

```bash
cd frontend
npm run dev
```

Open: http://localhost:8080

## ✅ Try Registration Again

Now try creating an account with:
- **First Name**: John
- **Last Name**: Doe  
- **Email**: john@example.com
- **Password**: password123 (min 8 characters)

## 🐛 Still Not Working?

### Check Backend Logs

Look in the backend terminal for errors. Common issues:

1. **Port 3000 already in use**
   ```bash
   # Windows: Find and kill process on port 3000
   netstat -ano | findstr :3000
   taskkill /PID <PID> /F
   ```

2. **Module not found errors**
   ```bash
   cd backend
   rm -rf node_modules
   npm install
   npm run dev
   ```

3. **Database errors**
   ```bash
   cd backend
   rm -f cropwise.db  # Start fresh
   npm run dev
   ```

### Check Frontend Logs

Look in the frontend terminal. Common issues:

1. **Can't connect to backend**
   - Make sure backend is running on port 3000
   - Check browser console (F12) for errors

2. **Port 8080 already in use**
   ```bash
   # The error will suggest an alternative port
   # Open that URL instead
   ```

## 📊 Expected Behavior

### Backend Console Should Show:
```
✅ Database connected successfully
✅ Database models synchronized
⚠️  Redis not available (optional for development)
⚠️  MQTT broker not available (optional for development)

🌱 CropWise Backend running on port 3000
📊 Environment: development
🔗 API: http://localhost:3000/api
📖 Health: http://localhost:3000/health
```

### When You Register:
- Backend logs should show: `POST /api/auth/register 201`
- Frontend should show: "User registered successfully"
- You should be logged in automatically

## 🎯 Quick Verification Checklist

- [ ] Backend terminal is running `npm run dev`
- [ ] You see "CropWise Backend running on port 3000"
- [ ] Frontend terminal is running `npm run dev`  
- [ ] Browser is open to http://localhost:8080
- [ ] You see the login/register page
- [ ] Try registration with valid data

## 💡 Pro Tips

1. **Keep both terminals visible** so you can see errors
2. **Check browser console (F12)** for frontend errors
3. **The database file** is created at `backend/cropwise.db`
4. **First user** becomes admin automatically

## 🆘 Need More Help?

If you're still stuck, provide me with:

1. Backend terminal output (last 20 lines)
2. Frontend terminal output (last 20 lines)
3. Browser console errors (F12 → Console tab)
4. The exact error message you see

---

## ✅ What Changed

### Files Modified:
1. `backend/src/config/database.js` - Now uses SQLite by default
2. `backend/src/index.js` - MQTT/Redis are now optional
3. `backend/src/models/index.js` - Added new models
4. `backend/package.json` - Added sqlite3 dependency

### Why This is Better:
- ✅ No PostgreSQL installation needed
- ✅ No Redis installation needed
- ✅ No MQTT broker needed for testing
- ✅ Works out of the box
- ✅ Database auto-creates tables
- ✅ Perfect for development

You can switch to PostgreSQL later for production by just changing the config!

---

**Try it now and let me know if registration works! 🚀**

