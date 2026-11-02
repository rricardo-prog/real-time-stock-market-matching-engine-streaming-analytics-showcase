# Discrete-Event Stock Exchange & Matching Engine (C++)

[![Language: C++17](https://img.shields.io/badge/Language-C%2B%2B17-blue.svg)](https://isocpp.org/)
[![Focus: Financial Systems & Algorithmic Matching](https://img.shields.io/badge/Focus-Order%20Book%20%26%20Simulation-brightgreen.svg)]()
[![Complexity: Sub--Linear Streaming](https://img.shields.io/badge/Complexity-Streaming%20Optimization-orange.svg)]()

A high-throughput, discrete-event stock exchange simulation and electronic matching engine developed in C++[cite: 23]. Designed to model limit order books across thousands of equities and traders, the engine processes streaming transaction feeds in real time, executes deterministic price-time priority matches, tracks live asset medians, and computes zero-sum trader reconciliations and maximum-profit arbitrage windows[cite: 24, 25, 31, 32].

---

> ### 🔒 Academic Integrity Notice
> This project was engineered for **EECS 281 (Data Structures and Algorithms) at the University of Michigan**[cite: 23]. In strict compliance with the University of Michigan College of Engineering Honor Code and course copyright regulations, implementation source code is maintained in a private repository[cite: 38].
>
> System architecture, algorithmic complexity breakdowns, and design invariants are detailed below. Live architecture reviews or private code walkthroughs are available upon request for interview evaluations.

---

## ⚡ Engineering Highlights & Architectural Metrics

* **Continuous Order Book Matching:** Maintains dual heap structures per equity (max-heap for buy bids, min-heap for sell asks) enforcing price-time priority matching rules and partial fill allocations in sub-linear time[cite: 25, 34, 36].
* **Two-Heap Running Median Invariant:** Implements dynamic streaming price percentiles using balanced max/min heaps, retrieving real-time medians in $O(1)$ time and incorporating incoming transactions in $O(\log k)$ time[cite: 31, 32].
* **$\Theta(1)$ Per-Order Temporal Arbitrage:** Analyzes optimal historical trading windows using single-pass streaming algorithms, tracking peak spread intervals across non-monotonic price sequences in linear overall runtime and $O(1)$ memory per asset[cite: 33].
* **Zero-Sum Balance Validation:** Formally audits market solvency at end-of-day, verifying across all participants that net share volumes and portfolio transfers reconcile precisely to zero[cite: 30, 32].
* **Zero Overhead I/O Pipeline:** Optimized stream extraction using buffered input pipelines (`std::cin` / pseudo-random stream generators), bypassing string re-allocation overheads during large transaction batches[cite: 27, 28].

---

## 🛠️ System Architecture & Data Structure Design

The simulation manages independent order books per asset while running concurrent analytics engines on transaction streams:

### 1. Dual Priority-Queue Order Book
* Each stock maintains two `std::priority_queue` containers[cite: 25, 36]:
  * **Buy Orders (Bids):** Max-priority queue ordered primarily by descending limit price, with ties broken by ascending order sequence ID (earliest first)[cite: 25, 36].
  * **Sell Orders (Asks):** Min-priority queue ordered primarily by ascending limit price, with ties broken by ascending order sequence ID[cite: 25, 36].
* **Match Invariant:** While $\text{top(Buy).price} \ge \text{top(Sell).price}$, the engine executes a trade at the price of the *resting* (earlier arriving) order, decrementing filled quantities until an order is exhausted[cite: 25, 34].

### 2. $O(\log K)$ Online Running Median Engine
* Computing medians by sorting after every transaction introduces unacceptable $O(K \log K)$ latency[cite: 32].
* Instead, each stock maintains a split-heap structure:
  * A max-heap for the lower half of prices.
  * A min-heap for the upper half of prices.
* Heaps are continuously balanced such that size differences never exceed 1. The median is calculated in $O(1)$ directly from heap roots, handling even/odd parity transitions seamlessly[cite: 31].

### 3. Online Linear Arbitrage ("Time Travelers")
* Identifies the optimal historical pair $(T_{\text{buy}}, T_{\text{sell}})$ where $T_{\text{buy}} < T_{\text{sell}}$ maximizing profit per share[cite: 33].
* Instead of quadratic brute-force comparisons, the engine runs an online state machine per stock:
  * Tracks the current running minimum sell price seen so far and its timestamp[cite: 33].
  * Whenever a new buy order arrives, calculates hypothetical profit against the running minimum; if greater than the global maximum spread, it commits the new optimal window in $\Theta(1)$ time and memory[cite: 33].

---

## ⏱️ Algorithmic Complexity

| Engine Module | Operation / Event | Time Complexity | Auxiliary Space |
| :--- | :--- | :--- | :--- |
| **Order Book Match** | Insert & Match per Order | $O(M \log N)$ | $O(N)$ |
| **Running Median** | Dynamic Transaction Insertion | $O(\log K)$ | $O(K)$ |
| **Running Median** | Query Current Median | $O(1)$ | $O(1)$ |
| **Time Traveler** | Single-Pass Order Evaluation | $\Theta(1)$ per order | $\Theta(1)$ per stock |
| **Trader Bookkeeping** | Position and P&L Update | $O(1)$ | $O(T)$ |

*Where $N$ is active unfulfilled orders, $M$ is completed trades per order, $K$ is total stock trades, and $T$ is total registered traders.*

---

## 💻 Tech Stack & Tooling

* **Language Standard:** C++17[cite: 23, 37]
* **Standard Library:** Containers (`std::vector`, `std::priority_queue`), Custom Comparator Functors, Dynamic Memory Management[cite: 23, 36]
* **Build System:** GNU `Make` / `g++` (`-O3`, `-Wall`, `-Wextra`, `-pedantic`)[cite: 36, 37]
* **Testing & Verification:** `Valgrind` dynamic analysis (ensuring zero leaks or uninitialized memory read violations across long streaming sessions)[cite: 39]; custom boundary-condition suites testing crossed books, zero-spread transactions, and simultaneous timestamp updates[cite: 27, 31, 38].

---

## 📈 Engineering Learnings & Quantitative Insights

* **Deterministic Priority Functors:** Built custom comparator structs that resolve multi-criteria strict weak ordering (price precedence followed by monotonic arrival timestamps) without ambiguous equivalences[cite: 25, 36].
* **Streaming Architecture:** Implemented single-pass processing where orders are matched or queued on arrival, preventing large batch-buffering latency[cite: 28].
* **Numerical Reconciliations:** Audited the transaction graph to verify zero-sum financial invariants across multi-party trade networks[cite: 32].
