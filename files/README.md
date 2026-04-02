# 编译时使用到的文件

本仓库提供了一套用于 OpenWrt 设备的配置文件与脚本，主要实现两个功能：

- **智能风扇控制**：根据系统温度自动调节风扇转速。
- **动态 APK 源替换**：在网络接口（如 WAN）状态变化时，自动切换 APK 软件源并恢复已安装的软件包，便于在不同网络环境（如内网镜像源、外网官方源）间切换。

## 目录结构

```
.
├── etc
│   ├── apk
│   │   ├── index.html                    # Web 界面占位文件
│   │   └── repositories.d
│   │       ├── customfeeds.list          # 自定义 APK 源列表
│   │       ├── index.html
│   │       └── README.md                 # APK 源相关说明
│   ├── hotplug.d
│   │   ├── iface
│   │   │   ├── 88-replace-apk-source     # 网络接口 up 时替换 APK 源
│   │   │   ├── 99-restore-package        # 网络接口 down 时恢复原软件包
│   │   │   └── index.html
│   │   └── index.html
│   ├── index.html
│   ├── init.d
│   │   ├── fan-control                   # 风扇控制开机脚本
│   │   └── index.html
│   └── uci-defaults
│       └── index.html                    # UCI 默认配置占位
├── index.html
├── README.md                             # 本文件
└── usr
    ├── bin
    │   ├── fan-control.sh                # 风扇控制主脚本
    │   ├── index.html
    │   ├── replace-apk-source.sh         # 替换 APK 源脚本
    │   └── restore-packages.sh           # 恢复已安装软件包脚本
    └── index.html
```

> `index.html` 文件仅用于防止 Web 服务器列出目录内容，无实际功能。

## 组件功能说明

### 1. 风扇控制

- **服务脚本**：`/etc/init.d/fan-control`  
- **执行脚本**：`/usr/bin/fan-control.sh`  
- **功能**：后台守护进程，定期读取 CPU/主板温度，根据预设的温度阈值调整风扇 PWM 占空比。  
- **启动方式**：  
  ```bash
  /etc/init.d/fan-control enable
  /etc/init.d/fan-control start
  ```

### 2. 动态 APK 源替换

适用于使用 `apk` 包管理器（如 Alpine Linux 或某些定制 OpenWrt 固件）的系统。

- **自定义源文件**：`/etc/apk/repositories.d/customfeeds.list`  
  用户可将内网镜像源地址写入此文件。

- **热插拔脚本**：  
  - `88-replace-apk-source`：当网络接口（如 `wan`）**启动 (up)** 时，调用 `/usr/bin/replace-apk-source.sh`，将 APK 源替换为 `customfeeds.list` 中的地址。  
  - `99-restore-package`：当网络接口**关闭 (down)** 时，调用 `/usr/bin/restore-packages.sh`，恢复原有的软件包列表（例如从备份重新安装）。

- **手动执行**：  
  ```bash
  /usr/bin/replace-apk-source.sh
  /usr/bin/restore-packages.sh
  ```

## 安装方法

1. **将文件复制到设备**（假设当前目录在 OpenWrt 根文件系统下）：  
   ```bash
   cp -r etc/* /etc/
   cp -r usr/* /usr/
   ```

2. **设置可执行权限**：  
   ```bash
   chmod +x /etc/init.d/fan-control
   chmod +x /etc/hotplug.d/iface/*.sh
   chmod +x /usr/bin/*.sh
   ```

3. **启用并启动风扇控制服务**：  
   ```bash
   /etc/init.d/fan-control enable
   /etc/init.d/fan-control start
   ```

4. **（可选）配置自定义 APK 源**：  
   编辑 `/etc/apk/repositories.d/customfeeds.list`，每行一个源地址，例如：
   ```
   http://192.168.1.100/alpine/v3.19/main
   http://192.168.1.100/alpine/v3.19/community
   ```

## 使用说明

### 风扇控制

- 默认温度阈值可在 `fan-control.sh` 中修改（`TEMP_LOW`、`TEMP_HIGH`、`PWM_MIN`、`PWM_MAX` 等变量）。  
- 查看当前风扇转速：  
  ```bash
  cat /sys/class/hwmon/hwmon*/pwm1  # 具体路径依硬件而定
  ```

### APK 源管理

- **自动切换**：  
  当 WAN 接口获得 IP 时，会自动启用自定义源；当 WAN 接口断开时，会恢复默认源并重装之前通过自定义源安装的软件包。  
  热插拔脚本依赖接口名称（默认 `wan`），如接口名称不同请修改脚本中的 `INTERFACE` 变量。

- **手动恢复出厂包状态**：  
  执行 `restore-packages.sh` 前，请确保 `/etc/apk/repositories` 已指向官方源，脚本会尝试重新安装 `/usr/lib/apk/original_packages.list` 中记录的包。

## 注意事项

1. 本配置假设系统使用 **apk** 包管理器（而非 opkg）。若为 opkg 系统，需对应修改脚本中的 `apk` 命令为 `opkg`。
2. 热插拔脚本会在接口 up 时替换源，但不会自动安装新包；仅在接口 down 时恢复并重装原有包。如需在 up 时自动安装自定义源中的包，请自行修改 `88-replace-apk-source`。
3. 风扇控制依赖硬件 PWM 接口（通常位于 `/sys/class/hwmon`），请确认你的设备支持且路径正确。
4. 建议在部署前备份原系统配置文件。

## 依赖

- `bash` 或兼容 POSIX shell
- `apk` (Alpine Package Keeper)
- `lm-sensors`（可选，用于温度读取，脚本中可直接读取 `/sys/class/thermal`）

## 维护与自定义

- 修改风扇温度阈值：编辑 `/usr/bin/fan-control.sh` 中的 `TEMP_LOW`、`TEMP_HIGH` 变量。  
- 修改需要监控的网络接口：编辑 `/etc/hotplug.d/iface/88-replace-apk-source` 和 `99-restore-package`，将 `wan` 改为实际接口名。

---

**许可证**：GPL-3.0
**贡献**：欢迎提交 Issue 或 Pull Request。
