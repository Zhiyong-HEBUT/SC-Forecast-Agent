# SmartSCM Agents

> 智能供应链需求预测、库存风险分析与 AI 多智能体决策系统

SmartSCM Agents 是一个面向供应链场景的端到端 AI 项目。它把 **需求预测、库存规则、多智能体分析和 Streamlit 可视化** 串成一条完整流程：从历史销量出发，预测未来需求，识别缺货风险，计算补货建议，并生成管理者可以直接阅读的中文决策报告。

这个仓库重点展示的不是单一模型，而是如何把数据、模型、业务规则和 LLM 组合成一个可运行、可解释、可继续扩展的业务应用。

## 项目亮点

- **真实时序数据流程**：基于 M5 Forecasting 数据结构，完成数据整理、特征构建与模型训练。
- **递归多步预测**：LightGBM 按天递归预测未来 14 天需求，每一步都会更新 lag 与 rolling 特征。
- **可解释库存规则**：结合当前库存、安全库存和补货提前期，输出 HIGH / MEDIUM / LOW 风险与建议补货量。
- **多智能体协作**：需求分析 Agent、库存规划 Agent 和报告 Agent 分工完成解释与管理报告生成。
- **双入口演示**：既可以通过命令行快速验证，也可以启动 Streamlit Dashboard 进行交互演示。
- **可替换组件**：预测模型、库存规则、LLM 模型和数据源都可以独立替换，方便继续做成企业内部工具。

## 系统流程

```text
历史销量 / 价格 / 日历
        │
        ▼
数据处理与特征工程
(date / lag / rolling / price)
        │
        ▼
LightGBM 需求预测
(未来 14 天递归预测)
        │
        ▼
库存决策规则
(安全库存 / 提前期 / 风险 / 补货量)
        │
        ├──────────────► Dashboard 风险看板
        │
        ▼
AI Agents
需求分析 → 库存解释 → 管理报告
```

## 当前演示数据

仓库已经包含一份处理后的演示数据和训练好的基线模型，因此安装依赖后可以直接运行演示，无需先下载完整 M5 数据集。

当前默认配置：

- 门店：`CA_1`
- 商品数量：50
- 预测周期：14 天
- 模型：LightGBM
- 模型文件：`models/baseline_lgbm_ca1.pkl`
- 销量数据：`data/processed/daily_sales.csv`
- 库存数据：`data/processed/inventory.csv`

> `inventory.csv` 中的库存参数是根据历史需求构造的演示数据，用于验证完整决策流程，不代表真实企业库存。

## 项目目录

```text
SmartSCM-Agents/
├─ data/
│  └─ processed/
│     ├─ daily_sales.csv          # 处理后的日销量数据
│     └─ inventory.csv            # 演示用库存数据
├─ models/
│  └─ baseline_lgbm_ca1.pkl       # 已训练的 LightGBM 基线模型
├─ src/
│  ├─ data_prep/
│  │  ├─ build_dataset.py         # M5 原始数据清洗与长表构建
│  │  └─ build_inventory.py       # 构造演示库存表
│  ├─ forecasting/
│  │  ├─ features.py              # 时间、lag、rolling 特征
│  │  ├─ train_baseline.py        # 模型训练与评估
│  │  └─ forecast_service.py      # 递归多步预测服务
│  ├─ inventory/
│  │  └─ rules.py                 # 库存风险与补货规则
│  ├─ agents/
│  │  ├─ base.py                  # LLM Agent 基础封装
│  │  ├─ tools.py                 # 预测与库存工具层
│  │  └─ domain_agents.py         # 需求 / 库存 / 报告 Agent
│  └─ app/
│     ├─ demo_one_item.py         # 单商品命令行演示
│     ├─ run_daily_planning.py    # 全商品风险排序
│     ├─ run_agents_planning.py   # 多智能体管理报告
│     └─ dashboard.py             # Streamlit Dashboard
├─ demo.pdf                       # 简体中文演示文档
├─ PATCH_NOTES.md                 # 预测链路修复说明
├─ requirements.txt
└─ README.md
```

## 快速开始

推荐使用 Python 3.10 或 3.11 创建独立环境。

### 1. 安装依赖

```bash
python -m venv .venv
```

Windows：

```bash
.venv\Scripts\activate
```

macOS / Linux：

```bash
source .venv/bin/activate
```

然后安装：

```bash
pip install -r requirements.txt
```

### 2. 直接运行单商品演示

```bash
python -m src.app.demo_one_item
```

程序会输出未来 14 天预测、当前库存、安全库存、预计剩余库存、风险等级和建议补货量。

### 3. 查看全商品风险清单

```bash
python -m src.app.run_daily_planning --top_n 20
```

系统会按照风险等级和预计剩余库存排序，输出优先处理的商品。

### 4. 启动 Dashboard

```bash
streamlit run src/app/dashboard.py
```

Dashboard 提供：

- 高 / 中 / 低风险商品统计
- 风险商品明细与补货建议
- 预测需求、近期需求、趋势与波动指标
- AI Agents 中文管理报告入口

## 启用 AI Agents

LLM 功能需要 OpenAI API Key。可以在项目根目录创建 `.env`：

```env
OPENAI_API_KEY=你的_API_KEY
```

然后运行：

```bash
python -m src.app.run_agents_planning --top_n 10
```

或在 Dashboard 中点击“生成今日 AI 报告”。

如果不配置 API Key，需求预测、库存风险计算和普通 Dashboard 数据展示仍然可以独立使用。

## 重新训练模型

如果你希望从原始 M5 数据重新构建数据和模型，请把以下文件放入 `data/raw/`：

```text
sales_train_validation.csv
calendar.csv
sell_prices.csv
```

M5 数据集：Kaggle - M5 Forecasting Accuracy。

然后依次执行：

```bash
python -m src.data_prep.build_dataset
python -m src.data_prep.build_inventory
python -m src.forecasting.train_baseline
```

训练完成后会生成或覆盖：

```text
models/baseline_lgbm_ca1.pkl
```

## 核心逻辑

### 需求预测

模型使用的主要特征包括：

- `dow`、`weekofyear`、`month`、`year`
- `sell_price`
- `lag_7`、`lag_14`
- `rollmean_7`、`rollmean_28`

多步预测采用递归方式：每预测一天，就把预测结果加入历史序列，再重新计算后续日期的 lag 和 rolling 特征。未来价格在演示版本中使用最近一次已知价格作为基线假设。

### 库存风险

补货提前期内的预测需求：

```text
lead_time_demand = sum(forecast[:lead_time_days])
```

预计剩余库存：

```text
projected_remaining = current_inventory - lead_time_demand
```

风险等级：

```text
LOW     : projected_remaining >= safety_stock
MEDIUM  : 0 <= projected_remaining < safety_stock
HIGH    : projected_remaining < 0
```

建议补货量：

```text
target_level = safety_stock + lead_time_demand
reorder_qty = max(0, target_level - current_inventory)
```

### 需求解释指标

为了让 Agent 的解释尽量基于数据而不是“编故事”，工具层还计算：

- 未来平均需求与最近 28 天实际平均需求的变化
- 预测期前后半段趋势
- 标准差与变异系数 CV
- 需求层级、趋势方向、波动等级

Agent Prompt 明确要求只根据这些指标进行解释，不擅自假设促销、节日或价格变化。

## 模型评估

训练脚本输出：

- RMSE
- MAE
- WMAPE
- sMAPE
- Forecast Bias

对于实际销量为 0 的情况，指标计算做了保护，避免传统 MAPE 出现异常放大。

## 可以继续扩展的方向

这个版本已经具备完整演示链路，后续可以继续做成更接近生产环境的项目，例如：

- 接入真实 ERP / WMS 库存与采购数据
- 增加供应商最小起订量、箱规、采购预算等约束
- 把安全库存改为基于服务水平和需求波动动态计算
- 增加促销、节假日、天气等外生变量
- 引入多门店、多仓库、调拨和缺货成本优化
- 增加预测回测、模型监控与数据质量监控
- 使用企业内部模型或其他兼容 LLM 替换当前 Agent 模型

## 演示文档

打开仓库中的 `demo.pdf` 可以查看简体中文项目演示说明和当前样例风险清单。

## 项目信息

- 项目名称：SmartSCM Agents
- 项目定位：智能供应链预测与补货决策演示系统
- 维护者：`<Zhiyong>`
- 仓库地址：`https://github.com/<your-account>/SmartSCM-Agents`

发布到公开仓库前，建议把上面的维护者和仓库地址替换成你自己的信息，并确认第三方数据、模型和代码所适用的许可条款。M5 数据本身请遵守其原始数据集与竞赛规则。
