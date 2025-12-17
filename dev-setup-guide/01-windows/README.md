# 🪟 Windows 开发环境完全指南

> 从零开始搭建专业级 Windows 开发环境

## 📋 目录

- [系统准备](#系统准备)
- [包管理器](#包管理器)
- [终端配置](#终端配置)
- [Git 配置](#git-配置)
- [Java 环境](#java-环境)
- [Python 环境](#python-环境)
- [Go 环境](#go-环境)
- [Node.js 与 Vue](#nodejs-与-vue)
- [PowerShell 进阶](#powershell-进阶)
- [Docker 环境](#docker-环境)
- [IDE 配置](#ide-配置)
- [WSL2 配置](#wsl2-配置)
- [笔记工具](#笔记工具)
- [效率工具](#效率工具)
- [配置同步](#配置同步)

---

## 系统准备

### 1. Windows 版本要求

- **最低**: Windows 10 21H2
- **推荐**: Windows 11 23H2+

```powershell
# 检查 Windows 版本
winver
```

### 2. 启用开发者模式

```
设置 → 隐私和安全性 → 开发者选项 → 开发人员模式 (开启)
```

### 3. 启用必要的 Windows 功能

以管理员身份运行 PowerShell:

```powershell
# 启用 WSL
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart

# 启用虚拟机平台
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart

# 启用 Hyper-V (Docker 需要)
dism.exe /online /enable-feature /featurename:Microsoft-Hyper-V-All /all /norestart

# 重启计算机
Restart-Computer
```

### 4. 更新 WSL2 内核

```powershell
# 下载并安装 WSL2 内核更新
wsl --update

# 设置默认版本为 WSL2
wsl --set-default-version 2
```

---

## 包管理器

### Winget (Windows 自带)

Windows 11 已内置，Windows 10 需手动安装。

```powershell
# 检查是否已安装
winget --version

# 搜索软件
winget search <软件名>

# 安装软件
winget install <软件ID>

# 升级所有软件
winget upgrade --all
```

### Scoop (推荐 - 开发者友好)

```powershell
# 安装 Scoop
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
Invoke-RestMethod -Uri https://get.scoop.sh | Invoke-Expression

# 添加常用 bucket
scoop bucket add extras
scoop bucket add java
scoop bucket add versions
scoop bucket add nerd-fonts

# 基础工具
scoop install git curl wget 7zip
```

### Chocolatey (备选)

```powershell
# 安装 Chocolatey (管理员权限)
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

### 推荐安装列表 (Scoop)

```powershell
# 开发工具
scoop install git gh lazygit
scoop install vscode sublime-text
scoop install windows-terminal
scoop install docker

# 语言环境
scoop install temurin-lts-jdk  # Java
scoop install python           # Python
scoop install go               # Go
scoop install nodejs-lts       # Node.js

# 实用工具
scoop install fzf ripgrep fd bat eza
scoop install jq yq
scoop install neovim

# 字体
scoop install nerd-fonts/JetBrainsMono-NF
scoop install nerd-fonts/FiraCode-NF
```

---

## 终端配置

### Windows Terminal

```powershell
# 安装
scoop install windows-terminal
# 或
winget install Microsoft.WindowsTerminal
```

### 配置文件位置

```
%LOCALAPPDATA%\Packages\Microsoft.WindowsTerminal_8wekyb3d8bbwe\LocalState\settings.json
```

### 推荐配置

打开 Windows Terminal → 设置 → 打开 JSON 文件:

```json
{
    "$schema": "https://aka.ms/terminal-profiles-schema",
    "defaultProfile": "{574e775e-4f2a-5b96-ac1e-a2962a402336}",
    "profiles": {
        "defaults": {
            "font": {
                "face": "JetBrainsMono Nerd Font",
                "size": 12
            },
            "opacity": 95,
            "useAcrylic": true,
            "padding": "8"
        },
        "list": [
            {
                "guid": "{574e775e-4f2a-5b96-ac1e-a2962a402336}",
                "name": "PowerShell 7",
                "source": "Windows.Terminal.PowershellCore",
                "colorScheme": "One Half Dark"
            },
            {
                "guid": "{2c4de342-38b7-51cf-b940-2309a097f518}",
                "name": "Ubuntu",
                "source": "Windows.Terminal.Wsl"
            }
        ]
    },
    "schemes": [],
    "actions": [
        { "command": "paste", "keys": "ctrl+v" },
        { "command": "copy", "keys": "ctrl+c" },
        { "command": "find", "keys": "ctrl+shift+f" },
        { "command": { "action": "splitPane", "split": "horizontal" }, "keys": "alt+shift+-" },
        { "command": { "action": "splitPane", "split": "vertical" }, "keys": "alt+shift+=" }
    ]
}
```

### Oh My Posh (美化提示符)

```powershell
# 安装
scoop install oh-my-posh

# 查看可用主题
Get-PoshThemes

# 在 PowerShell profile 中启用 (见下节)
```

---

## Git 配置

### 安装

```powershell
scoop install git
```

### 基础配置

```powershell
# 用户信息
git config --global user.name "Your Name"
git config --global user.email "your@email.com"

# 默认编辑器
git config --global core.editor "code --wait"

# 换行符处理 (Windows)
git config --global core.autocrlf true

# 默认分支
git config --global init.defaultBranch main

# 凭证管理
git config --global credential.helper manager
```

### 常用别名

```powershell
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
git config --global alias.d difftool
```

### SSH 密钥配置

```powershell
# 生成密钥
ssh-keygen -t ed25519 -C "your@email.com"

# 启动 SSH Agent 服务
Get-Service ssh-agent | Set-Service -StartupType Automatic
Start-Service ssh-agent

# 添加密钥
ssh-add $env:USERPROFILE\.ssh\id_ed25519

# 查看公钥 (复制到 GitHub)
Get-Content $env:USERPROFILE\.ssh\id_ed25519.pub
```

### GitHub CLI

```powershell
# 安装
scoop install gh

# 登录
gh auth login

# 常用命令
gh repo clone <owner/repo>
gh pr create
gh pr list
gh issue list
```

---

## Java 环境

### 安装 JDK

```powershell
# 使用 Scoop 安装 (推荐 Temurin)
scoop install temurin-lts-jdk

# 或安装多个版本
scoop install temurin17-jdk
scoop install temurin21-jdk
```

### 环境变量 (Scoop 自动配置)

手动配置方式:

```powershell
# 设置 JAVA_HOME
[Environment]::SetEnvironmentVariable("JAVA_HOME", "C:\Users\<用户名>\scoop\apps\temurin-lts-jdk\current", "User")

# 添加到 PATH
$path = [Environment]::GetEnvironmentVariable("PATH", "User")
[Environment]::SetEnvironmentVariable("PATH", "$path;%JAVA_HOME%\bin", "User")
```

### 验证安装

```powershell
java -version
javac -version
echo $env:JAVA_HOME
```

### 多版本管理

```powershell
# 使用 Scoop 切换版本
scoop reset temurin17-jdk  # 切换到 Java 17
scoop reset temurin21-jdk  # 切换到 Java 21
```

### Maven 配置

```powershell
# 安装
scoop install maven

# 验证
mvn -version

# 配置镜像 (阿里云)
# 编辑 %USERPROFILE%\.m2\settings.xml
```

`settings.xml` 内容:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.2.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.2.0
          https://maven.apache.org/xsd/settings-1.2.0.xsd">

    <localRepository>${user.home}/.m2/repository</localRepository>

    <mirrors>
        <mirror>
            <id>aliyun</id>
            <name>Aliyun Maven Mirror</name>
            <url>https://maven.aliyun.com/repository/public</url>
            <mirrorOf>central</mirrorOf>
        </mirror>
    </mirrors>

    <profiles>
        <profile>
            <id>jdk-21</id>
            <activation>
                <activeByDefault>true</activeByDefault>
                <jdk>21</jdk>
            </activation>
            <properties>
                <maven.compiler.source>21</maven.compiler.source>
                <maven.compiler.target>21</maven.compiler.target>
            </properties>
        </profile>
    </profiles>
</settings>
```

### Gradle 配置

```powershell
# 安装
scoop install gradle

# 验证
gradle -version
```

配置镜像 `%USERPROFILE%\.gradle\init.gradle`:

```groovy
allprojects {
    repositories {
        maven { url 'https://maven.aliyun.com/repository/public/' }
        maven { url 'https://maven.aliyun.com/repository/spring/' }
        mavenCentral()
    }
}
```

---

## Python 环境

### 安装 Python

```powershell
# 方式1: Scoop (推荐)
scoop install python

# 方式2: 官方安装器
winget install Python.Python.3.12

# 验证
python --version
pip --version
```

### pip 配置镜像

```powershell
# 创建配置目录
mkdir $env:APPDATA\pip -Force

# 创建配置文件
@"
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
trusted-host = pypi.tuna.tsinghua.edu.cn

[install]
trusted-host = pypi.tuna.tsinghua.edu.cn
"@ | Out-File -FilePath "$env:APPDATA\pip\pip.ini" -Encoding utf8
```

### 虚拟环境管理

#### venv (内置)

```powershell
# 创建虚拟环境
python -m venv .venv

# 激活 (PowerShell)
.\.venv\Scripts\Activate.ps1

# 激活 (CMD)
.\.venv\Scripts\activate.bat

# 退出
deactivate
```

#### Poetry (推荐)

```powershell
# 安装
scoop install poetry

# 或使用官方脚本
(Invoke-WebRequest -Uri https://install.python-poetry.org -UseBasicParsing).Content | python -

# 配置
poetry config virtualenvs.in-project true

# 新建项目
poetry new my-project

# 已有项目初始化
poetry init

# 安装依赖
poetry install

# 添加依赖
poetry add requests
poetry add pytest --group dev

# 运行命令
poetry run python main.py

# 进入虚拟环境
poetry shell
```

#### Conda (数据科学)

```powershell
# 安装 Miniconda
scoop install miniconda3

# 初始化
conda init powershell

# 创建环境
conda create -n myenv python=3.12

# 激活环境
conda activate myenv

# 配置镜像
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
conda config --set show_channel_urls yes
```

### 常用工具

```powershell
# 代码格式化
pip install black isort

# 代码检查
pip install flake8 pylint mypy

# 开发工具
pip install ipython jupyter notebook

# HTTP 工具
pip install httpie requests
```

---

## Go 环境

### 安装

```powershell
# Scoop 安装
scoop install go

# 验证
go version
```

### 环境变量

```powershell
# 设置 GOPATH (可选，默认是 %USERPROFILE%\go)
[Environment]::SetEnvironmentVariable("GOPATH", "$env:USERPROFILE\go", "User")

# 启用 Go Modules
[Environment]::SetEnvironmentVariable("GO111MODULE", "on", "User")

# 设置代理 (国内必须)
[Environment]::SetEnvironmentVariable("GOPROXY", "https://goproxy.cn,direct", "User")
```

立即生效:

```powershell
$env:GOPROXY = "https://goproxy.cn,direct"
$env:GO111MODULE = "on"
```

### 目录结构

```
%USERPROFILE%\go\
├── bin\        # 可执行文件
├── pkg\        # 编译缓存
└── src\        # 源代码 (Go Modules 后不常用)
```

### 常用命令

```powershell
# 初始化模块
go mod init github.com/username/project

# 整理依赖
go mod tidy

# 下载依赖
go mod download

# 构建
go build -o app.exe .

# 运行
go run main.go

# 测试
go test ./...

# 安装工具
go install github.com/xxx/tool@latest
```

### 开发工具

```powershell
# 语言服务器 (VS Code 会自动安装)
go install golang.org/x/tools/gopls@latest

# 代码格式化
go install golang.org/x/tools/cmd/goimports@latest

# 静态检查
go install honnef.co/go/tools/cmd/staticcheck@latest

# 热重载
go install github.com/cosmtrek/air@latest

# API 文档生成
go install github.com/swaggo/swag/cmd/swag@latest
```

---

## Node.js 与 Vue

### 安装 Node.js

```powershell
# 方式1: Scoop
scoop install nodejs-lts

# 方式2: nvm-windows (多版本管理)
scoop install nvm
nvm install lts
nvm use lts

# 验证
node -v
npm -v
```

### npm 配置

```powershell
# 设置镜像
npm config set registry https://registry.npmmirror.com

# 查看配置
npm config list

# 全局包目录
npm config set prefix "$env:USERPROFILE\.npm-global"
```

### pnpm (推荐替代 npm)

```powershell
# 安装
npm install -g pnpm

# 或 Scoop
scoop install pnpm

# 设置镜像
pnpm config set registry https://registry.npmmirror.com

# 常用命令
pnpm install          # 安装依赖
pnpm add <pkg>        # 添加依赖
pnpm add -D <pkg>     # 添加开发依赖
pnpm remove <pkg>     # 移除依赖
pnpm run dev          # 运行脚本
```

### Vue 开发

```powershell
# 创建 Vue 3 项目 (推荐使用 create-vue)
pnpm create vue@latest

# 交互式选择:
# ✔ Project name: my-vue-app
# ✔ Add TypeScript? Yes
# ✔ Add JSX Support? No
# ✔ Add Vue Router? Yes
# ✔ Add Pinia? Yes
# ✔ Add Vitest? Yes
# ✔ Add ESLint? Yes
# ✔ Add Prettier? Yes

# 进入项目
cd my-vue-app
pnpm install
pnpm run dev
```

### 常用全局包

```powershell
pnpm add -g typescript ts-node
pnpm add -g @vue/cli
pnpm add -g vite
pnpm add -g eslint prettier
pnpm add -g http-server
pnpm add -g pm2
```

---

## PowerShell 进阶

### 安装 PowerShell 7

```powershell
# Winget
winget install Microsoft.PowerShell

# 或 Scoop
scoop install pwsh
```

### Profile 配置

```powershell
# 查看 Profile 路径
$PROFILE

# 创建 Profile
if (!(Test-Path -Path $PROFILE)) {
    New-Item -ItemType File -Path $PROFILE -Force
}

# 编辑 Profile
code $PROFILE
```

### 推荐 Profile 内容

```powershell
# PowerShell Profile
# 路径: $HOME\Documents\PowerShell\Microsoft.PowerShell_profile.ps1

# ===== Oh My Posh 主题 =====
oh-my-posh init pwsh --config "$env:POSH_THEMES_PATH\powerlevel10k_rainbow.omp.json" | Invoke-Expression

# ===== 模块导入 =====
Import-Module posh-git        # Git 集成
Import-Module Terminal-Icons  # 文件图标
Import-Module PSReadLine      # 命令行增强

# ===== PSReadLine 配置 =====
Set-PSReadLineOption -PredictionSource History
Set-PSReadLineOption -PredictionViewStyle ListView
Set-PSReadLineOption -EditMode Windows
Set-PSReadLineKeyHandler -Key Tab -Function MenuComplete
Set-PSReadLineKeyHandler -Key UpArrow -Function HistorySearchBackward
Set-PSReadLineKeyHandler -Key DownArrow -Function HistorySearchForward

# ===== 常用别名 =====
Set-Alias -Name vim -Value nvim
Set-Alias -Name g -Value git
Set-Alias -Name k -Value kubectl
Set-Alias -Name d -Value docker
Set-Alias -Name dc -Value docker-compose
Set-Alias -Name py -Value python
Set-Alias -Name open -Value explorer

# ===== 函数 =====
# 快速进入目录
function dev { Set-Location D:\dev }
function proj { Set-Location D:\projects }
function home { Set-Location $HOME }

# 创建并进入目录
function mkcd {
    param([string]$dir)
    New-Item -ItemType Directory -Path $dir -Force | Out-Null
    Set-Location $dir
}

# 查看目录大小
function dirsize {
    param([string]$path = ".")
    Get-ChildItem -Path $path -Recurse | Measure-Object -Property Length -Sum |
    Select-Object @{Name="Size(MB)";Expression={[math]::Round($_.Sum / 1MB, 2)}}
}

# 搜索历史命令
function hg {
    param([string]$pattern)
    Get-Content (Get-PSReadLineOption).HistorySavePath | Select-String $pattern
}

# Git 快捷函数
function gs { git status }
function ga { git add . }
function gc { param([string]$msg) git commit -m $msg }
function gp { git push }
function gl { git pull }
function glog { git log --oneline -20 }
function gd { git diff }

# 查找端口占用
function port {
    param([int]$portNumber)
    Get-NetTCPConnection -LocalPort $portNumber -ErrorAction SilentlyContinue |
    Select-Object LocalPort, OwningProcess, @{Name="ProcessName";Expression={(Get-Process -Id $_.OwningProcess).ProcessName}}
}

# 杀死端口占用进程
function killport {
    param([int]$portNumber)
    $conn = Get-NetTCPConnection -LocalPort $portNumber -ErrorAction SilentlyContinue
    if ($conn) {
        Stop-Process -Id $conn.OwningProcess -Force
        Write-Host "Killed process on port $portNumber"
    } else {
        Write-Host "No process found on port $portNumber"
    }
}

# 快速编辑配置文件
function editprofile { code $PROFILE }
function edithosts { code C:\Windows\System32\drivers\etc\hosts }

# 环境信息
function sysinfo {
    Write-Host "PowerShell: $($PSVersionTable.PSVersion)" -ForegroundColor Cyan
    Write-Host "OS: $([System.Environment]::OSVersion.VersionString)" -ForegroundColor Cyan
    Write-Host "User: $env:USERNAME@$env:COMPUTERNAME" -ForegroundColor Cyan
}

# ===== 启动信息 =====
Write-Host "PowerShell $($PSVersionTable.PSVersion) | $(Get-Date -Format 'yyyy-MM-dd HH:mm')" -ForegroundColor DarkGray
```

### 安装推荐模块

```powershell
# 安装模块
Install-Module posh-git -Scope CurrentUser -Force
Install-Module Terminal-Icons -Scope CurrentUser -Force
Install-Module PSReadLine -Scope CurrentUser -Force
Install-Module PSFzf -Scope CurrentUser -Force  # fzf 集成
```

---

## Docker 环境

### 安装 Docker Desktop

```powershell
# Winget 安装
winget install Docker.DockerDesktop

# 或下载安装
# https://www.docker.com/products/docker-desktop/
```

### 配置

启动 Docker Desktop，进入设置:

1. **General**:
   - ✅ Use the WSL 2 based engine
   - ✅ Start Docker Desktop when you sign in

2. **Resources → WSL Integration**:
   - ✅ Enable integration with my default WSL distro
   - ✅ Ubuntu (如已安装)

3. **Docker Engine** (配置镜像):

```json
{
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn",
    "https://registry.docker-cn.com"
  ],
  "dns": ["8.8.8.8", "8.8.4.4"]
}
```

### 验证安装

```powershell
docker --version
docker-compose --version
docker run hello-world
```

### 常用命令

```powershell
# 镜像管理
docker images                    # 列出镜像
docker pull nginx               # 拉取镜像
docker rmi <image>              # 删除镜像
docker build -t myapp .         # 构建镜像

# 容器管理
docker ps                       # 运行中的容器
docker ps -a                    # 所有容器
docker run -d -p 80:80 nginx    # 运行容器
docker stop <container>         # 停止容器
docker rm <container>           # 删除容器
docker logs <container>         # 查看日志
docker exec -it <container> sh  # 进入容器

# 资源清理
docker system prune             # 清理未使用资源
docker system prune -a          # 清理所有未使用资源
```

### Docker Compose 示例

`docker-compose.yml`:

```yaml
version: '3.8'

services:
  mysql:
    image: mysql:8
    container_name: mysql
    restart: unless-stopped
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: devdb
    volumes:
      - mysql_data:/var/lib/mysql

  redis:
    image: redis:7-alpine
    container_name: redis
    restart: unless-stopped
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

  postgres:
    image: postgres:16-alpine
    container_name: postgres
    restart: unless-stopped
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres123
      POSTGRES_DB: devdb
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  mysql_data:
  redis_data:
  postgres_data:
```

---

## IDE 配置

### VS Code

#### 安装

```powershell
scoop install vscode
# 或
winget install Microsoft.VisualStudioCode
```

#### 推荐扩展

```powershell
# 通用
code --install-extension ms-vscode.powershell
code --install-extension ms-vscode-remote.remote-ssh
code --install-extension ms-vscode-remote.remote-wsl
code --install-extension ms-vscode-remote.remote-containers
code --install-extension eamodio.gitlens
code --install-extension github.copilot
code --install-extension esbenp.prettier-vscode
code --install-extension editorconfig.editorconfig
code --install-extension usernamehw.errorlens
code --install-extension streetsidesoftware.code-spell-checker

# Java
code --install-extension vscjava.vscode-java-pack
code --install-extension vmware.vscode-spring-boot

# Python
code --install-extension ms-python.python
code --install-extension ms-python.vscode-pylance
code --install-extension ms-python.black-formatter

# Go
code --install-extension golang.go

# Vue/前端
code --install-extension vue.volar
code --install-extension dbaeumer.vscode-eslint

# 主题和图标
code --install-extension pkief.material-icon-theme
code --install-extension github.github-vscode-theme
```

#### settings.json

```json
{
    // 编辑器基础
    "editor.fontSize": 14,
    "editor.fontFamily": "'JetBrains Mono', 'Fira Code', Consolas, monospace",
    "editor.fontLigatures": true,
    "editor.tabSize": 4,
    "editor.insertSpaces": true,
    "editor.wordWrap": "on",
    "editor.minimap.enabled": false,
    "editor.renderWhitespace": "boundary",
    "editor.bracketPairColorization.enabled": true,
    "editor.guides.bracketPairs": true,
    "editor.inlineSuggest.enabled": true,
    "editor.formatOnSave": true,
    "editor.codeActionsOnSave": {
        "source.fixAll.eslint": "explicit",
        "source.organizeImports": "explicit"
    },

    // 文件
    "files.autoSave": "afterDelay",
    "files.autoSaveDelay": 1000,
    "files.trimTrailingWhitespace": true,
    "files.insertFinalNewline": true,
    "files.exclude": {
        "**/.git": true,
        "**/.DS_Store": true,
        "**/node_modules": true,
        "**/__pycache__": true,
        "**/.venv": true
    },

    // 终端
    "terminal.integrated.defaultProfile.windows": "PowerShell",
    "terminal.integrated.fontFamily": "JetBrainsMono Nerd Font",
    "terminal.integrated.fontSize": 13,

    // Git
    "git.autofetch": true,
    "git.confirmSync": false,
    "git.enableSmartCommit": true,

    // 语言特定
    "[python]": {
        "editor.defaultFormatter": "ms-python.black-formatter",
        "editor.tabSize": 4
    },
    "[javascript][typescript][vue][json]": {
        "editor.defaultFormatter": "esbenp.prettier-vscode",
        "editor.tabSize": 2
    },
    "[java]": {
        "editor.tabSize": 4
    },
    "[go]": {
        "editor.defaultFormatter": "golang.go",
        "editor.tabSize": 4
    },
    "[markdown]": {
        "editor.wordWrap": "on",
        "editor.quickSuggestions": {
            "other": true,
            "comments": false,
            "strings": true
        }
    },

    // Python
    "python.defaultInterpreterPath": "python",
    "[python]": {
        "editor.formatOnSave": true,
        "editor.defaultFormatter": "ms-python.black-formatter"
    },

    // Go
    "go.useLanguageServer": true,
    "go.lintTool": "golangci-lint",
    "[go]": {
        "editor.formatOnSave": true,
        "editor.codeActionsOnSave": {
            "source.organizeImports": "explicit"
        }
    },

    // 工作台
    "workbench.colorTheme": "GitHub Dark Default",
    "workbench.iconTheme": "material-icon-theme",
    "workbench.startupEditor": "none"
}
```

### JetBrains IDEs

#### 安装

```powershell
# 使用 Toolbox 统一管理
scoop install jetbrains-toolbox
```

通过 Toolbox 安装:
- IntelliJ IDEA (Java)
- PyCharm (Python)
- GoLand (Go)
- WebStorm (Vue/前端)

#### 通用设置

1. **外观**: Settings → Appearance → Theme → Darcula
2. **字体**: Settings → Editor → Font → JetBrains Mono, Size: 14
3. **编码**: Settings → Editor → File Encodings → UTF-8
4. **快捷键**: Settings → Keymap → VS Code (如果习惯 VS Code)

#### 插件推荐

- **Chinese Language Pack** - 中文语言包
- **GitToolBox** - Git 增强
- **Rainbow Brackets** - 彩虹括号
- **.ignore** - 忽略文件支持
- **String Manipulation** - 字符串处理
- **Key Promoter X** - 快捷键提示

---

## WSL2 配置

### 安装 Ubuntu

```powershell
# 查看可用发行版
wsl --list --online

# 安装 Ubuntu
wsl --install -d Ubuntu-22.04

# 设置默认发行版
wsl --set-default Ubuntu-22.04
```

### 基础配置

进入 WSL:

```bash
# 更新系统
sudo apt update && sudo apt upgrade -y

# 安装基础工具
sudo apt install -y git curl wget vim build-essential

# 安装开发工具
sudo apt install -y zsh tmux htop tree jq

# 切换到 Zsh
chsh -s $(which zsh)
```

### Oh My Zsh

```bash
# 安装 Oh My Zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"

# 安装常用插件
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

# 编辑配置
vim ~/.zshrc
```

`.zshrc` 关键配置:

```bash
# 主题
ZSH_THEME="agnoster"

# 插件
plugins=(
    git
    docker
    docker-compose
    kubectl
    zsh-autosuggestions
    zsh-syntax-highlighting
)
```

### WSL 与 Windows 互操作

```bash
# 访问 Windows 文件
cd /mnt/c/Users/YourName

# 从 WSL 打开 Windows 程序
explorer.exe .
code .

# 从 Windows 访问 WSL 文件
# 资源管理器地址栏输入: \\wsl$\Ubuntu-22.04
```

### WSL 配置文件

创建 `%USERPROFILE%\.wslconfig`:

```ini
[wsl2]
memory=8GB
processors=4
swap=4GB
localhostForwarding=true

[experimental]
autoMemoryReclaim=gradual
```

---

## 笔记工具

### Obsidian

```powershell
# 安装
scoop install obsidian
# 或
winget install Obsidian.Obsidian
```

**推荐插件**:
- **Calendar** - 日历视图
- **Dataview** - 数据查询
- **Templater** - 模板系统
- **Git** - Git 同步
- **Excalidraw** - 手绘图
- **Advanced Tables** - 表格增强

### Notion

```powershell
winget install Notion.Notion
```

### Typora

```powershell
scoop install typora
```

### 知识管理目录结构

```
D:\Notes\                    # Obsidian Vault
├── 00-Inbox\               # 临时收集
├── 01-Daily\               # 每日笔记
│   └── 2024\
│       └── 12\
├── 02-Projects\            # 项目笔记
│   ├── project-a\
│   └── project-b\
├── 03-Areas\               # 持续关注领域
│   ├── Career\
│   ├── Health\
│   └── Finance\
├── 04-Resources\           # 知识库
│   ├── Tech\
│   │   ├── Java\
│   │   ├── Python\
│   │   ├── Go\
│   │   └── Vue\
│   ├── Tools\
│   └── Books\
├── 05-Archive\             # 归档
├── Templates\              # 模板
│   ├── daily.md
│   ├── meeting.md
│   └── project.md
└── Attachments\            # 附件
```

---

## 效率工具

### 必装工具

```powershell
# 文件搜索
scoop install everything

# 启动器
scoop install flow-launcher    # 或 PowerToys Run

# 截图
scoop install snipaste

# 剪贴板历史
scoop install ditto

# PDF 阅读
scoop install sumatrapdf

# 解压缩
scoop install 7zip

# 系统增强
winget install Microsoft.PowerToys
```

### PowerToys 推荐功能

- **PowerToys Run** - 快速启动 (Alt+Space)
- **FancyZones** - 窗口布局管理
- **Color Picker** - 取色器 (Win+Shift+C)
- **File Explorer Add-ons** - 文件预览增强
- **Keyboard Manager** - 按键重映射

### 其他推荐

```powershell
# API 调试
scoop install postman
# 或轻量级
scoop install insomnia

# 数据库客户端
scoop install dbeaver

# Redis 客户端
scoop install another-redis-desktop-manager

# 远程桌面
winget install Rustdesk.Rustdesk

# 密码管理
scoop install bitwarden
```

---

## 配置同步

### Dotfiles 仓库

```powershell
# 创建 dotfiles 仓库
mkdir ~\.dotfiles
cd ~\.dotfiles
git init

# 复制配置文件
cp $PROFILE .\PowerShell_profile.ps1
cp ~\.gitconfig .\gitconfig
cp "$env:LOCALAPPDATA\Packages\Microsoft.WindowsTerminal_8wekyb3d8bbwe\LocalState\settings.json" .\terminal-settings.json
```

创建安装脚本 `install.ps1`:

```powershell
# Dotfiles 安装脚本

$DotfilesDir = $PSScriptRoot

# 创建符号链接
function New-SymLink {
    param([string]$Source, [string]$Target)
    if (Test-Path $Target) {
        Remove-Item $Target -Force
    }
    New-Item -ItemType SymbolicLink -Path $Target -Target $Source
}

# PowerShell Profile
New-SymLink "$DotfilesDir\PowerShell_profile.ps1" $PROFILE

# Git 配置
New-SymLink "$DotfilesDir\gitconfig" "$HOME\.gitconfig"

# Windows Terminal
$TerminalSettings = "$env:LOCALAPPDATA\Packages\Microsoft.WindowsTerminal_8wekyb3d8bbwe\LocalState\settings.json"
New-SymLink "$DotfilesDir\terminal-settings.json" $TerminalSettings

Write-Host "Dotfiles 安装完成!" -ForegroundColor Green
```

### VS Code 设置同步

1. 登录 GitHub/Microsoft 账号
2. Settings → Turn on Settings Sync
3. 选择要同步的内容

### Scoop 导出/导入

```powershell
# 导出已安装应用列表
scoop export > scoop-apps.json

# 在新机器上导入
scoop import scoop-apps.json
```

---

## 快捷键速查

### Windows 系统

| 快捷键 | 功能 |
|--------|------|
| `Win + V` | 剪贴板历史 |
| `Win + .` | 表情符号 |
| `Win + Shift + S` | 截图 |
| `Win + Tab` | 任务视图 |
| `Alt + Tab` | 切换窗口 |
| `Win + D` | 显示桌面 |
| `Win + E` | 打开资源管理器 |
| `Win + I` | 打开设置 |

### VS Code

| 快捷键 | 功能 |
|--------|------|
| `Ctrl + P` | 快速打开文件 |
| `Ctrl + Shift + P` | 命令面板 |
| `Ctrl + B` | 切换侧边栏 |
| `Ctrl + ` ` | 切换终端 |
| `Ctrl + /` | 注释 |
| `Alt + ↑/↓` | 移动行 |
| `Ctrl + D` | 选择下一个匹配 |
| `F12` | 跳转定义 |
| `Shift + F12` | 查看引用 |

### PowerShell

| 快捷键 | 功能 |
|--------|------|
| `Ctrl + R` | 搜索历史 |
| `Tab` | 自动补全 |
| `Ctrl + L` | 清屏 |
| `Ctrl + C` | 中断命令 |
| `↑/↓` | 历史命令 |

---

## 常见问题

### Q: PowerShell 执行策略报错
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### Q: Scoop 下载慢
```powershell
# 使用代理
scoop config proxy 127.0.0.1:7890

# 或使用镜像
scoop config SCOOP_REPO https://gitee.com/glsnames/scoop-installer
```

### Q: Docker 启动失败
1. 确保 Hyper-V 和 WSL2 已启用
2. 在 BIOS 中启用虚拟化
3. 重启 Docker Desktop

### Q: Git 中文乱码
```powershell
git config --global core.quotepath false
git config --global gui.encoding utf-8
git config --global i18n.commitencoding utf-8
git config --global i18n.logoutputencoding utf-8
```

### Q: npm 全局包权限问题
```powershell
npm config set prefix "$env:USERPROFILE\.npm-global"
# 添加到 PATH
[Environment]::SetEnvironmentVariable("PATH", "$env:PATH;$env:USERPROFILE\.npm-global", "User")
```

---

**下一步**: [Ubuntu 本地开发环境](../02-ubuntu-local/README.md)
