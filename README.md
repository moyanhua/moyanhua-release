# 莫言花 · MoyanHua

![version](https://img.shields.io/badge/version-1.4.9-blue)
![platform](https://img.shields.io/badge/platform-Android%2011%2B%20%C2%B7%20ARM64-green)
![framework](https://img.shields.io/badge/framework-Magisk%20%7C%20KernelSU%20%7C%20APatch-orange)
![license](https://img.shields.io/badge/license-Apache--2.0-lightgrey)

> 在天原作比翼鸟，于地原作连理枝

莫言花是一个面向 **Android 11+ / ARM64** 的性能与调度调优模块（Magisk / KernelSU / APatch）。

它由常驻 root 的守护进程 `sys.moyanhua-service` 与配套管理 App 组成：守护进程负责读写内核节点、
按负载与帧率做实时决策；App 负责可视化配置、状态显示与应用管理。所有参数集中在
`/data/adb/.config/moyanhua/` 下的配置文件里，**改文件即热重载**，不必重启守护进程。

**本仓库只发布安装包（Releases 附件），不包含源码。**

---

## 主要功能

### FAS · 帧感知调度

以**游戏实时帧率**为唯一依据的独立调度模式，只对白名单应用生效，退出游戏自动回到全局模式。

- **两种频率控制模型**：`lock` 把目标频率直接钉在 `scaling_max_freq` 并切到 `game_governor`；
  `floor` 只抬下限，上限交给系统 governor 按负载升降
- **完整 PID 控制器**：P / I / D 三档独立可调，可按目标帧率自动缩放增益（高刷游戏响应更快）
- **独立频率上限**：`cpu_ceil_pct` 按各簇自己的频率表计算，不继承接管前被压低的旧上限
- **温控与能耗**：SoC 温度三档（warn / hot / critical）分档限频，电池高温单独一档；
  可选能耗感知，按电池电压×电流估算整机功耗，超限即压向甜点频率，带滞回防抖
- **保守升频**：默认只在连续确认多帧「大卡顿」后才升频，单帧超时不会立刻拉满
- **加载画面检测 / 冷启动加速**：识别加载阶段冻结控制省电，进入工作状态后短时降低升频阈值快速起频
- **每应用三态开关**：`强制开启` / `默认（跟随白名单）` / `强制关闭`，可单独指定运行档位
- **三档调优 + 调频引擎切换**：省电 / 均衡 / 性能各自独立的 PID、余量、阈值、确认帧数；
  `engine` 可选 `moyanhua`（自研接管）或 `system`（让位给手机自带调度，只做白名单与模式管理）

### MYH · 用户态负载调速器

MoyanHua Load Governor：按 `/proc/stat` 逐核差分算出实时利用率，自主决定**每个簇的上下限**，
与系统 governor（walt / schedutil）协同工作。

- **四种接管方式**：`band`（上下限都写）/ `floor`（只抬下限）/ `lock`（钉死）/ `rate_limit`（不写频率，只调 governor 升频限速）
- **四档预置**：省电 / 均衡 / 性能 / 极速，各档独立的地板系数与回落斜率
- **逐簇参数表**：地板、上限、触摸时上限、触摸升频下限，按「簇角色序号」索引（按各簇最高频升序排名，0 = 最弱的簇）
- **快升慢降与迟滞防抖**：升频后短时禁降、降频阻尼而非取消，避免贴着阈值来回跳
- **尖峰抑制 / 上限饱和自愈**：识别瞬时尖峰不跟涨，上限撞顶时自动收敛
- **息屏 Doze**：熄屏后切入低功耗档
- **无交互降频**：长时间无触摸且负载低于阈值时，把下限直接放开到该簇最低频；一有触摸或负载立刻弹回
- **静置孤立大核**：静置时压低 `core_ctl/max_cpus`，让内核收掉多余大核；触摸或负载恢复时立即放开
- **`target_loads` 升频曲线旋钮**：默认留空（完全不碰），填了才逐簇写入，释放接管时按原值还原

### 墓碑 · 进程冻结

由 root 守护进程执行，不依赖 LSPosed / Xposed（APK 内另带可选的 LSPosed 模块形态，
用于在 system_server 侧做更早的介入，两条路径互斥仲裁，同一时刻只有一个在执行）。

- **三级冻结回退**：`Process.setProcessFrozen` → 直写 `cgroup.freeze` → `SIGSTOP`
- **冻结策略**：息屏冻结、后台冻结、前台自动解冻、新进程按规则冻结
- **崩溃自愈**：遗留的冻结状态在开机时自动兜底解冻，不会留下「假死」应用
- **已冻结显示器**：实时列出被冻结的进程，显示冻结时长、真实 RSS、是否已被系统放开

### 广告拦截

内置完整的 AdGuard Home（不依赖外部模块）：DNS 重定向、DoT / IPv6 53·853 端口丢弃、
广告缓存目录锁定，模块内提供 Web 控制台与日志查看。**默认关闭**，需手动开启。

### 其它子系统

- 应用编译器：逐包 dex2oat 编译并显示进度
- 应用程序优先级、刷新率调节、I/O 调度器切换、频率偏移
- 旁路充电、热芯（ThermalCore）温控服务、内存优化、文件系统分区检修、游戏预载

## 下载

前往 **[Releases](https://github.com/moyanhua/moyanhua-release/releases)** 下载最新版本的
`moyanhua-<版本>-<日期>-<标记>.zip`。当前最新：

| 版本 | 文件 | 大小 |
| --- | --- | --- |
| [v1.4.9](https://github.com/moyanhua/moyanhua-release/releases/tag/v1.4.9) | [moyanhua-1.4.9-20260918-myh-hotplug.zip](https://github.com/moyanhua/moyanhua-release/releases/download/v1.4.9/moyanhua-1.4.9-20260918-myh-hotplug.zip) | 126.6 MiB |

## 安装

### 刷入模块

1. 在 **Magisk / KernelSU / APatch** 的模块页面选择「从本地安装」
2. 选中下载好的 zip，等待刷入完成
3. 重启设备

重启后 App 会出现在启动器（名称 `莫言花`），首次进入按引导开启所需功能。守护进程随开机自启，
日志可在 App 内查看。

> 只支持 **ARM64** 设备；32 位设备与 x86 模拟器不支持。

## 使用与配置

### 配置文件

| 路径 | 说明 |
| --- | --- |
| `/data/adb/.config/moyanhua/` | 全部配置目录，改文件后**热重载**，不需要重启守护进程 |
| `/data/adb/.config/moyanhua/fas_config.toml` | FAS / MYH 的全部参数 |

配置更新采用**原地合并**：只改你动过的那一行，其余行、注释、自定义参数都会原样保留。

### FAS 示例

```toml
[fas]
# 调频引擎："moyanhua" = 自研接管；"system" = 让位给系统自带调度
engine = "moyanhua"
enabled = true
# lock = 把频率钉在目标点；floor = 只抬下限，上限交给 governor
control_mode = "lock"
# FAS 自己的上限（占各簇频率表顶档的百分比，100 = 用满）
cpu_ceil_pct = 100
# lock 模式下切换到的 governor
game_governor = "performance"
# 控制循环轮询间隔（毫秒）
poll_interval_ms = 100
# 能耗感知：超过上限就按比例压向甜点频率
power_aware = false
power_limit_w = 6.0
# 保守升频：只在连续确认多帧大卡顿后才升频
conservative_boost = true

[modes.performance]
enabled = true          # 每个模式独立的开关，默认全部关闭
margin = 5.0            # 帧率低于 目标+margin 才升频
thermal_threshold = 85.0
kp = 0.00025            # PID 增益
ki = 0.0
kd = 0.0

[modes.eco]
enabled = false

[modes.balance]
enabled = false
```

白名单与每应用模式：把包名写进对应模式的 `apps` 列表，该应用就固定按该模式运行；
每应用的三态开关（强制开启 / 默认 / 强制关闭）在 App 的应用列表中设置。
**FAS 生效条件 = 总开关开 && 当前模式开关开 && 前台应用在白名单（或被强制开启）。**

### MYH 示例

```toml
[myh.walt]
enabled = true
# 升频曲线（governor 的升频积极性）：负载% 与频率成对
# 留空 "" = 完全不碰（默认），填了才逐簇写入、释放接管时按原值还原
target_loads = ""
up_rate_limit_us = "0"
down_rate_limit_us = "0"

[myh.hotplug]
# 静置孤立大核：长时间无交互时各簇保留的在线核数上限
# 按「簇角色序号」索引（按各簇最高频升序排名），0 = 该簇不参与
enabled = false
idle_max_cpus = [0, 0, 0, 1, 0, 0, 0, 0]
```

MYH 的四档预置、逐簇频率表、无交互降频阈值等，均可在 App 的 MYH 页面可视化编辑，
改动会直接落回配置文件；手动改文件后再进 App，界面会重新加载以保持一致。

## 能力总览

| 模块 | 作用 | 关键旋钮 |
| --- | --- | --- |
| FAS | 按游戏实时帧率决定频率 | 白名单、每应用三态、`control_mode`、`cpu_ceil_pct`、PID、温控、能耗上限、加载画面检测、冷启动加速 |
| MYH | 按逐核负载决定每簇上下限 | 四档预置、`take_over` 四种接管方式、逐簇频率表、快升慢降、无交互降频、静置孤立大核、`target_loads` |
| 墓碑 | 冻结后台与息屏进程 | cgroup v2 / v1 / SIGSTOP 三级回退、冻结策略、崩溃自愈、冻结显示器 |
| 广告拦截 | 内置 AdGuard Home | DNS 重定向、DoT / IPv6 丢弃、Web 控制台 |

## 校验

zip 内每个文件都带一个同名的 `.sha256`（内容为裸哈希，无换行），可逐个核对：

```bash
# 查看包内期望值
unzip -p moyanhua-1.4.9-20260918-myh-hotplug.zip moyanhua.apk.sha256

# 与本机实际值比对
sha256sum moyanhua.apk

unzip -p moyanhua-1.4.9-20260918-myh-hotplug.zip libs/arm64-v8a/sys.moyanhua-service.sha256
sha256sum libs/arm64-v8a/sys.moyanhua-service
```

本次发布包的整包校验值：

```
moyanhua-1.4.9-20260918-myh-hotplug.zip
SHA-256  454147c13a051d87a44480e9fc3477ab12aa2c6aa276266ce82fc3031e57e295
大小     132,770,390 字节（126.6 MiB）
```

## 常见问题

**Q：开了 FAS，游戏里却没有反应？**
A：FAS 的生效条件是「总开关开启 **且** 当前模式开关开启 **且** 前台应用在白名单内（或被单独设为强制开启）」，
三者缺一不可。另外 `engine` 若是 `system`，模块不会接管调频，只做白名单与模式管理。

**Q：为什么小核的负载常年显示 80~100%？**
A：逐核负载是**非空闲时间占比**（`/proc/stat` 里 idle 之外的时间都算忙），不是「算力占用」。
省电核停在最低频时，单位时间内仍有大量非空闲时间，所以数字高但实际算力占用很低。
小核停在最低档属于正常行为，不需要处理。

**Q：打开了「静置孤立大核」但看不出效果？**
A：它依赖内核暴露 `<cpuN>/core_ctl/max_cpus` 节点，部分机型没有该节点，此时该项自动跳过；
另外 `take_over` 设为 `rate_limit` 时整个频率控制循环不运行，这一项也不参与。

**Q：`target_loads` 填了却没变化？**
A：它默认留空，留空表示**完全不碰**；只有填了值才会写到对应 governor 的 `target_loads` 节点。
不同内核该节点的路径与格式可能不同，写入失败会自动跳过，不影响其它功能。

**Q：冻结会把应用杀掉吗？后台还会不会收到消息？**
A：冻结只是让进程暂停（cgroup freeze 或 SIGSTOP），**不回收内存、不杀进程**，解冻后继续运行。
被冻结期间该应用的后台任务（下载、推送等）会暂停，需要时可在 App 中把它加入不冻结名单。

**Q：刷完会不会更费电？**
A：守护进程只在需要时写节点，轮询间隔可调（默认 100ms）；`target_loads` 与静置孤立大核默认都不启用。
FAS 默认的保守升频也意味着「不轻易拉满」。

**Q：模块会不会和系统自带的性能调度打架？**
A：`fas.engine` 设为 `system` 时模块完全不碰频率节点；墓碑的冻结会与系统「暂停执行已缓存」仲裁，
两条路径写的是同一个 `cgroup.freeze`，不会互相覆盖。

## 已知限制

- 仅支持 **ARM64**；32 位设备与 x86 模拟器不支持。
- 「静置孤立大核」依赖内核暴露 `core_ctl` 节点，缺失时该项自动跳过（不影响其它功能）。
- `target_loads` 属于 governor 本职参数，默认**不修改**；只有手动填值后才会写入，并在释放接管时按原值还原。
- 部分内核未实现 eBPF 帧探针，此时 FAS 会降级为不带帧数据的模式继续工作，功能不至于完全失效。

## 更新日志

见 **[Releases](https://github.com/moyanhua/moyanhua-release/releases)**。

## 致谢

- [AdGuard Home](https://github.com/AdguardTeam/AdGuardHome) —— 内置广告拦截
- [Magisk](https://github.com/topjohnwu/Magisk) / [KernelSU](https://github.com/tiann/KernelSU) / [APatch](https://github.com/bmax121/APatch) —— 模块框架
- 包内部分组件遵循其各自的开源协议，详见包内文件

## 协议

Copyright © 2026-2027 莫言花（MoyanHua）

本项目以 [Apache License 2.0](LICENSE) 授权发布。

## 免责声明

刷机与内核调优存在风险，请自行评估并做好备份。因使用本模块造成的任何直接或间接损失，
作者不承担任何责任。
