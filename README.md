# CS Major Simulator

基于 Monte Carlo Simulation 的 CS2 Major Pick'Em 分析工具。

> Forked from [Wenhuaren5/cs-major-simulator](https://github.com/Wenhuaren5/cs-major-simulator)

---

## 快速开始

```
https://zhangmai19.github.io/cs-major-simulator/web/
```

打开后选择对应 Stage，给每队打分（1-10），可选标记状态不稳的队伍，点击"开始模拟"。

---

## 与上游原版的差异

### 1. 胜率模型：线性 → Logistic

| | 原版 | 本 Fork |
|---|---|---|
| 公式 | `ratingA / (ratingA + ratingB)` | `1 / (1 + e^{-K × (ratingA - ratingB)})` |
| 类型 | 线性比例 | Logistic / Sigmoid |
| 差 1 分 | 56%（5 vs 4）不可控 | **73%** 可控 |
| 差 2 分 | 60% | **88%** |
| 差 3 分 | 64% | **95%** |
| K 值 | — | `K = 1`（固定） |

原版的线性模型对分差不敏感——5 分打 4 分和 9 分打 1 分的差距完全一样。Logistic 模型更接近真实竞技：分差越大，胜率加速拉开。

### 2. 不稳定标记 ⚡ — 新增

每支队伍可以勾选 **"不稳定"**，代表该队状态波动大、容易爆冷：

| 情况 | K 值 | 差 3 分时的胜率 |
|---|---|---|
| 正常 | K = 1 | 95% |
| 一方不稳定 | K = 0.5 | 82% |
| 双方不稳定 | K = 0.25 | 68% |

勾选框在评分框旁边，灰色默认未勾，勾选后变金色。保存评分时会一并持久化到 localStorage。

### 3. Stage 2 模拟器 — 新增

原版只有 Stage 1。本 fork 新增完整的 **Stage 2 模拟器**，包含 Stage 2 的真实第一轮对阵。

### 4. Stage 3 模拟器 — 新增

完整的 **Stage 3（Playoffs）模拟器**，支持两种 Scenario：
- **Scenario A**：BIG 晋级
- **Scenario B**：B8 晋级

用户可以在下拉菜单中切换，评分独立保存。

### 5. Pick'Em 推荐去重 — 修复

原版 `generatePickemRecommendation` 中，3-0 推荐的队伍可能再次出现在晋级推荐里，导致同一队伍出现在两个位置。

本 fork 的 `getBestAdvancePicks` 新增 `excludedTeams` 参数，自动排除已被 3-0 选中的队伍。

### 6. Pick'Em 计分逻辑 — 修复

| 条件 | 原版 | 本 Fork |
|---|---|---|
| 3-0 预测 → 队伍 3-0 | ✓ | ✓ |
| 3-0 预测 → 队伍晋级但非 3-0 | ✗ 仍计分 | ✓ **不计分** |
| 晋级预测 → 队伍 3-0 | ✓ 计分 | ✓ **不计分**（要求 losses > 0） |

### 7. Test Lab 使用 Logistic 模型 — 修复

原版 Test Lab 在继续模拟剩余比赛时使用固定 50/50 随机（忽略 rating），本 fork 改为使用 Logistic 模型根据用户评分计算胜率。

### 8. C++ 命令行版 — 同步更新

C++ 版本的胜率模型也加入了不稳定标记支持，输入评分后可选择标记不稳定的队伍。

### 9. 项目结构

```
cs-major-simulator/
├── index.html                          # 根目录跳转
├── web/
│   ├── index.html                      # 主页（所有 Stage 入口）
│   ├── css/style.css
│   ├── js/
│   │   ├── simulation.js               # Swiss 引擎 + Team 类
│   │   ├── montecarlo.js               # Monte Carlo + Logistic 胜率
│   │   └── recommendation.js           # Pick'Em 推荐系统
│   ├── simulator/
│   │   ├── simulator_stage1/           # Stage 1 模拟器
│   │   ├── simulator_stage2/           # Stage 2 模拟器（新增）
│   │   └── simulator_stage3/           # Stage 3 模拟器（新增）
│   └── test/
│       ├── test_stage1/                # Stage 1 Test Lab
│       ├── test_stage2/                # Stage 2 Test Lab（新增）
│       └── test_stage3/                # Stage 3 Test Lab（新增）
└── cpp/
    └── main.cpp                        # C++ 命令行版
```

---

## 技术细节

### Logistic 胜率模型

```
P(A 赢) = 1 / (1 + e^{-K × (ratingA - ratingB)})
```

其中 K 受不稳定标记影响：
- 双方稳定：K = 1
- 单方不稳定：K = 0.5
- 双方不稳定：K = 0.25

### Monte Carlo 模拟

默认 10,000 次模拟。每次模拟完整走过瑞士轮 5 轮，使用 Swiss pairing 规则（Buchholz 排序 + 避免重复对阵）。

### 保5最优推荐

遍历候选组合（3-0 前 5 选 2 × 晋级前 9 选 6 × 0-3 前 5 选 2），计算每套 Pick'Em 的保5概率，选出最优解。

---

## 部署

GitHub Pages，从 `web/` 目录自动部署。

---

## 记录

- 上游原版：[Wenhuaren5/cs-major-simulator](https://github.com/Wenhuaren5/cs-major-simulator)
- 本 Fork：[zhangmai19/cs-major-simulator](https://github.com/zhangmai19/cs-major-simulator)
