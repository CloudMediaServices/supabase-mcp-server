# Cloud Docker Deployment Guide

This guide covers deploying the Supabase MCP Server to your cloud Docker instance.

## 🚀 Quick Deployment

### Option 1: Automated Deployment (Recommended)

1. **Upload files to your cloud instance**:
   ```bash
   # Copy all files to your cloud server
   scp -r supabase-mcp-server/ user@your-server.com:/home/user/
   ```

2. **SSH into your cloud instance**:
   ```bash
   ssh user@your-server.com
   cd /home/user/supabase-mcp-server
   ```

3. **Run deployment script**:
   ```bash
   chmod +x scripts/deploy.sh
   ./scripts/deploy.sh
   ```

### Option 2: Manual Deployment

1. **Prepare environment**:
   ```bash
   # Copy and configure environment
   cp .env.production .env
   nano .env  # Edit with your Supabase credentials
   ```

2. **Deploy MCP server only** (if you have existing Supabase):
   ```bash
   docker-compose -f docker-compose.mcp-only.yml up -d
   ```

3. **Deploy full stack** (includes self-hosted Supabase):
   ```bash
   docker-compose up -d
   ```

## 📋 Prerequisites

### System Requirements
- Ubuntu 20.04+ / CentOS 8+ / Debian 11+
- Docker 24.0+
- Docker Compose 2.0+
- 2GB+ RAM
- 10GB+ disk space
- Open ports: 8080 (MCP), 3000 (Studio), 8000 (API)

### Install Docker (if needed)
```bash
# Ubuntu/Debian
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER

# Install Docker Compose
sudo apt install docker-compose-plugin

# Verify installation
docker --version
docker compose version
```

## 🔧 Configuration

### Environment Variables

Edit `.env` file with your configuration:

```env
# Required: Your existing Supabase instance
SUPABASE_URL=https://your-project-id.supabase.co
SUPABASE_SERVICE_ROLE_KEY=your-service-role-key-here

# Optional: Server configuration
LOG_LEVEL=INFO
```

### Firewall Configuration

Open required ports:
```bash
# Ubuntu/Debian with UFW
sudo ufw allow 8080/tcp  # MCP Server
sudo ufw allow 3000/tcp  # Supabase Studio (if using full stack)
sudo ufw allow 8000/tcp  # Supabase API (if using full stack)

# CentOS/RHEL with firewalld
sudo firewall-cmd --permanent --add-port=8080/tcp
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --permanent --add-port=8000/tcp
sudo firewall-cmd --reload
```

## 🔄 Deployment Options

### Option A: MCP Server Only
Use this if you already have a Supabase instance (cloud or self-hosted):

```bash
# Build and deploy
docker build -t supabase-mcp-server:latest .
docker-compose -f docker-compose.mcp-only.yml up -d

# Check status
docker-compose -f docker-compose.mcp-only.yml ps
docker-compose -f docker-compose.mcp-only.yml logs mcp-server
```

**Access**: MCP server available on port 8080

### Option B: Full Stack Deployment
Includes self-hosted Supabase components:

```bash
# Configure additional environment variables in .env
POSTGRES_PASSWORD=your-secure-password
JWT_SECRET=your-jwt-secret-32-chars-minimum

# Deploy full stack
docker-compose up -d

# Check status
docker-compose ps
```

**Access**:
- MCP Server: Port 8080
- Supabase Studio: http://your-server.com:3000
- Supabase API: http://your-server.com:8000

## 🔍 Monitoring & Management

### Check Server Status
```bash
# View running containers
docker ps

# View logs
docker-compose -f docker-compose.mcp-only.yml logs -f mcp-server

# Check resource usage
docker stats
```

### Container Management
```bash
# Stop services
docker-compose -f docker-compose.mcp-only.yml down

# Restart services
docker-compose -f docker-compose.mcp-only.yml restart

# Update and redeploy
git pull
docker-compose -f docker-compose.mcp-only.yml down
docker build -t supabase-mcp-server:latest .
docker-compose -f docker-compose.mcp-only.yml up -d
```

### Health Checks
```bash
# Check container health
docker inspect supabase-mcp-server | grep -A 10 Health

# Test MCP server responsiveness
docker exec supabase-mcp-server python -c "import sys; sys.exit(0)"
```

## 🔐 Security Best Practices

### Container Security
- ✅ Non-root user in container
- ✅ Read-only filesystem where possible
- ✅ Limited resource usage
- ✅ Health checks enabled

### Network Security
```bash
# Create dedicated network
docker network create mcp-network --subnet=172.20.0.0/16

# Use specific subnet in compose file
# (already configured in docker-compose.yml)
```

### Secrets Management
```bash
# Use Docker secrets for sensitive data
echo "your-service-role-key" | docker secret create supabase_key -

# Reference in compose file:
# secrets:
#   - supabase_key
```

## 📊 Performance Tuning

### Resource Limits
Add to docker-compose.yml:
```yaml
services:
  mcp-server:
    # ... existing config
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 1G
        reservations:
          cpus: '0.5'
          memory: 512M
```

### Logging Configuration
```yaml
logging:
  driver: "json-file"
  options:
    max-size: "10m"
    max-file: "3"
```

## 🔄 Backup & Recovery

### Database Backup (if using full stack)
```bash
# Create backup
docker exec supabase-db pg_dump -U supabase supabase > backup_$(date +%Y%m%d).sql

# Restore backup
docker exec -i supabase-db psql -U supabase supabase < backup_20240809.sql
```

### Configuration Backup
```bash
# Backup configuration
tar -czf mcp-server-config-$(date +%Y%m%d).tar.gz .env docker-compose*.yml

# Automated backup script
cat > backup.sh << 'EOF'
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
tar -czf "/backups/mcp-config-$DATE.tar.gz" .env docker-compose*.yml logs/
find /backups -name "mcp-config-*.tar.gz" -mtime +7 -delete
EOF

chmod +x backup.sh
```

## 🐛 Troubleshooting

### Common Issues

1. **Container won't start**:
   ```bash
   # Check logs
   docker-compose logs mcp-server
   
   # Check environment variables
   docker exec supabase-mcp-server printenv | grep SUPABASE
   ```

2. **Port conflicts**:
   ```bash
   # Check what's using the port
   sudo netstat -tulpn | grep :8080
   
   # Change port in docker-compose.yml
   ports:
     - "8081:8000"  # Use 8081 instead of 8080
   ```

3. **Permission issues**:
   ```bash
   # Fix logs directory permissions
   sudo chown -R 1001:1001 logs/
   chmod 755 logs/
   ```

4. **Out of disk space**:
   ```bash
   # Clean up Docker
   docker system prune -a
   docker volume prune
   ```

### Log Analysis
```bash
# View recent errors
docker-compose logs mcp-server | grep ERROR

# Monitor live logs
docker-compose logs -f mcp-server

# Export logs for analysis
docker-compose logs mcp-server > mcp-server-logs.txt
```

## 📈 Scaling

### Horizontal Scaling
```yaml
# docker-compose.override.yml
services:
  mcp-server:
    deploy:
      replicas: 3
```

### Load Balancing
```bash
# Install and configure nginx
sudo apt install nginx

# Configure upstream in /etc/nginx/sites-available/mcp-server
upstream mcp_servers {
    server localhost:8080;
    server localhost:8081;
    server localhost:8082;
}

server {
    listen 80;
    location / {
        proxy_pass http://mcp_servers;
    }
}
```

## 🔗 Integration

### Connect AI Clients

**Claude Desktop Configuration**:
```json
{
  "servers": {
    "supabase": {
      "command": "docker",
      "args": ["exec", "supabase-mcp-server", "python", "src/server.py"],
      "env": {}
    }
  }
}
```

**Direct Connection**:
```bash
# Test with MCP Inspector
npx @modelcontextprotocol/inspector \
  docker exec supabase-mcp-server python src/server.py
```

## 📞 Support

### Health Monitoring
```bash
# Setup monitoring script
cat > monitor.sh << 'EOF'
#!/bin/bash
if ! docker inspect supabase-mcp-server | grep -q '"Running": true'; then
    echo "MCP Server is down! Restarting..."
    docker-compose -f docker-compose.mcp-only.yml restart mcp-server
fi
EOF

# Add to crontab for automatic monitoring
echo "*/5 * * * * /path/to/monitor.sh" | crontab -
```

### Performance Monitoring
```bash
# Install monitoring tools
docker run -d \
  --name=grafana \
  -p 3001:3000 \
  grafana/grafana

# Monitor resource usage
docker stats --format "table {{.Container}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```

---

🎉 **Your Supabase MCP Server is now deployed!**

For additional support, check the main README.md or create an issue in the project repository.
