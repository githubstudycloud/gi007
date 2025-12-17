# macOS 开发环境配置指南

## 1. 安装 Homebrew（包管理器）

Homebrew 是 macOS 上最重要的软件管理工具：

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

安装完成后，根据提示将 Homebrew 添加到 PATH：

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

## 2. 安装开发语言

### Python（使用 pyenv 管理版本）

```bash
brew install pyenv

# 添加到 shell 配置
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.zshrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.zshrc
echo 'eval "$(pyenv init -)"' >> ~/.zshrc

source ~/.zshrc

# 安装 Python
pyenv install 3.12
pyenv global 3.12
```

### Go

```bash
brew install go

# 验证安装
go version
```

### Java（使用 OpenJDK）

```bash
brew install openjdk@21

# 添加到 PATH
echo 'export PATH="/opt/homebrew/opt/openjdk@21/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc

# 验证安装
java -version
```

### Node.js（使用 nvm 管理版本）

```bash
brew install nvm

# 创建 nvm 目录
mkdir ~/.nvm

# 添加到 shell 配置
echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.zshrc
echo '[ -s "/opt/homebrew/opt/nvm/nvm.sh" ] && \. "/opt/homebrew/opt/nvm/nvm.sh"' >> ~/.zshrc
source ~/.zshrc

# 安装 Node.js LTS 版本
nvm install --lts
nvm use --lts

# 验证安装
node -v
npm -v
```

## 3. 管理软件更新

### 更新所有软件

```bash
brew update && brew upgrade
```

### 查看可更新的软件

```bash
brew outdated
```

### 更新特定软件

```bash
brew upgrade <软件名>
```

### 清理旧版本

```bash
brew cleanup
```

### 查看已安装的软件

```bash
brew list
```

## 4. 生成 SSH 公钥

### 生成密钥（推荐 Ed25519）

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
```

或者使用 RSA：

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

### 生成过程

1. **保存位置**：直接回车使用默认路径 `~/.ssh/id_ed25519`
2. **密码**：可以设置密码保护私钥，或直接回车留空

### 查看公钥

```bash
cat ~/.ssh/id_ed25519.pub
```

### 复制公钥到剪贴板

```bash
pbcopy < ~/.ssh/id_ed25519.pub
```

### 添加到 GitHub/GitLab

1. 打开 GitHub → Settings → SSH and GPG keys
2. 点击 "New SSH key"
3. 粘贴公钥内容
4. 保存

### 测试连接

```bash
ssh -T git@github.com
```

## 5. 推荐工具

### IDE/编辑器

```bash
# VS Code
brew install --cask visual-studio-code

# JetBrains 全家桶
brew install --cask intellij-idea    # Java
brew install --cask goland           # Go
brew install --cask pycharm          # Python
brew install --cask webstorm         # Node.js
```

### 终端工具

```bash
# iTerm2（更好的终端）
brew install --cask iterm2

# Oh My Zsh（终端美化）
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### Git 配置

```bash
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
```

## 6. 常用目录结构

建议的项目目录结构：

```
~/
├── projects/           # 所有项目
│   ├── python/
│   ├── go/
│   ├── java/
│   └── nodejs/
└── .ssh/               # SSH 密钥
    ├── id_ed25519      # 私钥（不要分享）
    └── id_ed25519.pub  # 公钥
```

创建目录：

```bash
mkdir -p ~/projects/{python,go,java,nodejs}
```
