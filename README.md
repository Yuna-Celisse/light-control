# light-control

基于 KiCad 10 的 fNIRS 光学采集与灯光控制板工程。

当前硬件包含 ESP32-S3FN8、USB-C 与 ESD 保护、3.3 V 降压和 1.8 V 电源、ADS1115、双 AS7341 光谱传感器，以及板载双路光电接收与双波长 LED 驱动电路。

## 打开工程

1. 安装 KiCad 10。本次检查使用 KiCad 10.0.7。
2. 克隆仓库并打开 `light-control.kicad_pro`。
3. 通过项目管理器打开原理图和 PCB。
4. 安装 KiCad 标准符号及封装库。项目自定义库通过 `${KIPRJMOD}` 相对路径加载。

## 文件结构

|路径|内容|
|---|---|
|`light-control.kicad_pro`|KiCad 项目|
|`light-control.kicad_sch`|顶层原理图|
|`01_*.kicad_sch` 至 `10_*.kicad_sch`|十张功能子页|
|`light-control.kicad_pcb`|四层 PCB 布局与布线|
|`light-control.kicad_dru`|自定义设计规则|
|`FNIRS.pretty`、`Lights_Edge.pretty`|自定义封装|
|`FNIRS.kicad_sym`、`light-control-custom.kicad_sym`|自定义符号|
|`BOM/20261029`|2026-10-29 导出的历史 BOM 与明细|
|`docs/review-2026-11-10.md`|本次修复、证据和待办|
|`docs/checks`|发布前 ERC、DRC 和源文件哈希|

## 当前验证状态

2026-11-10 针对仓库中的已保存文件重新运行检查。

|检查|结果|
|---|---|
|KiCad DRC|0 errors、0 warnings、0 unconnected、0 schematic parity|
|KiCad ERC|0 errors、0 warnings|

已保存板具有 166 个封装、115 个网络、1470 条走线和 168 个过孔。复核记录包含晶振下方信号绕行、降压输入去耦支路缩短及 U101 输出反馈支路缩短。

用户指出编辑器截图仍显示旧走线。IPC 回读与已保存板数据包含上述修改，但截图仍显示 1467 条走线和 166 个过孔，界面差异尚待查明。后续复核应结合仓库版本、实际打开路径和编辑器画面逐项确认。

USB 阻抗、晶体具体 MPN 与负载/起振裕量、TIA 稳定性与噪声、电源瞬态、LED 驱动电流和温升仍需验证。完整电气设计审查尚有待办。历史 BOM 包含待确认的采购型号。

详细说明见 [复核记录](docs/review-2026-11-10.md)。
