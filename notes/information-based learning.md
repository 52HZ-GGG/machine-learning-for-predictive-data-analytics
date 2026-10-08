# 基于信息的学习学习笔记

> 覆盖：决策树基础、熵、信息增益、ID3、Play Tennis；信息增益比、Gini、连续特征、回归树、剪枝、Bagging、Boosting、Gradient Boosting

**总主线：**

```text
决策树基础
    → 熵：度量不确定性
    → 信息增益：选择分裂特征
    → ID3：自顶向下、贪心、递归建树
    → 特征选择度量改进：信息增益比 / Gini
    → 连续描述特征阈值化
    → 连续目标：回归树
    → 噪声、过拟合与剪枝
    → 模型集成：Bagging / Boosting / Gradient Boosting
    → 决策树优缺点总结
```

---

## 1. 决策树基础（Decision Tree）

**定义：**  
决策树是一种监督学习模型。核心思想是递归地选择最优特征对数据空间进行划分，使划分后的子集尽可能“纯”，最终用叶节点给出预测。

**节点类型：**

| 节点 | 含义 |
|---|---|
| 根节点 | 第一个分裂特征 |
| 内部节点 | 中间测试特征 |
| 叶子节点 | 最终预测类别或值 |

**关键目标：**  
每次分裂后，子集的类别纯度越高越好。

---

## 2. 香农熵模型（Shannon’s Entropy Model）

**熵：** 与结果概率相关，用来度量不确定性。

$$
H(X)=-\sum_{i=1}^{n}p_i\log_b p_i
$$

**性质：**

- 熵越小，数据集越纯；
- 熵越大，不确定性越高；
- 对数底为 2 时，单位为 **bit**；
- 对数底为 $e$ 时，单位为 **nat**。

**决策树中的经验熵：**

设数据集 $D$，目标类别集合为 $\{C_1,\dots,C_K\}$：

$$
H(D)=-\sum_{k=1}^{K}\frac{\lvert C_k \rvert}{\lvert D \rvert}\log_2\frac{\lvert C_k \rvert}{\lvert D \rvert}
$$

**条件熵：**

设特征 $A$ 有 $V$ 个取值，$D^v$ 是 $A=v$ 的子集：

$$
H(D \mid A)=\sum_{v=1}^{V}\frac{\lvert D^v \rvert}{\lvert D \rvert}H(D^v)
$$

---

## 3. 信息增益（Information Gain）

**直观含义：**  
知道特征 $X$ 后，目标 $Y$ 不确定性减少的程度。  
等价于互信息：

$$
IG(Y,X)=I(Y;X)=H(Y)-H(Y \mid X)
$$

**决策树中的定义：**

$$
g(D,A)=H(D)-H(D \mid A)
$$

其中：

- $H(D)$：划分前的经验熵；
- $H(D \mid A)$：按特征 $A$ 划分后的条件熵；
- $g(D,A)$：信息增益。

**用途：**

- ID3 决策树：选择信息增益最大的特征作为分裂特征；
- 特征选择：衡量特征与目标的相关性；
- 互信息：可捕捉非线性依赖。

**优点：**

- 直观，反映不确定性减少量；
- 计算简单，有信息论基础。

**缺点：**

- 偏向取值较多的特征；
- 连续特征需离散化；
- 对类别不平衡敏感。

**改进：**

- C4.5 使用信息增益率：

$$
\text{GainRatio}(D,A)=\frac{g(D,A)}{H_A(D)}
$$

其中 $H_A(D)$ 为特征 $A$ 的固有值。

- CART 使用基尼指数。

---

## 4. 标准方法：ID3 算法

**ID3：** Iterative Dichotomiser 3，第三版迭代二分器。  
由 Ross Quinlan 提出，是一种自顶向下、贪心、分治的决策树分类算法。

**核心：** 使用信息增益选择分裂属性，递归生成多叉决策树。

### 4.1 算法流程

1. 从根节点开始，使用全部训练样本；
2. 如果当前样本全属于同一类，生成叶节点，返回该类；
3. 如果没有剩余属性可用，返回当前样本中的多数类；
4. 否则，计算每个属性的信息增益；
5. 选择信息增益最大的属性作为当前分裂属性；
6. 按该属性的每个取值划分子集；
7. 对每个子集递归调用 ID3；
8. 直到满足停止条件。

### 4.2 伪代码

```text
ID3(S, Attributes, Target):
    if S 中所有样本属于同一类别 c:
        return Leaf(c)

    if Attributes 为空:
        return Leaf(S 中的多数类)

    A_best = argmax_{A in Attributes} Gain(S, A)
    tree = Node(A_best)

    for each value v in Values(A_best):
        S_v = {x in S | x[A_best] = v}

        if S_v 为空:
            tree.add_branch(v, Leaf(S 中的多数类))
        else:
            tree.add_branch(
                v,
                ID3(S_v, Attributes \ {A_best}, Target)
            )

    return tree
```

---

## 5. Play Tennis 完整示例

### 5.1 训练数据

| Day | Outlook | Temperature | Humidity | Wind | PlayTennis |
|---:|---|---|---|---|---|
| 1 | Sunny | Hot | High | Weak | No |
| 2 | Sunny | Hot | High | Strong | No |
| 3 | Overcast | Hot | High | Weak | Yes |
| 4 | Rain | Mild | High | Weak | Yes |
| 5 | Rain | Cool | Normal | Weak | Yes |
| 6 | Rain | Cool | Normal | Strong | No |
| 7 | Overcast | Cool | Normal | Strong | Yes |
| 8 | Sunny | Mild | High | Weak | No |
| 9 | Sunny | Cool | Normal | Weak | Yes |
| 10 | Rain | Mild | Normal | Weak | Yes |
| 11 | Sunny | Mild | Normal | Strong | Yes |
| 12 | Overcast | Mild | High | Strong | Yes |
| 13 | Overcast | Hot | Normal | Weak | Yes |
| 14 | Rain | Mild | High | Strong | No |

目标分布：Yes = 9，No = 5。

### 5.2 根节点选择

根节点熵：

$$
H(S)=-\frac{9}{14}\log_2\frac{9}{14}-\frac{5}{14}\log_2\frac{5}{14}\approx 0.940
$$

各属性条件熵与信息增益：

| 属性 | 条件熵 $H(S \mid A)$ | 信息增益 $Gain(S,A)$ |
|---|---:|---:|
| Outlook | 0.694 | **0.246** |
| Temperature | 0.911 | 0.029 |
| Humidity | 0.788 | 0.152 |
| Wind | 0.892 | 0.048 |

选择 `Outlook` 作为根节点。

### 5.3 递归分支

#### Outlook = Overcast

所有样本均为 `Yes`：

```text
Overcast -> Yes
```

#### Outlook = Sunny

子集：Yes = 2，No = 3。  
剩余属性信息增益：

| 属性 | 信息增益 |
|---|---:|
| Temperature | 0.571 |
| Humidity | **0.971** |
| Wind | 0.020 |

选择 `Humidity`：

```text
Sunny
├─ Humidity = High   -> No
└─ Humidity = Normal -> Yes
```

#### Outlook = Rain

子集：Yes = 3，No = 2。  
剩余属性信息增益：

| 属性 | 信息增益 |
|---|---:|
| Temperature | 0.020 |
| Humidity | 0.020 |
| Wind | **0.971** |

选择 `Wind`：

```text
Rain
├─ Wind = Weak   -> Yes
└─ Wind = Strong -> No
```

### 5.4 最终决策树

```text
Outlook
├─ Sunny
│  ├─ Humidity = High   -> No
│  └─ Humidity = Normal -> Yes
├─ Overcast             -> Yes
└─ Rain
   ├─ Wind = Weak       -> Yes
   └─ Wind = Strong     -> No
```

### 5.5 IF-THEN 规则

```text
IF Outlook = Overcast THEN Yes
IF Outlook = Sunny AND Humidity = High THEN No
IF Outlook = Sunny AND Humidity = Normal THEN Yes
IF Outlook = Rain AND Wind = Weak THEN Yes
IF Outlook = Rain AND Wind = Strong THEN No
```

---

## 6. 备选特征选择度量

### 6.1 信息增益比（Information Gain Ratio）

**问题：**  
熵基信息增益 $IG$ 偏好取值多的特征。

**解决：**  
用信息增益比，把信息增益除以“确定该特征取值所需的信息量”（分裂信息）：

$$
GR(d,D)=\frac{IG(d,D)}{-\sum_{l\in levels(d)}P(d=l)\log_2 P(d=l)}
\tag{4.8}
$$

其中：

- $D$：数据集，当前节点上的实例集合；
- $d$：候选分裂特征；
- $levels(d)$：特征 $d$ 的所有可能取值；
- $P(d=l)$：在数据集 $D$ 中，特征 $d$ 取值为 $l$ 的实例比例；
- 分母：特征 $d$ 自身的取值分布熵，也叫**分裂信息**或**固有值**，用来惩罚取值过多的特征。

**植被分类数据集示例：**

| 特征 | IG | 分裂信息 | 信息增益比 GR |
|---|---:|---:|---:|
| STREAM | 0.3060 | 0.9852 | 0.3106 |
| SLOPE | 0.5774 | 1.1488 | ≈0.5026 |
| ELEVATION | 0.8774 | 1.8424 | ≈0.4762 |

**选择建议：**

| 情况 | 建议 |
|---|---|
| 计算资源有限 | 信息增益计算更便宜 |
| 特征取值数量差异大 | 信息增益比可能更好 |
| 实际效果 | 因领域而异，最好实验比较 |
| 通用做法 | 尝试不同度量，选择模型效果最好的 |

---

### 6.2 Gini 指数（Gini Index）

**定义：**

$$
Gini(t,D)=1-\sum_{l\in levels(t)}P(t=l)^2
\tag{4.9}
$$

**含义：**  
若按数据集中目标水平的分布随机分类，Gini 指数可理解为“误分类的期望频率”。

**性质：**

| 情况 | Gini 值 |
|---|---|
| 所有实例同一目标水平 | 0 |
| 有 $k$ 个目标水平且等概率 | $1-\frac{1}{k}$ |
| 二分类等概率 | 0.5 |
| 四分类等概率 | 0.75 |

**用 Gini 计算信息增益：**  
把熵替换为 Gini 指数即可：

$$
IG_{Gini}=Gini(D)-\sum_{l\in levels(d)}\frac{\lvert D_{d=l} \rvert}{\lvert D \rvert}\times Gini(D_{d=l})
$$

**植被数据集示例：**

$$
Gini(VEGETATION,D)
=1-\left[\left(\frac{3}{7}\right)^2+\left(\frac{2}{7}\right)^2+\left(\frac{2}{7}\right)^2\right]
=0.6531
$$

**表 4.7 结果：**

| 特征 | Gini 余项 Rem. | 信息增益 Info. Gain |
|---|---:|---:|
| STREAM | 0.5476 | 0.1054 |
| SLOPE | 0.4000 | 0.2531 |
| ELEVATION | 0.3333 | 0.3198 |

**结论：**  
虽然数值与熵不同，但特征相对排名相同。植被数据集上，用 Gini 生成的树与用熵生成的树相同。

**选择建议：**

- Gini 与熵没有绝对优劣  
- 实践中应尝试不同不纯度度量  
- 选择在验证集上表现最好的

---

## 7. 处理连续描述特征

### 7.1 基本思路

连续特征不能直接按取值分支。  
最简单方法：**设定阈值**，把连续特征变成布尔特征：

```text
ELEVATION >= threshold  → true / false
```

### 7.2 如何选阈值

1. 按连续特征值排序  
2. 只考虑相邻实例中**目标分类不同**的边界  
3. 取相邻值的中间点作为候选阈值  
4. 对每个候选阈值计算信息增益  
5. 选择信息增益最高的阈值

### 7.3 植被数据集连续 ELEVATION 示例

排序后候选边界：

| 边界 | 计算 | 阈值 |
|---|---:|---:|
| d2 与 d4 | $(300+1200)/2$ | 750 |
| d4 与 d3 | $(1200+1500)/2$ | 1350 |
| d3 与 d7 | $(1500+3000)/2$ | 2250 |
| d1 与 d5 | $(3900+4450)/2$ | 4175 |

候选阈值的信息增益：

| 阈值 | Rem. | Info. Gain |
|---|---:|---:|
| $\ge 750$ | 1.2507 | 0.3060 |
| $\ge 1350$ | 1.3728 | 0.1839 |
| $\ge 2250$ | 0.9650 | 0.5917 |
| $\ge 4175$ | 0.6935 | **0.8631** |

**最佳阈值：** $\ge 4175$

### 7.4 注意点

- 连续特征可以在同一路径上多次使用  
- 每次使用可以有不同的阈值  
- 动态创建的布尔特征可以和其他分类特征竞争  
- 每个节点都可以重新选阈值

---

## 8. 预测连续目标：回归树

### 8.1 目标

回归树的目标：  
让每个叶节点内目标值的**方差**尽可能小。

### 8.2 节点不纯度：方差

$$
\mathrm{var}(t,D)=\frac{\sum_{i=1}^{n}(t_i-\overline{t})^2}{n-1}
\tag{4.10}
$$

### 8.3 选择分裂特征

选择使加权方差最小的特征：

$$
\mathbf{d}[best]=\underset{d\in \mathbf{d}}{\arg\min}
\sum_{l\in level(d)}\frac{\lvert D_{d=l} \rvert}{\lvert D \rvert}\times \mathrm{var}(t,D_{d=l})
\tag{4.11}
$$

### 8.4 ID3 的修改

| 原 ID3 | 回归树修改 |
|---|---|
| 叶节点返回类别 | 叶节点返回目标均值 |
| 用熵/信息增益选特征 | 用方差/加权方差选特征 |
| 可加提前停止 | 分区实例少于阈值就停止 |
| Line 8 用 $IG$ | 替换为最小化加权方差 |

**自行车租赁示例：**

| 特征 | 加权方差 |
|---|---:|
| SEASON | 1,379,331 |
| WORK DAY | 2,551,813 |

选择 SEASON，因为加权方差更小。

---

## 9. 噪声、过拟合与树剪枝

### 9.1 过拟合

- 决策树过拟合：对无关特征也进行分裂  
- 树越深，越容易基于小样本做出偶然分类  
- 训练集一致 ≠ 新数据泛化好

### 9.2 剪枝

**剪枝：** 识别并移除可能由噪声和样本方差导致的子树。  
剪枝后树可能不再完全拟合训练集，但泛化能力通常更好。

**两类剪枝：**

| 类型 | 做法 | 别名 |
|---|---|---|
| 预剪枝 | 提前停止递归分裂 | 前向剪枝 |
| 后剪枝 | 先长满，再剪掉过拟合分支 | — |

### 9.3 预剪枝

常见策略：

1. **Early stopping**
   - 分区实例数低于阈值
   - 信息增益不足
   - 树深度超过限制
2. **$\chi^2$ 剪枝**
   - 用统计显著性检验判断子树重要性

**优点：** 计算高效，适合小数据。  
**缺点：** 可能错过子树中才出现的特征交互。

### 9.4 后剪枝与减少误差剪枝

**后剪枝：**  
先让树完全生长，再剪掉导致过拟合的枝。

**减少误差剪枝（Reduced Error Pruning）：**

1. 树建到完整  
2. 自底向上、从左到右搜索可剪子树  
3. 用验证集比较：
   - 子树根节点预测误差
   - 子树所有叶节点预测误差  
4. 若根节点误差 ≤ 叶节点组合误差，则剪掉该子树

**数据划分：**

```text
Training Set | Validation Set | Test Set
```

验证集不参与树诱导，只用于评估剪枝。

### 9.5 剪枝优缺点

| 优点 | 说明 |
|---|---|
| 树更小 | 更容易解释 |
| 提高泛化 | 噪声抑制 |
| 防止过拟合 | 牺牲训练集一致性换泛化 |

---

## 10. 模型集成（Model Ensembles）

### 10.1 定义

不是只建一个模型，而是建一组模型，再聚合输出。

**两个定义特征：**

1. 用同一数据集的不同修改版本，训练多个不同模型  
2. 聚合多个模型的预测

### 10.2 聚合方式

| 目标类型 | 聚合方法 |
|---|---|
| 分类目标 | 投票机制，多数投票 |
| 连续目标 | 均值、中位数等集中趋势 |

### 10.3 独立模型多数投票错误率

**例：**  
11 个独立模型，每个错误率 0.2。  
多数投票出错 = 6 个或以上模型同时出错。

用二项分布：

$$
\sum_{k=6}^{11}\binom{11}{k}0.2^k(1-0.2)^{11-k}=0.017
$$

即集成错误率约 **1.7%**，远低于单个模型的 20%。

---

## 11. Bagging

### 11.1 原理

**Bagging = Bootstrap Aggregating**

- 每个模型训练于一个 **bootstrap sample**  
- bootstrap sample：有放回抽样，大小与原数据集相同  
- 每个样本不同 → 模型不同  
- 决策树对数据变化敏感，特别适合 Bagging

### 11.2 随机森林

**随机森林 = Bagging + 子空间采样 + 决策树**

- Bagging：有放回抽样  
- 子空间采样：每次只用随机选择的描述特征子集  
- 决策树：作为基模型

### 11.3 心脏病风险示例

三个 bootstrap 样本训练出三棵树：

| 树 | 根节点 | 查询预测 |
|---|---|---|
| Tree 1 | EXERCISE | rare → high |
| Tree 2 | SMOKER | false → low |
| Tree 3 | OBESE | true → high |

查询：

```text
EXERCISE = rarely, SMOKER = false, OBESE = true, FAMILY = yes
```

投票：

- Tree 1：high  
- Tree 2：low  
- Tree 3：high  

**多数投票 → high**

---

## 12. Boosting

### 12.1 原理

- 迭代创建模型，加入集成  
- 每个新模型更关注之前模型错分的实例  
- 通过**加权数据集**实现

### 12.2 加权数据集

- 每个实例有权重 $w_i\ge 0$  
- 初始权重：$1/n$  
- 每轮后：
  - 正确分类的实例权重降低
  - 错误分类的实例权重提高  
- 按权重分布抽样，生成复制的训练集

### 12.3 权重更新与置信因子

每轮训练：

1. 计算总误差：

$$
\epsilon=\sum_{\text{误分类实例}}w_i
$$

2. 增加误分类权重：

$$
w[i]\leftarrow w[i]\times \left(\frac{1}{2\epsilon}\right)
\tag{4.12}
$$

3. 降低正确分类权重：

$$
w[i]\leftarrow w[i]\times \left(\frac{1}{2(1-\epsilon)}\right)
\tag{4.13}
$$

4. 计算模型置信因子：

$$
\alpha=\frac{1}{2}\log_e\left(\frac{1-\epsilon}{\epsilon}\right)
\tag{4.14}
$$

$\epsilon$ 越小，$\alpha$ 越大，模型投票权越大。

### 12.4 聚合

- 分类：加权投票  
- 连续：加权均值  
- 权重：各模型的置信因子 $\alpha$



---

## 13. 梯度提升（Gradient Boosting）

### 13.1 原理

- 迭代训练模型  
- 后期模型直接纠正前期模型的错误  
- 比传统 Boosting 更激进

### 13.2 基本流程

初始模型：

$$
M_0(\mathbf{d})=\frac{1}{n}\sum_{i=1}^{n}t_i
\tag{4.15}
$$

连续目标中，$M_0$ 通常预测训练集目标均值。

迭代加入新模型：

$$
M_1(\mathbf{d})=M_0(\mathbf{d})+M_{\Delta1}(\mathbf{d})
\tag{4.16}
$$

$$
M_i(\mathbf{d})=M_{i-1}(\mathbf{d})+M_{\Delta i}(\mathbf{d})
\tag{4.17}
$$

最终模型：

$$
M_4(\mathbf{d})
=M_0(\mathbf{d})+M_{\Delta1}(\mathbf{d})+M_{\Delta2}(\mathbf{d})+M_{\Delta3}(\mathbf{d})+M_{\Delta4}(\mathbf{d})
$$

### 13.3 扩展

- 可加学习率 $\alpha$：

$$
M_i(\mathbf{d})=M_{i-1}(\mathbf{d})+\alpha\times M_{\Delta i}(\mathbf{d})
\tag{4.19}
$$

- 可适应分类目标：初始模型预测各类别概率，后续模型纠正概率  
- 可最小化其他损失函数，不限于 MSE

---

## 14. Bagging vs Boosting

| 维度 | Bagging | Boosting |
|---|---|---|
| 训练方式 | 并行、独立 | 串行、迭代 |
| 样本 | Bootstrap 有放回抽样 | 加权数据集，关注错误 |
| 模型关系 | 相互独立 | 后期模型依赖前期模型 |
| 聚合 | 投票 / 均值 | 加权投票 / 加权均值 |
| 过拟合 | 较稳健 | 更易过拟合 |
| 适合特征数 | > 4000 时随机森林更好 | ≤ 4000 时提升树常最好 |
| 实现 | 简单，易并行 | 较复杂 |
| 典型代表 | 随机森林 | AdaBoost、Gradient Boosting |

---

## 15. 决策树优缺点总结

| 优点 | 缺点 |
|---|---|
| 可解释 | 连续特征会导致树很大 |
| 处理分类和连续特征 | 表达力强，敏感，易过拟合 |
| 能建模特征交互 | 是 eager learner |
| 相对抗维度灾难 | 不适合概念漂移 |
| 剪枝后抗噪声 | 需重新训练才能适应变化 |

---

## 16. 公式速查卡

### 16.1 基础部分

| 指标 | 公式 |
|---|---|
| 香农熵 | $H(X)=-\sum_{i=1}^{n}p_i\log_b p_i$ |
| 经验熵 | $H(D)=-\sum_{k=1}^{K}\frac{\lvert C_k \rvert}{\lvert D \rvert}\log_2\frac{\lvert C_k \rvert}{\lvert D \rvert}$ |
| 条件熵 | $H(D \mid A)=\sum_{v=1}^{V}\frac{\lvert D^v \rvert}{\lvert D \rvert}H(D^v)$ |
| 信息增益 | $g(D,A)=H(D)-H(D \mid A)$ |
| 互信息 | $IG(Y,X)=I(Y;X)=H(Y)-H(Y \mid X)$ |
| 信息增益率 | $\text{GainRatio}(D,A)=\frac{g(D,A)}{H_A(D)}$ |
| 单位 | 底 2 → bit；底 $e$ → nat |

### 16.2 特征选择

| 指标 | 公式 |
|---|---|
| 信息增益比 | $GR(d,D)=\frac{IG(d,D)}{-\sum P(d=l)\log_2P(d=l)}$ |
| Gini 指数 | $Gini(t,D)=1-\sum P(t=l)^2$ |
| Gini 信息增益 | $IG_{Gini}=Gini(D)-\sum \frac{\lvert D_{d=l} \rvert}{\lvert D \rvert}Gini(D_{d=l})$ |

### 16.3 回归树

| 指标 | 公式 |
|---|---|
| 方差 | $\mathrm{var}(t,D)=\frac{\sum(t_i-\bar{t})^2}{n-1}$ |
| 最佳分裂 | $\arg\min\sum \frac{\lvert D_{d=l} \rvert}{\lvert D \rvert}\mathrm{var}(t,D_{d=l})$ |

### 16.4 集成学习

| 指标 | 公式 |
|---|---|
| 二项分布错误率 | $\sum_{k=m}^{n}\binom{n}{k}p^k(1-p)^{n-k}$ |
| Boosting 误分类权重 | $w[i]\leftarrow w[i]\times\frac{1}{2\epsilon}$ |
| Boosting 正确分类权重 | $w[i]\leftarrow w[i]\times\frac{1}{2(1-\epsilon)}$ |
| 置信因子 | $\alpha=\frac{1}{2}\ln\left(\frac{1-\epsilon}{\epsilon}\right)$ |
| 梯度提升初始模型 | $M_0(d)=\frac{1}{n}\sum t_i$ |
| 梯度提升迭代 | $M_i=M_{i-1}+M_{\Delta i}$ |
| 加学习率 | $M_i=M_{i-1}+\alpha M_{\Delta i}$ |

### 16.5 方法速查

| 类别 | 方法 |
|---|---|
| 特征选择 | 信息增益、信息增益比、Gini |
| 连续描述特征 | 排序、候选阈值、信息增益选阈值 |
| 连续目标 | 回归树、方差、加权方差 |
| 剪枝 | 预剪枝、后剪枝、减少误差剪枝 |
| 集成 | Bagging、随机森林、Boosting、梯度提升 |
| 聚合 | 多数投票、加权投票、均值、中位数 |

---

## 17. 术语中英对照

| 中文 | 英文 |
|---|---|
| 决策树 | Decision Tree |
| 根节点 | Root Node |
| 内部节点 | Internal Node |
| 叶子节点 | Leaf Node |
| 香农熵 | Shannon’s Entropy |
| 信息增益 | Information Gain |
| 互信息 | Mutual Information |
| 条件熵 | Conditional Entropy |
| 经验熵 | Empirical Entropy |
| 信息增益率 | Information Gain Ratio |
| 基尼指数 | Gini Index |
| 固有值 | Intrinsic Value |
| 迭代二分器 | Iterative Dichotomiser |
| 连续描述特征 | Continuous Descriptive Features |
| 回归树 | Regression Tree |
| 预剪枝 | Pre-pruning |
| 后剪枝 | Post-pruning |
| 减少误差剪枝 | Reduced Error Pruning |
| 模型集成 | Model Ensemble |
| 自助聚合 | Bagging / Bootstrap Aggregating |
| 随机森林 | Random Forest |
| 子空间采样 | Subspace Sampling |
| 提升 | Boosting |
| 梯度提升 | Gradient Boosting |
| 置信因子 | Confidence Factor |
| 加权数据集 | Weighted Dataset |

---

## 18. 总结

```text
决策树 → 熵 → 信息增益 → ID3 → Play Tennis
→ 信息增益比 / Gini
→ 连续特征阈值化
→ 回归树
→ 过拟合与剪枝
→ Bagging / 随机森林
→ Boosting / Gradient Boosting
→ 决策树优缺点
```

**一句话：**  
从熵和信息增益出发，理解决策树如何选择分裂特征；再用信息增益比、Gini 改进特征选择，处理连续特征和连续目标；用剪枝抑制过拟合；用 Bagging、Boosting、Gradient Boosting 提升泛化能力。