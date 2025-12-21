# 🚀 端口转发管理工具 (Port Forward Manager)

一款 Linux 端口转发管理工具，支持 7 种转发方案，自动安装依赖和优化网络性能。

## ✨ 特性

- 🎯 **7 种转发方案** - iptables / HAProxy / socat / gost / realm / rinetd / nginx stream
- 🔧 **自动部署** - 自动安装依赖、配置服务、优化内核
- 📊 **实时状态** - 查看所有活跃转发规则和延迟检测
- 🚄 **性能优化** - 自动启用 BBR、TCP Fast Open、大缓冲区等优化
- 🔄 **多规则共存** - 不同转发方案可同时运行，互不影响
- 💾 **配置备份** - 自动备份配置，支持快速恢复
- 🎨 **友好界面** - 彩色交互式菜单，操作简单直观

## 📋 支持系统

- Debian 10/11/12 ✅
- Ubuntu 20.04/22.04/24.04 ✅
- CentOS 7 ⚠️ (基本支持，使用 yum)

> 脚本主要在 Debian/Ubuntu 上测试，CentOS/RHEL 系列基本可用但部分依赖可能需要手动安装。

## 🛠️ 安装

### 一键安装

```bash
bash <(curl -sL https://raw.githubusercontent.com/Chil30/port-forward/main/port_forward.sh)
```

### 手动安装

```bash
# 下载脚本
wget https://raw.githubusercontent.com/Chil30/port-forward/main/port_forward.sh

# 添加执行权限
chmod +x port_forward.sh

# 运行脚本
./port_forward.sh
```

首次运行会自动安装到 `/usr/local/bin/pf`，之后可直接使用 `pf` 命令。

## 📖 使用方法

### 启动工具

```bash
pf
```

### 主菜单

```
============================================================================
                      端口转发管理工具 v1.0.0
============================================================================
  状态: 运行中    转发规则: 3 条
  作者: Chil30    命令: pf
  项目: https://github.com/Chil30/port-forward
============================================================================

  1) 配置新的端口转发
  2) 查看当前转发状态
  3) 查看运行日志
  4) 停止转发服务
  5) 查看备份文件
  6) 卸载转发服务
  0) 退出
```

### 转发方案对比

| 方案 | 延迟 | 适用场景 | 特点 |
|------|------|----------|------|
| iptables DNAT | ⭐ 最低 | 游戏/RDP/VNC | 内核级转发，性能最佳 |
| realm | ⭐⭐ 较低 | 高并发场景 | Rust 编写，高性能 |
| HAProxy | ⭐⭐ 较低 | Web/负载均衡 | 功能丰富，支持健康检查 |
| nginx stream | ⭐⭐ 较低 | Web/SSL | 与现有 nginx 集成 |
| socat | ⭐⭐ 较低 | 通用转发 | 简单可靠 |
| rinetd | ⭐⭐ 较低 | 多端口转发 | 配置简单 |
| gost | ⭐⭐⭐ 中等 | 加密代理 | 支持多协议、加密 |

**性能排序**: iptables > realm > HAProxy/nginx > socat/rinetd > gost

**功能排序**: gost > nginx/HAProxy > realm > socat/rinetd > iptables

### 配置示例

1. 运行 `pf` 进入主菜单
2. 选择 `1) 配置新的端口转发`
3. 输入目标服务器 IP 和端口
4. 输入本地监听端口
5. 选择转发方案
6. 确认配置开始部署

```
目标服务器IP/域名: 192.168.1.100
目标端口 [3389]: 22
本地监听端口 [22]: 33389
请选择方案 [1]: 5

配置确认：
目标服务器: 192.168.1.100:22
本地监听: 0.0.0.0:33389
转发方案: realm

确认配置并开始部署? [Y/n]: y
```

## 🔧 性能优化

脚本会自动应用以下内核优化：

- ✅ BBR 拥塞控制算法
- ✅ TCP Fast Open (减少握手延迟)
- ✅ 256MB 网络缓冲区
- ✅ 早期重传机制
- ✅ 瘦流优化
- ✅ 禁用延迟 ACK
- ✅ 连接跟踪优化

## 📁 文件位置

| 文件 | 路径 |
|------|------|
| 脚本命令 | `/usr/local/bin/pf` |
| 配置备份 | `/root/.port_forward_backups/` |
| iptables 备份 | `/root/.port_forward_iptables_running.txt` |
| realm 配置 | `/etc/realm/config.toml` |
| gost 配置 | `/etc/gost/config.json` |
| HAProxy 配置 | `/etc/haproxy/haproxy.cfg` |
| rinetd 配置 | `/etc/rinetd.conf` |
| nginx stream | `/etc/nginx/stream.d/port-forward-*.conf` |

## ❓ 常见问题

### Q: iptables 规则重启后丢失？

A: 脚本会自动备份 iptables 规则到 `/root/.port_forward_iptables_running.txt`，可通过菜单 `4) 启动转发服务` 恢复。建议安装 `iptables-persistent` 实现持久化：

```bash
apt install iptables-persistent
netfilter-persistent save
```

### Q: nginx stream 报错 "unknown directive stream"？

A: Debian/Ubuntu 默认的 nginx 包不包含 stream 模块，脚本会自动安装 `nginx-full` 并加载模块。如果已有 nginx 运行，脚本会保留现有配置，只添加 stream 转发。

### Q: 如何同时使用多种转发方案？

A: 脚本支持多种方案共存。部署新方案时只会清理同类型的旧配置，不影响其他方案。例如可以同时运行 iptables 转发端口 A 和 realm 转发端口 B。

### Q: rinetd 端口监听失败？

A: 确保端口未被占用，脚本会自动处理 rinetd 服务重启。如仍有问题，尝试：

```bash
killall rinetd
systemctl restart rinetd
```

### Q: 如何查看转发是否生效？

A: 
1. 使用菜单 `2) 查看当前转发状态` 查看所有活跃规则和延迟
2. 使用 `telnet 本机IP 本地端口` 测试连接
3. 使用 `ss -tlnp` 查看端口监听状态

### Q: 如何完全卸载？

A: 使用菜单 `6) 卸载转发服务`，选择 `8) 卸载所有服务`。这会停止所有服务、清理配置文件和 iptables 规则。

### Q: 转发延迟很高怎么办？

A: 
1. 优先使用 iptables DNAT 方案（内核级，延迟最低）
2. 确认 BBR 已启用：`sysctl net.ipv4.tcp_congestion_control`
3. 检查目标服务器网络质量

## 🔗 相关链接

- GitHub: https://github.com/Chil30/port-forward
- Issues: https://github.com/Chil30/port-forward/issues

## 📄 许可证

MIT License

## 🙏 致谢

感谢以下开源项目：
- [realm](https://github.com/zhboner/realm)
- [gost](https://github.com/ginuerzh/gost)
- [HAProxy](https://www.haproxy.org/)
- [nginx](https://nginx.org/)
