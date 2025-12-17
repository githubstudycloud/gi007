# 绿联 DXP4800 Plus 部署方案

## 硬件概览

| 组件 | 规格 | 特性说明 |
|------|------|----------|
| 固态1 | 2TB NVMe | 高性能TLC/MLC |
| 固态2 | 2TB NVMe (QLC) | 写入寿命较低,适合读多写少场景 |
| 机械1 | 8TB 西数红盘Plus | NAS专用盘,高可靠性 |
| 机械2 | 8TB 西数红盘Plus | NAS专用盘,高可靠性 |
| 机械3 | 14TB 红盘(二手) | 需要监控SMART状态 |
| 机械4 | 18TB 西数企业盘 | 高负载场景 |

> **重要提示**: QLC固态不适合高频写入场景,二手硬盘需要密切监控健康状态

---

## 一、存储架构设计

### 1.1 设计原则

```
┌─────────────────────────────────────────────────────────────────┐
│                    绿联 DXP4800 Plus 定位                        │
│                                                                 │
│   🎯 主要角色: 媒体中心 + 冷备份 + 家庭服务                        │
│   🔄 与极空间关系: 互为备份 + 服务分流                             │
│   ⚠️ 注意事项: QLC盘避免写密集型任务                              │
└─────────────────────────────────────────────────────────────────┘
```

### 1.2 RAID 方案建议

```
存储池规划:
┌─────────────────────────────────────────────────────────────────┐
│ Pool 1: SSD存储池 (2TB TLC)                                     │
│   - 用途: 系统、热数据、Docker                                   │
│   - RAID: 无RAID (单盘)                                         │
│   - 原因: 保留QLC盘做专用读取池                                  │
├─────────────────────────────────────────────────────────────────┤
│ Pool 2: SSD读取池 (2TB QLC)                                     │
│   - 用途: 媒体缓存、静态资源、只读数据                            │
│   - RAID: 无RAID (单盘)                                         │
│   - 原因: QLC读取性能好但写入寿命短                              │
├─────────────────────────────────────────────────────────────────┤
│ Pool 3: 高可靠存储池 (8TB + 8TB 红盘Plus)                        │
│   - 用途: 重要数据、照片原片、家庭文档                            │
│   - RAID: RAID 1 (镜像)                                         │
│   - 有效容量: 8TB                                                │
│   - 原因: 双红盘Plus可靠性最高,做镜像保护关键数据                 │
├─────────────────────────────────────────────────────────────────┤
│ Pool 4: 大容量存储池 (14TB二手 + 18TB企业)                       │
│   - 用途: 媒体库、大文件、冷归档                                 │
│   - RAID: 基本卷 (各自独立) 或 JBOD                              │
│   - 有效容量: 32TB                                               │
│   - 原因: 不同盘龄,不建议RAID;媒体数据可重新获取                 │
└─────────────────────────────────────────────────────────────────┘
```

### 1.3 具体空间分配

#### 2TB TLC固态 - 系统盘
```
/volume1/ssd-system/
├── system/              # 100GB - UGOS系统数据
│   └── docker/          # Docker数据目录
├── app-data/            # 200GB - 应用程序数据
│   ├── home-assistant/  # 智能家居
│   ├── homebridge/      # iOS智能家居桥接
│   └── databases/       # 轻量数据库
├── download-temp/       # 500GB - 下载临时目录
├── transcode-cache/     # 500GB - 视频转码缓存
└── hot-data/            # 700GB - 热点数据
    ├── recent-photos/   # 近期照片(待整理)
    └── working-files/   # 当前工作文件
```

#### 2TB QLC固态 - 只读缓存盘
```
/volume2/ssd-cache/
├── media-cache/         # 1TB - 媒体元数据缓存
│   ├── jellyfin-meta/   # Jellyfin元数据
│   ├── plex-meta/       # Plex元数据(备用)
│   └── thumbnail/       # 缩略图缓存
├── static-resources/    # 500GB - 静态资源
│   ├── fonts/           # 字体库
│   ├── icons/           # 图标库
│   └── templates/       # 模板文件
└── read-cache/          # 500GB - SSD读取缓存
    └── frequently-read/ # 高频访问文件
```

#### 8TB RAID1镜像池 - 关键数据
```
/volume3/critical-data/
├── family-vault/        # 3TB - 家庭数字保险箱
│   ├── documents/       # 重要文档(证件、合同等)
│   ├── photos-master/   # 照片原片归档
│   └── videos-precious/ # 珍贵视频(家庭录像等)
├── backup-vault/        # 3TB - 备份仓库
│   ├── zspace-sync/     # 极空间关键数据同步
│   ├── device-backup/   # 设备备份
│   │   ├── mac-critical/
│   │   ├── phone-backup/
│   │   └── pc-critical/
│   └── config-backup/   # 配置文件备份
└── encrypted-data/      # 2TB - 加密存储
    └── sensitive/       # 敏感数据(加密卷)
```

#### 14TB 二手红盘 - 媒体库A
```
/volume4/media-a/
├── movies/              # 6TB - 电影库
│   ├── 4k-hdr/          # 4K HDR电影
│   ├── 1080p/           # 1080P电影
│   └── classic/         # 经典老片
├── tv-shows/            # 5TB - 剧集库
│   ├── ongoing/         # 追更剧集
│   └── completed/       # 完结剧集
└── anime/               # 3TB - 动漫库
```

#### 18TB 企业盘 - 媒体库B + 归档
```
/volume5/media-b/
├── media-overflow/      # 8TB - 媒体溢出
│   ├── movies-extra/    # 更多电影
│   ├── documentary/     # 纪录片
│   └── concerts/        # 演唱会/音乐视频
├── raw-archive/         # 5TB - 原始素材归档
│   ├── video-raw/       # 视频原始素材
│   └── photo-raw/       # RAW照片归档
├── cold-storage/        # 3TB - 冷数据
│   ├── old-projects/    # 历史项目
│   └── legacy-data/     # 遗留数据
└── disaster-recovery/   # 2TB - 灾备
    └── zspace-backup/   # 极空间灾备副本
```

---

## 二、服务部署方案

### 2.1 服务架构定位

```
┌─────────────────────────────────────────────────────────────────┐
│                  绿联 DXP4800 Plus (家庭媒体中心)                  │
├─────────────────────────────────────────────────────────────────┤
│                        媒体服务层 (核心)                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │   Jellyfin  │  │    Plex     │  │   Emby      │            │
│  │  主媒体服务  │  │   备用/iOS  │  │   备用      │            │
│  └─────────────┘  └─────────────┘  └─────────────┘            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │   Navidrome │  │   Immich    │  │   Kavita    │            │
│  │   音乐流媒体 │  │   照片管理   │  │  电子书漫画 │            │
│  └─────────────┘  └─────────────┘  └─────────────┘            │
├─────────────────────────────────────────────────────────────────┤
│                        下载管理层                                │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │ qBittorrent │  │  Transmission│  │   Aria2    │            │
│  │   BT下载    │  │   轻量BT    │  │  多协议下载 │            │
│  └─────────────┘  └─────────────┘  └─────────────┘            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │   Sonarr    │  │   Radarr    │  │   Jackett   │            │
│  │   剧集追踪   │  │   电影追踪   │  │  索引聚合   │            │
│  └─────────────┘  └─────────────┘  └─────────────┘            │
├─────────────────────────────────────────────────────────────────┤
│                        智能家居层                                │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │Home Assistant│ │  Homebridge │  │  Node-RED   │            │
│  │  智能家居中心 │  │  HomeKit桥接│  │  自动化流程 │            │
│  └─────────────┘  └─────────────┘  └─────────────┘            │
├─────────────────────────────────────────────────────────────────┤
│                        家庭服务层                                │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │  Nextcloud  │  │   Vaultwarden│ │  Bitwarden │            │
│  │  家庭云盘    │  │   密码管理   │  │  (备用)    │            │
│  └─────────────┘  └─────────────┘  └─────────────┘            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │   Alist     │  │  FileBrowser│  │  Syncthing │            │
│  │  网盘聚合    │  │   文件管理   │  │  设备同步   │            │
│  └─────────────┘  └─────────────┘  └─────────────┘            │
├─────────────────────────────────────────────────────────────────┤
│                        网络服务层                                │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐            │
│  │ AdGuard Home│  │   Pi-hole   │  │  Unbound   │            │
│  │   广告过滤   │  │   (备用)    │  │  DNS递归   │            │
│  └─────────────┘  └─────────────┘  └─────────────┘            │
│  ┌─────────────┐  ┌─────────────┐                             │
│  │  Tailscale  │  │   Nginx PM  │                             │
│  │   组网VPN   │  │   反向代理   │                             │
│  └─────────────┘  └─────────────┘                             │
└─────────────────────────────────────────────────────────────────┘
```

### 2.2 资源分配

| 服务类别 | 内存分配 | 优先级 | 说明 |
|----------|----------|--------|------|
| 媒体服务 | 4GB | 高 | Jellyfin硬解转码 |
| 智能家居 | 2GB | 高 | 需要持续运行 |
| 下载服务 | 2GB | 中 | 按需运行 |
| 照片管理 | 2GB | 中 | Immich索引 |
| 家庭服务 | 1GB | 中 | Nextcloud等 |
| 网络服务 | 512MB | 高 | DNS必须稳定 |

---

## 三、Docker Compose 配置

### 3.1 基础设施

```yaml
# docker-compose.base.yml
version: '3.8'

services:
  nginx-proxy-manager:
    image: jc21/nginx-proxy-manager:latest
    container_name: nginx-proxy-manager
    restart: always
    ports:
      - "80:80"
      - "443:443"
      - "81:81"
    volumes:
      - ./npm/data:/data
      - ./npm/letsencrypt:/etc/letsencrypt
    networks:
      - proxy-net

  portainer:
    image: portainer/portainer-ce:latest
    container_name: portainer
    restart: always
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./portainer/data:/data
    ports:
      - "9000:9000"
    networks:
      - proxy-net

  watchtower:
    image: containrrr/watchtower
    container_name: watchtower
    restart: always
    environment:
      - WATCHTOWER_CLEANUP=true
      - WATCHTOWER_SCHEDULE=0 0 5 * * *  # 每天凌晨5点
      - WATCHTOWER_INCLUDE_STOPPED=false
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock

networks:
  proxy-net:
    driver: bridge
```

### 3.2 媒体服务

```yaml
# docker-compose.media.yml
version: '3.8'

services:
  jellyfin:
    image: jellyfin/jellyfin:latest
    container_name: jellyfin
    restart: always
    environment:
      - TZ=Asia/Shanghai
      - JELLYFIN_PublishedServerUrl=https://media.home.local
    volumes:
      - ./jellyfin/config:/config
      - /volume2/ssd-cache/jellyfin-meta:/config/metadata
      - /volume1/ssd-system/transcode-cache:/transcode
      - /volume4/media-a/movies:/media/movies:ro
      - /volume4/media-a/tv-shows:/media/tv:ro
      - /volume4/media-a/anime:/media/anime:ro
      - /volume5/media-b/movies-extra:/media/movies-extra:ro
      - /volume5/media-b/documentary:/media/documentary:ro
    ports:
      - "8096:8096"
    devices:
      - /dev/dri:/dev/dri  # Intel 硬件转码
    deploy:
      resources:
        limits:
          memory: 4G
    networks:
      - media-net

  plex:
    image: lscr.io/linuxserver/plex:latest
    container_name: plex
    restart: unless-stopped
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai
      - VERSION=docker
    volumes:
      - ./plex/config:/config
      - /volume2/ssd-cache/plex-meta:/config/Library
      - /volume4/media-a:/media-a:ro
      - /volume5/media-b:/media-b:ro
    ports:
      - "32400:32400"
    devices:
      - /dev/dri:/dev/dri
    deploy:
      resources:
        limits:
          memory: 2G
    networks:
      - media-net

  navidrome:
    image: deluan/navidrome:latest
    container_name: navidrome
    restart: always
    environment:
      ND_SCANSCHEDULE: 1h
      ND_LOGLEVEL: info
      ND_BASEURL: ""
      ND_ENABLETRANSCODINGCONFIG: "true"
      ND_TRANSCODINGCACHESIZE: "1GB"
    volumes:
      - ./navidrome/data:/data
      - /volume3/critical-data/family-vault/music:/music:ro
    ports:
      - "4533:4533"
    deploy:
      resources:
        limits:
          memory: 512M
    networks:
      - media-net

  immich:
    image: ghcr.io/immich-app/immich-server:release
    container_name: immich-server
    restart: always
    environment:
      - DB_HOSTNAME=immich-postgres
      - DB_USERNAME=immich
      - DB_PASSWORD=${IMMICH_DB_PASSWORD}
      - DB_DATABASE_NAME=immich
      - REDIS_HOSTNAME=immich-redis
      - UPLOAD_LOCATION=/photos
      - IMMICH_MACHINE_LEARNING_URL=http://immich-ml:3003
    volumes:
      - /volume3/critical-data/family-vault/photos-master:/photos
      - ./immich/upload:/upload
    ports:
      - "2283:3001"
    depends_on:
      - immich-postgres
      - immich-redis
    deploy:
      resources:
        limits:
          memory: 2G
    networks:
      - media-net

  immich-ml:
    image: ghcr.io/immich-app/immich-machine-learning:release
    container_name: immich-ml
    restart: always
    volumes:
      - ./immich/ml-cache:/cache
    deploy:
      resources:
        limits:
          memory: 2G
    networks:
      - media-net

  immich-postgres:
    image: tensorchord/pgvecto-rs:pg14-v0.2.0
    container_name: immich-postgres
    restart: always
    environment:
      POSTGRES_PASSWORD: ${IMMICH_DB_PASSWORD}
      POSTGRES_USER: immich
      POSTGRES_DB: immich
    volumes:
      - ./immich/postgres:/var/lib/postgresql/data
    networks:
      - media-net

  immich-redis:
    image: redis:7-alpine
    container_name: immich-redis
    restart: always
    networks:
      - media-net

  kavita:
    image: lscr.io/linuxserver/kavita:latest
    container_name: kavita
    restart: always
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai
    volumes:
      - ./kavita/config:/config
      - /volume3/critical-data/family-vault/books:/books
      - /volume4/media-a/comics:/comics
    ports:
      - "5000:5000"
    deploy:
      resources:
        limits:
          memory: 512M
    networks:
      - media-net

networks:
  media-net:
    driver: bridge
```

### 3.3 下载管理

```yaml
# docker-compose.download.yml
version: '3.8'

services:
  qbittorrent:
    image: lscr.io/linuxserver/qbittorrent:latest
    container_name: qbittorrent
    restart: always
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai
      - WEBUI_PORT=8080
    volumes:
      - ./qbittorrent/config:/config
      - /volume1/ssd-system/download-temp:/downloads
      - /volume4/media-a:/media-a
      - /volume5/media-b:/media-b
    ports:
      - "8080:8080"
      - "6881:6881"
      - "6881:6881/udp"
    deploy:
      resources:
        limits:
          memory: 1G
    networks:
      - download-net

  aria2:
    image: p3terx/aria2-pro:latest
    container_name: aria2
    restart: always
    environment:
      - PUID=1000
      - PGID=1000
      - RPC_SECRET=${ARIA2_SECRET}
      - RPC_PORT=6800
      - LISTEN_PORT=6888
      - DISK_CACHE=64M
      - IPV6_MODE=false
      - UPDATE_TRACKERS=true
    volumes:
      - ./aria2/config:/config
      - /volume1/ssd-system/download-temp:/downloads
    ports:
      - "6800:6800"
      - "6888:6888"
      - "6888:6888/udp"
    deploy:
      resources:
        limits:
          memory: 512M
    networks:
      - download-net

  ariang:
    image: p3terx/ariang:latest
    container_name: ariang
    restart: always
    ports:
      - "6880:6880"
    networks:
      - download-net

  sonarr:
    image: lscr.io/linuxserver/sonarr:latest
    container_name: sonarr
    restart: always
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai
    volumes:
      - ./sonarr/config:/config
      - /volume4/media-a/tv-shows:/tv
      - /volume1/ssd-system/download-temp:/downloads
    ports:
      - "8989:8989"
    deploy:
      resources:
        limits:
          memory: 512M
    networks:
      - download-net

  radarr:
    image: lscr.io/linuxserver/radarr:latest
    container_name: radarr
    restart: always
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai
    volumes:
      - ./radarr/config:/config
      - /volume4/media-a/movies:/movies
      - /volume1/ssd-system/download-temp:/downloads
    ports:
      - "7878:7878"
    deploy:
      resources:
        limits:
          memory: 512M
    networks:
      - download-net

  jackett:
    image: lscr.io/linuxserver/jackett:latest
    container_name: jackett
    restart: always
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai
      - AUTO_UPDATE=true
    volumes:
      - ./jackett/config:/config
    ports:
      - "9117:9117"
    deploy:
      resources:
        limits:
          memory: 256M
    networks:
      - download-net

  bazarr:
    image: lscr.io/linuxserver/bazarr:latest
    container_name: bazarr
    restart: always
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai
    volumes:
      - ./bazarr/config:/config
      - /volume4/media-a/movies:/movies
      - /volume4/media-a/tv-shows:/tv
    ports:
      - "6767:6767"
    deploy:
      resources:
        limits:
          memory: 256M
    networks:
      - download-net

networks:
  download-net:
    driver: bridge
```

### 3.4 智能家居

```yaml
# docker-compose.smarthome.yml
version: '3.8'

services:
  homeassistant:
    image: ghcr.io/home-assistant/home-assistant:stable
    container_name: homeassistant
    restart: always
    privileged: true
    environment:
      - TZ=Asia/Shanghai
    volumes:
      - /volume1/ssd-system/app-data/home-assistant:/config
      - /run/dbus:/run/dbus:ro
    network_mode: host
    deploy:
      resources:
        limits:
          memory: 1G

  homebridge:
    image: homebridge/homebridge:latest
    container_name: homebridge
    restart: always
    environment:
      - TZ=Asia/Shanghai
      - HOMEBRIDGE_CONFIG_UI_PORT=8581
    volumes:
      - /volume1/ssd-system/app-data/homebridge:/homebridge
    network_mode: host
    deploy:
      resources:
        limits:
          memory: 512M

  node-red:
    image: nodered/node-red:latest
    container_name: node-red
    restart: always
    environment:
      - TZ=Asia/Shanghai
    volumes:
      - ./node-red/data:/data
    ports:
      - "1880:1880"
    deploy:
      resources:
        limits:
          memory: 512M
    networks:
      - smarthome-net

  mosquitto:
    image: eclipse-mosquitto:latest
    container_name: mosquitto
    restart: always
    volumes:
      - ./mosquitto/config:/mosquitto/config
      - ./mosquitto/data:/mosquitto/data
      - ./mosquitto/log:/mosquitto/log
    ports:
      - "1883:1883"
      - "9001:9001"
    deploy:
      resources:
        limits:
          memory: 128M
    networks:
      - smarthome-net

  zigbee2mqtt:
    image: koenkk/zigbee2mqtt:latest
    container_name: zigbee2mqtt
    restart: always
    environment:
      - TZ=Asia/Shanghai
    volumes:
      - ./zigbee2mqtt/data:/app/data
      - /run/udev:/run/udev:ro
    ports:
      - "8085:8080"
    devices:
      - /dev/ttyUSB0:/dev/ttyUSB0  # Zigbee协调器
    deploy:
      resources:
        limits:
          memory: 256M
    networks:
      - smarthome-net

networks:
  smarthome-net:
    driver: bridge
```

### 3.5 家庭服务

```yaml
# docker-compose.family.yml
version: '3.8'

services:
  nextcloud:
    image: nextcloud:latest
    container_name: nextcloud
    restart: always
    environment:
      - MYSQL_HOST=nextcloud-db
      - MYSQL_DATABASE=nextcloud
      - MYSQL_USER=nextcloud
      - MYSQL_PASSWORD=${NEXTCLOUD_DB_PASSWORD}
      - NEXTCLOUD_ADMIN_USER=admin
      - NEXTCLOUD_ADMIN_PASSWORD=${NEXTCLOUD_ADMIN_PASSWORD}
      - NEXTCLOUD_TRUSTED_DOMAINS=cloud.home.local
      - REDIS_HOST=nextcloud-redis
    volumes:
      - ./nextcloud/html:/var/www/html
      - /volume3/critical-data/family-vault:/data/family
    ports:
      - "8083:80"
    depends_on:
      - nextcloud-db
      - nextcloud-redis
    deploy:
      resources:
        limits:
          memory: 1G
    networks:
      - family-net

  nextcloud-db:
    image: mariadb:10.11
    container_name: nextcloud-db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: ${NEXTCLOUD_DB_ROOT_PASSWORD}
      MYSQL_DATABASE: nextcloud
      MYSQL_USER: nextcloud
      MYSQL_PASSWORD: ${NEXTCLOUD_DB_PASSWORD}
    volumes:
      - ./nextcloud/db:/var/lib/mysql
    networks:
      - family-net

  nextcloud-redis:
    image: redis:7-alpine
    container_name: nextcloud-redis
    restart: always
    networks:
      - family-net

  vaultwarden:
    image: vaultwarden/server:latest
    container_name: vaultwarden
    restart: always
    environment:
      - DOMAIN=https://vault.home.local
      - SIGNUPS_ALLOWED=false
      - ADMIN_TOKEN=${VAULTWARDEN_ADMIN_TOKEN}
      - WEBSOCKET_ENABLED=true
    volumes:
      - ./vaultwarden/data:/data
    ports:
      - "8084:80"
    deploy:
      resources:
        limits:
          memory: 256M
    networks:
      - family-net

  alist:
    image: xhofe/alist:latest
    container_name: alist
    restart: always
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Shanghai
    volumes:
      - ./alist/data:/opt/alist/data
      - /volume3/critical-data:/mnt/critical:ro
      - /volume4/media-a:/mnt/media-a:ro
      - /volume5/media-b:/mnt/media-b:ro
    ports:
      - "5244:5244"
    deploy:
      resources:
        limits:
          memory: 512M
    networks:
      - family-net

  syncthing:
    image: syncthing/syncthing:latest
    container_name: syncthing
    restart: always
    hostname: ugreen-nas
    environment:
      - PUID=1000
      - PGID=1000
    volumes:
      - ./syncthing/config:/var/syncthing/config
      - /volume3/critical-data/backup-vault/device-backup:/data/backup
      - /volume1/ssd-system/hot-data/working-files:/data/sync
    ports:
      - "8384:8384"
      - "22000:22000/tcp"
      - "22000:22000/udp"
      - "21027:21027/udp"
    deploy:
      resources:
        limits:
          memory: 512M
    networks:
      - family-net

  filebrowser:
    image: filebrowser/filebrowser:latest
    container_name: filebrowser
    restart: always
    environment:
      - PUID=1000
      - PGID=1000
    volumes:
      - ./filebrowser/database.db:/database.db
      - ./filebrowser/config.json:/.filebrowser.json
      - /volume3/critical-data:/srv/critical
      - /volume4/media-a:/srv/media-a
      - /volume5/media-b:/srv/media-b
    ports:
      - "8086:80"
    deploy:
      resources:
        limits:
          memory: 256M
    networks:
      - family-net

networks:
  family-net:
    driver: bridge
```

### 3.6 网络服务

```yaml
# docker-compose.network.yml
version: '3.8'

services:
  adguardhome:
    image: adguard/adguardhome:latest
    container_name: adguardhome
    restart: always
    volumes:
      - ./adguardhome/work:/opt/adguardhome/work
      - ./adguardhome/conf:/opt/adguardhome/conf
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "3000:3000"  # 首次配置界面
      - "8087:80"    # Web管理界面
    deploy:
      resources:
        limits:
          memory: 256M
    networks:
      - network-net

  unbound:
    image: mvance/unbound:latest
    container_name: unbound
    restart: always
    volumes:
      - ./unbound/config:/opt/unbound/etc/unbound
    ports:
      - "5335:53/tcp"
      - "5335:53/udp"
    deploy:
      resources:
        limits:
          memory: 128M
    networks:
      - network-net

  tailscale:
    image: tailscale/tailscale:latest
    container_name: tailscale
    hostname: ugreen-nas
    restart: always
    environment:
      - TS_AUTHKEY=${TAILSCALE_AUTHKEY}
      - TS_EXTRA_ARGS=--advertise-routes=192.168.1.0/24 --accept-dns=false
      - TS_STATE_DIR=/var/lib/tailscale
    volumes:
      - ./tailscale/state:/var/lib/tailscale
      - /dev/net/tun:/dev/net/tun
    cap_add:
      - net_admin
      - sys_module
    network_mode: host

  ddns-go:
    image: jeessy/ddns-go:latest
    container_name: ddns-go
    restart: always
    volumes:
      - ./ddns-go:/root
    ports:
      - "9876:9876"
    deploy:
      resources:
        limits:
          memory: 64M
    networks:
      - network-net

networks:
  network-net:
    driver: bridge
```

---

## 四、硬盘健康监控

### 4.1 SMART监控脚本

```bash
#!/bin/bash
# /opt/scripts/disk-health-check.sh

# 配置
CRITICAL_DISKS="sdc"  # 14TB二手盘,重点监控
ALERT_EMAIL="your@email.com"
ALERT_WEBHOOK="https://your-webhook-url"

# 检查所有硬盘SMART状态
check_smart() {
    local disk=$1
    local status=$(smartctl -H /dev/$disk | grep -i "result" | awk '{print $NF}')
    local reallocated=$(smartctl -A /dev/$disk | grep "Reallocated_Sector" | awk '{print $10}')
    local pending=$(smartctl -A /dev/$disk | grep "Current_Pending_Sector" | awk '{print $10}')
    local temp=$(smartctl -A /dev/$disk | grep "Temperature" | head -1 | awk '{print $10}')

    echo "Disk: /dev/$disk"
    echo "  Health: $status"
    echo "  Reallocated Sectors: $reallocated"
    echo "  Pending Sectors: $pending"
    echo "  Temperature: ${temp}°C"

    # 告警条件
    if [[ "$status" != "PASSED" ]] || [[ "$reallocated" -gt 100 ]] || [[ "$pending" -gt 0 ]]; then
        send_alert "Disk /dev/$disk health warning!" "Status: $status, Reallocated: $reallocated, Pending: $pending"
    fi

    if [[ "$temp" -gt 50 ]]; then
        send_alert "Disk /dev/$disk temperature warning!" "Current: ${temp}°C"
    fi
}

send_alert() {
    local title=$1
    local message=$2

    # 发送邮件
    echo "$message" | mail -s "$title" $ALERT_EMAIL

    # 发送Webhook
    curl -X POST $ALERT_WEBHOOK \
        -H "Content-Type: application/json" \
        -d "{\"title\":\"$title\",\"message\":\"$message\"}"
}

# 主循环
for disk in sda sdb sdc sdd sde sdf; do
    if [ -b /dev/$disk ]; then
        check_smart $disk
        echo "---"
    fi
done
```

### 4.2 定时任务配置

```bash
# crontab -e

# 每天检查硬盘健康
0 8 * * * /opt/scripts/disk-health-check.sh >> /var/log/disk-health.log 2>&1

# 每周进行SMART短测试
0 3 * * 0 smartctl -t short /dev/sdc  # 重点监控二手盘

# 每月进行SMART长测试
0 4 1 * * smartctl -t long /dev/sdc
```

---

## 五、与极空间联动

### 5.1 数据同步架构

```
┌─────────────────┐                    ┌─────────────────┐
│   极空间 Z423   │                    │  绿联 4800plus  │
│   (开发主力)    │                    │   (媒体中心)    │
├─────────────────┤                    ├─────────────────┤
│                 │                    │                 │
│  GitLab数据 ────┼── rsync/定时 ────>│  灾备副本       │
│                 │                    │                 │
│  数据库备份 ────┼── rsync/定时 ────>│  灾备副本       │
│                 │                    │                 │
│  开发配置 ──────┼── Syncthing ────>│  配置同步       │
│                 │                    │                 │
│                 │<── Syncthing ────┼──  照片原片     │
│                 │                    │   (双向同步)    │
│                 │                    │                 │
│  媒体访问 ──────┼── NFS/SMB ───────┼──  媒体库       │
│   (只读挂载)    │                    │   (主存储)      │
└─────────────────┘                    └─────────────────┘
```

### 5.2 同步脚本

```bash
#!/bin/bash
# /opt/scripts/nas-sync.sh

# 极空间 -> 绿联 (关键数据备份)
ZSPACE_HOST="zspace.local"
UGREEN_BACKUP="/volume5/media-b/disaster-recovery/zspace-backup"

# GitLab备份同步
rsync -avz --delete \
    $ZSPACE_HOST:/volume2/ssd-fast/gitlab-data/backups/ \
    $UGREEN_BACKUP/gitlab/

# 数据库备份同步
rsync -avz --delete \
    $ZSPACE_HOST:/volume3/dev-storage/backups/ \
    $UGREEN_BACKUP/databases/

# 配置文件备份
rsync -avz --delete \
    $ZSPACE_HOST:/volume1/ssd-main/system/docker/ \
    $UGREEN_BACKUP/docker-configs/

echo "Sync completed at $(date)"
```

---

## 六、客户端配置

### 6.1 Mac Mini

```bash
# 挂载媒体库 (只读)
# 在 /etc/fstab 或使用 automount
//ugreen.local/media-a /Volumes/NAS-Media-A cifs ro,credentials=/etc/nas-creds,uid=501,gid=20 0 0

# 安装客户端应用
brew install --cask jellyfin-media-player
brew install --cask infuse

# Syncthing 配置
brew install syncthing
brew services start syncthing
# 访问 http://localhost:8384 配置同步
```

### 6.2 Windows 笔记本/主机

```powershell
# 映射网络驱动器
net use M: \\ugreen.local\media-a /persistent:yes
net use B: \\ugreen.local\backup-vault /persistent:yes

# 配置备份任务 (Windows任务计划程序)
# 使用 robocopy 同步重要文件到 NAS
robocopy "C:\Users\%USERNAME%\Documents" "B:\device-backup\%COMPUTERNAME%\Documents" /MIR /R:3 /W:5
```

### 6.3 手机/平板

| 用途 | iOS | Android |
|------|-----|---------|
| 视频播放 | Infuse Pro / Jellyfin | Jellyfin / VLC |
| 音乐流媒体 | play:Sub / Amperfy | Ultrasonic / Symfonium |
| 照片同步 | PhotoSync / Immich | PhotoSync / Immich |
| 文件管理 | FE File Explorer | Solid Explorer |
| 密码管理 | Bitwarden | Bitwarden |
| 智能家居 | Home app / HA Companion | HA Companion |

---

## 七、服务访问地址

| 服务 | 地址 | 用途 |
|------|------|------|
| Portainer | http://ugreen:9000 | 容器管理 |
| NPM | http://ugreen:81 | 反向代理管理 |
| Jellyfin | http://ugreen:8096 | 视频媒体 |
| Plex | http://ugreen:32400 | 视频媒体(备用) |
| Navidrome | http://ugreen:4533 | 音乐流媒体 |
| Immich | http://ugreen:2283 | 照片管理 |
| Kavita | http://ugreen:5000 | 电子书/漫画 |
| qBittorrent | http://ugreen:8080 | BT下载 |
| Sonarr | http://ugreen:8989 | 剧集追踪 |
| Radarr | http://ugreen:7878 | 电影追踪 |
| Home Assistant | http://ugreen:8123 | 智能家居 |
| Homebridge | http://ugreen:8581 | HomeKit桥接 |
| Node-RED | http://ugreen:1880 | 自动化 |
| Nextcloud | http://ugreen:8083 | 家庭云盘 |
| Vaultwarden | http://ugreen:8084 | 密码管理 |
| Alist | http://ugreen:5244 | 网盘聚合 |
| Syncthing | http://ugreen:8384 | 设备同步 |
| AdGuard Home | http://ugreen:8087 | DNS/广告过滤 |

---

## 八、QLC固态使用注意事项

### 8.1 适合QLC的场景
- ✅ 媒体元数据缓存(读多写少)
- ✅ 缩略图存储
- ✅ 静态资源托管
- ✅ 只读数据集
- ✅ 系统只读分区

### 8.2 避免用QLC的场景
- ❌ 数据库存储
- ❌ Docker数据目录
- ❌ 下载临时目录
- ❌ 日志存储
- ❌ 频繁写入的缓存

### 8.3 QLC寿命监控

```bash
#!/bin/bash
# 检查NVMe磨损度
nvme smart-log /dev/nvme1  # QLC盘

# 关注以下指标:
# - Percentage Used: 磨损百分比
# - Data Units Written: 写入数据量
# - Media Errors: 介质错误数
```

---

## 九、启动脚本

```bash
#!/bin/bash
# /opt/scripts/start-all.sh

cd /opt/docker

echo "Starting base infrastructure..."
docker compose -f docker-compose.base.yml up -d
sleep 10

echo "Starting network services..."
docker compose -f docker-compose.network.yml up -d
sleep 5

echo "Starting media services..."
docker compose -f docker-compose.media.yml up -d

echo "Starting download services..."
docker compose -f docker-compose.download.yml up -d

echo "Starting smart home services..."
docker compose -f docker-compose.smarthome.yml up -d

echo "Starting family services..."
docker compose -f docker-compose.family.yml up -d

echo "All services started!"
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```
