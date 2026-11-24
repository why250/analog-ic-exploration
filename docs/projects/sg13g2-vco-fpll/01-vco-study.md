# 01 — VCO 源码拆解与前后仿数据分析

日期：2026-11-24。上游 commit：dd4be9d800bd5c466d709e9b814459eb9a9308cf。

## 问题

2.4 GHz VCO 如何决定频率、起振与调谐能力？版图寄生会怎样改变指标？先读独立 VCO，不从整个 PLL 开始。

## 规格和证据

[上游 README](https://github.com/Manimohan05/SG13G2_2.4GHz_LC_VCO_FPLL/blob/dd4be9d800bd5c466d709e9b814459eb9a9308cf/README.md)给出 1.2 V 电源、10 MHz 参考、2.40–2.48 GHz PLL 输出目标，150 kHz 典型环路带宽和 55° 典型相位裕度。VCO 目标 2.35–2.55 GHz。作者报告后仿相噪 −100.8 dBc/Hz（1 MHz offset）、总功耗 12.73 mW，但完整后仿闭环锁定和全 PVT 未归档；这些不是本次实测结果。README 的总体达成声明与独立后仿扫频结果存在覆盖范围差异，以具体实验条件分别判断。

## 读懂 VCO

阅读[原理图](https://github.com/Manimohan05/SG13G2_2.4GHz_LC_VCO_FPLL/blob/dd4be9d800bd5c466d709e9b814459eb9a9308cf/xschem/lc-vco/LC_VCO.sch)（已确认文件路径，尚未在 GUI 打开）与[可运行测试台](https://github.com/Manimohan05/SG13G2_2.4GHz_LC_VCO_FPLL/blob/dd4be9d800bd5c466d709e9b814459eb9a9308cf/simulations/LC_VCO_tb.spice)（已读源码）。网表中：
- MM1/2 为 NMOS 交叉耦合对，MM3/4 为 PMOS 交叉耦合对，两对为 tank 补充能量。
- MM5/6/7 形成尾电流偏置，VBGR 为参考；MM8/9 输出缓冲，MM10/11 提供对称 dummy 负载。
- CC1 固定电容，CC2/3 为受 VCTRL 控制的 varactor；差分电感为 4nH_INDUCTOR。
- 实际 EM 等效网络见[电感模型](https://github.com/Manimohan05/SG13G2_2.4GHz_LC_VCO_FPLL/blob/dd4be9d800bd5c466d709e9b814459eb9a9308cf/simulations/4nH_INDUCTOR.spice)，不是理想 L；[spice/LC_VCO.spice](https://github.com/Manimohan05/SG13G2_2.4GHz_LC_VCO_FPLL/blob/dd4be9d800bd5c466d709e9b814459eb9a9308cf/spice/LC_VCO.spice)内部为 LVS 黑盒，不能当成完整瞬态测试台。

用差分端口统一定义 L、C 和损耗时，理想频率 f = 1/(2π√(LC))。若 Ldiff = 4 nH，2.45 GHz 对应有效 Cdiff ≈ 1.055 pF。README 报告 Qdiff = 16.8；窄带并联近似 Rp ≈ QωL ≈ 1.03 kΩ。若忽略 ro、尾节点交流运动等影响，交叉耦合对的差分负电导约为 −(gmn+gmp)/2；起振需 gmn+gmp > 2/Rp ≈ 1.93 mS。实际还需计入缓冲负载、电容损耗、偏置和频率相关 EM 模型，不能将该估算当作真实起振裕度。

## 对上游归档数据做定量分析

已实际读取两份 CSV：[pre](https://github.com/Manimohan05/SG13G2_2.4GHz_LC_VCO_FPLL/blob/dd4be9d800bd5c466d709e9b814459eb9a9308cf/xschem/top-pll/simulations/vco_tuning_pre.csv)、[post](https://github.com/Manimohan05/SG13G2_2.4GHz_LC_VCO_FPLL/blob/dd4be9d800bd5c466d709e9b814459eb9a9308cf/xschem/top-pll/simulations/vco_tuning_post.csv)。以下为选取的数据（作者仿真，非本次运行）：

| VCTRL (V) | 前仿 (GHz) | 后仿 (GHz) |
|---:|---:|---:|
| 0.0 | 2.384046 | 2.325503 |
| 0.6 | 2.415955 | 2.355607 |
| 1.0 | 2.464393 | 2.401211 |
| 1.2 | 2.490582 | 2.425601 |

由 0.5/0.7 V 两点中心差分，0.6 V 附近 KVCO 前仿约 93.56 MHz/V，后仿约 87.94 MHz/V。由 f 反推有效 Cdiff（假定 Ldiff 不变）：0.6 V 时约 1.085 pF → 1.141 pF，增加约 56 fF（约 5.2%）。这是本次解析计算，不能证明真实物理寄生电容恰好是 56 fF；需要 PEX 网表和 EM 对照进一步定位。后仿最高 2.425601 GHz，距离 2.48 GHz 仍差约 54.4 MHz。

## 学到了什么

VCO 的频率取决于全 tank 有效电容，而不只是原理图上的可调电容。对于 GHz 振荡器，几十 fF 寄生即可造成不可忽略的频偏。下一步[运行独立 VCO 仿真](02-reproduction.md)，再比较自己的结果与归档 CSV。
