# hy-smi - 海光DCU管理工具Skill

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Platform: Hygon DCU](https://img.shields.io/badge/Platform-Hygon%20DCU-blue.svg)](https://www.hygon.cn/)

**hy-smi** 是海光DCU（Data Center Unit，数据中心加速器）的系统管理接口工具，对应AMD ROCm生态中的 `rocm-smi`。支持C-3000系列HCU（Hardware Compute Unit）设备管理。

本仓库是 **Hermes Agent** 的技能文件，包含完整的 hy-smi 命令参考和使用指南。

## 目录结构

```
hy-smi/
├── skill/                    # Hermes Agent 技能文件
│   └── SKILL.md             # 完整命令参考文档
├── scripts/
│   └── examples.sh          # 常用命令示例脚本
└── README.md                # 本文件
```

## 快速开始

### 安装

将 `skill/SKILL.md` 文件复制到你的 Hermes Agent 技能目录：

```bash
# Hermes Agent 技能目录
~/.hermes/skills/

# 或
~/.agents/skills/
```

### 基本使用

```bash
# 查看所有设备概览
hy-smi

# 查看设备ID和序列号
hy-smi -i --showserial --showuniqueid

# 查看温度/功耗
hy-smi -t -P

# 查看利用率
hy-smi -u --showhcuutil --showcuutil

# 查看内存使用
hy-smi --showmemuse --showmemavailable

# 查看时钟频率
hy-smi -c -g --sclk --showclkfrq

# 查看正在运行的进程
hy-smi --showpids

# JSON格式输出
hy-smi -a --json
```

### 容器中使用

```bash
docker exec <container> hy-smi <options>
```

## 输出示例

```
================================= System Management Interface ==================================
================================================================================================
HCU     Temp     AvgPwr     Perf     PwrCap     VRAM%      HCU%      Dec%      Enc%      Mode
0       40.0C    157.0W     high     800.0W     0%         0.0%      0.0%      0.0%      Normal
1       41.0C    162.0W     high     800.0W     0%         0.0%      0.0%      0.0%      Normal
2       39.0C    159.0W     high     800.0W     0%         0.0%      0.0%      0.0%      Normal
3       37.0C    161.0W     high     800.0W     0%         0.0%      0.0%      0.0%      Normal
4       40.0C    158.0W     high     800.0W     0%         0.0%      0.0%      0.0%      Normal
5       38.0C    161.0W     high     800.0W     0%         0.0%      0.0%      0.0%      Normal
6       36.0C    160.0W     high     800.0W     0%         0.0%      0.0%      0.0%      Normal
7       36.0C    161.0W     high     800.0W     0%         0.0%      0.0%      0.0%      Normal
================================================================================================
======================================== End of SMI Log ========================================
```

## 常用命令速查

| 用途 | 命令 |
|------|------|
| 查看所有设备概览 | `hy-smi -a` |
| 查看设备ID和序列号 | `hy-smi -i --showserial --showuniqueid` |
| 查看温度/功耗 | `hy-smi -t -P` |
| 查看利用率 | `hy-smi -u --showhcuutil --showcuutil` |
| 查看内存使用 | `hy-smi --showmemuse --showmemavailable` |
| 查看时钟频率 | `hy-smi -c -g --sclk --showclkfrq` |
| 查看正在运行的进程 | `hy-smi --showpids` |
| JSON格式输出 | `hy-smi -a --json` |

## 错误排查

```bash
# 查看hycu相关内核日志
dmesg -T | grep hycu
```

如果存在异常报错建议使用者可以尝试**重启设备**，如果无法解决再将报错信息提交到 https://forum.sourcefind.cn/ 寻求帮助。

常见问题：

| 问题 | 解决方案 |
|------|----------|
| `Permission denied` | 需要root权限，加sudo或--privileged |
| 设备未找到 | 检查HCU是否在线，驱动是否加载 `lsmod |grep -E "hydcu|hycu"` |
| 操作失败 | 查看 `--showexceptioninfo` 查看异常信息 |
| 温度过高 | 检查散热系统，`-t --showtemp` 查看温度 |

## 环境要求

- 驱动版本: 6.2.22+ (支持 `--finegrain` 选项)
- 需要root权限 (部分操作如驱动加载)
- 容器内使用需添加 `--privileged` 或映射 `/dev/` 设备：
  - `/dev/kfd` - HSA kernel driver
  - `/dev/dri` - Direct Rendering Manager

## 注意事项

⚠️ **执行设置类操作前，请务必阅读上方安全警告。**

1. **手动性能等级**: 设置 `--setsclk`/`--setmclk` 前需先 `--setperflevel manual`
2. **超频风险**: 显存/HCU超频可能损坏硬件，请谨慎操作
3. **多设备操作**: 使用 `-d` 指定多个设备，设备ID从0开始
4. **操作前检查**: 重要操作前建议先执行 `hy-smi -a` 查看当前状态备份配置

## SSH远程执行

```bash
sshpass -p '<password>' ssh -o StrictHostKeyChecking=no root@<host> \
  "docker exec <container> hy-smi <command>"
```

## 相关资源

- [海光开发者社区](https://forum.sourcefind.cn/)
- [海光DCU官网](https://www.hygon.cn/)

## License

MIT License - 详见 [LICENSE](LICENSE) 文件