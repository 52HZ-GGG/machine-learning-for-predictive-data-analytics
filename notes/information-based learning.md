# Information-based Learning

## Fundamentals
### Decision Tree
决策树是一种监督学习模型。其核心思想是：递归地选择最优特征对数据空间进行划分，使划分后的子集尽可能“纯”，最终用叶节点给出预测。

决策树包含根节点、内部节点、叶子节点。

### Shannon’s Entropy Model 
entropy熵：realated to the probability of a outcome.
$$
H(X)=-\sum_{i=1}^{n} p_i \log_b p_i
$$

熵越小，数据集越纯

### Infomation Gain
信息增益表示知道特征 \(X\) 后，目标 \(Y\) 不确定性减少的程度，等价于互信息：

$$
IG(Y,X)=I(Y;X)=H(Y)-H(Y|X)
$$

#### 决策树中的定义

设数据集 \(D\)，目标类别集合，特征 \(A\)。

经验熵：
$$
H(D)=-\sum_{k=1}^{K}\frac{|C_k|}{|D|}\log_2\frac{|C_k|}{|D|}
$$

条件熵：
$$
H(D|A)=\sum_{v=1}^{V}\frac{|D^v|}{|D|}H(D^v)
$$

信息增益：
$$
g(D,A)=H(D)-H(D|A)
$$

其中 \(D^v\) 是特征 \(A\) 取值为 \(v\) 的子集。

#### 用途

- ID3 决策树：选择信息增益最大的特征作为分裂特征。
- 特征选择：衡量特征与目标的相关性。
- 互信息：可捕捉非线性依赖。

## 优缺点

优点：
- 直观，反映不确定性减少量。
- 计算简单，有信息论基础。

缺点：
- 偏向取值较多的特征。
- 连续特征需离散化。
- 对类别不平衡敏感。

改进：
- C4.5 使用信息增益率：
$$
\text{GainRatio}(D,A)=\frac{g(D,A)}{H_A(D)}
$$
其中 \(H_A(D)\) 为特征 \(A\) 的固有值。
- CART 使用基尼指数。

#### 单位

对数底为 2 时单位为 bit；为 \(e\) 时单位为 nat。

## Standard Approach:The ID3 Algorithm
ID3:Iterative Dichotomizer 3(3是指第三版)