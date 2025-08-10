# 🚀 SSH Deployment Quick Reference

Deploy your Supabase MCP Server to any cloud instance in 3 commands!

## Prerequisites
- Cloud server with SSH access
- Docker installed on server
- Your Supabase URL and service role key

## Deployment Commands

```bash
# 1. SSH into your server
ssh username@your-server-ip

# 2. Clone and deploy (one command!)
git clone https://github.com/YOUR_USERNAME/supabase-mcp-server.git && cd supabase-mcp-server && chmod +x scripts/deploy.sh && ./scripts/deploy.sh

# 3. Configure firewall
sudo ufw allow 8085/tcp && sudo ufw --force enable
```

## That's it! 🎉

Your server is now running on port 8085.

## Quick Management

```bash
# View logs
docker-compose -f docker-compose.mcp-only.yml logs -f mcp-server

# Restart server
docker-compose -f docker-compose.mcp-only.yml restart

# Stop server
docker-compose -f docker-compose.mcp-only.yml down

# Update and redeploy
git pull && docker-compose down && docker build -t supabase-mcp-server:latest . && docker-compose up -d
```

## Access URLs
- **MCP Server**: `http://YOUR_SERVER_IP:8085`
- **Health Check**: SSH in and run `docker ps`

## Need Help?
- 📖 [Complete Setup Guide](GITHUB_SETUP.md)
- 🔧 [Deployment Documentation](DEPLOYMENT.md)
- 📚 [Main README](README.md)

---
*Replace `YOUR_USERNAME` and `your-server-ip` with your actual values*
