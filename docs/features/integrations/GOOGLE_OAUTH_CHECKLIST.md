# ✅ Google OAuth Setup Checklist

Use this checklist to verify your Google OAuth setup is complete.

## 🔍 Pre-Flight Checklist

### Google Cloud Console

- [ ] Created Google Cloud Project "CropWise"
- [ ] Configured OAuth Consent Screen (External)
- [ ] Added scopes: email, profile, openid
- [ ] Created OAuth 2.0 Client ID (Web application)
- [ ] Added authorized JavaScript origins:
  - [ ] http://localhost:8080
  - [ ] http://localhost:3000
- [ ] Added authorized redirect URIs:
  - [ ] http://localhost:3000/api/auth/google/callback
  - [ ] http://localhost:8080/auth/google/success
- [ ] Copied Client ID and Client Secret

### Backend Configuration

- [ ] Opened `backend/.env` file
- [ ] Added `GOOGLE_CLIENT_ID=...`
- [ ] Added `GOOGLE_CLIENT_SECRET=...`
- [ ] Added `GOOGLE_REDIRECT_URI=http://localhost:3000/api/auth/google/callback`
- [ ] Added `FRONTEND_URL=http://localhost:8080`
- [ ] Saved the file
- [ ] Restarted backend server

### Testing

- [ ] Backend is running on port 3000
- [ ] Frontend is running on port 8080
- [ ] Visited http://localhost:8080/login
- [ ] Can see "Continue with Google" button
- [ ] Clicked the button
- [ ] Google login page appeared
- [ ] Successfully logged in
- [ ] Redirected to dashboard

## 🎯 Quick Test Commands

```powershell
# Check if backend is running
Test-NetConnection -ComputerName localhost -Port 3000

# Check if frontend is running
Test-NetConnection -ComputerName localhost -Port 8080

# View backend logs
cd backend
npm start

# Test login page
Start-Process http://localhost:8080/login
```

## ✅ If Everything Works

You should see:
1. ✅ "Continue with Google" button on login page
2. ✅ Google OAuth consent screen when clicked
3. ✅ Redirect to dashboard after login
4. ✅ Your name and email displayed in sidebar

**Success!** 🎉 Google OAuth is working!

## ❌ If Something Doesn't Work

See the troubleshooting section below.

---

## 🆘 Troubleshooting Guide

### Issue 1: "Continue with Google" button doesn't work

**Symptoms:**
- Button appears but clicking does nothing
- Console shows error

**Solutions:**

**A. Check Environment Variables**
```powershell
cd backend
Get-Content .env | Select-String -Pattern "GOOGLE"
```

Should show:
```
GOOGLE_CLIENT_ID=xxxxx.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=GOCSPX-xxxxx
```

**B. Restart Backend**
```powershell
# Stop backend
Get-Process node | Stop-Process -Force

# Start again
cd backend
npm start
```

---

### Issue 2: "Redirect URI mismatch" error

**Symptoms:**
- Google shows error: "redirect_uri_mismatch"
- Cannot complete login

**Solutions:**

**A. Check Redirect URI in Google Console**

1. Go to: https://console.cloud.google.com/apis/credentials
2. Click your OAuth Client ID
3. Under "Authorized redirect URIs", verify you have:
   ```
   http://localhost:3000/api/auth/google/callback
   ```
4. Make sure there are NO typos or extra spaces
5. Click "SAVE" if you made changes
6. Wait 5 minutes for changes to propagate

**B. Check Backend .env**
```powershell
cd backend
notepad .env
```

Verify:
```env
GOOGLE_REDIRECT_URI=http://localhost:3000/api/auth/google/callback
```

Must match EXACTLY with Google Console (including http://, no https://)

---

### Issue 3: "Access blocked: This app's request is invalid"

**Symptoms:**
- Google shows: "Access blocked: This app's request is invalid"

**Solutions:**

**A. Add Test User**

1. Go to: https://console.cloud.google.com/apis/credentials/consent
2. Scroll to "Test users"
3. Click "+ ADD USERS"
4. Add your Gmail address
5. Click "SAVE"

**B. Check OAuth Consent Screen**

1. Go to: https://console.cloud.google.com/apis/credentials/consent
2. Status should be: "Testing" (this is OK for development)
3. Make sure scopes include:
   - .../auth/userinfo.email
   - .../auth/userinfo.profile

---

### Issue 4: "Invalid Client ID" error

**Symptoms:**
- Backend logs show: "Invalid client id"
- Can't initiate OAuth flow

**Solutions:**

**A. Verify Client ID Format**

Client ID should look like:
```
123456789-abc123def456.apps.googleusercontent.com
```

**NOT like:**
```
123456789  (too short)
GOCSPX-xxx  (this is the secret, not ID)
```

**B. Re-copy from Google Console**

1. Go to: https://console.cloud.google.com/apis/credentials
2. Click your OAuth Client ID name
3. Copy "Client ID" (right side)
4. Paste into backend/.env
5. Restart backend

---

### Issue 5: Login works but user not created

**Symptoms:**
- Google login succeeds
- But can't access dashboard
- Database error in logs

**Solutions:**

**A. Check Database Connection**
```powershell
cd backend
npm start
```

Look for: "✅ Database connected successfully"

**B. Run Database Migrations**
```powershell
cd backend
npm run migrate
```

**C. Check User Model**

The User model should have a `googleId` field. This was added in recent updates.

---

### Issue 6: "Network Error" or "CORS Error"

**Symptoms:**
- Frontend can't reach backend
- Console shows CORS error

**Solutions:**

**A. Check Backend is Running**
```powershell
Test-NetConnection -ComputerName localhost -Port 3000
```

Should show: "TcpTestSucceeded : True"

**B. Check Frontend API URL**
```powershell
cd frontend
Get-Content src/services/api.js
```

Should have:
```javascript
baseURL: 'http://localhost:3000'
```

---

### Issue 7: Works on localhost but not production

**Symptoms:**
- Google OAuth works locally
- Fails when deployed to AWS/production

**Solutions:**

**A. Update Authorized Origins in Google Console**

Add your production URLs:
```
https://yourdomain.com
https://api.yourdomain.com
```

**B. Update Redirect URIs**
```
https://api.yourdomain.com/api/auth/google/callback
https://yourdomain.com/auth/google/success
```

**C. Update Environment Variables**

Production .env should have:
```env
GOOGLE_REDIRECT_URI=https://api.yourdomain.com/api/auth/google/callback
FRONTEND_URL=https://yourdomain.com
```

**D. Use HTTPS (not HTTP)**

Google OAuth requires HTTPS in production.

---

## 📞 Still Having Issues?

### Check These Common Mistakes:

1. ❌ **Typo in redirect URI** - Must match EXACTLY
2. ❌ **Using HTTPS locally** - Use HTTP for localhost
3. ❌ **Wrong Client Secret** - Make sure you copied the right one
4. ❌ **Project not selected** - Select the right project in Google Console
5. ❌ **Didn't restart backend** - Always restart after .env changes
6. ❌ **Cache issues** - Clear browser cache or use incognito mode

### Debug Commands:

```powershell
# View backend environment variables (will show loaded config)
cd backend
node -e "require('dotenv').config(); console.log(process.env.GOOGLE_CLIENT_ID)"

# Test Google OAuth endpoint directly
curl http://localhost:3000/api/auth/google

# Check backend logs
cd backend/logs
Get-Content combined.log -Tail 50
```

### Get Help:

1. Check backend logs: `backend/logs/combined.log`
2. Check browser console for errors (F12)
3. Verify all URLs match between Google Console and .env
4. Try in incognito mode to rule out cache issues

---

## ✅ Verification Steps

Run these to verify everything:

```powershell
# 1. Check backend is running
Test-NetConnection -ComputerName localhost -Port 3000

# 2. Check frontend is running  
Test-NetConnection -ComputerName localhost -Port 8080

# 3. Check environment variables are loaded
cd backend
node -e "require('dotenv').config(); console.log('Client ID:', process.env.GOOGLE_CLIENT_ID ? 'SET ✅' : 'NOT SET ❌')"

# 4. Test OAuth endpoint
curl http://localhost:3000/api/auth/google -L | Select-String "accounts.google.com"
```

All checks should pass ✅

---

## 🎉 Success Criteria

Your Google OAuth is working correctly if:

✅ "Continue with Google" button appears  
✅ Clicking redirects to Google login  
✅ After login, redirects to dashboard  
✅ User profile shows in sidebar  
✅ Logout works correctly  
✅ Can login again with same Google account  
✅ New users are automatically created  

**All working?** Congratulations! 🎊

---

## 📚 Additional Resources

- [Official Setup Guide](../backend/GOOGLE_OAUTH_SETUP.md)
- [Google OAuth Documentation](https://developers.google.com/identity/protocols/oauth2)
- [CropWise GitHub Issues](https://github.com/your-repo/issues)

---

**Last Updated:** 2024
**Status:** Complete ✅

