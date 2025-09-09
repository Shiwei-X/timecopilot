# TimeCopilot 使用指南 / Usage Guide

## 快速开始 / Quick Start

### 1. 最简单的使用方式 / Simplest Usage

```bash
# 使用 uvx 直接从 URL 预测
uvx timecopilot forecast https://otexts.com/fpppy/data/AirPassengers.csv
```

这个命令会：
- 自动下载数据
- 分析时间序列特征
- 选择最佳模型
- 生成预测结果
- 提供自然语言解释

### 2. 自定义 LLM 模型 / Custom LLM Model

```bash
# 使用 GPT-4o 获得更好的分析质量
uvx timecopilot forecast data.csv --llm openai:gpt-4o

# 使用其他 LLM 提供商
uvx timecopilot forecast data.csv --llm anthropic:claude-3-sonnet
```

### 3. 自然语言查询 / Natural Language Queries

```bash
# 询问特定问题
uvx timecopilot forecast data.csv \
  --llm openai:gpt-4o \
  --query "未来12个月总共预期有多少乘客？"

# 业务相关问题
uvx timecopilot forecast sales_data.csv \
  --query "下个季度我们需要准备多少库存？"
```

## Python API 使用 / Python API Usage

### 基础使用模式 / Basic Usage Pattern

```python
import pandas as pd
from timecopilot import TimeCopilot

# 加载数据
df = pd.read_csv("your_data.csv")

# 初始化 TimeCopilot
tc = TimeCopilot(
    llm="openai:gpt-4o",
    retries=3,  # 重试次数
)

# 生成预测
result = tc.forecast(
    df=df,
    freq="MS",        # 月度开始频率
    h=12,             # 预测12期
    seasonality=12,   # 12个月季节性
)

# 查看结果
print(result.output)      # 详细分析报告
print(result.fcst_df)     # 预测数据框
print(result.features_df) # 特征分析
print(result.eval_df)     # 模型评估结果
```

### 数据格式要求 / Data Format Requirements

TimeCopilot 要求输入数据包含以下列：

```python
# 必需的列
df = pd.DataFrame({
    'unique_id': ['Series1', 'Series1', ...],  # 时间序列标识符
    'ds': ['2020-01-01', '2020-02-01', ...],   # 日期列 (datetime)
    'y': [100, 105, 98, ...],                  # 目标变量 (numeric)
})

# 确保日期列为 datetime 格式
df['ds'] = pd.to_datetime(df['ds'])
```

### 高级配置 / Advanced Configuration

```python
# 使用自定义模型列表
from timecopilot.models.stats import AutoARIMA, AutoETS, Theta
from timecopilot.models.foundation import Chronos
from timecopilot import TimeCopilotForecaster

# 创建自定义预测器
custom_models = [
    AutoARIMA(),
    AutoETS(),
    Theta(),
    Chronos(repo_id="amazon/chronos-t5-small"),
]

forecaster = TimeCopilotForecaster(
    models=custom_models,
    fallback_model=AutoARIMA(),  # 回退模型
)

# 使用自定义预测器
tc = TimeCopilot(llm="openai:gpt-4o")
result = tc.forecast(df=df, forecaster=forecaster)
```

## 实际应用场景 / Real-world Use Cases

### 1. 销售预测 / Sales Forecasting

```python
# 销售数据预测
sales_df = pd.read_csv("sales_data.csv")

tc = TimeCopilot(llm="openai:gpt-4o")
result = tc.forecast(
    df=sales_df,
    freq="D",  # 日频数据
    h=30,      # 预测30天
    query="下个月的销售趋势如何？是否需要调整库存策略？"
)

print(result.output.user_query_response)
```

### 2. 网站流量预测 / Website Traffic Forecasting

```python
# 网站流量预测
traffic_df = pd.read_csv("website_traffic.csv")

tc = TimeCopilot(llm="openai:gpt-4o")
result = tc.forecast(
    df=traffic_df,
    freq="H",  # 小时频数据
    h=168,     # 预测一周 (7*24 小时)
    query="哪些时段流量会达到峰值？需要准备多少服务器资源？"
)
```

### 3. 金融数据预测 / Financial Data Forecasting

```python
# 股价或汇率预测
price_df = pd.read_csv("price_data.csv")

tc = TimeCopilot(llm="openai:gpt-4o")
result = tc.forecast(
    df=price_df,
    freq="B",  # 工作日频率
    h=20,      # 预测20个工作日
    query="未来一个月的价格走势如何？有哪些风险因素需要关注？"
)
```

### 4. 能源消耗预测 / Energy Consumption Forecasting

```python
# 电力消耗预测
energy_df = pd.read_csv("energy_consumption.csv")

tc = TimeCopilot(llm="openai:gpt-4o")
result = tc.forecast(
    df=energy_df,
    freq="D",
    h=90,  # 预测三个月
    query="夏季用电高峰期间，每日用电量预计是多少？"
)
```

## 结果解释和分析 / Results Interpretation

### 输出结构 / Output Structure

```python
result = tc.forecast(df=df)

# 主要输出字段
print("特征分析:", result.output.tsfeatures_analysis)
print("选择的模型:", result.output.selected_model)
print("模型详情:", result.output.model_details)
print("模型比较:", result.output.model_comparison)
print("优于基准模型:", result.output.is_better_than_seasonal_naive)
print("选择原因:", result.output.reason_for_selection)
print("预测分析:", result.output.forecast_analysis)
print("查询回答:", result.output.user_query_response)
```

### 可视化结果 / Visualizing Results

```python
import matplotlib.pyplot as plt

# 绘制预测结果
fig, ax = plt.subplots(figsize=(12, 6))

# 历史数据
historical = df.tail(24)  # 最后24期
ax.plot(historical['ds'], historical['y'], label='历史数据', marker='o')

# 预测数据
forecast = result.fcst_df
ax.plot(forecast['ds'], forecast[result.output.selected_model], 
        label='预测', marker='s', color='red')

ax.set_title(f'{result.output.selected_model} 预测结果')
ax.set_xlabel('日期')
ax.set_ylabel('值')
ax.legend()
ax.grid(True, alpha=0.3)
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()
```

## 性能优化建议 / Performance Optimization

### 1. 模型选择优化 / Model Selection Optimization

```python
# 对于简单数据，使用较少模型
simple_models = [AutoARIMA(), AutoETS(), Theta()]

# 对于复杂数据，添加基础模型
complex_models = [
    AutoARIMA(), AutoETS(), Theta(),
    Chronos(repo_id="amazon/chronos-t5-small"),  # 使用较小的模型
]
```

### 2. 批处理优化 / Batch Processing Optimization

```python
# 调整批处理大小
chronos = Chronos(
    repo_id="amazon/chronos-t5-small",
    batch_size=32,  # 根据 GPU 内存调整
)
```

### 3. LLM 成本优化 / LLM Cost Optimization

```python
# 使用较便宜的模型进行快速分析
tc_fast = TimeCopilot(llm="openai:gpt-4o-mini")

# 重要分析使用高质量模型
tc_detailed = TimeCopilot(llm="openai:gpt-4o")
```

## 错误处理和调试 / Error Handling & Debugging

### 常见问题解决 / Common Issue Resolution

```python
try:
    result = tc.forecast(df=df)
except Exception as e:
    print(f"预测失败: {e}")
    
    # 检查数据格式
    print("数据列:", df.columns.tolist())
    print("数据类型:", df.dtypes)
    print("缺失值:", df.isnull().sum())
    
    # 使用回退策略
    from timecopilot.models.stats import SeasonalNaive
    fallback_forecaster = TimeCopilotForecaster(
        models=[SeasonalNaive()],
    )
    result = tc.forecast(df=df, forecaster=fallback_forecaster)
```

### 调试模式 / Debug Mode

```python
import logging

# 启用详细日志
logging.basicConfig(level=logging.DEBUG)

# 使用更多重试次数
tc = TimeCopilot(llm="openai:gpt-4o", retries=5)
```

## 部署和生产使用 / Deployment & Production Usage

### 环境变量配置 / Environment Variables

```bash
# 设置 OpenAI API 密钥
export OPENAI_API_KEY="your-api-key"

# 设置其他 LLM 提供商密钥
export ANTHROPIC_API_KEY="your-anthropic-key"

# 可选：设置日志配置
export LOGFIRE_TOKEN="your-logfire-token"
```

### Docker 部署 / Docker Deployment

```dockerfile
FROM python:3.11-slim

# 安装依赖
COPY requirements.txt .
RUN pip install -r requirements.txt

# 复制代码
COPY . /app
WORKDIR /app

# 设置环境变量
ENV OPENAI_API_KEY=${OPENAI_API_KEY}

# 运行应用
CMD ["python", "-m", "timecopilot.cli"]
```

### API 服务 / API Service

```python
from fastapi import FastAPI
from timecopilot import TimeCopilot
import pandas as pd

app = FastAPI()
tc = TimeCopilot(llm="openai:gpt-4o-mini")

@app.post("/forecast")
async def forecast_endpoint(data: dict):
    df = pd.DataFrame(data["data"])
    result = tc.forecast(
        df=df,
        freq=data.get("freq"),
        h=data.get("h", 12),
        query=data.get("query"),
    )
    
    return {
        "analysis": result.output.dict(),
        "forecast": result.fcst_df.to_dict("records"),
    }
```

## 最佳实践 / Best Practices

### 1. 数据准备 / Data Preparation
- 确保数据质量和完整性
- 处理缺失值和异常值
- 使用适当的频率标识

### 2. 模型选择 / Model Selection
- 从简单模型开始
- 根据数据复杂性逐步添加高级模型
- 考虑计算资源和时间成本

### 3. 查询设计 / Query Design
- 使用具体、明确的问题
- 结合业务场景提问
- 避免过于技术性的术语

### 4. 结果验证 / Result Validation
- 始终检查预测的合理性
- 结合领域知识验证结果
- 定期重新训练和评估模型

通过遵循这些指南，您可以充分利用 TimeCopilot 的强大功能，在各种实际场景中获得高质量的时间序列预测结果。