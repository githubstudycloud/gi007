# 🐧 Ubuntu 本地开发环境完全指南

> 从零开始搭建专业级 Ubuntu 桌面开发环境

## 📋 目录

- [系统准备](#系统准备)
- [包管理与镜像](#包管理与镜像)
- [终端与 Shell](#终端与-shell)
- [Git 配置](#git-配置)
- [Java 环境](#java-环境)
- [Python 环境](#python-环境)
- [Go 环境](#go-环境)
- [Node.js 与 Vue](#nodejs-与-vue)
- [Shell 脚本进阶](#shell-脚本进阶)
- [Docker 环境](#docker-环境)
- [IDE 配置](#ide-配置)
- [笔记工具](#笔记工具)
- [系统优化](#系统优化)
- [常用命令](#常用命令)

---

## 系统准备

### 1. Ubuntu 版本选择

- **推荐**: Ubuntu 22.04 LTS 或 24.04 LTS
- 桌面环境建议 GNOME (默认)

```bash
# 查看系统版本
lsb_release -a
cat /etc/os-release
```

### 2. 系统更新

```bash
# 更新软件源
sudo apt update

# 升级所有包
sudo apt upgrade -y

# 升级发行版
sudo apt dist-upgrade -y

# 清理旧包
sudo apt autoremove -y
sudo apt autoclean
```

### 3. 安装基础工具

```bash
sudo apt install -y \
    build-essential \
    git curl wget \
    vim neovim \
    htop btop \
    tree jq \
    unzip zip \
    net-tools \
    software-properties-common \
    apt-transport-https \
    ca-certificates \
    gnupg lsb-release
```

### 4. 安装常用字体

```bash
# 安装微软字体
sudo apt install -y ttf-mscorefonts-installer

# 安装开发字体
mkdir -p ~/.local/share/fonts

# 下载 JetBrains Mono Nerd Font
cd /tmp
wget https://github.com/ryanoasis/nerd-fonts/releases/latest/download/JetBrainsMono.zip
unzip JetBrainsMono.zip -d JetBrainsMono
cp JetBrainsMono/*.ttf ~/.local/share/fonts/

# 刷新字体缓存
fc-cache -fv
```

### 5. 中文输入法 (可选)

```bash
# 安装 fcitx5
sudo apt install -y fcitx5 fcitx5-chinese-addons fcitx5-config-qt

# 设置环境变量
cat >> ~/.profile << 'EOF'
export GTK_IM_MODULE=fcitx
export QT_IM_MODULE=fcitx
export XMODIFIERS=@im=fcitx
EOF

# 重新登录后生效
```

---

## 包管理与镜像

### apt 镜像配置

```bash
# 备份原文件
sudo cp /etc/apt/sources.list /etc/apt/sources.list.bak

# 使用清华镜像 (Ubuntu 22.04)
sudo tee /etc/apt/sources.list << 'EOF'
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ jammy-security main restricted universe multiverse
EOF

# 更新
sudo apt update
```

### Snap 包管理

```bash
# Snap 已预装在 Ubuntu 中
snap --version

# 常用命令
snap find <软件名>      # 搜索
snap install <软件名>   # 安装
snap remove <软件名>    # 卸载
snap refresh            # 更新所有
snap list               # 已安装列表
```

### Homebrew (可选)

```bash
# 安装 Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# 添加到 PATH
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> ~/.bashrc
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"

# 安装依赖
sudo apt install build-essential
brew install gcc
```

---

## 终端与 Shell

### 终端模拟器

#### GNOME Terminal (默认)

已预装，配置通过 GUI 进行。

#### Alacritty (推荐 - 高性能)

```bash
# 安装
sudo apt install alacritty

# 或使用 snap
sudo snap install alacritty --classic
```

配置文件 `~/.config/alacritty/alacritty.toml`:

```toml
[font]
size = 12.0

[font.normal]
family = "JetBrainsMono Nerd Font"
style = "Regular"

[font.bold]
family = "JetBrainsMono Nerd Font"
style = "Bold"

[window]
opacity = 0.95
padding = { x = 8, y = 8 }
decorations = "Full"

[colors.primary]
background = "#1e1e2e"
foreground = "#cdd6f4"

[cursor]
style = { shape = "Block", blinking = "On" }
```

#### Tilix

```bash
sudo apt install tilix
```

### Zsh 配置

```bash
# 安装 Zsh
sudo apt install zsh

# 设为默认 Shell
chsh -s $(which zsh)

# 注销重新登录后生效
```

### Oh My Zsh

```bash
# 安装 Oh My Zsh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### Zsh 插件

```bash
# zsh-autosuggestions (自动建议)
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

# zsh-syntax-highlighting (语法高亮)
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

# zsh-completions (额外补全)
git clone https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-completions

# fzf (模糊搜索)
git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf
~/.fzf/install
```

### .zshrc 配置

```bash
# ~/.zshrc

# ===== Oh My Zsh 配置 =====
export ZSH="$HOME/.oh-my-zsh"

# 主题 (推荐: powerlevel10k)
ZSH_THEME="agnoster"

# 插件
plugins=(
    git
    docker
    docker-compose
    kubectl
    golang
    python
    pip
    npm
    node
    extract
    z
    zsh-autosuggestions
    zsh-syntax-highlighting
)

source $ZSH/oh-my-zsh.sh

# ===== 环境变量 =====
export LANG=en_US.UTF-8
export EDITOR='vim'
export PATH="$HOME/.local/bin:$PATH"

# ===== 别名 =====
alias ll='ls -alF'
alias la='ls -A'
alias l='ls -CF'
alias cls='clear'

# Git 别名
alias g='git'
alias gs='git status'
alias ga='git add'
alias gc='git commit'
alias gp='git push'
alias gl='git pull'
alias glog='git log --oneline --graph --decorate -20'
alias gd='git diff'

# Docker 别名
alias d='docker'
alias dc='docker-compose'
alias dps='docker ps'
alias dpsa='docker ps -a'

# 目录导航
alias ..='cd ..'
alias ...='cd ../..'
alias dev='cd ~/dev'
alias proj='cd ~/projects'

# 系统
alias update='sudo apt update && sudo apt upgrade -y'
alias cleanup='sudo apt autoremove -y && sudo apt autoclean'
alias ports='netstat -tulanp'

# ===== 函数 =====
# 创建并进入目录
mkcd() {
    mkdir -p "$1" && cd "$1"
}

# 解压任意格式
extract() {
    if [ -f "$1" ]; then
        case "$1" in
            *.tar.bz2)   tar xjf "$1"     ;;
            *.tar.gz)    tar xzf "$1"     ;;
            *.bz2)       bunzip2 "$1"     ;;
            *.rar)       unrar x "$1"     ;;
            *.gz)        gunzip "$1"      ;;
            *.tar)       tar xf "$1"      ;;
            *.tbz2)      tar xjf "$1"     ;;
            *.tgz)       tar xzf "$1"     ;;
            *.zip)       unzip "$1"       ;;
            *.Z)         uncompress "$1"  ;;
            *.7z)        7z x "$1"        ;;
            *)           echo "'$1' cannot be extracted" ;;
        esac
    else
        echo "'$1' is not a valid file"
    fi
}

# 查找端口占用
port() {
    lsof -i :$1
}

# 快速查看日志
logs() {
    tail -f "$1" | bat --paging=never -l log
}

# ===== 工具配置 =====
# fzf
[ -f ~/.fzf.zsh ] && source ~/.fzf.zsh
export FZF_DEFAULT_OPTS='--height 40% --layout=reverse --border'

# ===== 启动信息 =====
echo "$(date '+%Y-%m-%d %H:%M') | $(whoami)@$(hostname)"
```

### Powerlevel10k 主题 (可选)

```bash
# 安装
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git ${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/themes/powerlevel10k

# 修改 .zshrc
ZSH_THEME="powerlevel10k/powerlevel10k"

# 重启终端后运行配置向导
p10k configure
```

---

## Git 配置

### 安装

```bash
# 安装最新版
sudo add-apt-repository ppa:git-core/ppa
sudo apt update
sudo apt install git
```

### 基础配置

```bash
# 用户信息
git config --global user.name "Your Name"
git config --global user.email "your@email.com"

# 默认编辑器
git config --global core.editor "vim"

# 换行符
git config --global core.autocrlf input

# 默认分支
git config --global init.defaultBranch main

# 彩色输出
git config --global color.ui auto
```

### 别名配置

```bash
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.st status
git config --global alias.unstage 'reset HEAD --'
git config --global alias.last 'log -1 HEAD'
git config --global alias.lg "log --color --graph --pretty=format:'%Cred%h%Creset -%C(yellow)%d%Creset %s %Cgreen(%cr) %C(bold blue)<%an>%Creset' --abbrev-commit"
```

### SSH 密钥

```bash
# 生成密钥
ssh-keygen -t ed25519 -C "your@email.com"

# 启动 SSH Agent
eval "$(ssh-agent -s)"

# 添加密钥
ssh-add ~/.ssh/id_ed25519

# 查看公钥
cat ~/.ssh/id_ed25519.pub

# 测试连接
ssh -T git@github.com
```

### GitHub CLI

```bash
# 安装
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update
sudo apt install gh

# 登录
gh auth login
```

---

## Java 环境

### 安装 OpenJDK

```bash
# 方式1: apt 安装
sudo apt install openjdk-21-jdk

# 方式2: 安装多个版本
sudo apt install openjdk-17-jdk openjdk-21-jdk
```

### SDKMAN (推荐 - 多版本管理)

```bash
# 安装 SDKMAN
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"

# 验证
sdk version

# 查看可用 Java 版本
sdk list java

# 安装 Java
sdk install java 21.0.2-tem      # Temurin 21
sdk install java 17.0.10-tem     # Temurin 17

# 切换版本
sdk use java 21.0.2-tem          # 当前会话
sdk default java 21.0.2-tem      # 设为默认

# 查看已安装
sdk list java | grep installed
```

### 环境变量

```bash
# 添加到 ~/.zshrc 或 ~/.bashrc
export JAVA_HOME=$(dirname $(dirname $(readlink -f $(which java))))
export PATH="$JAVA_HOME/bin:$PATH"
```

### 验证

```bash
java -version
javac -version
echo $JAVA_HOME
```

### Maven

```bash
# apt 安装
sudo apt install maven

# 或 SDKMAN
sdk install maven

# 验证
mvn -version
```

配置镜像 `~/.m2/settings.xml`:

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
            <name>Aliyun Maven</name>
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

### Gradle

```bash
# SDKMAN 安装
sdk install gradle

# 验证
gradle -version
```

---

## Python 环境

### 系统 Python

Ubuntu 默认安装 Python 3，但不建议直接使用系统 Python。

```bash
# 查看版本
python3 --version

# 安装 pip
sudo apt install python3-pip python3-venv
```

### pyenv (推荐 - 多版本管理)

```bash
# 安装依赖
sudo apt install -y make build-essential libssl-dev zlib1g-dev \
    libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm \
    libncursesw5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev \
    libffi-dev liblzma-dev

# 安装 pyenv
curl https://pyenv.run | bash

# 添加到 ~/.zshrc
export PYENV_ROOT="$HOME/.pyenv"
[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"

# 重新加载
source ~/.zshrc

# 查看可用版本
pyenv install --list | grep "^\s*3\."

# 安装 Python
pyenv install 3.12.1
pyenv install 3.11.7

# 设置全局版本
pyenv global 3.12.1

# 项目级版本
cd your-project
pyenv local 3.11.7
```

### pip 配置

```bash
# 创建配置目录
mkdir -p ~/.config/pip

# 配置镜像
cat > ~/.config/pip/pip.conf << 'EOF'
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
trusted-host = pypi.tuna.tsinghua.edu.cn

[install]
trusted-host = pypi.tuna.tsinghua.edu.cn
EOF
```

### 虚拟环境

#### venv (内置)

```bash
# 创建
python -m venv .venv

# 激活
source .venv/bin/activate

# 退出
deactivate
```

#### Poetry (推荐)

```bash
# 安装
curl -sSL https://install.python-poetry.org | python3 -

# 添加到 PATH
export PATH="$HOME/.local/bin:$PATH"

# 配置
poetry config virtualenvs.in-project true

# 使用
poetry new my-project
cd my-project
poetry install
poetry add requests
poetry add pytest --group dev
poetry shell
```

### 常用工具

```bash
pip install --upgrade pip
pip install ipython jupyter
pip install black isort flake8 mypy pylint
pip install httpie requests
```

---

## Go 环境

### 安装

```bash
# 方式1: 官方包
GO_VERSION="1.22.0"
wget https://go.dev/dl/go${GO_VERSION}.linux-amd64.tar.gz
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go${GO_VERSION}.linux-amd64.tar.gz
rm go${GO_VERSION}.linux-amd64.tar.gz

# 方式2: snap
sudo snap install go --classic
```

### 环境变量

添加到 `~/.zshrc`:

```bash
# Go 环境
export GOROOT=/usr/local/go
export GOPATH=$HOME/go
export PATH=$GOROOT/bin:$GOPATH/bin:$PATH

# 代理 (国内必须)
export GOPROXY=https://goproxy.cn,direct
export GO111MODULE=on
export GOPRIVATE=github.com/your-org/*
```

### 验证

```bash
source ~/.zshrc
go version
go env
```

### 目录结构

```bash
mkdir -p ~/go/{bin,src,pkg}
```

### 常用命令

```bash
# 模块操作
go mod init github.com/user/project
go mod tidy
go mod download

# 构建运行
go build -o app .
go run main.go
go test ./...

# 安装工具
go install github.com/xxx/tool@latest
```

### 开发工具

```bash
# gopls (语言服务器)
go install golang.org/x/tools/gopls@latest

# goimports
go install golang.org/x/tools/cmd/goimports@latest

# staticcheck
go install honnef.co/go/tools/cmd/staticcheck@latest

# 热重载
go install github.com/cosmtrek/air@latest

# API 文档
go install github.com/swaggo/swag/cmd/swag@latest
```

---

## Node.js 与 Vue

### nvm (推荐 - 版本管理)

```bash
# 安装 nvm
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# 添加到 ~/.zshrc (安装脚本通常自动添加)
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"

# 重新加载
source ~/.zshrc

# 安装 Node.js
nvm install --lts          # 安装 LTS 版本
nvm install 20             # 安装指定版本
nvm use 20                 # 使用指定版本
nvm alias default 20       # 设置默认版本

# 查看已安装
nvm list
```

### npm 配置

```bash
# 设置镜像
npm config set registry https://registry.npmmirror.com

# 查看配置
npm config list
```

### pnpm (推荐)

```bash
# 安装
npm install -g pnpm

# 配置镜像
pnpm config set registry https://registry.npmmirror.com

# 常用命令
pnpm install
pnpm add <pkg>
pnpm add -D <pkg>
pnpm run dev
```

### Vue 开发

```bash
# 创建 Vue 3 项目
pnpm create vue@latest

# 选择配置
# ✔ Project name: my-vue-app
# ✔ Add TypeScript? Yes
# ✔ Add Vue Router? Yes
# ✔ Add Pinia? Yes
# ✔ Add Vitest? Yes
# ✔ Add ESLint? Yes
# ✔ Add Prettier? Yes

cd my-vue-app
pnpm install
pnpm run dev
```

### 全局包

```bash
pnpm add -g typescript ts-node
pnpm add -g vite
pnpm add -g eslint prettier
pnpm add -g http-server
pnpm add -g pm2
```

---

## Shell 脚本进阶

### Bash 脚本基础

创建脚本模板 `~/bin/script-template.sh`:

```bash
#!/bin/bash

#===============================================================================
# 脚本名称: script-template.sh
# 描述: 脚本模板
# 作者: Your Name
# 日期: 2024-12-01
#===============================================================================

set -euo pipefail  # 严格模式
IFS=$'\n\t'

# 颜色定义
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[0;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

# 日志函数
log_info() { echo -e "${BLUE}[INFO]${NC} $1"; }
log_success() { echo -e "${GREEN}[SUCCESS]${NC} $1"; }
log_warning() { echo -e "${YELLOW}[WARNING]${NC} $1"; }
log_error() { echo -e "${RED}[ERROR]${NC} $1" >&2; }

# 使用说明
usage() {
    cat << EOF
Usage: $(basename "$0") [OPTIONS] <argument>

Description:
    脚本功能描述

Options:
    -h, --help      显示帮助信息
    -v, --verbose   详细输出
    -d, --dry-run   模拟运行

Examples:
    $(basename "$0") -v input.txt
    $(basename "$0") --dry-run input.txt

EOF
    exit 1
}

# 参数解析
VERBOSE=false
DRY_RUN=false

while [[ $# -gt 0 ]]; do
    case $1 in
        -h|--help)
            usage
            ;;
        -v|--verbose)
            VERBOSE=true
            shift
            ;;
        -d|--dry-run)
            DRY_RUN=true
            shift
            ;;
        -*)
            log_error "未知选项: $1"
            usage
            ;;
        *)
            ARGS+=("$1")
            shift
            ;;
    esac
done

# 主函数
main() {
    log_info "脚本开始执行..."

    if [[ $VERBOSE == true ]]; then
        log_info "详细模式已启用"
    fi

    if [[ $DRY_RUN == true ]]; then
        log_warning "模拟运行模式"
    fi

    # 实际逻辑
    log_success "执行完成"
}

# 执行
main "$@"
```

### 常用脚本示例

#### 项目初始化脚本

`~/bin/project-init.sh`:

```bash
#!/bin/bash

set -euo pipefail

PROJECT_NAME="${1:-my-project}"
PROJECT_TYPE="${2:-node}"

echo "创建项目: $PROJECT_NAME (类型: $PROJECT_TYPE)"

mkdir -p "$PROJECT_NAME"
cd "$PROJECT_NAME"

# 初始化 Git
git init

# 创建 .gitignore
cat > .gitignore << 'EOF'
# Dependencies
node_modules/
vendor/
.venv/
__pycache__/

# Build
dist/
build/
target/

# IDE
.idea/
.vscode/
*.swp

# Environment
.env
.env.local

# OS
.DS_Store
Thumbs.db
EOF

# 创建 README
cat > README.md << EOF
# $PROJECT_NAME

## Getting Started

\`\`\`bash
# Installation
# ...
\`\`\`

## Development

\`\`\`bash
# ...
\`\`\`

## License

MIT
EOF

# 根据项目类型初始化
case $PROJECT_TYPE in
    node)
        pnpm init
        ;;
    python)
        poetry init --no-interaction
        ;;
    go)
        go mod init "github.com/user/$PROJECT_NAME"
        ;;
    java)
        mkdir -p src/main/java src/main/resources src/test/java
        ;;
esac

echo "项目 $PROJECT_NAME 创建完成!"
```

#### 服务管理脚本

`~/bin/service-manager.sh`:

```bash
#!/bin/bash

# 常用服务管理

ACTION="${1:-status}"
SERVICE="${2:-all}"

services=("docker" "mysql" "redis" "nginx")

manage_service() {
    local action=$1
    local svc=$2

    case $action in
        start)
            sudo systemctl start "$svc"
            echo "✓ $svc started"
            ;;
        stop)
            sudo systemctl stop "$svc"
            echo "✓ $svc stopped"
            ;;
        restart)
            sudo systemctl restart "$svc"
            echo "✓ $svc restarted"
            ;;
        status)
            systemctl is-active --quiet "$svc" && echo "● $svc: running" || echo "○ $svc: stopped"
            ;;
    esac
}

if [[ $SERVICE == "all" ]]; then
    for svc in "${services[@]}"; do
        manage_service "$ACTION" "$svc" 2>/dev/null || true
    done
else
    manage_service "$ACTION" "$SERVICE"
fi
```

### 添加脚本到 PATH

```bash
# 创建 bin 目录
mkdir -p ~/bin

# 添加到 PATH (在 ~/.zshrc 中)
export PATH="$HOME/bin:$PATH"

# 设置执行权限
chmod +x ~/bin/*.sh
```

---

## Docker 环境

### 安装 Docker

```bash
# 卸载旧版本
sudo apt remove docker docker-engine docker.io containerd runc

# 安装依赖
sudo apt update
sudo apt install -y ca-certificates curl gnupg lsb-release

# 添加 Docker GPG 密钥
sudo mkdir -m 0755 -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

# 添加仓库
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

# 安装 Docker
sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### 配置非 root 用户

```bash
# 添加当前用户到 docker 组
sudo usermod -aG docker $USER

# 重新登录或执行
newgrp docker

# 验证
docker run hello-world
```

### 配置镜像加速

```bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json << 'EOF'
{
    "registry-mirrors": [
        "https://docker.mirrors.ustc.edu.cn",
        "https://registry.docker-cn.com"
    ],
    "dns": ["8.8.8.8", "8.8.4.4"],
    "log-driver": "json-file",
    "log-opts": {
        "max-size": "100m",
        "max-file": "3"
    }
}
EOF

# 重启 Docker
sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 常用命令

```bash
# 镜像
docker images
docker pull nginx
docker rmi nginx
docker build -t myapp .

# 容器
docker ps
docker ps -a
docker run -d -p 80:80 nginx
docker stop <container>
docker rm <container>
docker logs -f <container>
docker exec -it <container> bash

# Compose
docker compose up -d
docker compose down
docker compose logs -f

# 清理
docker system prune -a
docker volume prune
```

### 开发环境 Compose

`~/docker/dev-stack/docker-compose.yml`:

```yaml
version: '3.8'

services:
  mysql:
    image: mysql:8
    container_name: dev-mysql
    restart: unless-stopped
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASE: devdb
    volumes:
      - mysql_data:/var/lib/mysql
    command: --default-authentication-plugin=mysql_native_password

  postgres:
    image: postgres:16-alpine
    container_name: dev-postgres
    restart: unless-stopped
    ports:
      - "5432:5432"
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres123
      POSTGRES_DB: devdb
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    container_name: dev-redis
    restart: unless-stopped
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes

  mongo:
    image: mongo:7
    container_name: dev-mongo
    restart: unless-stopped
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: admin123
    volumes:
      - mongo_data:/data/db

  adminer:
    image: adminer
    container_name: dev-adminer
    restart: unless-stopped
    ports:
      - "8080:8080"
    depends_on:
      - mysql
      - postgres

volumes:
  mysql_data:
  postgres_data:
  redis_data:
  mongo_data:
```

---

## IDE 配置

### VS Code

```bash
# 安装
sudo snap install code --classic

# 或使用 apt
wget -qO- https://packages.microsoft.com/keys/microsoft.asc | gpg --dearmor > packages.microsoft.gpg
sudo install -D -o root -g root -m 644 packages.microsoft.gpg /etc/apt/keyrings/packages.microsoft.gpg
echo "deb [arch=amd64,arm64,armhf signed-by=/etc/apt/keyrings/packages.microsoft.gpg] https://packages.microsoft.com/repos/code stable main" | sudo tee /etc/apt/sources.list.d/vscode.list > /dev/null
sudo apt update
sudo apt install code
```

### 推荐扩展

```bash
# 安装扩展
code --install-extension ms-vscode-remote.remote-ssh
code --install-extension ms-vscode-remote.remote-containers
code --install-extension eamodio.gitlens
code --install-extension esbenp.prettier-vscode
code --install-extension dbaeumer.vscode-eslint

# Java
code --install-extension vscjava.vscode-java-pack

# Python
code --install-extension ms-python.python
code --install-extension ms-python.vscode-pylance

# Go
code --install-extension golang.go

# Vue
code --install-extension vue.volar
```

### JetBrains IDE

```bash
# 安装 Toolbox
wget https://download.jetbrains.com/toolbox/jetbrains-toolbox-2.1.3.18901.tar.gz
tar -xzf jetbrains-toolbox-*.tar.gz
./jetbrains-toolbox-*/jetbrains-toolbox

# 通过 Toolbox 安装:
# - IntelliJ IDEA
# - PyCharm
# - GoLand
# - WebStorm
```

### Neovim (可选)

```bash
# 安装
sudo apt install neovim

# 或最新版
sudo add-apt-repository ppa:neovim-ppa/unstable
sudo apt update
sudo apt install neovim
```

基础配置 `~/.config/nvim/init.lua`:

```lua
-- 基础设置
vim.opt.number = true
vim.opt.relativenumber = true
vim.opt.tabstop = 4
vim.opt.shiftwidth = 4
vim.opt.expandtab = true
vim.opt.smartindent = true
vim.opt.wrap = false
vim.opt.termguicolors = true
vim.opt.scrolloff = 8
vim.opt.signcolumn = "yes"
vim.opt.clipboard = "unnamedplus"

-- Leader 键
vim.g.mapleader = " "

-- 快捷键
vim.keymap.set("n", "<leader>w", ":w<CR>")
vim.keymap.set("n", "<leader>q", ":q<CR>")
vim.keymap.set("n", "<C-h>", "<C-w>h")
vim.keymap.set("n", "<C-l>", "<C-w>l")
vim.keymap.set("n", "<C-j>", "<C-w>j")
vim.keymap.set("n", "<C-k>", "<C-w>k")
```

---

## 笔记工具

### Obsidian

```bash
# Snap 安装
sudo snap install obsidian --classic

# 或 AppImage
wget https://github.com/obsidianmd/obsidian-releases/releases/download/v1.5.3/Obsidian-1.5.3.AppImage
chmod +x Obsidian-*.AppImage
./Obsidian-*.AppImage
```

### Typora

```bash
# 添加仓库
wget -qO - https://typora.io/linux/public-key.asc | sudo tee /etc/apt/trusted.gpg.d/typora.asc
sudo add-apt-repository 'deb https://typora.io/linux ./'
sudo apt update
sudo apt install typora
```

### Joplin (免费替代)

```bash
# 安装
wget -O - https://raw.githubusercontent.com/laurent22/joplin/dev/Joplin_install_and_update.sh | bash
```

### 目录结构

```bash
mkdir -p ~/Documents/Notes/{daily,projects,tech/{java,python,go,vue},reading,templates}
mkdir -p ~/Documents/Projects/{work,personal,learning}
mkdir -p ~/Documents/Resources/{ebooks,cheatsheets,configs}
```

---

## 系统优化

### 性能调优

```bash
# 增加 inotify 监控数量 (大型项目需要)
echo "fs.inotify.max_user_watches=524288" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# 增加文件描述符限制
cat >> ~/.zshrc << 'EOF'
ulimit -n 65535
EOF
```

### 常用 GNOME 扩展

```bash
# 安装扩展管理器
sudo apt install gnome-shell-extension-manager

# 推荐扩展:
# - Dash to Dock
# - AppIndicator
# - Clipboard Indicator
# - Caffeine
```

### 系统监控

```bash
# htop (进程管理)
sudo apt install htop

# btop (美观版)
sudo apt install btop

# 磁盘使用
sudo apt install ncdu

# 网络监控
sudo apt install iftop nethogs
```

---

## 常用命令

### 文件操作

```bash
# 查找文件
find /path -name "*.java"
fd "*.java"              # 更快 (需安装 fd-find)

# 搜索内容
grep -r "pattern" /path
rg "pattern"             # 更快 (需安装 ripgrep)

# 查看文件
cat file.txt
bat file.txt             # 带语法高亮 (需安装 bat)
less file.txt

# 文件大小
du -sh /path
ncdu /path               # 交互式
```

### 进程管理

```bash
# 查看进程
ps aux | grep java
pgrep -f java

# 结束进程
kill <pid>
pkill -f java

# 端口占用
ss -tulanp | grep 8080
lsof -i :8080
```

### 网络

```bash
# 测试连接
ping google.com
curl -I https://api.example.com

# DNS
dig example.com
nslookup example.com

# 下载
wget https://example.com/file.zip
curl -O https://example.com/file.zip
```

### 压缩解压

```bash
# tar
tar -czvf archive.tar.gz folder/
tar -xzvf archive.tar.gz

# zip
zip -r archive.zip folder/
unzip archive.zip
```

---

## 快捷键速查

### 终端

| 快捷键 | 功能 |
|--------|------|
| `Ctrl + A` | 移到行首 |
| `Ctrl + E` | 移到行尾 |
| `Ctrl + U` | 删除到行首 |
| `Ctrl + K` | 删除到行尾 |
| `Ctrl + W` | 删除前一个词 |
| `Ctrl + R` | 搜索历史 |
| `Ctrl + L` | 清屏 |
| `Ctrl + C` | 中断命令 |
| `Ctrl + Z` | 挂起进程 |
| `Ctrl + D` | 退出 Shell |

### GNOME

| 快捷键 | 功能 |
|--------|------|
| `Super` | 活动概览 |
| `Super + A` | 应用程序 |
| `Super + D` | 显示桌面 |
| `Alt + Tab` | 切换窗口 |
| `Super + ←/→` | 窗口左/右半屏 |
| `Super + ↑` | 最大化 |
| `Super + L` | 锁屏 |

---

**上一步**: [Windows 开发环境](../01-windows/README.md)
**下一步**: [Mac Mini 开发环境](../03-mac-mini/README.md)
