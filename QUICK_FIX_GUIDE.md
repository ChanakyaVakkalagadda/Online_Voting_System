# 🚨 QUICK FIX: Password Reset on GitHub Pages

## What Was Fixed

✅ **Fixed Files:**
1. `src/pages/auth/ForgotPassword.tsx` - Now uses correct redirect URL with base path
2. `src/pages/auth/Register.tsx` - Now uses correct redirect URL for email confirmation
3. `.env.example` - Added `VITE_DEPLOY_TARGET` documentation
4. Created `SUPABASE_SETUP.md` - Complete Supabase configuration guide

## ⚡ Immediate Action Required

### Step 1: Add Redirect URLs to Supabase (CRITICAL!)

1. **Go to Supabase Dashboard:**
   ```
   https://app.supabase.com/project/zajxliunquqazmuiagfl/auth/url-configuration
   ```

2. **Click "Add URL" and add these THREE URLs:**
   ```
   http://localhost:5173/auth/reset-password
   http://localhost:3000/auth/reset-password
   https://chanakyavakkalagadda.github.io/Online_Voting_System/auth/reset-password
   ```
   
   ⚠️ **IMPORTANT:** The last URL must include `/Online_Voting_System` - this is the GitHub Pages base path!

3. **Also add these for email confirmation (Register):**
   ```
   http://localhost:5173/
   http://localhost:3000/
   https://chanakyavakkalagadda.github.io/Online_Voting_System/
   ```

4. **Click "Save"**

### Step 2: Configure Email Service (If Not Done)

1. **Go to:** 
   ```
   https://app.supabase.com/project/zajxliunquqazmuiagfl/settings/auth
   ```

2. **Scroll to "SMTP Settings"**

3. **Choose ONE option:**

   **Option A: Use Supabase Default (Quick Test)**
   - Already enabled by default
   - Limited to 4 emails/hour
   - May go to spam

   **Option B: Custom SMTP (Recommended for Production)**
   - Enable "Custom SMTP"
   - Use Gmail, SendGrid, or AWS SES
   - See `SUPABASE_SETUP.md` for detailed instructions

### Step 3: Verify Email Template

1. **Go to:**
   ```
   https://app.supabase.com/project/zajxliunquqazmuiagfl/auth/templates
   ```

2. **Click "Reset Password" template**

3. **Ensure it's enabled** (toggle should be ON)

4. **Save**

### Step 4: Rebuild and Deploy

1. **Commit the changes:**
   ```bash
   git add .
   git commit -m "Fix password reset redirect URL for GitHub Pages"
   git push origin master
   ```

2. **Wait for GitHub Actions to deploy** (2-3 minutes)

3. **Check deployment:**
   - Go to: `https://github.com/ChanakyaVakkalagadda/Online_Voting_System/actions`
   - Wait for green checkmark ✓

### Step 5: Test the Fix

1. **On GitHub Pages:**
   - Go to: `https://chanakyavakkalagadda.github.io/Online_Voting_System/auth/forgot-password`
   - Enter your email
   - Submit

2. **Check Email:**
   - Look in inbox AND spam folder
   - Click the reset link
   - Should redirect to: `https://chanakyavakkalagadda.github.io/Online_Voting_System/auth/reset-password`

3. **Reset Password:**
   - Enter new password
   - Confirm password
   - Submit
   - Should redirect to login page

## Why It Wasn't Working Before

### Problem:
```javascript
// OLD CODE (Wrong!)
redirectTo: `${window.location.origin}/auth/reset-password`
// This generated: https://chanakyavakkalagadda.github.io/auth/reset-password
//                  ❌ Missing /Online_Voting_System/ base path!
```

### Solution:
```javascript
// NEW CODE (Correct!)
const basePath = import.meta.env.VITE_DEPLOY_TARGET === 'github-pages' ? '/Online_Voting_System' : '';
const redirectUrl = `${window.location.origin}${basePath}/auth/reset-password`;
// This generates: https://chanakyavakkalagadda.github.io/Online_Voting_System/auth/reset-password
//                  ✅ Correct URL with base path!
```

## Verification Checklist

After completing all steps, verify:

- [ ] All 6 redirect URLs added to Supabase (3 for reset, 3 for register)
- [ ] Email template is enabled in Supabase
- [ ] SMTP is configured (default or custom)
- [ ] Code changes committed and pushed
- [ ] GitHub Actions deployment completed successfully
- [ ] Can receive password reset email on GitHub Pages
- [ ] Reset link redirects to correct URL (includes /Online_Voting_System/)
- [ ] Can successfully reset password
- [ ] Works on different devices/laptops

## Still Having Issues?

### Issue: Email not received
- Check spam folder
- Wait 5-10 minutes
- Check Supabase logs: **Authentication** → **Logs**
- Verify SMTP configuration

### Issue: 404 Error when clicking reset link
- Verify redirect URL is whitelisted in Supabase
- Check URL includes `/Online_Voting_System/` base path
- Clear browser cache and try again

### Issue: "Invalid or expired reset link"
- Links expire after 1 hour
- Request a new reset link
- Make sure you're clicking the latest email

### Issue: Works locally but not on GitHub Pages
- Verify GitHub Actions build has `VITE_DEPLOY_TARGET: github-pages`
- Check `.github/workflows/deploy.yml` line 36
- Rebuild and redeploy

## Need More Help?

See `SUPABASE_SETUP.md` for detailed configuration guide.
