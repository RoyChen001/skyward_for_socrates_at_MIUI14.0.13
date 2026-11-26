![Skyward Recovery](assets/hero.jpg)

# Skyward

[English](README.md) | 简体中文

Skyward 是使用 Rust 编写的 recovery，从零开始编写。

> [!NOTE]
> **REDMI K60 Pro（`socrates`）— MIUI 14.0.13（Android 13）适配分支**
>
> 本仓库基于 [kirklin/skyward](https://github.com/kirklin/skyward) 修改，专为运行 **MIUI 14.0.13（`V14.0.13.0.TMKCNXM`，Android 13）** 的 **REDMI K60 Pro（`socrates`）** 适配构建，已实机验证**触控、Wi-Fi 与 `/data` 分区密码解密**均正常工作：
> - 将 `devices/socrates.toml` 切换为官方 `V14.0.13.0.TMKCNXM` 原厂 recovery 底包，`os_version_prop` 改为 `ro.build.version.incremental`，移除 MIUI 14 驱动不存在的 `touch_thp_breakline_mode` / `touch_thp_breakline_result` 节点，并在 `[wlan].libraries` 中补充 `/vendor/lib64/vndk`。
> - 在 `crates/skyward/src/secure.rs` 中增加 `com.android.vndk.current` APEX 回退挂载及目录形式 APEX 兼容。
> - 在 `crates/skyward/src/keys.rs` 中纳入 `system` 分区以识别 Android 13 的 `hwservicemanager`，在 `crates/skyward-programs/src/secure.rs` 中排除 `/system/bin/gatekeeperd` 避免与 Gatekeeper HAL 冲突，并在 `crates/skyward-crypto/src/keystore.rs` 中放行 Android 13 密钥目录下的 `none` / `nopassword` `stretching` 文件。
> - 将 `.github/workflows/build.yml` 设为默认仅构建 `socrates`。
> - **请勿**在 Skyward 设置中使用“Skyward 在线更新”，上游官方 Release 基于 HyperOS 3/4 构建，刷入后会与 MIUI 14.0.13 不兼容。

<p align="center">
  <img src="assets/home.png" width="30%" alt="首页">
  <img src="assets/settings-dark.png" width="30%" alt="深色模式下的设置">
  <img src="assets/time-zone.png" width="30%" alt="时区">
</p>

## 功能

### 双系统

![一台手机可以安装两个系统](slides/zh-CN/out/01-dual-boot.png)

![切换系统只改写两个分区](slides/zh-CN/out/02-dual-boot-partitions.png)

第二系统的安装包格式和启动过程见下文的[双系统的工作方式](#双系统的工作方式)。

### 解密

![用锁屏图案或密码解密 data 分区](slides/zh-CN/out/03-decrypt.png)

### 安装

![从三个来源安装四种安装包](slides/zh-CN/out/04-install.png)

刷机包的安装脚本可以使用 `7za` 和 `zstd`，支持刷入 super 镜像使用 zstd 压缩的第三方 ROM；`unzip -p` 边解压边输出镜像。安装脚本还可以使用 `avbctl` 关闭验证启动的校验，使用 `resetprop` 读取和设置属性。

分区镜像按内容识别：boot、init_boot、vendor_boot、recovery、dtbo、vbmeta 和整个 super 的镜像各自刷入对应的分区；包含 erofs、ext4 或 f2fs 文件系统的镜像可以刷入当前槽位在 super 中的任何一个逻辑分区，刷入前这个逻辑分区会调整为镜像的大小，和 fastbootd 的做法相同。内容无法识别的镜像按文件名刷入同名的分区，例如 `modem.img` 刷入 modem，这个分区必须在两个槽位各有一份，或者是 super 中的逻辑分区。sparse 格式的镜像在刷入时展开。系统更新尚未合并的时候，暂不刷入 super 和其中的逻辑分区。

### 在线更新

![系统和 Skyward 都能在线更新](slides/zh-CN/out/05-updates.png)

### 备份和恢复

![系统、应用、照片和 IMEI 都可以备份](slides/zh-CN/out/06-backup.png)

### 文件管理

![在 Skyward 中管理手机里的文件](slides/zh-CN/out/07-files.png)

### 终端

![手机上就能运行 root shell](slides/zh-CN/out/08-terminal.png)

### 连接电脑

![用数据线连接电脑，传文件和刷写分区](slides/zh-CN/out/09-computer.png)

### 清除数据

![每一项都写明会清除哪些内容](slides/zh-CN/out/10-wipe.png)

### 替换原厂 recovery

![系统发来的请求，Skyward 照常处理](slides/zh-CN/out/11-requests.png)

### 搜索

![输入拼音首字母，就能找到功能](slides/zh-CN/out/12-search.png)

### 高级

![系统出了问题，可以先在这里修复](slides/zh-CN/out/13-tools.png)

### 关于本机

![电池剩余容量和充电循环都能查看](slides/zh-CN/out/14-specs.png)

### 其他功能

- 重启到系统、recovery、fastboot、bootloader 和 EDL，切换启动槽位
- 浅色和深色模式
- 15 种语言：简体中文、繁體中文、English、Türkçe、Русский、Українська、Español、Português (Brasil)、Bahasa Indonesia、Tiếng Việt、Deutsch、Français、Italiano、Polski 和日本語。系统语言在这些语言之中时，Skyward 启动后使用系统语言，其他情况使用英文

### 没有系统时

![super 中没有系统，也能装回一个系统](slides/zh-CN/out/15-no-system.png)

## 支持的机型

| 机型 | 代号 | 测试的系统版本 |
| --- | --- | --- |
| REDMI K60 Pro | `socrates` | V14.0.13.0.TMKCNXM（Android 13） |

## 安装 Skyward

Skyward 需要已解锁的 bootloader。从 [Releases](https://github.com/kirklin/skyward/releases) 下载对应机型的 zip，每个文件都标有机型和代号，例如 Xiaomi 17 Ultra (nezha)。使用 `SHA256SUMS` 核对并解压之后，在手机处于 fastboot 模式的时候，把其中的镜像刷入当前运行槽位的 recovery 分区：

```sh
unzip skyward-<commit>-socrates-V14.0.13.0.TMKCNXM.zip
fastboot flash recovery_a skyward-<commit>-socrates-V14.0.13.0.TMKCNXM.img
# 可选：同时刷入 B 槽
# fastboot flash recovery_b skyward-<commit>-socrates-V14.0.13.0.TMKCNXM.img
fastboot reboot recovery
```

> [!TIP]
> `socrates` 的 recovery 镜像为纯 ramdisk（不含内核），请勿使用 `fastboot boot`，需直接刷入 `recovery_a`（或 `recovery`）分区后执行 `fastboot reboot recovery`。

之后连接 WLAN，就可以在“设置”的“Skyward 更新”页面更新到最新版本，也可以在 Skyward 自己的“安装”页面安装解压出来的新版镜像。打开 **自动重新刷入 Skyward** 之后，系统更新或者刷机包覆盖 recovery 分区的时候，Skyward 会重新刷回这些分区。

Skyward 中的 adb 是 root shell，手机上无需授权：手机处于 recovery 的时候，任何人拿到手机和数据线都能访问其中的存储。


## 构建

Skyward 使用 `rust-toolchain.toml` 指定版本的 Rust 和它的 `aarch64-linux-android` 目标、Android NDK r29 和 [cargo-ndk](https://github.com/bbqsrc/cargo-ndk) 构建。recovery 服务针对手机构建，打包工具针对电脑构建：

```sh
cargo ndk -t arm64-v8a --platform 30 build --release -p skyward
cargo build --release -p skyward-pack
```

第二系统启动时运行的第一个程序在任何动态库可用之前运行，因此构建为静态程序：

```sh
RUSTFLAGS="-C target-feature=+crt-static" cargo ndk -t arm64-v8a --platform 30 build --release -p skyward-system-init --target-dir target/system-init
```

打包工具从小米服务器上这个版本的完整更新包中取出原厂 recovery 分区，只读取这个分区所在的部分，并且按照设备配置文件核对 SHA-256：

```sh
target/release/skyward-pack stock --config devices/nezha.toml --out target/stock/nezha/recovery.img
```

然后在它的基础上构建镜像：

```sh
target/release/skyward-pack recovery \
  --stock target/stock/nezha/recovery.img \
  --binary target/aarch64-linux-android/release/skyward \
  --system-init target/system-init/aarch64-linux-android/release/system-init \
  --config devices/nezha.toml \
  --font resources/fonts/MiSansVF.ttf \
  --mono resources/fonts/KirkMono-Regular.ttf \
  --mono-license resources/fonts/KirkMono-OFL.txt \
  --out target/skyward-nezha.img
```

测试和代码检查在电脑上运行；只在手机上运行的代码按照手机目标检查：

```sh
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
cargo ndk -t arm64-v8a --platform 30 clippy --workspace --all-targets -- -D warnings
```

名称以 `on_the_phone` 开头的测试会创建文件系统、挂载分区、设置加密密钥，这些操作只有手机能够完成。把一台已经进入 Skyward、数据分区已经打开的手机通过 adb 连接到电脑，`cargo ndk` 会把每个测试程序推送到手机上运行，测试逐个执行：

```sh
cargo ndk -t arm64-v8a --platform 30 test -p skyward -p skyward-block -p skyward-files -p skyward-install -p skyward-mtp -p skyward-vab on_the_phone -- --test-threads=1
```

`skyward-preview` 无需手机，在电脑上把页面绘制成 PNG 文件：每个页面完整绘制成长截图的样子，覆盖它可能处于的各种状态，另外还有触摸和动画过程中某些时刻的屏幕。`--lang` 按照地区代码选择语言，例如 `en` 或者 `zh-TW`：

```sh
cargo run --release -p skyward-preview -- target/preview --lang en
```

### 翻译

每种语言的文字是 `crates/skyward-ui/locales` 里的一个文件，以地区代码命名，例如 `tr.toml`、`pt-BR.toml`。`en.toml` 是参照文件，每条文字上方的注释说明它显示在哪里。构建时会读取全部文件，缺少某条文字，或者花括号里的名称和英文不一致，构建就会停止。

修正翻译时，直接修改对应语言文件里的文字。用 `skyward-preview --lang` 把页面按这种语言绘制出来检查效果：每绘制完一个页面，它会列出这个页面上显示不下的文字：`cut short in` 是以省略号结尾的文字，`set at 80% in` 是为了放进按键、按钮或滑动条而缩小了字号的文字，`past the edge in` 是超出屏幕的文字，`overlap in` 是互相重叠的两段文字。

## 许可证

[GPL-3.0-or-later](LICENSE)
