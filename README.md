# 莫言花 · MoyanHua

![version](https://img.shields.io/badge/version-1.4.9-blue)
![platform](https://img.shields.io/badge/platform-Android%2011%2B%20%C2%B7%20ARM64-green)
![license](https://img.shields.io/badge/license-Apache--2.0-lightgrey)

> 在天原作比翼鸟，于地原作连理枝

莫言花是一个面向 **Android 11+ / ARM64** 的性能与调度调优模块（Magisk / KernelSU / APatch）。
它由常驻 root 守护进程 `sys.moyanhua-service` 与配套管理 App 组成：守护进程负责实际读写内核节点、
按负载与帧率做实时决策；App 负责可视化配置、状态显示与应用管理。所有配置集中在
`/data/adb/.config/moyanhua/`，改文件即可热重载，不必重启守护进程。

**本仓库只发布安装包（Releases 附件），不包含源码。**

---

## 下载与安装

1. 打开本仓库的 **Releases** 页，下载最新版本的 `moyanhua-<版本>-<日期>-<标记>.zip`。
2. 在 **Magisk / KernelSU / APatch** 中「从本地安装」选中该 zip，刷入后重启。
3. 重启后 App 会出现在启动器（`莫言花`），首次进入按引导开启所需功能。

> 只支持 **ARM64** 设备；模块与 App 均不依赖 LSPosed / Xposed。

## 版本

| 项目 | 值 |
| --- | --- |
| 模块版本 | 1.4.9 |
| 模块 ID | `moyanhua` |
| 支持框架 | Magisk / KernelSU / APatch |
| 支持系统 | Android 11+（ARM64） |
| 建议机型 | 骁龙 8+ 及以上 / 天玑 8100 及以上 |

## 主要功能

- **FAS 帧感知调度**：以游戏实时帧率为依据的独立第四模式，只对白名单应用生效，退出游戏自动回到全局模式。
  支持 `lock` / `floor` 两种频率控制模型、完整 PID 控制器（P/I/D 可调）、升降温控与能耗上限、
  加载画面检测、冷启动加速、每应用的「强制开启 / 默认 / 强制关闭」三态开关与省电·均衡·性能三档调优。
- **MYH 用户态调速器**：内置的用户态负载调速器（MoyanHua Load Governor），按 `/proc/stat` 逐核差分
  算出实时利用率，自主决定每簇上下限，与系统 governor（walt / schedutil）协同。支持 `band` / `floor` /
  `lock` / `rate_limit` 四种接管方式、省电·均衡·性能·极速四档预置、快升慢降与迟滞防抖、尖峰抑制、
  上限饱和自愈、息屏 Doze、无交互降频（静置时把下限直接放开，含用户配过的逐簇下限）、
  以及 `target_loads` 升频曲线旋钮。不关核心：静置只放开频率下限，核心数交给系统。
  不人工封顶：`perf_ceil` 只夹「我们自己写下去的下限最高抬到哪」，不改内核 `scaling_max_freq`；
  真封顶有两条路 —— `freq_cap` / `freq_cap_touch`（绝对 kHz、逐簇），或逐档的 `max_cap`
  （比例、仅 `freq_mode = auto` 生效），两者默认都是 0 = 不封，取值更低者。
- **墓碑（进程冻结）**：由 root 守护进程执行，无需 LSPosed。cgroup v2 / v1 / SIGSTOP 三级自动探测与回退，
  息屏冻结、后台冻结、前台自动解冻、新进程冻结；崩溃自愈（遗留冻结状态开机自动兜底解冻），
  并提供实时「已冻结」显示器（冻结时长、真实 RSS、是否被系统放开）。
- **内置去广告**：完整内置 AdGuard Home（不依赖外部模块），含 DNS 重定向、DoT/IPv6 53·853 丢弃、
  广告缓存目录锁定，模块内提供 Web 控制台与日志查看。默认关闭，需手动开启。
- **其他**：应用编译器（逐包 dex2oat 编译并显示进度）、应用程序优先级、刷新率调节、I/O 调度器切换、
  旁路充电、热芯（ThermalCore）温控服务、频率偏移、内存优化、文件系统分区检修、游戏预载等。

## 校验

zip 内每个文件都带一个同名的 `.sha256`（内容为裸哈希，无换行），可逐个核对：

```bash
# 查看包内期望值
unzip -p moyanhua-1.4.9-20260919-myh-params3.zip moyanhua.apk.sha256

# 与本机实际值比对
sha256sum moyanhua.apk
unzip -p moyanhua-1.4.9-20260919-myh-params3.zip libs/arm64-v8a/sys.moyanhua-service.sha256
sha256sum libs/arm64-v8a/sys.moyanhua-service
```

本次发布包的整包校验值：

```
moyanhua-1.4.9-20260919-myh-params3.zip
SHA-256  79aae76ec01569af79341f475a8450e2bb2dae899ce78ac6ac720d8ff05f5184
大小     102,369,894 字节
```

## 已知限制

- 仅 ARM64；32 位设备与 x86 模拟器不支持。
- `target_loads` 属于 governor 本职参数，默认**不修改**，只有手动填值后才会写入，并在释放接管时按原值还原。

## 协议

本项目以 [Apache License 2.0](LICENSE) 授权发布。

## 免责声明

刷机与内核调优存在风险，请自行评估并做好备份。因使用本模块造成的任何直接或间接损失，
作者不承担任何责任。
