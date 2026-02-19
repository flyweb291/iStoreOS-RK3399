![iStoreOS Logo](https://github.com/Lemon1151/iStoreOS-RK3399/raw/RK3399-dev/istoreos.png)
## iStore OS 固件 

[![iStore使用文档](https://img.shields.io/badge/使用文档-iStore%20OS-brightgreen?style=flat-square)](https://doc.linkease.com/zh/guide/istoreos) 

## 仓库介绍
**iStoreOS** 是入门级的路由系统，也是入门级的 NAS 系统， 基于原版 OpenWRT，在 ARS2 上经过长期迭代，最终开放适配到多个硬件平台。 

更多信息请参阅https://github.com/istoreos

> [!TIP]
> 此仓库为 **RK3399设备构建iStoreOS，后续更新添加设备中；非官方构建，不保证完全无BUG；如遇无法启动，请接ttl查看输出日志。需要定制的自行fork本仓库后，修改配置.config** 。  
> RK35XX的iStoreOS仓库地址：[xiaomeng9597/iStoreOS-RK35XX](https://github.com/xiaomeng9597/iStoreOS-RK35XX)

> **如果某些设备WiFi不可用，请去[仓库](https://github.com/armbian/firmware)找对应的无线网卡驱动，复制到对应目录替换**。

## RK3399-dev

| ----           | 支持设备                                                                                                                               |
| -------------- | ---------------------------------------------------------------------------------------------------------------------------------------|
| RK3399-dev     | am40,dg3399,dlfr100,fine3399,fmx1-pro,fnet-3399,h3399pc,king3399,mpc1903,sv901-eaio,sv-33a6x,tn3399,tpm312,tvi3315a,xiaobao-nas,zysj   |


## 默认配置

- 用户名: `root`
- 密  码: `password`
- 如果设备只有一个网口，则此网口就是 `LAN` ；如果设备有两个网口，则一个是 `WAN`，一个是`LAN`。
- 关于管理 IP：`.config` 中已将 `CONFIG_TARGET_PREINIT_IP` 设为 `192.168.101.210`，并启用了 `CONFIG_TARGET_DEFAULT_LAN_IP_FROM_PREINIT=y`，正式启动后的 LAN 管理地址也将统一为 `192.168.101.210`。
- 关于 OpenClash：已在 `feeds.conf.default` 加入 OpenClash 源；若在 `.config` 里启用 `CONFIG_PACKAGE_luci-app-openclash=y`，请确保编译前执行 `./scripts/feeds update -a && ./scripts/feeds install luci-app-openclash` 以拉取并安装依赖，否则固件里不会生成该包。

## ✨ 新特性：Kmods 软件源

从现在开始，每次编译固件时会**自动生成独立的 kmods 软件源归档**，让您可以部署自己的内核模块软件仓库！

### 主要特点
- 🚀 **零额外编译时间** - 利用固件编译过程，无需额外步骤
- 📦 **标准 OpenWrt 格式** - 完全兼容 opkg 包管理器
- 🔄 **版本一致性** - 内核模块与固件内核完全匹配
- 🌐 **即插即用** - 解压后直接可作为 HTTP 软件源使用

### 快速开始
1. 从 [Releases](../../releases) 下载 `kmods-repository-{设备名}-aarch64_cortex-a53.tar.gz`
2. 使用 Docker 一键部署：
   ```bash
   tar -xzf kmods-repository-*.tar.gz
   docker run -d -p 8080:80 -v $(pwd)/kmods-repository:/usr/share/nginx/html:ro nginx:alpine
   ```
3. 在路由器上配置：
   ```bash
   echo "src/gz custom_kmods http://YOUR_SERVER:8080/packages/aarch64_cortex-a53" >> /etc/opkg/customfeeds.conf
   opkg update
   opkg install kmod-usb-storage  # 示例：安装 USB 存储模块
   ```

### 📚 详细文档
- 📖 [完整使用指南](KMODS_REPOSITORY_GUIDE.md) - 部署方法、应用场景、故障排查
- 🚀 [实施说明](IMPLEMENTATION_SUMMARY_CN.md) - 技术方案和功能说明
- 📋 [快速参考](QUICK_REFERENCE.md) - 常用命令速查



## 鸣谢

- [istoreos](https://github.com/istoreos/istoreos)
- [P3TERX/Actions-OpenWrt](https://github.com/P3TERX/Actions-OpenWrt)
- [xiaomeng9597](https://github.com/xiaomeng9597)
- [cm9vdA](https://github.com/cm9vdA/build-linux)
- [GitHub Actions](https://github.com/features/actions)
- [OpenWrt](https://github.com/openwrt/openwrt)
- [Lean&#39;s OpenWrt](https://github.com/coolsnowwolf/lede)
- [csexton/debugger-action](https://github.com/csexton/debugger-action)
- [Cowtransfer](https://cowtransfer.com)
- [Mikubill/transfer](https://github.com/Mikubill/transfer)
- [softprops/action-gh-release](https://github.com/softprops/action-gh-release)
- [ActionsRML/delete-workflow-runs](https://github.com/ActionsRML/delete-workflow-runs)
- [dev-drprasad/delete-older-releases](https://github.com/dev-drprasad/delete-older-releases)


##  免责声明
- 本固件仅供学习研究，严禁用于任何商业用途
- 使用本固件产生的所有后果均由使用者自行承担
- 固件可能存在bug，开发者不提供任何形式的技术支持
- 请严格遵守国家网络安全法律法规，合法使用
