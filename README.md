# 🚀 PnS++ Language Compiler v0.1.0-1 (Genesis-P)

> PnS++ is a high-performance probabilistic simulation and statistical language—featuring Python-like simplicity while delivering blazing-fast data processing speeds approaching those of C++.

PnS++ (Probabilistic & Statistical Language) is a Domain-Specific Language (DSL) designed for Monte Carlo simulations, statistical analysis, and parallel array computing. The compiler utilizes the Lark EBNF Parser to construct the AST and employs Numba JIT (LLVM Compiler Framework) as the backend to automatically enable Auto-SIMD and multi-threaded OpenMP execution.

---

## ⚡ Real-world Performance (Benchmark: 10M Samples)

| Processing Stage | Number of Elements | Execution Time | Notes |
| :--- | :--- | :--- | :--- |
| **Data Generation (`norm`)** | 10,000,000 | **~50.24 ms** | Parallel random array initialization |
| **SIMD Vector Filter (`> 2.0`)** | 10,000,000 | **6.86 ms** | RAM bandwidth optimization |
| **Native Mean Calculation** | 10,000,000 | **1.38 ms** | CPU register integration |
| **🚀 Total Computation Time** | **10,000,000** | **8.24 ms** | **Performance approaching C/C++** |

---

## 💻 Syntax Example (`demo.pspp`)

```pspp
data = @stats.norm(mean=0.0, std=1.0, n=10000000)
filtered = data > 2.0
result = @stats.mean(filtered)
out result
