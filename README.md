# ImmortalWrt / OpenWrt 自建软件包仓库

本仓库用于托管适用于 **aarch64_cortex-a53** 架构的 ImmortalWrt/OpenWrt 软件包（`.apk` 格式），包含 `base`、`luci`、`packages` 等分类，支持 `25.12.0` 稳定版及 `snapshots` 快照版。

## 目录结构

```
├── immortalwrt/          # ImmortalWrt 发行版
│   ├── 25.12.0/          # 稳定版目录
│   │   ├── packages/     # 软件包分类目录
│   │   │   ├── aarch64_cortex-a53/
│   │   │   │   ├── base/       # 系统核心包
│   │   │   │   ├── luci/       # Luci 界面相关
│   │   │   │   └── packages/   # 扩展应用
│   │   └── targets/      # 固件镜像及内核模块
│   ├── snapshots/        # 每日快照版
│   └── public-key.pem    # 仓库公钥（用于签名验证）
├── scripts/              # 辅助脚本（索引生成等）
└── README.md
```

## 使用方法

1. 导入仓库公钥

   各子仓库公钥位置：
   
   - [/immortalwrt](/immortalwrt "/immortalwrt")
   - [/openwrt](/openwrt "/openwrt")

2. 添加软件源

   各软件源位置：
   
   - [/immortalwrt/snapshots](/immortalwrt/snapshots "/immortalwrt/snapshots")
   - [/immortalwrt/25.12.0](/immortalwrt/25.12.0 "/immortalwrt/25.12.0")
   - [/openwrt/snapshots](/openwrt/snapshots "/openwrt/snapshots")
   - [/openwrt/25.12.2](/openwrt/25.12.2 "/openwrt/25.12.2")

3. 更新软件源

   ```bash
   apk update
   ```

## 签名验证

本仓库的 `packages.adb` 索引文件已使用配套私钥签名，对应的公钥即为 `public-key.pem`。

- 若更新时出现 `UNTRUSTED signature` 警告，说明公钥未正确安装或索引文件未被签名。
- 请确保已将公钥放置到正确的系统目录（`/etc/apk/keys/`）。

## 注意事项

- 本仓库仅适用于 **aarch64_cortex-a53** 架构（如 Banana Pi BPI-R4 等设备）。
- 请勿在生产环境中关闭签名验证（`option check_signature 0`）。
- 如需其他架构或版本，请自行修改 URL 中的路径。

## 更多信息

- [OpenWrt 软件包签名指南](https://openwrt.org/docs/guide-user/additional-software/opkg_signature)
- [apk 包管理器使用说明](https://wiki.alpinelinux.org/wiki/Alpine_Package_Keeper)
