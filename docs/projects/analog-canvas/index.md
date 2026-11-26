# Analog Canvas

- 上游仓库：[cascode-ai/analog-canvas](https://github.com/cascode-ai/analog-canvas)
- 核验日期：2026-11-26；上游版本：[d10f572](https://github.com/cascode-ai/analog-canvas/commit/d10f572eac20ea8b2673310107e0ac44521f5952)
- 许可证：[AGPL-3.0-only](https://github.com/cascode-ai/analog-canvas/blob/d10f572eac20ea8b2673310107e0ac44521f5952/LICENSE.md)（上游 README 声明；第三方资源另有许可）
- 类型：AI 辅助模拟电路设计 / 原理图自动化 / EDA MCP / 规格验证

本页基于上游 README、目录及 Agent、CLI、仿真和 EDA 导入文档的阅读；尚未安装或运行工具，不构成复现证据。学习状态和下一步只在[项目索引](../../catalog.md)维护。

## 为什么学习

Analog Canvas 是有电气连接语义的原理图编辑平台，不只是绘图工具。其结构化电路模型、MCP/CLI、确定性网表导出、仿真规格判定及 Virtuoso 导入链路，适合研究如何让 LLM 参与模拟电路设计并提供可审查的结果。

学习目标：

- 理解器件、引脚、Net、Cell、参数与原理图几何之间的关系，以及 Agent 如何可靠操作电路。
- 学习从规格、Testbench 到仿真测量、判定和证据归档的闭环。
- 验证 Canvas 到 Virtuoso 的原生 schematic 导入及电气一致性；探索生成学习笔记用的 SVG 和网表。

前置知识：MOS 共源放大器、五管 OTA、小信号分析、SPICE、基本 Node.js 和 MCP 使用；Virtuoso 阶段另需已授权的 EDA、PDK 和 Bridge 会话。

## 项目拆解

| 模块 | 学习内容及证据 |
| --- | --- |
| 电路数据与编辑 | [README](https://github.com/cascode-ai/analog-canvas/blob/d10f572eac20ea8b2673310107e0ac44521f5952/README.md)、[model](https://github.com/cascode-ai/analog-canvas/tree/d10f572eac20ea8b2673310107e0ac44521f5952/packages/model)、[edit-engine](https://github.com/cascode-ai/analog-canvas/tree/d10f572eac20ea8b2673310107e0ac44521f5952/packages/edit-engine)：结构化数据、事务、连接关系 |
| Agent | [Agent 文档](https://github.com/cascode-ai/analog-canvas/blob/d10f572eac20ea8b2673310107e0ac44521f5952/docs/agent/README.md)、[MCP Tools](https://github.com/cascode-ai/analog-canvas/blob/d10f572eac20ea8b2673310107e0ac44521f5952/docs/agent/mcp/tools.md)：inspect、place、wire、verify、render；原子化操作与版本冲突恢复 |
| CLI | [Command line](https://github.com/cascode-ai/analog-canvas/blob/d10f572eac20ea8b2673310107e0ac44521f5952/docs/user/command-line.md)：本地 Project、JSON actions、SVG/PNG 与 SPICE/Spectre 导出 |
| 仿真 | [Simulation](https://github.com/cascode-ai/analog-canvas/blob/d10f572eac20ea8b2673310107e0ac44521f5952/docs/user/analog-simulation.md)、[Specs](https://github.com/cascode-ai/analog-canvas/blob/d10f572eac20ea8b2673310107e0ac44521f5952/docs/agent/simulation-specs.md)：测量与验收分离、缺失测量不判通过、快照与运行证据 |
| EDA 导入 | [EDA Tools](https://github.com/cascode-ai/analog-canvas/blob/d10f572eac20ea8b2673310107e0ac44521f5952/docs/eda-import.md)：通过 process map 生成 Bridge manifest 并调用 virtuoso-bridge-lite |
| 电路资源 | [Community Gallery](https://analog-canvas.tokenzhang.com/)：拓扑和结构化数据；不等同于经完整验证的芯片项目 |

## 最小实验：无 PDK 共源放大器

先不引入 Agent 或商业 EDA，运行上游 CLI 的 common-source 示例，再改变一个电阻参数观察网表变化。此实验验证结构化编辑与导出能力，不验证实际晶体管性能。

依赖：Node.js 24+、Git、可访问 GitHub Releases 的网络；独立 workspace。参考上述 CLI 指南，使用固定源码版本：

```bash
git clone https://github.com/cascode-ai/analog-canvas.git
cd analog-canvas
git checkout --detach d10f572eac20ea8b2673310107e0ac44521f5952
node scripts/canvas-local.mjs help
```

CLI 首次使用会下载 MCP release；实验前应核验兼容 release 并通过 `--version` 或 `ANALOG_CANVAS_VERSION` 固定版本，记录实际版本及校验结果。以下为在独立实验目录中执行的命令模板，先将脚本和 [common-source actions](https://github.com/cascode-ai/analog-canvas/blob/d10f572eac20ea8b2673310107e0ac44521f5952/docs/user/examples/common-source/place.json) 从固定 checkout 放到实验目录（或者使用绝对路径）：

```bash
node canvas-local.mjs new cs --name "Common-source amplifier"
node canvas-local.mjs apply cs place.json wire.json labels.json
node canvas-local.mjs png cs
node canvas-local.mjs svg cs
node canvas-local.mjs netlist cs
```

验收条件：

- 生成 `project.icproj.json`、`schematic.png`、`schematic.svg` 和 `netlist.sp`；记录命令、版本、诊断。
- 根据示例人工核对 MOS 与电阻的端点、网名和参数；检查图形与网表一致。必要时用 CLI verify 对照预期网表。
- 修改电阻值后，导出网表应体现该变化，连接拓扑不变；保存前后 diff。对同一快照重复导出并检查网表一致性。
- 不把原理图正确或导出成功当作仿真验证通过。

## 后续学习路线

1. **Agent 操作**：用公开示例试验 MCP inspect/place/wire/verify/render，观察事务失败、恢复与人工审查；固定 MCP 版本。
2. **SKY130 OTA**：使用 [ngspice OTA 示例](https://github.com/cascode-ai/analog-canvas/tree/d10f572eac20ea8b2673310107e0ac44521f5952/netlists/ngspice-ota-qualification)；先运行原始示例，再定义增益、带宽、功耗等测量和规格，核对 raw、CSV、日志、输入快照和判定。指标值依据实验确定，不预设已通过。
3. **Canvas → Virtuoso**：在独立测试库中配置目标 PDK 的 master、pin、parameter 映射，生成 Bridge manifest，导入并 readback；核对器件、参数、连线与 Spectre 网表，再做相同模型、偏置及 testbench 下的 OP/AC 对比。拓扑一致与电气等效分别验收。不要将商业 PDK、私有网表或凭据上传到在线服务。
4. **总结复用**：评估是否需要为 ai-eda-lab 增加可选的项目级 MCP 配置；为 analog-design-bench 借鉴规格及结果证据设计，为 mkdocs_why 评估 SVG 原理图生成。均在独立任务中决定，不在本仓库实现集成框架。

## 局限和待核验

- 本地 CLI 支持绘图及导出；离线 Windows 桌面预览不提供 Agent 和仿真服务。浏览器 SKY130 ngspice 可供普通用户本地运行；其他托管仿真环境存在访问限制。
- Virtuoso 导入需要真实 Bridge 环境、PDK、目标库和映射；上游文档说明原生验收仅覆盖少量 Cell，并未证明电气等效性。尚未验证与个人 IC251 及 fork 的兼容性。
- 未发现 ADS 原生导入适配器；不是替代 Virtuoso 或 ADS 的完整商业 EDA。
- AGPL 源码复用需单独审查许可；本仓库仅记录自己的解释和上游链接。

## 学习记录与总结

尚无实际实验笔记。执行最小实验后再新增独立笔记，记录实测结果、失败原因与可复用结论。
