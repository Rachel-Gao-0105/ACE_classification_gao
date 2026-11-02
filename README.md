# ACE 分类与模式识别

本部分基于 BRFSS 2023 预处理后的 35,754 名 45 岁及以上受访者数据，首先核对并累加 11 个童年不良经历（ACE）二元条目，将受访者划分为 0、1、2–3 和 4+ 四组，分析各组分布、ACE 两两共现及完整暴露组合，并比较各组的性别、年龄、教育、收入、婚姻和种族/族裔构成。在此基础上，采用二元潜在类别分析（LCA）拟合并比较不同类别数的模型，综合模型拟合、分类质量和可解释性，最终选择五类模型，识别低 ACE 暴露型、身体与言语虐待型、性虐待主导型、家庭功能障碍型和多重 ACE 暴露型，并进一步描述各类的人口统计特征及主观认知下降（SCD）比例。

## 当前文件结构

```text
项目根目录/
├── Preprocessing/
│   └── BRFSS_2023_ACE_SCD_main.csv     # 两本 Notebook 共用的输入
└── ACE_classification_gao/
    ├── ACE_classification.ipynb       # ACE 评分、分组、共现及人口统计
    ├── ACE_LCA.ipynb                  # LCA 模型比较及五类后续分析
    ├── README.md
    ├── requirements.txt
    ├── data/                         # 主分析分类数据
    ├── tables/                       # 主分析结果表及 validation.json
    ├── figures/                      # 主分析 5 张图的 PNG/PDF
    └── lca/
        ├── data/                     # 五类分类数据及个人后验概率
        ├── tables/                   # 模型比较、分类质量、SCD、人口统计等
        ├── figures/                  # LCA 6 张主图及 4/5/6 类轮廓对照图
        └── models/                   # K=1–6 模型参数和 selection.json
```

## 环境与运行

1. 使用 Python 3 环境。
2. 如需安装依赖，在项目根目录执行：

   ```powershell
   python -m pip install -r ACE_classification_gao/requirements.txt
   ```
3. 内核工作目录应为项目根目录，或 `ACE_classification_gao` 本目录；确保上述 CSV 相对位置不变。只复制 Notebook 而不提供输入 CSV，无法完整重现分析。

`ACE_classification.ipynb` 每次重新读取 CSV 并生成主分析结果。`ACE_LCA.ipynb` 的控制变量为：

```python
REFIT = True
```

- `True`：从输入 CSV 完整重新拟合；固定随机种子 `20261102`，K=2–6 每个模型 30 个随机起点，K=1 为单次基准，最终采用 `selected_k=5`。
- `False`：读取已有 LCA 输出；缺少 `lca/models/selection.json` 时仍会拟合。该模式需要配套保存的模型、分类数据和结果表，不能只保留 JSON。输入数据或估计设置改变后应使用 `True`。

LCA 使用 EM，最大迭代次数为 10,000，相邻迭代对数似然的绝对变化小于 `1e-5` 时判定收敛。同一分析重新运行会更新对应数据表、模型和图片；不会自动改写本 README，因此调整模型或数据后需同步更新本文。

## 重点结果文件

| 路径                                                                                            | 内容                                                                                                            |
| ----------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| [data/BRFSS_2023_ACE_classified.csv](data/BRFSS_2023_ACE_classified.csv)                         | 原始列、重新计算的 ACE 分数、四组分类及人口统计标签                                                             |
| [tables/ace_group_distribution.csv](tables/ace_group_distribution.csv)                           | 四组原始人数、未加权与加权占比                                                                                  |
| [tables/ace_exact_patterns_all.csv](tables/ace_exact_patterns_all.csv)                           | 全样本所有互斥完整 ACE 组合                                                                                     |
| [tables/demographics_by_ace_group_long.csv](tables/demographics_by_ace_group_long.csv)           | 四组人口统计计数和组内比例                                                                                      |
| [tables/validation.json](tables/validation.json)                                                 | 输入 SHA256、评分核对和主分析校验记录                                                                           |
| [lca/models/selection.json](lca/models/selection.json)                                           | 五类最终选择、六类 BIC 最优记录、设置及分类诊断                                                                 |
| [lca/tables/model_comparison.csv](lca/tables/model_comparison.csv)                               | AIC、BIC、entropy、类别大小与起点稳定性；`bic_best` 和 `selected_candidate` 分别表示 BIC 最优及实际采用模型 |
| [lca/tables/random_start_diagnostics.csv](lca/tables/random_start_diagnostics.csv)               | 每个随机起点的种子、收敛、似然和迭代次数                                                                        |
| [lca/tables/selected_item_probabilities.csv](lca/tables/selected_item_probabilities.csv)         | 五类条件概率 P(ACE 条目=1｜类别)                                                                                |
| [lca/tables/selected_class_summary.csv](lca/tables/selected_class_summary.csv)                   | 五类名称、比例、样本量、后验分类质量及 SCD 比例                                                                 |
| [lca/tables/classification_posterior_matrix.csv](lca/tables/classification_posterior_matrix.csv) | 各硬分类组内的平均后验概率矩阵                                                                                  |
| [lca/data/BRFSS_2023_ACE_LCA_classified.csv](lca/data/BRFSS_2023_ACE_LCA_classified.csv)         | 每人的类别、五个后验概率、最大后验概率及不确定标志                                                              |
| [lca/tables/demographics_by_lca_class_long.csv](lca/tables/demographics_by_lca_class_long.csv)   | 五类人口统计计数和组内比例；另有 n（加权 %）宽表                                                                |
| `lca/tables/class_vs_ace_score_group_*.csv`                                                   | 五类与 ACE 四组的交叉表及类内百分比                                                                             |
| `lca/tables/item_probabilities_k4.csv`、`k5.csv`、`k6.csv`                                | 不同类别数模型的条件概率对照表（后两者使用同一文件名前缀）                                                      |

`source_row` 表示输入 CSV 的行号（含表头，数据从第 2 行开始），用于核对行顺序，不是受访者唯一 ID。LCA 后验概率之和为 1，`lca_class` 为后验概率最大的类别，`lca_uncertain_lt_0_70` 表示最大后验概率低于 0.70。

## 图表导航

每张图均提供 300 dpi PNG。

| 主分析                           | 图表                                                        |
| -------------------------------- | ----------------------------------------------------------- |
| ACE 四组分布                     | [Fig1](figures/Fig1_ACE_group_distribution.png)              |
| 两项 ACE 共现热图                | [Fig2](figures/Fig2_ACE_cooccurrence_heatmap.png)            |
| 全样本前 20 个非空完整组合 UpSet | [Fig3](figures/Fig3_ACE_UpSet_top20.png)                     |
| 四组性别、教育、收入构成         | [Fig4](figures/Fig4_demographics_sex_education_income.png)   |
| 四组年龄、婚姻、种族/族裔构成    | [Fig5](figures/Fig5_demographics_age_group_marital_race.png) |

| LCA 分析                      | 图表                                                                                                                                         |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| AIC/BIC 与 entropy 比较       | [LCA1](lca/figures/LCA1_model_comparison.png)                                                                                                 |
| 最终五类条件概率热图          | [LCA2](lca/figures/LCA2_class_profiles.png)                                                                                                   |
| 模型比例与硬分类比例          | [LCA3](lca/figures/LCA3_class_sizes.png)                                                                                                      |
| 五类 SCD 描述性比例           | [LCA4](lca/figures/LCA4_SCD_by_class.png)                                                                                                     |
| 五类性别、教育、收入构成      | [LCA5](lca/figures/LCA5_demographics_sex_education_income.png)                                                                                |
| 五类年龄、婚姻、种族/族裔构成 | [LCA6](lca/figures/LCA6_demographics_age_group_marital_race.png)                                                                              |
| 统一色标的类别轮廓对照        | [4 类](lca/figures/LCA_class_profiles_k4.png) · [5 类](lca/figures/LCA_class_profiles_k5.png) · [6 类](lca/figures/LCA_class_profiles_k6.png) |

不同 K 的类别分别按期望 ACE 条目数排序，跨模型的 Class 编号不能直接一一对应。五类内部的人口统计图以各类为分母，保留 Missing；类别条件概率热图展示模型估计的概率，两者含义不同。

## ACE 评分与描述性分析

### ACE 分组结果

| ACE 组 | 未加权 n | 未加权 % | 加权 % |
| ------ | -------: | -------: | -----: |
| 0      |   14,342 |    40.11 |  38.77 |
| 1      |    8,245 |    23.06 |  23.10 |
| 2-3    |    7,649 |    21.39 |  21.90 |
| 4+     |    5,518 |    15.43 |  16.23 |

### 共现与组合

- Physical abuse + Verbal abuse：共同暴露 n=5,171，加权比例 14.86%，加权 phi=0.387。
- Household alcohol problems + Verbal abuse：共同暴露 n=4,185，加权比例 11.58%，加权 phi=0.287。
- Parental separation/divorce + Verbal abuse：共同暴露 n=3,485，加权比例 10.75%，加权 phi=0.185。

前 20 个非空组合覆盖全部样本的 32.72%；图中没有展示的组合仍保留在完整结果表。

### 人口统计比较

- sex / Female：0组 54.6%；1组 50.4%；2-3组 53.0%；4+组 59.6%。
- education / College graduate：0组 36.9%；1组 34.9%；2-3组 34.1%；4+组 25.9%。
- income / <$15k：0组 3.7%；1组 3.8%；2-3组 4.1%；4+组 7.1%。
- income / Missing：0组 22.3%；1组 18.3%；2-3组 15.7%；4+组 12.4%。

## 五类 LCA：模型选择与结果

本研究采用基于伯努利混合模型的二元潜在类别分析（Binary LCA），以 11 个 ACE 二元指标作为输入，通过最大似然估计和 EM 算法识别不同的 ACE 暴露模式。比较 1–6 类模型后，综合模型拟合、分类质量与可解释性，最终采用五类模型。

### 估计与模型选择

样本 n=35,754，11 个二元 ACE 条目；把相同回答聚合为 1020 种模式，按模式人数估计，与逐人未加权似然等价。采用条件独立 Bernoulli 混合模型和 EM，K=1 为基准，K=2–6 各采用 30 个固定种子的随机初始值。SCD 和人口统计变量不参与类别拟合。

参数数目为 11K+K−1；AIC=−2LL+2p，BIC=−2LL+p log(n)。采用未加权最大似然比较模型，调查权重仅用于拟合后的描述性汇总；这不是复杂抽样加权 LCA。

分类 entropy 定义为 1 + Σ_iΣ_c r_ic log(r_ic)/(n log K)，越接近 1 分类越清晰，K=1 无定义；它不是单独确定类别数的依据。

| K |      AIC |      BIC | Entropy | 最小模型类别 % | 最优值重复次数 |
| - | -------: | -------: | ------: | -------------: | -------------: |
| 1 | 303915.6 | 304008.9 |  不适用 |         100.00 |              1 |
| 2 | 267721.1 | 267916.2 |   0.817 |          23.35 |             30 |
| 3 | 259922.7 | 260219.6 |   0.830 |           7.80 |             30 |
| 4 | 257561.9 | 257960.7 |   0.847 |           3.87 |             21 |
| 5 | 256397.5 | 256898.0 |   0.816 |           3.83 |             17 |
| 6 | 255429.5 | 256031.9 |   0.771 |           3.56 |             28 |

本次最终采用 **5 类**，entropy=0.816；最高后验概率低于 0.70 的受访者占 13.79%。类别按模型期望 ACE 条目数从低到高排列，Class 编号不是诊断标签。

#### 五类模型选择理由

选择五类是为了平衡模型拟合效果、分类质量、模型复杂度和可解释性。虽然六类模型的 AIC 和 BIC 更低，但五类模型 entropy 更高（0.816 vs. 0.771），类别区分相对更清晰。相比四类模型，五类模型细分了暴露轮廓，能够区分家庭功能障碍突出与身体、言语虐待突出群体；低暴露、身体与言语虐待、性虐待主导、家庭功能障碍和多重 ACE 五种轮廓具有实际解释价值。因此研究者选择五类作为主要描述性分析模型。

这一决定是研究者基于解释性与简约性的取舍，**不是信息准则支持五类优于六类**：五类相对六类 ΔBIC=866.18，差异明显；四类 entropy 也更高（0.847），因此不能声称五类的分类质量在所有候选中最佳。六类继续作为拟合更优的比较模型保留。

#### 稳定性、样本量与后验分类检查

五类模型 30/30 个起点收敛，其中 17 个起点重复达到最优对数似然（差值 <0.01）。这支持数值解的可重复性，但不等同于 bootstrap 重抽样下的人群类别稳定性。

| 类别 | 名称             | 硬分类 n | 模型比例 % | 平均归属后验概率 | 类内最高概率 <0.70 % |
| ---- | ---------------- | -------: | ---------: | ---------------: | -------------------: |
| 1    | 低 ACE 暴露型    |   25,095 |      67.58 |            0.928 |                 6.10 |
| 2    | 身体与言语虐待型 |    6,132 |      17.38 |            0.750 |                44.59 |
| 3    | 性虐待主导型     |    1,407 |       4.41 |            0.895 |                12.86 |
| 4    | 家庭功能障碍型   |    1,800 |       6.80 |            0.869 |                15.11 |
| 5    | 多重 ACE 暴露型  |    1,320 |       3.83 |            0.885 |                15.98 |

完整平均后验概率矩阵见 [classification_posterior_matrix.csv](lca/tables/classification_posterior_matrix.csv)；应结合各类指标判断不确定性，不能仅凭整体 entropy 认为所有类别均清晰。

**范围限制：BIC 最优落在搜索上限 6 类，只能称为 1–6 类中的最优候选，不能声称已经找到全局最佳类别数。更复杂模型也可能只是吸收条目间的局部依赖。**

### 类别解释

- Class 1（低 ACE 暴露型）：模型比例 67.58%，期望 ACE 条目数 0.53；最高概率条目为 Parental separation/divorce: 0.13; Verbal abuse: 0.10; Physical abuse: 0.09。硬分类 n=25,095，已分入该类者的平均后验概率 0.928。
- Class 2（身体与言语虐待型）：模型比例 17.38%，期望 ACE 条目数 2.80；最高概率条目为 Verbal abuse: 0.70; Physical abuse: 0.57; Household alcohol problems: 0.41。硬分类 n=6,132，已分入该类者的平均后验概率 0.750。
- Class 3（性虐待主导型）：模型比例 4.41%，期望 ACE 条目数 3.86；最高概率条目为 Sexual touching: 0.94; Made to touch sexually: 0.83; Verbal abuse: 0.42。硬分类 n=1,407，已分入该类者的平均后验概率 0.895。
- Class 4（家庭功能障碍型）：模型比例 6.80%，期望 ACE 条目数 4.30；最高概率条目为 Household alcohol problems: 0.73; Verbal abuse: 0.64; Household drug misuse: 0.59。硬分类 n=1,800，已分入该类者的平均后验概率 0.869。
- Class 5（多重 ACE 暴露型）：模型比例 3.83%，期望 ACE 条目数 7.56；最高概率条目为 Sexual touching: 0.96; Verbal abuse: 0.90; Made to touch sexually: 0.85。硬分类 n=1,320，已分入该类者的平均后验概率 0.885。

用上述条目特征解释类别，完整概率见 [selected_item_probabilities.csv](lca/tables/selected_item_probabilities.csv)。五类名称基于已拟合概率轮廓审阅后指定，不参与模型估计，也不是仅根据某一项概率机械命名。家庭功能障碍型也可能经历虐待，性虐待主导型也可能经历其他 ACE；名称不表示暴露互斥或诊断。

### SCD 衔接与分类不确定性

| 类别 | 硬分类 n | 未加权 SCD % | 加权 SCD % |
| ---- | -------: | -----------: | ---------: |
| 1    |   25,095 |        13.61 |      13.86 |
| 2    |    6,132 |        22.00 |      21.51 |
| 3    |    1,407 |        26.87 |      29.13 |
| 4    |    1,800 |        26.00 |      27.95 |
| 5    |    1,320 |        38.11 |      41.39 |

以上按最高后验概率硬分类计算，只是描述性关联；没有进行因果解释、显著性检验或回归推断。另提供后验概率加权的 SCD 描述性敏感性表，但这种简单加权不是 BCH/三步法等分类误差校正，不能替代正规的远端结局分析。后续 D 部分不应把硬分类当成无误差的真实类别。

### 模型诊断与限制

模型预测与观测的最大两项共现残差为 1.40 个百分点（见 [pairwise_model_residuals.csv](lca/tables/pairwise_model_residuals.csv)）。这是描述性诊断，不是局部独立性的正式检验。三个性虐待条目可能存在额外依赖；更高类别数可能反映该依赖，而非真实人群亚型。

同时检查随机起点最优值重复、收敛、边界概率、类别大小和完整概率轮廓。研究者选用的五类解并非已验证的人群分类；没有对类别数进行 bootstrap 检验或纳入复杂抽样设计。结果受州覆盖、完整病例筛选、回忆误差及样本权重影响。

### 方法来源

- [Linzer &amp; Lewis (2011), poLCA: An R Package for Polytomous Variable Latent Class Analysis](https://www.jstatsoft.org/article/view/v042i10)：有限混合模型、EM 与信息准则。
- [poLCA 官方文档](https://search.r-project.org/CRAN/refmans/poLCA/html/poLCA.html)：局部独立假设及多起点拟合。

## 五类 LCA：人口统计特征

### 分析口径

根据五类模型的最高后验概率分配类别，沿用 Class 1–5 编号和名称。人口统计变量不参与 LCA 拟合；这些结果是硬分类后的描述性汇总，不包含分类误差校正，不进行显著性检验或因果推断。

### 人口统计构成

比较性别、教育、收入、年龄、婚姻及种族/族裔。人数为原始样本人数；百分比使用 _LLCPWT，以各 LCA 类的全部受访者权重和为分母，Missing 单独保留。分类映射沿用 Notebook 内置的统一字段定义。长表同时包含未加权和加权比例；宽表为 n（加权 %）。

| 类别 | 名称             | 女性 % | 大学毕业 % | 收入 <$15k % | 收入缺失 % |
| ---- | ---------------- | -----: | ---------: | -----------: | ---------: |
| 1    | 低 ACE 暴露型    |   53.0 |       35.5 |          3.9 |       20.6 |
| 2    | 身体与言语虐待型 |   53.1 |       34.5 |          4.3 |       13.5 |
| 3    | 性虐待主导型     |   71.7 |       31.4 |          6.3 |       13.6 |
| 4    | 家庭功能障碍型   |   45.1 |       25.4 |          5.9 |       13.5 |
| 5    | 多重 ACE 暴露型  |   70.8 |       21.2 |          8.7 |       10.5 |
