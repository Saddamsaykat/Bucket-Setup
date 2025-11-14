### START


# Complete VPS Setup Guide with NVM
## Deploy Next.js & React Projects with NVM, Nginx, PM2, and SSL

---

## Prerequisites
- Hostinger VPS account
- Domain name pointed to your VPS IP
- SSH access to your server
- Git repository URL

---

## Step 1: Update System & Install Required Packages

```bash
sudo apt update && sudo apt upgrade -y
```

### Install Nginx, Git, and Curl
```bash
sudo apt install nginx git curl build-essential -y
```

**Note:** We're NOT installing Node.js and NPM via apt because we'll use NVM instead.

---

## Step 2: Install NVM (Node Version Manager)

### Download and install NVM
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
```

### Load NVM into current session
```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
```

### Verify NVM installation
```bash
nvm --version
```

You should see something like: `0.39.7`

---

## Step 3: Install Node.js using NVM

### Install latest LTS version
```bash
nvm install --lts
```

### Or install specific version
```bash
# Install Node.js 20
nvm install 20

# Install Node.js 18
nvm install 18

# Install Node.js 16
nvm install 16
```

### Set default Node.js version
```bash
nvm alias default 20
```

### Verify installation
```bash
node -v
npm -v
nvm current
```

### List installed Node.js versions
```bash
nvm ls
```

### Switch between Node.js versions
```bash
nvm use 20
# or
nvm use 18
```

---

## Step 4: Create Project Directory

```bash
sudo mkdir -p /var/www/project
```

### Set proper permissions
```bash
sudo chown -R $USER:$USER /var/www/project
```

---

## Step 5: Clone Your Repository

```bash
cd /var/www/project
git clone https://your-repo-url.git my-git-file
```

**Replace:**
- `https://your-repo-url.git` with your actual repository URL
- `my-git-file` with your project folder name

---

## Step 6: Install Project Dependencies

```bash
cd my-git-file
npm install
```

### If you need a specific Node.js version for this project
Create `.nvmrc` file in project root:
```bash
echo "20" > .nvmrc
```

Then use it:
```bash
nvm use
```

---

## Step 7: Build Your Project

### For Next.js:
```bash
npm run build
```

### For React (Vite):
```bash
npm run build
```

---

## Step 8: Install PM2 Process Manager

```bash
npm install pm2@latest -g
```

**Note:** Since we're using NVM, PM2 is installed per Node.js version. No need for `sudo`.

---

## Step 9: Start Application with PM2

### For Next.js:
```bash
pm2 start npm --name "my-app" -- start
```

```bash
pm2 start npm --name "nextjs-app-3001" -- start -- -p 3001
```

### For React (using serve):
```bash
# Install serve globally
npm install -g serve

# Start with PM2
pm2 start serve --name "my-app" -- -s build -l 3000
```

### For React (using Vite preview):
```bash
pm2 start npm --name "my-app" -- run preview
```

---

## Step 10: Save PM2 Configuration

```bash
pm2 save
```

---

## Step 11: Enable PM2 Startup on Boot (with NVM)

This is **CRITICAL** for NVM users - the standard `pm2 startup` won't work properly.

### Step 11a: Get PM2 startup command
```bash
pm2 startup
```

This will output a command like:
```bash
sudo env PATH=$PATH:/home/username/.nvm/versions/node/v20.x.x/bin ...
```

### Step 11b: Run the command it provides
Copy and paste the entire command from the output.

### Step 11c: Verify PM2 will start on boot
```bash
pm2 save
sudo reboot
# After reboot, check:
pm2 list
```

---

## Step 12: Configure Nginx

### Create Nginx configuration file
```bash
sudo nano /etc/nginx/sites-available/my-app
```

### Basic Configuration (Single Frontend)

```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Advanced Configuration (Frontend + Backend)

```nginx
server {
    listen 80;
    server_name yourdomain.com www.yourdomain.com;

    # Frontend
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # Backend API
    location /api/ {
        proxy_pass http://localhost:8000/;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

**Replace:**
- `yourdomain.com` with your actual domain
- Port `3000` with your frontend port
- Port `8000` with your backend port (if applicable)

### Save and exit
Press `CTRL + X`, then `Y`, then `Enter`

---

## Step 13: Enable Nginx Site

```bash
sudo ln -s /etc/nginx/sites-available/my-app /etc/nginx/sites-enabled/
```

---

## Step 14: Test Nginx Configuration

```bash
sudo nginx -t
```

You should see:
```
nginx: configuration file /etc/nginx/nginx.conf test is successful
```

---

## Step 15: Reload Nginx

```bash
sudo systemctl reload nginx
```

Or restart if reload doesn't work:
```bash
sudo systemctl restart nginx
```

---

## Step 16: Install SSL Certificate (Let's Encrypt)

### Install Certbot
```bash
sudo apt install certbot python3-certbot-nginx -y
```

### Obtain SSL Certificate
```bash
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com
```

Follow the prompts:
1. Enter your email address
2. Agree to terms of service
3. Choose whether to redirect HTTP to HTTPS (recommended: Yes)

### Auto-renewal test
```bash
sudo certbot renew --dry-run
```

---

## Step 17: Configure Firewall (Optional but Recommended)

```bash
# Install UFW
sudo apt install ufw -y

# Allow SSH
sudo ufw allow OpenSSH

# Allow HTTP and HTTPS
sudo ufw allow 'Nginx Full'

# Enable firewall
sudo ufw enable

# Check status
sudo ufw status
```

---

## Multiple Projects Setup with Different Node Versions

### Project 1: horizonlinetours.com (Node.js 20)
```bash
cd /var/www/project
git clone https://github.com/user/horizon-tours.git horizon-tours
cd horizon-tours

# Create .nvmrc
echo "20" > .nvmrc

# Use Node.js 20
nvm use

# Install and build
npm install
npm run build

# Start with PM2
pm2 start npm --name "horizon-tours" -- start
```

**Nginx config:** `/etc/nginx/sites-available/horizon-tours`
```nginx
server {
    listen 80;
    server_name horizonlinetours.com www.horizonlinetours.com;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }

    location /api/ {
        proxy_pass http://localhost:8000/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

### Project 2: book-world.nexsoftdev.com (Node.js 18)
```bash
cd /var/www/project
git clone https://github.com/user/book-world.git book-world
cd book-world

# Create .nvmrc
echo "18" > .nvmrc

# Use Node.js 18
nvm use

# Install and build
npm install
npm run build

# Start with PM2
pm2 start npm --name "book-world" -- run preview
```

**Nginx config:** `/etc/nginx/sites-available/book-world`
```nginx
server {
    listen 80;
    server_name book-world.nexsoftdev.com;

    location / {
        proxy_pass http://localhost:5173;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

**Enable both sites:**
```bash
sudo ln -s /etc/nginx/sites-available/horizon-tours /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/book-world /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

**Get SSL for both:**
```bash
sudo certbot --nginx -d horizonlinetours.com -d www.horizonlinetours.com
sudo certbot --nginx -d book-world.nexsoftdev.com
```

**Save PM2 configuration:**
```bash
pm2 save
```

---

## NVM Commands Reference

```bash
# Install latest LTS version
nvm install --lts

# Install specific version
nvm install 20
nvm install 18.17.0

# List installed versions
nvm ls

# List available versions
nvm ls-remote

# Use specific version
nvm use 20
nvm use 18

# Use version from .nvmrc
nvm use

# Set default version
nvm alias default 20

# Current version
nvm current

# Uninstall version
nvm uninstall 16

# Run command with specific version
nvm exec 18 node app.js

# Install global packages for current version
npm install -g pm2 serve
```

---

## PM2 Commands

```bash
# List all applications
pm2 list

# View logs
pm2 logs my-app

# View specific app logs
pm2 logs my-app --lines 100

# Restart application
pm2 restart my-app

# Stop application
pm2 stop my-app

# Delete application
pm2 delete my-app

# Delete all
pm2 delete all

# Monitor applications
pm2 monit

# Save current process list
pm2 save

# Show detailed info
pm2 show my-app

# Flush logs
pm2 flush
```

---

## Nginx Commands

```bash
# Test configuration
sudo nginx -t

# Reload Nginx
sudo systemctl reload nginx

# Restart Nginx
sudo systemctl restart nginx

# Check Nginx status
sudo systemctl status nginx

# View error logs
sudo tail -f /var/log/nginx/error.log

# View access logs
sudo tail -f /var/log/nginx/access.log
```

---

## Updating Your Application

```bash
# Navigate to project
cd /var/www/project/my-git-file

# Ensure correct Node version
nvm use

# Pull latest changes
git pull origin main

# Install new dependencies
npm install

# Rebuild
npm run build

# Restart PM2 application
pm2 restart my-app
```

---

## Creating Deployment Script

Create `deploy.sh` in your project:

```bash
#!/bin/bash

# Load NVM
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

# Navigate to project
cd /var/www/project/my-git-file

# Use correct Node version
nvm use

# Pull latest changes
echo "Pulling latest changes..."
git pull origin main

# Install dependencies
echo "Installing dependencies..."
npm install

# Build project
echo "Building project..."
npm run build

# Restart PM2
echo "Restarting application..."
pm2 restart my-app

echo "Deployment complete!"
```

Make it executable:
```bash
chmod +x deploy.sh
```

Run it:
```bash
./deploy.sh
```

---

## Troubleshooting

### NVM command not found after reboot
Add to `~/.bashrc` or `~/.zshrc`:
```bash
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
```

Then:
```bash
source ~/.bashrc
```

### PM2 not starting on boot with NVM
The PATH in the startup script must include NVM's Node.js path. Run:
```bash
pm2 unstartup
pm2 startup
# Copy and run the command it provides
pm2 save
```

### Different Node versions for different projects
```bash
# Project 1
cd /var/www/project/project1
nvm use 20
pm2 start npm --name "project1" -- start

# Project 2
cd /var/www/project/project2
nvm use 18
pm2 start npm --name "project2" -- start

pm2 save
```

### Port already in use
```bash
# Find process using port
sudo lsof -i :3000

# Kill process
sudo kill -9 <PID>
```

### Nginx 502 Bad Gateway
```bash
# Check if your app is running
pm2 list

# Check PM2 logs
pm2 logs my-app

# Restart your app
pm2 restart my-app
```

### Permission denied errors
```bash
# Change ownership to current user
sudo chown -R $USER:$USER /var/www/project/my-git-file
```

### SSL renewal issues
```bash
# Test renewal
sudo certbot renew --dry-run

# Force renewal
sudo certbot renew --force-renewal
```

---

## Complete Setup Commands (Quick Copy-Paste)

```bash
# 1. Update system
sudo apt update && sudo apt upgrade -y

# 2. Install dependencies
sudo apt install nginx git curl build-essential -y

# 3. Install NVM
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# 4. Load NVM
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

# 5. Install Node.js
nvm install --lts
nvm alias default node

# 6. Verify
node -v
npm -v
nvm --version

# 7. Create project directory
sudo mkdir -p /var/www/project
sudo chown -R $USER:$USER /var/www/project

# 8. Clone repository
cd /var/www/project
git clone https://your-repo-url.git my-git-file
cd my-git-file

# 9. Install dependencies
npm install

# 10. Build project
npm run build

# 11. Install PM2
npm install -g pm2

# 12. Start with PM2
pm2 start npm --name "my-app" -- start

# 13. Save PM2 config
pm2 save

# 14. Setup PM2 startup (run the command it outputs)
pm2 startup

# 15. Configure Nginx
sudo nano /etc/nginx/sites-available/my-app

# 16. Enable site
sudo ln -s /etc/nginx/sites-available/my-app /etc/nginx/sites-enabled/

# 17. Test Nginx
sudo nginx -t

# 18. Reload Nginx
sudo systemctl reload nginx

# 19. Install SSL
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d yourdomain.com -d www.yourdomain.com

# 20. Setup firewall
sudo apt install ufw -y
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable
```

---

## Environment Variables with NVM

Create `.env` file in your project:

```bash
cd /var/www/project/my-git-file
nano .env
```

Example `.env`:
```env
NODE_ENV=production
PORT=3000
DATABASE_URL=your-database-url
API_URL=https://yourdomain.com/api
```

**Important:** Rebuild after changing environment variables:
```bash
nvm use
npm run build
pm2 restart my-app
```

---

## Security Best Practices

1. **Use NVM instead of system Node.js** - Better version control
2. **Change default SSH port**
3. **Disable root login**
4. **Use SSH keys instead of passwords**
5. **Keep system updated:** `sudo apt update && sudo apt upgrade -y`
6. **Enable firewall:** `sudo ufw enable`
7. **Regular backups of /var/www/project**
8. **Monitor logs regularly**
9. **Use `.nvmrc` files** for project-specific Node versions
10. **Don't use `sudo` with npm when using NVM**

---

## Port Configuration Reference

| Service Type | Default Port | Used For |
|--------------|-------------|----------|
| Next.js | 3000 | Production server |
| React (Vite) | 5173 | Preview/Dev server |
| Backend API | 8000, 5000, 4000 | Express/Node API |
| Nginx | 80 (HTTP), 443 (HTTPS) | Web server |

---

## Quick Setup Checklist

- [ ] Update system packages
- [ ] Install Nginx, Git, Curl, Build-essential
- [ ] Install NVM
- [ ] Install Node.js via NVM
- [ ] Set default Node version
- [ ] Create project directory
- [ ] Clone repository
- [ ] Create .nvmrc file (optional)
- [ ] Install dependencies
- [ ] Build project
- [ ] Install PM2 globally
- [ ] Start app with PM2
- [ ] Save PM2 configuration
- [ ] Setup PM2 startup (IMPORTANT for NVM)
- [ ] Configure Nginx
- [ ] Enable Nginx site
- [ ] Test Nginx configuration
- [ ] Reload Nginx
- [ ] Install SSL certificate
- [ ] Configure firewall
- [ ] Test website access
- [ ] Create deployment script

---

## Why Use NVM?

✅ **Multiple Node versions** on the same server
✅ **Easy switching** between versions
✅ **No sudo required** for global npm packages
✅ **Project-specific versions** with .nvmrc
✅ **Clean uninstall** of Node versions
✅ **Better permission management**
✅ **No conflicts** between system Node and project Node

---

## SSH Key Setup for GitHub Actions

### Step 1: Generate SSH Key on Server

```bash
# 1. Go to home directory
cd ~

# 2. Navigate to SSH directory
cd ~/.ssh/

# 3. List files
ls

# 4. View authorized keys (if exists)
cat authorized_keys

# 5. Generate new SSH key
ssh-keygen -t ed25519 -C "github-actions@yourdomain.com"
# Press Enter to accept default location
# Press Enter twice for no passphrase (required for automation)

# 6. List files again
ls

# 7. Display private key (KEEP THIS SECRET!)
cat id_ed25519

# 8. Display public key
cat id_ed25519.pub
```

### Step 2: Add Public Key to Authorized Keys

```bash
# Add your public key to authorized_keys
cat id_ed25519.pub >> authorized_keys

# Set proper permissions
chmod 600 authorized_keys
chmod 700 ~/.ssh
```

### Step 3: Copy Private Key for GitHub Secrets

```bash
cat id_ed25519
```

Copy the entire output (including `-----BEGIN OPENSSH PRIVATE KEY-----` and `-----END OPENSSH PRIVATE KEY-----`)

---

## GitHub Actions Setup

### Step 1: Add GitHub Secrets

Go to your GitHub repository:
1. **Settings** → **Secrets and variables** → **Actions**
2. Click **New repository secret**

Add these secrets:

| Secret Name | Value | Example |
|-------------|-------|---------|
| `SERVER_SSH_KEY` | Private key from `cat id_ed25519` | Entire private key content |
| `SERVER_HOST` | Your server IP or domain | `123.456.789.0` or `srv742002.hstgr.cloud` |
| `SERVER_USER` | SSH username | `root` or `ubuntu` |
| `SERVER_PATH` | Project path on server | `/var/www/project/my-app` |
| `PM2_APP_NAME` | PM2 application name | `my-app` |

### Step 2: Create GitHub Actions Workflow

Create `.github/workflows/deploy.yml` in your repository:

```yaml
name: Auto Deploy to Server

on:
  push:
    branches:
      - main
    paths-ignore:
      - '**.md'
      - '.github/**'

jobs:
  deploy:
    name: Deploy to Production Server
    # Only run if any commit message in the push contains "SERVER_PUSH"
    if: contains(join(github.event.commits.*.message, ' '), 'SERVER_PUSH')
    runs-on: ubuntu-latest

    steps:
      - name: Check out repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20' # or use '18', '24', etc.
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build project
        run: npm run build
        # Build outputs:
        # - Next.js (static): `out/`
        # - Next.js (SSR): `.next/`
        # - Vite/React: `dist/`

      - name: Prepare SSH key
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SERVER_SSH_KEY }}" > ~/.ssh/deploy_key
          chmod 600 ~/.ssh/deploy_key
          ssh-keyscan -H ${{ secrets.SERVER_HOST }} >> ~/.ssh/known_hosts

      - name: Deploy to Server
        run: |
          ssh -i ~/.ssh/deploy_key -o StrictHostKeyChecking=no \
            ${{ secrets.SERVER_USER }}@${{ secrets.SERVER_HOST }} \
            "cd ${{ secrets.SERVER_PATH }} && \
             git pull origin main && \
             source ~/.nvm/nvm.sh && \
             nvm use && \
             npm ci && \
             npm run build"

      - name: Restart PM2 process
        run: |
          ssh -i ~/.ssh/deploy_key -o StrictHostKeyChecking=no \
            ${{ secrets.SERVER_USER }}@${{ secrets.SERVER_HOST }} \
            "source ~/.nvm/nvm.sh && \
             pm2 restart ${{ secrets.PM2_APP_NAME }} || \
             echo 'PM2 app not found or already stopped'"

      - name: Clean up
        if: always()
        run: rm -f ~/.ssh/deploy_key
```

---

## Alternative: Deploy Build Files Only

If you want to deploy only the build files (not use git pull):

```yaml
name: Deploy Build to Server

on:
  push:
    branches:
      - main
    paths-ignore:
      - '**.md'
      - '.github/**'

jobs:
  deploy:
    name: Deploy to Production Server
    if: contains(join(github.event.commits.*.message, ' '), 'SERVER_PUSH')
    runs-on: ubuntu-latest

    steps:
      - name: Check out repository
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Build project
        run: npm run build

      - name: Prepare SSH key
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SERVER_SSH_KEY }}" > ~/.ssh/deploy_key
          chmod 600 ~/.ssh/deploy_key
          ssh-keyscan -H ${{ secrets.SERVER_HOST }} >> ~/.ssh/known_hosts

      - name: Create backup on server
        run: |
          ssh -i ~/.ssh/deploy_key -o StrictHostKeyChecking=no \
            ${{ secrets.SERVER_USER }}@${{ secrets.SERVER_HOST }} \
            "cd ${{ secrets.SERVER_PATH }} && \
             [ -d .next ] && cp -r .next .next.backup || true && \
             [ -d dist ] && cp -r dist dist.backup || true"

      - name: Upload build files
        run: |
          # For Next.js
          rsync -avz -e "ssh -i ~/.ssh/deploy_key -o StrictHostKeyChecking=no" \
            .next/ ${{ secrets.SERVER_USER }}@${{ secrets.SERVER_HOST }}:${{ secrets.SERVER_PATH }}/.next/
          
          # For React/Vite (uncomment if needed)
          # rsync -avz -e "ssh -i ~/.ssh/deploy_key -o StrictHostKeyChecking=no" \
          #   dist/ ${{ secrets.SERVER_USER }}@${{ secrets.SERVER_HOST }}:${{ secrets.SERVER_PATH }}/dist/

      - name: Restart PM2 process
        run: |
          ssh -i ~/.ssh/deploy_key -o StrictHostKeyChecking=no \
            ${{ secrets.SERVER_USER }}@${{ secrets.SERVER_HOST }} \
            "source ~/.nvm/nvm.sh && \
             pm2 restart ${{ secrets.PM2_APP_NAME }}"

      - name: Clean up
        if: always()
        run: rm -f ~/.ssh/deploy_key
```

---

## Example Workflow for Multiple Projects

```yaml
name: Multi-Project Deploy

on:
  push:
    branches:
      - main

jobs:
  deploy-frontend:
    name: Deploy Frontend
    if: contains(join(github.event.commits.*.message, ' '), 'FRONTEND_PUSH')
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm run build
      - name: Deploy
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SERVER_SSH_KEY }}" > ~/.ssh/deploy_key
          chmod 600 ~/.ssh/deploy_key
          ssh-keyscan -H ${{ secrets.SERVER_HOST }} >> ~/.ssh/known_hosts
          ssh -i ~/.ssh/deploy_key ${{ secrets.SERVER_USER }}@${{ secrets.SERVER_HOST }} \
            "cd /var/www/project/frontend && \
             git pull && \
             source ~/.nvm/nvm.sh && nvm use && \
             npm ci && npm run build && \
             pm2 restart frontend"

  deploy-backend:
    name: Deploy Backend
    if: contains(join(github.event.commits.*.message, ' '), 'BACKEND_PUSH')
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
      - run: npm ci
      - name: Deploy
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SERVER_SSH_KEY }}" > ~/.ssh/deploy_key
          chmod 600 ~/.ssh/deploy_key
          ssh-keyscan -H ${{ secrets.SERVER_HOST }} >> ~/.ssh/known_hosts
          ssh -i ~/.ssh/deploy_key ${{ secrets.SERVER_USER }}@${{ secrets.SERVER_HOST }} \
            "cd /var/www/project/backend && \
             git pull && \
             source ~/.nvm/nvm.sh && nvm use && \
             npm ci && \
             pm2 restart backend"
```

---

## How to Use

### Trigger Deployment

Add `SERVER_PUSH` to your commit message:

```bash
git add .
git commit -m "Update homepage layout SERVER_PUSH"
git push origin main
```

Or for multiple projects:
```bash
git commit -m "Update API endpoints BACKEND_PUSH"
git commit -m "Fix navbar issue FRONTEND_PUSH"
```

### View Deployment Status

1. Go to your GitHub repository
2. Click **Actions** tab
3. See running/completed workflows

---

## Deployment Script Variations

### For Next.js SSR (with PM2)
```yaml
- name: Deploy and Restart
  run: |
    ssh -i ~/.ssh/deploy_key ${{ secrets.SERVER_USER }}@${{ secrets.SERVER_HOST }} \
      "cd ${{ secrets.SERVER_PATH }} && \
       git pull origin main && \
       source ~/.nvm/nvm.sh && nvm use && \
       npm ci && \
       npm run build && \
       pm2 restart ${{ secrets.PM2_APP_NAME }}"
```

### For React/Vite (Static Files)
```yaml
- name: Deploy Static Files
  run: |
    rsync -avz --delete -e "ssh -i ~/.ssh/deploy_key" \
      dist/ ${{ secrets.SERVER_USER }}@${{ secrets.SERVER_HOST }}:${{ secrets.SERVER_PATH }}/dist/
    
    ssh -i ~/.ssh/deploy_key ${{ secrets.SERVER_USER }}@${{ secrets.SERVER_HOST }} \
      "pm2 restart ${{ secrets.PM2_APP_NAME }}"
```

### With Database Migrations
```yaml
- name: Run Migrations
  run: |
    ssh -i ~/.ssh/deploy_key ${{ secrets.SERVER_USER }}@${{ secrets.SERVER_HOST }} \
      "cd ${{ secrets.SERVER_PATH }} && \
       source ~/.nvm/nvm.sh && nvm use && \
       npm run migrate"
```

---

## Troubleshooting GitHub Actions

### SSH Connection Failed
```yaml
# Add debug step
- name: Debug SSH Connection
  run: |
    ssh -i ~/.ssh/deploy_key -v ${{ secrets.SERVER_USER }}@${{ secrets.SERVER_HOST }} "echo 'Connected successfully'"
```

### NVM Not Found
```yaml
# Load NVM in every SSH command
- name: Deploy
  run: |
    ssh -i ~/.ssh/deploy_key ${{ secrets.SERVER_USER }}@${{ secrets.SERVER_HOST }} \
      "export NVM_DIR=\"\$HOME/.nvm\" && \
       [ -s \"\$NVM_DIR/nvm.sh\" ] && . \"\$NVM_DIR/nvm.sh\" && \
       cd ${{ secrets.SERVER_PATH }} && \
       nvm use && npm ci && npm run build && pm2 restart app"
```

### Permission Denied
```bash
# On server, check permissions
ls -la ${{ secrets.SERVER_PATH }}
sudo chown -R $USER:$USER ${{ secrets.SERVER_PATH }}
```

### PM2 Command Not Found
```yaml
# Use full path to PM2
- name: Restart PM2
  run: |
    ssh -i ~/.ssh/deploy_key ${{ secrets.SERVER_USER }}@${{ secrets.SERVER_HOST }} \
      "source ~/.nvm/nvm.sh && \
       nvm use && \
       ~/.nvm/versions/node/v20.*/bin/pm2 restart ${{ secrets.PM2_APP_NAME }}"
```

---

## Advanced: Zero-Downtime Deployment

```yaml
- name: Zero-Downtime Deploy
  run: |
    ssh -i ~/.ssh/deploy_key ${{ secrets.SERVER_USER }}@${{ secrets.SERVER_HOST }} << 'EOF'
      cd ${{ secrets.SERVER_PATH }}
      git pull origin main
      source ~/.nvm/nvm.sh && nvm use
      npm ci
      npm run build
      
      # Start new instance
      pm2 start npm --name "app-new" -- start
      sleep 5
      
      # Switch traffic
      pm2 delete app
      pm2 restart app-new --name app
      
      pm2 save
    EOF
```

---

## Complete Example: Real Project

```yaml
name: Deploy to Hostinger VPS

on:
  push:
    branches:
      - main
      - production

jobs:
  deploy:
    name: Deploy Application
    if: contains(join(github.event.commits.*.message, ' '), 'DEPLOY')
    runs-on: ubuntu-latest

    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js 20
        uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Build project
        run: npm run build

      - name: Setup SSH
        run: |
          mkdir -p ~/.ssh
          echo "${{ secrets.SERVER_SSH_KEY }}" > ~/.ssh/deploy_key
          chmod 600 ~/.ssh/deploy_key
          ssh-keyscan -H ${{ secrets.SERVER_HOST }} >> ~/.ssh/known_hosts

      - name: Create backup
        run: |
          ssh -i ~/.ssh/deploy_key ${{ secrets.SERVER_USER }}@${{ secrets.SERVER_HOST }} \
            "cd ${{ secrets.SERVER_PATH }} && \
             tar -czf backup-$(date +%Y%m%d-%H%M%S).tar.gz .next node_modules || true"

      - name: Deploy application
        run: |
          ssh -i ~/.ssh/deploy_key ${{ secrets.SERVER_USER }}@${{ secrets.SERVER_HOST }} \
            "cd ${{ secrets.SERVER_PATH }} && \
             git pull origin main && \
             source ~/.nvm/nvm.sh && \
             nvm use && \
             npm ci --production && \
             npm run build"

      - name: Restart PM2
        run: |
          ssh -i ~/.ssh/deploy_key ${{ secrets.SERVER_USER }}@${{ secrets.SERVER_HOST }} \
            "source ~/.nvm/nvm.sh && pm2 restart ${{ secrets.PM2_APP_NAME }}"

      - name: Health check
        run: |
          sleep 10
          curl -f https://yourdomain.com || exit 1

      - name: Cleanup
        if: always()
        run: rm -f ~/.ssh/deploy_key

      - name: Notify on failure
        if: failure()
        run: echo "Deployment failed! Check logs."
```

---

## Security Best Practices

1. **Never commit private keys** to repository
2. **Use GitHub Secrets** for all sensitive data
3. **Use ed25519 keys** instead of RSA (more secure)
4. **Rotate SSH keys** periodically
5. **Limit SSH key permissions** on server
6. **Use separate deploy keys** per project
7. **Enable branch protection** rules
8. **Review workflow logs** regularly
9. **Use environment-specific secrets**
10. **Clean up SSH keys** after deployment

---

## Support & Resources

- **NVM GitHub:** https://github.com/nvm-sh/nvm
- **Nginx Documentation:** https://nginx.org/en/docs/
- **PM2 Documentation:** https://pm2.keymetrics.io/docs/
- **GitHub Actions:** https://docs.github.com/en/actions
- **Let's Encrypt:** https://letsencrypt.org/
- **Hostinger Tutorials:** https://www.hostinger.com/tutorials/





### END

# PM2 + Vite React Project Setup Guide

## Prerequisites
- Node.js and npm installed
- Project directory: `/var/www/project/Book-World`
- Domain: `book-world.nexsoftdev.com`

---

## Step 1: Install Required Packages

### Install serve globally
```bash
npm install -g serve
```

### Install PM2 globally
```bash
npm install -g pm2
```

---

## Step 2: Configure Vite

Update your `vite.config.js` file:

```javascript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

// https://vitejs.dev/config/
export default defineConfig({
  base: '/',
  plugins: [react()],
  server: {
    allowedHosts: [
      'book-world.nexsoftdev.com',
      '.nexsoftdev.com', // Allows all subdomains
    ],
    host: true,
    port: 5173,
  },
  preview: {
    allowedHosts: [
      'book-world.nexsoftdev.com',
      '.nexsoftdev.com',
    ],
    host: true,
    port: 5173,
  }
})
```

---

## Step 3: Start Application with PM2

Navigate to your project directory:
```bash
cd /var/www/project/Book-World
```

### Option A: Run Development Server (Not recommended for production)
```bash
pm2 start npm --name "book-world-dev" -- run dev -- --host
```

### Option B: Run Production Build (Recommended)
```bash
# Build the project
npm run build

# Start preview server with PM2
pm2 start npm --name "book-world" -- run preview
```

### Option C: Use Serve Package (Alternative)
```bash
# Build the project
npm run build

# Start with serve
pm2 start serve --name "book-world" -- -s dist -l 5173
```

---

## Step 4: PM2 Process Management

### View running processes
```bash
pm2 list
```

### View logs
```bash
pm2 logs book-world-dev
# or
pm2 logs book-world
```

### Restart application
```bash
pm2 restart book-world-dev
```

### Stop application
```bash
pm2 stop book-world-dev
```

### Delete process from PM2
```bash
pm2 delete book-world-dev
```

### Save PM2 process list
```bash
pm2 save
```

### Auto-start PM2 on system reboot
```bash
pm2 startup
# Follow the instructions shown in the terminal
```

---

## Step 5: Create Ecosystem File (Optional but Recommended)

Create `ecosystem.config.js` in your project root:

```javascript
module.exports = {
  apps: [
    {
      name: 'book-world-dev',
      script: 'npm',
      args: 'run dev -- --host',
      cwd: '/var/www/project/Book-World',
      instances: 1,
      autorestart: true,
      watch: false,
      max_memory_restart: '1G',
      env: {
        NODE_ENV: 'development',
        PORT: 5173
      }
    },
    {
      name: 'book-world-prod',
      script: 'npm',
      args: 'run preview',
      cwd: '/var/www/project/Book-World',
      instances: 1,
      autorestart: true,
      watch: false,
      max_memory_restart: '1G',
      env: {
        NODE_ENV: 'production',
        PORT: 5173
      }
    }
  ]
}
```

### Start with ecosystem file
```bash
# For development
pm2 start ecosystem.config.js --only book-world-dev

# For production
npm run build
pm2 start ecosystem.config.js --only book-world-prod
```

---

## Step 6: Nginx Configuration (If using reverse proxy)

If you're using Nginx to proxy to your Vite app:

```nginx
server {
    listen 80;
    server_name book-world.nexsoftdev.com;

    location / {
        proxy_pass http://localhost:5173;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Restart Nginx:
```bash
sudo systemctl restart nginx
```

---

## Troubleshooting

### Port already in use
```bash
# Find process using port 5173
lsof -i :5173
# or
netstat -tulpn | grep 5173

# Kill the process
kill -9 <PID>
```

### PM2 logs showing errors
```bash
pm2 logs book-world-dev --lines 100
```

### Update package.json scripts (Optional)
Add to your `package.json`:
```json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "preview": "vite preview --host",
    "pm2:dev": "pm2 start npm --name book-world-dev -- run dev -- --host",
    "pm2:prod": "npm run build && pm2 start npm --name book-world -- run preview"
  }
}
```

---

## Quick Reference Commands

```bash
# Install dependencies
npm install -g serve pm2

# Navigate to project
cd /var/www/project/Book-World

# Start dev with PM2
pm2 start npm --name "book-world-dev" -- run dev -- --host

# View status
pm2 list

# View logs
pm2 logs book-world-dev

# Restart
pm2 restart book-world-dev

# Save and setup auto-start
pm2 save
pm2 startup

# Access application
http://book-world.nexsoftdev.com
# or
http://localhost:5173
```

---

## Notes

- **Development mode** (`npm run dev`) includes Hot Module Replacement (HMR) but uses more resources
- **Production mode** (`npm run preview`) serves optimized build files
- For actual production deployment, consider using Nginx to serve static files instead of PM2
- Always run `npm run build` before starting production mode
