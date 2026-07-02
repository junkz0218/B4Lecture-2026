---
marp: true
theme: default
paginate: true
math: mathjax
style: |
  :root {
    --accent:       #2d6a4f;
    --accent-light: #40916c;
    --sub:          #74c69d;
    --bg:           #f8faf9;
    --card:         #ffffff;
    --code-bg:      #1e1e2e;
    --text:         #1a1a2e;
  }
  section {
    background: var(--bg);
    color: var(--text);
    font-family: 'Arial', 'Meiryo', sans-serif;
    font-size: 28px;
    padding: 25px 50px;
    line-height: 1.5;
    position: relative;
  }
  section::after {
    position: absolute;
    top: 36px;
    right: 56px;
    color: #000000;
    font-size: 0.72em;
    font-weight: bold;
    border: 2px solid #000000;
    border-radius: 50%;
    width: 32px;
    height: 32px;
    display: flex;
    justify-content: center;
    align-items: center;
  }
  h1 {
    color: var(--accent);
    font-size: 1.5em;
    border-bottom: 3px solid var(--accent);
    padding-bottom: 8px;
    margin: 0 0 16px 0;
  }
  h2 {
    color: var(--accent-light);
    font-size: 1.0em;
    margin: 12px 0 5px 0;
  }
  strong { color: var(--accent); }
  ul, ol { margin: 3px 0; padding-left: 1.3em; }
  li { margin: 2px 0; }
  p { margin: 5px 0; }
  code {
    background: #e8f4ee;
    color: #1b4332;
    border-radius: 3px;
    padding: 1px 5px;
    font-size: 0.85em;
  }
  pre {
    background: var(--code-bg);
    border-radius: 8px;
    padding: 12px 16px;
    margin: 7px 0;
    border-left: 4px solid var(--sub);
    overflow: hidden;
  }
  pre code {
    background: transparent;
    color: #cdd6f4;
    padding: 0;
    font-size: 0.76em;
    line-height: 1.5;
  }
  pre code .token.comment {
    color: #a6adc8;
    font-style: normal;
  }
  pre code .hljs-built_in {
    color: #F86102;
  }
  pre code .hljs-comment {
    color: #a6adc8;
    font-style: normal;
  }
  pre code .hljs-number {
    color: #f9e2af;
  }
  table {
    width: 100%;
    border-collapse: collapse;
    font-size: 0.82em;
    margin: 7px 0;
  }
  th {
    background: var(--accent);
    color: #fff;
    padding: 6px 10px;
    text-align: center;
  }
  td {
    border: 1px solid #c8e6d4;
    padding: 6px 10px;
    text-align: center;
    background: var(--card);
  }
  tr:nth-child(even) td { background: #edf7f0; }
  .box {
    background: var(--card);
    border: 1.5px solid var(--sub);
    border-radius: 8px;
    padding: 9px 14px;
    margin: 7px 0;
  }
  .box-green {
    background: #d8f3dc;
    border: 1.5px solid var(--accent);
    border-radius: 8px;
    padding: 9px 14px;
    margin: 7px 0;
  }
  .cols2 { display: grid; grid-template-columns: 1fr 1fr; gap: 14px; }
  .cols3 { display: grid; grid-template-columns: 1fr 1fr 1fr; gap: 10px; }
  .small { font-size: 0.76em; color: #555; }
  .center { text-align: center; }
  section.title {
    background: linear-gradient(135deg, #1b4332 0%, #2d6a4f 60%, #40916c 100%);
    color: #fff;
    justify-content: center;
  }
  section.title h1 {
    color: #fff;
    border-bottom-color: rgba(255,255,255,0.35);
    font-size: 1.85em;
  }
  section.title p { color: rgba(255,255,255,0.88); }
  section.title .meta {
    margin-top: 24px;
    font-size: 0.85em;
    color: rgba(255,255,255,0.7);
    border-top: 1px solid rgba(255,255,255,0.25);
    padding-top: 10px;
  }
  section.sec {
    background: linear-gradient(135deg, #2d6a4f, #52b788);
    color: #fff;
    justify-content: center;
    align-items: center;
    text-align: center;
  }
  section.sec h1 { color: #fff; border-bottom: none; font-size: 2em; }
  section.sec p { color: rgba(255,255,255,0.85); font-size: 1em; }
  section::after {
    position: absolute;
    top: 36px;
    right: 56px;
    color: #000000;
    font-size: 0.72em;
  }
---
<!-- _class: title -->

# HMM による観測系列のモデル推定

*課題 6：Forward / Viterbi アルゴリズムの実装と評価*

<div class="meta">
B4 輪講　／　M1 余吾純之介
</div>

---

# 発表の概要

授業で学んだ HMM の**評価問題**と**復号化問題**を実装し、複数モデルへの帰属推定に応用した。

| 課題 | 内容 | アルゴリズム |
|:---:|:---|:---:|
| **6-1** | 尤度 $P(O\|m_k)$ によるモデル推定 | **Forward** |
| **6-2** | 最尤確率 $\log P^*(O\|m_k)$ によるモデル推定 | **Viterbi** |
| **6-3** | 両アルゴリズムの性能比較と考察 | ― |

<div class="box-green">

**推定の基本方針（両課題共通）**

観測系列 $O$ に対して全モデルのスコアを計算 → 最大スコアのモデルを予測ラベルとする

$$\hat{m} = \arg\max_{m_k}\ \text{score}(O,\ m_k)$$

</div>

---

<!-- _class: sec -->

# 1. 実装の背景

HMM の 3 つの基本問題と今回の位置づけ

---

# HMM の 3 つの基本問題

| 問題 | 既知 | 求めるもの | アルゴリズム | 計算量 |
|:---:|:---|:---|:---:|:---:|
| **評価問題** | $O,\,\mu$ | $P(O\|\mu)$ | **Forward** | $O(N^2 T)$ |
| **復号化問題** | $O,\,\mu$ | $\arg\max_X P(X\|O,\mu)$ | **Viterbi** | $O(N^2 T)$ |
| 推定問題 | $O$ | $\mu=(A,B,\Pi)$ | Baum-Welch | $O(N^2 T)$ |

<div class="box-green">

**今回の応用：モデル推定**

$k$ 個の HMM モデル $\{m_0, \ldots, m_{k-1}\}$ が既知のとき、観測系列 $O$ がどのモデルから生成されたかを推定する。

- **6-1**：Forward で $\log P(O|m_k)$ を算出 → argmax で推定
- **6-2**：Viterbi で $\log P^*(O|m_k)$ を算出 → argmax で推定

</div>

---

# なぜ対数空間で計算するのか

<div class="cols2">

<div class="box">

**問題：アンダーフロー**

HMM のパラメータはすべて確率値 $\in [0,1]$。  
積を繰り返すと値が**急激に 0 へ収束**し、浮動小数点演算の限界を超える。

$$\alpha_i(t) = \prod_{s=1}^{t}(\text{確率値} \leq 1) \to 0$$

</div>

<div class="box-green">

**解決：対数変換**

$$\log P = \sum_t \log(\text{確率値})$$

- 掛け算 → **足し算** に変換
- アンダーフローを防止
- 大小関係は保たれる → $\arg\max$ はそのまま使える

</div>

</div>

```python
# process_data.py：π, A, B を一括で対数変換（系列ループの外で実行）
log_PI = np.log(PI[:, 0] + EPS)   # EPS = 1e-300（log(0) 防止）
log_A  = np.log(A + EPS)
log_B  = np.log(B + EPS)
```

---

<!-- _class: sec -->

# 2. Forward アルゴリズム

評価問題：$P(O \mid m_k)$ の計算

---

# Forward の原理

**前向き変数** $\alpha_i(t)$ を $t=1$ から順に一つ前の時刻の結果を使いながら $t=T$ まで更新する。

<div class="box-green">

$$\alpha_i(t) = P(o_1, \ldots, o_t,\ X_t = i \mid \mu)$$

「時刻 $t$ までの観測を得られて、かつ状態 $i$ にいる同時確率」

</div>

<div class="cols3">

<div class="box">

**① 初期化**
$$\alpha_i(1) = \pi_i \cdot b_i(o_1)$$

</div>

<div class="box">

**② 帰納**
$$\alpha_j(t{+}1) = \!\left(\sum_i \alpha_i(t)\, a_{ij}\right)\! b_j(o_{t+1})$$

</div>

<div class="box">

**③ 終了**
$$P(O|\mu) = \sum_i \alpha_i(T)$$

</div>

</div>

総当たり $O(N^T T)$ → **動的計画法で $O(N^2 T)$ に削減**

---

# Forward の実装（forward.py）

```python
# 初期化（通常確率空間）
alpha[0] = prob_PI * prob_B[:, O[0]]

# 帰納：alpha[t] @ A で Σ_i α_i(t)・a_ij を行列積として一括計算
for t in range(T - 1):
    alpha[t + 1] = (alpha[t] @ A) * prob_B[:, O[t + 1]]

# 最後だけ対数に変換して返す
log_likelihood = np.log(alpha[-1].sum() + EPS)
```

<div class="cols2">

<div class="box">

**`alpha[t] @ A` の意味**

$$(\alpha(t)^\top A)_j = \sum_i \alpha_i(t)\, a_{ij}$$

NumPy 行列積で全状態の $\Sigma$ を一度に計算。

</div>

<div class="box-green">

**設計上の注意**

帰納ステップを行列積 `@ A` で実装するため内部は通常確率空間で計算。最終合計後にのみ `np.log` を適用して対数尤度を返す。（Viterbi は全ステップ対数空間で統一。）

</div>

</div>

---

<!-- _class: sec -->

# 3. Viterbi アルゴリズム

復号化問題：最尤状態系列と $\log P^*(O \mid m_k)$

---

# Viterbi の原理

**本質：Forward の $\sum$（全経路の和）を $\max$（最良経路の選択）に置き換える**

<div class="cols2">

<div>

| | Forward | Viterbi |
|:---:|:---:|:---:|
| 帰納の操作 | $\sum$ | $\max$ |
| 求めるもの | $P(O\|\mu)$ | $\max_X P(X,O\|\mu)$ |
| 追加出力 | なし | 最尤状態系列 $X^*$ |

</div>

<div class="box-green">

**Viterbi 変数**

$$\delta_j(t) = \max_{X_1,\ldots,X_{t-1}} P(\ldots,\, X_t{=}j \mid \mu)$$

**バックトレース用変数**

$$\psi_j(t) = \arg\max_i\ \delta_i(t)\, a_{ij}$$

最良経路の「直前の状態」を記録。末尾から逆順に辿って $X^*$ を復元する。

</div>

</div>

---

# Viterbi の実装（viterbi.py）

```python
# 初期化（対数空間）
delta[0] = log_PI + log_B[:, O[0]]

# 帰納：Forward の "@ A" が "np.max + log_A" に変わる
for t in range(T - 1):
    trans        = delta[t][:, None] + log_A   # (N,N)：全遷移スコア
    psi[t + 1]   = np.argmax(trans, axis=0)    # 最良の前状態を記録
    delta[t + 1] = np.max(trans, axis=0) + log_B[:, O[t + 1]]

# バックトレース：psi を末尾から辿って最尤状態系列を復元
best_path = [int(np.argmax(delta[-1]))]
for t in range(T - 1, 0, -1):
    best_path.append(int(psi[t][best_path[-1]]))
best_path.reverse()
```

<div class="box">

`trans[i, j] = δᵢ(t) + log aᵢⱼ` ← 全 $i \to j$ の対数スコアを $(N \times N)$ 行列で一括計算  
`np.max(axis=0)` で各 $j$ への最良経路スコアを選択 ／ `psi` にその前状態を記録

</div>

---

<!-- _class: sec -->

# 4. モデル推定への応用

スコア行列と argmax による推定フロー

---

# 推定フロー（score_sequences）

`score.py` の `score_sequences()` が中核。全系列 × 全モデルのスコアを計算する。

```
         m_0     m_1    ...   m_{k-1}
  O_0 [ -12.3   -45.1   ...   -18.7  ]
  O_1 [ -33.8    -9.2   ...   -41.0  ]  ← scores[i, m]
  O_p [ -20.1   -22.4   ...   -11.5  ]
            ↓  np.argmax(scores, axis=1)
  予測ラベル: [0,  1,  ...,  k-1]
```

<div class="cols2">

<div class="box">

**Forward の場合**

`score =` $\log P(O \mid m_k)$（対数尤度）

全経路の確率の和をスコアとして使う

</div>

<div class="box-green">

**Viterbi の場合**

`score =` $\log P^*(O \mid m_k)$（最尤経路の対数確率）

最良 1 経路の確率をスコアとして使う

</div>

</div>

---

# Left-to-Right / Ergodic の判別（classify.py）

<div class="cols2">

<div class="box-green">

**Left-to-Right HMM**

状態は左→右にのみ遷移。  
遷移行列 $A$ が**上三角行列**になる。

$$A = \begin{pmatrix} * & * & * \\ 0 & * & * \\ 0 & 0 & * \end{pmatrix}$$

</div>

<div class="box">

**Ergodic HMM**

全状態間の遷移が可能。  
$A$ の下三角成分が 0 でない。

$$A = \begin{pmatrix} * & * & * \\ * & * & * \\ * & * & * \end{pmatrix}$$

</div>

</div>

```python
def is_left_to_right(A, eps=1e-5):
    return np.all(np.tril(A, k=-1) <= eps)   # 下三角がすべて ≈ 0

def is_ergodic(A, eps=1e-5):
    return np.all(A > eps)                    # 全成分が 0 でない
```

---

<!-- _class: sec -->

# 5. 実験結果

data1 〜 data4 における推定精度・混同行列・最尤状態系列

---

# 6-1・6-2 実験結果（精度・計算時間）

| データセット | HMM 種別 | $k$ | $p$ | Forward 精度 | Viterbi 精度 | Fwd 時間 | Vtb 時間 |
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
| data1 | Left-to-Right | 3 | 300 | **0.867** | 0.857 | 0.056s | 0.081s |
| data2 | Ergodic | 3 | 300 | **0.703** | 0.427 | 0.043s | 0.108s |
| data3 | Left-to-Right | 5 | 500 | **0.964** | 0.958 | 0.149s | 0.400s |
| data4 | Ergodic | 5 | 500 | **0.584** | 0.420 | 0.158s | 0.442s |

<div class="box-green">

- **Left-to-Right（data1・3）**：両アルゴリズムの精度差が小さい（≦ 1.0pt）
- **Ergodic（data2・4）**：Forward が Viterbi を大幅に上回る（+16〜+28pt）
- 計算時間：Viterbi は Forward の約 **2〜2.5 倍**

</div>

---

# 混同行列：data1（Left-to-Right, k=3）

<div class="cols2">

![w:500](./0514/data1_forward.png)

![w:500](./0514/data1_viterbi.png)

</div>

<div class="small center">

**Forward: 86.7%　／　Viterbi: 85.7%**（差: 1.0pt）　計算時間：Forward 0.056s　／　Viterbi 0.081s

</div>

---

# 混同行列：data2（Ergodic, k=3）

<div class="cols2">

![w:500](./0514/data2_forward.png)

![w:500](./0514/data2_viterbi.png)

</div>

<div class="small center">

**Forward: 70.3%　／　Viterbi: 42.7%**（差: 27.6pt）　計算時間：Forward 0.043s　／　Viterbi 0.108s

</div>

---

# 混同行列：data3（Left-to-Right, k=5）

<div class="cols2">

![w:500](./0514/data3_forward.png)

![w:500](./0514/data3_viterbi.png)

</div>

<div class="small center">

**Forward: 96.4%　／　Viterbi: 95.8%**（差: 0.6pt）　計算時間：Forward 0.149s　／　Viterbi 0.400s

</div>

---

# 混同行列：data4（Ergodic, k=5）

<div class="cols2">

![w:500](./0514/data4_forward.png)

![w:500](./0514/data4_viterbi.png)

</div>

<div class="small center">

**Forward: 58.4%　／　Viterbi: 42.0%**（差: 16.4pt）　計算時間：Forward 0.158s　／　Viterbi 0.442s

</div>

---

# 最尤状態系列（Viterbi 実測値）

<div class="cols2">

<div class="box-green">

**Left-to-Right（data1・3）**

状態番号が**単調増加**し、一度進んだ状態には戻らない

```
# data1（T=20）
系列0 (真:m1, 推定:m1):
  [0,1,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3,3]
系列2 (真:m0, 推定:m0):
  [0,2,2,2,2,2,3,3,3,3,3,3,3,3,3,3,3,3,3,3]
# data3（T=30）
系列0 (真:m3, 推定:m3):
  [0,4,4,4,4,4,4,...,4]
```

→ $s_0 \to s_j \to s_{N-1}$ への一方向構造を確認

</div>

<div class="box">

**Ergodic（data2・4）**

状態番号が**前後に行き来**し、多様な経路をたどる

```
# data2（T=20）
系列1 (真:m1, 推定:m1):
  [0,0,2,1,1,3,0,0,0,0,0,0,0,0,0,0,0,0,0,0]
# data4（T=30）
系列0 (真:m2, 推定:m2):
  [3,1,0,2,3,1,1,0,3,1,1,1,0,3,4,
   1,3,4,4,4,1,1,0,3,1,0,2,3,4,4]
```

→ 全状態間を自由に遷移する構造を確認

</div>

</div>

---

<!-- _class: sec -->

# 6. 考察

Forward vs Viterbi　／　Left-to-Right vs Ergodic

---

# 考察①：推定精度と計算時間

<div class="cols2">

<div class="box">

**精度：Left-to-Right では同程度、Ergodic では Forward が優位**

Forward は $\sum_X P(X,O|\mu)$（全経路の和）  
Viterbi は $\max_X P(X,O|\mu)$（最良1経路）

どちらも argmax でモデルを選ぶ点は同じだが、Ergodic では尤度の推定精度の差が推定結果に表れる。

</div>

<div class="box-green">

**計算時間：Viterbi が約 2〜2.5 倍遅い**

理論計算量は両者とも $O(N^2 T)$ だが Viterbi には追加処理がある：

- `psi` 行列（$T \times N$）の記録・保持
- バックトレース $O(T)$ の実行
- メモリ使用量も大きい

→ 定数倍のオーバーヘッドが生じる

</div>

</div>

---

# 考察②：Left-to-Right HMM vs Ergodic HMM

<div class="cols2">

<div class="box-green">

**Left-to-Right HMM**

遷移行列が上三角 → 有効な経路数が大幅に限定される。最尤経路（1本）が全経路の和を良く近似する。

$$\sum_X P(X,O|\mu) \approx \max_X P(X,O|\mu)$$

→ **Forward と Viterbi の精度差が小さい**  
（data1: 1.0pt、data3: 0.6pt）

</div>

<div class="box">

**Ergodic HMM**

全状態間を行き来できるため有効経路数が指数的に多い。最尤1経路だけでは全体を代表しにくく、混同行列でも特定モデルへの集中が起きる。

$$\sum_X P(X,O|\mu) \gg \max_X P(X,O|\mu)$$

→ **Forward が尤度をより正確に推定**  
（data2: 27.7pt 差、data4: 16.4pt 差）

</div>

</div>

<div class="box">

**結論**　精度優先 → **Forward** ／ 状態系列も必要 → **Viterbi** ／ Left-to-Right 主体 → どちらでも可

</div>

---

# まとめ

<div class="box-green">

**実装のまとめ**

| | Forward | Viterbi |
|:---:|:---|:---|
| **計算対象** | 全経路の和：$\sum_X P(X,O\|\mu)$ | 最良経路：$\max_X P(X,O\|\mu)$ |
| **帰納の核** | `alpha[t] @ A`（行列積・通常確率空間） | `np.max(delta[:,None] + log_A)`（対数空間） |
| **追加出力** | なし | 最尤状態系列（バックトレース） |

</div>

<div class="box">

**考察のまとめ**

- **Left-to-Right HMM**：有効経路数が限定的 → 最尤経路が全経路の和を良く近似 → 両者の精度差が小さい
- **Ergodic HMM**：有効経路数が指数的に多い → Forward が Viterbi を大幅に上回る（最大 +28pt）
- 計算時間は Viterbi が約 **2〜2.5 倍**（`psi` 記録 + バックトレースのオーバーヘッド）

</div>
