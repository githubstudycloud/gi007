# 🍎 Mac Mini 开发环境完全指南

> 从零开始搭建专业级 macOS 开发环境

## 📋 目录

- [系统准备](#系统准备)
- [Homebrew 包管理](#homebrew-包管理)
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
- [效率工具](#效率工具)
- [系统优化](#系统优化)

---

## 系统准备

### 1. macOS 版本

- **推荐**: macOS Sonoma (14.x) 或 Ventura (13.x)
- Apple Silicon (M1/M2/M3) 或 Intel 均可

```bash
# 查看系统版本
sw_vers

# 查看芯片架构
uname -m
# arm64 = Apple Silicon
# x86_64 = Intel
```

### 2. 安装 Xcode Command Line Tools

```bash
# 安装命令行工具 (必须)
xcode-select --install

# 验证
xcode-select -p
```

### 3. 系统偏好设置

#### 安全与隐私

```
系统偏好设置 → 隐私与安全性 → 允许从以下位置下载的应用 → App Store 和被认可的开发者
```

#### 键盘设置

```
系统偏好设置 → 键盘 →
- 按键重复: 最快
- 重复前延迟: 最短
- 触控栏: 展开的功能控制条
```

#### 触控板 (如连接外接触控板)

```
系统偏好设置 → 触控板 →
- 轻点来点按: 开启
- 三指拖移: 辅助功能 → 指针控制 → 触控板选项
```

### 4. Finder 设置

```bash
# 显示隐藏文件
defaults write com.apple.finder AppleShowAllFiles YES

# 显示路径栏
defaults write com.apple.finder ShowPathbar -bool true

# 显示状态栏
defaults write com.apple.finder ShowStatusBar -bool true

# 显示文件扩展名
defaults write NSGlobalDomain AppleShowAllExtensions -bool true

# 重启 Finder
killall Finder
```

### 5. 创建开发目录

```bash
mkdir -p ~/dev ~/projects ~/Documents/Notes
```

---

## Homebrew 包管理

### 安装 Homebrew

```bash
# 安装 Homebrew
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Apple Silicon Mac 需要添加到 PATH
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"

# 验证
brew --version
```

### 配置镜像 (可选 - 国内加速)

```bash
# 使用清华镜像
export HOMEBREW_BREW_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/brew.git"
export HOMEBREW_CORE_GIT_REMOTE="https://mirrors.tuna.tsinghua.edu.cn/git/homebrew/homebrew-core.git"
export HOMEBREW_BOTTLE_DOMAIN="https://mirrors.tuna.tsinghua.edu.cn/homebrew-bottles"

# 添加到 ~/.zshrc
```

### 常用命令

```bash
brew search <软件名>      # 搜索
brew install <软件名>     # 安装命令行工具
brew install --cask <软件名>  # 安装 GUI 应用
brew uninstall <软件名>   # 卸载
brew update              # 更新 Homebrew
brew upgrade             # 升级所有包
brew list                # 已安装列表
brew doctor              # 诊断问题
brew cleanup             # 清理旧版本
```

### 初始安装清单

```bash
# 基础工具
brew install git curl wget tree jq
brew install vim neovim
brew install htop btop
brew install ripgrep fd bat eza
brew install fzf

# 开发语言 (详见各章节)
brew install --cask temurin  # Java
brew install python          # Python
brew install go              # Go
brew install node            # Node.js

# 终端增强
brew install starship        # 提示符
brew install zsh-autosuggestions
brew install zsh-syntax-highlighting

# 字体
brew tap homebrew/cask-fonts
brew install --cask font-jetbrains-mono-nerd-font
brew install --cask font-fira-code-nerd-font

# 常用应用
brew install --cask visual-studio-code
brew install --cask iterm2
brew install --cask docker
brew install --cask obsidian
brew install --cask raycast
brew install --cask rectangle
```

### Brewfile (批量管理)

创建 `~/Brewfile`:

```ruby
# Taps
tap "homebrew/cask"
tap "homebrew/cask-fonts"

# CLI Tools
brew "git"
brew "curl"
brew "wget"
brew "tree"
brew "jq"
brew "vim"
brew "neovim"
brew "htop"
brew "ripgrep"
brew "fd"
brew "bat"
brew "eza"
brew "fzf"
brew "starship"
brew "zsh-autosuggestions"
brew "zsh-syntax-highlighting"

# Languages
brew "python"
brew "go"
brew "node"

# Fonts
cask "font-jetbrains-mono-nerd-font"
cask "font-fira-code-nerd-font"

# Applications
cask "iterm2"
cask "visual-studio-code"
cask "docker"
cask "obsidian"
cask "raycast"
cask "rectangle"
cask "temurin"
```

使用 Brewfile:

```bash
# 从 Brewfile 安装
brew bundle install

# 导出当前安装
brew bundle dump

# 检查差异
brew bundle check
```

---

## 终端与 Shell

### iTerm2 (推荐)

```bash
brew install --cask iterm2
```

#### 推荐配置

1. **外观**: Preferences → Appearance → Theme → Minimal
2. **字体**: Preferences → Profiles → Text → Font → JetBrainsMono Nerd Font, 14pt
3. **配色**: Preferences → Profiles → Colors → Color Presets → 导入喜欢的主题
4. **透明度**: Preferences → Profiles → Window → Transparency → 10%
5. **无限滚动**: Preferences → Profiles → Terminal → Unlimited scrollback

#### 常用快捷键

| 快捷键 | 功能 |
|--------|------|
| `Cmd + D` | 垂直分屏 |
| `Cmd + Shift + D` | 水平分屏 |
| `Cmd + [/]` | 切换分屏 |
| `Cmd + T` | 新标签 |
| `Cmd + W` | 关闭标签 |
| `Cmd + 数字` | 切换标签 |

### Zsh 配置

macOS 默认使用 Zsh，无需安装。

### Oh My Zsh

```bash
# 安装
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### Zsh 插件

```bash
# zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

# zsh-syntax-highlighting
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

# zsh-completions
git clone https://github.com/zsh-users/zsh-completions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-completions

# fzf 集成
$(brew --prefix)/opt/fzf/install
```

### .zshrc 配置

```bash
# ~/.zshrc

# ===== Oh My Zsh 配置 =====
export ZSH="$HOME/.oh-my-zsh"

# 主题
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
    macos
    brew
    z
    extract
    zsh-autosuggestions
    zsh-syntax-highlighting
)

source $ZSH/oh-my-zsh.sh

# ===== Homebrew =====
eval "$(/opt/homebrew/bin/brew shellenv)"

# ===== 环境变量 =====
export LANG=en_US.UTF-8
export EDITOR='vim'
export PATH="$HOME/.local/bin:$PATH"

# ===== 别名 =====
# 文件操作
alias ll='eza -la --icons'
alias la='eza -a --icons'
alias ls='eza --icons'
alias tree='eza --tree --icons'
alias cat='bat'

# Git
alias g='git'
alias gs='git status'
alias ga='git add'
alias gc='git commit'
alias gp='git push'
alias gl='git pull'
alias glog='git log --oneline --graph --decorate -20'
alias gd='git diff'

# Docker
alias d='docker'
alias dc='docker compose'
alias dps='docker ps'
alias dpsa='docker ps -a'

# 目录
alias ..='cd ..'
alias ...='cd ../..'
alias dev='cd ~/dev'
alias proj='cd ~/projects'

# macOS 特有
alias showfiles='defaults write com.apple.finder AppleShowAllFiles YES; killall Finder'
alias hidefiles='defaults write com.apple.finder AppleShowAllFiles NO; killall Finder'
alias flushdns='sudo dscacheutil -flushcache; sudo killall -HUP mDNSResponder'
alias cleanup='brew cleanup && rm -rf ~/Library/Caches/*'

# 系统
alias ip='ipconfig getifaddr en0'
alias ports='lsof -i -P | grep LISTEN'

# ===== 函数 =====
# 创建并进入目录
mkcd() {
    mkdir -p "$1" && cd "$1"
}

# 快速查找
ff() {
    find . -name "*$1*"
}

# 在 Finder 中打开当前目录
f() {
    open -a Finder "${1:-.}"
}

# 查看端口占用
port() {
    lsof -i :$1
}

# 杀掉端口占用进程
killport() {
    kill -9 $(lsof -t -i:$1)
}

# ===== 工具配置 =====
# fzf
[ -f ~/.fzf.zsh ] && source ~/.fzf.zsh
export FZF_DEFAULT_OPTS='--height 40% --layout=reverse --border'
export FZF_DEFAULT_COMMAND='fd --type f --hidden --follow --exclude .git'

# ===== 语言环境 (详见各章节) =====
# Java - SDKMAN
[[ -s "$HOME/.sdkman/bin/sdkman-init.sh" ]] && source "$HOME/.sdkman/bin/sdkman-init.sh"

# Python - pyenv
export PYENV_ROOT="$HOME/.pyenv"
[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"

# Go
export GOPATH=$HOME/go
export PATH=$GOPATH/bin:$PATH
export GOPROXY=https://goproxy.cn,direct

# Node - nvm
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

# ===== 启动信息 =====
echo "$(date '+%Y-%m-%d %H:%M') | $(whoami)@$(hostname -s)"
```

### Starship 主题 (可选)

```bash
# 安装
brew install starship

# 添加到 ~/.zshrc (替换 Oh My Zsh 主题)
eval "$(starship init zsh)"
```

配置 `~/.config/starship.toml`:

```toml
[character]
success_symbol = "[➜](bold green)"
error_symbol = "[✗](bold red)"

[directory]
truncation_length = 3
truncate_to_repo = true

[git_branch]
symbol = " "
format = "[$symbol$branch]($style) "

[git_status]
format = '([$all_status$ahead_behind]($style) )'

[java]
symbol = " "

[python]
symbol = " "

[golang]
symbol = " "

[nodejs]
symbol = " "

[docker_context]
symbol = " "
```

---

## Git 配置

### 安装

```bash
brew install git gh
```

### 基础配置

```bash
# 用户信息
git config --global user.name "Your Name"
git config --global user.email "your@email.com"

# 默认编辑器
git config --global core.editor "code --wait"

# 换行符 (Mac/Linux)
git config --global core.autocrlf input

# 默认分支
git config --global init.defaultBranch main

# 大小写敏感
git config --global core.ignorecase false
```

### 别名

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

# 添加到 SSH Agent
eval "$(ssh-agent -s)"

# 创建/编辑 SSH 配置
cat >> ~/.ssh/config << 'EOF'
Host *
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519
EOF

# 添加密钥到 Keychain
ssh-add --apple-use-keychain ~/.ssh/id_ed25519

# 查看公钥
cat ~/.ssh/id_ed25519.pub

# 测试连接
ssh -T git@github.com
```

### GitHub CLI

```bash
# 安装
brew install gh

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

### SDKMAN (推荐)

```bash
# 安装
curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"

# 验证
sdk version
```

### 安装 Java

```bash
# 查看可用版本
sdk list java

# 安装 Temurin (推荐)
sdk install java 21.0.2-tem
sdk install java 17.0.10-tem

# 切换版本
sdk use java 21.0.2-tem      # 当前会话
sdk default java 21.0.2-tem  # 默认版本

# 验证
java -version
echo $JAVA_HOME
```

### 备选: Homebrew 安装

```bash
# 安装 Temurin JDK
brew install --cask temurin

# 或指定版本
brew install --cask temurin@17
brew install --cask temurin@21

# 切换版本
export JAVA_HOME=$(/usr/libexec/java_home -v 21)
```

### Maven

```bash
# SDKMAN 安装
sdk install maven

# 或 Homebrew
brew install maven

# 验证
mvn -version
```

配置 `~/.m2/settings.xml`:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings>
    <mirrors>
        <mirror>
            <id>aliyun</id>
            <name>Aliyun Maven</name>
            <url>https://maven.aliyun.com/repository/public</url>
            <mirrorOf>central</mirrorOf>
        </mirror>
    </mirrors>
</settings>
```

### Gradle

```bash
sdk install gradle
# 或
brew install gradle
```

---

## Python 环境

### pyenv (推荐)

```bash
# 安装
brew install pyenv pyenv-virtualenv

# 添加到 ~/.zshrc
export PYENV_ROOT="$HOME/.pyenv"
[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"

# 重新加载
source ~/.zshrc
```

### 安装 Python

```bash
# 安装依赖 (编译需要)
brew install openssl readline sqlite3 xz zlib

# 查看可用版本
pyenv install --list | grep "^\s*3\."

# 安装
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
# 创建配置
mkdir -p ~/.config/pip
cat > ~/.config/pip/pip.conf << 'EOF'
[global]
index-url = https://pypi.tuna.tsinghua.edu.cn/simple
trusted-host = pypi.tuna.tsinghua.edu.cn
EOF
```

### Poetry

```bash
# 安装
curl -sSL https://install.python-poetry.org | python3 -

# 配置
poetry config virtualenvs.in-project true

# 使用
poetry new my-project
poetry install
poetry add requests
poetry shell
```

### 常用工具

```bash
pip install ipython jupyter
pip install black isort flake8 mypy
pip install httpie requests
```

---

## Go 环境

### 安装

```bash
# Homebrew 安装
brew install go

# 验证
go version
```

### 环境变量

添加到 `~/.zshrc`:

```bash
export GOPATH=$HOME/go
export PATH=$GOPATH/bin:$PATH
export GOPROXY=https://goproxy.cn,direct
export GO111MODULE=on
```

### 目录结构

```bash
mkdir -p ~/go/{bin,src,pkg}
```

### 开发工具

```bash
# gopls
go install golang.org/x/tools/gopls@latest

# goimports
go install golang.org/x/tools/cmd/goimports@latest

# staticcheck
go install honnef.co/go/tools/cmd/staticcheck@latest

# air (热重载)
go install github.com/cosmtrek/air@latest
```

---

## Node.js 与 Vue

### nvm (推荐)

```bash
# 安装
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.7/install.sh | bash

# 添加到 ~/.zshrc
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"

# 安装 Node.js
nvm install --lts
nvm alias default lts/*

# 验证
node -v
npm -v
```

### pnpm

```bash
# 安装
npm install -g pnpm

# 配置镜像
pnpm config set registry https://registry.npmmirror.com
```

### Vue 开发

```bash
# 创建项目
pnpm create vue@latest

cd my-vue-app
pnpm install
pnpm run dev
```

---

## Shell 脚本进阶

### 脚本模板

```bash
#!/bin/bash

set -euo pipefail

# 颜色
RED='\033[0;31m'
GREEN='\033[0;32m'
NC='\033[0m'

log_info() { echo -e "${GREEN}[INFO]${NC} $1"; }
log_error() { echo -e "${RED}[ERROR]${NC} $1" >&2; }

# 主逻辑
main() {
    log_info "开始执行..."
    # ...
    log_info "完成"
}

main "$@"
```

### macOS 专用脚本示例

#### 应用管理

```bash
#!/bin/bash
# 快速打开/关闭应用

APP="$1"
ACTION="${2:-toggle}"

if pgrep -x "$APP" > /dev/null; then
    if [[ "$ACTION" == "toggle" || "$ACTION" == "close" ]]; then
        osascript -e "quit app \"$APP\""
        echo "Closed $APP"
    fi
else
    if [[ "$ACTION" == "toggle" || "$ACTION" == "open" ]]; then
        open -a "$APP"
        echo "Opened $APP"
    fi
fi
```

#### 开发环境启动

```bash
#!/bin/bash
# 启动开发环境

# 启动 Docker
open -a Docker

# 等待 Docker 启动
echo "等待 Docker 启动..."
while ! docker info > /dev/null 2>&1; do
    sleep 1
done
echo "Docker 已就绪"

# 启动服务
cd ~/docker/dev-stack
docker compose up -d

# 打开 IDE
open -a "Visual Studio Code"

echo "开发环境已启动"
```

---

## Docker 环境

### 安装 Docker Desktop

```bash
brew install --cask docker

# 启动 Docker Desktop
open -a Docker
```

### 配置

Docker Desktop → Preferences:

1. **Resources**: 分配适当的 CPU 和内存
2. **Docker Engine**: 添加镜像配置

```json
{
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn"
  ]
}
```

### 验证

```bash
docker --version
docker compose version
docker run hello-world
```

### 常用服务

`~/docker/dev-stack/docker-compose.yml`:

```yaml
version: '3.8'

services:
  mysql:
    image: mysql:8
    platform: linux/amd64  # Apple Silicon 兼容
    container_name: dev-mysql
    ports:
      - "3306:3306"
    environment:
      MYSQL_ROOT_PASSWORD: root123
    volumes:
      - mysql_data:/var/lib/mysql

  redis:
    image: redis:7-alpine
    container_name: dev-redis
    ports:
      - "6379:6379"

  postgres:
    image: postgres:16-alpine
    container_name: dev-postgres
    ports:
      - "5432:5432"
    environment:
      POSTGRES_PASSWORD: postgres123

volumes:
  mysql_data:
```

---

## IDE 配置

### VS Code

```bash
brew install --cask visual-studio-code
```

#### 推荐扩展

```bash
code --install-extension ms-vscode-remote.remote-ssh
code --install-extension eamodio.gitlens
code --install-extension esbenp.prettier-vscode
code --install-extension vscjava.vscode-java-pack
code --install-extension ms-python.python
code --install-extension golang.go
code --install-extension vue.volar
code --install-extension github.copilot
```

### JetBrains IDE

```bash
brew install --cask jetbrains-toolbox
```

通过 Toolbox 安装需要的 IDE。

---

## 笔记工具

### Obsidian

```bash
brew install --cask obsidian
```

### 其他选择

```bash
brew install --cask notion
brew install --cask typora
brew install --cask bear  # Apple 原生
```

### iCloud 同步

Obsidian Vault 放在 iCloud Drive 中:

```
~/Library/Mobile Documents/iCloud~md~obsidian/Documents/MyVault
```

---

## 效率工具

### Raycast (启动器 - 强烈推荐)

```bash
brew install --cask raycast
```

替代 Spotlight，支持:
- 快速启动应用
- 剪贴板历史
- 窗口管理
- 代码片段
- 计算器
- 翻译

### Rectangle (窗口管理)

```bash
brew install --cask rectangle
```

快捷键:
- `Ctrl + Option + ←/→`: 左/右半屏
- `Ctrl + Option + ↑`: 最大化
- `Ctrl + Option + Enter`: 全屏

### 其他工具

```bash
# 解压工具
brew install --cask the-unarchiver

# 截图
brew install --cask snipaste

# 菜单栏管理
brew install --cask hiddenbar

# 视频播放
brew install --cask iina

# 快速预览增强
brew install --cask qlmarkdown qlcolorcode qlstephen quicklook-json
```

---

## 系统优化

### 禁用不需要的动画

```bash
# 减少动画
defaults write NSGlobalDomain NSAutomaticWindowAnimationsEnabled -bool false
defaults write NSGlobalDomain NSWindowResizeTime -float 0.001

# Dock 动画
defaults write com.apple.dock autohide-time-modifier -float 0.5
defaults write com.apple.dock autohide-delay -float 0

# 应用动画
killall Dock
```

### 开发相关优化

```bash
# 增加文件描述符限制
echo 'ulimit -n 65536' >> ~/.zshrc

# 禁用 .DS_Store 在网络卷上生成
defaults write com.apple.desktopservices DSDontWriteNetworkStores true
```

### 清理工具

```bash
# Homebrew 清理
brew cleanup

# 系统缓存
rm -rf ~/Library/Caches/*

# 日志
sudo rm -rf /private/var/log/*
```

---

## 快捷键速查

### 系统

| 快捷键 | 功能 |
|--------|------|
| `Cmd + Space` | Spotlight |
| `Cmd + Tab` | 切换应用 |
| `Cmd + Q` | 退出应用 |
| `Cmd + W` | 关闭窗口 |
| `Cmd + ,` | 偏好设置 |
| `Cmd + Shift + 3` | 全屏截图 |
| `Cmd + Shift + 4` | 区域截图 |
| `Cmd + Shift + 5` | 截图工具 |
| `Cmd + Ctrl + Q` | 锁屏 |

### 终端

| 快捷键 | 功能 |
|--------|------|
| `Ctrl + A` | 行首 |
| `Ctrl + E` | 行尾 |
| `Ctrl + U` | 删除到行首 |
| `Ctrl + K` | 删除到行尾 |
| `Ctrl + R` | 搜索历史 |
| `Ctrl + L` | 清屏 |
| `Ctrl + C` | 中断 |
| `Ctrl + Z` | 挂起 |

---

**上一步**: [Ubuntu 本地开发](../02-ubuntu-local/README.md)
**下一步**: [远程 Ubuntu 服务器](../04-ubuntu-remote/README.md)
