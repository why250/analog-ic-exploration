# 02 — 环境预检与最小瞬态复现实验

日期：2026-11-24。实验计划；尚未执行 EDA 仿真。当前 ChatGPT 执行环境未找到 ngspice、Docker 或预期位置的 PDK。

## 选择环境

上游 [CLAUDE.md](https://github.com/Manimohan05/SG13G2_2.4GHz_LC_VCO_FPLL/blob/dd4be9d800bd5c466d709e9b814459eb9a9308cf/CLAUDE.md) 使用 [uniccass-icdesign-tools](https://github.com/unic-cass/uniccass-icdesign-tools) 容器，其中包含 Xschem、ngspice、KLayout 和 SG13G2 PDK。优先在可用 Linux 或 Windows WSL2/Docker 环境按该容器项目的 README 配置；镜像版本、启动方式需现场核实。先不安装全部工具到宿主机。

上游物理验证记录的 IHP-Open-PDK 版本为 7e53c77（不代表所有原始仿真均使用该版本）。记录实际容器镜像、ngspice 版本、PDK SHA 以及 OSDI 模型加载方式。

## 获取独立工作副本

在实际仿真环境中运行（不要把上游代码提交到学习仓库）：

```bash
git clone https://github.com/Manimohan05/SG13G2_2.4GHz_LC_VCO_FPLL.git
cd SG13G2_2.4GHz_LC_VCO_FPLL
git checkout dd4be9d800bd5c466d709e9b814459eb9a9308cf
git rev-parse HEAD
ngspice --version
printf 'PDK_ROOT=%s PDK=%s PDKPATH=%s\n' "$PDK_ROOT" "$PDK" "$PDKPATH"
```

在容器内确认 PDK_ROOT（上游示例 /foss/pdks 或 /opt/pdks）、PDK=ihp-sg13g2 以及 PDKPATH 是否指向实际工艺目录。检查 cornerMOSlv.lib、cornerMOShv.lib、cornerRES.lib、cornerCAP.lib 的路径与库 section；确认 ngspice 初始化文件能找到库和所需模型。不要未经检查修改全局设置。

## 实验 A：原始测试台

[LC_VCO_tb.spice](https://github.com/Manimohan05/SG13G2_2.4GHz_LC_VCO_FPLL/blob/dd4be9d800bd5c466d709e9b814459eb9a9308cf/simulations/LC_VCO_tb.spice) 自带晶体管级子电路和 EM 电感 include，原始 VDD=1.2 V、VBGR=0.6 V、VCTRL 由 0.4 V 在 10–100 ns 上升到 0.8 V，transient 设为 10 ps/500 ns，从 100 ns 保存。原始 .param temp=27 不一定等价于仿真器温度设置，需检查实际 TEMP。先按上游建议：

```bash
cd simulations
ngspice -b LC_VCO_tb.spice > vco-original.log 2>&1
```

该命令是上游推荐入口，不保证直接成功：控制块含 plot（无 GUI 时可能报错）、FFT、外部 Python 脚本以及相对 include。若失败先保留日志，在独立副本中仅调整必要的库路径、模型加载和无 GUI 输出方式；不要改 golden LVS 网表或静默换理想电感。确认 raw 文件存在、内容有效，不以退出码作为唯一成功证据。

## 实验 B：单点稳态频率

原始 deck 跑通后才复制一份测试台，将 VCTRL 固定 0.6 V；明确 TT、VDD=1.2 V、VBGR=0.6 V、实际 TEMP=27°C。保留 EM 模型。若理想对称电路不起振，记录无扰动结果后在 OUTp 加可追踪小初值并复测。保存 OUTp、OUTn、FOUT 与供电电流；限制最大步长、检查步长减半是否收敛。

在稳态选取至少 20 个同方向零交叉（差分波形）计算频率；检查周期稳定与幅度，而非只看 FFT 峰值。上游 0.6 V 前仿参考 2.415955 GHz，单端 OUTp 摆幅约 0.7694 Vpp。初次复现的学习验收建议：持续振荡且频率相对误差 ≤1%；若超过则记录差异，核对 PDK、温度、模型、时间步和初值。功耗比较前必须对齐作者的 core power 统计边界。

## 实际记录（待填写）

| 项目 | 实测 |
|---|---|
| 环境/PDK/版本 | 待执行 |
| 原始日志与波形 | 待执行 |
| 稳态频率/摆幅/功耗 | 待执行 |
| 差异与排查 | 待执行 |

通过单点后，再做 0–1.2 V、0.1 V 步进的独立 DC 控制点瞬态扫描；保存 CSV、脚本和误差对比。不要将连续 VCTRL ramp 的单次 FFT 当作静态调谐曲线。
