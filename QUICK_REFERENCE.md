# Fine3399 Kmods 软件源 - 快速参考卡

## 🚀 一键部署命令

### Docker 部署（推荐）
```bash
# 下载最新版本
wget https://github.com/flyweb291/iStoreOS-RK3399/releases/latest/download/kmods-repository-fine3399-aarch64_cortex-a53.tar.gz

# 解压
tar -xzf kmods-repository-fine3399-aarch64_cortex-a53.tar.gz

# 启动服务（一键）
docker run -d --name kmods -p 8080:80 \
  -v $(pwd)/kmods-repository:/usr/share/nginx/html:ro \
  --restart unless-stopped \
  nginx:alpine

# 测试
curl http://localhost:8080/packages/aarch64_cortex-a53/Packages.gz
```

### Nginx 部署
```bash
sudo apt install nginx -y
sudo tar -xzf kmods-repository-*.tar.gz -C /var/www/html/
sudo chmod -R 755 /var/www/html/kmods-repository
sudo systemctl restart nginx
```

## 🔌 路由器配置（Fine3399）

### 一键配置脚本
```bash
#!/bin/bash
# 在 fine3399 路由器上运行

# 配置变量（修改为您的服务器地址）
SERVER_IP="192.168.1.100"
SERVER_PORT="8080"

# 添加软件源
cat >> /etc/opkg/customfeeds.conf << EOF
src/gz custom_kmods http://${SERVER_IP}:${SERVER_PORT}/packages/aarch64_cortex-a53
EOF

# 更新
opkg update

echo "✅ Kmods 软件源配置完成！"
echo "📦 可以使用: opkg install kmod-XXX"
```

### 常用 kmod 包安装
```bash
# USB 存储
opkg install kmod-usb-storage kmod-usb-storage-uas

# 文件系统
opkg install kmod-fs-ext4 kmod-fs-ntfs kmod-fs-exfat

# 网络
opkg install kmod-usb-net-rtl8152 kmod-macvlan

# 虚拟化/容器
opkg install kmod-veth kmod-br-netfilter

# 硬盘
opkg install kmod-ata-ahci kmod-scsi-core
```

## 📊 验证检查清单

```bash
# 1. 检查服务器运行状态
curl -I http://YOUR_SERVER:8080/packages/aarch64_cortex-a53/Packages.gz

# 2. 在路由器上检查软件源
opkg update
opkg list | grep kmod- | head -10

# 3. 测试安装
opkg install kmod-usb-storage

# 4. 查看已安装模块
opkg list-installed | grep kmod-
lsmod | grep usb_storage
```

## 🔧 故障排查

| 问题 | 检查命令 | 解决方法 |
|------|----------|----------|
| 无法访问服务器 | `ping SERVER_IP` | 检查防火墙/网络 |
| opkg update 失败 | `cat /etc/opkg/customfeeds.conf` | 检查 URL 配置 |
| 包安装失败 | `opkg info kmod-XXX \| grep Kernel` | 检查内核版本匹配 |
| 模块加载失败 | `dmesg \| tail -20` | 查看内核错误日志 |

## 📈 服务器监控

### Docker 容器管理
```bash
# 查看状态
docker ps | grep kmods

# 查看日志
docker logs -f kmods

# 重启
docker restart kmods

# 停止
docker stop kmods

# 删除
docker rm -f kmods
```

### Nginx 管理
```bash
# 查看状态
sudo systemctl status nginx

# 查看访问日志
sudo tail -f /var/log/nginx/access.log

# 重启
sudo systemctl restart nginx
```

## 🔄 更新软件源

### 自动更新脚本
```bash
#!/bin/bash
# update-kmods.sh - 定期更新 kmods 软件源

REPO="flyweb291/iStoreOS-RK3399"
LATEST_URL="https://github.com/${REPO}/releases/latest/download/kmods-repository-fine3399-aarch64_cortex-a53.tar.gz"
DEPLOY_DIR="/var/www/html"

cd /tmp
wget -q ${LATEST_URL} -O kmods-new.tar.gz

if [ $? -eq 0 ]; then
    echo "✅ 下载成功，开始更新..."
    sudo rm -rf ${DEPLOY_DIR}/kmods-repository-old
    sudo mv ${DEPLOY_DIR}/kmods-repository ${DEPLOY_DIR}/kmods-repository-old
    sudo tar -xzf kmods-new.tar.gz -C ${DEPLOY_DIR}/
    sudo systemctl reload nginx
    echo "✅ 更新完成！"
    rm -f kmods-new.tar.gz
else
    echo "❌ 下载失败！"
fi
```

### 添加到 Crontab
```bash
# 每月1号凌晨2点自动更新
0 2 1 * * /path/to/update-kmods.sh >> /var/log/kmods-update.log 2>&1
```

## 🌐 内网穿透（可选）

如果需要从外网访问：

### 使用 frp
```ini
# frpc.ini
[common]
server_addr = YOUR_FRP_SERVER
server_port = 7000

[kmods_http]
type = tcp
local_ip = 127.0.0.1
local_port = 8080
remote_port = 6080
```

### 使用 Cloudflare Tunnel
```bash
# 安装 cloudflared
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64
chmod +x cloudflared-linux-amd64
sudo mv cloudflared-linux-amd64 /usr/local/bin/cloudflared

# 创建隧道
cloudflared tunnel --url http://localhost:8080
```

## 📱 移动端测试

使用手机测试软件源：
```bash
# 在手机浏览器访问
http://YOUR_SERVER_IP:8080/packages/aarch64_cortex-a53/

# 应该能看到所有 .ipk 文件列表
# Packages.gz 文件应该可以下载
```

## 🎯 生产环境检查清单

- [ ] 服务器已配置自动启动（systemd/docker restart policy）
- [ ] 防火墙已开放端口（80/8080/443）
- [ ] （可选）配置 HTTPS 证书
- [ ] （可选）配置访问认证
- [ ] 设置自动更新脚本
- [ ] 配置日志轮转
- [ ] 监控服务器状态
- [ ] 测试从路由器访问和安装

## 💡 专业提示

1. **版本管理**: 建议保留最近3个版本的软件源，方便回滚
2. **监控**: 使用 Prometheus + Grafana 监控访问量和状态
3. **CDN**: 生产环境使用 CDN 加速（阿里云 OSS/腾讯云 COS）
4. **备份**: 定期备份 kmods 归档文件
5. **测试**: 在测试设备上先验证新版本再推广

## 📞 支持

- 📖 完整文档: `KMODS_REPOSITORY_GUIDE.md`
- 🔍 实施说明: `IMPLEMENTATION_SUMMARY_CN.md`
- 💬 问题反馈: GitHub Issues

---

**快速参考版本**: v1.0
**更新日期**: 2026-02-19
**适用设备**: fine3399 (RK3399)
