# Analog IC Exploration

面向模拟/RF IC 工程师的开源项目学习实验室：发现、拆解、复现和总结可复用的电路设计与 AI/EDA 方法。

- [项目索引与学习状态](docs/catalog.md) — 唯一状态入口
- [学习流程](docs/workflow.md)
- [学习笔记入口](docs/notes/index.md)
- [新增项目/笔记模板](templates/)

## 在线浏览

网站地址（首次部署成功后生效）：https://why250.github.io/analog-ic-exploration/

维护者首次启用：打开仓库 Settings → Pages → Build and deployment → Source，选择 GitHub Actions。合并到 main 后自动构建并部署；也可在 Actions → Documentation 中手动运行。PR 只构建检查，不部署。构建或部署失败时查看 Actions 日志。

## 本地浏览

直接使用 GitHub 阅读 Markdown，或安装 Zensical（推荐在虚拟环境中）：

```bash
python -m pip install -r requirements.txt
zensical serve
```

浏览 http://localhost:8000 。`zensical build` 生成静态站点。新增学习页面时，维护 `zensical.toml` 的导航及项目页面中的链接。

## 范围与边界

聚焦公开的模拟/RF/混合信号电路、LLM/Agent 设计方法和可复用 EDA 工具。这里记录学习过程，不承担正式芯片设计验收或 EDA 环境配置。未经验证的候选与实测结果分开。禁止提交公司私有 PDK、网表、许可凭据及非公开设计数据。上游源码默认不复制；如需引用或复用，记录许可与 commit SHA。
