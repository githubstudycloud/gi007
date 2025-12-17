# 🌐 远程 Ubuntu 服务器配置完全指南

> 从零开始搭建安全、高效的远程开发与部署环境

## 📋 目录

- [初始连接与安全加固](#初始连接与安全加固)
- [用户与权限管理](#用户与权限管理)
- [SSH 高级配置](#ssh-高级配置)
- [防火墙配置](#防火墙配置)
- [开发环境部署](#开发环境部署)
- [Docker 环境](#docker-环境)
- [Web 服务器](#web-服务器)
- [数据库服务](#数据库服务)
- [监控与日志](#监控与日志)
- [自动化部署](#自动化部署)
- [备份策略](#备份策略)
- [性能调优](#性能调优)
- [远程开发](#远程开发)
- [常用运维脚本](#常用运维脚本)

---

## 初始连接与安全加固

### 1. 首次连接

```bash
# 从本地机器连接
ssh root@your_server_ip

# 或使用密钥
ssh -i ~/.ssh/your_key root@your_server_ip
```

### 2. 系统更新

```bash
# 更新包列表
apt update

# 升级所有包
apt upgrade -y

# 升级发行版
apt dist-upgrade -y

# 清理
apt autoremove -y
apt autoclean
```

### 3. 设置时区

```bash
# 查看当前时区
timedatectl

# 设置时区
timedatectl set-timezone Asia/Shanghai

# 同步时间
apt install -y chrony
systemctl enable chrony
systemctl start chrony
```

### 4. 设置主机名

```bash
# 设置主机名
hostnamectl set-hostname myserver

# 编辑 hosts
echo "127.0.0.1 myserver" >> /etc/hosts
```

### 5. 安装基础工具

```bash
apt install -y \
    vim curl wget \
    htop btop \
    git \
    tree jq \
    unzip zip \
    net-tools \
    fail2ban \
    ufw
```

---

## 用户与权限管理

### 创建管理用户

```bash
# 创建新用户
adduser deploy

# 添加到 sudo 组
usermod -aG sudo deploy

# 验证
groups deploy
```

### 配置 sudo 免密 (可选)

```bash
# 编辑 sudoers
visudo

# 添加行
deploy ALL=(ALL) NOPASSWD:ALL
```

### 禁用 root 登录

```bash
# 先确保新用户可以 SSH 登录后再执行!

# 编辑 SSH 配置
vim /etc/ssh/sshd_config

# 修改
PermitRootLogin no

# 重启 SSH
systemctl restart sshd
```

### 切换到新用户

```bash
# 从本地重新连接
ssh deploy@your_server_ip
```

---

## SSH 高级配置

### 服务器端 SSH 配置

编辑 `/etc/ssh/sshd_config`:

```bash
# 端口 (建议修改默认端口)
Port 22222

# 禁用 root 登录
PermitRootLogin no

# 禁用密码登录 (配置密钥后)
PasswordAuthentication no

# 只允许特定用户
AllowUsers deploy

# 连接保活
ClientAliveInterval 60
ClientAliveCountMax 3

# 限制登录尝试
MaxAuthTries 3

# 禁用空密码
PermitEmptyPasswords no

# 禁用 X11 转发 (如不需要)
X11Forwarding no
```

应用更改:

```bash
# 检查配置
sshd -t

# 重启 SSH
systemctl restart sshd
```

### 配置 SSH 密钥登录

**在本地机器上:**

```bash
# 生成密钥 (如果没有)
ssh-keygen -t ed25519 -C "your@email.com"

# 复制公钥到服务器
ssh-copy-id -i ~/.ssh/id_ed25519.pub deploy@your_server_ip

# 或手动复制
cat ~/.ssh/id_ed25519.pub
# 复制输出内容
```

**在服务器上:**

```bash
# 创建 .ssh 目录
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# 添加公钥
echo "your_public_key_content" >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

### 本地 SSH 配置

编辑本地 `~/.ssh/config`:

```bash
# 远程服务器
Host myserver
    HostName your_server_ip
    User deploy
    Port 22222
    IdentityFile ~/.ssh/id_ed25519
    ServerAliveInterval 60
    ServerAliveCountMax 3

# 开发服务器
Host dev
    HostName dev.example.com
    User deploy
    Port 22222
    IdentityFile ~/.ssh/id_ed25519

# 跳板机配置 (如有)
Host internal
    HostName 10.0.0.10
    User deploy
    ProxyJump myserver
```

使用:

```bash
# 直接用别名连接
ssh myserver

# 通过跳板机连接
ssh internal
```

---

## 防火墙配置

### UFW (Uncomplicated Firewall)

```bash
# 安装
apt install -y ufw

# 默认策略
ufw default deny incoming
ufw default allow outgoing

# 允许 SSH (使用你的端口)
ufw allow 22222/tcp

# 允许 HTTP/HTTPS
ufw allow 80/tcp
ufw allow 443/tcp

# 允许其他服务 (按需)
ufw allow 3306/tcp    # MySQL
ufw allow 5432/tcp    # PostgreSQL
ufw allow 6379/tcp    # Redis

# 启用防火墙
ufw enable

# 查看状态
ufw status verbose

# 查看规则编号
ufw status numbered

# 删除规则
ufw delete 3
```

### Fail2ban (防暴力破解)

```bash
# 安装
apt install -y fail2ban

# 创建配置
cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local

# 编辑配置
vim /etc/fail2ban/jail.local
```

配置内容:

```ini
[DEFAULT]
bantime = 1h
findtime = 10m
maxretry = 5
ignoreip = 127.0.0.1/8

[sshd]
enabled = true
port = 22222
filter = sshd
logpath = /var/log/auth.log
maxretry = 3
bantime = 24h
```

启动:

```bash
systemctl enable fail2ban
systemctl start fail2ban

# 查看状态
fail2ban-client status sshd

# 查看被禁 IP
fail2ban-client status sshd | grep "Banned IP"

# 解禁 IP
fail2ban-client set sshd unbanip 1.2.3.4
```

---

## 开发环境部署

### Java 环境

```bash
# 安装 OpenJDK 21
apt install -y openjdk-21-jdk

# 或 SDKMAN (推荐多版本管理)
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"
sdk install java 21.0.2-tem

# 验证
java -version
```

### Python 环境

```bash
# 系统 Python
apt install -y python3 python3-pip python3-venv

# pyenv (多版本)
curl https://pyenv.run | bash

# 添加到 ~/.bashrc
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"

# 安装依赖
apt install -y make build-essential libssl-dev zlib1g-dev \
    libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm \
    libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev \
    libffi-dev liblzma-dev

# 安装 Python
pyenv install 3.12.1
pyenv global 3.12.1
```

### Go 环境

```bash
# 下载安装
GO_VERSION="1.22.0"
wget https://go.dev/dl/go${GO_VERSION}.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go${GO_VERSION}.linux-amd64.tar.gz
rm go${GO_VERSION}.linux-amd64.tar.gz

# 添加到 ~/.bashrc
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export PATH=$GOROOT/bin:$GOPATH/bin:$PATH
export GOPROXY=https://goproxy.cn,direct

# 验证
go version
```

### Node.js 环境

```bash
# nvm 安装
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash
source ~/.bashrc

# 安装 Node.js
nvm install --lts
nvm alias default lts/*

# 验证
node -v
npm -v
```

---

## Docker 环境

### 安装 Docker

```bash
# 卸载旧版本
apt remove docker docker-engine docker.io containerd runc

# 安装依赖
apt install -y ca-certificates curl gnupg lsb-release

# 添加 Docker GPG 密钥
mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 添加仓库
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | tee /etc/apt/sources.list.d/docker.list > /dev/null

# 安装
apt update
apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 添加用户到 docker 组
usermod -aG docker deploy

# 启动并设置开机启动
systemctl enable docker
systemctl start docker
```

### 配置 Docker

```bash
mkdir -p /etc/docker
cat > /etc/docker/daemon.json << 'EOF'
{
    "registry-mirrors": [
        "https://docker.mirrors.ustc.edu.cn"
    ],
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "100m",
        "max-file": "3"
    },
    "storage-driver": "overlay2",
    "live-restore": true
}
EOF

systemctl daemon-reload
systemctl restart docker
```

### Docker Compose 服务模板

`/opt/docker/docker-compose.yml`:

```yaml
version: '3.8'

services:
  # 应用服务
  app:
    image: your-app:latest
    container_name: app
    restart: unless-stopped
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=prod
      - DB_HOST=mysql
      - REDIS_HOST=redis
    depends_on:
      - mysql
      - redis
    networks:
      - app-network
    logging:
      driver: "json-file"
      options:
        max-size: "100m"
        max-file: "3"

  # MySQL
  mysql:
    image: mysql:8
    container_name: mysql
    restart: unless-stopped
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: ${MYSQL_ROOT_PASSWORD}
      MYSQL_DATABASE: ${MYSQL_DATABASE}
    volumes:
      - mysql_data:/var/lib/mysql
      - ./mysql/conf.d:/etc/mysql/conf.d
    networks:
      - app-network

  # Redis
  redis:
    image: redis:7-alpine
    container_name: redis
    restart: unless-stopped
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes --requirepass ${REDIS_PASSWORD}
    networks:
      - app-network

  # Nginx 反向代理
  nginx:
    image: nginx:alpine
    container_name: nginx
    restart: unless-stopped
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./nginx/conf.d:/etc/nginx/conf.d
      - ./certbot/www:/var/www/certbot
      - ./certbot/conf:/etc/letsencrypt
    depends_on:
      - app
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  mysql_data:
  redis_data:
```

环境变量 `.env`:

```bash
MYSQL_ROOT_PASSWORD=your_secure_password
MYSQL_DATABASE=appdb
REDIS_PASSWORD=your_redis_password
```

---

## Web 服务器

### Nginx 配置

```bash
# 安装
apt install -y nginx

# 启动
systemctl enable nginx
systemctl start nginx
```

站点配置 `/etc/nginx/sites-available/myapp`:

```nginx
upstream app_backend {
    server 127.0.0.1:8080;
    keepalive 32;
}

server {
    listen 80;
    server_name example.com www.example.com;

    # 重定向到 HTTPS
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name example.com www.example.com;

    # SSL 证书
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    # SSL 配置
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 1d;

    # 安全头
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;

    # 日志
    access_log /var/log/nginx/myapp_access.log;
    error_log /var/log/nginx/myapp_error.log;

    # 静态文件
    location /static/ {
        alias /var/www/myapp/static/;
        expires 30d;
        add_header Cache-Control "public, immutable";
    }

    # API 代理
    location /api/ {
        proxy_pass http://app_backend;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Connection "";
        proxy_connect_timeout 60s;
        proxy_read_timeout 60s;
    }

    # 前端
    location / {
        root /var/www/myapp/dist;
        try_files $uri $uri/ /index.html;
        expires 7d;
    }
}
```

启用站点:

```bash
ln -s /etc/nginx/sites-available/myapp /etc/nginx/sites-enabled/
nginx -t
systemctl reload nginx
```

### SSL 证书 (Let's Encrypt)

```bash
# 安装 Certbot
apt install -y certbot python3-certbot-nginx

# 获取证书
certbot --nginx -d example.com -d www.example.com

# 自动续期测试
certbot renew --dry-run

# 设置定时任务
crontab -e
# 添加:
0 0 1 * * /usr/bin/certbot renew --quiet
```

---

## 数据库服务

### MySQL 8

```bash
# 安装
apt install -y mysql-server

# 安全配置
mysql_secure_installation

# 创建用户
mysql -u root -p
```

```sql
-- 创建数据库
CREATE DATABASE myapp CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

-- 创建用户
CREATE USER 'appuser'@'localhost' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON myapp.* TO 'appuser'@'localhost';

-- 远程访问 (如需要)
CREATE USER 'appuser'@'%' IDENTIFIED BY 'secure_password';
GRANT ALL PRIVILEGES ON myapp.* TO 'appuser'@'%';

FLUSH PRIVILEGES;
```

配置 `/etc/mysql/mysql.conf.d/mysqld.cnf`:

```ini
[mysqld]
# 性能
innodb_buffer_pool_size = 1G
innodb_log_file_size = 256M
innodb_flush_log_at_trx_commit = 2

# 连接
max_connections = 200
wait_timeout = 600

# 字符集
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

# 慢查询日志
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 2
```

### PostgreSQL

```bash
# 安装
apt install -y postgresql postgresql-contrib

# 切换用户
sudo -u postgres psql
```

```sql
-- 创建用户
CREATE USER appuser WITH PASSWORD 'secure_password';

-- 创建数据库
CREATE DATABASE myapp OWNER appuser;

-- 授权
GRANT ALL PRIVILEGES ON DATABASE myapp TO appuser;
```

### Redis

```bash
# 安装
apt install -y redis-server

# 配置
vim /etc/redis/redis.conf
```

关键配置:

```ini
# 绑定地址
bind 127.0.0.1

# 密码
requirepass your_secure_password

# 持久化
appendonly yes
appendfsync everysec

# 内存限制
maxmemory 256mb
maxmemory-policy allkeys-lru
```

启动:

```bash
systemctl enable redis-server
systemctl restart redis-server
```

---

## 监控与日志

### 系统监控脚本

`/usr/local/bin/server-status.sh`:

```bash
#!/bin/bash

echo "========== 服务器状态 =========="
echo "时间: $(date '+%Y-%m-%d %H:%M:%S')"
echo ""

echo "=== 系统信息 ==="
echo "主机名: $(hostname)"
echo "运行时间: $(uptime -p)"
echo "负载: $(cat /proc/loadavg | awk '{print $1, $2, $3}')"
echo ""

echo "=== CPU 使用率 ==="
top -bn1 | grep "Cpu(s)" | awk '{print "使用: " 100-$8 "%"}'
echo ""

echo "=== 内存使用 ==="
free -h | awk 'NR==2{printf "已用: %s / %s (%.1f%%)\n", $3, $2, $3/$2*100}'
echo ""

echo "=== 磁盘使用 ==="
df -h / | awk 'NR==2{printf "已用: %s / %s (%s)\n", $3, $2, $5}'
echo ""

echo "=== Docker 容器 ==="
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}" 2>/dev/null || echo "Docker 未运行"
echo ""

echo "=== 最近登录 ==="
last -n 5
```

### 日志轮转

`/etc/logrotate.d/myapp`:

```
/var/log/myapp/*.log {
    daily
    missingok
    rotate 14
    compress
    delaycompress
    notifempty
    create 0640 www-data www-data
    sharedscripts
    postrotate
        systemctl reload nginx > /dev/null 2>&1 || true
    endscript
}
```

### 监控工具

#### htop / btop

```bash
apt install -y htop btop
```

#### Netdata (实时监控面板)

```bash
# 安装
bash <(curl -Ss https://my-netdata.io/kickstart.sh)

# 访问: http://your_server_ip:19999
```

#### Prometheus + Grafana (生产环境)

```yaml
# docker-compose.monitoring.yml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus
    container_name: prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus:/etc/prometheus
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'

  grafana:
    image: grafana/grafana
    container_name: grafana
    ports:
      - "3000:3000"
    volumes:
      - grafana_data:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin

  node-exporter:
    image: prom/node-exporter
    container_name: node-exporter
    ports:
      - "9100:9100"

volumes:
  prometheus_data:
  grafana_data:
```

---

## 自动化部署

### 部署脚本

`/opt/scripts/deploy.sh`:

```bash
#!/bin/bash

set -euo pipefail

# 配置
APP_NAME="myapp"
DEPLOY_DIR="/opt/${APP_NAME}"
BACKUP_DIR="/opt/backups/${APP_NAME}"
DOCKER_IMAGE="your-registry/${APP_NAME}"
COMPOSE_FILE="${DEPLOY_DIR}/docker-compose.yml"

# 颜色
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[0;33m'
NC='\033[0m'

log_info() { echo -e "${GREEN}[INFO]${NC} $1"; }
log_warn() { echo -e "${YELLOW}[WARN]${NC} $1"; }
log_error() { echo -e "${RED}[ERROR]${NC} $1"; exit 1; }

# 备份当前版本
backup() {
    log_info "备份当前版本..."
    BACKUP_FILE="${BACKUP_DIR}/backup_$(date +%Y%m%d_%H%M%S).tar.gz"
    mkdir -p "${BACKUP_DIR}"

    if docker ps -q -f name="${APP_NAME}" | grep -q .; then
        docker commit "${APP_NAME}" "${APP_NAME}_backup:$(date +%Y%m%d_%H%M%S)"
    fi

    log_info "备份完成: ${BACKUP_FILE}"
}

# 拉取最新镜像
pull_image() {
    log_info "拉取最新镜像..."
    docker pull "${DOCKER_IMAGE}:latest" || log_error "镜像拉取失败"
}

# 部署
deploy() {
    log_info "开始部署..."

    cd "${DEPLOY_DIR}"

    # 停止旧容器
    docker compose down || true

    # 启动新容器
    docker compose up -d

    # 等待健康检查
    log_info "等待服务启动..."
    sleep 10

    # 检查服务状态
    if docker ps -q -f name="${APP_NAME}" | grep -q .; then
        log_info "部署成功!"
    else
        log_error "部署失败，请检查日志"
    fi
}

# 回滚
rollback() {
    log_warn "执行回滚..."
    # 实现回滚逻辑
}

# 健康检查
health_check() {
    log_info "执行健康检查..."

    for i in {1..30}; do
        if curl -sf http://localhost:8080/health > /dev/null 2>&1; then
            log_info "健康检查通过"
            return 0
        fi
        sleep 2
    done

    log_error "健康检查失败"
}

# 清理旧镜像
cleanup() {
    log_info "清理旧镜像..."
    docker image prune -af --filter "until=168h"
}

# 主函数
main() {
    case "${1:-deploy}" in
        deploy)
            backup
            pull_image
            deploy
            health_check
            cleanup
            ;;
        rollback)
            rollback
            ;;
        status)
            docker ps -a
            ;;
        logs)
            docker compose -f "${COMPOSE_FILE}" logs -f
            ;;
        *)
            echo "Usage: $0 {deploy|rollback|status|logs}"
            exit 1
            ;;
    esac
}

main "$@"
```

### GitHub Actions 自动部署

`.github/workflows/deploy.yml`:

```yaml
name: Deploy to Server

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Build Docker image
        run: |
          docker build -t ${{ secrets.DOCKER_REGISTRY }}/myapp:${{ github.sha }} .
          docker tag ${{ secrets.DOCKER_REGISTRY }}/myapp:${{ github.sha }} ${{ secrets.DOCKER_REGISTRY }}/myapp:latest

      - name: Push to registry
        run: |
          echo ${{ secrets.DOCKER_PASSWORD }} | docker login -u ${{ secrets.DOCKER_USERNAME }} --password-stdin
          docker push ${{ secrets.DOCKER_REGISTRY }}/myapp:${{ github.sha }}
          docker push ${{ secrets.DOCKER_REGISTRY }}/myapp:latest

  deploy:
    needs: build
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to server
        uses: appleboy/ssh-action@master
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SERVER_SSH_KEY }}
          port: ${{ secrets.SERVER_PORT }}
          script: |
            cd /opt/myapp
            ./deploy.sh deploy
```

---

## 备份策略

### 数据库备份脚本

`/opt/scripts/backup-db.sh`:

```bash
#!/bin/bash

set -euo pipefail

# 配置
BACKUP_DIR="/opt/backups/mysql"
MYSQL_USER="backup"
MYSQL_PASS="backup_password"
RETENTION_DAYS=7

# 创建备份目录
mkdir -p "${BACKUP_DIR}"

# 备份日期
DATE=$(date +%Y%m%d_%H%M%S)

# 执行备份
mysqldump -u "${MYSQL_USER}" -p"${MYSQL_PASS}" \
    --all-databases \
    --single-transaction \
    --routines \
    --triggers \
    --events \
    | gzip > "${BACKUP_DIR}/all_databases_${DATE}.sql.gz"

# 删除旧备份
find "${BACKUP_DIR}" -name "*.sql.gz" -mtime +${RETENTION_DAYS} -delete

echo "备份完成: ${BACKUP_DIR}/all_databases_${DATE}.sql.gz"
```

### 定时任务

```bash
crontab -e

# 每天凌晨 2 点备份数据库
0 2 * * * /opt/scripts/backup-db.sh >> /var/log/backup.log 2>&1

# 每周日凌晨 3 点完整备份
0 3 * * 0 /opt/scripts/full-backup.sh >> /var/log/backup.log 2>&1

# 每天清理 Docker
0 4 * * * docker system prune -af >> /var/log/docker-cleanup.log 2>&1
```

### 远程备份同步

```bash
#!/bin/bash

# 同步到远程存储
rsync -avz --delete \
    /opt/backups/ \
    backup@backup-server:/backups/myserver/ \
    -e "ssh -p 22222"

# 或上传到 S3
aws s3 sync /opt/backups/ s3://my-backup-bucket/myserver/ --delete
```

---

## 性能调优

### 系统参数优化

`/etc/sysctl.conf`:

```ini
# 网络优化
net.core.somaxconn = 65535
net.core.netdev_max_backlog = 65535
net.ipv4.tcp_max_syn_backlog = 65535
net.ipv4.tcp_fin_timeout = 15
net.ipv4.tcp_keepalive_time = 300
net.ipv4.tcp_keepalive_probes = 5
net.ipv4.tcp_keepalive_intvl = 15

# 内存优化
vm.swappiness = 10
vm.dirty_ratio = 60
vm.dirty_background_ratio = 5

# 文件描述符
fs.file-max = 2097152
fs.inotify.max_user_watches = 524288
```

应用:

```bash
sysctl -p
```

### 文件描述符限制

`/etc/security/limits.conf`:

```
* soft nofile 65535
* hard nofile 65535
* soft nproc 65535
* hard nproc 65535
```

### JVM 调优 (Java 应用)

```bash
# docker-compose.yml 中设置
environment:
  - JAVA_OPTS=-Xms512m -Xmx2g -XX:+UseG1GC -XX:MaxGCPauseMillis=200
```

---

## 远程开发

### VS Code Remote SSH

1. 安装 Remote - SSH 扩展
2. 配置 `~/.ssh/config`
3. `Ctrl+Shift+P` → "Remote-SSH: Connect to Host"

### VS Code 服务器扩展

在远程服务器上预装常用扩展:

```bash
# 安装 code-server (可选)
curl -fsSL https://code-server.dev/install.sh | sh

# 配置
mkdir -p ~/.config/code-server
cat > ~/.config/code-server/config.yaml << 'EOF'
bind-addr: 0.0.0.0:8443
auth: password
password: your_secure_password
cert: false
EOF

# 启动
systemctl enable --user code-server
systemctl start --user code-server
```

### SSH 端口转发

```bash
# 本地转发 (访问远程服务)
ssh -L 8080:localhost:8080 myserver

# 动态转发 (SOCKS 代理)
ssh -D 1080 myserver
```

---

## 常用运维脚本

### 服务管理

`/opt/scripts/service-manager.sh`:

```bash
#!/bin/bash

ACTION="${1:-status}"

case "$ACTION" in
    start)
        systemctl start nginx mysql redis docker
        docker compose -f /opt/myapp/docker-compose.yml up -d
        ;;
    stop)
        docker compose -f /opt/myapp/docker-compose.yml down
        systemctl stop nginx mysql redis
        ;;
    restart)
        $0 stop
        sleep 3
        $0 start
        ;;
    status)
        echo "=== 系统服务 ==="
        for svc in nginx mysql redis docker; do
            status=$(systemctl is-active $svc)
            echo "$svc: $status"
        done
        echo ""
        echo "=== Docker 容器 ==="
        docker ps --format "table {{.Names}}\t{{.Status}}"
        ;;
    *)
        echo "Usage: $0 {start|stop|restart|status}"
        ;;
esac
```

### 日志查看

`/opt/scripts/logs.sh`:

```bash
#!/bin/bash

SERVICE="${1:-app}"
LINES="${2:-100}"

case "$SERVICE" in
    app)
        docker logs -f --tail "$LINES" myapp
        ;;
    nginx)
        tail -f /var/log/nginx/access.log /var/log/nginx/error.log
        ;;
    mysql)
        tail -f /var/log/mysql/error.log
        ;;
    system)
        journalctl -f
        ;;
    *)
        echo "Usage: $0 {app|nginx|mysql|system} [lines]"
        ;;
esac
```

### 快速诊断

`/opt/scripts/diagnose.sh`:

```bash
#!/bin/bash

echo "========== 系统诊断 =========="
date

echo ""
echo "=== 磁盘空间 ==="
df -h | head -10

echo ""
echo "=== 内存使用 ==="
free -h

echo ""
echo "=== CPU 负载 ==="
uptime

echo ""
echo "=== 网络连接 ==="
ss -tuln | head -20

echo ""
echo "=== Docker 状态 ==="
docker ps -a

echo ""
echo "=== 最近错误日志 ==="
journalctl -p err --since "1 hour ago" --no-pager | tail -20

echo ""
echo "=== 失败的服务 ==="
systemctl --failed

echo ""
echo "========== 诊断完成 =========="
```

---

## 安全检查清单

- [ ] 禁用 root SSH 登录
- [ ] 使用 SSH 密钥认证
- [ ] 修改 SSH 默认端口
- [ ] 配置防火墙 (UFW)
- [ ] 安装 Fail2ban
- [ ] 定期系统更新
- [ ] 配置自动安全更新
- [ ] 数据库使用强密码
- [ ] 敏感数据加密存储
- [ ] 定期备份
- [ ] 监控异常登录
- [ ] HTTPS 全站启用
- [ ] 日志集中管理

---

## 快速命令参考

```bash
# 系统
uptime                    # 运行时间
free -h                   # 内存使用
df -h                     # 磁盘使用
htop                      # 进程监控

# 网络
ss -tuln                  # 监听端口
netstat -anp | grep LISTEN
curl -I https://example.com

# Docker
docker ps -a              # 所有容器
docker logs -f <container>
docker stats              # 资源使用
docker compose up -d      # 启动服务
docker compose down       # 停止服务
docker system prune -af   # 清理

# 日志
journalctl -f             # 系统日志
tail -f /var/log/syslog   # 系统日志
tail -f /var/log/nginx/access.log

# 服务
systemctl status <service>
systemctl restart <service>
systemctl enable <service>
```

---

**上一步**: [Mac Mini 开发环境](../03-mac-mini/README.md)
**返回**: [主页](../README.md)
