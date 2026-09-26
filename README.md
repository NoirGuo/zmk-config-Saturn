# Saturn ZMK Config

Saturn 分体键盘的 [ZMK](https://zmk.dev) 固件配置。

## 硬件

- **主控**：左右各一颗 nRFMicro 1.3（nrfmicro_13）
- **左手（外设）**：4 行 6 列矩阵（row0–row3, col0–col5）+ 1 个直接引脚按键 + 1 个 EC11 编码器 + WS2812 RGB underglow（chain=1）
- **右手（中央）**：4 行 7 列矩阵（row0–row3, col6–col12）+ PAW3222 轨迹球

## 目录结构

```
├── boards/shields/saturn/
│   ├── Kconfig.defconfig        # 默认配置（右手 = central）
│   ├── Kconfig.shield           # shield 定义
│   ├── saturn.dtsi              # 公共定义（矩阵变换、编码器、物理布局）
│   ├── saturn-layouts.dtsi      # 物理布局（53 键位 + 左手编码器）
│   ├── saturn_left.overlay      # 左手：矩阵 + direct pin + 编码器 + RGB
│   ├── saturn_right.overlay     # 右手：矩阵 + 轨迹球监听
│   ├── trackball.dtsi           # PAW3222 轨迹球（SPI1）
│   ├── saturn_left.conf         # 左手配置
│   ├── saturn_right.conf        # 右手配置
│   └── saturn.zmk.yml           # shield 元数据
├── config/
│   ├── west.yml                 # ZMK DYA fork + PAW3222 驱动 + DYA 模块
│   ├── saturn.keymap            # 键位映射（6 层）
│   ├── saturn.json              # keymap-drawer 布局
│   └── saturn.conf              # 全局配置
├── build.yaml                   # 构建矩阵（GitHub Actions）
├── keymap_drawer.config.yaml    # keymap-drawer 绘图配置
└── keymap-drawer/               # 由 CI 自动生成的键位图（svg/yaml）
```

## 接线（nRF52840 GPIO，P0.x / P1.x）

| 功能 | 左手 | 右手 |
| --- | --- | --- |
| 矩阵列 | col0=P0.13, col1=P0.24, col2=P1.11, col3=P0.02, col4=P0.05, col5=P0.09 | col6=P0.13, col7=P0.24, col8=P1.11, col9=P0.02, col10=P0.05, col11=P0.09, col12=P0.10 |
| 矩阵行 | row0=P1.10, row1=P0.03, row2=P0.28, row3=P1.13 | 同左手 |
| 直接引脚 | P0.30 | — |
| 编码器 | A=P0.31, B=P0.29 | — |
| WS2812 数据 | P0.11 (SPI3 MOSI) | — |
| PAW3222 SCLK / SDIO | — | P0.30 / P0.26 (SPI1) |
| PAW3222 NCS / MOTION | — | P0.29 / P0.31 |

> 注意：WS2812 数据引脚在原表格中未列出（原为 P0.31，已让给编码器 A 相），当前暂用 **P0.11** 占位，请按实际 PCB 接线在 `saturn_left.overlay` 中修正。

## 构建

推送到 `main` 分支后 GitHub Actions 自动构建，固件在 Actions 页面下载：

- `saturn_left`（左手，外设）
- `saturn_right`（右手，中央，含 ZMK Studio RPC）
- `settings_reset`（清空设置）

## DYA Studio / ZMK Studio

- 右手固件内置 Studio 支持（USB 连接电脑后打开 <https://studio.zmk.dev> 即可在线改键、调整轨迹球参数与编码器行为）。
- 编码器使用 `zmk,behavior-runtime-sensor-rotate`，可在 Studio 中实时修改滚动/音量行为。

## 层

- **BASE**：QWERTY 主键区（左手 13 键 + 右手 13 键）
- **NUM**：数字小键盘（左手数字行 1–5 + 右手 0–9 与运算符），编码器为音量
- **NAVI**：导航（方向键、Home/End/PgUp/PgDn、F 区），编码器为滚动
- **MOUSE**：鼠标键（左/右键 + 轨迹球滚动）
- **CONF**：蓝牙（BT_SEL 0/1/2、BT_CLR）、输出切换、Studio 解锁
- 第 6 空层（备用）

## 注意事项

- **轨迹球 CPI**：`trackball.dtsi` 中 `cpi = <1600>`（PAW3222 支持 608–4826）。
- **驱动版本**：使用 `tokyo2006/zmk-driver-paw3222` 的 **v0.3** 分支（兼容 ZMK v0.3 的 `pixart,paw3222`）；其 main 分支已迁移到 v0.4 的 `xinta` 命名空间，**不要**直接引用。
- **RGB 供电**：左手启用了 `CONFIG_ZMK_EXT_POWER`（nRFMicro 1.3 的 EXT_POWER 控制引脚），若灯带不亮可检查该引脚供电。
- **键位布局**：53 个物理位置（左手 26 + 右手 27），矩阵为 4 行 8 列、右半 `row-offset = 4`。
- **NFC 引脚**：P0.09/P0.10 用作矩阵列，已开启 `CONFIG_NFCT_PINS_AS_GPIOS`。

## 本地生成键位图

```bash
pip install keymap-drawer
keymap -c keymap_drawer.config.yaml parse -z config/saturn.keymap -o keymap-drawer/saturn.yaml
keymap -c keymap_drawer.config.yaml draw -j config/saturn.json keymap-drawer/saturn.yaml -o keymap-drawer/saturn.svg
```
