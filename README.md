# Random Edge Sampling for Spanner Construction

**B.Sc. Thesis · Computer Science · Faculty of Mathematics, K. N. Toosi University of Technology · Summer 2026**

**Author:** Mohammadreza Ahmadi  **Supervisor:** [Dr. Hossein Jowhari](https://scholar.google.com/citations?user=a5yP56oAAAAJ)

> 📄 The thesis is written in **Persian** (XeLaTeX / `xepersian`). An English abstract is included at the end of the PDF and summarized below.

---

## TL;DR

Can you build a graph spanner by flipping an **independent coin for every edge**?
Prior randomized constructions (e.g. Baswana–Sen, Elkin–Neiman) inject randomness through *sequential, dependent* processes: sample radii, broadcast, cluster, then pick edges. This thesis asks a simpler question — assign each edge $e$ a probability $p_e$ computed from its local topology, sample all edges **independently**, and prove the result is a spanner.

**Main result.** For an unweighted graph $G=(V,E)$ with $n$ vertices and $m$ edges, and stretch $t = 2k-1$, there is a probability assignment $\mathbf p = (p_e)_{e\in E}$ such that the independently sampled subgraph $H_{\mathbf p}$ is, with high probability, a $(2k-1)$-spanner with

$$\mathbb{E}\big[\vert{}E(H_{\mathbf p})\vert{}\big] = O\!\left(2^{2k}\, n^{1+1/k} \log^2 n\right).$$

Choosing $k = \Theta(\sqrt{\log n})$ yields **near‑linear** sparsity $n^{1+o(1)}$.
A polynomial‑time ($O(m^3)$) two‑phase greedy algorithm makes the framework implementable while preserving every guarantee (up to a factor $t$ in the size bound).

---

## Background

A subgraph $H \subseteq G$ is a **multiplicative $\alpha$-spanner** if $d_H(u,v) \le \alpha \cdot d_G(u,v)$ for all $u,v \in V$. It suffices to check this for edges of $G$ (Prop. 1.2 in the thesis). Finding the sparsest $t$-spanner is NP‑complete even for $t=2$, and under Erdős' girth conjecture no $(2k-1)$-spanner can beat $\Omega(n^{1+1/k})$ edges — the bound achieved by the classical greedy algorithm of Althöfer et al. and, in the distributed setting, by Elkin–Neiman's exponential‑shift algorithm.

The thesis surveys these results (Chapter 1) before turning to the independent‑sampling question.

---

## Why naive independent sampling fails (Chapter 2)

| Idea | What goes wrong |
|---|---|
| **Global lower bound** $p_e \ge p_{\min}$ for all edges | A union bound over $m$ edges forces $p_{\min} \approx 1 - \delta/(tm)$, so $\mathbb{E}[\lvert E_{\mathbf p}\rvert] = \Omega(m)$ — no sparsification at all. |
| **Deterministic deletion** of any edge with a short alternative path | *Cascading failures*: the alternative path for $e_1$ contains $e_2$, whose alternative path contains $e_3$, … and the whole chain disappears. |
| **Count "good" edge‑disjoint alternative paths** $x_e$, set $p_e = \min(1, c/x_e)$ | *Dependency bottleneck*: a "good" path may consist of edges that themselves have huge $x_{e'}$ and therefore $p_{e'} \approx 0$, so the path's survival probability $S_P \to 0$ and the failure probability tends to $1 - c/x_e$. |

---

## The $2c$‑cutoff framework (Chapter 3)

**Safe paths.** For edge $e=(u,v)$, an alternative $u$–$v$ path $P$ is *safe* if
1. it is edge‑disjoint from the other safe paths of $e$,
2. $\vert{}P\vert{} \le t$, and
3. **every edge $e' \in P$ satisfies $x_{e'} \le 2c$.**

Let $x_e$ be the number of safe paths of $e$ and set

$$p_e = \min\!\left(1, \frac{c}{x_e}\right), \qquad c = \Theta\!\left(2^{2k} \log n\right).$$

Condition 3 is the key: every edge on a safe path has $p_{e'} \ge 1/2$, so

- **Lemma 3.1 (safe‑path survival):** $S_P \ge 2^{-t}$ for every safe path.
- **Lemma 3.2 (failure bound):** with $c \ge 2^t \ln(m/\delta)$, $\Pr[H_{\mathbf p} \text{ is not a } t\text{-spanner}] \le m\,e^{-c/2^t} \le \delta$.
- **Lemma 3.3 (expected size):** partition edges into *critical* ($x_e \le c$, kept w.p. 1) and *dense* ($x_e > c$). Both classes decompose into $O(x_e)$ edge‑disjoint subgraphs of girth $> 2k$ (a greedy decomposition à la Althöfer et al.), each with $O(n^{1+1/k})$ edges by the Moore bound. Logarithmic bucketing of the dense edges — where exponential bucket growth is exactly cancelled by exponential probability decay — gives the final $O(2^{2k} n^{1+1/k}\log^2 n)$.

---

## Making it polynomial‑time (Chapter 4)

Exactly computing $x_e$ (maximum number of edge‑disjoint paths of bounded length) is NP‑hard for $t \ge 4$ (Itai–Perl–Shiloach), and "safety" is circularly defined. The thesis breaks the circle with a **two‑phase greedy**:

1. **Phase 1 – upper bound.** For each $e$, repeatedly extract edge‑disjoint paths of length $\le t$ in $G \setminus \{e\}$ via truncated BFS, ignoring safety. The count $q_e^{(0)}$ upper‑bounds the true value; edges with $q_e^{(0)} \le 2c$ are certified **safe**, others are marked **heavy**.
2. **Phase 2 – safe‑only extraction.** Remove heavy edges and repeat the extraction; the resulting count $q_e$ counts only genuinely safe paths.

**Lemma 4.1:** $x_e / t \le q_e \le x_e$ (maximality + each greedy path blocks at most $t$ optimal paths).
**Running time:** $O(m^3)$.
**Guarantees preserved:** survival bound and failure bound hold unchanged; expected size degrades by at most a factor $t$, to $O(k \cdot 2^{2k} n^{1+1/k} \log^2 n)$.

---

## Open problems

- Better than a $t$‑approximation for $x_e$ (e.g. fractional‑flow rounding) to tighten the size bound.
- Optimizing the cutoff $\alpha c$ instead of the fixed $2c$.
- Removing the $2^{2k}$ and $\log^2 n$ overheads to reach $\tilde O(n^{1+1/k})$ — or proving they are inherent to independent sampling.
- Independent probability rules **not** based on disjoint‑path counting: what are the true limits of fully independent edge sampling?

---

## Repository layout

```text
.
├── Document/
│   ├── main.tex              # Full XeLaTeX source (Persian, xepersian)
│   ├── main.pdf              # Compiled thesis
│   └── kntuarm.jpg           # University logo used on the title pages
├── Presentations/
│   ├── finalPresentation/    # Final thesis defense slides
│   └── firstPresentation/    # Initial presentation files
├── Proposal/
│   ├── Proposal.tex          # Thesis proposal source
│   └── Proposal.pdf          # Compiled thesis proposal
├── References/               # Key reference papers (PDFs)
└── README.md
```

## Building the PDF

Requirements: a TeX Live distribution with `xelatex` and `xepersian`, plus the fonts **XB Niloofar** and **IranNastaliq** installed system‑wide.

```bash
cd Document
xelatex main.tex
xelatex main.tex   # second pass for the table of contents and references
```

## Key references

- D. Peleg, A. A. Schäffer. *Graph spanners.* J. Graph Theory, 1989.
- I. Althöfer, G. Das, D. Dobkin, D. Joseph, J. Soares. *On sparse spanners of weighted graphs.* DCG, 1993.
- S. Baswana, S. Sen. *A simple and linear time randomized algorithm for computing sparse spanners in weighted graphs.* RSA, 2007.
- M. Elkin, O. Neiman. *Efficient algorithms for constructing very sparse spanners and emulators.* ACM TALG, 2019.
- N. Alon, S. Hoory, N. Linial. *The Moore bound for irregular graphs.* Graphs & Combinatorics, 2002.
- A. Itai, Y. Perl, Y. Shiloach. *The complexity of finding maximum disjoint paths with length constraints.* Networks, 1982.

## Citation

```bibtex
@mastersthesis{ahmadi2026spanner,
  author = {Mohammadreza Ahmadi},
  title  = {Random Edge Sampling for Spanner Construction},
  school = {K. N. Toosi University of Technology, Faculty of Mathematics},
  year   = {2026},
  type   = {B.Sc. Thesis},
  note   = {Supervisor: Hossein Jowhari. In Persian, with English abstract.}
}
```

## Contact

Mohammadreza Ahmadi · mohamadrezahmadi631@gmail.com

---

<div dir="rtl">

## خلاصه فارسی

این پایان‌نامه یک چارچوب تصادفی برای ساخت اسپنرهای ضربی گراف بر پایه‌ی **نمونه‌برداری مستقل یال‌ها** ارائه می‌دهد. با تعریف «مسیرهای امن» و به‌کارگیری فیلتر آستانه‌ی $2c$، وابستگی‌های مخرب میان بقای مسیرها حذف می‌شود و ثابت می‌شود که زیرگراف حاصل با احتمال بالا یک $(2k-1)$-اسپنر با امید ریاضی $O(2^{2k} n^{1+1/k} \log^2 n)$ یال است. با انتخاب $k=\Theta(\sqrt{\log n})$ به خلوتی نزدیک به خطی $n^{1+o(1)}$ می‌رسیم. برای عبور از سختی محاسباتی شمارش دقیق مسیرهای مجزای یالی، یک الگوریتم حریصانه‌ی دوفازی با زمان $O(m^3)$ پیشنهاد شده که تمامی تضمین‌های نظری را حفظ می‌کند.

</div>
