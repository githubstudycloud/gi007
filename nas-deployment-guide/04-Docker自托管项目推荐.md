# Docker 自托管项目推荐清单

## 一、开发工具

### 代码与版本控制
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **Gitea** | 轻量级Git托管，GitLab替代 | 内存256MB+ |
| **Forgejo** | Gitea社区分支 | 内存256MB+ |
| **Soft Serve** | 终端风格Git服务器 | 极低 |
| **OneDev** | GitLab替代，内置CI/CD | 内存1GB+ |

```yaml
gitea:
  image: gitea/gitea:latest
  ports:
    - "3000:3000"
    - "2222:22"
  volumes:
    - ./gitea:/data
```

### API开发与测试
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **Hoppscotch** | Postman开源替代 | 低 |
| **Insomnia** | API设计测试 | 中 |
| **Swagger UI** | API文档展示 | 极低 |
| **MockServer** | API Mock服务 | 低 |

```yaml
hoppscotch:
  image: hoppscotch/hoppscotch:latest
  ports:
    - "3000:3000"
```

### 数据库管理
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **Adminer** | 轻量数据库管理 | 极低 |
| **pgAdmin** | PostgreSQL专用 | 低 |
| **phpMyAdmin** | MySQL专用 | 低 |
| **Mongo Express** | MongoDB管理 | 低 |
| **RedisInsight** | Redis可视化 | 低 |
| **DBngin** | 多数据库客户端 | 中 |

```yaml
adminer:
  image: adminer:latest
  ports:
    - "8080:8080"
```

### 代码质量与分析
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **SonarQube** | 代码质量检测 | 内存2GB+ |
| **Codacy** | 自动代码审查 | 高 |
| **Renovate** | 依赖自动更新 | 低 |
| **Dependabot** | GitHub依赖更新 | - |

---

## 二、文档与知识管理

### Wiki与文档
| 项目 | 说明 | 资源需求 | 推荐度 |
|------|------|----------|--------|
| **Outline** | 现代团队Wiki | 内存1GB | ⭐⭐⭐⭐⭐ |
| **Wiki.js** | 功能强大的Wiki | 内存512MB | ⭐⭐⭐⭐⭐ |
| **BookStack** | 书籍式文档组织 | 内存256MB | ⭐⭐⭐⭐ |
| **Docusaurus** | 技术文档站点 | 低 | ⭐⭐⭐⭐ |
| **MkDocs** | Markdown文档 | 极低 | ⭐⭐⭐ |
| **Docsify** | 无需构建的文档 | 极低 | ⭐⭐⭐ |

```yaml
outline:
  image: outlinewiki/outline:latest
  environment:
    - SECRET_KEY=your-secret
    - DATABASE_URL=postgres://user:pass@postgres:5432/outline
    - REDIS_URL=redis://redis:6379
  ports:
    - "3000:3000"

wikijs:
  image: ghcr.io/requarks/wiki:2
  environment:
    - DB_TYPE=postgres
    - DB_HOST=postgres
    - DB_PORT=5432
    - DB_USER=wiki
    - DB_PASS=wikijsrocks
    - DB_NAME=wiki
  ports:
    - "3000:3000"
```

### 笔记应用
| 项目 | 说明 | 资源需求 | 推荐度 |
|------|------|----------|--------|
| **Memos** | 轻量Flomo替代 | 极低 | ⭐⭐⭐⭐⭐ |
| **SiYuan** | 思源笔记 | 中 | ⭐⭐⭐⭐⭐ |
| **Joplin Server** | Joplin同步服务 | 低 | ⭐⭐⭐⭐ |
| **Trilium** | 层级笔记 | 中 | ⭐⭐⭐⭐ |
| **Standard Notes** | 加密笔记 | 中 | ⭐⭐⭐⭐ |
| **Notesnook** | 端到端加密 | 中 | ⭐⭐⭐ |
| **Blinko** | AI驱动笔记 | 中 | ⭐⭐⭐ |

```yaml
memos:
  image: neosmemo/memos:stable
  ports:
    - "5230:5230"
  volumes:
    - ./memos:/var/opt/memos

siyuan:
  image: b3log/siyuan:latest
  ports:
    - "6806:6806"
  volumes:
    - ./siyuan/workspace:/siyuan/workspace
  command: ['--workspace=/siyuan/workspace']
```

### 书签与稍后读
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **Linkwarden** | 书签管理+存档 | 低 |
| **Linkding** | 轻量书签管理 | 极低 |
| **Shiori** | Pocket替代 | 极低 |
| **Wallabag** | 稍后读服务 | 低 |
| **Omnivore** | 现代稍后读 | 中 |
| **Hoarder** | AI书签管理 | 中 |

```yaml
linkwarden:
  image: ghcr.io/linkwarden/linkwarden:latest
  environment:
    - DATABASE_URL=postgresql://user:pass@postgres:5432/linkwarden
    - NEXTAUTH_SECRET=your-secret
    - NEXTAUTH_URL=http://localhost:3000
  ports:
    - "3000:3000"

linkding:
  image: sissbruecker/linkding:latest
  ports:
    - "9090:9090"
  volumes:
    - ./linkding:/etc/linkding/data
```

---

## 三、自动化与效率

### 自动化平台
| 项目 | 说明 | 资源需求 | 推荐度 |
|------|------|----------|--------|
| **n8n** | 工作流自动化 | 内存512MB | ⭐⭐⭐⭐⭐ |
| **Huginn** | 自建IFTTT | 内存1GB | ⭐⭐⭐⭐ |
| **Activepieces** | Zapier替代 | 中 | ⭐⭐⭐⭐ |
| **Automatisch** | 开源Zapier | 中 | ⭐⭐⭐ |
| **Windmill** | 开发者自动化 | 中 | ⭐⭐⭐ |

```yaml
n8n:
  image: n8nio/n8n:latest
  ports:
    - "5678:5678"
  volumes:
    - ./n8n:/home/node/.n8n
  environment:
    - N8N_BASIC_AUTH_ACTIVE=true
    - N8N_BASIC_AUTH_USER=admin
    - N8N_BASIC_AUTH_PASSWORD=password
```

### RSS与信息聚合
| 项目 | 说明 | 资源需求 | 推荐度 |
|------|------|----------|--------|
| **FreshRSS** | 功能完整RSS | 低 | ⭐⭐⭐⭐⭐ |
| **Miniflux** | 极简RSS | 极低 | ⭐⭐⭐⭐⭐ |
| **Tiny Tiny RSS** | 经典RSS | 低 | ⭐⭐⭐⭐ |
| **RSSHub** | RSS生成器 | 低 | ⭐⭐⭐⭐⭐ |
| **Kill the Newsletter** | 邮件转RSS | 极低 | ⭐⭐⭐ |

```yaml
freshrss:
  image: freshrss/freshrss:latest
  ports:
    - "8080:80"
  volumes:
    - ./freshrss/data:/var/www/FreshRSS/data
    - ./freshrss/extensions:/var/www/FreshRSS/extensions

rsshub:
  image: diygod/rsshub:latest
  ports:
    - "1200:1200"
  environment:
    - NODE_ENV=production
    - CACHE_TYPE=redis
    - REDIS_URL=redis://redis:6379/
```

### 任务与日程
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **Vikunja** | Todoist替代 | 低 |
| **Planka** | Trello替代 | 低 |
| **WeKan** | 看板工具 | 中 |
| **Focalboard** | Notion看板替代 | 低 |
| **AppFlowy** | Notion替代 | 中 |

```yaml
vikunja:
  image: vikunja/vikunja:latest
  ports:
    - "3456:3456"
  volumes:
    - ./vikunja/files:/app/vikunja/files
  environment:
    - VIKUNJA_DATABASE_TYPE=sqlite
```

---

## 四、AI与机器学习

### LLM与聊天
| 项目 | 说明 | 资源需求 | 推荐度 |
|------|------|----------|--------|
| **Ollama** | 本地LLM运行 | 内存8GB+ | ⭐⭐⭐⭐⭐ |
| **Open WebUI** | ChatGPT风格界面 | 低 | ⭐⭐⭐⭐⭐ |
| **LocalAI** | OpenAI API兼容 | 高 | ⭐⭐⭐⭐ |
| **Text Generation WebUI** | 模型推理界面 | 高 | ⭐⭐⭐⭐ |
| **LM Studio** | 桌面LLM工具 | - | ⭐⭐⭐⭐ |
| **Jan** | 桌面AI助手 | - | ⭐⭐⭐ |

```yaml
ollama:
  image: ollama/ollama:latest
  ports:
    - "11434:11434"
  volumes:
    - ./ollama:/root/.ollama
  deploy:
    resources:
      reservations:
        devices:
          - driver: nvidia
            count: all
            capabilities: [gpu]

open-webui:
  image: ghcr.io/open-webui/open-webui:main
  ports:
    - "3000:8080"
  volumes:
    - ./open-webui:/app/backend/data
  environment:
    - OLLAMA_BASE_URL=http://ollama:11434
  depends_on:
    - ollama
```

### RAG与知识库
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **AnythingLLM** | 文档问答 | 中 |
| **PrivateGPT** | 私有文档AI | 高 |
| **Quivr** | 第二大脑 | 中 |
| **Dify** | LLM应用开发 | 中 |
| **Flowise** | LLM流程编排 | 低 |
| **Langflow** | 可视化LLM | 中 |

```yaml
anythingllm:
  image: mintplexlabs/anythingllm:latest
  ports:
    - "3001:3001"
  volumes:
    - ./anythingllm:/app/server/storage
  environment:
    - STORAGE_DIR=/app/server/storage

dify:
  image: langgenius/dify-web:latest
  # 需要配合API服务一起部署，见官方docker-compose
```

### 图像生成
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **Stable Diffusion WebUI** | AI绘图 | 显存8GB+ |
| **ComfyUI** | 节点式AI绘图 | 显存8GB+ |
| **Fooocus** | 简化版SD | 显存6GB+ |
| **InvokeAI** | SD管理界面 | 显存8GB+ |

### 语音与转写
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **Whisper** | OpenAI语音识别 | 中-高 |
| **Faster Whisper** | 加速版Whisper | 中 |
| **Piper** | 本地TTS | 低 |

---

## 五、安全与隐私

### 密码管理
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **Vaultwarden** | Bitwarden兼容 | 极低 |
| **Passbolt** | 团队密码管理 | 中 |
| **Padloc** | 现代密码管理 | 低 |

### VPN与网络安全
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **WireGuard** | 现代VPN | 极低 |
| **WG-Easy** | WireGuard WebUI | 极低 |
| **Headscale** | Tailscale自建 | 低 |
| **Netbird** | 零信任网络 | 低 |
| **Firezone** | WireGuard管理 | 低 |
| **Pritunl** | 企业级VPN | 中 |

```yaml
wg-easy:
  image: ghcr.io/wg-easy/wg-easy:latest
  cap_add:
    - NET_ADMIN
    - SYS_MODULE
  sysctls:
    - net.ipv4.conf.all.src_valid_mark=1
    - net.ipv4.ip_forward=1
  ports:
    - "51820:51820/udp"
    - "51821:51821/tcp"
  volumes:
    - ./wg-easy:/etc/wireguard
  environment:
    - WG_HOST=your.domain.com
    - PASSWORD=your-password
```

### 认证与SSO
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **Authentik** | 完整IdP解决方案 | 内存1GB |
| **Authelia** | 轻量认证网关 | 低 |
| **Keycloak** | 企业级IAM | 高 |
| **Zitadel** | 现代IAM | 中 |
| **Casdoor** | 国产IAM | 中 |

```yaml
authelia:
  image: authelia/authelia:latest
  ports:
    - "9091:9091"
  volumes:
    - ./authelia/config:/config
  environment:
    - TZ=Asia/Shanghai
```

### 安全工具
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **CrowdSec** | 协作式防火墙 | 低 |
| **Fail2Ban** | 入侵防护 | 极低 |
| **Wazuh** | SIEM平台 | 高 |
| **OpenVAS** | 漏洞扫描 | 高 |

---

## 六、网络工具

### DNS与广告过滤
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **AdGuard Home** | DNS+广告过滤 | 低 |
| **Pi-hole** | 网络广告过滤 | 低 |
| **Blocky** | 快速DNS代理 | 极低 |
| **Technitium DNS** | 功能完整DNS | 低 |

### 网络监控
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **Speedtest Tracker** | 网速监控 | 低 |
| **Ntopng** | 网络流量分析 | 中 |
| **Gatus** | 服务健康检查 | 极低 |
| **Changedetection** | 网页变化监控 | 低 |
| **Statping-ng** | 状态页面 | 低 |

```yaml
speedtest-tracker:
  image: ghcr.io/alexjustesen/speedtest-tracker:latest
  ports:
    - "8080:80"
  volumes:
    - ./speedtest:/config
  environment:
    - PUID=1000
    - PGID=1000

changedetection:
  image: ghcr.io/dgtlmoon/changedetection.io:latest
  ports:
    - "5000:5000"
  volumes:
    - ./changedetection:/datastore
```

### 代理与隧道
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **frp** | 内网穿透 | 极低 |
| **NPS** | 内网穿透 | 极低 |
| **Cloudflare Tunnel** | 零信任隧道 | 极低 |
| **Nginx Proxy Manager** | 反向代理 | 低 |
| **Traefik** | 云原生代理 | 低 |
| **Caddy** | 自动HTTPS | 低 |

```yaml
frps:  # 服务端
  image: snowdreamtech/frps:latest
  ports:
    - "7000:7000"
    - "7500:7500"
  volumes:
    - ./frp/frps.toml:/etc/frp/frps.toml

cloudflared:
  image: cloudflare/cloudflared:latest
  command: tunnel run
  environment:
    - TUNNEL_TOKEN=your-tunnel-token
```

---

## 七、通讯与协作

### 即时通讯
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **Matrix/Synapse** | 去中心化通讯 | 高 |
| **Element** | Matrix客户端 | - |
| **Rocket.Chat** | Slack替代 | 高 |
| **Mattermost** | 企业协作 | 高 |
| **Zulip** | 话题式聊天 | 中 |

### 邮件服务
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **Mailcow** | 完整邮件方案 | 高 |
| **Mailu** | 轻量邮件套件 | 中 |
| **Poste.io** | 简单邮件服务器 | 中 |
| **Stalwart** | 现代邮件服务器 | 中 |

### 视频会议
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **Jitsi Meet** | 视频会议 | 高 |
| **BigBlueButton** | 在线教室 | 很高 |
| **LiveKit** | 实时音视频 | 高 |

---

## 八、媒体相关补充

### 媒体管理
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **Dim** | 媒体管理器 | 中 |
| **Stash** | 成人内容管理 | 中 |
| **Fileflows** | 媒体自动处理 | 中 |
| **Tdarr** | 视频转码集群 | 高 |
| **Unmanic** | 媒体自动转码 | 中 |

### 字幕工具
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **Bazarr** | 字幕自动下载 | 低 |
| **Subcleaner** | 字幕清理 | 极低 |
| **Subliminal** | 字幕搜索 | 低 |

### 播客与有声书
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **Audiobookshelf** | 有声书服务器 | 低 |
| **Podgrab** | 播客下载 | 低 |
| **PodFetch** | 播客管理 | 低 |

```yaml
audiobookshelf:
  image: ghcr.io/advplyr/audiobookshelf:latest
  ports:
    - "13378:80"
  volumes:
    - ./audiobookshelf/config:/config
    - ./audiobookshelf/metadata:/metadata
    - /path/to/audiobooks:/audiobooks
    - /path/to/podcasts:/podcasts
```

---

## 九、财务与生活

### 财务管理
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **Actual** | 预算管理 | 低 |
| **Firefly III** | 个人财务 | 中 |
| **Maybe** | 财务规划 | 中 |
| **Ghostfolio** | 投资组合追踪 | 中 |

```yaml
actual:
  image: actualbudget/actual-server:latest
  ports:
    - "5006:5006"
  volumes:
    - ./actual:/data
```

### 食谱管理
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **Mealie** | 食谱管理 | 低 |
| **Tandoor** | 食谱数据库 | 中 |
| **Grocy** | 家庭库存管理 | 低 |

```yaml
mealie:
  image: ghcr.io/mealie-recipes/mealie:latest
  ports:
    - "9925:9000"
  volumes:
    - ./mealie:/app/data
  environment:
    - ALLOW_SIGNUP=false
    - TZ=Asia/Shanghai
```

### 健康追踪
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **Grafana + InfluxDB** | 健康数据可视化 | 中 |
| **Home Assistant** | 整合健康设备 | 中 |

---

## 十、游戏相关

### 游戏服务器
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **Minecraft Server** | MC服务器 | 内存2GB+ |
| **Pterodactyl** | 游戏服务器面板 | 中 |
| **AMP** | 游戏服务器管理 | 中 |
| **LinuxGSM** | 游戏服务器脚本 | 因游戏而异 |

```yaml
minecraft:
  image: itzg/minecraft-server:latest
  ports:
    - "25565:25565"
  volumes:
    - ./minecraft:/data
  environment:
    - EULA=TRUE
    - MEMORY=4G
    - TYPE=PAPER
```

### 游戏管理
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **GameVault** | 游戏库管理 | 中 |
| **Romm** | ROM管理 | 低 |
| **EmuDeck** | 模拟器整合 | - |

### 云游戏
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **Sunshine** | 游戏串流主机 | 需要GPU |
| **Moonlight** | 串流客户端 | - |
| **Wolf** | 云游戏服务器 | 需要GPU |

---

## 十一、其他实用工具

### 文件分享
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **Send** | Firefox Send替代 | 低 |
| **PsiTransfer** | 简单文件分享 | 极低 |
| **Pingvin Share** | 现代文件分享 | 低 |
| **Pairdrop** | P2P文件传输 | 极低 |
| **LocalSend** | 局域网传输 | - |

```yaml
pingvin-share:
  image: stonith404/pingvin-share:latest
  ports:
    - "3000:3000"
  volumes:
    - ./pingvin/data:/opt/app/backend/data
```

### 剪贴板同步
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **PrivateBin** | 加密粘贴板 | 极低 |
| **Pastebin** | 代码分享 | 极低 |
| **Clipboard Sync** | 剪贴板同步 | 极低 |

```yaml
privatebin:
  image: privatebin/nginx-fpm-alpine:latest
  ports:
    - "8080:8080"
  volumes:
    - ./privatebin:/srv/data
```

### 短链接
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **Shlink** | 短链接服务 | 低 |
| **YOURLS** | 经典短链接 | 低 |
| **Kutt** | 现代短链接 | 低 |
| **Dub** | 分析型短链接 | 中 |

```yaml
shlink:
  image: shlinkio/shlink:stable
  ports:
    - "8080:8080"
  environment:
    - DEFAULT_DOMAIN=short.example.com
    - IS_HTTPS_ENABLED=true
    - GEOLITE_LICENSE_KEY=your-key
```

### 网页存档
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **ArchiveBox** | 网页存档 | 中 |
| **SingleFile** | 单文件存档 | 极低 |
| **Webrecorder** | 网页录制 | 中 |

```yaml
archivebox:
  image: archivebox/archivebox:latest
  ports:
    - "8000:8000"
  volumes:
    - ./archivebox:/data
  environment:
    - ALLOWED_HOSTS=*
    - MEDIA_MAX_SIZE=750m
```

### 仪表盘
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **Homepage** | 服务仪表盘 | 低 |
| **Dashy** | 功能丰富仪表盘 | 低 |
| **Homer** | 极简仪表盘 | 极低 |
| **Flame** | 应用启动器 | 极低 |
| **Homarr** | 可拖拽仪表盘 | 低 |
| **Mafl** | 简洁仪表盘 | 极低 |

### 其他
| 项目 | 说明 | 资源需求 |
|------|------|----------|
| **IT Tools** | 开发者工具集 | 极低 |
| **CyberChef** | 数据转换工具 | 极低 |
| **Stirling PDF** | PDF工具集 | 中 |
| **Excalidraw** | 手绘风格画板 | 低 |
| **Draw.io** | 流程图工具 | 低 |

```yaml
it-tools:
  image: corentinth/it-tools:latest
  ports:
    - "8080:80"

stirling-pdf:
  image: frooodle/s-pdf:latest
  ports:
    - "8080:8080"
  volumes:
    - ./stirling/training:/usr/share/tessdata
    - ./stirling/config:/configs
```

---

## 十二、推荐部署优先级

### 第一优先级 (必装)
- Memos - 日常记录
- n8n - 自动化
- FreshRSS + RSSHub - 信息获取
- IT Tools - 开发工具集
- Stirling PDF - PDF处理

### 第二优先级 (推荐)
- Outline/Wiki.js - 知识管理
- Ollama + Open WebUI - 本地AI
- Linkwarden - 书签管理
- Vikunja - 任务管理

### 第三优先级 (按需)
- Authentik - SSO统一认证
- Audiobookshelf - 有声书
- ArchiveBox - 网页存档
- Actual - 财务管理

---

## 十三、资源参考

### 优质项目列表
- [Awesome-Selfhosted](https://github.com/awesome-selfhosted/awesome-selfhosted)
- [Awesome-Docker](https://github.com/veggiemonk/awesome-docker)

### 一键部署工具
- [CasaOS](https://casaos.io/) - 简单的NAS系统
- [Cosmos](https://cosmos-cloud.io/) - 自托管平台
- [Runtipi](https://runtipi.io/) - 家庭服务器
- [Umbrel](https://umbrel.com/) - 个人服务器OS

### Docker Compose 模板
- [docker-compose-collection](https://github.com/Haxxnet/Compose-Examples)
- [awesome-compose](https://github.com/docker/awesome-compose)
