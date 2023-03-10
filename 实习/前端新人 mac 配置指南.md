# 前端新人 mac 配置指南
> 本文适合初入职场且第一次使用 mac 的前端新人

## 安装 brew

```
/bin/bash -c "$(curl -fsSL`` https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh``)"
```

## 安装 git

```
brew install git
```

检查下是否安装成功`git --version`，成功一般有版本号

生成自己的密钥，并添加到远程仓库中

然后就可以 clone 仓库或远程连接仓库获取代码进行开发了

## 申请代码开发权限

## 下载 Chrome 浏览器

```Plain
brew install --cask google-chrome
```

下载成功后，在启动台开启即可。

然后安装掘金起始页插件

安装**SwitchyOmega插件**

## 下载 VSCode 及其插件

![vsCode.png](https://www.z4a.net/images/2023/03/06/vsCode.png)

## 下载node

## 一些小坑

### 安装依赖时报错，可能是因为 node 版本过高

可询问 mentor 项目使用的 node 版本

安装 nvm 来安装指定的 node 版本

```
brew install nvm
```

编辑 ~/.bash_profile

```Bash
#输入安装好 nvm 后出现的三行代码，每个电脑可能不一样吧，请自己复制粘贴
export NVM_DIR="$HOME/.nvm"
[ -s "/usr/local/opt/nvm/nvm.sh" ] && \. "/usr/local/opt/nvm/nvm.sh"  # This loads nvm
[ -s "/usr/local/opt/nvm/etc/bash_completion.d/nvm" ] && \. "/usr/local/opt/nvm/etc/bash_completion.d/nvm"  # This loads nvm bash_completion
#保存退出
```

执行命令行命令

```
source ~/.bash_profile
```

nvm --version 查看是否安装成功

`nvm install 版本号`安装指定版本 node

`nvm list`查看安装的 node 版本

`nvm use 版本号`使用指定版本 node

### nvm 在 VSCode 找不到

安装了 nvm 但是在 VSCode 上使用 nvm 出现 `zsh: command not found:nvm`

解决办法：

编辑 ~/.zshrc 

```
vim ~/.zshrc
```

输入 

```Bash
source ~/.bash_profile
#保存退出
```

执行命令行命令 `source ~/.zshrc`

### 切走页面又切回来，输入法状态发生改变

问题：编写飞书文档时使用中文状态，切到其他页面查资料，切回来输入法状态变成英文。

在 系统偏好设置->键盘->输入法 打勾自动切换到文稿的输入法

[![mac.png](https://www.z4a.net/images/2023/03/06/mac.png)](https://www.z4a.net/image/VTGmn9)

### 两块屏幕间移动程序坞

鼠标放在边角中几秒钟可解决

### 用VSCode打开文件和文件夹

https://cloud.tencent.com/developer/article/1750457