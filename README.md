# RandomNumberGenerator

**Impact of Random Number Quality on Monte Carlo Option Pricing Accuracy**

**随机数质量对蒙特卡洛期权定价方法精确度的影响**

---

## Overview / 概述

This project evaluates and compares the quality and performance of various random number generators (RNGs) and their impact on the accuracy of Monte Carlo simulations for European call option pricing.

本项目通过定量分析，系统地比较和评价了多种随机数生成算法的质量与性能，并研究其对蒙特卡洛欧式看涨期权定价精确度的影响。

The study covers 13 uniform distribution generators and 7 normal distribution generators from multiple libraries, using a comprehensive suite of statistical tests to assess sequence quality.

研究涵盖了来自多个计算库的 13 种均匀分布生成算法和 7 种正态分布生成算法，通过一组完整的统计检验来评估序列质量。

## Methods / 方法

### Uniform Distribution Generators / 均匀分布生成算法

| Method | Source | Description |
|--------|--------|-------------|
| Linear Congruential | base | 简单线性同余算法 |
| Mersenne Twister | Python `random` | Python 标准库默认算法 |
| PCG64 | NumPy | `np.random.default_rng` 默认算法 |
| Latin Hypercube | SciPy | 拉丁超立方采样 |
| Poisson Disk | SciPy | 泊松盘采样 |
| Halton | SciPy | Halton 低差异序列 |
| Sobol (SciPy) | SciPy | Sobol 低差异序列 |
| Sobol (Unit) | QuantLib | Sobol 序列，单位初始化矩阵 |
| Sobol (Kuo) | QuantLib | Sobol 序列，Kuo 初始化矩阵 |
| Sobol (Kuo3) | QuantLib | Sobol 序列，Kuo3 初始化矩阵 |
| Sobol (Jaeckel) | QuantLib | Sobol 序列，Jaeckel 初始化矩阵 |
| Sobol (Levitan-Lemieux) | QuantLib | Sobol 序列，Levitan-Lemieux 初始化矩阵 |
| Sobol (Joe-Kuo D7) | QuantLib | Sobol 序列，Joe-Kuo D7 初始化矩阵 |

### Normal Distribution Generators / 正态分布生成算法

| Method | Source | Description |
|--------|--------|-------------|
| Box-Muller (standard) | Python `random` | 标准库 Box-Muller 算法 |
| Ziggurat (NumPy) | NumPy | NumPy 的 Ziggurat 算法 |
| Ziggurat (SciPy) | SciPy | SciPy 的 Ziggurat 算法 |
| Inverse CDF | base + uniform | 逆 CDF 变换法 |
| Central Limit Theorem | base + uniform | 中心极限定理叠加法 |
| Box-Muller | base + uniform | 基于均匀分布的 Box-Muller 算法 |
| Rejection Sampling | base + uniform | 拒绝采样法 |

The "base + uniform" methods derive normal samples from uniform sequences, enabling cross-evaluation of how upstream uniform generator quality propagates into downstream normal distribution quality and final Monte Carlo accuracy.

标注为 "base + uniform" 的方法从均匀分布序列中生成正态分布样本，从而实现对上游均匀分布质量如何传导至下游正态分布质量和最终蒙特卡洛精度的交叉评估。

### Sequence Variants / 序列变体

Each generator produces three sequence variants for comparison:

每个生成器产生三种序列变体用于比较：

- **Origin** — Raw output from the generator / 生成器的原始输出
- **Dual** — Antithetic variates (mirror image for variance reduction) / 对偶变量（对称镜像，用于方差缩减）
- **Standard** — Normalized to standard range / 标准化至标准范围

### Statistical Tests / 统计检验

**Uniform distribution quality / 均匀分布质量：**

- Kolmogorov-Smirnov test (KS test)
- Cramér-von Mises test (CM test)
- Star discrepancy / 星偏差

**Normal distribution quality / 正态分布质量：**

- Kolmogorov-Smirnov test (KS test)
- Cramér-von Mises test (CM test)
- Lilliefors test (KSL test)
- D'Agostino's K-squared test (SK test)
- Jarque-Bera test (JB test)
- Skewness and kurtosis / 偏度与峰度

### Monte Carlo Pricing / 蒙特卡洛定价

European call option prices are computed via Monte Carlo simulation assuming geometric Brownian motion, then compared against the closed-form Black-Scholes solution. 10 test cases with varying parameters (S, σ, K, T, r) are used to measure relative pricing error across all RNG combinations.

采用几何布朗运动假设，通过蒙特卡洛模拟计算欧式看涨期权价格，并与 Black-Scholes 公式的解析解进行比较。使用 10 组不同参数 (S, σ, K, T, r) 的测试样例，衡量各 RNG 组合的相对定价误差。

## Project Structure / 文件结构

```
.
├── base.py              # Constants, helpers, Distribution abstract base class
│                        # 常量、工具函数、Distribution 抽象基类
├── Uniformer.py         # Uniform distribution generator (13 methods)
│                        # 均匀分布生成器（13 种算法）
├── Normalist.py         # Normal distribution generator (7 methods)
│                        # 正态分布生成器（7 种算法）
├── MonteCarlo.py        # Monte Carlo option pricing test
│                        # 蒙特卡洛期权定价测试
├── docs/
│   ├── document.ipynb   # Full analysis report (Jupyter Notebook)
│   │                    # 完整分析报告（Jupyter Notebook）
│   └── assets/          # Figures used in the report
│                        # 报告中使用的插图
├── reports/             # Generated test reports (not tracked)
│                        # 生成的测试报告（未纳入版本控制）
└── cache/               # Pickled intermediate data (not tracked)
                         # 缓存的中间数据（未纳入版本控制）
```

## Usage / 使用方法

### Prerequisites / 前置依赖

```bash
pip install numpy scipy pandas sympy statsmodels QuantLib-Python matplotlib seaborn
```

### Configuration / 配置

Key parameters are defined in `base.py`:

主要参数定义在 `base.py` 中：

```python
RANDOM_SEED = 42            # Random seed / 随机种子
SKIP_PATHS = 2 ** 18 - 1   # QuantLib sequence skip parameter / QuantLib 序列跳跃参数
MAX_BATCH_SIZE = 16384      # Max single batch size / 最大单次采样规模
STATS_SIZE = 65536          # Statistical sample size / 统计采样规模
BASE_SCALE = 1000000        # Data scale / 数据规模
```

### Running / 运行

The three modules should be run sequentially, as each depends on the cached results of the previous step:

三个模块需按顺序运行，每一步依赖上一步的缓存结果：

```bash
# Step 1: Generate and evaluate uniform distribution sequences
# 第一步：生成并评估均匀分布序列
python Uniformer.py

# Step 2: Generate and evaluate normal distribution sequences
# 第二步：生成并评估正态分布序列
python Normalist.py

# Step 3: Run Monte Carlo pricing tests
# 第三步：运行蒙特卡洛定价测试
python MonteCarlo.py
```

Test reports are saved to `reports/`. Cached generator objects are saved to `cache/` for reuse.

测试报告保存至 `reports/`。缓存的生成器对象保存至 `cache/` 以便复用。

### Analysis Report / 分析报告

See `docs/document.ipynb` for the full analysis with visualizations. The notebook's plotting functions require the cached data files generated by the steps above.

完整的分析报告及可视化详见 `docs/document.ipynb`。笔记本中的绘图函数需要上述步骤生成的缓存数据文件。

## License / 许可

This project is for research and educational purposes.

本项目用于研究与教学目的。
