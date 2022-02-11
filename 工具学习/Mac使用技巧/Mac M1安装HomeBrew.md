# Mac M1安装HomeBrew

## 安装HomeBrew

```bash
$ /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```



## 附加操作

## 

对于Apple Silicon的机型

### 1. 添加 Homebrew shell 配置

```bash
echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv)"
```

## 验证是否安装完成

```bash
brew doctor
```



具体文章链接：https://mac.install.guide/homebrew/3.html