---
title: 使用 OpenClaw 搭建全天候 AI 助手与透明代理
date: 2026-04-06 23:00:00
tags: [OpenClaw, AI, NAS, Linux, 教程, 代理]
categories: 技术分享
---

# 使用 OpenClaw 搭建全天候 AI 助手与透明代理

> 通过 OpenClaw 在 NAS 上部署 AI 助手，配合 Clash 实现全家透明代理

## 背景

之前一直在个人电脑上运行 AI 助手，但电脑不能 24 小时开机。后来把 AI 助手迁移到了 NAS（飞牛私有盘），实现了真正的全天候服务。同时配合 Clash 做透明代理，全家设备无需设置即可科学上网。

本文记录整个搭建过程，供有类似需求的同学参考。

## 硬件环境

- **NAS**: 飞牛私有盘 FN-NAS-WYSE
- **CPU**: Intel Pentium Silver J5005 (4核)
- **内存**: 15GB DDR4
- **系统盘**: 16GB eMMC (系统占用 ~54%)
- **数据盘**: 1TB + 500GB SSD (LVM 存储)

## 软件架构

```
用户 (QQ/微信/Telegram)
       ↓
OpenClaw Gateway (NAS:18789)
       ↓
AI 模型 (MiniMax-M2)
       ↓
Clash (NAS:7890) ← 透明代理
       ↓
华硕路由器 (192.168.1.1)
       ↓
全家设备 (手机/电脑/TV)
```

## 1. 安装 OpenClaw

### 方式一：官方脚本

```bash
curl -fsSL https://get.openclaw.ai | sh
```

### 方式二：Docker 部署

```bash
docker run -d \
  --name openclaw \
  --restart unless-stopped \
  -p 18789:18789 \
  -v ~/openclaw:/app/data \
  openclaw/openclaw:latest
```

### 配置 QQ 频道机器人

1. 打开 https://q.qq.com 创建机器人
2. 获取 AppID 和 Token
3. 在 OpenClaw 控制台配置渠道

## 2. 安装 Clash (mihomo)

使用 Docker 安装 clash-verge：

```bash
docker run -d \
  --name clash-verge \
  --restart unless-stopped \
  --network host \
  -v /home/ssh/.openclaw/clash-config/mihomo-config.yaml:/config/config.yaml \
  -v /home/ssh/.openclaw/clash-config/profiles:/profiles \
  -v /home/ssh/.openclaw/clash-config/Country.mmdb:/config/Country.mmdb \
  -p 7890:7890 \
  -p 7891:7891 \
  -p 9097:9097 \
  clash-verge:1
```

## 3. 配置 mihomo

关键配置项说明：

```yaml
# 端口设置
mixed-port: 7890    # HTTP/SOCKS5 混合代理
socks-port: 7891    # 纯 SOCKS5
redir-port: 7892    # 重定向代理
allow-lan: true     # 允许局域网访问
mode: rule          # 规则模式
log-level: info     # 日志级别

# DNS 设置（避免 DNS 污染）
dns:
  enable: true
  listen: 0.0.0.0:53
  enhanced-mode: fake-ip
  fake-ip-range: 198.18.0.1/16
```

## 4. 节点选择策略

根据我长期使用的经验，节点优先级：

| 优先级 | 节点 | 原因 |
|--------|------|------|
| 1 | 台湾 HiNet | 延迟低，IP 干净 |
| 2 | 日本 NTT | 速度快，线路稳 |
| 3 | 新加坡 | 备选，速度尚可 |
| ❌ | 香港 | 大多数 AI 服务屏蔽 |

**故障切换配置：**

```yaml
proxy-groups:
  - name: ♻️ 故障切换
    type: fallback
    proxies:
      - 🇹🇼 台湾 01 HiNet
      - 🇯🇵 日本 19 NTT
      - 🇸🇬 新加坡 02
    url: https://cp.cloudflare.com/generate_204
    interval: 300
```

## 5. 透明代理设置

### 方法一：路由器 DHCP 指 NAS

在华硕路由器后台：
1. LAN → DHCP 服务器
2. DNS 服务器填写 NAS IP (192.168.1.4)
3. 保存重启

### 方法二：Clash TUN 模式

在 mihomo 配置中开启：

```yaml
tun:
  enable: true
  stack: system
  auto-route: true
  auto-detect-interface: true
```

## 6. NAS 系统优化

### 清理系统盘

我的 NAS 系统盘只有 15GB，容易满。清理方法：

```bash
# 清理 npm 缓存
npm cache clean --force

# 清理 Docker 残留镜像
docker image prune -a -f

# 清理旧内核
apt-get autoremove --purge
```

### 将大文件移到数据盘

```bash
# 将 npm 全局目录移到 /vol1
cp -a ~/.npm-global /vol1/.npm-global
rm -rf ~/.npm-global
ln -s /vol1/.npm-global ~/.npm-global
```

## 7. DDNS 配置

使用阿里云 DDNS，保证外网随时访问：

1. 在阿里云购买域名
2. 创建 AccessKey
3. 使用 ddns-go 工具自动更新

```bash
# DDNS-GO 配置示例
url: https://api.example.com
domain: yourdomain.com
token: your-access-token
```

## 8. 安全建议

1. **SSH 密钥登录**：关闭密码登录，使用密钥
2. **防火墙**：只开放必要端口
3. **代理认证**：Clash 开启 secret 密码
4. **定期备份**：配置文件定期备份到 /vol1

## 总结

通过以上配置，实现了：

- ✅ 24 小时 AI 助手服务
- ✅ 全家设备透明代理
- ✅ 节点自动故障切换
- ✅ 外网安全访问

整个方案成本低（NAS 已有）、功耗小、稳定可靠。有问题欢迎留言交流。

## 参考链接

- [OpenClaw 官网](https://openclaw.ai)
- [Clash Verge Wiki](https://github.com/clash-verge/clash-verge)
- [mihomo 文档](https://wiki.metacubex.one/)
