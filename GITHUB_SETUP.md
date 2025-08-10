# GitHub Repository Setup & SSH Deployment Guide

This guide covers setting up your GitHub repository and deploying the Supabase MCP Server to your cloud instance via SSH.

## 🚀 Part 1: GitHub Repository Setup

### Step 1: Create GitHub Repository

1. **Go to GitHub** and create a new repository:
   - Repository name: `supabase-mcp-server`
   - Description: `A Model Context Protocol server for Supabase database operations`
   - Visibility: Public or Private (your choice)
   - ✅ Add README file: **NO** (we already have one)
   - ✅ Add .gitignore: **NO** (we already have one)
   - ✅ Choose a license: MIT License (recommended)

2. **Copy the repository URL** (you'll need this):
   ```
   https://github.com/YOUR_USERNAME/supabase-mcp-server.git
   ```

### Step 2: Initialize and Push to GitHub

Run these commands in your local project directory:

```bash
# Navigate to your project directory
cd /path/to/supabase-mcp-server

# Initialize git repository
git init

# Add all files to git
git add .

# Create initial commit
git commit -m "Initial commit: Supabase MCP Server with Docker deployment"

# Add GitHub repository as remote origin
git remote add origin https://github.com/YOUR_USERNAME/supabase-mcp-server.git

# Push to GitHub
git push -u origin main
```

### Step 3: Verify Repository

Your GitHub repository should now contain:
```
📁 supabase-mcp-server/
├── 🐳 Docker files (Dockerfile, docker-compose.yml)
├── 📝 Documentation (README.md, DEPLOYMENT.md)
├── ⚙️ Configuration (.env.example, pyproject.toml)
├── 💻 Source code (src/, tests/)
├── 🚀 Scripts (scripts/deploy.sh)
└── 📋 Project docs (PLANNING.md, TASK.md)
```

---

## 📡 Part 2: SSH Deployment to Cloud Instance

### Prerequisites

Before starting, ensure you have:
- ✅ Cloud server running (Ubuntu 20.04+, CentOS 8+, or Debian 11+)
- ✅ SSH access to your server
- ✅ Server has Docker and Docker Compose installed
- ✅ Your Supabase URL and service role key ready

### Step 1: Connect to Your Cloud Instance

```bash
# SSH into your cloud server
ssh username@your-server-ip

# Example:
# ssh ubuntu@203.0.113.10
# ssh root@your-domain.com
# ssh user@my-server.example.com
```

### Step 2: Install Prerequisites (if needed)

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install Docker (if not installed)
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add your user to docker group
sudo usermod -aG docker $USER

# Install Docker Compose (if not installed)
sudo apt install docker-compose-plugin -y

# Verify installation
docker --version
docker compose version

# Log out and back in for group changes to take effect
exit
# SSH back in
ssh username@your-server-ip
```

### Step 3: Clone Repository and Deploy

```bash
# Clone your GitHub repository
git clone https://github.com/YOUR_USERNAME/supabase-mcp-server.git

# Navigate to project directory
cd supabase-mcp-server

# Make deployment script executable
chmod +x scripts/deploy.sh

# Run automated deployment
./scripts/deploy.sh
```

### Step 4: Configure Environment

The deployment script will guide you through configuration:

1. **Choose deployment type**:
   - Option 1: MCP Server only (recommended if you have existing Supabase)
   - Option 2: Full stack (deploy MCP + self-hosted Supabase)

2. **When prompted, edit the .env file**:
   ```bash
   # The script will create .env from template
   # Edit it with your actual Supabase credentials:
   nano .env
   ```

3. **Add your credentials**:
   ```env
   SUPABASE_URL=https://your-project-id.supabase.co
   SUPABASE_SERVICE_ROLE_KEY=your-actual-service-role-key-here
   LOG_LEVEL=INFO
   ```

4. **Save and continue**: Press `Ctrl+X`, then `Y`, then `Enter`

### Step 5: Verify Deployment

After deployment completes:

```bash
# Check if containers are running
docker ps

# View server logs
docker-compose -f docker-compose.mcp-only.yml logs mcp-server

# Check server health
curl -f http://localhost:8085/health || echo "Health endpoint not available (normal for MCP servers)"

# Monitor resource usage
docker stats --no-stream
```

### Step 6: Configure Firewall (Important!)

```bash
# Open port 8085 for MCP server
sudo ufw allow 8085/tcp

# If using full stack, also open:
sudo ufw allow 3000/tcp  # Supabase Studio
sudo ufw allow 8000/tcp  # Supabase API

# Enable firewall if not already enabled
sudo ufw --force enable

# Check firewall status
sudo ufw status
```

---

## 🔧 Alternative: Manual Deployment

If you prefer manual deployment:

### Option A: MCP Server Only (Recommended)

```bash
# Clone repository
git clone https://github.com/YOUR_USERNAME/supabase-mcp-server.git
cd supabase-mcp-server

# Create environment file
cp .env.production .env
nano .env  # Edit with your credentials

# Deploy
docker-compose -f docker-compose.mcp-only.yml up -d

# Check status
docker-compose -f docker-compose.mcp-only.yml ps
```

### Option B: Full Stack Deployment

```bash
# Clone repository
git clone https://github.com/YOUR_USERNAME/supabase-mcp-server.git
cd supabase-mcp-server

# Create environment file with additional variables
cp .env.production .env
nano .env  # Configure all variables including Postgres passwords

# Deploy full stack
docker-compose up -d

# Check status
docker-compose ps
```

---

## 📊 Post-Deployment Management

### View Logs
```bash
# MCP server logs
docker-compose -f docker-compose.mcp-only.yml logs -f mcp-server

# All services logs (if using full stack)
docker-compose logs -f
```

### Restart Services
```bash
# Restart MCP server
docker-compose -f docker-compose.mcp-only.yml restart mcp-server

# Restart all services
docker-compose restart
```

### Stop Services
```bash
# Stop MCP server
docker-compose -f docker-compose.mcp-only.yml down

# Stop all services
docker-compose down
```

### Update Deployment
```bash
# Pull latest changes
git pull

# Rebuild and redeploy
docker-compose down
docker build -t supabase-mcp-server:latest .
docker-compose up -d
```

---

## 🔗 Access Your Deployed Server

### For AI Client Integration

Your MCP server is now accessible at:
- **Internal**: `http://localhost:8085`
- **External**: `http://YOUR_SERVER_IP:8085`

### Claude Desktop Configuration Example

```json
{
  "servers": {
    "supabase-cloud": {
      "command": "python",
      "args": ["-c", "import subprocess; subprocess.run(['ssh', 'username@YOUR_SERVER_IP', 'docker', 'exec', 'supabase-mcp-server', 'python', 'src/server.py'])"],
      "env": {}
    }
  }
}
```

### Direct Connection Testing

```bash
# Test with MCP Inspector (on your local machine)
npx @modelcontextprotocol/inspector ssh username@YOUR_SERVER_IP docker exec supabase-mcp-server python src/server.py
```

---

## 🛠️ Troubleshooting

### Common Issues

1. **Port 8085 not accessible**:
   ```bash
   sudo ufw status  # Check firewall
   docker ps        # Check if container is running
   ```

2. **Container won't start**:
   ```bash
   docker-compose logs mcp-server  # Check logs
   cat .env                        # Verify environment variables
   ```

3. **Permission denied errors**:
   ```bash
   sudo chown -R $USER:$USER .
   chmod +x scripts/deploy.sh
   ```

4. **Out of disk space**:
   ```bash
   docker system prune -a  # Clean up unused images
   df -h                   # Check disk usage
   ```

### Getting Help

- Check the [DEPLOYMENT.md](DEPLOYMENT.md) for comprehensive troubleshooting
- View container logs: `docker-compose logs mcp-server`
- Check Docker status: `docker ps -a`

---

## ✅ Success Checklist

After deployment, you should have:

- ✅ GitHub repository with your code
- ✅ MCP server running on your cloud instance
- ✅ Port 8085 accessible (verify with `curl http://YOUR_SERVER_IP:8085`)
- ✅ Container healthy and logging properly
- ✅ Firewall configured correctly
- ✅ Environment variables set properly

**🎉 Your Supabase MCP Server is now live and ready for AI integration!**

---

## 📞 Quick Reference Commands

```bash
# Repository operations
git clone https://github.com/YOUR_USERNAME/supabase-mcp-server.git
git pull                    # Update from GitHub
git add . && git commit -m "Update" && git push  # Push changes

# Deployment operations
./scripts/deploy.sh         # Automated deployment
docker-compose -f docker-compose.mcp-only.yml up -d    # Manual deploy
docker-compose -f docker-compose.mcp-only.yml logs -f mcp-server  # View logs
docker-compose -f docker-compose.mcp-only.yml down     # Stop services

# Server management
docker ps                   # List running containers
docker stats               # Resource usage
sudo ufw status            # Firewall status
```

This guide provides everything you need to get your Supabase MCP Server running in the cloud! 🚀
