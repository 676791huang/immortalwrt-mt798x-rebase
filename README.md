# OpenFi 6C — ImmortalWrt MT798x (25.12)

OpenFi 6C（MT7981B）的固件树。底座是
[`chasey-dev/immortalwrt-mt798x-rebase`](https://github.com/chasey-dev/immortalwrt-mt798x-rebase)
（ImmortalWrt 25.12 + MTK OpenWrt Feeds），我们只加这一块板子的支持：
**底座一行不改、每天自动跟随上游**。

| 分支 | 是什么 |
|---|---|
| **`product`** | 我们的树（默认分支）：底座 + OpenFi 6C 支持 |
| `main` | 上游镜像：永远等于上游 `25.12`，只由同步 workflow 快进（禁止 force-push / 删除） |

- 要做固件 → 用 `product`。
- 想知道「从哪来、改了哪些、许可怎么算」→ **[ATTRIBUTION.md](ATTRIBUTION.md)**。

---

## 设备

| 项 | 说明 |
|---|---|
| SoC | MediaTek MT7981B（Filogic 820），256 MB RAM |
| 5G | M.2 模组；面板用 [`luci-app-5gmodem`](https://github.com/fildunsky/luci-app-5gmodem)（钉 `v4.0.9`） |
| 无线 | MT7981，`mt_wifi` 闭源驱动（构建时由 feed 拉取，不在本仓库里） |

## 我们在底座上加了什么（一眼版）

| 类别 | 内容 |
|---|---|
| 设备树 / 板级 | `mt7981b-openfi-6c.dts`、`filogic.mk`、`01_leds`、`02_network` |
| 内核 | `mtk_hnat` 四个文件（`hnat.c` / `hnat.h` / `hnat_debugfs.c` / `hnat_nf_hook.c`）的修复 |
| 自研包 | 独立仓库 [`openfi-packages`](https://github.com/676791huang/openfi-packages)：风扇 `luci-app-openfi-fan` + 公共库 `openfi-lib`，以 feed 拉取 |
| 出场调优 | SQM/CAKE（`files/etc/config/sqm`）、关掉 turboacc 的 HNAT 快路、`sysctl` 网络调优、5G 模组默认项 |
| 脚本 | `scripts/openfi/`：一致性检查、LED 检查、musl 检查、同步到树 |
| CI | `build.yml`（五层缓存出固件）、`sync-upstream.yml`（镜像 + 合并 + 两道闸门） |

## 上游同步（自动）

`.github/workflows/sync-upstream.yml`，每天 **UTC 20:00（北京 04:00）** 跑一次：

1. 上游 `25.12` → `main` **快进**（上游不是 `main` 的后代就报错停下，防 force-push 搞乱）；
2. `main` 合并进 `product`；有冲突就开 issue，**这次一个字节都不推**；
3. **复核闸门**：上游若动到我们改过的那 8 个上游文件（`hnat*`、设备树、`filogic.mk`、`01_leds`、`02_network`），
   开 issue 附改动 diff，要求人工过一眼 —— 能合并不等于语义正确；
4. **构建闸门**：同步成功后自动触发一次 `build.yml`。

手动跑：Actions → `sync-upstream` → *Run workflow*，勾 `dry_run` 只试合并、什么都不推。

## 构建

- Actions → `build` → *Run workflow*；勾 `publish` 才会建 Release。
- 推 `product` 且改动涉及设备/包/脚本/内核补丁时自动触发；每周一次保活构建（UTC 周日 18:00）。
- 产物：`openfi6c-*.bin` + MD5，日志在 Artifacts。
- 缓存五层（dl / feeds / 工具链 / 内核与包编译目录 / ccache）：冷启动 60~90 分钟，之后 15~25 分钟。

## 这棵树的上游链

```
OpenWrt (openwrt/openwrt)
  └── ImmortalWrt (immortalwrt/immortalwrt) @ openwrt-25.12
        └── chasey-dev/immortalwrt-mt798x-rebase @ 25.12
              └── 本仓库
```

上游 README 里钉住的三个提交，我们完整继承（没有改动）：

- ImmortalWrt：[`761b7ff`](https://github.com/immortalwrt/immortalwrt/commit/761b7ff22cbf7c13f28fc631bd70a84d295fd627) — *Merge Official Source*
- MTK OpenWrt Feeds：[`3f66af2`](https://github.com/mediatek/mtk-openwrt-feeds/commit/3f66af253becc66364f90669e58d59db19de6d95) — *Fix patch conflict issue*
- l1parser：[`081bb31`](https://github.com/chasey-dev/l1parser/commit/081bb31211efc74594d25bfd1bb5811f3408a205) — *feat(ucode): add get all device map support*

## 关于外部设备 HNAT

> [!WARNING]
> 外部设备 HNAT 支持比较基础，未覆盖各种类型，谨慎使用。

> [!IMPORTANT]
> 使用外部设备 HNAT 时，桥接口（如 `br-lan`）里要保留 `rxppd` 接口。

|               | Ext as WAN | Ext as LAN          |
|   :----:      |   :----:   | :----:              |
| **Ethernet**  |     ✔️     | ❌                  |
| **AP/ApCli**  |     ✔️     | ⚠️(**Untested**)   |

## 致谢

外部设备 HNAT 的实现沿用 [Padavanonly 的仓库](https://github.com/padavanonly/immortalwrt-mt798x-6.6)。

## 许可

GPL-2.0，见 `COPYING` / `LICENSES/`。各上游组件的版权归各自作者，详见 [ATTRIBUTION.md](ATTRIBUTION.md)。