
# AGENTS.md

## 数据分析与算法实验规范

当任务涉及以下任意内容时：

* 数据探索与 EDA
* 数据质量分析
* 数据统计分析
* 特征工程与特征验证
* 算法验证
* 模型训练
* 模型评估
* 模型对比
* 参数调优
* 阈值验证
* 消融实验
* 可视化分析
* 临时算法验证

必须遵循：

`docs/experiment-guidelines.md`

---

## 强制规则

### 1. 所有实验必须归档

任何数据分析或算法验证任务，都必须归属于：

```text
experiments/<experiment_name>/
```

禁止在项目根目录、`src/`、`scripts/` 或其他无关目录中随意生成实验文件。

---

### 2. 所有正式运行必须创建 Run

每一次具有实际分析意义的运行，都必须创建：

```text
experiments/<experiment_name>/runs/<run_id>/
```

`run_id` 格式：

```text
YYYYMMDD_HHMMSS_<short_description>
```

例如：

```text
20260813_203500_tcn_ae_baseline
20260813_211200_window_120
20260814_093000_xgboost_feature_v2
```

禁止使用：

```text
test
test2
new
latest
final
final2
result
```

作为实验或 Run 名称。

---

### 3. 所有生成文件必须进入当前 Run

实验产生的：

* 图片
* CSV / Parquet
* JSON
* 模型权重
* 预测结果
* 日志
* 临时文件
* 中间结果

必须存放在当前 Run 中。

禁止散落到项目其他位置。

---

### 4. 原始数据禁止修改

`data/raw/` 中的数据视为只读。

禁止：

* 原地修改
* 覆盖
* 重命名原始数据
* 将算法输出写回原始数据目录

需要加工的数据应生成新的中间数据或处理后数据。

---

### 5. 每个重要 Run 必须记录实验信息

Run 完成后必须至少包含：

```text
RUN.md
metrics.json        # 存在定量指标时必须生成
config.yaml         # 存在重要实验参数时应生成
```

`RUN.md` 必须记录：

* 实验目的
* 输入数据
* 数据范围
* 数据过滤条件
* 使用算法
* 使用特征
* 数据划分方式
* 重要参数
* 主要指标
* 主要图表
* 关键发现
* 实验结论
* 下一步建议

禁止仅将实验结果保留在终端输出或聊天上下文中。

---

### 6. 开始新实验前必须检查历史实验

开始新的数据分析或算法验证前，应首先检查：

```text
experiments/INDEX.md
```

以及相关 Experiment 的：

```text
README.md
runs/*/RUN.md
```

判断：

* 是否已经做过类似实验
* 是否已有可复用代码
* 是否已有失败结论
* 是否已有基线结果
* 当前任务应创建新 Experiment 还是复用已有 Experiment

避免重复实验。

---

### 7. 实验完成后必须更新索引

对于具有实际价值的 Run，完成后更新：

```text
experiments/INDEX.md
```

记录至少包括：

* 日期
* Experiment
* Run ID
* 实验目标
* 方法
* 主要指标
* 核心结论
* Run 路径

---

### 8. 禁止无意义文件命名

禁止创建：

```text
test.py
test2.py
new.py
final.py
final2.py

result.csv
result2.csv
new_result.csv
final_result.csv

1.png
test.png
new.png
result.png
```

文件名必须描述文件实际内容。

例如：

```text
reconstruction_error_distribution.png
anomaly_score_timeseries.png
feature_statistics.csv
anomaly_cells.csv
prediction_vs_actual.png
model_comparison.csv
```

---

### 9. 区分正式代码和临时代码

可复用的实验代码：

```text
experiments/<experiment_name>/scripts/
```

临时分析、调试代码：

```text
runs/<run_id>/tmp/
```

如果临时代码变得具有复用价值，应整理后移动到 `scripts/`。

不要长期保留：

```text
tmp.py
test.py
debug2.py
new_test.py
```

等无明确用途的脚本。

---

### 10. 任务结束前检查文件

完成数据分析或算法验证任务之前：

1. 检查是否存在散落在实验目录之外的生成文件。
2. 将有效产物整理进入当前 Run。
3. 删除无价值的临时文件。
4. 更新 `RUN.md`。
5. 更新 `metrics.json`。
6. 对重要实验更新 `experiments/INDEX.md`。
7. 执行：

```bash
git status --short
```

检查是否产生了意外文件。

---

## 核心原则

数据分析和算法验证统一遵循：

```text
问题
  ↓
Experiment
  ↓
Run
  ↓
代码 + 配置
  ↓
运行
  ↓
指标 + 图像 + 表格 + 模型 + 预测结果
  ↓
RUN.md
  ↓
INDEX.md
  ↓
下一次实验
```

如果无法判断某个实验生成文件应该放在哪里：

**优先放入当前 Run，而不是项目根目录。**




















# 数据分析与算法实验管理规范

## 1. 目标

本规范用于解决数据分析和算法开发过程中常见的以下问题：

* Python 脚本大量散落
* CSV、PNG、JSON 到处生成
* 不清楚一个结果来自哪次实验
* 参数调整后无法追踪历史
* 不清楚哪些方法已经尝试过
* 不知道某个模型文件对应哪些参数
* 很难复现实验
* AI Agent 多次执行任务后项目结构逐渐混乱

核心目标：

> 任何一个实验结果，在未来都能够回答：
>
> **为什么做、用了什么数据、用了什么方法、参数是什么、结果怎么样、结论是什么。**

---

# 2. Experiment 与 Run

整个实验管理采用两层结构：

```text
Experiment
    └── Run
```

## Experiment

Experiment 表示一个长期研究问题。

例如：

```text
cell_voltage_anomaly
temperature_consistency
soh_estimation
rul_prediction
iv_curve_step_detection
power_forecasting
```

同一个研究问题应持续复用同一个 Experiment。

不要因为调整一个参数就新建 Experiment。

---

## Run

Run 表示一次具有独立比较意义的实验执行。

当以下内容发生明显变化时，应创建新的 Run：

* 数据集
* 时间范围
* 样本筛选方式
* 数据划分
* 特征
* 算法
* 模型结构
* 重要超参数
* 阈值策略
* 评价方法

例如：

```text
cell_voltage_anomaly/
└── runs/
    ├── 20260813_203000_tcn_ae_baseline/
    ├── 20260813_210000_tcn_ae_window120/
    ├── 20260813_220000_tcn_ae_feature_v2/
    └── 20260814_090000_mtad_gat_baseline/
```

---

# 3. 标准实验目录

统一使用：

```text
experiments/
├── INDEX.md
│
└── <experiment_name>/
    ├── README.md
    │
    ├── scripts/
    │   ├── prepare_data.py
    │   ├── analyze.py
    │   ├── extract_features.py
    │   ├── train.py
    │   ├── evaluate.py
    │   └── visualize.py
    │
    ├── configs/
    │
    └── runs/
        └── <run_id>/
            ├── RUN.md
            ├── config.yaml
            ├── metrics.json
            │
            ├── figures/
            ├── tables/
            ├── predictions/
            ├── models/
            ├── logs/
            └── tmp/
```

---

# 4. 每个目录的职责

## scripts/

保存能够复用的实验代码。

例如：

```text
prepare_data.py
extract_features.py
train.py
evaluate.py
visualize.py
compare_models.py
```

---

## configs/

保存可复用实验配置。

例如：

```text
baseline.yaml
tcn_ae.yaml
mtad_gat.yaml
xgboost.yaml
```

重要参数尽可能通过配置文件管理，而不是散落在 Python 文件中。

---

## figures/

保存实验产生的图片。

例如：

```text
voltage_distribution.png
feature_correlation.png
reconstruction_error_distribution.png
anomaly_score_timeseries.png
prediction_vs_actual.png
```

---

## tables/

保存分析型表格。

例如：

```text
feature_statistics.csv
anomaly_cells.csv
model_comparison.csv
threshold_analysis.csv
```

---

## predictions/

保存逐样本、逐时间点的模型输出。

例如：

```text
predictions.parquet
predictions.csv
anomaly_scores.parquet
```

较大数据优先考虑 Parquet。

---

## models/

保存：

* 模型权重
* scaler
* encoder
* 模型序列化文件

例如：

```text
model.pt
model.pkl
model.onnx
scaler.pkl
```

---

## logs/

保存运行日志。

例如：

```text
train.log
evaluate.log
run.log
```

---

## tmp/

只允许存放当前 Run 的临时文件。

临时文件不应该长期保留。

---

# 5. RUN.md

每个重要 Run 必须生成：

```text
RUN.md
```

推荐格式：

```markdown
# 实验记录

## 1. 实验目的

本次实验希望验证什么问题？

## 2. 输入数据

数据来源：

数据路径：

时间范围：

样本数量：

数据筛选条件：

## 3. 数据处理

清洗方式：

缺失值处理：

异常值处理：

归一化方式：

数据划分：

## 4. 方法

算法：

模型：

特征：

核心处理流程：

## 5. 参数

主要参数：

模型参数：

阈值：

随机种子：

## 6. 结果

核心指标：

主要实验现象：

重要图表：

重要输出：

## 7. 关键发现

本次实验说明了什么？

与已有实验相比有什么变化？

## 8. 问题

当前方法还有什么问题？

## 9. 结论

继续 / 暂停 / 放弃 / 需要进一步验证。

## 10. 下一步

下一次最值得验证什么？
```

---

# 6. metrics.json

只要存在定量评价，就应该生成：

```text
metrics.json
```

例如分类问题：

```json
{
  "accuracy": 0.931,
  "precision": 0.925,
  "recall": 0.899,
  "f1": 0.912
}
```

时序预测：

```json
{
  "mae": 12.31,
  "rmse": 18.26,
  "mape": 0.071
}
```

异常检测：

```json
{
  "auc": 0.942,
  "precision": 0.901,
  "recall": 0.873,
  "f1": 0.887
}
```

重要指标不得只打印到 terminal。

---

# 7. INDEX.md

`experiments/INDEX.md` 是整个算法研究过程的总入口。

推荐：

```markdown
# Experiment Index

| 日期 | Experiment | Run | 方法 | 主要结果 | 结论 |
|---|---|---|---|---|---|
| 2026-08-13 | cell_voltage_anomaly | 20260813_203000_tcn_ae_baseline | TCN-AE | F1=0.91 | 可继续 |
| 2026-08-13 | cell_voltage_anomaly | 20260813_210000_window120 | TCN-AE | F1=0.93 | 优于 baseline |
| 2026-08-14 | cell_voltage_anomaly | 20260814_090000_mtad_gat | MTAD-GAT | F1=0.86 | 暂无优势 |
```

Agent 开始新的算法实验前，应优先读取 INDEX。

---

# 8. 数据管理

## 原始数据

```text
data/raw/
```

必须视为不可变数据。

禁止覆盖。

---

## 可复用处理数据

可以进入：

```text
data/processed/
```

前提是：

> 该数据未来有多个 Experiment 复用价值。

---

## 单次实验中间数据

必须放在：

```text
runs/<run_id>/
```

不要污染公共 `data/` 目录。

---

# 9. 文件命名

文件名表达：

> 这个文件里面是什么？

推荐：

```text
cell_voltage_distribution.png
reconstruction_error_distribution.png
abnormal_cell_ranking.csv
feature_importance.csv
prediction_vs_actual.png
```

禁止：

```text
1.png
2.png
test.png
result.png
result2.csv
new.csv
final.csv
final2.csv
```

---

# 10. 实验完成标准

只有满足以下条件，一个正式实验才算完成：

* [ ] 实验输出已经整理到 Run
* [ ] 没有重要文件散落在项目其他位置
* [ ] 已记录实验参数
* [ ] 已保存核心指标
* [ ] 已保存重要图表
* [ ] 已生成或更新 `RUN.md`
* [ ] 已写出实验结论
* [ ] 已写出下一步建议
* [ ] 重要 Run 已更新 `experiments/INDEX.md`

实验的终点不是：

> “代码运行成功。”

而应该是：

> **“实验问题已经得到记录完整、可追踪、可复现的结论。”**
