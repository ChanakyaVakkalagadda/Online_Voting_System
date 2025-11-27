# AWS EC2 Free Tier Setup Guide for Ansible Deployment

## Step 1: Create AWS Account

1. Go to https://aws.amazon.com/free/
2. Click **"Create a Free Account"**
3. Enter your details:
   - Email address
   - Account name
   - Password
4. **Enter credit/debit card** (Required for verification - won't be charged for free tier)
5. Verify phone number
6. Choose **"Free" support plan**

**Important:** AWS Free Tier includes:
- ✅ 750 hours/month of t2.micro instance (for 12 months)
- ✅ Enough for 1 server running 24/7

---

## Step 2: Launch EC2 Instance

1. **Sign in to AWS Console**: https://console.aws.amazon.com/
2. Select **Region** (top-right) - Choose closest to you:
   - US East (N. Virginia) - `us-east-1`
   - Asia Pacific (Mumbai) - `ap-south-1`
3. Search for **"EC2"** in the search bar and click it
4. Click **"Launch Instance"** (orange button)

### Configure Instance:

**Name and Tags:**
- Name: `election-voting-system`

**Application and OS Images (AMI):**
- Select **"Ubuntu"**
- Choose **"Ubuntu Server 22.04 LTS"**
- Make sure it shows **"Free tier eligible"**

**Instance Type:**
- Select **"t2.micro"** (should be pre-selected)
- Shows **"Free tier eligible"**

**Key Pair (Login):**
- Click **"Create new key pair"**
  - Name: `voting-system-key`
  - Type: **RSA**
  - Format: **`.pem`** (for WSL/Linux) or **`.ppk`** (for PuTTY)
  - Click **"Create key pair"**
- **IMPORTANT:** Save the downloaded `.pem` file safely!
  - Save to: `C:\Users\YourName\.ssh\voting-system-key.pem`

**Network Settings:**
- Click **"Edit"**
- **Auto-assign public IP:** Enable
- **Firewall (security groups):**
  - Create new security group
  - Name: `voting-system-sg`
  - Add these rules:
    1. **SSH** - Port 22 - Source: My IP (for Ansible)
    2. **Custom TCP** - Port 3000 - Source: Anywhere (for your app)
    3. **HTTP** - Port 80 - Source: Anywhere (optional)

**Storage:**
- Keep default: **8 GB gp3** (Free tier eligible)

**Advanced Details:**
- Leave default

5. Click **"Launch Instance"** (orange button)
6. Wait 2-3 minutes for instance to start

---

## Step 3: Get Instance Details

1. Click **"View all instances"**
2. Find your instance (should show "Running")
3. Note down:
   - **Public IPv4 address:** e.g., `54.123.45.67`
   - **Public IPv4 DNS:** e.g., `ec2-54-123-45-67.compute-1.amazonaws.com`

---

## Step 4: Configure SSH Key on Your PC

### In WSL (Ubuntu):

```bash
# Create .ssh directory if it doesn't exist
mkdir -p ~/.ssh

# Copy the key from Windows to WSL
cp /mnt/c/Users/YourName/.ssh/voting-system-key.pem ~/.ssh/

# Set correct permissions (VERY IMPORTANT!)
chmod 600 ~/.ssh/voting-system-key.pem

# Test SSH connection
ssh -i ~/.ssh/voting-system-key.pem ubuntu@YOUR_EC2_PUBLIC_IP
```

Replace `YOUR_EC2_PUBLIC_IP` with actual IP like `54.123.45.67`

If connection works, you'll see:
```
Welcome to Ubuntu 22.04 LTS
```

Type `exit` to close connection.

---

## Step 5: Update Ansible Inventory

Edit `ansible/inventory.ini`:

```ini
[webservers]
production ansible_host=54.123.45.67 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/voting-system-key.pem

[webservers:vars]
ansible_python_interpreter=/usr/bin/python3
```

**Replace `54.123.45.67` with your actual EC2 public IP!**

---

## Step 6: Set Environment Variables

In WSL terminal:

```bash
# Set your Supabase credentials
export VITE_SUPABASE_URL=your_supabase_url
export VITE_SUPABASE_PUBLISHABLE_KEY=your_supabase_key
export VITE_SUPABASE_PROJECT_ID=your_project_id

# Verify they're set
echo $VITE_SUPABASE_URL
```

---

## Step 7: Test Ansible Connection

```bash
# Navigate to ansible directory
cd /mnt/e/3rd\ Year/Odd\ Sem/CICD/Project/election-aura-vote-main/ansible

# Test connection
ansible webservers -m ping

# Expected output:
# production | SUCCESS => {
#     "ping": "pong"
# }
```

---

## Step 8: Deploy with Ansible

```bash
# Run the playbook
ansible-playbook playbook.yml -v

# This will take 5-10 minutes
# You'll see output showing each step:
# - Installing Docker
# - Cloning repository
# - Building image
# - Starting container
```

---

## Step 9: Access Your Application

Once deployment completes:

**Your application URL:**
```
http://YOUR_EC2_PUBLIC_IP:3000
```

Example: `http://54.123.45.67:3000`

---

## Common Issues & Solutions

### Issue 1: Permission Denied (publickey)

```bash
# Fix key permissions
chmod 600 ~/.ssh/voting-system-key.pem
```

### Issue 2: Connection Timeout

- Check Security Group allows your IP for port 22
- Verify instance is "Running"
- Try using Public DNS instead of IP

### Issue 3: Port 3000 Not Accessible

- Check Security Group allows port 3000 from Anywhere
- Wait 2-3 minutes after deployment

### Issue 4: Ansible "Host key verification failed"

```bash
# Disable strict host checking
export ANSIBLE_HOST_KEY_CHECKING=False
ansible-playbook playbook.yml
```

Or add to `ansible.cfg`:
```ini
[defaults]
host_key_checking = False
```

---

## Verify Deployment

### Check if container is running:

```bash
# SSH into server
ssh -i ~/.ssh/voting-system-key.pem ubuntu@YOUR_EC2_IP

# Check Docker containers
docker ps

# Check logs
docker logs election-voting-system

# Exit
exit
```

### Check via Ansible:

```bash
# Check container status
ansible webservers -a "docker ps"

# Check logs
ansible webservers -a "docker logs election-voting-system --tail 20"
```

---

## AWS Free Tier Monitoring

**Important:** Monitor your usage to stay within free tier!

1. AWS Console → **Billing Dashboard**
2. Check **"Free Tier Usage"**
3. Set up **Billing Alerts** (recommended):
   - CloudWatch → Billing → Create Alarm
   - Alert when charges > $1

---

## Stop/Start Instance (Save Free Hours)

### Stop instance when not using:

```bash
# In AWS Console:
EC2 → Instances → Select instance → Instance State → Stop

# Or via AWS CLI:
aws ec2 stop-instances --instance-ids i-1234567890abcdef0
```

### Start when needed:

```bash
# In AWS Console:
EC2 → Instances → Select instance → Instance State → Start

# Note: Public IP will change! Update inventory.ini
```

**Tip:** Use Elastic IP (free if instance is running) to keep same IP.

---

## Useful Commands

### From your PC (WSL):

```bash
# Test connection
ansible webservers -m ping

# Deploy/Update application
ansible-playbook playbook.yml

# Check server status
ansible webservers -a "uptime"
ansible webservers -a "df -h"
ansible webservers -a "free -m"

# Restart application
ansible webservers -a "docker restart election-voting-system"

# View logs
ansible webservers -a "docker logs election-voting-system --tail 50"
```

### SSH to server:

```bash
ssh -i ~/.ssh/voting-system-key.pem ubuntu@YOUR_EC2_IP
```

---

## Clean Up (When Done)

To avoid charges after free tier expires:

1. **Terminate Instance:**
   - EC2 → Instances → Select → Instance State → Terminate
   
2. **Delete Security Group:**
   - EC2 → Security Groups → Select → Actions → Delete

3. **Release Elastic IP** (if created):
   - EC2 → Elastic IPs → Select → Actions → Release

---

## For Evaluation Demo

### What to show:

1. **AWS Console** - Show running EC2 instance
2. **inventory.ini** - Show server configuration
3. **Terminal** - Run ansible-playbook command
4. **Browser** - Show deployed application
5. **Logs** - Show automated deployment steps

### Demo Script:

```bash
# 1. Test connection
ansible webservers -m ping

# 2. Check current status
ansible webservers -a "docker ps"

# 3. Deploy
ansible-playbook playbook.yml -v

# 4. Access application
# Open browser: http://YOUR_EC2_IP:3000
```

---

## Troubleshooting Checklist

- [ ] EC2 instance is running (green dot)
- [ ] Security group allows SSH (port 22) from your IP
- [ ] Security group allows HTTP (port 3000) from anywhere
- [ ] SSH key has correct permissions (600)
- [ ] inventory.ini has correct IP address
- [ ] Environment variables are set
- [ ] Can SSH manually to server
- [ ] Ansible ping works

---

## Cost Optimization

- ✅ Use t2.micro (free tier)
- ✅ Stop instance when not testing
- ✅ Delete snapshots you don't need
- ✅ Monitor free tier usage
- ✅ Set billing alerts

**Free tier is enough for development and demonstration!**

---

## Next Steps After Setup

1. Set up domain name (optional)
2. Configure SSL certificate
3. Set up monitoring
4. Create backup scripts
5. Document everything for evaluation

---

## Support Resources

- AWS Free Tier FAQ: https://aws.amazon.com/free/
- EC2 Documentation: https://docs.aws.amazon.com/ec2/
- Ansible Documentation: https://docs.ansible.com/
- Your project README: `ansible/ANSIBLE_DEPLOYMENT.md`

---

## Quick Reference

```bash
# Always work in WSL (not PowerShell!)
wsl

# Navigate to project
cd /mnt/e/3rd\ Year/Odd\ Sem/CICD/Project/election-aura-vote-main

# Set environment variables
export VITE_SUPABASE_URL=your_url
export VITE_SUPABASE_PUBLISHABLE_KEY=your_key
export VITE_SUPABASE_PROJECT_ID=your_id

# Deploy
cd ansible
ansible-playbook playbook.yml

# Access app
# http://YOUR_EC2_IP:3000
```
