# iStoreOS RK3399 Kmods 软件源完整方案

## 📋 方案概述

本方案实现了在编译 iStoreOS 固件的同时，自动生成独立的 **kmods 软件源归档**，使您能够部署自己的内核模块软件仓库。

### ✨ 主要特性

- 🚀 **零额外编译时间**：利用固件编译过程中已生成的 kmod 包
- 📦 **标准 OpenWrt 格式**：完全兼容 opkg 包管理器
- 🔄 **版本一致性保证**：kmod 包与固件内核版本完全匹配
- 🌐 **即插即用**：解压后直接可用作 HTTP 软件源
- 📊 **自动索引生成**：包含 Packages.gz 索引文件
- ✅ **SHA256 校验**：确保包完整性和安全性

## 🔧 技术实现

### 工作流程

1. **固件编译阶段**：正常编译 iStoreOS 固件和所有内核模块
2. **Kmods 收集阶段**：
   - 从 `bin/targets/rockchip/armv8/packages/` 收集所有 kmod-*.ipk
   - 按架构组织目录结构
3. **索引生成阶段**：
   - 生成 Packages 索引文件
   - 创建 Packages.gz 压缩索引
   - 计算 SHA256 校验和
4. **打包发布阶段**：
   - 创建 kmods-repository.tar.gz 归档
   - 与固件一起上传到 GitHub Release

### 目录结构

```
kmods-repository/
├── README.md                          # 使用说明
└── packages/
    └── aarch64_cortex-a53/           # 架构目录
        ├── kmod-*.ipk                # 内核模块包
        ├── Packages                  # 软件源索引
        └── Packages.gz               # 压缩索引
```

## 📥 使用指南

### 方式一：使用 Nginx 部署（推荐）

#### 使用 Docker

```bash
# 1. 下载 kmods 归档
wget https://github.com/YOUR_USERNAME/iStoreOS-RK3399/releases/download/TAG/kmods-repository-fine3399-aarch64_cortex-a53.tar.gz

# 2. 解压
tar -xzf kmods-repository-fine3399-aarch64_cortex-a53.tar.gz

# 3. 使用 Docker 运行 Nginx
docker run -d \
  --name kmods-server \
  -p 8080:80 \
  -v $(pwd)/kmods-repository:/usr/share/nginx/html:ro \
  nginx:alpine

# 4. 测试访问
curl http://localhost:8080/packages/aarch64_cortex-a53/Packages.gz
```

#### 使用系统 Nginx

```bash
# 1. 安装 nginx
sudo apt install nginx  # Debian/Ubuntu
# 或
sudo yum install nginx  # CentOS/RHEL

# 2. 解压到 web 目录
sudo tar -xzf kmods-repository-fine3399-aarch64_cortex-a53.tar.gz -C /var/www/html/

# 3. 设置权限
sudo chmod -R 755 /var/www/html/kmods-repository
sudo chown -R www-data:www-data /var/www/html/kmods-repository

# 4. 重启 nginx
sudo systemctl restart nginx
```

### 方式二：使用 Python HTTP 服务器（测试用）

```bash
# 1. 解压
tar -xzf kmods-repository-fine3399-aarch64_cortex-a53.tar.gz

# 2. 启动 HTTP 服务器
cd kmods-repository
python3 -m http.server 8080

# 访问: http://YOUR_IP:8080
```

### 方式三：使用 Apache

```bash
# 1. 安装 apache2
sudo apt install apache2

# 2. 解压到 web 目录
sudo tar -xzf kmods-repository-fine3399-aarch64_cortex-a53.tar.gz -C /var/www/html/

# 3. 设置权限
sudo chmod -R 755 /var/www/html/kmods-repository

# 4. 重启 apache
sudo systemctl restart apache2
```

## 🔌 在路由器上配置软件源

### 1. 添加自定义软件源

SSH 登录到您的 iStoreOS 路由器：

```bash
ssh root@192.168.1.1
```

添加软件源配置：

```bash
# 创建或编辑自定义 feeds 配置文件
cat >> /etc/opkg/customfeeds.conf << EOF
src/gz custom_kmods http://YOUR_SERVER_IP:8080/packages/aarch64_cortex-a53
EOF
```

### 2. 更新软件源

```bash
opkg update
```

### 3. 搜索和安装 kmod

```bash
# 搜索可用的 kmod 包
opkg list | grep kmod-

# 安装特定的 kmod
opkg install kmod-usb-storage
opkg install kmod-fs-ext4
opkg install kmod-usb-net-rtl8152

# 查看已安装的 kmod
opkg list-installed | grep kmod-
```

## 🎯 应用场景

### 场景 1：内网离线环境

在无法访问外网的环境中，部署本地 kmods 软件源：

```bash
# 内网服务器部署
tar -xzf kmods-repository-*.tar.gz -C /srv/openwrt-repo/

# 路由器配置（使用内网 IP）
echo "src/gz local_kmods http://192.168.1.100/kmods-repository/packages/aarch64_cortex-a53" >> /etc/opkg/customfeeds.conf
```

### 场景 2：多设备统一管理

为多台 fine3399 设备提供统一的软件源：

```bash
# 中心服务器部署软件源
# 所有设备指向同一服务器
for device in router1 router2 router3; do
  ssh root@$device "echo 'src/gz central_kmods http://central.server.local/kmods-repository/packages/aarch64_cortex-a53' >> /etc/opkg/customfeeds.conf"
  ssh root@$device "opkg update"
done
```

### 场景 3：自定义内核模块分发

开发并编译自定义内核模块后，通过此软件源分发：

```bash
# 1. 将自定义 .ipk 添加到软件源
cp custom-kmod-*.ipk kmods-repository/packages/aarch64_cortex-a53/

# 2. 重新生成索引
cd kmods-repository/packages/aarch64_cortex-a53/
opkg-make-index . > Packages
gzip -k -f Packages

# 3. 客户端更新并安装
opkg update
opkg install custom-kmod-xxx
```

## 🔒 安全建议

### 1. 使用 HTTPS

生产环境建议配置 HTTPS：

```nginx
server {
    listen 443 ssl http2;
    server_name kmods.yourdomain.com;
    
    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    root /var/www/html/kmods-repository;
    
    location / {
        autoindex on;
    }
}
```

路由器配置：

```bash
echo "src/gz secure_kmods https://kmods.yourdomain.com/packages/aarch64_cortex-a53" >> /etc/opkg/customfeeds.conf
```

### 2. 访问控制

限制访问来源：

```nginx
location /kmods-repository/ {
    allow 192.168.1.0/24;    # 允许本地网络
    deny all;                 # 拒绝其他
}
```

### 3. 校验包完整性

安装前验证 SHA256：

```bash
# 下载包和校验文件
opkg download kmod-usb-storage
sha256sum -c kmod-usb-storage*.sha
```

## 🛠️ 故障排除

### 问题 1：opkg update 失败

```bash
# 检查服务器是否可访问
curl http://YOUR_SERVER/packages/aarch64_cortex-a53/Packages.gz

# 检查软件源配置
cat /etc/opkg/customfeeds.conf

# 查看详细错误
opkg update -V
```

### 问题 2：包安装后无法加载

```bash
# 检查内核版本是否匹配
uname -r
opkg info kmod-xxx | grep Kernel

# 手动加载模块
modprobe module_name

# 查看加载失败原因
dmesg | tail -20
```

### 问题 3：Packages.gz 索引损坏

```bash
# 重新生成索引
cd /path/to/kmods-repository/packages/aarch64_cortex-a53/
opkg-make-index . > Packages
gzip -k -f Packages
```

## 📊 性能优化

### CDN 加速

使用 CDN 加速软件源访问：

```bash
# 上传到支持 CDN 的对象存储
# 例如：阿里云 OSS、腾讯云 COS、AWS S3

# 路由器配置使用 CDN 地址
echo "src/gz cdn_kmods https://cdn.yourdomain.com/kmods-repository/packages/aarch64_cortex-a53" >> /etc/opkg/customfeeds.conf
```

### 本地缓存

设置 opkg 缓存以减少重复下载：

```bash
# 编辑 /etc/opkg.conf
echo "option cache /tmp/opkg-cache" >> /etc/opkg.conf
mkdir -p /tmp/opkg-cache
```

## 🔄 自动化维护

### 定期更新软件源

创建 cron 任务定期更新：

```bash
#!/bin/bash
# update-kmods.sh

# 下载最新的 kmods 归档
cd /var/www/html
rm -rf kmods-repository-old
mv kmods-repository kmods-repository-old

wget https://github.com/YOUR_USER/iStoreOS-RK3399/releases/latest/download/kmods-repository-fine3399-aarch64_cortex-a53.tar.gz
tar -xzf kmods-repository-*.tar.gz

# 重启 nginx
systemctl reload nginx

# 通知客户端
echo "Kmods repository updated on $(date)" | mail -s "Kmods Update" admin@example.com
```

添加到 crontab：

```bash
# 每月1号凌晨2点更新
0 2 1 * * /path/to/update-kmods.sh
```

## 📝 版本兼容性

| iStoreOS 版本 | 内核版本 | 架构 | 兼容性 |
|--------------|---------|------|-------|
| 24.10 | 6.6.x | aarch64_cortex-a53 | ✅ 完全兼容 |
| 22.03 | 5.15.x | aarch64_cortex-a53 | ⚠️ 需要匹配版本 |

**重要提示**：确保路由器的内核版本与 kmods 软件源的编译版本一致，否则可能导致内核模块加载失败。

## 🤝 贡献和反馈

如有问题或建议，欢迎提交 Issue 或 Pull Request。

## 📄 许可证

本项目遵循 MIT 许可证。

## 🙏 致谢

- [OpenWrt](https://openwrt.org/) - 开源路由器固件
- [iStoreOS](https://github.com/istoreos/istoreos) - 基于 OpenWrt 的路由系统
- 所有为开源社区做出贡献的开发者

---

**编译日期**: 2026-02
**适用设备**: fine3399 (RK3399)
**维护者**: flyweb291
