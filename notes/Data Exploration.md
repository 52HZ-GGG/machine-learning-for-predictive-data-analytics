# 1. 描述统计（Descriptive Statistics）

数据探索贯穿 CRISP-DM 的 **Data Understanding** 与 **Data Preparation**。描述统计用集中趋势、变异程度等标准度量刻画 ABT 中的每个特征。

**探索总目标：**

1. 熟悉 ABT 中各特征的集中趋势、变异与分布形态
2. 识别质量问题：缺失值、不规则基数、离群点
3. 修正因**无效数据**导致的问题
4. 将因**有效数据**导致的问题记入 Data Quality Plan，并写明处理策略
5. 确认数据质量与数量足以支撑后续建模

---

## 1.1 连续特征（Continuous Features）

### 1.1.1 集中趋势（Central Tendency）

| 指标 | 定义 | 公式 / 说明 |
|------|------|-------------|
| **均值 Mean** | 算术平均 | $\bar{a}=\dfrac{1}{n}\sum_{i=1}^{n}a_i$ |
| **中位数 Median** | 排序后中间值 | $n$ 为偶数时，取中间两值的均值 |

- 样本（sample）：特征在 ABT 中的一组取值
- 集中趋势只是对“中心位置”的**近似**
- 中位数对极端值更稳健

### 1.1.2 变异程度（Variation）

> 统计与分析的核心在于描述和理解**变异（variation）**。

| 指标 | 公式 | 说明 |
|------|------|------|
| **极差 Range** | $\max(a)-\min(a)$ | 最简单的变异度量 |
| **方差 Variance** | $\mathrm{var}(a)=\dfrac{1}{n-1}\sum_{i=1}^{n}(a_i-\bar{a})^2$ | 与均值的平均偏离；样本方差分母为 $n-1$ |
| **标准差 Std** | $sd(a)=\sqrt{\mathrm{var}(a)}$ | 与原变量同量纲 |
| **百分位 Percentiles** | 第 $i$ 百分位：约 $i/100$ 的取值 ≤ 该值 | 见下 |
| **四分位距 IQR** | $Q_3-Q_1$ | 25th = 下四分位 $Q_1$；75th = 上四分位 $Q_3$ |

**百分位计算步骤：**

1. 将 $n$ 个值升序排序
2. 计算 $index = n \times i/100$
3. 若 $index$ 为整数 → 取第 $index$ 个位置的值
4. 若 $index$ 非整数 → 令 $index\_w$ 为整数部分、$index\_f$ 为小数部分，插值：
   $$
   \text{percentile} = (1-index\_f)\times a_{index\_w} + index\_f \times a_{index\_w+1}
   $$

**例（$n=8$）：**

- 25th：$0.25\times 8 = 2$ → 取排序后第 2 个值
- 80th：$0.80\times 8 = 6.4$ → $0.6\times a_6 + 0.4\times a_7$

---

## 1.2 分类特征（Categorical Features）

| 概念 | 定义 |
|------|------|
| **频数 Frequency count** | 该水平在样本中出现的次数 |
| **占比 Proportion** | 频数 ÷ 样本总量 |
| **频数表 Frequency table** | 各水平频数与占比的汇总表 |
| **众数 Mode** | 出现最多的水平（分类特征的集中趋势） |
| **第二众数 2nd mode** | 第二常见的水平 |

---

## 1.3 总体与样本（Population & Sample）

| 术语 | 含义 |
|------|------|
| **总体 Population** | 研究/分析中关心的全部可能观测或结果 |
| **样本 Sample** | 从总体中选出、实际用于分析的子集 |

- 民调中的 **margin of error（误差范围）** 反映结果基于样本而非总体

---

# 2. 数据可视化（Data Visualization）

| 图型 | 适用 | 要点 |
|------|------|------|
| **条形图 Bar plot** | 分类特征 | 可画频数、占比（density）、按占比排序（ordered）；**不能**直接用于连续特征 |
| **直方图 Histogram** | 连续特征 | 将取值范围切成 bins，统计各 bin 频数/密度 |
| **箱线图 Box plot** | 连续特征 | 中位数、Q1、Q3、须，便于看分布与离群 |

---

## 2.1 常见直方图形态

| 形态 | 含义 | 例子 |
|------|------|------|
| **Uniform 均匀** | 各区间取值概率大致相同 | — |
| **Normal 正态（单峰）** | 向中心集中，两侧近似对称 | 身高、体重 |
| **Skewed right 右偏** | 长尾在右侧（高值） | 收入等 |
| **Skewed left 左偏** | 长尾在左侧（低值） | — |
| **Exponential 指数** | 低值概率很高，随数值增大迅速衰减 | — |
| **Multimodal 多峰** | 两个或更多明显分离的高发区间 | 特征混合了多个子群体 |

## 2.2 正态分布（Normal / Gaussian）

**概率密度函数：**

$$
N(x\mid\mu,\sigma)=\frac{1}{\sqrt{2\pi}\,\sigma}\,e^{-\frac{(x-\mu)^2}{2\sigma^2}}
$$

- $x$：任意取值
- $\mu$：总体均值（控制位置）
- $\sigma$：总体标准差（控制宽度）

**标准正态分布：** $\mu=0,\ \sigma=1$

**68–95–99.7 法则：**

| 范围 | 大约包含的观测比例 |
|------|--------------------|
| $\mu \pm 1\sigma$ | 68% |
| $\mu \pm 2\sigma$ | 95% |
| $\mu \pm 3\sigma$ | 99.7% |

---

# 3. 数据质量报告（The Data Quality Report）

对 ABT（Analytics Base Table）中每个特征，用集中趋势与变异度量做**表格报告**，并配可视化：

- 连续特征 → **直方图**
- 分类特征 → **条形图**

### 3.1 报告表格结构

| 连续特征表字段 | 分类特征表字段 |
|----------------|----------------|
| Count / Missing % | Count / Missing % |
| Cardinality | Cardinality |
| Mean / Min / Max | Mode / Mode % |
| 1st Q / Median / 3rd Q | 2nd Mode / 2nd Mode % |
| SD / IQR | |

### 3.2 关键字段

- **Cardinality**：该特征在 ABT 中的不同取值个数
- **Mode / Mode %**：最常见水平及其占比
- **2nd Mode / 2nd Mode %**：第二常见水平及其占比

### 3.3 读表要点

**分类特征：**

- 看 mode、2nd mode 及其占比
- 判断是否有某水平**主导**数据集

**连续特征：**

- mean 与 std → 集中趋势与波动
- min 与 max → 可能取值范围

---

# 4. 识别数据质量问题（Identifying Data Quality Issues）

## 4.1 定义与分类

**数据质量问题**：ABT 中任何“不寻常”的数据。

| 类型 | 原因 | 处理原则 |
|------|------|----------|
| **无效数据** | 生成 ABT 的流程出错 | 立即修正 → 重新生成 ABT 与质量报告 |
| **有效数据** | 真实世界如此 | 记入 Data Quality Plan，选处理策略 |

## 4.2 三大常见问题

### （1）缺失值 Missing values

- 查看 Missing %
- 注意异常模式：如“大量为 0”

### （2）不规则基数 Irregular cardinality

基数与特征定义预期不符时，视为问题。

**检查清单：**

- [ ] Cardinality = 1（常量特征，无信息）
- [ ] 分类特征被标成连续（或连续被标成分类）
- [ ] 分类特征基数远超定义预期
- [ ] 水平数过多（如 >50）需调查

### （3）离群点 Outliers

- **定义**：远离特征集中趋势的取值
- **识别方法**：
  1. 用 min / max + **领域知识**判断是否合理
  2. 比较 median、min、max、Q1、Q3 的间距
  3. 若 **Q3 与 max 的间距**明显大于 **median 与 Q3 的间距** → max 很可能是离群点

## 4.3 讲义案例（车险欺诈 ABT）

| 问题类型 | 具体表现 |
|----------|----------|
| 缺失 | NUM. SOFT TISSUE 约 2%；MARITAL STATUS >60%；INCOME 大量为 0 |
| 基数 | INSURANCE TYPE=1；FRAUD FLAG 实为分类却当连续；多个 count 型特征基数 <10 |
| 离群 | CLAIM AMOUNT 最小值 −99,999（错误）；个别高额保单/重伤导致 max 异常高 |

---

# 5. 处理质量问题（Handling Data Quality Issues）

## 5.1 缺失值

| 策略 | 做法 |
|------|------|
| Approach 1 | 直接删掉有缺失的特征 |
| Approach 2 | **Complete case analysis**：只保留完整样本 |
| Approach 3 | 派生 **missing indicator** 特征（标记是否缺失） |
| **Imputation 填补** | 用已有值估计一个合理值替换缺失；最常用**集中趋势**（均值/中位数） |

**填补经验阈值：**

- 缺失 **>30%**：通常不愿做填补
- 缺失 **>50%**：强烈不建议填补

## 5.2 离群点 — Clamp（截断）

把超过上下阈值的取值压到阈值：

$$
a_i \leftarrow
\begin{cases}
\text{lower} & a_i < \text{lower}\\
\text{upper} & a_i > \text{upper}\\
a_i & \text{otherwise}
\end{cases}
$$

### 阈值设定方法一：IQR 法（箱线图思路）

$$
\begin{aligned}
\text{lower} &= Q_1 - 1.5 \times \mathrm{IQR}\\
\text{upper} &= Q_3 + 1.5 \times \mathrm{IQR}
\end{aligned}
$$

**例（CLAIM AMOUNT）：**

- lower = 3,322.3 − 1.5 × 8,923.2 = **−10,062.5**
- upper = 12,245.5 + 1.5 × 8,923.2 = **25,630.3**
  
IQR 是 Interquartile Range 的缩写，中文叫 四分位距。
它表示数据中间 50% 的分布范围，计算公式是：
$$
IQR=Q3−Q1
$$

### 阈值设定方法二：均值 ± 2×标准差

$$
\begin{aligned}
\text{lower} &= \bar{a} - 2 \times sd(a)\\
\text{upper} &= \bar{a} + 2 \times sd(a)
\end{aligned}
$$

**例（AMOUNT RECEIVED）：**

- lower = 13,051.9 − 2 × 30,547.2 = **−48,042.5**
- upper = 13,051.9 + 2 × 30,547.2 = **74,146.3**

## 5.3 Data Quality Plan

所有质量问题及处理策略应写入 **数据质量计划（Data Quality Plan）**，便于复现与团队协作。

---

# 6. 本章小结（Summary）

- [ ] 熟悉 ABT 各特征的集中趋势、变异与分布
- [ ] 识别缺失值、不规则基数、离群点
- [ ] 修正无效数据导致的问题
- [ ] 有效数据问题记入 Data Quality Plan + 处理策略
- [ ] 确认数据足以支撑后续项目

---

# 7. 公式速查卡

| 指标 | 公式 |
|------|------|
| 均值 | $\bar{a}=\frac{1}{n}\sum a_i$ |
| 样本方差 | $\mathrm{var}(a)=\frac{1}{n-1}\sum(a_i-\bar{a})^2$ |
| 标准差 | $sd(a)=\sqrt{\mathrm{var}(a)}$ |
| 极差 | $\max(a)-\min(a)$ |
| IQR | $Q_3-Q_1$ |
| Clamp IQR 阈值 | $Q_1-1.5\cdot\mathrm{IQR}$ / $Q_3+1.5\cdot\mathrm{IQR}$ |
| Clamp 均值阈值 | $\bar{a}\pm 2\cdot sd(a)$ |
| 正态 PDF | $\frac{1}{\sqrt{2\pi}\sigma}e^{-\frac{(x-\mu)^2}{2\sigma^2}}$ |
| 68–95–99.7 | $\mu\pm1\sigma$ / $\mu\pm2\sigma$ / $\mu\pm3\sigma$ |

---

*Reading: Section 3.1–3.4 + Appendix A*
