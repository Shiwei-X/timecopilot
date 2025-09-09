# TimeCopilot 项目分析 / Project Analysis

## 项目概述 / Project Overview

TimeCopilot 是一个开源的生成式 AI 预测代理（GenAI Forecasting Agent），它将大语言模型（LLM）与最先进的时间序列基础模型相结合。该项目旨在自动化和解释复杂的预测工作流，使时间序列分析更加易于使用，同时保持专业级的准确性。

TimeCopilot is an open-source generative AI forecasting agent that combines Large Language Models (LLMs) with state-of-the-art time series foundation models. It automates and explains complex forecasting workflows, making time series analysis more accessible while maintaining professional-grade accuracy.

## 核心特性 / Core Features

### 1. 智能模型选择 / Intelligent Model Selection
- 基于数据特征自动选择最适合的预测模型
- 解释技术决策的自然语言说明
- 支持多种模型系列：统计模型、机器学习模型、神经网络模型、基础模型

### 2. 自然语言交互 / Natural Language Interaction  
- 用自然语言提问关于预测的问题
- 获得技术分析的详细解释
- 支持领域特定的预测查询

### 3. 统一预测接口 / Unified Forecasting Interface
- 单一 API 处理多个时间序列模型
- 自动交叉验证和模型比较
- 面板数据（多序列）支持

## 项目架构 / Project Architecture

### 核心组件 / Core Components

```
timecopilot/
├── agent.py              # 主要的 AI 代理逻辑
├── forecaster.py         # 统一预测器类
├── _cli.py               # 命令行接口
├── models/               # 各种预测模型
│   ├── foundation/       # 基础模型 (Chronos, TimeGPT, TimesFM, etc.)
│   ├── stats.py          # 统计模型 (ARIMA, ETS, Theta, etc.)
│   ├── neural.py         # 神经网络模型
│   ├── ml.py             # 机器学习模型
│   └── prophet.py        # Prophet 模型
├── utils/                # 工具函数
└── gift_eval/            # 评估工具
```

### 1. Agent 模块 (`agent.py`)
- **功能**: 核心 AI 代理，使用 pydantic-ai 框架
- **职责**: 
  - 解析时间序列特征
  - 指导模型选择
  - 生成自然语言解释
  - 处理用户查询

### 2. Forecaster 模块 (`forecaster.py`)
- **功能**: 统一的多模型预测接口
- **特点**:
  - 支持多种预测模型的统一调用
  - 自动处理模型失败的回退机制
  - 提供一致的 API 接口

### 3. Models 模块
#### Foundation Models (基础模型)
- **Chronos**: Amazon 的预训练时间序列变换器
- **TimeGPT**: Nixtla 的商业时间序列模型
- **TimesFM**: Google 的时间序列基础模型
- **TIREX**: 时间序列表示学习模型
- **TOTO**: 时间序列预测模型

#### Statistical Models (统计模型)
- **AutoARIMA**: 自动 ARIMA 模型选择
- **AutoETS**: 自动指数平滑模型
- **Theta**: Theta 预测方法
- **SeasonalNaive**: 季节性简单模型

#### Neural/ML Models (神经网络/机器学习模型)
- **Prophet**: Facebook 的时间序列预测工具
- 其他神经网络和机器学习模型

### 4. CLI 模块 (`_cli.py`)
- **功能**: 命令行界面，使用 Fire 库
- **特点**:
  - 支持直接从 URL 获取数据
  - 可配置 LLM 模型
  - 支持自然语言查询

## 技术栈 / Technical Stack

### 核心依赖 / Core Dependencies
- **pydantic-ai**: AI 代理框架
- **openai**: LLM 集成
- **pandas**: 数据处理
- **fire**: CLI 框架

### 预测相关库 / Forecasting Libraries
- **neuralforecast**: 神经网络预测
- **statsforecast**: 统计预测
- **mlforecast**: 机器学习预测
- **tsfeatures**: 时间序列特征提取

### 基础模型集成 / Foundation Model Integrations
- **timecopilot-chronos-forecasting**
- **timecopilot-timesfm** 
- **timecopilot-tirex**
- **timecopilot-toto**
- **timecopilot-uni2ts**

## 使用模式 / Usage Patterns

### 1. 命令行使用 / CLI Usage
```bash
# 基础预测
uvx timecopilot forecast https://otexts.com/fpppy/data/AirPassengers.csv

# 指定 LLM 模型
uvx timecopilot forecast data.csv --llm openai:gpt-4o

# 自然语言查询
uvx timecopilot forecast data.csv --query "未来12个月预期有多少乘客？"
```

### 2. Python API 使用 / Python API Usage
```python
from timecopilot import TimeCopilot

# 初始化代理
tc = TimeCopilot(llm="openai:gpt-4o", retries=3)

# 生成预测
result = tc.forecast(df=df, freq="MS")

# 访问结果
print(result.output)  # 详细分析
print(result.fcst_df)  # 预测数据框
```

## 开发与部署 / Development & Deployment

### 项目配置 / Project Configuration
- **Python 版本**: 3.10+
- **构建系统**: Hatchling
- **版本**: 0.0.17 (预发布版)
- **许可证**: MIT

### 开发工具 / Development Tools
- **测试**: pytest
- **代码质量**: ruff (linting)
- **文档**: MkDocs
- **CI/CD**: GitHub Actions

### 文档结构 / Documentation Structure
```
docs/
├── index.md              # 主页
├── installation.md       # 安装指南
├── model-hub.md         # 模型中心
├── roadmap.md           # 发展路线图
├── contributing.md      # 贡献指南
├── api/                 # API 文档
├── examples/            # 示例
└── experiments/         # 实验
```

## 核心工作流 / Core Workflow

1. **数据输入**: 接收时间序列数据（CSV、DataFrame）
2. **特征提取**: 使用 tsfeatures 提取统计特征
3. **AI 分析**: LLM 分析特征并推荐模型
4. **模型选择**: 基于数据特征自动选择最佳模型
5. **交叉验证**: 比较多个模型的性能
6. **预测生成**: 使用选定模型生成预测
7. **结果解释**: 提供自然语言解释和分析
8. **查询回答**: 回答用户的特定问题

## 设计原则 / Design Principles

### 1. 模块化设计 / Modular Design
- 清晰的组件分离
- 可扩展的模型架构
- 统一的接口标准

### 2. AI 驱动 / AI-Driven
- LLM 指导的模型选择
- 自然语言解释
- 智能决策支持

### 3. 易用性 / Usability
- 简单的 CLI 接口
- 直观的 Python API
- 自动化的工作流

### 4. 可扩展性 / Extensibility
- 支持多种模型类型
- 可插拔的模型架构
- 开放的社区贡献

## 未来发展 / Future Development

根据项目结构和文档，TimeCopilot 正在积极发展中，关注以下方向：

1. **模型生态系统扩展**: 集成更多基础模型和传统模型
2. **性能优化**: 提高预测准确性和运行效率
3. **用户体验改进**: 增强 CLI 和 API 的易用性
4. **企业级功能**: 支持大规模部署和生产使用
5. **社区生态**: 建设活跃的开发者社区

## 总结 / Summary

TimeCopilot 是一个创新的时间序列预测项目，它成功地将传统的统计/机器学习方法与现代的大语言模型和基础模型相结合。其独特之处在于：

- **智能化**: 使用 AI 自动选择和解释模型
- **全面性**: 支持从统计模型到最新基础模型的全范围方案  
- **实用性**: 提供简洁易用的接口和丰富的功能
- **开放性**: 采用开源许可，鼓励社区贡献

该项目代表了时间序列预测领域的一个重要发展方向，将复杂的技术通过 AI 的方式变得更加易用和智能化。