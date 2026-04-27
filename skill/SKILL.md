# SKILL.md - 海光DCU管理工具-hy-smi

> 触发词：海光DCU、海光加速器、DCU、HCU、C-3000

## 简介

**hy-smi** 是海光DCU（Data Center Unit，数据中心加速器）的系统管理接口工具，对应AMD ROCm生态中的`rocm-smi`。支持C-3000系列HCU（Hardware Compute Unit）设备管理。

在容器中运行时：
```bash
docker exec <container> hy-smi <options>
```

## 快速常用命令

| 用途 | 命令 |
|------|------|
| 查看所有设备概览 | `hy-smi -a` |
| 查看设备ID和序列号 | `hy-smi -i --showserial --showuniqueid` |
| 查看温度/功耗 | `hy-smi -t -P` |
| 查看利用率 | `hy-smi -u --showhcuutil --showcuutil` |
| 查看内存使用 | `hy-smi --showmemuse --showmemavailable` |

| 查看时钟频率 | `hy-smi -c -g --showclkfrq` |
| 查看正在运行的进程 | `hy-smi --showpids` |
| JSON格式输出 | `hy-smi -a --json` |


## 选项分类

### 🔍 显示/查询选项

**基本信息查询**
- `-i, --showid` - HCU ID
- `-v, --showvbios` - VBIOS版本
- `--showdriverversion` - 内核驱动版本
- `--showproductname` - SKU/厂商名称
- `--showserial` - 序列号
- `--showpartnum` - Part Number
- `--showuniqueid` - Unique ID
- `--showbus` - PCI总线号
- `--showfwinfo` - 固件信息

**硬件状态**
- `-t, --showtemp` - 温度
- `-P, --showpower` - 当前平均功耗
- `-u, --showuse` - HCU利用率
- `--showmemuse` - 显存使用
- `--showhcuutil` - HCU活跃时间占比
- `--showcuutil` - CU利用率（波前占用率）
- `--showwaveutil` - 波前驻留率
- `--showvoltage` - 电压
- `--showhash` - 哈希状态

**内存信息**
- `--showmemvendor` - 显存厂商
- `--showmemdc` - 显存Date Code
- `--showmemavailable` - 可用显存
- `--showmemoverdrive` - 显存超频等级
- `--showmeminfo TYPE` - 指定内存块信息
- `--showpagesinfo` - 页面信息（retired/pending/unreservable）
- `--detail` - 显示坏页详情

**时钟与频率**
- `-c, --showclocks` - 当前时钟频率
- `-g, --showhcuclocks` - HCU时钟频率
- `-s, --showclkfrq` - 支持的HCU和Memory时钟
- `-o, --showoverdrive` - 查看/设置HCU时钟超频等级
- `-m, --showmemoverdrive` - 查看/设置显存超频等级
- `--getmaxsclk` - 获取最大sclk数值(MHz)
- `--getminsclk` - 获取最小sclk数值(MHz)
- `--getmaxsocclk` - 获取最大socclk数值(MHz)
- `--getminsocclk` - 获取最小socclk数值(MHz)
- `--showmaxpower` - 最大功耗
- `--showboost` - gfx boost等级
- `--showfdr` - 频率调整原因
- `--showfdrhistory` - 频率调整历史

**拓扑与连接**
- `--showtopo` - 硬件拓扑
- `--showtopoaccess` - HCU间链接可达性
- `--showtopoweight` - HCU间相对权重
- `--showtopohops` - HCU间跳数
- `--showtopotype` - 链接类型
- `--shownodeid` - 节点ID
- `--showxhclstatus` - xhcl链接状态
- `--showxgmierr` - XGMI错误计数
- `--showtoponuma` - NUMA节点

**带宽相关**
- `-b, --showbw` - PCIe带宽
- `--showdfbw/--showmembw` - DF读写带宽
- `--showumcbw` - UMC读写带宽
- `--showxhclbw` - xhcl带宽
- `--channel CHANNEL` - 选择UMC通道(0-31或all)
- `--link LINK` - 选择xhcl链接(0-6或all)
- `--direction` - 0=接收,1=发送
- `--delay` - 延迟(ms)
- `--count` - 统计次数
- `--bwdetail` - 带宽详情

**健康与异常**
- `--healthcheck` - 健康状态检查
- `--showrasinfo BLOCK` - RAS使能信息/错误计数
- `--showexceptioninfo` - 异常信息

### ⚙️ 设置选项

> ⚠️ **安全警告：以下操作涉及硬件配置变更，部分操作不可逆或可能导致服务中断，操作前请务必确认。**

**🔴 高危操作（可能损坏硬件或导致DCU不可用）**
| 操作 | 风险 | 建议 |
|------|------|------|
| `--setmemiovol` | 设置GDDR6 memio电压，过高会烧毁显存 | 仅在厂商指导下操作 |
| `--setpoweroverdrive` | 超过默认功耗上限，长期使用影响硬件寿命 | 谨慎使用，监控温度 |
| `--rasinject` | 向指定block注入RAS poison（仅对 unsecured 板卡有效） | 极其危险，仅用于测试 |
| `--loaddriver` / `--unloaddriver` | 卸载驱动会导致正在运行的计算任务中断 | 确保无重要任务运行 |

**🟡 中危操作（影响运行状态或需要重启生效）**
| 操作 | 风险 | 建议 |
|------|------|------|
| `--setsclk` / `--setmclk` / `--setsocclk` | 手动超频可能引发稳定性问题或过热 | 先查温度，确保散热良好；切换到 `manual` 性能等级后再操作 |
| `--enablelowpower` / `--setlowpowerdelay` | 低功耗模式可能影响性能响应 | 根据业务需求谨慎开启 |
| `--setboost` | 切换boost等级影响频率策略 | 确认当前散热和功耗预算 |
| `--rasenable` / `--rasdisable` | 变更RAS策略可能影响错误检测 | 了解相关block的RAS功能后再操作 |

**🟢 低危操作（查询和安全设置）**
- `--setperflevel` - 切换性能等级（auto/low/high/manual），一般不会造成硬件损坏
- `--setprofile` - 设置电源配置文件
- `--setmaxprocess` - 设置最大进程数
- `--setautorunhyckptd` / `--runhyckptd` / `--stophyckptd` - Hyckptd服务管理
- `--setautoloaddriver` - 驱动自动加载设置（重启后生效）
- `--setcustomer` - 客户标识设置

---

**频率与功耗**
- `--setsclk LEVEL` - 设置HCU时钟等级
- `--setmclk LEVEL` - 设置显存时钟等级
- `--setsocclk LEVEL` - 设置SOC时钟等级
- `--setperflevel LEVEL` - 设置DPM性能等级(auto/low/high/manual)
- `--setpoweroverdrive WATTS` - 设置Power OverDrive(瓦特)
- `--setboost LEVEL` - 设置boost等级(0-AI,1-HPC,2-AI(typical),3-HPC(typical))
- `--enablelowpower 1/0` - 启用/禁用低功耗模式
- `--setlowpowerdelay TIME` - 进入低功耗延迟(ms)
- `--setmemiovol VOLTAGE LEVEL` - 设置GDDR6 memio电压

**驱动管理**
- `--loaddriver` - 手动加载驱动
- `--unloaddriver` - 手动卸载驱动
- `--setautoloaddriver <value>` - 设置启动自动加载(0/1)
- `--getautoloaddriver` - 获取自动加载设置
- `--setdriverparams <value>` - 设置驱动参数
- `--getdriverparams` - 获取驱动参数
- `--addstabledriverparams` - 添加稳定驱动参数
- `--clearstabledriverparams` - 清除稳定驱动参数

**其他设置**
- `--setmaxprocess COUNT` - 设置最大进程数
- `--setprofile LEVEL` - 指定电源配置文件等级
- `--rasenable BLOCK ERRTYPE` - 启用RAS
- `--rasdisable BLOCK ERRTYPE` - 禁用RAS
- `--rasinject BLOCK ERRTYPE` - 注入RAS poison

**配置保存/恢复**
> ⚠️ 重要操作前建议先备份配置
- `--save FILE` - 将当前HCU配置保存到指定文件
- `--load FILE` - 从指定文件加载配置（需确认设备ID匹配）

### 🔄 重置选项
- `-r, --resetclocks` - 重置时钟和超频
- `--resetpoweroverdrive` - 重置功耗OverDrive
- `--resetxgmierr` - 重置XGMI错误计数

### 📊 输出选项
- `--json` - JSON格式输出
- `--csv` - CSV格式输出
- `--old` - CSV中时间使用旧格式
- `--loglevel LEVEL` - 日志级别(debug/info/warning/error/critical)

## 使用示例

### 1. 默认查看所有设备（不带参数）
```bash
hy-smi
```
输出示例：
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

### 2. 批量查询多卡信息
```bash
hy-smi -d 0 1 2 -a  # 查看设备0,1,2的所有信息
```

### 3. 设置单卡频率
```bash
hy-smi -d 0 --setsclk 7 --setperflevel manual  # 设置卡0到等级7并切换手动模式
```

### 4. 导出配置
```bash
hy-smi -d 0 --save /tmp/dcu_config.txt        # 保存配置
hy-smi --load /tmp/dcu_config.txt             # 加载配置
```

## 环境要求

- 驱动版本: 6.2.22+ (支持 `--finegrain` 选项)
- 需要root权限 (部分操作如驱动加载)
- 容器内使用需添加 `--privileged` 或映射 `/dev/` 设备：
  - `/dev/kfd` - HSA kernel driver
  - `/dev/dri` - Direct Rendering Manager

## 注意事项

> ⚠️ **执行设置类操作前，请务必阅读上方安全警告。**

1. **手动性能等级**: 设置 `--setsclk`/`--setmclk` 前需先 `--setperflevel manual`
2. **超频风险**: 显存/HCU超频可能损坏硬件，请谨慎操作
3. **多设备操作**: 使用 `-d` 指定多个设备，设备ID从0开始
4. **操作前检查**: 重要操作前建议先执行 `hy-smi -a` 查看当前状态备份配置

## SSH远程执行模板

```bash
sshpass -p '<password>' ssh -o StrictHostKeyChecking=no root@<host> \
  "docker exec <container> hy-smi <command>"
```

## 错误排查

**通用排查步骤**
```bash
dmesg -T | grep hycu  # 查看hycu相关内核日志
```
如果存在异常报错建议使用者可以尝试**重启设备**，如果无法解决再将报错信息提交到 https://forum.sourcefind.cn/ 寻求帮助。

| 问题 | 解决方案 |
|------|----------|
| `Permission denied` | 需要root权限，加sudo或--privileged |
| 设备未找到 | 检查HCU是否在线，驱动是否加载 `lsmod |grep -E "hydcu|hycu"` |
| 操作失败 | 查看 `--showexceptioninfo` 查看异常信息 |
| 温度过高 | 检查散热系统，`-t --showtemp` 查看温度 |