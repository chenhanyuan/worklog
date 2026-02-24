# VS Code 使用 GitHub Copilot 说明（WSL 环境）

本文档介绍在 Windows Subsystem for Linux (WSL) 下的 Ubuntu 系统中配置 GitHub Copilot 的步骤。由于 Copilot 需要访问 GitHub API，WSL 中的网络代理配置是关键。

## 前提条件

- Windows 主机上已安装并运行 VPN，确保能够访问 GitHub
- VPN 客户端允许局域网连接（通常称为「允许局域网」或「Allow LAN」）
- WSL 中已安装 Ubuntu 发行版，并已安装 VS Code 及 GitHub Copilot 扩展

## 配置步骤

### 1. 获取 WSL 网关 IP

在 WSL Ubuntu 终端中执行以下命令，获取 Windows 主机的网关 IP 地址：

```bash
ip route show default
```

输出类似：

```
default via 172.17.64.1 dev eth0
```

其中的 `172.17.64.1` 即为网关地址，也就是 Windows 主机的 IP（在 WSL 网络中的地址）。

### 2. 设置永久代理环境变量

将以下三行添加到 `~/.bashrc`（或 `~/.zshrc`，取决于你使用的 Shell）中，使代理配置永久生效。请将 `172.17.64.1` 替换为第一步获取的网关 IP：

```bash
export http_proxy="http://172.17.64.1:7890"
export https_proxy="http://172.17.64.1:7890"
export no_proxy="localhost,127.0.0.1,::1,.local"
```

> **注意**：端口 `7890` 为常见代理端口，请根据你的 VPN 或代理软件实际监听的端口进行修改（例如 Clash 默认端口为 7890，v2ray 可能为 10809 等）。

保存文件后，执行以下命令使配置立即生效：

```bash
source ~/.bashrc
```

### 3. 测试代理连通性

执行以下命令，验证能否通过代理访问 GitHub API：

```bash
curl -I https://api.github.com
```

如果返回 `HTTP/2 200` 或类似成功状态码，说明代理配置正确，WSL 可以正常访问 GitHub。

### 4. 登录 GitHub Copilot

在 VS Code 中，按以下步骤操作：

1. 打开命令面板（`Ctrl+Shift+P`）
2. 输入并选择 `GitHub Copilot: Sign in to GitHub`
3. 按照提示完成设备授权流程
4. 登录成功后，即可在编辑器中享受 Copilot 代码建议
