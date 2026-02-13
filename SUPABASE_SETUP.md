# Supabase Configuration for Password Reset

## Critical Setup Steps for Email Password Reset

### 1. Add Redirect URLs to Supabase

Go to your Supabase Dashboard:
https://app.supabase.com/project/zajxliunquqazmuiagfl/auth/url-configuration

Add the following **Redirect URLs**:

```
http://localhost:5173/auth/reset-password
http://localhost:3000/auth/reset-password
https://chanakyavakkalagadda.github.io/Online_Voting_System/auth/reset-password
```

⚠️ **IMPORTANT:** The GitHub Pages URL **MUST** include the `/Online_Voting_System` base path!

### 2. Configure Email Service

#### Option A: Using Supabase's Built-in Email (Development)
1. Go to **Authentication** → **Settings** 
2. Ensure **Email** provider is enabled
3. For production, configure custom SMTP below

#### Option B: Custom SMTP (Production - Recommended)
1. Go to **Authentication** → **Settings** → **SMTP Settings**
2. Enable **Custom SMTP**
3. Configure with your provider:

**Gmail Example:**
- Host: `smtp.gmail.com`
- Port: `587`
- Username: `your-email@gmail.com`
- Password: [Use App Password, not regular password]
- Sender Email: `your-email@gmail.com`
- Sender Name: `Online Voting System`

**SendGrid Example:**
- Host: `smtp.sendgrid.net`
- Port: `587`
- Username: `apikey`
- Password: [Your SendGrid API Key]
- Sender Email: `noreply@yourdomain.com`
- Sender Name: `Online Voting System`

### 3. Verify Email Template

Go to **Authentication** → **Email Templates** → **Reset Password**

Ensure the template contains:
```html
<h2>Reset Password Request</h2>
<p>Click the link below to reset your password:</p>
<p><a href="{{ .ConfirmationURL }}">Reset Password</a></p>
<p>If you didn't request this, you can safely ignore this email.</p>
<p>This link expires in 1 hour.</p>
```

### 4. Set Environment Variable for Deployment

When building for GitHub Pages, ensure:
```bash
VITE_DEPLOY_TARGET=github-pages
```

This is typically set in your GitHub Actions workflow or build script.

### 5. Test the Flow

1. **Local Development:**
   - Run: `npm run dev`
   - Go to: `http://localhost:5173/auth/forgot-password`
   - Enter email and submit
   - Check email for reset link
   - Link should go to: `http://localhost:5173/auth/reset-password`

2. **GitHub Pages (Production):**
   - Go to: `https://chanakyavakkalagadda.github.io/Online_Voting_System/auth/forgot-password`
   - Enter email and submit
   - Check email for reset link
   - Link should go to: `https://chanakyavakkalagadda.github.io/Online_Voting_System/auth/reset-password`

### Troubleshooting

**Problem:** Email not received
- Check Supabase SMTP configuration
- Check spam folder
- Verify email address is correct
- Check Supabase logs: **Authentication** → **Logs**

**Problem:** Reset link gives 404 error
- Verify redirect URL is whitelisted in Supabase
- Ensure URL includes base path for GitHub Pages
- Clear browser cache

**Problem:** "Invalid or expired reset link" error
- Links expire after 1 hour by default
- Request a new reset link
- Ensure you're clicking the latest email

**Problem:** Works locally but not on GitHub Pages
- Verify `VITE_DEPLOY_TARGET=github-pages` is set during build
- Check that redirect URL includes `/Online_Voting_System` path
- Verify the URL is whitelisted in Supabase

### GitHub Actions Build Configuration

If using GitHub Actions, ensure your workflow sets the environment variable:

```yaml
- name: Build
  run: npm run build
  env:
    VITE_DEPLOY_TARGET: github-pages
    VITE_SUPABASE_URL: ${{ secrets.VITE_SUPABASE_URL }}
    VITE_SUPABASE_PUBLISHABLE_KEY: ${{ secrets.VITE_SUPABASE_PUBLISHABLE_KEY }}
    VITE_SUPABASE_PROJECT_ID: ${{ secrets.VITE_SUPABASE_PROJECT_ID }}
```

### Security Notes

- Reset links expire after 1 hour (Supabase default)
- Only the user's email receives the reset link
- Old links are invalidated when a new one is requested
- SMTP credentials should be kept secure
- Never commit `.env` files to git
