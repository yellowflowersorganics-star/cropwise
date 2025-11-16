# Google OAuth Setup for CropWise

## Prerequisites
- Google Cloud Platform account
- CropWise backend running

## Step 1: Create Google Cloud Project

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select an existing one
3. Enable the Google+ API

## Step 2: Create OAuth 2.0 Credentials

1. Navigate to **APIs & Services** → **Credentials**
2. Click **Create Credentials** → **OAuth client ID**
3. Select **Web application**
4. Configure:
   - **Name**: CropWise
   - **Authorized JavaScript origins**:
     - `http://localhost:8080` (development)
     - `https://yourdomain.com` (production)
   - **Authorized redirect URIs**:
     - `http://localhost:3000/api/auth/google/callback` (development)
     - `https://api.yourdomain.com/api/auth/google/callback` (production)
5. Click **Create**
6. Copy your **Client ID** and **Client Secret**

## Step 3: Configure Environment Variables

Add the following to your `backend/.env` file:

```env
# Google OAuth Configuration
GOOGLE_CLIENT_ID=your-google-client-id-here.apps.googleusercontent.com
GOOGLE_CLIENT_SECRET=your-google-client-secret-here
GOOGLE_REDIRECT_URI=http://localhost:3000/api/auth/google/callback

# Frontend URL (for OAuth callbacks)
FRONTEND_URL=http://localhost:8080
```

## Step 4: Restart Backend

```bash
cd backend
npm start
```

## Testing

1. Go to `http://localhost:8080/login`
2. Click **"Continue with Google"**
3. You should be redirected to Google's login page
4. After authentication, you'll be redirected back to the dashboard

## Production Deployment

For production:
1. Update the redirect URIs in Google Cloud Console
2. Update environment variables:
   ```env
   GOOGLE_REDIRECT_URI=https://api.yourdomain.com/api/auth/google/callback
   FRONTEND_URL=https://yourdomain.com
   ```
3. Ensure HTTPS is enabled

## Security Notes

- Never commit `.env` files to version control
- Use different OAuth credentials for development and production
- Regularly rotate your client secrets
- Monitor OAuth usage in Google Cloud Console

## Troubleshooting

### Error: "redirect_uri_mismatch"
- Ensure the redirect URI in your `.env` matches exactly what's configured in Google Cloud Console
- Include the protocol (`http://` or `https://`)

### Error: "invalid_client"
- Check that your Client ID and Client Secret are correct
- Ensure there are no extra spaces or quotes in the `.env` file

### User creation fails
- Check backend logs for detailed error messages
- Ensure database migrations have run
- Verify the User model includes the `googleId` field

