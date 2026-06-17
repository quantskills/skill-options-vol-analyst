# 📈 期权波动率分析 Skill

**简体中文** | [English](README.en.md)

> 这个品种的期权波动率贵不贵？—— 期权链快照、隐含波动率 vs 历史波动率、IV 历史分位、期限结构、偏度/微笑、波动率溢价，一次性给出可追溯的中文波动率诊断报告。

创建者 / 维护者：[`abgyjaguo`](https://github.com/abgyjaguo)

<p align="center">
  <img alt="analysis modes" src="https://img.shields.io/badge/analysis_modes-5-blue">
  <img alt="iv percentile window" src="https://img.shields.io/badge/IV_percentile_window-252d_default-brightgreen">
  <img alt="hv windows" src="https://img.shields.io/badge/HV_windows-20%20%7C%2060%20%7C%20120d-orange">
  <img alt="report sections" src="https://img.shields.io/badge/report_sections-8-9cf">
  <img alt="data source" src="https://img.shields.io/badge/data-Pandadata-ff69b4">
  <img alt="requires" src="https://img.shields.io/badge/requires-pandadata--api-7c3aed">
</p>

---

## 📖 这是什么

`options-vol-analyst` 是一个 **Agent Skill**：给它一个期权品种、标的或具体合约，它会规划出可追溯的 Pandadata 数据方案，产出中文波动率分析报告 —— 聚焦**波动率环境的诊断**，而不是下单指令。

它最核心的纪律是**先验数据、再算指标、最后才解读**：

- 合约代码、标的代码、交易所、字段、波动率窗口一律从 Pandadata 元数据确认，不发明；
- IV/HV 单位先归一化（`0.24` 还是 `24`？），不同来源混算前先统一成百分比并注明换算；
- 剩余到期日 **< 7 个自然日**的合约标记为"临期噪声"，不混入正常期限结构。

> 数据契约一律来自姊妹技能 [`pandadata-api`](https://github.com/quantskills/skill-pandadata-api)：每次真实调用前先核对方法文档。

---

## ⚡ 分析流水线

```mermaid
flowchart LR
    A["💬 目标归一化<br/>品种 / 标的 / 合约 / 全市场"] --> B["🗂️ 确认合约范围<br/>get_option_underlying_detail<br/>get_option_detail"]
    B --> C["🧪 小范围冒烟测试<br/>单合约·单到期·短窗口"]
    C --> D["📅 日期与期限对齐<br/>同一交易日比对 · 临期合约标记"]
    D --> E["🧮 波动率诊断<br/>IV−HV溢价 · IV分位<br/>期限结构斜率 · 偏度"]
    E --> F["📝 中文波动率报告<br/>8 个固定章节 · 全程可溯源"]

    style A fill:#e3f2fd,stroke:#1976d2
    style E fill:#fff3e0,stroke:#ef6c00
    style F fill:#e8f5e9,stroke:#388e3c
```

---

## 🗂️ 接口映射

| 需求 | 主要接口 |
|---|---|
| 标的 / 期权品种范围 | `get_option_underlying_detail` · `get_option_detail` |
| 期权链最新或历史行情快照 | `get_option_daily` |
| 隐含波动率时间序列 / 截面 | `get_option_implied_volatility` |
| 标的历史波动率 | `get_option_underlying_volatility` |
| 标的收盘 / 收益确认 | `get_index_daily` · `get_stock_daily` · `get_future_daily`（按标的类型） |

---

## 🧮 五种分析模式 × 核心计算

| 模式 | 输出 |
|---|---|
| ⚡ **快速快照** | 最新期权链广度、活跃到期月、ATM 区域、IV 总体水平、IV−HV 溢价、一句话注意事项 |
| 📊 **IV 分位** | 当前 IV 在自身历史中的分位，标注样本窗口、分位算法、剔除的失效/低流动性样本、最新数据日 |
| 📅 **期限结构** | 按到期月 × 可比 moneyness 分组，判断近月 IV 相对远月偏贵/偏便宜 |
| 🙂 **偏度/微笑** | 同一到期内按行权价或 delta 代理对比 put 侧与 call 侧 IV，不过度解读低流动性两翼 |
| 📋 **完整报告** | 以上全部 + 表格、可选图表、方法附录、研究免责声明 |

核心公式（正式报告中会简要列出）：

- **剩余天数** = `到期日 − 交易日`（自然日），`< 7` 天标记为临期噪声；
- **IV−HV 溢价** = 单位归一化后的 `IV − HV`（波动率点）；
- **IV/HV 比** = 仅当 HV > 0 且窗口明确时计算；
- **IV 分位** = 剔除空值/失效/无成交样本后，当前 IV 在历史样本中的百分位（默认 252 个交易日，短历史合约用 60/120 日并标注）；
- **期限结构斜率** = 远月 ATM IV − 近月 ATM IV（可比 moneyness）；
- **偏度代理** = 距 ATM 可比距离的 put 侧 IV − call 侧 IV，或低行权价 IV − 高行权价 IV。

### 流动性与质量过滤

- 选代表性行权价时优先非零成交量/持仓量的合约；
- 深度虚值两翼、零成交行、过期报价、临期合约视为低置信证据；
- 画微笑曲线不混用到期月，除非明确标注为跨到期比较；
- 空结果不等于 API 故障 —— 先排查日期可用性、代码格式、上市状态、节假日与凭证配置。

---

## 🚀 快速开始

### 1️⃣ 安装（与 pandadata-api 一起）

```bash
# Claude Code（全局）
cp -r skill-pandadata-api       ~/.claude/skills/pandadata-api
cp -r skill-options-vol-analyst ~/.claude/skills/options-vol-analyst

# Codex（全局，推荐 Agent Skills 标准目录）
mkdir -p ~/.agents/skills
cp -r skill-pandadata-api       ~/.agents/skills/pandadata-api
cp -r skill-options-vol-analyst ~/.agents/skills/options-vol-analyst

# Cursor（项目级）
mkdir -p .cursor/skills
cp -r skill-pandadata-api       .cursor/skills/pandadata-api
cp -r skill-options-vol-analyst .cursor/skills/options-vol-analyst
```

### 2️⃣ 直接用自然语言提问

```text
分析一下50ETF期权的波动率环境
当前IV处于历史什么分位？和历史波动率比贵不贵？
看看这个品种的期权期限结构和偏度
给我一份完整的期权波动率分析报告
```

### 3️⃣ 完整报告结构（固定 8 章）

```
结论摘要 → 标的与合约范围 → 期权链快照 → IV vs HV
→ 期限结构 → 偏度/微笑 → 数据说明 → 研究结论
```

期权链证据表列：`到期月 | 剩余天数 | 行权价/moneyness | Call IV | Put IV | 成交量 | 持仓量 | IV−HV溢价 | 备注`。

---

## 📦 目录结构

```
options-vol-analyst/
├── SKILL.md                       # 技能入口：核心规则、工作流、接口映射、分析模式、输出标准
├── references/
│   └── analysis-playbook.md       # 📒 数据方案、默认窗口、单位检查、计算公式、流动性过滤、报告结构
└── agents/
    ├── openai.yaml                # OpenAI/Codex 适配
    ├── cursor-rule.mdc            # Cursor 项目规则适配
    └── portable-loader.md         # 无原生 skill 发现能力的 Agent 通用加载器
```

### 跨 Agent 使用

| 运行时 | 方式 |
|---|---|
| Claude Code / Codex / Hermes / OpenClaw | 完整文件夹装入各自 skill 根目录，直接加载（`$options-vol-analyst`） |
| Cursor | `agents/cursor-rule.mdc` 作项目规则，完整文件夹放 `.cursor/skills/options-vol-analyst` |
| 其他 Agent | 粘贴 `agents/portable-loader.md` 作加载提示词 |

> 安装时保持 `SKILL.md`、`agents/`、`references/` 完整 —— 完整报告依赖 references 里的 playbook。

---

## 📐 核心约束

| 约束 | 说明 |
|---|---|
| 🧾 先查契约 | 每次真实调用与契约检查都经 `pandadata-api`，先看方法文档再写代码 |
| 🚫 不发明标识 | 合约代码、标的代码、交易所、字段、窗口、交易日一律从 Pandadata 元数据确认 |
| 📏 单位归一化 | IV/HV 单位（小数 vs 百分数）先统一再计算，并注明换算 |
| 📅 同日对齐 | 期权链、IV、标的收盘、HV 在同一交易日比对；临期合约单独标记 |
| ⚖️ 三层分离 | 数据事实、衍生指标、解读分开标注；公式与窗口写明 |
| 🗣️ 只诊断不下单 | 用"显示 / 可能提示 / 需要验证 / 受流动性影响"措辞；策略落地交给策略类技能 |

---

## ⚠️ 免责声明

本报告基于公开数据与规则化分析生成，仅供研究参考，不构成任何投资建议。

## 📄 许可证

本项目以 GNU General Public License v3.0 发布，SPDX 标识为 `GPL-3.0-only`。完整文本见 [LICENSE](LICENSE)。

## 🐼 PandaAI / QUANTSKILLS 社群

<div align="center">
  <img src="https://raw.githubusercontent.com/quantskills/.github/main/profile/assets/pandaai-community-qr.jpg" alt="PandaAI 社群二维码" width="220">
  <br>
  <sub>扫码加入 PandaAI 社群，交流 QUANTSKILLS 技能、Agent 工作流与量化研究实践。</sub>
</div>
