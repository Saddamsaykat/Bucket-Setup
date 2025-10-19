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
