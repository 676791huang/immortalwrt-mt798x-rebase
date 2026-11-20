# 溯源声明 / Attribution

本仓库不是从零写的。它是在别人的树上**只加 OpenFi 6C 设备支持**得到的，所以在这里把
「从哪来、改了哪些、许可怎么算」写清楚。

---

## 一、这份代码从哪来

| 角色 | 仓库 | 分支 / 提交 |
|---|---|---|
| **本仓库** | `676791huang/immortalwrt-mt798x-rebase` | 分支 `product` |
| **直接上游** | [`chasey-dev/immortalwrt-mt798x-rebase`](https://github.com/chasey-dev/immortalwrt-mt798x-rebase) | 分支 `25.12`（本仓库镜像为 `main`） |
| **当前基点** | 同上 | `458d2f7e92`<br>*mediatek: refresh repo patches (2026-11-11)*<br>2026-11-15 |

> 本仓库是上游的 **fork**，不是某个时间点的快照：
> `main` 每天自动快进到上游 `25.12`，`product` = `main` + 我们的提交。
> 想知道现在跟到哪个提交，看 `main` 的最新提交即可。

## 二、上游自己又基于什么

`chasey-dev/immortalwrt-mt798x-rebase` 的 README 里钉了三个上游提交，
我们完整继承下来（**没有改动**）：

| 组件 | 仓库 | 提交 |
|---|---|---|
| ImmortalWrt | [`immortalwrt/immortalwrt`](https://github.com/immortalwrt/immortalwrt) | [`761b7ff`](https://github.com/immortalwrt/immortalwrt/commit/761b7ff22cbf7c13f28fc631bd70a84d295fd627) |
| MTK OpenWrt Feeds | [`mediatek/mtk-openwrt-feeds`](https://github.com/mediatek/mtk-openwrt-feeds) | [`3f66af2`](https://github.com/mediatek/mtk-openwrt-feeds/commit/3f66af253becc66364f90669e58d59db19de6d95) |
| l1parser | [`chasey-dev/l1parser`](https://github.com/chasey-dev/l1parser) | [`081bb31`](https://github.com/chasey-dev/l1parser/commit/081bb31211efc74594d25bfd1bb5811f3408a205) |

而这棵树的**再上游**是：

```
OpenWrt (openwrt/openwrt)
   └── ImmortalWrt (immortalwrt/immortalwrt) @ openwrt-25.12
          └── chasey-dev/immortalwrt-mt798x-rebase @ 25.12
                 └── 本仓库（main = 镜像，product = 我们的树）
```

`feeds.conf.default` 里 ImmortalWrt / OpenWrt 官方的五个 feed 指向各自的
`openwrt-25.12` 分支，**未做任何修改**：

```
src-git packages  https://github.com/immortalwrt/packages.git;openwrt-25.12
src-git luci      https://github.com/immortalwrt/luci.git;openwrt-25.12
src-git routing   https://github.com/openwrt/routing.git;openwrt-25.12
src-git telephony https://github.com/openwrt/telephony.git;openwrt-25.12
src-git video     https://github.com/openwrt/video.git;openwrt-25.12
```

我们只在末尾**追加**了一条 5G 面板的 feed（钉版本，避免上游漂移）：

```
src-git 5gmodem https://github.com/fildunsky/luci-app-5gmodem.git;v4.0.9
```

另外，外部设备 HNAT 相关实现沿用自
[`padavanonly/immortalwrt-mt798x-6.6`](https://github.com/padavanonly/immortalwrt-mt798x-6.6)
（这是上游 README 的 Acknowledgements 里就写明的，我们照旧保留）。

## 三、我们改了什么

**原则：底座一行不动，只加「让这块板子能用、能出厂」的部分。**
下面是相对当前基点（`458d2f7e92`）的完整清单。

### 新增

| 路径 | 内容 |
|---|---|
| `target/linux/mediatek/dts/mt7981b-openfi-6c.dts` | 设备树（LED / 按键 / GPIO 引出） |
| `defconfig/openfi6c.config` | 本设备的构建配置 |
| `openfi-packages` 仓库的 `luci-app-openfi-fan/` | 风扇控制（C 程序 + LuCI 页面 + 物理开关 + uci-defaults）；以 feed 拉取，不再放在本树 |
| `openfi-packages` 仓库的 `openfi-lib/` | 公共库；以 feed 拉取 |
| `scripts/openfi/` | 一致性检查、LED 检查、musl 检查、同步到树 |
| `files/etc/config/sqm`、`files/etc/config/turboacc` | 出场网络调优（CAKE 整形、关掉 turboacc 的 HNAT 快路） |
| `files/etc/sysctl.d/99-openfi-net-tweaks.conf` | 延迟 / 缓冲区调优 |
| `files/etc/uci-defaults/95-openfi-apk-mirror`、`96-openfi-5gmodem`、`97-openfi-usb-nosuspend` | 首启动默认项 |
| `.github/workflows/build.yml` | 出固件的 CI（五层缓存） |
| `.github/workflows/sync-upstream.yml` | 上游同步 CI（镜像快进 + 合并 + 两道闸门） |

### 修改

| 路径 | 改了什么 | 为什么 |
|---|---|---|
| `target/linux/mediatek/image/filogic.mk` | 追加 `openfi,6c` 设备定义 | 让目标系统认识这块板子 |
| `target/linux/mediatek/filogic/base-files/etc/board.d/01_leds` | 追加灯名映射；修掉上游给本板抄错的条目 | 上游引用了本板不存在的 `green:lan` / `eth0`，灯不会亮 |
| `target/linux/mediatek/filogic/base-files/etc/board.d/02_network` | 追加板级网络（WAN/LAN 映射） | 默认网络配置 |
| `target/linux/mediatek/files-6.12/.../mtk_hnat/hnat.c`、`hnat.h`、`hnat_debugfs.c`、`hnat_nf_hook.c` | 修 NULL 解引用、越界写、double free、锁与 probe 错误路径 | 上游这几处会崩；修复对通用设备也成立 |
| `feeds.conf.default` | 末尾追加 `5gmodem` feed | 5G 面板改用 `luci-app-5gmodem`（官方五个 feed 未动） |
| `README.md`、`ATTRIBUTION.md` | 重写为当前分支模型 | 原来的快照/补丁式说明已过时 |

### 不再包含

早期方向里的自研 `luci-app-openfi-modem` / `luci-app-openfi-sms` 已从产品线移除：
面板、短信、锁频、温度遥测全部交给 `5gmodem`，树里只留风扇包和公共库。

**除此之外，`target/`（除上表）、`package/`（除 `package/openfi/`）、`include/`、
`toolchain/`、`feeds.conf.default` 的官方部分，全部保持上游原样。**

## 四、许可

- 本仓库的 `COPYING` / `LICENSES/` 原样继承上游（**GPL-2.0**）。
- 底座（ImmortalWrt / OpenWrt / MTK OpenWrt Feeds / 上游 rebase 树）的版权
  归各自作者，许可随各自声明。
- **本仓库新增的部分**（设备树、`package/openfi/`、`scripts/openfi/`、
  `files/`、CI 配置）由本仓库作者编写，同样以 **GPL-2.0** 发布 —— 与底座一致，
  不引入额外限制。
- 闭源组件（MTK 的 `mt_wifi` 驱动等）不在这里重新分发，而是由
  `feeds` 在构建时从各自的来源拉取。

## 五、如果要向上游反馈

我们只加了设备支持，所以：

- **设备相关的问题**（这块板子、风扇包、出场配置）→ 提到本仓库。
- **底座 / 通用问题**（内核、无线驱动、其他机型）→ 提到上游
  [`chasey-dev/immortalwrt-mt798x-rebase`](https://github.com/chasey-dev/immortalwrt-mt798x-rebase)。
- **想向上游提修复**：从 `main`（上游镜像）拉一个只含目标修复的分支，
  向 `chasey-dev:25.12` 开 PR —— 不要把 `product` 直接开 PR，那上面有我们
  固件特有的东西。

请不要把我们加的设备支持当成上游的行为。

---

## 六、怎么在这个仓库上干活

本仓库是 [`chasey-dev/immortalwrt-mt798x-rebase`](https://github.com/chasey-dev/immortalwrt-mt798x-rebase)
的 fork，两条分支分工明确：

| 分支 | 用途 | 规则 |
|---|---|---|
| `main` | 纯上游镜像 | **只由 sync workflow 快进**；规则集禁止 force-push 和删除 |
| `product` | 我们的树（默认分支） | 日常开发都在这里；推上去会自动触发构建 |

**为什么这样分**：底座和「我们加的东西」混在一起，以后同步上游就只能整棵树重抄，
也看不出自己改了什么。分开之后，同步就是一次 merge，我们的改动永远是一个干净的 diff。

### 同步上游（平时是自动的）

`.github/workflows/sync-upstream.yml` 每天 **UTC 20:00（北京 04:00）** 跑：

1. 上游 `25.12` → `main` 快进（上游不是 `main` 的后代就报错停下）；
2. `main` merge 进 `product`；
3. **复核闸门**：上游动到我们改过的 8 个上游文件（`hnat*`、设备树、`filogic.mk`、
   `01_leds`、`02_network`）时开 issue，附 diff，等人过一眼；
4. **构建闸门**：同步成功后自动跑一次 `build.yml`。

手工做的话：

```sh
git fetch origin main
git checkout product
git merge origin/main        # 冲突在这里解
git push origin product
```

### 冲突了怎么办

workflow 遇到冲突会**一个字节都不推**，只开一个 `上游同步冲突` issue，列出冲突文件。
按 issue 里的命令本地解完、推 `product`，再手动跑一次 `sync-upstream` 即可。

冲突通常只落在设备树 / `filogic.mk` / `hnat` 这几个文件上 —— 因为只有它们同时是
「上游的文件」和「我们改过的文件」。

### 构建

- Actions → `build` → *Run workflow*；勾 `publish` 才会建 Release。
- 推 `product` 且改动涉及设备 / 包 / 脚本 / 内核补丁时自动触发。
- 每周一次保活构建（UTC 周日 18:00），顺便防止 GitHub 的缓存 7 天不访问被清掉。
- CI 有五层缓存（dl / feeds / 工具链 / 内核与包编译目录 / ccache），
  第一次约 60~90 分钟，之后 15~25 分钟。
- 换分支名的话，记得把 `build.yml` 里的 `push.branches` 一起改掉。