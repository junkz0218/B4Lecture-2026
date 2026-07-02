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
  pre code .token.comment { color: #a6adc8; font-style: normal; }
  pre code .hljs-built_in { color: #F86102; }
  pre code .hljs-comment { color: #a6adc8; font-style: normal; }
  pre code .hljs-number { color: #f9e2af; }
  table {
    width: 100%;
    border-collapse: collapse;
    font-size: 0.82em;
    margin: 7px 0;
  }
  th { background: var(--accent); color: #fff; padding: 6px 10px; text-align: center; }
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
  .cols5 { display: grid; grid-template-columns: repeat(5, 1fr); gap: 8px; align-items: end; }
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
---
<!-- _class: title -->

# VAE による画像生成

*課題 7：変分オートエンコーダの実装と評価*

<div class="meta">
B4 輪講　／　M1 余吾純之介<br>
</div>

---

# 発表の概要

授業で学んだ **VAE** を実装し、**MNIST 手書き数字**の再構成・生成を行った。

| 課題 | 内容 |
|:---:|:---|
| **7-1** | VAE の実装（encoder・reparametrization_trick・decoder・kld・forward） |
| **7-2** | 実行ごとに変わる結果を再現可能にする | 
| **7-3** | z_dim / h_dim / drop_rate / lr を変えて結果を比較 | 

<div class="box-green">

**到達目標**　
① 変分推論が必要な理由を説明できる
② VAEにおけるELBO の意味と損失の導出を説明できる
③ Reparametrization trick とその性質について説明できる

</div>

---

<!-- _class: sec -->

# 1. VAE の目的と変分推論

未知データを生成する確率モデル

---

# VAE の目的と潜在変数モデル

<div class="cols2">

<div class="box-green">

**目的：未知データの生成**

訓練データの分布 $p(x)$ を近似し、
新しいデータを生成する。

- $x$ は**潜在変数 $z$ から生成**されると仮定
- $z \sim p(z){=}N(0,I)$ → デコーダ → $x$

</div>

<div class="box">

**計算上の課題**

周辺尤度は積分で解析的に解くには
限界がある。

$$p(x)=\int p(x|z)\,p(z)\,dz$$

事後分布 $p(z|x)$ も計算困難 
→ 近似事後分布 $q(z|x)$ を導入
（**変分推論**）

</div>

</div>

---

<!-- _class: sec -->

# 2. ELBO と損失関数

対数周辺尤度の下界を最大化する

---

# ELBO の導出

<div class="box">

対数周辺尤度は **ELBO $\mathcal{L}$** と **KL ダイバージェンス**に分解できる。〔式 (2)〕

$$\log p_\theta(x) = D_{KL}\big(q_\phi(z|x)\,\|\,p_\theta(z|x)\big) + \mathcal{L}(\theta,\phi;x)$$

</div>

- $D_{KL}\geq 0$ より $\log p_\theta(x) \geq \mathcal{L}(\theta,\phi;x)$ … ELBO は対数周辺尤度の**下界**〔式 (3)〕
- $\log p_\theta(x)$ は直接最大化できない → **ELBO を最大化**（$q_\phi$ が真の事後分布に近づく）

<div class="box-green">

ELBO を変形〔式 (7)〕：　$\mathcal{L} = \underbrace{\mathbb{E}_{q_\phi}[\log p_\theta(x|z)]}_{\text{再構成項}} - \underbrace{D_{KL}\big(q_\phi(z|x)\,\|\,p_\theta(z)\big)}_{\text{正則化項}}$

損失 $=-\mathcal{L}$　（VAE = AE ＋ 正則化）

</div>

<div class="small">Kingma &amp; Welling, *Auto-Encoding Variational Bayes*, arXiv:1312.6114 (2014). 式番号は同論文に対応。</div>

---

# 損失関数の具体形

<div class="cols2">

<div class="box">

**再構成項**（Appendix C.1）

$$\log p_\theta(x|z)=\sum_{d=1}^{D} x_d\log y_d+(1{-}x_d)\log(1{-}y_d)$$

各画素を Bernoulli に分布すると仮定。
（0=黒, 1=白）
$y$ はデコーダの Sigmoid 出力。

</div>

<div class="box-green">

**正則化項**（Appendix B）

$$-D_{KL}\big(q_\phi(z|x)\,\|\,p_\theta(z)\big)=\tfrac12\sum_{j=1}^{J}\big(1+\log\sigma_j^2-\mu_j^2-\sigma_j^2\big)$$

$q_\phi{=}N(\mu,\sigma^2 I)$, $p_\theta{=}N(0,I)$ より
積分が閉形式で解ける。

</div>

</div>

---

# Reparametrization Trick

<div class="box">

**問題：サンプリングは微分不可**

$z\sim N(\mu,\sigma^2)$ を直接サンプリングする操作は微分できない。
-> **エンコーダに勾配が伝わらない**。

</div>

<div class="box-green">

**解決：Reparametrization trick**（Section 2.4・式 (4)）

ノイズ $\varepsilon$ を外からサンプリングし、$z$ を決定論的な関数で表す。

$$z = g_\phi(\varepsilon,x)=\mu + \varepsilon \odot \exp\!\big(\tfrac12\log\sigma^2\big),\qquad \varepsilon\sim N(0,I)$$

$z$ が $\mu,\sigma$ で微分可能になり、**デコーダからエンコーダまで誤差逆伝播**が可能。

</div>

<div class="small">Kingma &amp; Welling, arXiv:1312.6114 (2014), Section 2.4, Eq. (4).</div>

---

<!-- _class: sec -->

# 3. 実装

VAE_skeleton の #TODO 5 箇所を実装

---

# 実装：`encoder` / `decoder`（符号化・復号化）

**encoder** : 入力画像を全結合層に通し、近似事後分布のパラメータを出力
**decoder** : 潜在表現 $z$ から画像を再構成

```python
def encoder(self, x: torch.Tensor):
    x = x.view(-1, self.x_dim)
    h = torch.relu(self.enc_fc1(x))   # 784 → h_dim
    h = torch.relu(self.enc_fc2(h))   # h_dim → h_dim//2
    mean    = self.enc_fc3_mean(h)    # h_dim//2 → z_dim（μ）
    log_var = self.enc_fc3_logvar(h)  # h_dim//2 → z_dim（logσ²）
    return mean, log_var
def decoder(self, z: torch.Tensor) -> torch.Tensor:
    h = torch.relu(self.dec_fc1(z))   # z_dim → h_dim//2
    h = torch.relu(self.dec_fc2(h))   # h_dim//2 → h_dim
    h = self.dec_drop(h)              # Dropout は dec_fc3 の直前
    return torch.sigmoid(self.dec_fc3(h))  # h_dim → 784, [0,1]
```

<div class="box-green">

encoder は同じ潜在表現 $h$ から $\mu$・$\log\sigma^2$ に分岐〔C.2〕。
decoder 出力は Bernoulli 尤度 $p_\theta(x|z)$ のパラメータ（画素の白確率）〔C.1〕。

</div>

---

# 実装：`reparametrization_trick` / `kld`

**Reparam**：直接サンプリングは微分不可 → 外部ノイズ $\varepsilon$ を使用し微分可能に
**kld**：$q$ と $p(z)$ の KL を閉形式で計算。

```python
def reparametrization_trick(self, mean, log_var):
    eps = torch.randn_like(mean)            # ε ~ N(0, I)
    z   = mean + eps * torch.exp(0.5 * log_var)  # z = μ + ε ⊙ σ
    return z
def kld(self, mean, log_var):
    kl = -0.5 * torch.sum(
        1 + log_var - mean.pow(2) - torch.exp(log_var), dim=1
    )
    return kl.sum()   # dim=1で潜在次元, .sum()でバッチを合算
```

<div class="box-green">

$z=\mu+\varepsilon\odot\exp(\tfrac12\log\sigma^2)$〔Sec 2.4, Eq. (4)〕
$D_{KL}=-\tfrac12\sum_j(1+\log\sigma_j^2-\mu_j^2-\sigma_j^2)$〔Appendix B〕

</div>

---

# 実装：`forward`

`encoder → reparam → decoder` を呼び、ELBO の 2 項 (再構成項 + 正則化項)を返す。

```python
def forward(self, x: torch.Tensor):
    mean, log_var = self.encoder(x)
    z = self.reparametrization_trick(mean, log_var)
    y = self.decoder(z)
    elbo_kl  = -self.kld(mean, log_var)   # 符号反転で ≤ 0
    # Bernoulli 対数尤度（Appendix C.1 Eq.11）
    elbo_rec = torch.sum(
        x * torch.log(y + self.eps) + (1 - x) * torch.log(1 - y + self.eps),
        dim=1,
    ).sum()
    return [elbo_kl, elbo_rec], z, y
```

<div class="small">
損失 L = -(elbo_kl + elbo_rec) を最小化。〔Eq. (3)〕
</div>


---

# 7-2 再現性の確保

<div class="box">

**問題**：`main.py` は実行ごとに結果が変わる。

**乱数が固定されていない箇所**：
重み初期化 ／ train/val 分割（`random_split`）／ バッチ shuffle ／ Reparam の $\varepsilon$（`randn`）

</div>

<div class="box-green">

**解決**：学習・データ生成より前に乱数シードを固定する。

```python
torch.manual_seed(42)
torch.cuda.manual_seed(42)
```

</div>

**検証**：同条件で 2 回実行し、Loss が完全一致することを確認した。

---

<!-- _class: sec -->

# 4. 実験結果

Loss の収束・再構成・潜在空間

---

# Loss 推移と再構成（z_dim = 2）

<div class="cols2">

<div>

![w:700](../images/z2_h400_drop0.2_lr0.001_ep100/lr0.001_loss_curve.png)

</div>

<div>

![w:700](../images/z2_h400_drop0.2_lr0.001_ep50/reconstruction/z2_0.png)

<div class="small center">上段=入力 ／ 下段=再構成</div>

</div>

</div>

<div class="box-green">

train / val とも単調に減少して収束（val ≈ 140）。
過学習は見られず、入力の数字に**追従して再構成**できている。

</div>

---

# 潜在空間と生成多様体（z_dim = 2）

<div class="cols2">

![w:410](../images/z2_h400_drop0.2_lr0.001_ep50/latent_space/z2_0_scatter.png)

![w:410](../images/z2_h400_drop0.2_lr0.001_ep50/lattice_point/z2.png)

</div>

<div class="cols2">

<div class="box">

**潜在空間（散布図）**

数字クラスごとに**塊が分離**。
正則化により $N(0,I)$ 付近へ整列。

</div>

<div class="box-green">

**格子点からの生成**

潜在点を少しずらすと、
生成される数字の形も**わずかに変化**

</div>

</div>

---


# 7-3 実験①：潜在次元 `z_dim`

<div class="cols2">

![w:580](../images/zdim_train_loss.png)

![w:580](../images/zdim_val_loss.png)

</div>

<div class="small center">左：train sweep ／ 右：val sweep（z = 2/5/10/20/50、他のパラメータはデフォルトで固定, epocs=50）</div>

<div class="box-green">

$z_{\dim}$ を増やすほど Loss は低下（表現力↑）。val は **141 → 103** で改善後ほぼ**頭打ち**。

</div>

---

# 再構成の鮮明さと潜在次元 z_dimの関係性

<div class="center">

<div class="small">z = 2　（上=入力 ／ 下=再構成）</div>

![w:500](../images/z2_h400_drop0.2_lr0.001_ep100/reconstruction/z2_39.png)

<div class="small">z = 10</div>

![w:500](../images/z10_h400_drop0.2_lr0.001_ep100/reconstruction/z10_39.png)

<div class="small">z = 50</div>

![w:500](../images/z50_h400_drop0.2_lr0.001_ep100/reconstruction/z50_39.png)

</div>

<div class="box-green">

**z_dim を上げるほど再構成が鮮明**になり、再構成誤差も低下する。
デコーダは期待値（平均像）を出すため生成はややぼやける。

</div>

---

# 7-3 実験②：中間層 `h_dim` ∈ {100,400,800}

<div class="cols3">

![w:580](../images/hdim100_loss.png)

![w:580](../images/hdim400_loss.png)

![w:580](../images/hdim800_loss.png)

</div>

<div class="small center">h_dim = 100 / 400 / 800 (他のパラメータはデフォルトで固定, , epocs=50)</div>

<div class="box-green">

中間層を広げるほど **Encoder/Decoder の表現力**が向上し再構成誤差（train）は低下。
一方 **trainとvalの間のギャップが拡大**し、800 で val の改善が止まる＝**過学習の兆候**。

</div>

---

# 7-3 実験③：Dropout 率 `drop_rate` ∈ {0.0,0.2,0.5}

<div class="cols3">

![w:580](../images/drop00_loss.png)

![w:580](../images/drop02_loss.png)

![w:580](../images/drop05_loss.png)

</div>

<div class="small center"> drop_rate = 0.0 / 0.2 / 0.5（他のパラメータはデフォルトで固定, , epocs=50）</div>

<div class="box-green">

**0.0**：train が val を一貫して下回っており、**軽微な過学習**が起きている
**0.2**：train と val がほぼ密着したまま収束しており、過学習が最も抑制されている
**0.5**：正則化が過剰で trainとval が逆転（**アンダーフィット**）

</div>

---

# 7-3 実験④：学習率 `lr` ∈ {0.01,0.001,0.0001}

<div class="cols2">

![w:580](../images/lr_train_loss.png)

![w:580](../images/lr_val_loss.png)

</div>

<div class="small center">左：train sweep ／ 右：val sweep ／ lr = 0.01/0.001/0.0001</div>

<div class="box-green">

**0.01**：収束は速いが学習曲線が**振動**し、early stopping が ep52 で発動
**0.001**：最良（val≈140、ep98 で最小）
**0.0001**：安定だが**収束が遅く**、val≈146 に達するのに約 150ep を要する。
→ **収束速度と安定性のトレードオフ**。

</div>

---

# まとめ：3 つの到達目標

<div class="box">

**① なぜ変分推論が必要か**
周辺尤度 $p(x){=}\int p(x|z)p(z)\,dz$ と事後分布 $p(z|x)$ は**解析的に解けない**。
→ 計算できる**近似事後分布 $q(z|x)$** を導入し、積分を**最適化問題に置き換える**。

</div>

<div class="box-green">

**② ELBO の意味と損失**
$\log p(x){=}\mathcal{L}{+}D_{KL}(q\|p(z|x)){\geq}\mathcal{L}$。解けない $\log p(x)$ の代わりに**下界 $\mathcal{L}$ を最大化**する。
損失 $=-\mathcal{L}=$ **再構成誤差**（Bernoulli 対数尤度）$+$ **正則化項**（KL）。

</div>

<div class="box">

**③ Reparametrization trick**
$z\sim N(\mu,\sigma^2)$ の直接サンプリングは**微分不可** → $z{=}\mu{+}\varepsilon{\odot}\sigma,\ \varepsilon\sim N(0,I)$ に分離。
$z$ が $\mu,\sigma$ で微分可能になり、**Encoder まで誤差逆伝播**できる。

</div>

---

# まとめ：実装と実験

<div class="box-green">

| 課題 | 取り組み |
|:---:|:---|
| **7-1** | encoder・reparametrization_trick・decoder・kld・forward を実装 | 
| **7-2** | 乱数シード固定 | 
| **7-3** | z_dim / h_dim / drop_rate / lr を比較 | 

</div>

<div class="box">

**今後の課題**

- ハイパーパラメータの**同時最適化**
- t-SNE・PCA による高次元潜在空間の可視化
- FashionMNIST や音声合成（VAE-SiFiGAN）への応用

</div>

