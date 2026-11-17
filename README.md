# light-control

基于 KiCad 10 的 fNIRS 光学采集与灯光控制板工程。

当前原理图主控为 STM32F103C8T6，包含 USB-C 与 ESD 保护、3.3 V 降压和 1.8 V 电源、ADS1115、双 AS7341 光谱传感器，以及板载双路光电接收与双波长 LED 驱动电路。光谱模块承担系统中的其他检测用途。

## 打开工程

1. 安装 KiCad 10。本次主控迁移检查使用 KiCad 10.0.8。
2. 克隆仓库并打开 `light-control.kicad_pro`。
3. 通过项目管理器打开原理图和 PCB。
4. 安装 KiCad 标准符号及封装库。项目自定义库通过 `${KIPRJMOD}` 相对路径加载。此次新注册的部分标准库仍引用本机其他工程路径，迁移环境时须重新配置库路径。

## 文件结构

|路径|内容|
|---|---|
|`light-control.kicad_pro`|KiCad 项目|
|`light-control.kicad_sch`|顶层原理图|
|`01_*.kicad_sch` 至 `10_*.kicad_sch`|十张在用功能子页，主控使用 `03_stm32f103.kicad_sch`|
|`light-control.kicad_pcb`|四层 PCB，全板布线、铺铜与器件特性隐患修复已完成，待导出制板文件|
|`light-control.kicad_dru`|自定义设计规则|
|`FNIRS.pretty`、`Lights_Edge.pretty`|自定义封装|
|`FNIRS.kicad_sym`、`light-control-custom.kicad_sym`|自定义符号|
|`docs/review-2026-11-10.md`|设计变更、隐患修复与打样复核记录|
|`docs/checks`|检查证据（`drc.json`、`erc.json`、`source-sha256.txt`），须核对生成时间与对应设计版本|

## 当前验证状态

2026-11-17 完成全板布线后的元器件特性隐患排查与最终打样复核（当前包含 164 个器件、1692 段走线、223 个过孔、3 个铺铜区，`In1.Cu` 保持完整地平面且无信号线穿越）：

- **隐患修复**：已移除 `L1` 电感本体正下方走线与过孔，将 `+3V3` 与 `/I2C0_SDA` 改道至外侧并补打西侧缝合地孔；将 `D101` B 面焊盘间穿越走线 `Net-(C117-Pad1)` 迁至 `In2.Cu`；将 `D103` courtyard 内的 3 颗过孔（`+3V3`、`/PROBE_EN`、`GND`）全部外移并重构 `/PROBE_EN` 与 `R8.2` 连接，消除所有孤立 `GND` 铜岛。
- **DRC / ERC 与可制造性**：重填铺铜并保存后，KiCad 10.0.8 实测 **DRC 0 错误、0 未连接项**（含 17 项既有警告：`silk_over_copper` ×12、`lib_footprint_mismatch` ×5，及 1 项 `Y1` 缺少 `C0 max` 字段的 `schematic_parity` 警告）；**ERC 0 错误**（27 项既有 `lib_symbol_mismatch` 警告）；`validate_for_manufacturing` 结论为 **`READY`**（可裸板打样，PCBA 贴片前建议补齐 BOM MPN、在拼板工艺边添加 Mark 点并核对 0.2 mm 过孔盖油及 USB 90 Ω 差分叠层参数）。

2026-11-11 完成十个模块内部布线、地铜补接及工程文件清理，随后补全模块间互连；SWD 使用六个长方形表面焊盘（1.5 × 3 mm、中心距 2 mm）。

2026-11-10 已将原理图主控迁移至 STM32F103C8T6，补齐 VDD、VDDA、VBAT 去耦、BOOT0/BOOT1、NRST、SWD 和 USB D+ 1.5 kΩ 上拉。两组 I²C 分别连接 ADS1115 与左光谱模块、右光谱模块。主控页使用 12 MHz HSE，PLL ×6 对应 72 MHz，USB 分频 1.5 对应 48 MHz。

晶振选用 ABM8W-12.0000MHZ-6-D1X-T3，C38/C39 初配 6 pF，寄生电容暂按 3 pF 估计，最终负载需结合完成后的 PCB 与实测校准。

模块连线整理阶段覆盖 11 张在用原理图，模块内部采用连续导线，模块之间采用网络标签。整理前后 164 个器件的型号、数值、封装与 UUID 一致，全部网络端点连接一致。逐页导线检查为 0 个悬空端点，全项目 ERC 为 0 个错误、27 个库符号差异警告。

迁移前后导出网表对比显示，主控块以外的业务模块间连接一致。删除 ESP32 专用 L2、L3、C36，新增 J2 调试接口，其余主控外围按 STM32 重新配置。

PCB 已同步最新原理图，含 164 个器件。155 个原器件的位置、角度与层完全沿用原布局，U5 沿用原中心并旋转 90°，7 颗主控外围器件做了避让微调。J2 按参考图改为靠近板边的一排长方形表面焊盘，距板边 0.6 mm，间距 2 mm，单焊盘 1.5 × 3 mm，顺序为 3V3、SWDIO、GND、SWCLK、NRST、GND。

最终 DRC/ERC 报告及核查明细见 [复核记录](docs/review-2026-11-10.md) 与 [`docs/checks/`](docs/checks)。
