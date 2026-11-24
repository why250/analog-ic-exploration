# SG13G2 2.4 GHz LC VCO / FPLL

核验日期：2026-11-24。上游版本：[dd4be9d](https://github.com/Manimohan05/SG13G2_2.4GHz_LC_VCO_FPLL/blob/dd4be9d800bd5c466d709e9b814459eb9a9308cf/README.md)。已阅读 README、CLAUDE.md、主要目录、独立 VCO 测试台及前后仿 CSV；未运行原始 EDA 仿真。公开可读，根目录未发现 LICENSE，许可未确认；不复制上游源码。

## 学习目标

理解从 PLL 规格到 LC VCO 设计、晶体管级仿真、寄生影响与自动化验证的全过程。

| 模块 | 学习重点 |
|---|---|
| VCO | LC tank、负阻起振、偏置、调谐、相位噪声 |
| CP / LF / PFD | 电流匹配、环路增益与稳定性 |
| Fractional-N divider | 平均分频比与 ΔΣ |
| EM / PEX | 电感模型、寄生引起的频偏、验证证据 |

## 逐步学习

1. [VCO 源码拆解与前后仿数据分析](01-vco-study.md)：已完成资料核验与解析计算。
2. [环境预检与最小瞬态复现实验](02-reproduction.md)：待在实际 EDA 环境执行。
3. 后续：VCTRL sweep、起振裕度、功耗和前后仿差异；再进入 CP/LF、PLL 闭环及 AI 自动化。

学习进度只在[项目索引](../../catalog.md)维护。
