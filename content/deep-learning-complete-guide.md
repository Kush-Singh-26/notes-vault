---
title: "Deep learning — complete exam guide"
description: "Full-course DL notes: mid-sem foundations, CNN, sequences, latent models, autoencoders, transfer learning, generative models, transformers. Worked numericals, formula sheet and memory drills."
---

# Deep Learning — Complete Exam Guide

*Built from the KIIT CS40001 slide decks (Lecture Slides I, II, III), the mid-sem deck, and your handwritten class notes. Covers the whole syllabus: mid-sem foundations in full and the post-mid half in full. Every section has four parts: the explanation, the formula, a worked numerical, and a `Remember` / `Exam` line.*

---

## 0. How to use this guide

**Symbols.** ★★★ = almost certain to be asked (derivation or numerical). ★★ = short-answer or definition. ★ = one-line definition, comparison table, or "no maths needed".

| Part | Topic | Most likely question form | Priority |
|---|---|---|---|
| I | Perceptron, XOR, MLP forward pass | numerical + "why hidden layer" | ★★★ |
| I | Backpropagation | derivation, algorithm, 1–2 iterations | ★★★ |
| I | Gradient descent, optimizers | define + algorithm + α + variants | ★★★ |
| I | Activation functions | formula + graph + range + pros/cons | ★★★ |
| I | Regularization, bias–variance | short answers + L1/L2 equations | ★★ |
| II | CNN | architecture diagram + feature-map numerical + parameter count | ★★★ |
| II | Efficient convolution (FFT, depthwise) | multiplication-count numerical | ★★★ |
| II | Pooling, geometry, receptive field | formula + numerical | ★★ |
| III | RNN forward pass | numerical, 1–2 iterations | ★★★ |
| III | BPTT, vanishing gradient | derivation + explanation | ★★★ |
| III | LSTM | draw cell + all gate equations | ★★★ |
| III | Bi-LSTM, seq2seq | diagram + comparison | ★★ |
| IV | PCA / PPCA / FA / ICA | definitions + differences table | ★★ |
| V | Autoencoders | architecture + types + where each is used | ★★★ |
| VI | Pre-training, transfer, domain adaptation | pipeline diagram + why it helps | ★★ |
| VII | VAE, GAN | ELBO / minimax objective + differences | ★★★ |
| VIII | Transformers | attention formula + why √d_k + block diagram | ★★★ |
| IX | Keras, sampling, DeepDream, NST | code/short answers | ★★ |

**Exam habits that earn marks.**
1. Write the formula first, substitute numbers second, state units/shapes third.
2. Every architecture question: draw the diagram even if not asked.
3. Every numerical: show two iterations (iteration 2 is what separates full marks from half).
4. Name the shapes at every layer (`(N, C, H, W)`) — it catches most errors.
5. If the question says "explain", give the *why* in one sentence after the *what*.

**The 7 diagrams you must be able to draw from a blank page.** These cover roughly 60% of the paper.

```
1. MLP with forward + backward arrows (backprop)   5. Autoencoder (hourglass)
2. CNN pipeline: conv → ReLU → pool → … → FC       6. GAN (G and D opposed)
3. LSTM cell with 3 gates + cell state line        7. Transformer block (MHA → Add&Norm → FFN → Add&Norm)
4. VAE (encoder → μ,σ → z → decoder)
```

---

# PART I — Foundations (mid-sem)

## 1. What deep learning is

### 1.1 The hierarchy

```
Artificial Intelligence  ⊃  Machine Learning  ⊃  Representation Learning  ⊃  Deep Learning
```

- **AI** — any technique that makes machines imitate intelligent behaviour (including rule-based systems).
- **ML** — algorithms that improve at a task T with experience E, measured by P (Mitchell). Instead of programming rules, you show examples and the machine fits a model.
- **Representation learning** — the model learns the features itself instead of a human hand-designing them.
- **Deep learning** — representation learning with *many composed layers*, so the features form a hierarchy.

**The hierarchy of features** (this sentence is worth a mark): a digit image → layer 1 learns edges and blobs → layer 2 learns corners and strokes → layer 3 learns digit parts → output layer learns whole digits. Lower layers = generic (reusable), upper layers = task-specific.

**Formally**, a deep feedforward network is a composition:

$$
\begin{aligned}
h^{(1)} &= g(W_1 x + b_1) \\
h^{(2)} &= g(W_2 h^{(1)} + b_2) \\
&\vdots \\
\hat{y} &= g(W_L h^{(L-1)} + b_L)
\end{aligned}
$$

so the whole network is one function $f(x; \theta)$ with $\theta = \{W_1, b_1, \dots, W_L, b_L\}$.

### 1.2 What makes DL different from ML ★ (draw the graph)

| Factor | Machine learning | Deep learning |
|---|---|---|
| Data needed | works on small/medium data | needs a lot; performance scales with data |
| Features | manual engineering | learned automatically |
| Hardware | CPU is enough | GPU/TPU needed (huge matrix products) |
| Training time | minutes–hours | hours–weeks |
| Interpretability | higher | lower ("black box") |
| Best when | small data, tabular, need explanation | images, speech, text, big data |

**The graph to draw.** x-axis = amount of data, y-axis = performance. The ML curve rises quickly and **plateaus**; the DL curve starts lower but keeps rising and **crosses above** ML as data grows. Label the crossing point "DL overtakes ML here". Add the sentence: *for small datasets ML is preferable, because a high-capacity DL model overfits.*

**Why ML plateaus and DL does not.** Classical ML models have limited capacity — after a point extra data cannot be exploited because the hypothesis space is too small. DL models have millions of parameters and can absorb more data by learning finer features; they were *data-limited*, not capacity-limited. So more data keeps helping (up to the point where the model is finally saturated).

### 1.3 Why DL only became popular after 2012 ★

Give two or three of these, with one line each:

| Enabler | Effect |
|---|---|
| **Data** | ImageNet (14M labelled images), COCO, web text — DL needs scale |
| **Hardware** | GPUs/TPUs make millions of matrix multiplications feasible |
| **Algorithms** | ReLU (fixes vanishing gradient), dropout, better initialization, batch norm |
| **Frameworks** | TensorFlow, PyTorch, Keras — lowered the engineering barrier |
| **Architectures** | AlexNet, VGG, ResNet, then Transformers |
| **Community** | open source, MOOCs, Kaggle, GitHub |
| **Results** | AlexNet won ImageNet 2012 by a large margin; DL has won since |

The 1980s already had backpropagation and MLPs. The blockers were: no large labelled datasets, no cheap parallel compute, and vanishing gradients in deep stacks.

### 1.4 Timeline ★ (one line each)

| Year | Event |
|---|---|
| 1943 | McCulloch–Pitts neuron — first mathematical neuron, **no learning** |
| 1958 | Rosenblatt's **perceptron** — first learning rule |
| 1969 | Minsky & Papert, *Perceptrons* — single layer cannot do XOR → first "AI winter" |
| 1986 | Backpropagation popularised (Rumelhart, Hinton, Williams) |
| 1989–98 | CNNs: LeNet-5 for digits (1998) |
| 2012 | **AlexNet** wins ImageNet (GPU + ReLU + dropout) |
| 2014 | GAN, VAE |
| 2015 | ResNet (152 layers, skip connections) |
| 2017 | **Transformer** ("Attention is All You Need") |
| 2018+ | BERT, GPT family, diffusion models |

> Your lecture dates the AlexNet win as 2013; the paper and the competition are 2012. Write "2012–13" in the exam and neither examiner can mark it wrong.

### 1.5 Applications ★

| Application | What the network does |
|---|---|
| Netflix / Spotify recommendations | predicts what you will watch/listen to next from your history and similar users |
| Snapchat / Instagram filters | CNN detects facial landmarks in real time, overlays graphics |
| Image classification, detection, segmentation | CNN |
| Speech recognition | spectrogram → CNN/RNN/Transformer |
| Machine translation, chatbots, summarisation | sequence models / Transformers |
| Medical imaging, drug discovery | CNN / graph networks |
| Fraud detection, credit scoring | dense networks on tabular data |
| Predictive maintenance, load forecasting | RNN/LSTM on sensor time series |
| Self-driving, robotics, control | CNN + RL |

### 1.6 Types of network and the data they fit ★

```
MLP / ANN          → tabular, fixed-size vectors
CNN                → grid data: images, spectrograms, video frames
RNN / LSTM / GRU   → sequences: text, audio, sensor traces
Autoencoder        → compression, denoising, anomaly detection
Transformer        → sequences with long-range dependencies (text, code, images as patches)
GAN / VAE          → generating new data
```

**Rule to memorise:** *"grid → conv, order → recurrent, long-range → attention, generate → generative."*

`Remember:` the hierarchy is **A-M-R-D** (AI, ML, Representation, Deep) — "A Machine Really Delivers".

`Exam:` "Differentiate ML and DL with a graph." Draw the two curves, give the table's three most important rows (features, data, hardware), and finish with the small-data rule.

---

## 2. From the biological neuron to the perceptron

### 2.1 Biological vs artificial neuron ★ (diagram question)

| Biological | Artificial |
|---|---|
| Dendrites | input connections $x_1 \dots x_n$ |
| Synapse strength | weight $w_i$ |
| Cell body / nucleus | summation $\sum w_i x_i$ + bias |
| Axon hillock threshold | activation function `f` |
| Action potential (fires / not) | output `y` |
| Synaptic plasticity | learning (weight update / backprop) |

**Similarities:** many simple units, massively parallel, learn by adjusting connection strengths, robust to unit failure.
**Differences:** the brain has ~10¹¹ neurons, is analogue, spiking, energy-efficient (~20 W) and learns from few examples; an ANN is a few million units, synchronous, numeric, needs labelled data and a GPU.

**Diagram to draw:** left = neuron labelled dendrite → soma → axon → terminal; right = perceptron with $x_1 \dots x_n$, weights, $\sum$, bias `b`, activation `f`, output `y`. Draw one arrow from "synapse" to "weight".

### 2.2 McCulloch–Pitts (MP) neuron

- Binary inputs $x_i \in \{0,1\}$, binary output $y \in \{0,1\}$.
- Computes $z = \sum w_i x_i$, fires if $z \geq t$ (threshold), else 0.

$$
y = \begin{cases}
1 & \text{if } \sum w_i x_i \geq t \\
0 & \text{otherwise}
\end{cases}
$$

- Weights may be **excitatory** (+) or **inhibitory** (−).
- **No learning** — the weights and threshold are set by hand to implement a logic function. The MP neuron can only be *analysed*, not trained.

**AND gate with an MP neuron** ($w_1 = w_2 = 1$, $t = 2$):

| x1 | x2 | z | y (z ≥ 2) |
|---|---|---|---|
| 0 | 0 | 0 | 0 |
| 0 | 1 | 1 | 0 |
| 1 | 0 | 1 | 0 |
| 1 | 1 | 2 | 1 |

**OR**: $w = (1,1)$, $t = 1$. **NOT**: $w = -1$, $t = 0$ (input 0 → z=0 ≥ 0 → 1). **NAND**: $w = (-1,-1)$, $t = -1$.

**Why AND is learnable by one neuron:** the classes are separated by the line $x_1 + x_2 = 1.5$, so AND/OR/NOT/NAND are **linearly separable**. XOR is not — see §2.4.

### 2.3 The perceptron ★

The perceptron is the MP neuron **plus a learning rule**.

$$
\begin{aligned}
z &= w_1 x_1 + w_2 x_2 + \dots + w_n x_n + b = w\cdot x + b \\
y &= \begin{cases} 1 & \text{if } z \geq 0 \\ 0 & \text{otherwise} \end{cases} \qquad (\text{step activation})
\end{aligned}
$$

Matrix form: $z = [w_1\ w_2\ \dots\ w_n\ b]\cdot[x_1\ x_2\ \dots\ x_n\ 1]^\top$. The `1` is a dummy input so the bias becomes just another weight.

**Geometry.** $w\cdot x + b = 0$ is the decision boundary:

| Inputs | Boundary | Name |
|---|---|---|
| 2 | $Ax + By + C = 0$ | line |
| 3 | $Ax + By + Cz + D = 0$ | plane |
| ≥4 | $w\cdot x + b = 0$ | hyperplane |

Points on one side → class 1, other side → class 0.

**Perceptron learning rule** (the exam formula):

$$
\begin{aligned}
w_{\text{new}} &= w_{\text{old}} + \alpha (t - o)\, x \\
b_{\text{new}} &= b_{\text{old}} + \alpha (t - o)
\end{aligned}
$$

$t$ = target, $o$ = predicted output, $\alpha$ = learning rate. If $t = o$ the update is zero — **it only learns from mistakes**. Convergence is guaranteed (Novikoff) if the data is linearly separable and α is small enough.

**Limitation.** A single perceptron is a **linear classifier**. It cannot represent XOR, or any non-linearly-separable function. Also the step function is not differentiable, so it cannot be trained with gradient descent — this is the motivation for the sigmoid and for backpropagation.

### 2.4 Proof that one perceptron cannot do XOR ★★★ (frequently asked)

XOR truth table: $y = 1$ for (0,1) and (1,0); $y = 0$ for (0,0) and (1,1).

Assume a perceptron exists: $y = 1 \iff w_1 x_1 + w_2 x_2 + b \geq 0$. Then

$$
\begin{aligned}
(0,0) \to 0{:}&\quad b < 0 &&\text{(i)} \\
(0,1) \to 1{:}&\quad w_2 + b \geq 0 &&\text{(ii)} \\
(1,0) \to 1{:}&\quad w_1 + b \geq 0 &&\text{(iii)} \\
(1,1) \to 0{:}&\quad w_1 + w_2 + b < 0 &&\text{(iv)}
\end{aligned}
$$

Add (ii) and (iii): $w_1 + w_2 + 2b \geq 0 \implies w_1 + w_2 + b \geq -b$. From (i), $b < 0$, so $-b > 0$, hence $w_1 + w_2 + b > 0$. That contradicts (iv). Therefore no such $w_1, w_2, b$ exist. ∎

Geometrically: one line cannot put (0,1) and (1,0) on one side and (0,0), (1,1) on the other.

**Fix:** add a **hidden layer**. The hidden layer maps the input into a new space where the classes *are* linearly separable; the output neuron then solves it.

`Remember:` XOR needs **two lines**, i.e. two hidden neurons. AND/OR/NOT need **zero** hidden neurons.

`Exam:` "Why can't a single perceptron learn XOR? Prove it." Write the four inequalities and the contradiction — that *is* the answer. Then add "adding a hidden layer fixes it" and draw the 2-2-1 network.

---

## 3. MLP: architecture, notation, forward propagation

### 3.1 Architecture

```
inputs → [hidden layer 1] → [hidden layer 2] → ... → [output layer]
```

- **Input layer** — holds the data; no computation, no parameters.
- **Hidden layers** — affine transform then non-linearity; they learn intermediate representations.
- **Output layer** — task-specific activation: sigmoid (binary), softmax (multiclass), linear (regression).

**Why hidden units matter (slide wording):** they *learn intermediate representations*, *transform the input space*, *make complex decision boundaries possible*, and *enable hierarchical feature learning*.

**Why non-linearity is mandatory ★ (2-mark proof).** Consider two layers with no activation:

$$
h = W_1 x, \qquad y = W_2 h = W_2 W_1 x = W' x, \qquad \text{where } W' = W_2 W_1
$$

A composition of linear maps is one linear map. So depth buys nothing — the network is still a linear classifier, and can still never do XOR. The non-linearity `g` breaks this: $g(W_2 g(W_1 x))$ is not collapsible.

### 3.2 Notation

| Symbol | Meaning |
|---|---|
| $a^{(l)}$ | activation (output) of layer `l`; $a^{(0)} = x$ |
| $z^{(l)}$ | pre-activation: $W^{(l)} a^{(l-1)} + b^{(l)}$ |
| $W^{(l)}$ | weight matrix, shape $(n^{(l-1)}, n^{(l)})$ |
| $b^{(l)}$ | bias vector, shape $(n^{(l)},)$ |
| $\delta^{(l)}$ | error signal at layer `l` = $\partial L/\partial z^{(l)}$ |
| $\theta$ | all trainable parameters |

Your class notes write $w^{k}_{ij}$ = weight in layer `k` from node `i` (previous layer) to node `j` (this layer), and `O` for activation. Same thing, different letters.

### 3.3 Counting trainable parameters ★★

For layer sizes $n_0 \to n_1 \to n_2 \to \dots \to n_L$:

$$
\begin{aligned}
\text{weights} &= n_0\cdot n_1 + n_1\cdot n_2 + \dots + n_{L-1}\cdot n_L \\
\text{biases} &= n_1 + n_2 + \dots + n_L \\
\text{total} &= \text{weights} + \text{biases}
\end{aligned}
$$

**Example A** (4 → 3 → 2 → 1): weights $4\cdot 3 + 3\cdot 2 + 2\cdot 1 = 12+6+2 = 20$; biases $3+2+1 = 6$; **total 26**.
**Example B** (2 → 2 → 1): weights $4+2 = 6$; biases $2+1 = 3$; **total 9**.

> Your notes said "20 parameters" — that counts weights only. In the exam write "20 weights + 6 biases = 26 trainable parameters" and you cannot be penalised.

**Memory hook:** *every connection is a weight, every node except the inputs is a bias.*

### 3.4 Forward propagation, matrix form ★★★

For every layer:

$$
\begin{aligned}
z^{(l)} &= W^{(l)\top} a^{(l-1)} + b^{(l)} \\
a^{(l)} &= g\big(z^{(l)}\big)
\end{aligned}
$$

**Shapes.** If $a^{(l-1)}$ is $n^{(l-1)} \times 1$, then $W^{(l)}$ must be $n^{(l-1)} \times n^{(l)}$, $b^{(l)}$ is $n^{(l)} \times 1$, $z^{(l)}$ is $n^{(l)} \times 1$. Write the shapes in the exam; it prevents transposes mistakes.

**Numerical (your notes: 4 → 3 → 2 → 1, all weights and biases = 1, sigmoid everywhere, x = (10,20,30,40)).**

Layer 1: $z = 10+20+30+40+1 = 101$ ⟹ $O_{1j} = \sigma(101) \approx 1$ (for all 3 nodes).
Layer 2: $z = 1\cdot 1 + 1\cdot 1 + 1\cdot 1 + 1 = 4$ ⟹ $O_{2j} = \sigma(4) = 1/(1+e^{-4}) = 0.9820$.
Output: $z = 0.9820 + 0.9820 + 1 = 2.9640$ ⟹ $\hat{y} = \sigma(2.9640) = 0.951$.

The other rows $(30,10,20,53)$ and $(5,10,15,25)$ also saturate layer 1 (σ of ≥ 120) → all give $\hat{y} \approx 0.951$. That saturation is exactly why real networks need normalization.

**Vectorized over a batch:** stack examples as rows, $A^{(l)} = g(A^{(l-1)} W^{(l)} + b^{(l)})$ with $A^{(l-1)}$ of shape $(\text{batch}, n^{(l-1)})$. Weight matrix then has shape $(n^{(l-1)}, n^{(l)})$ — the transpose disappears because the batch is on the left.

### 3.5 Learning XOR with a 2-2-1 MLP ★★★ (numerical, exactly as in your notes)

XOR decomposes as $y = (x_1 \text{ AND NOT } x_2) \text{ OR } (\text{NOT } x_1 \text{ AND } x_2)$, i.e. two hidden perceptrons $z_1 = x_1\cdot\bar{x}_2$ and $z_2 = \bar{x}_1\cdot x_2$, then $y = z_1 \text{ OR } z_2$.

Setup: step activation $f(v) = 1 \text{ if } v \geq \theta \text{ else } 0$, $\theta = 1$, $\alpha = 1.5$, all weights start at 1.

**Step 1 — learn $z_1$ (weights $w_{11}, w_{21}$).**
Start $w_{11} = w_{21} = 1$.
- (0,0): $f(0) = 0$ ✓
- (0,1): $f(0\cdot 1 + 1\cdot 1) = f(1) = 1$, target 0 ✗ ⟹

$$
w_{11} = 1 + 1.5(0-1)\cdot 0 = 1, \qquad w_{21} = 1 + 1.5(0-1)\cdot 1 = -0.5
$$

- Recheck with $(1, -0.5)$: (0,0)→0 ✓, (0,1)→f(−0.5)=0 ✓, (1,0)→f(1)=1 ✓, (1,1)→f(0.5)=0 ✓. **Learned.**

**Step 2 — learn $z_2$ (weights $w_{12}, w_{22}$).** Same procedure gives $w_{12} = -0.5,\ w_{22} = 1$.

**Step 3 — output neuron** with $v_1 = v_2 = 1$, $\theta = 1$: (z1,z2) = (0,0),(0,1),(1,0),(0,0) → 0,1,1,0 ✓. No update needed.

**Final network:** $x_1 \to Z_1 : 1$, $x_2 \to Z_1 : -0.5$, $x_1 \to Z_2 : -0.5$, $x_2 \to Z_2 : 1$, $Z_1 \to Y : 1$, $Z_2 \to Y : 1$.

| x1 | x2 | z1 | z2 | y |
|---|---|---|---|---|
| 0 | 0 | 0 | 0 | 0 |
| 0 | 1 | 0 | 1 | 1 |
| 1 | 0 | 1 | 0 | 1 |
| 1 | 1 | 0 | 0 | 0 |

**Check your other gates:** AND, OR, NAND, NOR are linearly separable → one perceptron. XNOR behaves like XOR → needs the hidden layer.

`Remember:` hidden neuron 1 = "x1 but not x2", hidden neuron 2 = "x2 but not x1", output = OR of the two. Say that sentence in the exam and the marker knows you understand.

`Exam:` "Design and train an MLP for XOR." Draw the 3-neuron network, write the initial weights, show the two update calculations, then the truth table. That is worth full marks.

### 3.6 XOR in closed form: 2 ReLU hidden units + 1 linear output ★★ (memorise these weights)

The §3.5 answer *trains* the network with the perceptron rule. This is the **finished network written down directly** — no training loop, exact weights, exact answer. Use it when the question says "give a network that computes XOR" rather than "train one".

**Weights (all six, exactly):**

| Layer | Unit | Weights | Bias | Activation |
|---|---|---|---|---|
| hidden 1 | $h_1$ | $(1, 1)$ | $0$ | ReLU |
| hidden 2 | $h_2$ | $(1, 1)$ | $-1$ | ReLU |
| output | $y$ | $(1, -2)$ | $0$ | **linear (identity)** |

$$
\begin{aligned}
h_1 &= \operatorname{ReLU}(x_1 + x_2) = \max(0,\; x_1 + x_2) \\
h_2 &= \operatorname{ReLU}(x_1 + x_2 - 1) = \max(0,\; x_1 + x_2 - 1) \\
y &= h_1 - 2h_2
\end{aligned}
$$

**ASCII diagram (draw this):**

```
        input              hidden (2 ReLU)                  output (linear)
                   ┌──────────────────────────────┐
  x1 ───────┬──────┤ z1 = x1 + x2 ,    b1 = 0     │  h1 ──(w= 1)──┐
            │      └──────────────────────────────┘               │
            │      ┌──────────────────────────────┐               ├──► y = h1 − 2h2
            ├──────┤ z2 = x1 + x2 − 1, b2 = −1    │  h2 ──(w=−2)──┘      (b = 0)
  x2 ───────┘      └──────────────────────────────┘
```

**Forward pass, all four inputs:**

| $x_1$ | $x_2$ | $z_1$ | $h_1$ | $z_2$ | $h_2$ | $y = h_1 - 2h_2$ | XOR |
|---|---|---|---|---|---|---|---|
| 0 | 0 | 0 | 0 | −1 | 0 | 0 | 0 ✓ |
| 0 | 1 | 1 | 1 | 0 | 0 | 1 | 1 ✓ |
| 1 | 0 | 1 | 1 | 0 | 0 | 1 | 1 ✓ |
| 1 | 1 | 2 | 2 | 1 | 1 | 0 | 0 ✓ |

**Why it works — one line of algebra the examiner likes.** Let $s = x_1 + x_2$. Then $h_1 = s$ and $h_2 = \max(0, s-1)$, so

$$
y = s - 2\max(0, s-1) =
\begin{cases}
s, & s \le 1 \\
2 - s, & s \ge 1
\end{cases}
\;=\; 1 - \lvert s - 1 \rvert = 1 - \lvert x_1 + x_2 - 1\rvert
$$

That is a **tent**: it rises to 1 at $s = 1$ (inputs $(0,1)$ and $(1,0)$) and falls back to 0 at the ends $s=0$ and $s=2$ (inputs $(0,0)$ and $(1,1)$).

```
 y
 1 ┤            /\
   │          /    \
   │        /        \
 0 ┤──────/────────────\──────  s = x1 + x2
       0       1        2
     (0,0)  (0,1)/(1,0) (1,1)
```

**Each ReLU is one hinge.** A ReLU has exactly one kink, so one hidden unit can only bend the plane once ($s=0$). XOR needs **two** kinks ($s=0$ for $h_1$, $s=1$ for $h_2$) — that is the real reason two hidden units are necessary and sufficient. The output stays **linear** because the kinks were already supplied by the hidden layer; adding an output non-linearity would be redundant (a sigmoid on top also works and gives near-0/near-1 values).

**Parameter count:** weights $2\cdot2 + 2\cdot1 = 6$, biases $2 + 1 = 3$, **total 9** — the same 2-2-1 network as Example B in §3.3.

`Exam:` "Give an MLP with ReLU hidden units that computes XOR." Write the three equations and the table. `Exam:` "Why two hidden neurons?" One kink per ReLU; XOR is a tent with two kinks.

---

## 4. Loss functions

The loss turns "how wrong is the network" into one number that can be differentiated. **The choice of loss is the choice of statistical model**: MSE = Gaussian noise, BCE = Bernoulli, categorical CE = multinoulli.

### 4.1 Regression losses

$$
\begin{aligned}
\text{MSE} &= \frac{1}{N}\sum (y_i - \hat{y}_i)^2 \\
\text{MAE} &= \frac{1}{N}\sum \lvert y_i - \hat{y}_i \rvert \\
\text{RMSE} &= \sqrt{\text{MSE}}
\end{aligned}
$$

| Loss | Pros | Cons |
|---|---|---|
| MSE | differentiable everywhere; punishes big errors | squared units; very outlier-sensitive |
| MAE | same units; robust to outliers | not differentiable at 0; constant gradient |
| RMSE | same units as output | still outlier-sensitive |

**Numerical** $y = (3,5,7)$, $\hat{y} = (2,5,9)$: errors 1, 0, −2 → $\text{MAE} = 3/3 = 1$; $\text{MSE} = (1+0+4)/3 = 1.667$; $\text{RMSE} = 1.291$.

### 4.2 Classification losses

$$
\begin{aligned}
&\text{Binary cross-entropy (sigmoid output, } y \in \{0,1\})\\
&L = -\big[ y \log \hat{y} + (1-y)\log(1-\hat{y}) \big]\\[1em]
&\text{Categorical cross-entropy (softmax output, one-hot } y)\\
&L = -\sum_k y_k \log \hat{y}_k
\quad\to\quad
\text{for the correct class } c:\ L = -\log \hat{y}_c\\[1em]
&\text{Hinge (SVM style, } y \in \{-1,+1\})\\
&L = \max(0,\ 1-y\cdot f(x))
\end{aligned}
$$

**Why cross-entropy and not MSE for classification?** Two reasons:
1. CE is the negative log-likelihood of the correct label, so minimising it *is* maximum likelihood.
2. With MSE + sigmoid the gradient contains $\sigma'(z)$, which vanishes when the model is confidently wrong; with CE the σ′ cancels and the gradient is $\hat{y} - y$ — large error, large gradient.

**Numerical** (y = 1): ŷ = 0.9 → $-\ln 0.9 = 0.105$; ŷ = 0.1 → $-\ln 0.1 = 2.303$. Confident mistakes are punished ~22× harder.

### 4.3 Softmax and its pairing with CE ★★★

$$
\hat{y}_k = \frac{e^{z_k}}{\sum_j e^{z_j}} \qquad (k = 1..K)
$$

Properties: $0 < \hat{y}_k < 1$; $\sum \hat{y}_k = 1$; **shift-invariant** ($e^{z+c}$ cancels); temperature can be added as $e^{z/\tau}$.

**Numerical** $z = (2, 1, 0.1)$: $e^2 = 7.389$, $e^1 = 2.718$, $e^{0.1} = 1.105$, sum $= 11.212$ → $(0.659, 0.242, 0.099)$.

**Derivation of the softmax-CE gradient** (a common 3-mark question):

$$
\begin{aligned}
L &= -\log \hat{y}_c = -z_c + \log \sum_j e^{z_j} \\
\frac{\partial L}{\partial z_j} &= -\mathbf{1}\{j = c\} + \frac{e^{z_j}}{\sum_j e^{z_j}} = \hat{y}_j - \mathbf{1}\{j = c\}
\end{aligned}
$$

i.e. $\partial L/\partial z = \hat{y} - y$. **This is the single most useful fact in the course**: softmax + CE and sigmoid + BCE both give the output-layer error $\delta = \hat{y} - y$. Remember it once and you never derive it again.

**Numerical stability:** subtract $\max(z)$ before exponentiating. $\exp(1000)$ overflows; $\exp(1000 - 1000)$ does not, and softmax is shift-invariant so the answer is unchanged.

`Remember:` **MSE for numbers, CE for categories.** And "cross-entropy = −log(probability of the truth)", so a perfect prediction gives 0 loss.

---

## 5. Gradient descent ★★★

### 5.1 Definition, use, and the learning rate

**Gradient descent** is an iterative optimisation algorithm that moves the parameters in the direction of the **negative gradient** of the loss until it reaches a minimum.

Why the negative gradient: the gradient $\nabla J(\theta)$ points in the direction of **steepest increase**. To decrease the loss, step the other way. Formally, from the first-order Taylor expansion

$$
J(\theta + \Delta\theta) \approx J(\theta) + \nabla J(\theta)^\top \Delta\theta
$$

we need $\nabla J(\theta)^\top \Delta\theta < 0$. Choosing $\Delta\theta = -\eta \nabla J(\theta)$ gives a change of $-\eta \lVert \nabla J \rVert^2 < 0$, so the loss always decreases for small enough $\eta$. **That is the proof that the update rule is correct** — write it if asked to justify GD.

**Update rule:**

$$
\theta_{\text{new}} = \theta_{\text{old}} - \eta \cdot \frac{\partial L}{\partial \theta}
$$

$\eta$ (or $\alpha$) = **learning rate / step size**.

**Role of α ★ (a "very important" question in your notes).**
- Too small → tiny steps, very slow convergence, many epochs, risk of getting stuck.
- Too large → overshoots the minimum, loss oscillates or diverges ($f(w) = w^2$, α = 1.1 → each step multiplies $w$ by −1.2).
- Just right → fast and stable. Can be fixed, decayed over time, or per-parameter (Adam).
- With a convex quadratic, stability requires $\eta < 2/L$ where $L$ is the largest curvature (largest eigenvalue of the Hessian).

### 5.2 Algorithm (write this)

1. Initialise $\theta$ randomly (small values; not all zero)
2. repeat until convergence (gradient $\approx 0$, loss plateaus, or max epochs):
3.   forward pass $\to$ predictions $\hat{y}$ and loss $L$
4.   backward pass $\to$ $g = \partial L/\partial \theta$ for every parameter
5.   update all parameters simultaneously: $\theta \leftarrow \theta - \eta g$
6. return $\theta$

### 5.3 The three variants ★★

| | Batch GD | Stochastic GD | Mini-batch GD |
|---|---|---|---|
| Gradient uses | all $m$ examples | 1 random example | $B$ examples (32–256) |
| Updates per epoch | 1 | $m$ | $m/B$ |
| Path | smooth, deterministic | very noisy | mildly noisy |
| Cost per step | high | low | low |
| Memory | whole dataset | one example | one batch |
| Converges to | exact minimum (convex) | neighbourhood of minimum | neighbourhood |
| Used in practice | small data | rarely | **almost always** |

**Definitions.** An **epoch** = one full pass over the training set. **Iteration** = one parameter update. **Shuffling** each epoch is what gives SGD its randomness.

**Why mini-batch wins:** the mini-batch gradient is an **unbiased estimate** of the true gradient (samples are i.i.d.), it averages away some noise (lower variance than single-sample SGD), and the matrix operations are large enough to use the GPU efficiently.

**SGD is an unbiased estimator** (slide-level fact):

$$
\mathbb{E}[g_B] = \frac{1}{\lvert B \rvert} \sum \mathbb{E}[\nabla L_i] = \nabla J(\theta)
\qquad \text{when } B \text{ is sampled uniformly}
$$

**Tiny numerical.** Minimise $f(w) = w^2$ from $w = 4$ with $\eta = 0.1$: $w \leftarrow w - 0.1\cdot 2w = 0.8w$ → 3.2, 2.56, 2.048, … → 0. With $\eta = 1.1$ the factor is $1 - 2.2 = -1.2$ → magnitude grows → divergence.

### 5.4 Convexity and what makes DL hard ★

- **Linear regression + squared loss** → convex bowl → one global minimum → GD finds it.
- **Neural networks** → non-convex: local minima, plateaus, saddle points. In high dimensions, saddle points (not local minima) are the real obstacle, because a random critical point is far more likely to have mixed curvature.

**Practical goal of DL optimisation (important nuance):** we do not need the global minimum; we need a point with low **training and test** loss. Most local minima in large networks are nearly as good as the global one. So the goal is "**reach a low-loss region quickly, without overfitting**", not "find the optimum".

### 5.5 GD for univariate linear regression (Chapter 2 link)

Model $h(x) = w_1 x + w_0$; loss $\sum (y_j - h(x_j))^2$.

$$
\begin{aligned}
\frac{\partial L}{\partial w_0} &= -2 \sum \big(y_j - h(x_j)\big) \\
\frac{\partial L}{\partial w_1} &= -2 \sum x_j \big(y_j - h(x_j)\big) \\[4pt]
w_0 &\leftarrow w_0 + \alpha \sum \big(y_j - h(x_j)\big) \qquad (\text{2 absorbed into } \alpha) \\
w_1 &\leftarrow w_1 + \alpha \sum x_j \big(y_j - h(x_j)\big)
\end{aligned}
$$

**Numerical** (1,2),(2,3),(3,5), start w0 = w1 = 0, α = 0.01: $\sum (y-h) = 10$, $\sum x(y-h) = 1\cdot 2+2\cdot 3+3\cdot 5 = 23$ → $w_0 = 0.10$, $w_1 = 0.23$. Closed form for comparison: $w_1 = 1.5$, $w_0 = 0.333$.

`Remember:` **GD = "roll downhill in the negative gradient direction".** α too big = you jump over the valley; α too small = you crawl.

`Exam:` "Define gradient descent, write its algorithm, explain α, compare batch/SGD/mini-batch." Four sub-answers, one table, done. Add the Taylor-expansion justification as the "significance" line.

---

## 6. Backpropagation ★★★ (derivation + algorithm + numerical)

### 6.1 The idea in one paragraph

Training is a loop: **(1) forward pass** computes $\hat{y}$ and the loss, **(2) backward pass** computes $\partial L/\partial\theta$ for every parameter by applying the chain rule from the output back to the input, reusing intermediate results, **(3) gradient descent** updates. Backpropagation is not a learning rule by itself — it is an *efficient algorithm for computing gradients*. Gradient descent is what actually learns.

**Why it is efficient.** Naively, computing $\partial L/\partial w$ for each of $P$ parameters separately would cost $O(P)$ forward passes. Backprop computes all $P$ gradients in roughly **one forward + one backward pass**, $O(P)$ total arithmetic — same order as the forward pass alone.

### 6.2 Chain rule (scalar and vector)

$$
\begin{aligned}
\text{Scalar:}\quad & y = f(u),\ u = g(x) \implies \frac{dy}{dx} = \frac{dy}{du}\frac{du}{dx} \\[6pt]
\text{Vector:}\quad & z = f(y),\ y = g(x),\ x \in \mathbb{R}^m,\ y \in \mathbb{R}^n \\
& \frac{\partial z}{\partial x_i} = \sum_j \frac{\partial z}{\partial y_j}\frac{\partial y_j}{\partial x_i} \\
& \nabla_x z = \left(\frac{\partial y}{\partial x}\right)^\top \nabla_y z \qquad \text{(Jacobian transpose} \times \text{upstream gradient)}
\end{aligned}
$$

**Computational graph:** nodes = variables/operations, edges = dependencies. $\hat{y} = \sigma(x^\top w + b)$ is the graph $w,x \to \text{dot} \to {+}b \to \sigma \to \hat{y}$. Each node only needs its **local derivative**; the chain rule stitches them together. That locality is why backprop works for any architecture, including CNNs (convolution is a node) and RNNs (unrolled over time).

### 6.3 Setup for the full derivation (2-2-1 network)

$$
\begin{aligned}
z_{11} &= w_{11}x_1 + w_{21}x_2 + b_{11}, & O_{11} &= \sigma(z_{11}) \\
z_{12} &= w_{12}x_1 + w_{22}x_2 + b_{12}, & O_{12} &= \sigma(z_{12}) \\
z_{21} &= W_{11}O_{11} + W_{21}O_{12} + b_{21}, & \hat{y} = O_{21} &= \sigma(z_{21}) \\
L &= (y-\hat{y})^2 & & \text{(MSE, regression)}
\end{aligned}
$$

Parameters: 6 weights (w11,w21,w12,w22,W11,W21) + 3 biases.

**Fact to quote:** $\sigma'(z) = \sigma(z)(1-\sigma(z))$.
Proof: $\sigma = (1+e^{-z})^{-1} \implies \sigma' = \frac{e^{-z}}{(1+e^{-z})^2} = \sigma(1-\sigma)$. Maximum 0.25 at z = 0.

### 6.4 The derivation

**Output layer.**

$$
\begin{aligned}
\frac{\partial L}{\partial \hat{y}} &= -2(y-\hat{y}) \\
\frac{\partial \hat{y}}{\partial z_{21}} &= \hat{y}(1-\hat{y}) \\
\delta_{\text{out}} = \frac{\partial L}{\partial z_{21}}
&= \frac{\partial L}{\partial \hat{y}}\cdot\frac{\partial \hat{y}}{\partial z_{21}}
 = -2(y-\hat{y})\,\hat{y}(1-\hat{y}) \\[6pt]
\frac{\partial L}{\partial W_{11}} &= \delta_{\text{out}}\cdot O_{11} \\
\frac{\partial L}{\partial W_{21}} &= \delta_{\text{out}}\cdot O_{12} \\
\frac{\partial L}{\partial b_{21}} &= \delta_{\text{out}}
\end{aligned}
$$

Rule: **gradient w.r.t. a weight = (error at the receiving node) × (activation of the sending node)**.

**Hidden layer.** The error signal flows back through the output weights:

$$
\begin{aligned}
\delta_1 = \frac{\partial L}{\partial z_{11}} &= \delta_{\text{out}}\cdot W_{11}\cdot O_{11}(1-O_{11}) \\
\delta_2 = \frac{\partial L}{\partial z_{12}} &= \delta_{\text{out}}\cdot W_{21}\cdot O_{12}(1-O_{12}) \\[6pt]
\frac{\partial L}{\partial w_{11}} &= \delta_1 x_1, \quad \frac{\partial L}{\partial w_{21}} = \delta_1 x_2, \quad \frac{\partial L}{\partial b_{11}} = \delta_1 \\
\frac{\partial L}{\partial w_{12}} &= \delta_2 x_1, \quad \frac{\partial L}{\partial w_{22}} = \delta_2 x_2, \quad \frac{\partial L}{\partial b_{12}} = \delta_2
\end{aligned}
$$

The three factors in $\delta_1$ are exactly the three chain-rule links: **upstream error × weight × local derivative**. Write them in that order and the derivation cannot go wrong.

**Update (learning rate η):**

$$
W_{11} \leftarrow W_{11} - \eta\frac{\partial L}{\partial W_{11}}, \quad w_{11} \leftarrow w_{11} - \eta\frac{\partial L}{\partial w_{11}}, \quad b_{21} \leftarrow b_{21} - \eta\frac{\partial L}{\partial b_{21}}, \ \dots \text{ all 9 parameters}
$$

> Your notes' short form $\partial L/\partial w_{11} = -2(y-\hat{y})\cdot W_{11}\cdot x_1$ omits the sigmoid derivatives. It is only valid if the activation derivative is treated as 1. The version above is the correct one.

### 6.5 General algorithm ★★★

1. Initialise weights and biases (Xavier/He, see §8.6)
2. Repeat for each mini-batch:
   a. **FORWARD:** for $l = 1 \dots L$: $z^{(l)} = W^{(l)}a^{(l-1)} + b^{(l)}$; $a^{(l)} = g(z^{(l)})$  
   b. **Loss:** $L(a^{(L)}, y)$  
   c. **Output delta:** $\delta^{(L)} = \partial L/\partial a^{(L)} \odot g'(z^{(L)})$  
      (for softmax-CE or sigmoid-BCE this collapses to $\delta^{(L)} = \hat{y} - y$)  
   d. **BACKWARD:** for $l = L-1$ down to 1:  
      $\delta^{(l)} = \left(W^{(l+1)\top}\delta^{(l+1)}\right) \odot g'(z^{(l)})$  
   e. **Gradients:** $\partial L/\partial W^{(l)} = \delta^{(l)}a^{(l-1)\top}$; $\partial L/\partial b^{(l)} = \delta^{(l)}$  
   f. **UPDATE:** $W^{(l)} \leftarrow W^{(l)} - \eta\,\partial L/\partial W^{(l)}$; $b^{(l)} \leftarrow b^{(l)} - \eta\,\partial L/\partial b^{(l)}$  

Batch version: $\partial L/\partial W^{(l)} = (1/B)\sum_b \delta_b^{(l)}a_b^{(l-1)\top}$, i.e. $\Delta^{(l)}A^{(l-1)\top}$ with $\Delta$ a matrix of per-example deltas.

**Derivation of the backward recursion** (worth memorising, it is 2 lines):

$$
\begin{aligned}
\delta^{(l)} &= \frac{\partial L}{\partial z^{(l)}} \\
&= \left(\frac{\partial z^{(l+1)}}{\partial z^{(l)}}\right)^\top \delta^{(l+1)} \odot g'(z^{(l)}) \qquad \text{(chain rule, } z^{(l)} \text{ affects } L \text{ only through } z^{(l+1)}\text{)} \\
&= \left(W^{(l+1)}\operatorname{diag}(g'(z^{(l)}))\right)^\top \delta^{(l+1)} \ \dots \\
&= \left(W^{(l+1)\top}\delta^{(l+1)}\right) \odot g'(z^{(l)})
\end{aligned}
$$

### 6.6 Worked numerical (one full iteration) ★★★

All weights = 1, all biases = 1, $x = (1,2)$, $y = 1$, $\eta = 1$, sigmoid, MSE.

**Forward.**

$$
\begin{aligned}
z_{11} &= 1\cdot 1 + 1\cdot 2 + 1 = 4, & O_{11} &= \sigma(4) = 0.98201 \\
z_{12} &= 4, & O_{12} &= 0.98201 \\
z_{21} &= 0.98201 + 0.98201 + 1 = 2.96402 \\
\hat{y} &= \sigma(2.96402) = 0.95092 \\
L &= (1-0.95092)^2 = 0.002409
\end{aligned}
$$

**Backward.**

$$
\begin{aligned}
\hat{y}(1-\hat{y}) &= 0.95092 \times 0.04908 = 0.046671 \\
\delta_{\text{out}} &= -2(1-0.95092)(0.046671) = -0.004581 \\[6pt]
\frac{\partial L}{\partial W_{11}} = \frac{\partial L}{\partial W_{21}} &= -0.004581 \times 0.98201 = -0.004499 \\
\frac{\partial L}{\partial b_{21}} &= -0.004581 \\[6pt]
O(1-O) &= 0.98201 \times 0.017986 = 0.017663 \\
\delta_1 = \delta_2 &= (-0.004581)(1)(0.017663) = -8.09 \times 10^{-5} \\[6pt]
\frac{\partial L}{\partial w_{11}} &= \delta_1 x_1 = -8.09 \times 10^{-5} \\
\frac{\partial L}{\partial w_{21}} &= \delta_1 x_2 = -1.62 \times 10^{-4} \\
\frac{\partial L}{\partial b_{11}} &= -8.09 \times 10^{-5} \qquad \text{(same for the second hidden node)}
\end{aligned}
$$

**Update (η = 1).**

$$
\begin{aligned}
W_{11} = W_{21} &= 1.004499, & b_{21} &= 1.004581 \\
w_{11} = w_{12} &= 1.0000809, & w_{21} = w_{22} &= 1.0001618, & b_{11} = b_{12} &= 1.0000809
\end{aligned}
$$

**Iteration 2 check.**

$$
\begin{aligned}
z_{11} &= 1.0000809 + 2(1.0001618) + 1.0000809 = 4.000485 \to O_{11} = 0.982023 \\
z_{21} &= 2(1.004499)(0.982023) + 1.004581 = 2.97746 \to \hat{y} = 0.95155 \\
L &= 0.002347 < 0.002409 \quad \checkmark\ \text{loss decreased}
\end{aligned}
$$

**The observation that earns the last mark:** the hidden gradients are ~50× smaller than the output gradients because $\sigma'(4) = 0.018$ is nearly saturated. That is the vanishing-gradient problem in one line.

### 6.7 Training, validation, testing

$$
\begin{aligned}
\text{Training set} &\to \text{adjust } \theta &&\text{(what backprop touches)} \\
\text{Validation set} &\to \text{choose hyperparameters, early stopping} \\
\text{Test set} &\to \text{final unbiased estimate} &&\text{(touched once)}
\end{aligned}
$$

**Underfitting:** training loss high. **Overfitting:** training loss low, validation loss rising. **Good fit:** both low and close.

`Remember:` **"Forward to compute, backward to blame, step to improve."** Backprop = chain rule + dynamic programming (cache the deltas).

`Exam:` "Derive the backpropagation algorithm for a 2-layer network and perform one iteration." Structure: diagram → forward numbers → δ_out → δ_hidden → gradients → update → iteration 2 → one-line observation. That is a 10/10 answer.

---

## 7. Activation functions ★★★

### 7.1 Why they exist, and what "good" looks like

An activation function is applied to the pre-activation $z$ to produce the neuron output $a = g(z)$. Its job is to add **non-linearity** (see §3.1) and to shape the gradient.

**Ideal properties:** non-linear; differentiable (backprop needs $g'$); cheap to compute; **zero-centred** (faster convergence); **non-saturating** (gradient does not vanish for large |z|); monotonic helps optimisation.

### 7.2 The table you must reproduce ★★★

| Function | Formula | Range | Derivative | Use / notes |
|---|---|---|---|---|
| Step | $1$ if $z \ge 0$ else $0$ | $\{0,1\}$ | $0$ (undefined at 0) | classic perceptron only; not trainable by GD |
| Sigmoid | $1/(1+e^{-z})$ | $(0,1)$ | $\sigma(1-\sigma) \le 0.25$ | binary output; probability; **not zero-centred**; saturates |
| Tanh | $(e^z-e^{-z})/(e^z+e^{-z})$ | $(-1,1)$ | $1 - \tanh^2 \le 1$ | hidden layers pre-ReLU; **zero-centred**; still saturates |
| ReLU | $\max(0,z)$ | $[0,\infty)$ | $1$ if $z>0$, $0$ if $z<0$ | default hidden activation; cheap; sparse; **dying ReLU** |
| Leaky ReLU | $z$ if $z>0$ else $0.01z$ | $(-\infty,\infty)$ | $1 / 0.01$ | fixes dying ReLU |
| PReLU | $z$ if $z>0$ else $\alpha z$, $\alpha$ learned | $(-\infty,\infty)$ | $1 / \alpha$ | data-driven slope |
| ELU | $z$ if $z>0$ else $\alpha(e^z-1)$ | $(-\alpha,\infty)$ | $1 / \alpha e^z$ | smooth, closer to zero mean; exp cost |
| SELU | $\lambda z$ / $\lambda\alpha(e^z-1)$, $\lambda\approx 1.0507$, $\alpha\approx 1.6733$ | $(-\lambda\alpha,\infty)$ | scaled | self-normalising (mean 0, var 1) |
| Softmax | $e^{z_k}/\sum_j e^{z_j}$ | $(0,1)$, $\sum = 1$ | $\hat{y} - y$ with CE | multiclass **output** only |

**Useful identities.** $\tanh(z) = 2\sigma(2z) - 1$. $\sigma'(z) = \sigma(z)(1-\sigma(z))$. $\tanh'(z) = 1 - \tanh^2(z)$.

**Numerical.** $\sigma(0) = 0.5$, $\sigma(1) = 0.731$, $\sigma(-1) = 0.269$, $\sigma(4) = 0.982$, $\sigma(10) \approx 0.99995$. $\tanh(1) = 0.762$. Softmax of $(2,1,0.1)$ = $(0.659, 0.242, 0.099)$.

### 7.3 Vanishing and exploding gradients ★★★

**Sigmoid case.** The derivative maxes at 0.25. Backprop through $L$ layers multiplies one factor per layer:

$$
\frac{\partial L}{\partial w^{(\text{first layer})}} \propto \prod_{l=1}^{L} \sigma'(z_l) \le 0.25^L
$$

$0.25^{10} \approx 9.5 \times 10^{-7}$, $0.25^{20} \approx 10^{-12}$ — early layers effectively stop learning. That is why pre-2010 deep networks could not be trained.

**General case.** With weight matrices, the gradient contains $\prod W^{(l)}\operatorname{diag}(g')$; its norm behaves like $(\rho\cdot\gamma)^L$ where $\rho$ is the largest singular value of $W$ and $\gamma$ the max derivative:
- product < 1 → **vanishes**;
- product > 1 → **explodes** (numerical overflow, NaN, wildly oscillating loss).

**Remedies.**

| Problem | Fix |
|---|---|
| Vanishing | ReLU family, Xavier/He init, batch norm, residual connections, LSTM/GRU for sequences |
| Exploding | gradient clipping (by norm or value), smaller LR, careful init, batch norm |
| Both | shorter unrolled sequences (truncated BPTT), better architecture |

**Gradient clipping** (formula):

$$
\text{if } \Vert g \Vert_2 > \tau: \quad g \leftarrow \tau \cdot g / \Vert g \Vert_2 \qquad \text{(clip by global norm)}
$$

Guarantees $\Vert g \Vert_2 \le \tau$.

### 7.4 Dying ReLU ★

If a neuron's pre-activation is negative for **every** sample, its output and gradient are 0, so its weights never update again — it is "dead". Causes: large negative bias, too-large learning rate, bad init. Fixes: Leaky ReLU/PReLU/ELU, lower LR, better init, batch norm.

> Your notes say dying ReLU is caused by non-differentiability at 0 — the real cause is the zero gradient for all-negative inputs.

### 7.5 Output-unit choice ★★

| Task | Output activation | Loss |
|---|---|---|
| Regression (real value) | linear (none) | MSE |
| Binary classification | sigmoid (1 unit) | binary cross-entropy |
| Multiclass, one label | softmax (K units) | categorical cross-entropy |
| Multilabel (several labels) | sigmoid on each unit | binary CE per label |

**Statistical justification:** linear output ↔ Gaussian $p(y \mid x) = N(\mu(x), \sigma^2)$; sigmoid ↔ Bernoulli; softmax ↔ multinoulli. Choosing the output unit is choosing the likelihood.

`Remember:` **"Sigmoid for yes/no, softmax for which-one, linear for how-much."** And sigmoid's derivative is $a(1-a)$ — output times (1 − output), no exponentials needed at backprop time.

`Exam:` "Compare sigmoid, tanh, ReLU, Leaky ReLU — formula, range, graph, advantages, disadvantages." Draw all four on one axis in 30 seconds; that single sketch answers the "graph" part of every activation question.

---

## 8. Generalisation, overfitting, and regularisation ★★

### 8.1 Bias–variance ★★

$$
\mathbb{E}[(y-h(x))^2] = \underbrace{(\mathbb{E}[h(x)]-f(x))^2}_{\text{bias}^2} + \underbrace{\mathbb{E}[(h(x)-\mathbb{E}[h(x)])^2]}_{\text{variance}} + \underbrace{\sigma^2}_{\text{irreducible}}
$$

- **Bias** — error from wrong assumptions (too simple a model). High bias → underfitting.
- **Variance** — error from sensitivity to the particular training set. High variance → overfitting.
- **Graph to draw:** x = model complexity, y = error. Bias falls, variance rises, total is U-shaped. Minimum = optimal complexity. Left of it = underfitting, right = overfitting.

| | Underfitting | Good fit | Overfitting |
|---|---|---|---|
| Bias | high | low | low |
| Variance | low | low | high |
| Train error | high | low | very low |
| Test error | high | low | high |

**Causes of overfitting:** too many parameters, too little data, noisy labels, training too long, no regularisation, too many irrelevant features.
**Causes of underfitting:** model too small, features too weak, over-regularisation, too few epochs, LR too high to converge.

**Fixes:** more data, augmentation, simpler model, weight decay, dropout, early stopping, batch norm.

### 8.2 Regularisation: the general form ★★

$$
\begin{aligned}
\text{Cost}(h) &= \text{EmpiricalLoss}(h) + \lambda\cdot\text{Complexity}(h) \\
\text{Complexity} = L_q(w) &= \sum_i |w_i|^q \qquad (q = 1 \to L1,\ q = 2 \to L2)
\end{aligned}
$$

λ controls the trade-off: $\lambda = 0$ → no penalty (overfit risk); $\lambda \to \infty$ → all weights → 0 (underfit).

### 8.3 L1 vs L2 ★★★

| | L1 (Lasso, q=1) | L2 (Ridge, q=2) |
|---|---|---|
| Penalty | $\lambda\sum_i \lvert w_i \rvert$ | $\lambda\sum_i w_i^2$ |
| Gradient added | $\lambda\cdot\operatorname{sign}(w)$ | $2\lambda w$ |
| Effect | weights driven **exactly to 0** | weights shrink smoothly |
| Result | **sparse**, feature selection | small weights, no zeros |
| Differentiable | no (at 0) | yes |
| Good when | few features matter | all features matter, multicollinearity |

**Why L1 is sparse (the geometry answer).** Minimise loss subject to $\sum_i \lvert w_i \rvert \le s$ (a diamond) vs $\sum_i w_i^2 \le s$ (a circle). The loss contours are ellipses around the unconstrained optimum; they first touch the **diamond at a corner**, which lies on an axis where one weight is 0. A circle is touched at a generic point with both weights non-zero.

**Why L2 is "weight decay".** The update becomes

$$
w \leftarrow w - \eta\left(\frac{\partial L_{\text{emp}}}{\partial w} + 2\lambda w\right) = (1-2\eta\lambda)w - \eta\frac{\partial L_{\text{emp}}}{\partial w}
$$

so every step multiplies the weight by slightly less than 1 — it decays unless the data gradient pushes back.

**Ridge closed form:** $w^* = (X^\top X + \lambda I)^{-1}X^\top y$. The $\lambda I$ also fixes singularity of $X^\top X$.

### 8.4 Dropout ★★

During **training**, randomly set a fraction $p$ of activations to zero, independently each iteration. At **test** time, use all units.

$$
\begin{aligned}
\text{train:}\quad & m \sim \text{Bernoulli}(1-p) &&\text{(mask)} \\
& h' = (m \odot h)/(1-p) &&\text{(inverted dropout: rescale at train time)} \\
\text{test:}\quad & h' = h
\end{aligned}
$$

**Why rescale?** $\mathbb{E}[m] = 1-p$, so $\mathbb{E}[(m\odot h)/(1-p)] = h$ — the expected activation is unchanged, so the next layer sees the same scale at train and test time. (The older alternative is to multiply by $1-p$ at test time; never both.)

**Why it works:** it prevents co-adaptation (a neuron cannot rely on a specific partner being present), it is equivalent to training an ensemble of $2^n$ thinned subnetworks and averaging them at test time, and it adds noise that acts like a regulariser.

**Practical notes:** not applied to the output layer; typically $p = 0.2$–$0.5$; disabled at inference (`model.eval()` in PyTorch, automatic in Keras).

### 8.5 Other regularisers ★

- **Early stopping:** monitor validation loss; stop when it has not improved for `patience` epochs (5–10) and **restore the best weights**. Cheap, effective, no architecture change.
- **Data augmentation:** generate plausible new samples (flip, rotate, crop, jitter colour, mixup; synonym replacement and back-translation for text). Reduces variance without new labels.
- **Batch normalisation:** normalise each mini-batch per feature to zero mean/unit variance, then apply learned scale $\gamma$ and shift $\beta$:

$$
\hat{x} = \frac{x-\mu_B}{\sqrt{\sigma_B^2+\epsilon}}, \qquad y = \gamma\hat{x} + \beta
$$

  Benefits: allows higher LR, less sensitive to init, slight regularisation, faster convergence. It also decouples the gradient from the scale of the weights, which helps the vanishing-gradient problem.
- **Weight tying / parameter sharing** (CNNs, RNNs) — fewer parameters = less overfitting.
- **Noise injection** (Gaussian noise on inputs) — equivalent to Tikhonov regularisation for small noise.

### 8.6 Weight initialisation ★★

Bad init → vanishing/exploding activations and gradients. **Never initialise all weights to zero:** every neuron computes the same output and receives the same gradient (symmetry), so the network never breaks symmetry. Biases may be zero.

Goal: keep the variance of activations constant across layers. For a layer with $n_{\text{in}}$ inputs and $n_{\text{out}}$ outputs:

$$
\begin{aligned}
\text{Xavier / Glorot (sigmoid, tanh):}\quad & \operatorname{Var}(w) = \frac{2}{n_{\text{in}}+n_{\text{out}}} &&\sim U\left(-\sqrt{\tfrac{6}{n_{\text{in}}+n_{\text{out}}}},\ +\dots\right) \\
\text{He / Kaiming (ReLU):}\quad & \operatorname{Var}(w) = \frac{2}{n_{\text{in}}} &&\sim N\left(0,\ \frac{2}{n_{\text{in}}}\right)
\end{aligned}
$$

**Memory hook:** *Xavier averages the two fan sizes; He doubles for ReLU because ReLU kills half the activations.*

`Remember:` **"L1 kills, L2 shrinks, dropout ensembles, early stopping saves."**

`Exam:` "What is overfitting? How can it be prevented?" Give 6 methods in one line each: more data, augmentation, L1/L2, dropout, early stopping, simpler model/batch norm.

---

## 9. Optimizers ★★★

### 9.1 Why plain SGD is not enough

Problems: (i) ill-conditioning — the loss is a narrow valley, so SGD zig-zags; (ii) saddle points and plateaus; (iii) one global learning rate is wrong for all parameters (rare features need bigger steps); (iv) noisy gradients.

### 9.2 SGD and momentum

$$
\begin{aligned}
\text{SGD:}\quad & \theta \leftarrow \theta - \eta g \\
\text{Momentum:}\quad & v \leftarrow \beta v + g &&(\beta \approx 0.9) \\
& \theta \leftarrow \theta - \eta v &&\text{(some texts: } v \leftarrow \beta v + (1-\beta)g\text{)}
\end{aligned}
$$

Expanding the recursion:

$$
v_t = \sum_{k=0}^{t} \beta^{t-k}g_k
$$

so $v$ is an exponentially weighted average of past gradients: consistent directions accumulate (fast), oscillating directions cancel (damped). Effective step size along a consistent direction is $\eta/(1-\beta) \approx 10\eta$ for β = 0.9.

**Nesterov momentum** (look-ahead variant): compute the gradient at $\theta - \eta\beta v$ instead of at $\theta$; it "looks ahead" and corrects overshoot. Usually slightly better than plain momentum.

### 9.3 AdaGrad, RMSProp, Adam ★★★

**AdaGrad** — per-parameter learning rate, divided by the accumulated gradient magnitude:

$$
\begin{aligned}
G &\leftarrow G + g^2 \\
\theta &\leftarrow \theta - \frac{\eta g}{\sqrt{G}+\epsilon}
\end{aligned}
$$

Great for sparse features (NLP), but $G$ only grows, so the effective LR decays to zero and learning stops.

**RMSProp** — fixes AdaGrad with an exponential average:

$$
\begin{aligned}
s &\leftarrow \beta s + (1-\beta)g^2 &&(\beta \approx 0.9) \\
\theta &\leftarrow \theta - \frac{\eta g}{\sqrt{s}+\epsilon}
\end{aligned}
$$

**Adam** = momentum + RMSProp + bias correction:

$$
\begin{aligned}
m &\leftarrow \beta_1 m + (1-\beta_1)g &&\text{first moment (mean)} \\
v &\leftarrow \beta_2 v + (1-\beta_2)g^2 &&\text{second moment (uncentred variance)} \\
\hat{m} &= m/(1-\beta_1^t) &&\text{bias correction} \\
\hat{v} &= v/(1-\beta_2^t) \\
\theta &\leftarrow \theta - \eta\cdot\frac{\hat{m}}{\sqrt{\hat{v}}+\epsilon}
\end{aligned}
$$

Defaults: $\eta = 0.001$, $\beta_1 = 0.9$, $\beta_2 = 0.999$, $\epsilon = 10^{-8}$.

**Why bias correction:** $m$ and $v$ start at 0, so early estimates are biased toward 0. Since $\mathbb{E}[m_t] = (1-\beta_1^t)\cdot\mathbb{E}[g]$, dividing by $(1-\beta_1^t)$ removes the bias.

**Numerical first step** ($g_1 = 0.5$): $m_1 = 0.05$, $v_1 = 0.00025$, $\hat{m}_1 = 0.5$, $\hat{v}_1 = 0.25$, update $= \eta\cdot 0.5/0.5 = \eta$. **The first Adam step is $\approx \eta$ regardless of gradient size** — a great one-line fact for the exam.

### 9.4 Comparison table ★★

| Optimizer | Uses | Learning rate | Best for | Weakness |
|---|---|---|---|---|
| Batch GD | all data | fixed | small convex problems | slow, memory |
| SGD | 1 example | fixed/decayed | huge data, good generalisation | noisy, slow |
| Momentum | 1st moment | fixed | ravines, faster convergence | one extra hyperparameter |
| AdaGrad | 2nd moment (sum) | per-param, decaying | sparse features | LR → 0 |
| RMSProp | 2nd moment (EMA) | per-param | non-stationary, RNNs | needs β tuning |
| Adam | 1st + 2nd moment | per-param, adaptive | **default choice** | can generalise slightly worse than SGD+momentum on vision |

**Learning-rate schedules:** step decay, exponential decay, cosine annealing, warm-up, one-cycle. `ReduceLROnPlateau` in Keras is the lazy default.

`Remember:` **"Momentum remembers direction, RMSProp remembers magnitude, Adam remembers both — and corrects for starting at zero."**

`Exam:` "Explain Adam with its equations." Write the four lines, then the bias-correction sentence and the "first step = η" check. That is a complete answer.

---

## 10. Chapter-2 toolkit (the ML background the paper still asks) ★

### 10.1 Learning paradigms

| Paradigm | Data | Goal | Example |
|---|---|---|---|
| Supervised | labelled $(x,y)$ | learn $f: x \to y$ | spam classification |
| Unsupervised | unlabelled | find structure | clustering, PCA |
| Semi-supervised | few labels + many unlabelled | use both | pseudo-labelling |
| Reinforcement | reward signal | maximise return | game playing |

**Supervised setup:** hypothesis $h \in H$ (hypothesis space), trained on $N$ pairs, judged on **unseen** data. A hypothesis is *consistent* if it reproduces the training labels — consistency is **not** the goal (a degree-12 polynomial through 13 points is consistent and useless). **Occam's razor:** prefer the simplest hypothesis consistent with the data.

### 10.2 Linear regression ★

$$
\begin{aligned}
\text{Model:}\quad & h(x) = w_1 x + w_0 \\
\text{Loss:}\quad & \sum_j \left(y_j - (w_1 x_j + w_0)\right)^2 \\
\text{Normal equations:}\quad & w_1 = \frac{N\sum xy - \sum x\sum y}{N\sum x^2 - (\sum x)^2}, \quad w_0 = \frac{\sum y - w_1\sum x}{N} \\
\text{Matrix form:}\quad & w^* = (X^\top X)^{-1}X^\top y
\end{aligned}
$$

**Numerical** (1,2),(2,3),(3,5): $\sum x = 6$, $\sum y = 10$, $\sum xy = 23$, $\sum x^2 = 14$, $N = 3$ → $w_1 = (69-60)/(42-36) = 1.5$, $w_0 = (10-9)/3 = 0.333$.

### 10.3 Logistic regression and MLE ★★

$$
\begin{aligned}
\text{logit:}\quad & \log\frac{p}{1-p} = w^\top x \implies p = \sigma(w^\top x) \\
\text{Likelihood:}\quad & L(w) = \prod_j \sigma(w^\top x_j)^{y_j}\left(1-\sigma(w^\top x_j)\right)^{1-y_j} \\
\text{Log-lik:}\quad & \ell(w) = \sum_j \left[y_j\log\sigma_j + (1-y_j)\log(1-\sigma_j)\right] \\
\text{Gradient:}\quad & \nabla\ell = \sum_j (y_j-\sigma_j)x_j \qquad \text{(no closed form} \to \text{gradient ascent)}
\end{aligned}
$$

MLE steps: write likelihood → take log → differentiate → set to 0 → solve (or iterate).

### 10.4 Metrics ★★

$$
\begin{aligned}
\text{Accuracy} &= \frac{TP+TN}{TP+TN+FP+FN} \\
\text{Precision} &= \frac{TP}{TP+FP} &&\text{"of those I flagged, how many were right"} \\
\text{Recall} &= \frac{TP}{TP+FN} &&\text{"of the real positives, how many I caught"} \\
\text{Specificity} &= \frac{TN}{TN+FP} \\
\text{F1} &= \frac{2PR}{P+R} = \frac{2TP}{2TP+FP+FN}
\end{aligned}
$$

**Worked example** (10,000 emails; TP=600, FN=300, FP=100, TN=9000): accuracy $0.96$; precision $600/700 = 0.857$; recall $600/900 = 0.667$; F1 $0.75$; specificity $0.989$.

**When to prefer which:** recall when a miss is dangerous (cancer, fraud detection where fraud is the positive class); precision when a false alarm is expensive (flagging a legitimate transaction, unnecessary biopsy). **Accuracy is misleading under class imbalance** — 99% accuracy is trivial if 99% of samples are negative.

**ROC/AUC:** TPR vs FPR over all thresholds; AUC = probability that a random positive scores above a random negative; 0.5 = random, 1.0 = perfect; threshold-independent.

### 10.5 Normalisation vs standardisation ★★

| | Min–max normalisation | Z-score standardisation |
|---|---|---|
| Formula | $(x-\min)/(\max-\min)$ | $(x-\mu)/\sigma$ |
| Range | $[0,1]$ | unbounded, mean 0, std 1 |
| Use when | unknown distribution, bounded inputs, images | roughly Gaussian features, linear models, PCA |
| Outliers | very sensitive | less sensitive |

**Numerical** on 10,20,30,40,50: min–max → $0, 0.25, 0.5, 0.75, 1$. Standardisation (μ=30, σ=14.14) → $-1.41, -0.71, 0, 0.71, 1.41$.

> Exam habit from your notes: the question gives a dataset and asks for the values — write the formula, then substitute row by row. Do not skip the table.

---

# PART II — Convolutional neural networks (post-mid)

## 11. Why convolution, and what it computes

### 11.1 Why a dense layer fails on images ★★

Take an RGB image of size $224 \times 224 \times 3$. Flattening gives **150,528 inputs**. A single dense layer with 64 neurons then needs

$$
150{,}528 \times 64 + 64 = 9{,}633{,}856 \text{ weights}
$$

for *one* layer, and you lose all spatial structure in the process.

Three separate problems:

1. **Parameters explode** — memory, compute, and massive overfitting on any realistic dataset.
2. **Flattening destroys neighbourhoods** — pixel (5,5) and (5,6) are adjacent in the image but far apart, or merely permuted, in the vector. A dense layer has no idea they are neighbours.
3. **No translation invariance** — a cat in the top-left and the same cat in the bottom-right are completely different input vectors to a dense layer; it must relearn the cat in every position.

**Convolution fixes all three with three ideas** (the slide's "three central CNN ideas"):

| Idea | What it gives |
|---|---|
| **Local connectivity** — each output looks at a small patch | far fewer parameters |
| **Weight sharing** — the same filter slides over the whole image | translation equivariance, parameters independent of image size |
| **Hierarchy / pooling** — stack layers, downsample | larger receptive fields, invariance to small shifts |

### 11.2 The convolution operation ★★★

**1-D discrete convolution:**

$$
(f * g)[n] = \sum_{\tau} f[\tau]\, g[n-\tau]
$$

**2-D convolution** (image $X$, kernel $K$):

$$
S[i,j] = \sum_m \sum_n X[i+m, j+n] \cdot K[m,n]
$$

**Cross-correlation** is the same sum without flipping the kernel:

$$
S[i,j] = \sum_m \sum_n X[i+m, j+n] \cdot K[m,n]
\qquad (\text{with } K \text{ flipped for true convolution})
$$

**The convention that trips people up:** deep-learning libraries (and this course) call it convolution but implement **cross-correlation** — the kernel is *not* flipped. Say this in the exam: "CNN convolution is technically cross-correlation; for learned kernels the flip is irrelevant because the network can learn the flipped kernel instead."

**Kernel intuition:** a filter is a small pattern detector. $\begin{bmatrix} 1 & 0 & -1 \\ 1 & 0 & -1 \\ 1 & 0 & -1 \end{bmatrix}$ responds strongly where there is a vertical edge (bright on the left, dark on the right). Its transpose detects horizontal edges. In a CNN these values are **learned**, not hand-designed.

### 11.3 Worked feature-map numerical ★★★

Input (5×5), kernel 3×3:

$$
X = \begin{bmatrix}
1 & 2 & 3 & 0 & 1 \\
4 & 5 & 6 & 1 & 2 \\
7 & 8 & 9 & 2 & 3 \\
1 & 2 & 3 & 4 & 5 \\
6 & 7 & 8 & 9 & 0
\end{bmatrix}
\qquad
K = \begin{bmatrix}
1 & 0 & -1 \\
1 & 0 & -1 \\
1 & 0 & -1
\end{bmatrix}
$$

Top-left patch $\begin{bmatrix} 1 & 2 & 3 \\ 4 & 5 & 6 \\ 7 & 8 & 9 \end{bmatrix}$:

$$
\begin{aligned}
& (1\cdot 1 + 2\cdot 0 + 3\cdot(-1)) + (4\cdot 1 + 5\cdot 0 + 6\cdot(-1)) + (7\cdot 1 + 8\cdot 0 + 9\cdot(-1)) \\
&= (1 - 3) + (4 - 6) + (7 - 9) \\
&= -2 - 2 - 2 = -6
\end{aligned}
$$

Sliding with stride 1 gives the full 3×3 feature map:

$$
S = \begin{bmatrix}
-6 & 12 & 12 \\
-6 & 8 & 8 \\
-6 & 2 & 12
\end{bmatrix}
$$

> Your notes sketched all nine entries as −6. Only the top-left is −6; the rest differ because each patch differs. Write the arithmetic for at least three positions.

**Straight-through check for position (0,1)** — patch starting at column 1:

$$
\begin{bmatrix} 2 & 3 & 0 \\ 5 & 6 & 1 \\ 8 & 9 & 2 \end{bmatrix} \cdot K
= (2-0) + (5-1) + (8-2) = 2 + 4 + 6 = 12\ \checkmark
$$

### 11.4 Multiple channels and filters ★★★

An RGB image is $H \times W \times 3$. Each filter must span **all input channels**, so it is $K \times K \times C_{\text{in}}$. The three channel-wise sums are added into **one** 2-D feature map:

$$
S[i,j] = \sum_{m,n,c} X[i+m, j+n, c] \cdot K[m,n,c]
$$

With $C_{\text{out}}$ filters you get $C_{\text{out}}$ feature maps — one per filter — stacked into an output volume $H' \times W' \times C_{\text{out}}$.

**Parameter count (very common 2-mark question):**

$$
\text{params} = (K \cdot K \cdot C_{\text{in}} + 1) \cdot C_{\text{out}}
$$

The $+1$ is one bias per filter. Example: 64 filters of 3×3 on RGB → $(3 \cdot 3 \cdot 3 + 1) \cdot 64 = 28 \cdot 64 = 1792$ parameters.

**Multiplications for the forward pass:**

$$
\text{mults} = H' \cdot W' \cdot C_{\text{out}} \cdot K \cdot K \cdot C_{\text{in}}
$$

Example: 12×12×3 input, 256 filters of 3×3, no padding → $10 \times 10$ output; $10 \cdot 10 \cdot 256 \cdot 9 \cdot 3 = 691{,}200$ multiplications; parameters $9 \cdot 3 \cdot 256 = 6{,}912$.

### 11.5 Output size, padding, stride, dilation ★★★

$$
\text{output} = \left\lfloor \frac{n + 2p - k}{s} \right\rfloor + 1
$$

$$
\begin{aligned}
n &= \text{input size} & k &= \text{kernel size} \\
p &= \text{padding} & s &= \text{stride}
\end{aligned}
$$

**"Same" padding** keeps the size: $p = (k - 1)/2$ (valid for odd k, stride 1). For $k = 3$, $p = 1$; $k = 5$, $p = 2$.

**Why padding exists:** (i) without it every convolution shrinks the image and eventually you run out of pixels; (ii) border pixels are used far fewer times than central ones, so information at the edges is under-weighted.

**Stride** is the step size of the slide. $s = 2$ halves the output — a cheap alternative to pooling.

**Dilation** inserts gaps in the kernel, so a $k \times k$ kernel with dilation $d$ behaves like an **effective kernel size**:

$$
k_{\text{eff}} = k + (k - 1)(d - 1) = d(k - 1) + 1
$$

Dilation grows the receptive field **without adding parameters** (used in DeepLab for segmentation, and in WaveNet).

**Practice set:**

| n | k | p | s | output |
|---|---|---|---|---|
| 28 | 3 | 1 | 1 | 28 (same) |
| 32 | 5 | 0 | 1 | 28 |
| 7 | 3 | 0 | 2 | 3 |
| 224 | 7 | 3 | 2 | 112 (the ResNet stem) |
| 13 | 3 | 1 | 2 | 7 |

Worked for the last one: $\lfloor (13 + 2 - 3)/2 \rfloor + 1 = \lfloor 12/2 \rfloor + 1 = 7$.

### 11.6 Receptive field ★★

The **receptive field** is the region of the input that can influence one output unit. It grows with depth:

$$
\begin{aligned}
r_0 &= 1 \\
r_l &= r_{l-1} + (k_l - 1) \cdot \prod_{i<l} s_i
\qquad (\text{jump} = \text{product of previous strides})
\end{aligned}
$$

Simpler recurrence for stride 1: $r_l = r_{l-1} + (k_l - 1)$, so three 3×3 convs have RF $1 + 2 + 2 + 2 = 7$.

**Why this matters:** two stacked 3×3 convs see a 5×5 region but use $2 \cdot 9 = 18$ weights instead of 25, and have two non-linearities instead of one. This is the VGG insight: **stack small kernels instead of using one big kernel**. Three 3×3 ≈ one 7×7, with 27 vs 49 weights.

**Exercise** (3 conv layers, k=3, s=1, starting from 1): $\text{RF} = 1 \to 3 \to 5 \to 7$.

**Pooling also grows the RF** by its stride; that is why classification needs a global pool or many layers.

### 11.7 Pooling ★★★

Pooling summarises a neighbourhood into one number, shrinking the map. It has **no learnable parameters**.

$$
\begin{aligned}
\text{Max pooling:}\quad y &= \max_{i \in R} x_i \\
\text{Average pooling:}\quad y &= \frac{1}{|R|} \sum_{i \in R} x_i \\
\text{Output size:}\quad &\left\lfloor \frac{n - f}{s} \right\rfloor + 1
\end{aligned}
$$

**Worked example** (2×2, stride 2):

$$
\text{Input:}\quad
\begin{bmatrix}
1 & 3 & 2 & 0 \\
4 & 6 & 5 & 1 \\
1 & 4 & 2 & 1 \\
4 & 7 & 2 & 4
\end{bmatrix}
\qquad
\text{Max pool:}\quad
\begin{bmatrix}
6 & 5 \\
7 & 4
\end{bmatrix}
\qquad
\text{Average pool:}\quad
\begin{bmatrix}
3.5 & 2.0 \\
4.0 & 2.25
\end{bmatrix}
$$

(Those average values use windows $[1,3,4,6]$, $[2,0,5,1]$, $[1,4,4,7]$, $[2,1,2,4]$.)

| Max pooling | Average pooling |
|---|---|
| selects the strongest response | preserves average evidence |
| good for feature **presence** (is there an edge?) | smooths activations |
| sharper, loses "where exactly" | softer, keeps more context |
| most common in practice | common in older architectures (LeNet) |

**Global average pooling (GAP):**

$$
y_c = \frac{1}{H \cdot W} \sum_{i,j} X[i,j,c]
$$

One number per channel, so a $H \times W \times C$ tensor becomes a $C$-vector. It replaces the huge flatten+dense head: for $7 \times 7 \times 512 \to 25{,}088$ inputs, a dense layer to 1000 classes needs 25M weights; GAP needs **0** weights, then one $512 \times 1000$ FC. It also makes the model input-size agnostic and reduces overfitting.

**Advantages of pooling:** fewer parameters and computations; translation invariance; nothing to train; reduces overfitting; enlarges the receptive field.
**Disadvantages:** information loss; exact location lost (bad for segmentation, detection, pose); can discard useful detail.

`Remember:` **conv = "what is here", pooling = "roughly where, and how strong".**

`Exam:` "Explain pooling, its types, and why it is used." Formula + 2×2 numerical + pros/cons + one line on global pooling.

### 11.8 Equivariance vs invariance ★★

- **Translation equivariance:** $f(T_{\Delta x}) = T_\Delta(f(x))$ — shift the input, the feature map shifts by the same amount. **Convolution has this property** (weight sharing), and it is what you want for detection/segmentation.
- **Translation invariance:** $f(T_{\Delta x}) \approx f(x)$ — the output does not change. **Pooling gives approximate local invariance**, which is what you want for classification ("a cat is a cat wherever it is").

**The distinction in one line:** *convolution is equivariant, pooling makes it approximately invariant.* A CNN is a stack of equivariant layers with periodic invariance.

**CNN as a strong prior:** the architecture *assumes* locality, translation structure, and hierarchy. That is why it needs less data than a fully connected network — the prior replaces data.

---

## 12. CNN architecture and famous models ★★★

### 12.1 The standard pipeline (draw this)

```
Input image
   ↓
[ CONV → ReLU → (POOL) ] × N        ← feature extraction
   ↓
Flatten / GlobalAveragePooling
   ↓
[ Dense → ReLU → Dropout ] × M      ← classifier
   ↓
Softmax → class probabilities
```

**Role of each block:**

| Block | Role |
|---|---|
| Convolution | learns local features; each filter → one feature map |
| ReLU | non-linearity; keeps positive evidence, kills negatives |
| Pooling | downsample, enlarge receptive field, add invariance |
| Flatten/GAP | convert maps to a vector |
| Dense | combine features into a decision |
| Softmax | turn scores into probabilities |

**Design choices that define an architecture:** number of conv layers; filters per layer (usually doubling as spatial size halves: 32→64→128); kernel size; stride/padding; pooling type; number of FC layers and units; activation; dropout; skip connections; normalisation.

**Shape tracing example** (a guaranteed exam question — write the shape after every line):

```
Input              (3, 32, 32)
Conv 32, 3×3, same (32, 32, 32)
MaxPool 2×2        (32, 16, 16)
Conv 64, 3×3, same (64, 16, 16)
MaxPool 2×2        (64, 8, 8)
Conv 128, 3×3      (128, 6, 6)      ← valid padding loses 2
GAP                (128,)
Dense 10           (10,)
```

### 12.2 The model zoo ★

| Model | Year | Key idea | Why it mattered |
|---|---|---|---|
| **LeNet-5** | 1998 | conv + subsampling + FC, 5×5 kernels | first successful CNN (digits, cheques) |
| **AlexNet** | 2012 | 8 layers, ReLU, dropout, GPU, data augmentation | won ImageNet by 10 points; started the DL boom |
| **VGG-16/19** | 2014 | only 3×3 convs, stacked deep, 16–19 layers | showed depth + small kernels is enough; simple and modular |
| **GoogLeNet / Inception** | 2014 | inception module: 1×1, 3×3, 5×5, pool in **parallel**, concatenated; 1×1 bottlenecks | efficient multi-scale features, 22 layers |
| **ResNet** | 2015 | **skip connection** $y = F(x) + x$ | trains 50–152+ layers; solves degradation/vanishing gradient |
| **DenseNet** | 2017 | each layer connected to all later layers | feature reuse, fewer parameters |
| **MobileNet** | 2017 | depthwise separable convolution | mobile/edge deployment |
| **EfficientNet** | 2019 | compound scaling of depth/width/resolution | best accuracy per FLOP |

**ResNet's equation** (know it):

$$
y = F(x, \{W_i\}) + x
\qquad
\frac{\partial y}{\partial x} = 1 + \frac{\partial F}{\partial x}
$$

The $+1$ in the gradient means the gradient can never vanish completely through a block — the identity path is a gradient highway. This is the single most important architectural idea after convolution itself.

**1×1 convolution** — a "channel mixer": it computes a weighted sum across channels at each pixel. Uses: dimensionality reduction (bottlenecks), adding non-linearity cheaply, and in GoogLeNet/MobileNet as the pointwise stage. Cost per pixel = $C_{\text{in}} \cdot C_{\text{out}}$, no spatial mixing.

**Applications of CNNs:** classification, localisation, object detection (YOLO, Faster R-CNN), semantic/instance segmentation (U-Net, Mask R-CNN), face recognition, pose estimation, OCR, medical imaging, barcode/QR scanning.

**Classification vs localisation vs detection vs segmentation:**

| Task | Output |
|---|---|
| Classification | one label per image |
| Localisation | label + bounding box |
| Detection | boxes + labels for **many** objects |
| Semantic segmentation | a class label per **pixel** |
| Instance segmentation | a class + object id per pixel |

`Remember:` **LeNet → AlexNet → VGG → GoogLeNet → ResNet** = "**L**ittle **A**lways **V**ery **G**ood **R**esults" (1998, 2012, 2014, 2014, 2015).

`Exam:` "Draw and explain the architecture of a CNN." Pipeline diagram + each block's role + one shape table. If they ask "CNN vs ANN", use §11.1's three problems.

---

## 13. Efficient convolution algorithms ★★★

This section is the most "numerical" part of the post-mid syllabus. Learn the four cost formulas and the selection rule.

### 13.1 Where the cost is

$$
\begin{aligned}
\text{Standard convolution cost} &= H' \cdot W' \cdot C_{\text{out}} \cdot K^2 \cdot C_{\text{in}} \\
\text{Standard parameter count} &= (K^2 \cdot C_{\text{in}} + 1) \cdot C_{\text{out}}
\end{aligned}
$$

The cost is **linear in every one of** output pixels, kernel area, input channels, output channels. Any speed-up attacks one of these factors.

### 13.2 im2col + GEMM

Convert every local patch into a column, so the convolution becomes one big matrix multiplication:

```
patches: (H'W') × (K²C_in)      filters: (K²C_in) × C_out
→ one GEMM:  (H'W') × C_out
```

**Advantage:** BLAS/cuBLAS matrix multiplication is extremely optimised, so a naive-looking algorithm wins in practice.
**Trade-off:** the im2col buffer repeats each input value $K^2$ times → up to $K^2\times$ more memory. That is why frameworks use tiled/streaming versions.

**Winograd** goes further: for 3×3 kernels it reduces multiplications by ~2.25× using a transform, at the cost of extra additions and numerics. FFT is the other transform-based option.

### 13.3 FFT convolution

**Convolution theorem:**

$$
\mathcal{F}(f * g) = \mathcal{F}(f) \cdot \mathcal{F}(g)
\qquad \implies \qquad
f * g = \mathcal{F}^{-1}\big( \mathcal{F}(f) \cdot \mathcal{F}(g) \big)
$$

Steps: FFT the image and the kernel → multiply pointwise → inverse FFT. Cost per channel:

$$
\begin{aligned}
\text{direct:}\quad & O(N^2 K^2) \qquad (N^2 \text{ output pixels} \times K^2 \text{ mults}) \\
\text{FFT:}\quad & O(N^2 \log N)
\end{aligned}
$$

**When it wins:** large kernels on large feature maps, where $K^2 > \log N$. For 3×3 kernels on small maps, the FFT overhead loses.

**Limitations:** complex arithmetic, boundary/padding handling, memory for spectra, and per-channel FFTs (the channel sum still has to happen).

**Frequency intuition:** low frequencies = smooth regions and overall brightness; high frequencies = edges, texture, noise. A blur is a low-pass filter; an edge detector is high-pass.

### 13.4 Spatially separable convolution

If a kernel is **rank-1**, it factorises into a column times a row:

$$
K = u v^\top
\qquad \text{e.g. } \text{Sobel}_x = \begin{bmatrix} 1 & 2 & 1 \end{bmatrix}^\top \cdot \begin{bmatrix} -1 & 0 & 1 \end{bmatrix}
$$

Then convolve with the $K \times 1$ kernel first, then with the $1 \times K$ kernel:

$$
\begin{aligned}
\text{cost per location:}\quad & K^2 \to 2K \\
3\times 3:\ 9 \text{ mults} \to 6 \qquad
5\times 5:\ 25 \to 10 \qquad
7\times 7:\ 49 \to 14
\end{aligned}
$$

**Condition:** the kernel must be exactly separable. In a learned CNN, arbitrary kernels are not, so this is only used where the structure is known (classical image processing, some hand-designed filters) or when a low-rank factorisation is imposed.

### 13.5 Depthwise separable convolution (MobileNet) ★★★

Two stages:

1. DEPTHWISE: one $K \times K$ filter per input channel (no channel mixing); cost $= H'W' \cdot C_{\text{in}} \cdot K^2$.
2. POINTWISE: $1 \times 1$ conv with $C_{\text{out}}$ filters (mixes channels); cost $= H'W' \cdot C_{\text{in}} \cdot C_{\text{out}}$.

$$
\text{Total} = H'W' \cdot C_{\text{in}} \cdot (K^2 + C_{\text{out}})
$$

**Reduction ratio vs standard:**

$$
\frac{\text{separable}}{\text{standard}} = \frac{K^2 + C_{\text{out}}}{K^2 \cdot C_{\text{out}}} = \frac{1}{C_{\text{out}}} + \frac{1}{K^2}
$$

For $K = 3$, $C_{\text{out}} = 256$: $1/256 + 1/9 \approx 0.115$ → about **8.7× fewer multiplications**.

**Parameter comparison** (the slide's example: $C_{\text{in}} = 3$, $C_{\text{out}} = 256$, $K = 3$):

$$
\begin{aligned}
\text{standard:}\quad & K^2 \cdot C_{\text{in}} \cdot C_{\text{out}} = 9 \cdot 3 \cdot 256 = 6{,}912 \\
\text{separable:}\quad & K^2 \cdot C_{\text{in}} + C_{\text{in}} \cdot C_{\text{out}} = 27 + 768 = 795 \qquad (8.7\times \text{ fewer})
\end{aligned}
$$

**Full numerical** (12×12×3 input, 256 3×3 filters, no padding → 10×10 output):

$$
\begin{aligned}
\text{standard:}\quad & 10 \cdot 10 \cdot 256 \cdot 9 \cdot 3 = 691{,}200 \\
\text{depthwise:}\quad & 10 \cdot 10 \cdot 3 \cdot 9 = 2{,}700 \\
\text{pointwise:}\quad & 10 \cdot 10 \cdot 3 \cdot 256 = 76{,}800 \\
\text{separable:}\quad & 79{,}500 \qquad (\approx 8.7\times \text{ fewer})
\end{aligned}
$$

**Per-output-pixel version** (32×32×3 image, 64 3×3 filters): standard $9 \cdot 3 \cdot 64 = 1728$; depthwise $9 \cdot 3 = 27$ + pointwise $3 \cdot 64 = 192$ → **219** (7.9× fewer).

**Used in:** MobileNet, Xception, EfficientNet, and most mobile/edge vision models. Accuracy is usually within ~1% of standard convolution.

### 13.6 Choosing a method ★★

| Situation | Choose | Why |
|---|---|---|
| Very large kernel on a large map | **FFT** | $K^2$ dominates $\log N$ |
| Mobile / limited compute | **depthwise separable** | $1/C_{\text{out}} + 1/K^2$ reduction |
| General GPU training | direct / im2col-GEMM / Winograd | hardware-optimised, exact |
| Exactly rank-1 kernel | spatially separable | $K^2 \to 2K$, exact |
| Many channels, need fewer | 1×1 bottleneck | cheap channel reduction |

| | Standard | Spatially separable | Depthwise separable |
|---|---|---|---|
| Decomposition | none | $K \times 1$ then $1 \times K$ | depthwise then pointwise |
| Channel mixing | yes | yes | only in pointwise |
| Params | highest | lower | much lower |
| Works for learned kernels? | yes | only if separable | always |
| Used in | classic CNNs | classical filters | MobileNet family |

`Remember:` depthwise ratio **$1/C_{\text{out}} + 1/K^2$** — two fractions, one for each stage. If you forget, recompute: the standard has $K^2 C_{\text{in}} C_{\text{out}}$; the separable has $K^2 C_{\text{in}} + C_{\text{in}} C_{\text{out}}$; divide through by $K^2 C_{\text{in}} C_{\text{out}}$.

`Exam:` "Compute the number of multiplications for standard vs depthwise separable convolution for a 12×12×3 input with 256 3×3 filters." Write the two formulas, substitute, then give the ratio and the conclusion sentence ("~8.7× cheaper, hence used in MobileNet").

---

## 14. Unsupervised CNN features ★

Before supervised pretraining was cheap, people learned features **without labels**. This is still examinable as "how can we learn features without labels?".

### 14.1 The general objective

$$
\min_\theta \sum_i L\big( x_i, D_\theta(E_\theta(x_i)) \big) \qquad (\text{reconstruction})
$$

or "find a representation where patches cluster / are sparse / are predictable".

### 14.2 Patch-based pipeline with k-means

1. Extract random patches (e.g. 6×6) from unlabelled images.
2. Normalise each patch (subtract mean, divide by std).
3. Cluster with k-means into $K$ centroids.
4. Each centroid becomes a convolution filter.
5. Convolve the image with all $K$ filters $\to$ $K$ feature maps.
6. Pool the maps $\to$ feature vector $\to$ classifier (SVM/logistic regression).

**k-means objective:**

$$
\min_\mu \sum_i \sum_j r_{ij} \Vert x_i - \mu_j \Vert^2
\qquad (r_{ij} = 1 \text{ if } x_i \text{ belongs to cluster } j)
$$

**Emergent filters:** the learned centroids turn out to be edge and colour-blob detectors — the same features a supervised CNN learns in layer 1. That is the punchline: *the first layer of a CNN is largely data-driven, not label-driven.*

### 14.3 Sparse coding

Represent each patch as a **sparse** linear combination of a learned dictionary:

$$
x \approx D\alpha
\qquad \text{with} \quad \min_{D,\alpha} \Vert x - D\alpha \Vert^2 + \lambda \Vert \alpha \Vert_1
$$

- $D$ = overcomplete dictionary (more atoms than dimensions).
- $\Vert \alpha \Vert_1$ forces most coefficients to zero → each patch is explained by a few atoms.
- The learned atoms are again edge-like, and resemble Gabor filters / V1 receptive fields.

**Connection to CNNs:** sparse coding is an unsupervised single-layer network with an L1 penalty; the dictionary is the weight matrix. Stacking such layers = greedy layer-wise pretraining (§27).

### 14.4 Hierarchy of features ★

| Layer | Learns | Task specificity |
|---|---|---|
| 1–2 | edges, colours, blobs | generic, reusable |
| 3–4 | textures, object parts | somewhat generic |
| 5+ | whole objects, scenes | task-specific |

**Why deeper features are more task-specific:** each layer composes the previous layer's features to serve the final loss. Early layers only need to preserve information useful for *any* vision task; later layers are shaped by the particular label set. This is exactly why **transfer learning freezes early layers and fine-tunes late ones**.

`Remember:` **"Early layers see edges, late layers see classes."**

---

## 15. CNN training details ★★

- **Data augmentation** is essential — CNNs memorise. Random crop, flip, rotation, colour jitter, cutout/mixup.
- **Batch normalisation** after conv, before ReLU (the original paper) or after (modern practice); it makes deep CNNs trainable with a high LR.
- **Dropout** is usually weaker in conv layers than in dense layers because weight sharing already regularises; `SpatialDropout2D` drops whole channels instead.
- **Weight decay** (L2) is the standard regulariser for CNNs.
- **Learning rate**: Adam 1e−3 or SGD+momentum 0.1 with cosine decay; CNNs often generalise slightly better with SGD+momentum.
- **Transfer learning** beats training from scratch whenever your dataset is smaller than ImageNet (§28).

---

# PART III — Sequence models

## 16. Why sequences need their own architecture

### 16.1 What makes sequence data different ★

In a sequence, **order carries meaning**: "dog bites man" ≠ "man bites dog". A feed-forward network sees a fixed-size vector and treats every position independently, so it cannot represent order or variable length.

Formally, sequence data violates three assumptions of a dense network:

1. **Variable length** — a dense layer needs a fixed input size, so you would need padding to the longest sequence (wasting compute) or truncation (losing information).
2. **Order matters** — permuting the inputs changes the answer, but an MLP is permutation-invariant on the input vector once flattened.
3. **Parameter sharing across time** — a pattern that appears at position 3 should be recognised at position 300, but a dense layer has separate weights for each position, so parameters explode with sequence length.

### 16.2 The five input–output patterns ★★

| Pattern | Mapping | Example |
|---|---|---|
| one-to-one | $x \to y$ | plain classification (not a sequence) |
| one-to-many | one input → sequence | image captioning |
| many-to-one | sequence → one output | sentiment analysis, sequence classification |
| many-to-many (aligned) | $T$ in → $T$ out | POS tagging, per-frame video labelling |
| many-to-many (unaligned) | $T$ in → $T'$ out | machine translation, summarisation (encoder–decoder) |

**Activity-style classification:** sentiment → many-to-one; music generation → one-to-many; video frame labelling → many-to-many aligned; translation → many-to-many unaligned; stock prediction from a window → many-to-one.

### 16.3 The basic RNN ★★★

An RNN keeps a **hidden state** that summarises everything seen so far, and reuses the same weights at every time step.

$$
\begin{aligned}
h_t &= \phi\big(W_{xh} x_t + W_{hh} h_{t-1} + b_h\big) \\
o_t &= W_{hy} h_t + b_y \\
\hat{y}_t &= \psi(o_t)
\end{aligned}
$$

- $\phi$ is usually `tanh` (sometimes ReLU); $\psi$ is sigmoid/softmax for classification, linear for regression.
- $W_{xh}$ (input→hidden), $W_{hh}$ (hidden→hidden, the **recurrent** weights), $W_{hy}$ (hidden→output) are **shared across all time steps**.
- $h_0$ is the zero vector (or a learned initial state).

**Why it is called recurrent:** the same operation is applied repeatedly; the network's output at time $t$ depends on its own previous output. Unrolled over $T$ steps it is a very deep network with $T$ layers that **share weights**.

**As a dynamical system** (slide phrasing):

$$
h_t = f(h_{t-1}, x_t) \qquad \implies \qquad h_t = f(f(\dots f(h_0, x_1)\dots, x_{t-1}), x_t)
$$

The final state depends recursively on *all* previous inputs. That is memory.

**Parameter count** (common 2-mark question):

$$
\text{params} = \underbrace{d\cdot h}_{\text{input}} + \underbrace{h\cdot h}_{\text{recurrent}} + \underbrace{h}_{\text{hidden bias}} + \underbrace{h\cdot o + o}_{\text{output}}
$$

Example $d = 6, h = 4, o = 1$: $24 + 16 + 4 + 4 + 1 = 49$ (44 weights + 5 biases).
> Your notes say 44 — that is weights only. Write "44 weights + 5 biases = 49 parameters".

Note that the count is **independent of sequence length** — that is the whole point of weight sharing.

### 16.4 Forward propagation numerical ★★★

Scalar RNN, $W_{xh} = 0.5$, $W_{hh} = 0.8$, $b = 0$, $W_{hy} = 1$, $h_0 = 0$, input $x = (1, 2)$, $\phi = \tanh$, $\psi = \sigma$.

$$
\begin{aligned}
t=1:\quad h_1 &= \tanh(0.5\cdot 1 + 0.8\cdot 0) = \tanh(0.5) = 0.4621 \\
t=2:\quad h_2 &= \tanh(0.5\cdot 2 + 0.8\cdot 0.4621) = \tanh(1.3697) = 0.8786 \\
\text{out:}\quad \hat{y} &= \sigma(1\cdot 0.8786) = 0.707
\end{aligned}
$$

Slide-style check with $W_{xh} = W_{hh} = 1$, $b = 0$, $x = (1,1,1)$:
$h_1 = \tanh(1) = 0.7616$; $h_2 = \tanh(1.7616) = 0.9427$; $h_3 = \tanh(1.9427) = 0.9597$.

**Interpretation to write after the numbers:** the state is saturating — each new input changes $h$ less than the previous one, the classic short-memory effect of tanh.

### 16.5 One-hot encoding and input shape ★★

Networks take numbers, so tokens are encoded. With vocabulary size $V$, one-hot encoding gives a $V$-length vector with a single 1. Example $\{\text{movie}, \text{is}, \text{good}, \text{bad}, \text{not}, \text{so}\}$ ($V = 6$): $\text{movie} = [1,0,0,0,0,0]$, $\text{is} = [0,1,0,0,0,0]$, …

A batch has shape `(samples, timesteps, features)`: 3 sentences padded to 5 tokens with $V = 6$ → $(3, 5, 6)$.

**Problems with one-hot:** no similarity information (cat and dog are as far apart as cat and car), dimension = vocabulary size, and sparse. **Fix: embeddings** — a learned $V\times E$ lookup table; $E \approx 50$–$300$. Embeddings place similar words nearby and are trained jointly with the network.

`Remember:` **RNN = "same layer applied at every tick, with a memory wire".** Draw the unrolled chain: $x_1 \to [\text{RNN}] \to h_1 \to [\text{RNN}] \to h_2 \to \dots$, with $h_{t-1}$ entering from the left.

`Exam:` "Explain RNN architecture and perform forward propagation for 2 time steps." Diagram + equations + 2 rows of arithmetic + parameter count.

---

## 17. Training RNNs: BPTT and gradient stability ★★★

### 17.1 Loss and why BPTT is needed

The loss is the sum over time steps:

$$
L = \sum_{t=1}^{T} L_t(\hat{y}_t, y_t)
$$

Because $W_{hh}$ is shared, it influences **every** future state, so its gradient is a sum over all time steps:

$$
\frac{\partial L}{\partial W_{hh}} = \sum_t \frac{\partial L}{\partial h_t}\,\frac{\partial h_t}{\partial W_{hh}}
$$

**BPTT = backpropagation through time:** unroll the network over $T$ steps into a $T$-layer feed-forward graph (with tied weights), then run ordinary backprop. At each step you accumulate the gradient contributions, and the tied weights receive the **sum** of the per-step gradients.

**Recurrent error signal.** Define $\delta_t = \partial L/\partial h_t$. Then

$$
\begin{aligned}
\delta_t &= \frac{\partial L_t}{\partial h_t} + \left(\frac{\partial h_{t+1}}{\partial h_t}\right)^{\top} \delta_{t+1} \\
&= \frac{\partial L_t}{\partial h_t} + \big(W_{hh}^{\top}\operatorname{diag}(\phi'(z_{t+1}))\big)\,\delta_{t+1}
\end{aligned}
$$

and the parameter gradient is $\partial L/\partial W_{hh} = \sum_t \delta_t h_{t-1}^{\top}$.

### 17.2 Why long-range gradients vanish or explode ★★★

The Jacobian of one step is

$$
\frac{\partial h_t}{\partial h_{t-1}} = \operatorname{diag}(\phi'(z_t))\, W_{hh}
$$

So the influence of step $k$ on step $t$ is a **product of $t-k$ Jacobians**:

$$
\frac{\partial h_t}{\partial h_k} = \prod_{j=k+1}^{t} \operatorname{diag}(\phi'(z_j))\, W_{hh}
$$

If the largest singular value $\rho$ of $W_{hh}$ and the max derivative $\gamma$ satisfy $\rho\gamma < 1$, the product decays like $(\rho\gamma)^{t-k}$:

$$
\begin{aligned}
0.5^{10} &= 1/1024 \approx 0.001 \\
0.5^{20} &\approx 10^{-6} \\
1.5^{10} &\approx 57.7 & \to \text{exploding} \\
1.5^{20} &\approx 3325 & \to \text{overflow}
\end{aligned}
$$

That is the numerical activity in the slides: after ten steps with factor 0.5 the gradient is a thousandth; with 1.5 it is ~58 times larger.

**Consequences of vanishing:** the network cannot learn dependencies longer than ~10–20 steps; it behaves as if it has no memory; early layers/steps barely update.
**Consequences of exploding:** huge updates, loss oscillation, NaN, divergence.

### 17.3 Fixes ★★★

| Fix | What it does |
|---|---|
| **Gradient clipping** | rescales the gradient so its norm ≤ τ; only addresses exploding |
| **Truncated BPTT** | backprop only $K$ steps back; lower memory, faster, but cannot learn dependencies longer than $K$ |
| **LSTM / GRU** | gated additive state; addresses vanishing |
| **Better init / orthogonal $W_{hh}$** | keeps $\rho \approx 1$, so signals neither vanish nor explode |
| **Layer normalisation** | stabilises hidden-state scale |
| **Teacher forcing** | feed the true previous token during training (seq2seq) |

**Gradient clipping by global norm:**

$$
\text{if } \Vert g\Vert_2 > \tau:\quad g \leftarrow \tau g / \Vert g\Vert_2
$$

**Truncated BPTT pipeline:** split the sequence into windows of length $K$; carry $h$ forward across windows (detached), but backprop only within the window.

**Practical RNN training recipe** (slide's list): use LSTM/GRU rather than vanilla RNN; clip gradients ($\tau \approx 1\text{--}5$); use a moderate LR with decay; batch sequences of similar length; use teacher forcing for generation tasks; monitor gradient norms.

`Remember:` **"BPTT is backprop on an unrolled graph; the gradient is a product of Jacobians, and a long product of numbers < 1 is zero."**

`Exam:` "Why do simple RNNs suffer from vanishing gradients? Derive it." Write $\partial h_t/\partial h_k = \prod \operatorname{diag}(\phi')W_{hh}$, say "product of $t-k$ factors each < 1", give the $0.5^{10}$ number, then the two fixes.

---

## 18. LSTM ★★★

### 18.1 Motivation

A vanilla RNN overwrites its whole state at every step:

$$
h_t = \tanh(W_{xh} x_t + W_{hh} h_{t-1} + b)
$$

There is no way to *keep* something unchanged across many steps — the tanh squashes and the matrix mixes. LSTM adds a **cell state $c_t$** that runs along the top like a conveyor belt, modified only by **addition** and **element-wise multiplication**, and three **gates** that decide what to erase, write and read.

### 18.2 The equations (memorise exactly) ★★★

With $u_t = [h_{t-1} ; x_t]$ (concatenation) and $\odot$ = element-wise product:

$$
\begin{aligned}
\text{forget gate}\qquad f_t &= \sigma(W_f u_t + b_f) \\
\text{input gate}\qquad i_t &= \sigma(W_i u_t + b_i) \\
\text{candidate}\qquad \tilde{c}_t &= \tanh(W_c u_t + b_c) \\
\text{cell update}\qquad c_t &= f_t \odot c_{t-1} + i_t \odot \tilde{c}_t \\
\text{output gate}\qquad o_t &= \sigma(W_o u_t + b_o) \\
\text{hidden state}\qquad h_t &= o_t \odot \tanh(c_t)
\end{aligned}
$$

Equivalent split form used in some slides: $f_t = \sigma(W_f x_t + U_f h_{t-1} + b_f)$, etc.

**What each gate means:**

| Gate | Question | Effect |
|---|---|---|
| $f_t$ (forget) | what of the old memory do I keep? | $f \to 1$ keep, $f \to 0$ erase |
| $i_t$ (input) | how much of the new candidate do I write? | $i \to 1$ write, $i \to 0$ ignore |
| $\tilde{c}_t$ (candidate) | what is the new information? | `tanh` → range $(-1,1)$ |
| $o_t$ (output) | how much of the memory do I expose? | controls $h_t$ |

**Shapes.** All of $f, i, o, \tilde{c}, c, h$ have the same length `h_units`. The gates are vectors, not scalars — each memory slot has its own forget switch.

**Why `tanh` for the candidate and $\sigma$ for the gates:** gates must be in (0,1) to act as soft switches; the candidate can be negative (it is *content*, not a valve).

**Parameters:**

$$
\text{params} = 4 \cdot \big[(h + d)\cdot h + h\big]
$$

(4 blocks — forget, input, candidate, output — each with a $(h+d) \times h$ weight matrix and an $h$ bias.) Example $d = 3, h = 4$: $4\cdot(7\cdot 4 + 4) = 128$. That is ~4× an RNN of the same width, which is the price of gating.

### 18.3 Why the LSTM fixes vanishing gradients ★★★ (the derivation)

From $c_t = f_t \odot c_{t-1} + i_t \odot \tilde{c}_t$:

$$
\frac{\partial c_t}{\partial c_{t-1}} = \operatorname{diag}(f_t)
$$

Across many steps:

$$
\frac{\partial c_t}{\partial c_k} = \prod_{j=k+1}^{t} \operatorname{diag}(f_j)
$$

**If the forget gates stay near 1, this product stays near 1** — the gradient flows back essentially unchanged. Compare the RNN, where every step multiplies by $\operatorname{diag}(\phi')W_{hh}$ (a small number). The LSTM replaced *repeated nonlinear replacement* with *additive accumulation*; addition has derivative 1.

The gates can also **learn** to open ($f \approx 1$) when a dependency must be preserved — e.g. "the gender of the subject was mentioned 40 tokens ago". This is why LSTMs handle long-range dependencies.

### 18.4 Numerical (scalar, one unit) ★★★

Given $c_{t-1} = 0.5$ and pre-activations $z_f = 0.4, z_i = 0.8, z_c = 0.6, z_o = 1.0$:

$$
\begin{aligned}
f &= \sigma(0.4) = 0.5987 \\
i &= \sigma(0.8) = 0.6900 \\
\tilde{c} &= \tanh(0.6) = 0.5370 \\[4pt]
c_t &= 0.5987\cdot 0.5 + 0.6900\cdot 0.5370 = 0.2993 + 0.3705 = 0.6699 \\
o_t &= \sigma(1.0) = 0.7311 \\
h_t &= 0.7311 \cdot \tanh(0.6699) = 0.7311 \cdot 0.5849 = 0.4276
\end{aligned}
$$

Then $(c_t, h_t)$ go to the next time step. Note the memory **grew** (0.5 → 0.67) because the input gate wrote more than the forget gate erased.

**Gate-interpretation activity examples (slide style):**
- $f = 0, i = 0$ → $c_t = 0$: memory fully reset.
- $f = 1, i = 0$ → $c_t = c_{t-1}$: memory perfectly preserved, the "copy" behaviour.
- $f = 1, i = 1, \tilde{c} = \text{new}$ → old memory plus new content: accumulation.
- $o = 0$ → $h_t = 0$ even though $c_t$ is non-zero: memory stored but not exposed.

### 18.5 Variants ★★

- **GRU** (Gated Recurrent Unit): two gates instead of three, no separate cell state.
  $$
  \begin{aligned}
  z_t &= \sigma(W_z u_t) & \text{update gate} \\
  r_t &= \sigma(W_r u_t) & \text{reset gate} \\
  \tilde{h}_t &= \tanh(W_h [r_t \odot h_{t-1} ; x_t]) \\
  h_t &= (1 - z_t) \odot h_{t-1} + z_t \odot \tilde{h}_t
  \end{aligned}
  $$
  ~25% fewer parameters than LSTM, often similar accuracy, faster.
- **Peephole LSTM:** gates also see $c_{t-1}$.
- **Stacked (deep) LSTM:** $h_t$ of layer $l$ is the input of layer $l+1$.
- **ConvLSTM:** replaces the linear maps with convolutions (video, precipitation nowcasting).

`Remember:` **FICO** — **F**orget, **I**nput, **C**andidate, **O**utput. Every gate is $\sigma(W[h;x]+b)$; only the candidate is `tanh`. The cell update is $c = f\odot c_{\text{old}} + i\odot \tilde{c}$ ("keep + write").

`Exam:` "Draw the LSTM cell and write all equations." Draw the horizontal $c$ line, the three $\sigma$ boxes, the tanh box, the two $\odot$ and one $\oplus$. Then write the six equations. Then one line: "the additive cell update with $\partial c_t/\partial c_{t-1} = f_t$ is why gradients survive."

---

## 19. Bidirectional LSTM and sequence-to-sequence ★★

### 19.1 Bi-LSTM

Two independent LSTMs run over the same sequence, one forward and one backward; their states are concatenated:

$$
\begin{aligned}
\overrightarrow{h}_t &= \operatorname{LSTM}_{\text{fwd}}(x_t, \overrightarrow{h}_{t-1}) \\
\overleftarrow{h}_t &= \operatorname{LSTM}_{\text{bwd}}(x_t, \overleftarrow{h}_{t+1}) \\
h_t &= [\,\overrightarrow{h}_t ; \overleftarrow{h}_t\,] \qquad \text{(or summed / averaged)}
\end{aligned}
$$

**Advantage:** every output sees both past and future context — essential for tagging, NER, speech, and any task where the whole sentence is available.
**Limitation:** **not usable for strict real-time / causal prediction**, because the backward pass needs the entire future. It also doubles the parameters and compute. For next-token prediction you must use a unidirectional (causal) model.

### 19.2 RNN vs LSTM vs Bi-LSTM summary ★★

| | RNN | LSTM | Bi-LSTM |
|---|---|---|---|
| State | $h$ only | $c$ and $h$ | two $(c,h)$ pairs |
| Gates | none | 3 + candidate | 3 + candidate, twice |
| Long dependencies | poor | good | good |
| Uses future context | no | no | yes |
| Real-time capable | yes | yes | no |
| Parameters | $d\cdot h + h^2 + h + h\cdot o + o$ | $4[(h+d)h + h]$ | ≈ 2× LSTM |

### 19.3 Encoder–decoder (seq2seq) ★★★ (your notes' starred questions)

For unaligned many-to-many tasks (translation), split the network into two halves:

$$
\begin{aligned}
\text{ENCODER:}\quad & x_1 \dots x_T \to h_1 \dots h_T \to \text{context vector } c \\
\text{DECODER:}\quad & c \to y_1 \dots y_{T'} \qquad \text{(one token at a time, its own output fed back)}
\end{aligned}
$$

**Block diagram (starred in your notes — draw this):**

```
  "Nice to meet you"                                        "आप से मिलकर अच्छा लगा"
        │                                                              ▲
   ┌────▼─────┐        context vector        ┌──────────┐              │
   │ ENCODER  │ ──────────────────────────→  │ DECODER  │ ─────────────┘
   │ (RNN/LSTM stack)│                        │ (RNN/LSTM stack)│
   └──────────┘                               └──────────┘
```

**How each half works (three starred points):**

1. Words arrive as **vectors** (one-hot or, better, embeddings) — the encoder is a stack of RNN/LSTM cells.
2. After processing the whole sequence, the encoder produces a **context vector**: a summary of the entire input.
3. The context vector is fed to the decoder, which processes it and emits the translation **word by word** (softmax over the vocabulary at each step).

**Context vector — role and content:** it is the fixed-size summary of the input sentence, carrying the information the decoder needs to produce the output. It is the **only** channel between the two halves, which is exactly its weakness.

**`<start>` and `<end>` tokens:** two extra tokens in the vocabulary. `<start>` tells the decoder "begin generating"; `<end>` lets the decoder stop, so output length is variable.

**Decoder output:** $p(w_t \mid \text{context}, \text{previous words}) = \operatorname{softmax}(W_o s_t + b_o)$ — a multi-class classification over the vocabulary; the highest-probability word (or a sampled one) is emitted.

**Improving encoder–decoder performance (your notes' two answers):**
1. **Embeddings** — replace one-hot vectors with a learned dense embedding, which reduces dimension and puts similar words nearby.
2. **Deep LSTM** — stack several LSTM layers so the encoder/decoder can learn hierarchical representations.

**Teacher forcing:** during training feed the *true* previous token rather than the model's own prediction — faster and more stable. At inference you must feed the model's own output (exposure bias: the model never saw its own mistakes).

**Drawbacks (your notes' exam question):**
1. The context vector is a **bottleneck**: if the summary is inaccurate, the translation is wrong.
2. **Long sentences** cannot be compressed into one fixed vector — performance degrades sharply with length.
3. It is **time-consuming** and cannot be parallelised over the sequence (sequential decoder).

→ These three drawbacks are precisely what **attention** was invented to fix.

### 19.4 Attention mechanism ★★★ (your notes: "Attention eqn → 5 marks")

**Idea:** instead of one context vector for the whole sentence, build a **new context vector at every decoder step**, as a weighted sum of *all* encoder hidden states. The decoder learns which input words to look at for each output word.

**Equations to derive (this is the 5-mark answer):**

$$
\begin{aligned}
&\text{Encoder states: } h_0, h_1, \dots, h_{n-1} \\
&\text{Decoder state: } s_{i-1} \quad \text{(state before producing output } i\text{)} \\[6pt]
&\text{Step 1 - score every encoder state against the decoder state:} \\
&\qquad \alpha_{ij} = f(h_{j-1}, s_{i-1}) \\[6pt]
&\text{Step 2 - normalise the scores into weights (they sum to 1):} \\
&\qquad \alpha_{ij} = \exp(\text{score}_{ij}) / \sum_k \exp(\text{score}_{ik}) \\[6pt]
&\text{Step 3 - context vector for decoder step } i\text{:} \\
&\qquad C_i = \sum_{j=0}^{n-1} \alpha_{ij} h_{j-1}
\end{aligned}
$$

Expanded for a 4-word input (your notes' worked form):

$$
C_i = \alpha_{i1} h_0 + \alpha_{i2} h_1 + \alpha_{i3} h_2 + \alpha_{i4} h_3
$$

and if $h = [1\ 0\ 0\ 0]$ is a **4-dimensional** vector, the number of inputs is **4** — the length of the context vector equals the number of encoder states, while its *dimension* equals the hidden size.

**Score functions you may use for $f$:**

$$
\begin{aligned}
\text{dot product:}\quad & \text{score} = h_{j-1}^{\top} s_{i-1} \\
\text{general:}\quad & \text{score} = h_{j-1}^{\top} W s_{i-1} \\
\text{concat (Bahdanau):}\quad & \text{score} = v^{\top} \tanh(W_h h_{j-1} + W_s s_{i-1})
\end{aligned}
$$

**Exam strategy written in your notes:** *"Write the reasons first, then explain attention using a diagram and an example. Derive C_ij and α_ij."*

**Worked micro-example.** Encoder states $h = [1, 0, 2]$, scores $[2, 1, 0]$:
$\alpha = \operatorname{softmax}([2,1,0]) = (0.665, 0.245, 0.090)$, so
$C = 0.665\cdot 1 + 0.245\cdot 0 + 0.090\cdot 2 = 0.845$.
Notice the context vector is a **blend**, not a copy — that is the whole point.

**Why attention solves the bottleneck:** the decoder no longer depends on one fixed summary; it can look directly at the relevant encoder state, however far back. The path length from any input to any output is **O(1)** instead of O(T).

**Drawbacks of attention (→ motivates the Transformer):**
1. **Time-consuming** — the decoder still runs step by step, so it cannot be parallelised.
2. **Computationally expensive** — every decoder step scores every encoder state.
3. Still built on RNN/LSTM, so it inherits **sequential training** and long-sequence difficulty.

→ **Transformers remove the recurrence entirely and keep only attention (plus MLPs).**

### 19.5 Recurrent vs recursive networks ★

| Recurrent | Recursive |
|---|---|
| chain-structured computation | tree-structured computation |
| one predecessor state | multiple child states |
| same weights at every time step | same weights at every tree node |
| speech frames, time series | parse trees, program ASTs, scene graphs |

**Memory hook:** *recurrent = a line, recursive = a tree.* Both share weights, but the topology differs.

`Remember:` **Bi-LSTM = "read the sentence twice, once backwards."**

`Exam:` "Why is Bi-LSTM unsuitable for real-time prediction?" Because the backward LSTM needs the whole future of the sequence; at time $t$ you do not have $x_{t+1}\dots x_T$ yet.

---

# PART IV — Latent variable models: PCA, PPCA, FA, ICA

## 20. Dimensionality reduction and PCA ★★

### 20.1 Why reduce dimensions?

Real data lies near a low-dimensional surface. Reducing $D$ features to $K \ll D$ while keeping most of the variance:

- fights the **curse of dimensionality** (in high dimensions data becomes sparse, distances lose meaning, and the number of samples needed grows exponentially);
- removes noise and redundant/correlated features;
- speeds up training and cuts memory;
- reduces overfitting;
- enables 2-D/3-D visualisation.

Your notes introduce PPCA/FA/ICA exactly this way: *"traditional tools for finding fixed hidden features in data — the curse of dimensionality makes models complex, so these reduce dimensionality."*

### 20.2 PCA (deterministic)

**Idea:** find new orthogonal axes (principal components) ordered by how much variance they explain. PC1 is the direction of maximum variance, PC2 the maximum of the remaining variance orthogonal to PC1, and so on.

**Steps (write these five):**

1. Centre (and usually standardise) the data: $X \leftarrow (X - \mu)/\sigma$
2. Covariance matrix: $C = \frac{1}{N-1} X^{\top} X$ ($D \times D$)
3. Eigen-decomposition: $C v = \lambda v$
4. Sort eigenvalues descending, keep the top $K$ eigenvectors $\to$ $W_K$ ($D \times K$)
5. Project: $Z = X W_K$ ($N \times K$)

**Explained-variance ratio:** $\lambda_i / \sum \lambda_j$. With eigenvalues 2 and 1, PC1 explains $2/3 = 66.7\%$. Choose $K$ for 90–95% cumulative variance (or look at the scree plot's "elbow").

**Key properties:**
- The components are **uncorrelated** (orthogonal) and ordered by variance.
- The projection minimises reconstruction error $\Vert X - ZW_K^{\top}\Vert^2$ — PCA is the best *linear* compressor in the least-squares sense.
- **Scaling matters:** always standardise first, otherwise a feature measured in millimetres dominates one in metres.
- PCA is **not** a generative model: no likelihood, no noise model, no way to sample meaningfully or handle missing data.

**t-SNE** (contrast): non-linear, preserves **local** neighbourhoods, great for 2-D visualisation, but distances between clusters are not meaningful and it is not stable across runs. Use PCA for compression, t-SNE/UMAP for pictures.

### 20.3 Probabilistic PCA (PPCA) ★★

**Definition in two points** (as your notes put it): (1) a dimensionality-reduction technique that works **probabilistically**; (2) it finds the **latent variables** — the hidden lower-dimensional causes — that generate the observed data.

**Generative model:**

$$
\begin{aligned}
z &\sim \mathcal{N}(0, I_K) &&\text{latent, } K \ll D \\
\epsilon &\sim \mathcal{N}(0, \sigma^2 I_D) &&\text{isotropic Gaussian noise} \\
x &= \mu + W z + \epsilon &&W \text{ is } D \times K \text{ (the loading matrix)}
\end{aligned}
$$

Therefore the likelihood is Gaussian: $p(x \mid z) = \mathcal{N}(\mu + Wz, \sigma^2 I)$.

**Marginal distribution** (integrate out $z$; the derivation is one line of covariance algebra):

$$
\begin{aligned}
E[x] &= \mu \\
\operatorname{Cov}(x) &= W \operatorname{Cov}(z) W^{\top} + \operatorname{Cov}(\epsilon) = W W^{\top} + \sigma^2 I \\
\implies x &\sim \mathcal{N}(\mu, W W^{\top} + \sigma^2 I)
\end{aligned}
$$

**Posterior** (what you actually want — the latent code of a given $x$): with $M = W^{\top}W + \sigma^2 I$ ($K \times K$),

$$
\begin{aligned}
p(z \mid x) &= \mathcal{N}\big(M^{-1} W^{\top}(x - \mu),\ \sigma^2 M^{-1}\big) \\
E[z \mid x] &= M^{-1} W^{\top} (x - \mu) \qquad \leftarrow \text{the low-dimensional representation}
\end{aligned}
$$

**Learning:** maximise the log-likelihood. There is a closed form via eigen-decomposition of the data covariance (keep the top $K$ eigenvectors, scale them), or use **EM**:

$$
\begin{aligned}
\text{E-step:}\quad & \text{compute } E[z \mid x] \text{ and } E[zz^{\top} \mid x] \\
\text{M-step:}\quad & W, \mu, \sigma^2 \leftarrow \text{update from those expectations}
\end{aligned}
$$

EM is the practical route when $D$ is large or data is missing.

**Why PPCA beats PCA:**

| Benefit | Explanation |
|---|---|
| Likelihood | you can compare models and choose $K$ statistically |
| Noise model | explicitly separates signal ($Wz$) from noise ($\epsilon$) |
| Uncertainty | the posterior is a distribution, not a point |
| Missing data | EM handles incomplete rows naturally |
| Sampling | it is a generative model — draw $z$, then $x$ |
| Extensions | mixtures of PPCA, Bayesian PPCA (automatic $K$) |

**Relation to PCA:** as $\sigma^2 \to 0$, $E[z \mid x]$ converges to the PCA projection. PCA is the zero-noise limit.

### 20.4 Factor Analysis (FA) ★★

Same linear latent model, but **diagonal** noise covariance:

$$
\begin{aligned}
x &= \mu + \Lambda z + \epsilon, \quad z \sim \mathcal{N}(0, I), \quad \epsilon \sim \mathcal{N}(0, \Psi), \quad \Psi = \operatorname{diag}(\psi_1 \dots \psi_D) \\
\implies x &\sim \mathcal{N}(\mu, \Lambda\Lambda^{\top} + \Psi)
\end{aligned}
$$

**Difference from PPCA:** PPCA assumes the *same* noise variance for every feature (isotropic); FA allows a *different* noise variance per feature (heteroscedastic). FA therefore has more parameters and more flexibility.

**Interpretation:** $\Lambda$ holds the **factor loadings** — how strongly each observed variable depends on each latent factor. FA is used to explain *correlations* between observed variables through a few common factors (psychometrics, finance: "the market factor").

**Rotation ambiguity:** $\Lambda R$ with $R$ orthogonal gives the same covariance ($\Lambda RR^{\top}\Lambda^{\top} = \Lambda\Lambda^{\top}$), so the factors are only identified up to rotation. Analysts rotate them (varimax) for interpretability.

### 20.5 ICA (Independent Component Analysis) ★★

**Problem it solves:** the *cocktail party* — several microphones record mixtures of several independent speakers; recover the speakers.

$$
\begin{aligned}
x &= A s & &x: \text{observed mixtures } (D) \\
& & &s: \text{independent sources } (D) \\
& & &A: \text{unknown mixing matrix} \\[6pt]
\text{goal:}\quad & \text{find } W \text{ such that } y = W x \approx s
\end{aligned}
$$

**Assumptions:** sources are **statistically independent**; at most one is Gaussian; the mixing is **linear** and instantaneous; there are at least as many mixtures as sources.

**Why non-Gaussianity is essential:** by the **central limit theorem**, a sum of independent variables is *more Gaussian* than any of its parts. So mixtures look Gaussian. To recover the sources you search for projections $y = w^{\top}x$ that are **maximally non-Gaussian** — those are the independent directions. (PCA fails here because it only uses second-order statistics, and uncorrelated ≠ independent for non-Gaussian data.)

**Measures of non-Gaussianity:**

$$
\begin{aligned}
\text{kurtosis}\quad & E[y^4] - 3E[y^2]^2 &&\text{(0 for Gaussian; can be + or -)} \\
\text{negentropy}\quad & J(y) = H(y_{\text{gauss}}) - H(y) &&(\ge 0,\ 0 \text{ iff Gaussian}) \\
\text{contrast functions}\quad & \text{(log cosh, etc.)}
\end{aligned}
$$

**Objective:**

$$
\max_W \sum_j J(w_j^{\top} x) \qquad \text{subject to decorrelation } (W W^{\top} = I)
$$

**Ambiguities — ICA cannot recover:** (1) the **order** of the sources (a permutation $P$: $x = (AP^{-1})(Ps)$), (2) their **scale**, (3) their **sign** ($x = (AD^{-1})(Ds)$). All three are unidentifiable because the model is unchanged by them.

**FastICA** uses a fixed-point iteration on negentropy; **Infomax** maximises output entropy; **JADE** uses fourth-order cumulants.

### 20.6 The comparison table ★★★ (learn this shape)

| | PCA | PPCA | Factor Analysis | ICA |
|---|---|---|---|---|
| Type | deterministic projection | probabilistic generative | probabilistic generative | blind source separation |
| Model | $Z = XW$ | $x = \mu + Wz + \epsilon$, $\epsilon \sim \mathcal{N}(0,\sigma^2 I)$ | $x = \mu + \Lambda z + \epsilon$, $\epsilon \sim \mathcal{N}(0,\Psi)$ | $x = As$ |
| Latent prior | none | Gaussian | Gaussian | **non-Gaussian** |
| Noise | none | isotropic | diagonal, per-feature | none (noise = extra sources) |
| Goal | max variance | max likelihood, noise-aware DR | explain correlations via factors | recover independent sources |
| Identifiable up to | rotation (ordered by variance) | rotation, ordered by variance | **rotation** (ambiguous) | **permutation + scale + sign** |
| Handles missing data | no | yes (EM) | yes (EM) | no |
| Typical use | compression, visualisation | DR, likelihood, sampling | psychometrics, finance | audio separation, EEG, artefact removal |

**One-line selection rule:**
- Only compression, no probability needed → **PCA**.
- Want a likelihood/noise model/sampling → **PPCA** (equal noise) or **FA** (per-feature noise).
- Want to *separate sources* rather than compress → **ICA**.

`Remember:` **PCA = rotate to max variance; PPCA = PCA + Gaussian noise; FA = PPCA with per-feature noise; ICA = unmix independent non-Gaussian sources.** The whole family is "x = matrix × latent + noise", differing only in the noise and the latent prior.

`Exam:` "Differentiate PCA, PPCA, FA and ICA." Draw the four generative equations in a column — that single column *is* the answer; then add the identifiability row.

---

# PART V — Autoencoders

## 21. The vanilla autoencoder ★★★

### 21.1 What it is

An autoencoder (AE) is a network trained to **reconstruct its own input** through a narrow bottleneck:

```
x → Encoder E_φ → z (latent code) → Decoder D_θ → x̂
```

$$
\begin{aligned}
z &= f(W_e x + b_e) \\
\hat{x} &= g(W_d z + b_d)
\end{aligned}
$$

- **Encoder** compresses $x \in \mathbb{R}^n$ to $z \in \mathbb{R}^d$, $d < n$.
- **Bottleneck / latent space**: the layer whose neuron count equals the latent dimension.
- **Decoder** reconstructs.
- **Trained unsupervised** — no labels are needed; the target *is* the input. (This is why AEs are called unsupervised even though the training signal is a reconstruction error.)

**Training objective:**

$$
\min_{\theta,\phi}\; \frac{1}{N}\sum_i \lVert x_i - D(E(x_i))\rVert^2
$$

**Loss choices:** MSE for real-valued inputs; **BCE** when inputs are binary or scaled to [0,1] (per-pixel Bernoulli):

$$
L_{\text{BCE}} = -\sum_i \big[ x_i \log \hat{x}_i + (1 - x_i)\log(1 - \hat{x}_i) \big]
$$

**All reconstruction losses** (pick by what the output *is* — the loss is the assumed data distribution):

| Input / output type | Loss | Formula | Decoder output unit |
|---|---|---|---|
| real-valued, Gaussian | MSE / squared error | $\lVert x - \hat{x}\rVert^2$ | linear |
| real-valued, robust to outliers | MAE / L1 | $\lVert x - \hat{x}\rVert_1$ | linear |
| binary or [0,1] | BCE | $-\sum[x\log\hat{x} + (1-x)\log(1-\hat{x})]$ | sigmoid |
| one-of-K / counts | categorical CE | $-\sum_k x_k \log \hat{x}_k$ | softmax |
| images (sharpness) | perceptual / SSIM | feature-space or structural distance | linear |

Every *regularised* AE below keeps the same reconstruction term and adds a penalty — that is the only difference.

**What the latent space gives you:** a compact, denoised, semantically meaningful representation — better for visualisation and downstream classifiers, and it captures the "essential factors" rather than raw pixels.

### 21.2 Undercomplete vs overcomplete ★★★

| | Undercomplete | Overcomplete |
|---|---|---|
| Latent size | $d \ll n$ | $d \geq n$ |
| Effect | forces compression, learns structure | can copy the input |
| Overfitting risk | **lower** | **higher** |
| Fix | none needed | regularise (sparse/denoising/contractive) |

**The identity-mapping trap** (the notes' ★ "main risk"): if the latent layer is large and there is no regularisation, the network learns $D(E(x)) = x$ with $E$ = identity. It reconstructs perfectly and has learned **nothing** useful. Any regulariser that limits capacity or adds noise prevents this.

### 21.3 Regularised AE = reconstruction + penalty

$$
\text{Loss} = L_{\text{reconstruction}} + \lambda \cdot R(\text{encoder})
$$

Different $R$ gives a different AE family. This single sentence is the answer to "what is a regularised autoencoder?".

### 21.4 Linear AE ≈ PCA ★★

A single-layer AE with **linear** activations and MSE loss learns a subspace equal to the top-$d$ PCA subspace. Its weights span the same space (up to rotation), even though $W_d \neq W_e^\top$ in general. With non-linear activations, the AE learns a **non-linear manifold** — strictly more powerful than PCA.

### 21.5 Convolutional AE (CAE)

Replace dense layers with conv/pool in the encoder and **transposed convolution** (or upsampling + conv) in the decoder. Weight sharing preserves spatial structure, so it is the right choice for images: fewer parameters, translation-equivariant features, and it scales to large images. Used for image denoising, restoration, and as the backbone of segmentation models (U-Net is a conv AE with skip connections).

> Note: your notes use "CAE" for both **Contractive** AE and **Convolutional** AE. In the exam, spell out the full name once — "contractive autoencoder (CAE)" — and there is no ambiguity.

---

## 22. Sparse autoencoder ★★

**Problem:** an overcomplete AE copies the input. **Fix:** force the code to be **sparse** — only a few hidden units may be active for any input.

Two ways to enforce it:

**(a) L1 penalty on activations** (the notes' version):

$$
\text{Loss} = L_{\text{reconstruction}} + \lambda \sum_j \lvert h_j(x)\rvert
$$

L1 drives small activations to exactly 0 (same geometry argument as L1 weight decay: the diamond's corners).

**(b) KL-divergence sparsity** (the standard formulation): let $\hat{\rho}_j = \frac{1}{N}\sum_i h_j(x_i)$ be the average activation of unit $j$, and $\rho$ the desired sparsity (e.g. 0.05). Then

$$
\begin{aligned}
\text{KL}(\rho \,\Vert\, \hat{\rho}_j) &= \rho \log\frac{\rho}{\hat{\rho}_j} + (1-\rho)\log\frac{1-\rho}{1-\hat{\rho}_j} \\
\text{Loss} &= L_{\text{reconstruction}} + \beta \sum_j \text{KL}(\rho \,\Vert\, \hat{\rho}_j)
\end{aligned}
$$

The KL term is 0 when $\hat{\rho}_j = \rho$ and grows without bound as $\hat{\rho}_j \to 0$ or 1 — it pins every unit's average firing rate to $\rho$.

**(c) Top-k / k-sparse:** keep only the $k$ largest activations and zero the rest. Example from your notes: hidden activations $[0.2, 0.8, 0.9, 0.7]$ with $k = 3 < d = 4$ → the 0.2 is suppressed, the rest survive. No hyperparameter $\lambda$ to tune, but non-differentiable (use straight-through or just threshold).

**Effect:** the code becomes disentangled and interpretable; each unit responds to a distinct pattern. Applications: feature selection, denoising, dimension reduction.

---

## 23. Denoising autoencoder (DAE) ★★

**Idea:** corrupt the input on purpose, train the network to recover the **clean** original.

$$
\begin{aligned}
\tilde{x} &= \operatorname{corrupt}(x) && (\text{noise added at input}) \\
z &= E(\tilde{x}) \\
\hat{x} &= D(z) && \text{target} = x \text{ (clean!)} \\
\text{Loss} &= \lVert x - \hat{x}\rVert^2
\end{aligned}
$$

**Why it works:** the network cannot copy the input (the input is damaged), so it must learn the **structure of the data distribution** — the correlations that let it fill in the missing part. It is a regulariser and a manifold learner at once: it learns to project noisy points back onto the data manifold.

**Noise types** (from your notes — know which noise suits which data):

| Noise | Formula | Use |
|---|---|---|
| **Gaussian** | $\tilde{x} = x + \varepsilon$, $\varepsilon \sim \mathcal{N}(0, \sigma^2)$ | real-valued inputs |
| **Masking** | randomly set some components to 0 | binary inputs, dropout-style |
| **Salt & pepper** | random pixels set to min or max | images |
| **Poisson** | $\tilde{x} \sim \mathrm{Poisson}(x)$ | count data, low-light images |

Examples: Gaussian $x = 0.3, \varepsilon = 0.1 \to \tilde{x} = 0.4$; masking $[1,0,1,0] \to [0,0,1,0]$; salt & pepper $[230,100,110,210] \to [250,100,100,250]$.

**Applications:** image/signal denoising, data cleaning, preprocessing, and **pretraining** (stacked DAEs were the original deep-network initialiser).

---

## 24. Contractive autoencoder (CAE) ★★

**Idea:** make the representation **insensitive to small changes in the input**. Instead of corrupting the input, penalise the *derivative* of the code with respect to the input.

$$
\begin{aligned}
\text{Loss} &= \lVert x - \hat{x}\rVert^2 + \lambda \lVert J_f(x)\rVert_F^2 \\
\lVert J_f(x)\rVert_F^2 &= \sum_{i,j}\left(\frac{\partial h_j}{\partial x_i}\right)^2
\end{aligned}
$$

The penalty is the squared **Frobenius norm of the Jacobian** of the encoder. It encourages the encoder to be flat in directions that are noise, and steep only along the data manifold. This is *exactly* the "distance from the manifold = noise" picture in your notes.

**Effect:** robust features, a smoother latent space, less sensitivity to perturbations; related to denoising (a first-order approximation of it) but deterministic and cheaper to evaluate.

**Applications:** feature learning, transfer learning, generation, and as a theoretical bridge to the VAE.

---

## 25. Variational autoencoder (VAE) ★★★

### 25.1 What changes

A vanilla AE maps each input to **one point** $z$. A VAE maps it to a **distribution**: the encoder outputs the mean $\mu$ and standard deviation $\sigma$ of a Gaussian, and $z$ is sampled from it.

$$
\begin{aligned}
\text{Encoder:}\quad & q_\phi(z \mid x) = \mathcal{N}\big(\mu_\phi(x), \operatorname{diag}(\sigma_\phi(x)^2)\big) \\
\text{Prior:}\quad & p(z) = \mathcal{N}(0, I) \\
\text{Decoder:}\quad & p_\theta(x \mid z) \qquad (\text{a distribution over } x \text{ given } z) \\
\text{Sample:}\quad & z \sim q_\phi(z \mid x) \;\to\; \hat{x} = D_\theta(z)
\end{aligned}
$$

**Why the randomness is a problem:** you cannot backpropagate through "sample from $\mathcal{N}(\mu,\sigma^2)$" — the sampling operation is not differentiable w.r.t. $\mu, \sigma$.

### 25.2 The reparameterisation trick ★★★

Move the randomness to an external noise variable:

$$
z = \mu + \sigma \odot \varepsilon, \qquad \varepsilon \sim \mathcal{N}(0, I)
$$

Now $z$ is a deterministic, differentiable function of $\mu$ and $\sigma$; the randomness lives in $\varepsilon$, which has no parameters. Gradients flow through $\mu$ and $\sigma$ normally.

**In practice the encoder outputs $\log \sigma^2$**, not $\sigma$:
- $\sigma$ must be positive; a linear layer outputs any real number.
- $\log \sigma^2$ is unconstrained, and $\sigma = \exp(\tfrac{1}{2}\log \sigma^2)$.
- It is numerically more stable (no division, no clipping).

### 25.3 The loss (ELBO) ★★★

$$
L_{\text{VAE}} = \underbrace{-\mathbb{E}_{q(z \mid x)}\big[\log p_\theta(x \mid z)\big]}_{\text{reconstruction}} + \underbrace{D_{\text{KL}}\big(q_\phi(z \mid x) \,\Vert\, p(z)\big)}_{\text{regularisation}}
$$

- **Reconstruction term:** expected negative log-likelihood → for Gaussian decoder it is MSE; for Bernoulli it is BCE.
- **KL term:** pulls the encoder's distribution toward the prior $\mathcal{N}(0,I)$, so the latent space is smooth, continuous and samplable.

**Closed-form KL for two Gaussians** (memorise; it is the most examined formula in this section):

$$
D_{\text{KL}}\big(\mathcal{N}(\mu, \sigma^2) \,\Vert\, \mathcal{N}(0,1)\big) = \frac{1}{2}\sum_i\big( \mu_i^2 + \sigma_i^2 - 1 - \log \sigma_i^2 \big)
$$

Using $\log \sigma^2$ directly:

$$
L_{\text{KL}} = \frac{1}{2}\sum_i\big( \mu_i^2 + \exp(\log \sigma_i^2) - 1 - \log \sigma_i^2 \big)
$$

**Why this is the ELBO:** the true marginal likelihood is $\log p(x) = \text{ELBO} + D_{\text{KL}}(q \,\Vert\, p(z \mid x))$, and KL $\geq 0$, so maximising the ELBO maximises a lower bound on $\log p(x)$. The ELBO is tight when $q$ equals the true posterior.

**Intuition for the two terms fighting:** reconstruction wants each input in its own distinct region; KL wants everything packed around 0. The balance gives a smooth, interpolatable latent space — which is why **VAE latent spaces can be interpolated** (morph one face into another) and GAN spaces often cannot.

**Generation:** after training, throw away the encoder, sample $z \sim \mathcal{N}(0,I)$, and decode.

### 25.4 VAE vs AE ★★

| | AE | VAE |
|---|---|---|
| Latent | one point per input | a distribution per input |
| Latent space | unstructured, holes | smooth, continuous, samplable |
| Generative | no | yes |
| Loss | reconstruction only | reconstruction + KL |
| Use | compression, denoising, features | generation, interpolation, anomaly detection |

`Remember:` **VAE = "encoder predicts a cloud, not a point; sample inside the cloud."** Reparameterisation = "move the dice outside the network". KL = $\frac{1}{2}\sum(\mu^2 + \sigma^2 - 1 - \log \sigma^2)$ — check it: for $\mu=0, \sigma=1$ every term is $0+1-1-0 = 0$, which is exactly right (the prior matches itself).

---

## 26. Stochastic autoencoders and the AE family tree

**Stochastic AE (SAE)** = any AE whose encoder, decoder, or latent representation is probabilistic, so the same input can give different reconstructions. Three types:

| Type | Idea |
|---|---|
| **Denoising AE** | noise on the *input* |
| **Variational AE** | distribution in the *latent space* |
| **Dropout AE** | randomly switch off neurons during training (prevents co-adaptation) |

**The full family summary** (your notes' starred table — a ready-made exam answer):

| Variant | Key mechanism | Typical applications |
|---|---|---|
| **Vanilla AE** | encoder → bottleneck → decoder, trained by backprop | dimensionality reduction, outlier detection, feature learning, compression |
| **Sparse AE** | L1 penalty or KL sparsity on activations | feature selection, dimension and noise reduction |
| **Denoising AE** | corrupt input, reconstruct clean | image/signal denoising, data cleaning, preprocessing |
| **Contractive AE** | Jacobian-norm penalty | robust features, transfer learning |
| **Convolutional AE** | conv/pool encoder, transposed-conv decoder | image reconstruction, denoising, segmentation (U-Net) |
| **Stacked AE** | several AEs stacked, layer-wise pretrained | deep feature learning, pretraining |
| **Variational AE** | probabilistic latent + KL + reparameterisation | generation, interpolation, anomaly detection, DR |

### 26.1 Representational power, capacity, depth ★★

**Representational power** = how well the AE can capture the important patterns in the data.

$$
\begin{aligned}
\text{capacity too low} &\to \text{underfitting (cannot reconstruct)} \\
\text{capacity just right} &\to \text{good compression + useful features} \\
\text{capacity too high} &\to \text{identity mapping (memorises, learns nothing)}
\end{aligned}
$$

**Five ways to control representational power** (your notes' list):
1. smaller latent layer (undercomplete, $d < n$);
2. sparsity constraint (sparse AE);
3. add noise (denoising AE);
4. weight decay (L2);
5. contractive regularisation.

**Effect of latent dimension $d$:**

$$
\begin{aligned}
d \text{ too small} &\to \text{information loss} \to \text{poor reconstruction} \\
d \text{ too large} &\to \text{memorisation} \to \text{weak feature learning} \\
d \text{ just right} &\to \text{enough compression to preserve the essentials}
\end{aligned}
$$

**Depth:** shallow AE (fewer hidden layers) for simple patterns; **deep AE** for complex, non-linear data — it learns hierarchical features (`pixels → edges → shapes → objects → high-level features`) and gives a richer latent representation. Costs: more computation, higher overfitting risk, harder training, and **vanishing/exploding gradients**.

### 26.2 Applications and anomaly detection ★★★

**Applications:** dimensionality reduction, feature learning, pretraining, denoising, image restoration, compression, anomaly detection, generation (VAE), recommendation.

**Anomaly detection rule** — the AE is trained on *normal* data only:

$$
\begin{aligned}
e(x) &= \lVert x - \hat{x}\rVert^2 \\
\text{anomaly if}\;& e(x) > \tau \qquad (\tau \text{ from validation quantiles or ROC})
\end{aligned}
$$

Why it works: the AE has only learned to reconstruct the normal manifold, so an anomalous input (fraud, a defective part, a diseased scan) falls off the manifold and produces a large reconstruction error.

**Choosing τ:** maximise F1 on a labelled validation set, or set a percentile of the training reconstruction error, or pick the point on the ROC closest to the top-left.

`Remember:` **"An autoencoder is a compressor that learned what normal looks like; anything it cannot compress is an anomaly."**

`Exam:` "Explain any four types of autoencoder with their loss functions." Vanilla (MSE), sparse (MSE + $\lambda\sum\lvert h\rvert$), denoising (noise then MSE), contractive (MSE + $\lambda\lVert J\rVert^2$), VAE (reconstruction + KL). One line each + one use case each.

---

# PART VI — Pre-training, transfer learning, domain adaptation

## 27. Greedy layer-wise unsupervised pre-training ★★

### 27.1 The problem it solved

Training a deep network directly in 2006–2012 failed because of: vanishing/exploding gradients, poor local minima and saddle regions, too little labelled data, slow convergence, weak initialisation, and overfitting in high-capacity models. The trick that made deep networks trainable *before* ReLU and batch norm was to **initialise each layer with unsupervised learning**.

### 27.2 The procedure (draw this pipeline)

1. **TRAIN L1** — train the first AE: $x \to h_1 \to \hat{x}_1$, minimise $\lVert x - \hat{x}_1\rVert^2$.
2. **FREEZE L1** — keep only the encoder; discard the decoder.
3. **TRAIN L2** — feed $h_1$ into a second AE: $h_1 \to h_2 \to \hat{h}_1$.
4. **REPEAT** — for as many layers as you want.
5. **COMBINE** — stack all encoders, add a supervised output layer.
6. **FINE-TUNE** — train the whole network with backprop on labels.

**"Greedy"** = each layer is optimised for its own local objective, one at a time, without looking ahead to the final task. **"Layer-wise"** = progress layer by layer, freezing what is already trained. **"Unsupervised"** = each stage uses only inputs, no labels. **"Pre-training"** = this phase happens before supervised training.

**Layer-wise objective:**

$$
\min_{\theta_l,\phi_l}\; \sum_i \left\lVert h_{l-1}^{(i)} - D_l\left(E_l\left(h_{l-1}^{(i)}\right)\right) \right\rVert^2
$$

**Fine-tuning objective:**

$$
\min_\theta\; \sum_i L\big(f(x_i;\theta), y_i\big) \qquad \text{starting from the pre-trained } \theta_0 \text{ (not random)}
$$

**After pre-training, retain the encoders only; the decoders were scaffolds** (like training wheels) and are thrown away.

### 27.3 Why it helps ★★

| Optimisation benefit | Representation benefit |
|---|---|
| starts in a good basin of the loss | learns generic features from unlabelled data |
| better-conditioned gradients | acts as a regulariser (restricts the hypothesis space) |
| faster convergence | useful when labels are scarce |
| avoids poor local minima early | provides a sensible initialisation for deep stacks |

**Limitations:** it is a multi-stage, expensive pipeline; each layer is optimised for the wrong (local) objective; the decoders are wasted; and once ReLU + good init + batch norm + dropout arrived, training from scratch became easier and pre-training fell out of fashion.

**Modern descendant — self-supervised learning:** same idea, better objective. Masked prediction (BERT, MAE), contrastive learning (SimCLR, MoCo), image–text alignment (CLIP), next-token prediction (GPT). All learn representations from unlabelled data, then fine-tune. The unifying idea is unchanged: *learn a good representation first, adapt it to the task later.*

### 27.4 Stacked autoencoder specifics ★

For each layer: train an AE on the previous layer's activations, keep the encoder, discard the decoder, freeze. After stacking $L$ encoders, add a classifier and fine-tune everything with backprop (optionally with a smaller LR for early layers).

`Remember:` **"Pretrain layer by layer with autoencoders, then fine-tune the whole stack with labels."** Six boxes: train → freeze → feed → repeat → stack → fine-tune.

---

## 28. Transfer learning ★★★

### 28.1 Definition and intuition

**Transfer learning** reuses knowledge learned on a **source task/dataset** for a different but related **target task**. Formally, the source domain/task $(D_S, T_S)$ differs from the target $(D_T, T_T)$, and you exploit $D_S$ to improve performance on $D_T$.

**Why it works for images:** early CNN layers learn edges, colours and textures — features useful for *every* vision task. Only the last layers are task-specific (faces, dog breeds, tumours). So you keep the generic part and retrain the specific part.

**Why we need it:** training from scratch needs a huge labelled dataset, high compute and long training. Transfer learning works when the target dataset is small, and it gives better initialisation, faster convergence, less overfitting and often higher accuracy.

### 28.2 The two strategies ★★★

**Strategy 1 — Feature extraction (frozen backbone):**

```
pretrained CNN  →  [frozen conv layers]  →  [new classifier head]  →  target labels
```

- Freeze all pre-trained layers (`trainable = False`).
- Replace the final layer(s) with your own (`1000 → num_classes`).
- Train **only the new head**. Fast, needs little data, cannot destroy the pre-trained features.

**Strategy 2 — Fine-tuning:**

```
pretrained CNN  →  [early layers frozen]  →  [later layers unfrozen]  →  new head
```

- Start from the pre-trained weights (not random).
- Unfreeze the top blocks and train them with a **small learning rate** (`1e−4` to `1e−5`, ~10–100× smaller than normal).
- Update rule is the ordinary one, just starting from a good $\theta_0$:

$$
W_{\text{new}} = W_{\text{old}} - \eta \frac{\partial L}{\partial W}
$$

**Why a smaller LR for the backbone:** the pre-trained weights are already good; a large LR would destroy them ("catastrophic forgetting") in the first few batches.

**How many layers to freeze?** Rule of thumb:

| Target dataset | Similar to source | Different from source |
|---|---|---|
| **Small** | freeze all, train the head | freeze early layers, fine-tune later ones |
| **Large** | fine-tune everything (small LR) | fine-tune everything / train from scratch |

### 28.3 Negative transfer ★★

**Negative transfer** = transfer *hurts*: the target performance is worse than training from scratch. Causes:

- source and target domains are too different (satellite images → speech);
- the transferred features are misleading;
- the label structure differs strongly (source has 1000 fine-grained classes, target is binary);
- fine-tuning overfits or destroys the useful features.

**Diagnosis:** compare against a from-scratch baseline on the target validation set. **Fixes:** unfreeze more/fewer layers, lower the LR, use a closer source model, or abandon transfer.

### 28.4 Transfer learning vs training from scratch ★★

| | From scratch | Transfer learning |
|---|---|---|
| Data needed | very large | small–medium |
| Time / compute | long | short |
| Initialisation | random | pre-trained |
| Overfitting risk | high on small data | lower |
| Best when | huge dataset, unusual domain | small dataset, natural images/text |

`Remember:` **"Freeze features, fine-tune specifics."** If the target is small and similar → freeze everything but the head; if it is large or different → unfreeze more, with a smaller LR.

`Exam:` "What is transfer learning? Explain feature extraction and fine-tuning with diagrams." Two layer diagrams (frozen vs partly unfrozen), one paragraph of advantages, one line on negative transfer.

---

## 29. Domain adaptation ★★

### 29.1 Definitions

A **domain** is a pair $D = \{X, P(X)\}$: the feature space and the data distribution.

**Domain adaptation (DA)** adapts a model trained on a **source domain** to a different but related **target domain**, where

$$
P_s(x) \neq P_t(x) \qquad \text{but} \qquad T_s = T_t \qquad (\text{the task/label function is the same})
$$

It is a **subset of transfer learning** ($\text{DA} \subseteq \text{TL}$): the task stays the same, only the input distribution shifts.

**Examples:**

| Source | Target |
|---|---|
| sunny-day pedestrian images | rainy-day images |
| studio product photos | mobile-phone photos |
| synthetic/simulated data | real sensor data |
| one hospital's scans | another hospital's scanner |
| labelled handwriting (MNIST) | unlabelled digits (USPS) |

**The canonical example from your notes:** cat-vs-dog classifier trained on studio images, deployed on mobile images. You want the network to learn **ears, fur, face, body shape** (class-relevant) and *not* **background, lighting, weather** (domain-specific).

### 29.2 Types of distribution shift ★★

| Shift | What changes | Example |
|---|---|---|
| **Covariate shift** | $P(x)$ changes, $P(y \mid x)$ stays | lighting, sensor, camera |
| **Label shift** | $P(y)$ changes, $P(x \mid y)$ stays | disease prevalence changes |
| **Concept shift** | $P(y \mid x)$ changes | spam tactics evolve; the same words mean something else |

**DA usually assumes covariate shift** — that is why aligning $P(F_s)$ with $P(F_t)$ is the standard approach.

### 29.3 Types of DA ★★

| Type | Target labels available |
|---|---|
| **Supervised DA** | all target labels (rare, easy) |
| **Semi-supervised DA** | a few labelled + many unlabelled target samples |
| **Unsupervised DA** | no target labels (the hard, common case) |

### 29.4 Techniques ★★★

1. **Fine-tuning** on target data (with a small LR; the simplest baseline).
2. **Feature alignment** — learn an extractor $F = f(X)$ such that $P(F_s) \approx P(F_t)$. Loss: MMD, CORAL, or a discrepancy term.
3. **Adversarial DA (DANN)** — add a **domain discriminator** $D$ that tries to predict "source or target" from the features; the feature extractor $G$ is trained to *fool* it. The result is a representation that is informative for the label but uninformative about the domain.
4. **Data transformation / style transfer** — convert source images to look like target images (CycleGAN: sunny → rainy).
5. **Self-training / pseudo-labelling** — predict on target data, keep confident predictions as labels, retrain.

**Adversarial DA objective:**

$$
\min_G \max_D\; \underbrace{L_{\text{task}}(G, C)}_{\text{classifier}} - \lambda \underbrace{\Big[ \mathbb{E}_{x \sim s}\log D(G(x)) + \mathbb{E}_{x \sim t}\log\big(1 - D(G(x))\big) \Big]}_{\text{domain confusion}}
$$

or the notes' compact form:

$$
L = L_{\text{task}} + \lambda L_{\text{domain}}
$$

$\lambda$ controls how hard the features are pushed to be domain-invariant. Too large → the features become domain-invariant but **useless for classification** (the classic failure mode); too small → no adaptation.

**Risks:** negative transfer; collapsing features to a constant (the discriminator cannot tell domains because there is no information at all); target labels leaking; the alignment destroying class-discriminative structure.

### 29.5 Transfer learning vs domain adaptation ★★

| | Transfer learning | Domain adaptation |
|---|---|---|
| Task | may also change | **same task** |
| What differs | task and/or domain | only the data distribution |
| Labels | source labelled, target labelled | target often unlabelled |
| Goal | reuse knowledge | become robust to distribution shift |
| Relation | superset | subset |

`Remember:` **TL = "different task, reuse the model"; DA = "same task, different data distribution."**

`Exam:` "Define domain adaptation. How does it differ from transfer learning?" Give $D = \{X, P(X)\}$, the inequality $P_s(x) \neq P_t(x)$, and the subset statement. Then the adversarial block diagram.

---

# PART VII — Deep generative models

## 30. Generative vs discriminative modelling ★★★

**Discriminative model** (DM) learns the conditional $p(y\mid x)$ — the boundary between classes:

```
x → p_θ(y|x) → label
```

**Generative model** (GM) learns the data distribution $p(x)$ (or $p(x\mid y)$), so it can *create* new samples:

```
z or y → p_θ(x|z) → new x
```

**A generative model should:** produce realistic samples, cover the diversity of the data (not one mode), and learn a meaningful latent structure (so you can interpolate and control).

**Why not just store pixels / sample randomly?** A $28\times 28$ grayscale image has $256^{784}$ possible values. Random pixels almost always give noise; real images occupy a tiny **structured subset** of that space. The model's job is to learn that subset. (The notes' counting example: 100 training images with class frequencies 0.5/0.3/0.2 — sampling a *label* is easy; sampling a *realistic image* is the hard part.)

### 30.1 Taxonomy ★★

```
Generative models
├── Explicit density  p_θ(x) written down
│   ├── Tractable            → autoregressive, normalising flows
│   └── Approximate
│       ├── Variational      → VAE
│       └── MCMC             → Boltzmann machines / RBMs
└── Implicit density  (no p_θ(x), only a sampling procedure)
    └── GAN
```

| Model | Likelihood | Sampling | Strength | Weakness |
|---|---|---|---|---|
| **VAE** | approximate (ELBO) | fast, one pass | smooth latent space, principled | blurry samples |
| **GAN** | implicit | fast | sharp, realistic samples | unstable, mode collapse, no likelihood |
| **Autoregressive** | exact | slow (sequential) | best likelihood, stable training | slow generation |
| **Diffusion** | approximate/ELBO | slow (many steps) | state-of-the-art quality | expensive inference |
| **Normalising flow** | exact | fast | exact likelihood + invertible | restrictive architecture |

**Latent-variable generative model:**

$$
\begin{aligned}
z &\sim p(z) && \text{(simple prior, usually } \mathcal{N}(0,I)\text{)} \\
x &\sim p_\theta(x\mid z) && \text{(the decoder/generator)} \\
p_\theta(x) &= \int p_\theta(x\mid z)\,p(z)\,dz && \text{(intractable in general } \to \text{ hence VAE/GAN/diffusion)}
\end{aligned}
$$

$z$ captures hidden factors: shape, pose, topic, style, identity.

---

## 31. VAE as a generative model (recap and generation)

Covered in §25. For the generative-model exam question, state:

1. It is an **explicit, approximate** density model (`ELBO` lower bound).
2. Encoder $q_\phi(z\mid x)$ ≈ posterior; decoder $p_\theta(x\mid z)$ = generator.
3. Train with $L = \text{reconstruction} + \text{KL}$.
4. Generate: sample $z\sim\mathcal{N}(0,I)$, decode.
5. **Latent space is smooth and interpolatable** — the main practical advantage over a GAN.
6. **Samples are blurrier** than a GAN's, because the Gaussian/Bernoulli likelihood averages over uncertainty.

---

## 32. GANs ★★★

### 32.1 Architecture

Two networks in opposition:

```
z ~ p_z(z)  →  Generator G  →  fake sample G(z)
                                    ↓
real sample x ~ p_data  ──────→  Discriminator D  →  score ∈ (0,1) real/fake
```

- **Generator**: maps noise to data space; wants $D(G(z)) \to 1$.
- **Discriminator**: classifies real vs fake; wants $D(x) \to 1$, $D(G(z)) \to 0$.

**Both are trained simultaneously** — the generator improves by learning to fool the discriminator, and the discriminator improves at spotting fakes. At the equilibrium, $G$ produces samples indistinguishable from real data and $D$ outputs 0.5 everywhere.

### 32.2 The minimax objective ★★★

$$
\min_G \max_D V(D,G) = \mathbb{E}_{x\sim p_{\text{data}}}\left[\log D(x)\right] + \mathbb{E}_{z\sim p_z}\left[\log\left(1 - D(G(z))\right)\right]
$$

- The **discriminator maximises** $\log D(x) + \log(1 - D(G(z)))$.
- The **generator minimises** $\log(1 - D(G(z)))$.

**Optimal discriminator** (for a fixed $G$):

$$
D^*(x) = \frac{p_{\text{data}}(x)}{p_{\text{data}}(x) + p_g(x)}
$$

Substituting this back shows the generator minimises the **Jensen–Shannon divergence** between $p_{\text{data}}$ and $p_g$. That is the theoretical justification of the GAN: it is minimising a divergence between distributions, which is why it can match the *shape* of the data distribution rather than just reconstruct points.

### 32.3 The non-saturating loss (what is actually used) ★★

At the start, $D$ easily rejects fakes, so $D(G(z)) \approx 0$ and $\log(1 - D(G(z)))$ has **near-zero gradient** — the generator stops learning. Fix: flip the generator's objective:

$$
\begin{aligned}
L_G &= -\mathbb{E}_{z\sim p_z}\left[\log D(G(z))\right] && \text{non-saturating; strong gradient early} \\
L_D &= -\mathbb{E}_{x\sim p_{\text{data}}}\left[\log D(x)\right] - \mathbb{E}_{z}\left[\log\left(1 - D(G(z))\right)\right]
\end{aligned}
$$

This has the same fixed point but much better gradients when $D$ is confident.

### 32.4 Training algorithm ★★

For each iteration:

1. sample a minibatch of real $x$
2. sample a minibatch of noise $z$
3. generate fakes $G(z)$
4. update $D$: maximise $\log D(x) + \log(1 - D(G(z)))$ (freeze $G$)
5. sample fresh noise $z$
6. update $G$: maximise $\log D(G(z))$ (freeze $D$)

Step 5 ("sample fresh noise") matters: reusing the same $z$ for the $D$ and $G$ steps biases the update.

**The two losses in the form written in your notes** (minibatch of size $m$; $D(\cdot)\in[0,1]$):

$$
\begin{aligned}
\text{Generator loss: } J_G &= -\frac{1}{m}\sum_{i=1}^{m}\log D(G(z_i)) \\
\text{Discriminator loss: } J_D &= -\frac{1}{m}\sum_{i=1}^{m}\log D(x_i) - \frac{1}{m}\sum_{i=1}^{m}\log\left(1 - D(G(z_i))\right) \\
\text{Min-max objective: } \min_G \max_D V(G,D) &= \mathbb{E}_{x\sim p_{\text{data}}}\left[\log D(x)\right] + \mathbb{E}_{z\sim p(z)}\left[\log(1 - D(G(z)))\right]
\end{aligned}
$$

**How to read them:** $J_D$ is the binary cross-entropy of "real = 1, fake = 0"; minimising it makes $D(x) \to 1$ and $D(G(z)) \to 0$. $J_G$ is the cross-entropy of "fake = 1"; minimising it makes $D(G(z)) \to 1$. So **both are ordinary BCE losses** — the adversarial part is only that they are optimised in alternation on the same $D$.

**How a GAN works — the 6-step answer your notes flag as "Imp":**

1. Generator's first move — $G(z)$ produces a fake from noise
2. Discriminator's turn — $D$ scores real and fake samples
3. Adversarial learning — each updates against the other's current version
4. Generator's improvement — $G$ adjusts to raise $D(G(z))$
5. Discriminator's adoption — $D$ adjusts to lower $D(G(z))$ and raise $D(x)$
6. Training progression — repeat until $D$ outputs $\approx 0.5$ (cannot distinguish)

> Your notes also record the teacher's remark: **"No numericals on GAN"** — expect architecture + losses + how it works, not arithmetic.

### 32.5 Conditional GAN (cGAN) ★★

Feed the condition $c$ (class label, text, edge map, segmentation map) to **both** networks:

$$
\begin{aligned}
& G(z,c) \to \text{fake sample of class } c \\
& D(x,c) \to \text{real / fake } \textbf{given}\text{ class } c \\
& \min_G \max_D \; \mathbb{E}\left[\log D(x,c)\right] + \mathbb{E}\left[\log(1 - D(G(z,c),c))\right]
\end{aligned}
$$

Without $c$ in $D$, the generator can produce any class and still fool the discriminator; conditioning $D$ forces class-consistency. **Uses:** class-conditional generation, text-to-image, super-resolution, edge→photo, segmentation→scene, sketch→image, style transfer.

### 32.6 DCGAN design rules ★★

| Rule | Why |
|---|---|
| Replace pooling with strided conv (D) and transposed conv (G) | learnable down/up-sampling |
| Batch norm in G and D (not on G's output / D's input) | stabilises training |
| No fully connected hidden layers | keeps spatial structure |
| ReLU in G (tanh on output), LeakyReLU in D | standard, avoids dead units |
| Adam with $\text{lr}=2\text{e-}4$, $\beta_1=0.5$ | lower $\beta_1$ damps momentum noise |

### 32.7 Failure modes ★★★

| Problem | What happens | Fix |
|---|---|---|
| **Mode collapse** | $G$ produces the same few samples ($G(z_1)\approx G(z_2)$), covering only some classes | minibatch discrimination, unrolled GAN, WGAN-GP, add diversity loss, more capacity |
| **Training instability** | losses oscillate/diverge; no convergence guarantee | lower LR, spectral norm, gradient penalty, balanced D/G updates |
| **Vanishing gradient** | $D$ too strong $\to$ $D(G(z))\approx 0$ $\to$ $G$ gets no signal | non-saturating loss, weaker D, label smoothing |
| **$D$ too weak** | $D$ cannot distinguish $\to$ $G$ gets useless feedback | stronger D, more D updates |

**A mode-collapsed generator can still produce sharp, realistic-looking images** — it just produces few of them. Quality ≠ diversity; that is why you need both metrics.

### 32.8 Evaluating generative models ★★

Aspects: sample realism (quality), diversity, coverage of the training distribution, semantic consistency, controllability, downstream usefulness.

| Metric | Idea | Limitation |
|---|---|---|
| **Inception Score (IS)** | sharpness × class diversity using a pretrained classifier | ignores the real data distribution; gameable |
| **FID** | Fréchet distance between feature distributions of real and generated images | lower is better; biased by sample size; needs a feature extractor |
| **Precision/Recall** | precision = realism, recall = coverage | needs many samples |
| Human evaluation | realism/diversity judgement | expensive, subjective |

**Quality–diversity trade-off:** pushing a GAN toward sharper samples tends to reduce variety and vice versa. The ideal generator sits on the Pareto front; in practice you tune the balance explicitly.

### 32.9 VAE vs GAN ★★★

| | VAE | GAN |
|---|---|---|
| Objective | ELBO (likelihood lower bound) | minimax / JSD |
| Density | explicit, approximate | implicit |
| Encoder | yes ($q(z\mid x)$) | no (inference needs optimisation) |
| Sample quality | blurry | sharp |
| Latent space | smooth, interpolatable | often unstructured |
| Training | stable | unstable, needs tuning |
| Mode coverage | good | mode collapse risk |
| Best for | representation learning, anomaly detection, interpolation | photorealistic generation |

`Remember:` **GAN = "forger vs detective"**; the forger improves until the detective can only guess (0.5). **Mode collapse = the forger learns one perfect forgery and repeats it.**

`Exam:` "Write the GAN minimax objective and explain each term. What is mode collapse?" Then the two lines: $\min_G \max_D$, $D$ wants real→1/fake→0, $G$ wants fake→1. Mode collapse: $G(z_1) \approx G(z_2)$, low diversity, sharp images; fixes.

---

## 33. Autoregressive, diffusion, and normalising-flow models ★★

These are in your notes as the other three types of deep generative model. One paragraph each is enough.

### 33.1 Autoregressive models

Factorise the joint distribution by the chain rule:

$$
p(x_1,\dots,x_n) = \prod_{i=1}^{n} p(x_i \mid x_1,\dots,x_{i-1})
$$

Generate one token/pixel at a time, each conditioned on all previous ones. Training is **exact likelihood** and stable (it is just next-token prediction with cross-entropy).

**Worked numerical** (from your notes): if $P(I) = 0.5$, $P(\text{love}\mid I) = 0.6$, $P(\text{AI}\mid \text{love}) = 0.8$, then

$$
P(\text{"I love AI"}) = 0.5 \times 0.6 \times 0.8 = 0.24
$$

and generation picks the sequence with the higher total probability (or samples from the distribution).

**Examples:** PixelCNN (images), WaveNet (audio), GPT (text). **Strength:** exact likelihood, stable. **Weakness:** generation is inherently sequential → slow; no global latent space.

### 33.2 Diffusion models

**Forward process:** gradually add Gaussian noise over $T$ steps until the data becomes pure noise.

$$
q(x_t \mid x_{t-1}) = \mathcal{N}\left(\sqrt{1-\beta_t}\,x_{t-1},\; \beta_t I\right)
$$

A useful closed form lets you jump directly to step $t$:

$$
x_t = \sqrt{\bar{\alpha}_t}\,x_0 + \sqrt{1-\bar{\alpha}_t}\,\epsilon, \qquad \bar{\alpha}_t = \prod_{s\leq t}(1-\beta_s), \qquad \epsilon\sim\mathcal{N}(0,I)
$$

**Reverse process:** learn a network $\epsilon_\theta(x_t,t)$ that predicts the noise, and denoise step by step:

$$
p_\theta(x_{t-1}\mid x_t) = \mathcal{N}\left(\mu_\theta(x_t,t),\; \Sigma_\theta(x_t,t)\right)
$$

**Simplified training loss** (DDPM):

$$
L = \mathbb{E}_{x_0,\epsilon,t}\left\Vert \epsilon - \epsilon_\theta\left(\sqrt{\bar{\alpha}_t}\,x_0 + \sqrt{1-\bar{\alpha}_t}\,\epsilon,\; t\right)\right\Vert^2
$$

i.e. "add a known amount of noise, ask the network to predict it". Generation starts from $x_T\sim\mathcal{N}(0,I)$ and applies the reverse steps.

**Strength:** state-of-the-art image/audio quality, stable training (regression loss). **Weakness:** slow sampling (many network calls).

### 33.3 Normalising flows

Use an **invertible** transformation $f$ with a tractable Jacobian, and the change-of-variables formula:

$$
\begin{aligned}
z &= f(x), \qquad x = f^{-1}(z) \\
\log p_X(x) &= \log p_Z(z) + \log\left|\det\left(\frac{\partial f}{\partial x}\right)\right|
\end{aligned}
$$

Stack invertible layers (coupling layers, e.g. RealNVP/Glow) so that the determinant is cheap.

**Strength:** **exact** likelihood and exact inference, fast sampling. **Weakness:** the architecture is constrained (must be invertible; dimensions must match), so expressiveness per parameter is lower.

### 33.4 One-table summary

| Model | How it generates | Likelihood | Speed | Quality |
|---|---|---|---|---|
| Autoregressive | one token at a time | exact | slow | high |
| VAE | sample $z$, one decoder pass | approximate (ELBO) | fast | blurry |
| GAN | noise → generator | implicit | fast | sharp |
| Diffusion | iterate denoising from noise | approximate | very slow | best |
| Flow | invertible map from noise | exact | fast | good |

`Remember:` **AR = "one at a time"; flow = "invertible"; diffusion = "noise → denoise"; VAE = "encode a cloud"; GAN = "forger vs detective".**

`Exam:` "List five types of deep generative models with one advantage and one disadvantage each." This table is the answer.

---

# PART VIII — Transformers

## 34. Why move beyond RNNs ★★

An RNN must process token $t$ before token $t+1$:

| Limitation | Consequence |
|---|---|
| **Sequential computation** | no parallelism over time → slow on long sequences and on GPUs |
| **Long dependency path** | information from token 1 to token 100 passes through 99 recurrent steps → gradients vanish |
| **Fixed-size state** | the whole past is compressed into one vector → bottleneck |
| **Vanishing gradients** | still present even with LSTM, just reduced |

A Transformer processes **all tokens in parallel** and connects any two positions with a **path length of 1**. That is the core advantage: $O(1)$ dependency path instead of $O(T)$.

**Cost of that choice:** attention is $O(T^2)$ in sequence length (every token attends to every token), so it is expensive for very long sequences. RNNs are $O(T)$; Transformers trade memory for parallelism and long-range modelling.

---

## 35. Scaled dot-product attention ★★★

### 35.1 Query, key, value

Each token embedding is projected into three vectors:

$$
Q = XW_Q, \qquad K = XW_K, \qquad V = XW_V
$$

- **Query** — what this token is looking for.
- **Key** — what this token offers for matching.
- **Value** — the information this token contributes if matched.

**Analogy:** a library. The query is your search phrase, the keys are the book titles/index entries, and the values are the book contents. You compare your query to every key, then take a weighted sum of the values.

### 35.2 The formula

$$
\operatorname{Attention}(Q,K,V) = \operatorname{softmax}\left(\frac{QK^\top}{\sqrt{d_k}}\right)V
$$

Step by step:

1. scores $S = QK^\top$ ($T\times T$; $S_{ij}$ = how much token $i$ cares about token $j$)
2. scale $S/\sqrt{d_k}$
3. weights $A = \operatorname{softmax}(S)$ (each row sums to 1)
4. output $AV$ (weighted average of the value vectors)

**Shapes:** $Q, K, V$ are $T\times d_k$ (usually $d_k = d_{\text{model}}/H$). $S$ and $A$ are $T\times T$. Output is $T\times d_v$.

**Attention matrix interpretation:** $A\in\mathbb{R}^{T\times T}$, $A_{ij}$ = how much token $i$ attends to token $j$, and $\sum_j A_{ij} = 1$ for every row. A large $A_{ij}$ means token $i$ pulls most of its information from token $j$.

### 35.3 Why divide by √d_k ★★★ (a guaranteed question)

The dot product of two independent random vectors of dimension $d_k$ (each component mean 0, variance 1) has variance $d_k$:

$$
\operatorname{Var}(q\cdot k) = \sum_{i=1}^{d_k}\operatorname{Var}(q_i k_i) = d_k
$$

So for $d_k = 512$, scores have standard deviation $\approx 22$. Large scores push softmax into saturation: one weight $\approx 1$, the rest $\approx 0$, and the **gradients through softmax become tiny** (`softmax` saturates). Dividing by $\sqrt{d_k}$ restores unit variance, keeping the softmax in its responsive range and the gradients healthy.

**Say it in one line:** *"scaling keeps the variance of the dot products at 1, so softmax does not saturate and gradients do not vanish."*

### 35.4 Multi-head attention ★★

One attention head can only express one kind of relationship. Use $H$ heads in parallel with smaller projections:

$$
\begin{aligned}
\text{head}_r &= \operatorname{Attention}\left(QW_r^Q,\; KW_r^K,\; VW_r^V\right) \\
\operatorname{MultiHead}(Q,K,V) &= \operatorname{Concat}(\text{head}_1, \dots, \text{head}_H)\,W^O
\end{aligned}
$$

Each head has dimension $d_k = d_{\text{model}} / H$, so total cost is the same as one full-width head. **Different heads learn different relations:** syntax (subject–verb), coreference ("it" → "animal"), local context, positional patterns, semantics.

**Why concatenate then project?** $W^O$ mixes the heads' information back into one $d_{\text{model}}$ representation.

### 35.5 The Transformer block ★★★ (draw this)

```
        x
        │
   ┌────┴────┐
   │ Multi-Head Attention
   └────┬────┘
        + ←──── x            (residual)
        │
    LayerNorm
        │
   ┌────┴────┐
   │ Feed-Forward (Linear → ReLU/GELU → Linear)
   └────┬────┘
        + ←──── (residual again)
        │
    LayerNorm
        │
        y
```

Two sub-layers, each with a **residual connection** and **layer normalisation**:

$$
\begin{aligned}
x' &= \operatorname{LayerNorm}\left(x + \operatorname{MultiHeadAttn}(x)\right) \\
y &= \operatorname{LayerNorm}\left(x' + \operatorname{FFN}(x')\right)
\end{aligned}
$$

- **Residual** $x + f(x)$ — gives the gradient highway (same idea as ResNet), so deep stacks train.
- **LayerNorm** — normalises across the feature dimension per token: $\frac{x-\mu}{\sqrt{\sigma^2+\epsilon}}\cdot\gamma+\beta$. It works on a single sample, so it is batch-size independent (unlike BatchNorm) — essential for variable-length text.
- **FFN** — a position-wise MLP, usually $d_{\text{model}} \to 4\cdot d_{\text{model}} \to d_{\text{model}}$; it does most of the parameter-heavy computation and acts as a key–value memory.

### 35.6 Positional encoding ★★★

Attention is **permutation-invariant**: shuffle the tokens and the output is the same shuffled output. Language needs order, so position information is added to the embeddings:

$$
x_i = \text{embedding}(\text{token}_i) + \text{positional\_encoding}(i)
$$

**Sinusoidal encoding:**

$$
\begin{aligned}
\text{PE}(\text{pos}, 2i) &= \sin\left(\text{pos} / 10000^{2i/d_{\text{model}}}\right) \\
\text{PE}(\text{pos}, 2i+1) &= \cos\left(\text{pos} / 10000^{2i/d_{\text{model}}}\right)
\end{aligned}
$$

Why sinusoids: each dimension is a sinusoid of a different frequency (like a binary counter), the encoding is unique per position, it generalises to longer sequences than seen in training, and relative positions are linear functions of the encoding (so the model can learn "attend to $k$ positions back").

**Learned positional embeddings** are the alternative (BERT, GPT) — a trainable table of size $\text{max\_len}\times d_{\text{model}}$. Simpler, but cannot extrapolate.

### 35.7 Encoder vs decoder transformers ★★

| | Encoder block | Decoder block |
|---|---|---|
| Attention | bidirectional self-attention | **masked** self-attention (causal) + cross-attention to the encoder |
| Sees | whole sequence at once | only tokens up to position $t$ |
| Good for | understanding: classification, NER, embeddings (BERT) | generation: translation, summarisation (T5, original Transformer) |
| Example | BERT, RoBERTa | GPT (decoder-only), T5 (enc–dec) |

**Causal mask:** before softmax, set $S_{ij} = -\infty$ for $j > i$. After softmax those weights are 0, so token $i$ cannot see the future. This is what makes next-token prediction possible.

**Cross-attention:** $Q$ comes from the decoder, $K$ and $V$ from the encoder output — this is how the decoder reads the source sentence.

### 35.8 Complexity and practical notes ★

$$
\begin{aligned}
\text{Self-attention:}\quad & O(T^2\cdot d) \quad \text{(compute)} \quad \text{and} \quad O(T^2) \text{ memory per head} \\
\text{FFN:}\quad & O(T\cdot d^2) \\
\text{RNN:}\quad & O(T\cdot d^2) \quad \text{compute, but sequential (}T\text{ steps)}
\end{aligned}
$$

For short sequences ($T < d$), attention is cheap; for very long $T$, $O(T^2)$ dominates — hence sparse attention, Linformer, Performer, FlashAttention.

**Activity-style example:** "The animal did not cross the street because **it** was too tired." The token "it" should attend strongly to "animal" (semantic plausibility), not "street" — a high $A_{\text{it},\text{animal}}$ with the right weights. That is exactly what multi-head attention learns.

### 35.9 Impact of transformers ★★★ (your notes: "explain all 5 points")

This is a flagged exam question in its own right — five named points, one line each:

| # | Impact | What to write |
|---|---|---|
| 1 | **Revolutionised NLP** | replaced RNN/LSTM everywhere; translation, summarisation, QA, chat all improved sharply |
| 2 | **Democratised AI** | pre-trained models (BERT, GPT) + transfer learning mean you fine-tune instead of training from scratch |
| 3 | **Multimodal capability** | the same block handles text, images (ViT), audio, video, protein sequences |
| 4 | **Accelerated generative AI** | GPT, ChatGPT, diffusion transformers — the engine of the current GenAI wave |
| 5 | **Unification of deep learning** | one architecture across modalities and tasks; CNNs/RNNs increasingly replaced or hybridised |

**Timeline to quote:** 2000–14 RNN/LSTM → 2014–15 encoder–decoder → 2016 attention → 2017 Transformer → 2018 BERT/GPT (transfer learning) → 2020–21 GenAI (GPT-3, Codex) → 2022+ ChatGPT.

**Advantages:** highly scalable (parallel training, low wall-clock time per epoch), transfer learning applies, multimodal, flexible architecture, integrates with generative and agentic AI.
**Disadvantages:** high compute/GPU cost, needs large datasets, overfitting risk from the large parameter count, high energy consumption.

### 35.10 Full architecture and the exam-marks rule ★★

**Components and their roles** (your notes list exactly these seven):

| Component | Role |
|---|---|
| **Input embedding** | token id → dense vector |
| **Positional encoding** | injects order information |
| **Multi-head attention** | tokens exchange information; **masked** in the decoder |
| **Add & Norm** | residual connection + layer normalisation, for stable deep training |
| **Feed-forward network** | position-wise MLP; most of the parameters |
| **Linear** | projects $d_{\text{model}}$ → vocabulary size |
| **Softmax** | logits → next-token probabilities |

**Stack size:** the original paper uses **6 encoder blocks + 6 decoder blocks** ($N = 6$); BERT-base uses 12, GPT-3 uses 96.

> Your notes say "6 decoders" without mentioning the encoders — the paper's figure is 6 of each. Write "6 encoder and 6 decoder blocks" and you are safe.

**Marks rule written in your notes:** *drawing the entire flowchart = 5 marks; "only the right flowchart" = 3 marks.* So even if you write nothing else, **draw the block diagram** — it is worth most of the marks.

```
INPUT → Embedding → +Positional Encoding
      → [ MHA → Add&Norm → FFN → Add&Norm ] × 6
      → Linear → Softmax → output probabilities
```

For the decoder, insert **masked** MHA first and **cross-attention** (Q from decoder, K/V from encoder) in the middle.

`Remember:` **"Q asks, K answers, V delivers."** And the scaling: **$\sqrt{d_k}$ because variance grows with $d_k$.** And **position because attention has no sense of order.**

`Exam:` "Explain scaled dot-product attention and multi-head attention with equations." Write Q/K/V projections, the softmax formula, the $\sqrt{d_k}$ justification, then the concat formula and one line on what different heads learn. Draw the block diagram for the full-mark version.

---

# PART IX — Practical deep learning

## 36. Keras / TensorFlow workflow ★★

The five-step pattern (this is the answer to "how do you build a model in Keras?"):

```python
import tensorflow as tf
from tensorflow.keras import layers, models

model = models.Sequential([
    layers.Input(shape=(28, 28, 1)),
    layers.Conv2D(32, (3, 3), activation='relu', padding='same'),
    layers.MaxPooling2D((2, 2)),
    layers.Conv2D(64, (3, 3), activation='relu', padding='same'),
    layers.MaxPooling2D((2, 2)),
    layers.Flatten(),
    layers.Dense(128, activation='relu'),
    layers.Dropout(0.5),
    layers.Dense(10, activation='softmax'),
])

model.compile(optimizer='adam',
              loss='sparse_categorical_crossentropy',
              metrics=['accuracy'])

model.summary()
history = model.fit(x_train, y_train,
                    epochs=10,
                    batch_size=32,
                    validation_split=0.2)
```

**What each call means:**

| Piece | Meaning |
|---|---|
| `Sequential` | a linear stack of layers |
| `Input(shape=…)` | the shape of **one** sample, without the batch dimension |
| `Conv2D(filters, kernel, padding, strides)` | 2-D convolution |
| `MaxPooling2D(pool_size)` | downsampling |
| `Flatten` | $(H,W,C) \to H\cdot W\cdot C$ vector |
| `Dense(units, activation)` | fully connected layer |
| `Dropout(p)` | regularisation |
| `compile(optimizer, loss, metrics)` | attaches the training algorithm |
| `fit(...)` | the training loop |
| `epochs` | full passes over the data |
| `batch_size` | examples per update |
| `validation_split` | fraction held out for validation |

**Loss/activation pairing table (memorise):**

| Task | Output activation | Loss (Keras name) |
|---|---|---|
| Binary classification | sigmoid, 1 unit | `binary_crossentropy` |
| Multiclass, integer labels | softmax, K units | `sparse_categorical_crossentropy` |
| Multiclass, one-hot labels | softmax, K units | `categorical_crossentropy` |
| Regression | linear, 1 unit | `mse` |
| Multilabel | sigmoid, K units | `binary_crossentropy` |

> The most common mistake in the course: **softmax + `binary_crossentropy`**, or **sigmoid + `categorical_crossentropy`**, or forgetting that `sparse_` means "integer labels, no one-hot".

**Shape reasoning for a convnet (write this in the exam):**

```
Input        (3, 32, 32)
Conv 32 3×3 same  → (32, 32, 32)      [same: p = 1]
MaxPool 2×2       → (32, 16, 16)
Conv 64 3×3 same  → (64, 16, 16)
MaxPool 2×2       → (64, 8, 8)
GlobalAvgPool     → (64,)
Dense 10          → (10,)
```

**Why `GlobalAveragePooling2D` instead of `Flatten` + `Dense`?** No weights (a $7\times 7\times 512$ flatten → 1000 dense needs 25M parameters; GAP needs 0), fewer parameters, less overfitting, and the model accepts any input size.

**`tf.GradientTape` (manual training loop):**

```python
with tf.GradientTape() as tape:
    logits = model(x_batch, training=True)
    loss = loss_fn(y_batch, logits)
grads = tape.gradient(loss, model.trainable_variables)
optimizer.apply_gradients(zip(grads, model.trainable_variables))
```

`GradientTape` records the forward operations so it can replay them backwards — it *is* backpropagation, implemented by autodiff. `tape.gradient` returns $\partial\,\text{loss}/\partial\,\text{variable}$ for each variable.

**Callbacks worth naming:** `ModelCheckpoint` (save the best weights), `EarlyStopping(patience=…)`, `ReduceLROnPlateau`, `TensorBoard`.

**Manual verification (you should be able to do this by hand):** given $w = 1$, $x = 2$, $\hat{y} = wx = 2$, $L = \frac{1}{2}(y-\hat{y})^2$ with $y = 3$: $\frac{dL}{dw} = (\hat{y}-y)\cdot x = (-1)(2) = -2$; update with $\eta = 0.1$ gives $w = 1.2$. Frameworks produce exactly this number — checking it once proves you understand autodiff.

---

## 37. Hugging Face and text generation ★★

**Hugging Face** provides pre-trained models, tokenizers, datasets, training utilities and inference pipelines.

```python
from transformers import pipeline

generator = pipeline("text-generation", model="gpt2")
print(generator("Deep learning is transforming", max_length=30))
```

**Common pipelines:** `sentiment-analysis`, `text-generation`, `summarization`, `translation`, `question-answering`, `fill-mask`, `image-classification`, `automatic-speech-recognition`.

**Caution:** a pipeline is only as good as its model and data — check the licence, the training data, the language, and the evaluation. Never assume the output is correct.

### 37.1 Sampling strategies ★★★

Given next-token probabilities $p(w \mid \text{context})$:

| Strategy | Rule | Effect |
|---|---|---|
| **Greedy** | pick $\arg\max p$ | deterministic, repetitive |
| **Beam search** | keep the $k$ best partial sequences | better likelihood, still repetitive, more compute |
| **Temperature** | sample from $\operatorname{softmax}(z_i/T)$ | $T < 1$ sharper/more deterministic; $T > 1$ flatter/more random |
| **Top-k** | sample from the $k$ highest-probability tokens | removes the long tail, keeps variety |
| **Nucleus (top-p)** | smallest set of tokens whose cumulative probability $\geq p$ (e.g. 0.9) | adaptive: keeps few tokens when confident, many when uncertain |

**Temperature formula:**

$$
p_T(w_i) = \frac{\exp(z_i/T)}{\sum_j \exp(z_j/T)}
$$

$T \to 0$ ⟹ greedy; $T = 1$ ⟹ the model's own distribution; $T \to \infty$ ⟹ uniform.

**Why temperature works:** dividing logits by $T < 1$ magnifies differences (sharper distribution); $T > 1$ compresses them (flatter).

### 37.2 LSTM text generation ★★

Train an LSTM to predict the next token:

$$
\begin{aligned}
h_t &= \operatorname{LSTM}(x_t, h_{t-1}) \\
p(w_{t+1} \mid \text{context}) &= \operatorname{softmax}(W_o h_t + b_o) \\
\text{Loss} &= -\sum_t \log p(w_{t+1} \mid w_1 \dots w_t)
\end{aligned}
$$

At generation time: feed the prompt, sample a token, append it, repeat. This is exactly an autoregressive model — the difference from GPT is the architecture, not the objective.

---

## 38. DeepDream and neural style transfer ★★

### 38.1 DeepDream

Pick a layer's activations and **maximise** them by gradient ascent **on the input image** (not on the weights):

$$
\begin{aligned}
L &= \sum \big(A_l(x)\big)^2 \qquad \text{(or a specific channel/filter)} \\
x &\leftarrow x + \eta \nabla_x L
\end{aligned}
$$

The network amplifies whatever patterns that layer detects — edges, eyes, dog faces — producing hallucinated, recursive imagery. It is feature visualisation, not generation.

**Interpretation:** it shows what a filter has learned. Early layers produce edges and swirls; deeper layers produce object-like patterns.

### 38.2 Neural style transfer

Combine the **content** of one image with the **style** of another. Use a pre-trained CNN (VGG) frozen as a feature extractor:

$$
\begin{aligned}
\text{Total loss:}\quad L &= \alpha L_{\text{content}} + \beta L_{\text{style}} \\
\text{Content loss:}\quad L_{\text{content}} &= \Vert F_l(\text{content}) - F_l(\text{generated}) \Vert^2 \\
&\qquad \text{(feature maps of a deep layer must match)} \\
\text{Style loss:}\quad \text{Gram matrix}\quad G_l &= F_l F_l^\top \qquad \text{(channel correlations = texture)} \\
L_{\text{style}} &= \sum_l \Vert G_l(\text{style}) - G_l(\text{generated}) \Vert^2
\end{aligned}
$$

Optimise the *pixels* of the generated image, not the weights.

**Why the Gram matrix captures style:** it measures how strongly two feature channels co-occur, which encodes texture and colour statistics independent of spatial arrangement. Matching Gram matrices transfers texture without copying layout.

**Diagram to draw:** content image + style image → pre-trained CNN → content loss (deep layer) and style loss (Gram, multiple layers) → total loss → gradient update on the generated image.

---

## 39. Applications, pipelines and ethics ★

**Tool selection** (a typical short question):

| Need | Tool |
|---|---|
| Text generation / chat | LLM + Hugging Face pipeline |
| Artistic style transfer | neural style transfer |
| Visualise what a filter learned | DeepDream |
| Smooth latent interpolation, anomaly detection | VAE |
| Photorealistic face/image generation | GAN / diffusion |
| Super-resolution, inpainting | conditional GAN / diffusion |

`Remember:` **Temperature = "creativity dial"; top-p = "cut the tail"; Gram matrix = "texture fingerprint"; DeepDream = "gradient ascent on the image".**

### 39.1 Application pipelines: vision, speech, NLP ★★

**Computer vision pipeline:**

```
image → resize/normalise → CNN (conv blocks) → global pooling → dense → softmax → label
```

**Speech-recognition pipeline:**

```
waveform → frame (25 ms, 10 ms hop) → spectrogram / log-mel features
        → CNN (local time-frequency patterns) + RNN/LSTM (temporal order)
        → CTC or seq2seq decoder → text
```

A **spectrogram** is time on the x-axis, frequency on the y-axis, energy as colour. It has two structures at once: **local** time-frequency patterns (good for convolution) and **long-range temporal order** (good for recurrence/attention) — which is why hybrid CNN+RNN models are common.

**NLP pipeline:**

```
text → tokenise → embedding lookup (V × E) → sequence encoder (RNN/LSTM/Bi-LSTM/Transformer)
     → task head → output
```

| Task | Pattern | Head |
|---|---|---|
| Sentiment | many-to-one | last hidden state → dense → sigmoid/softmax |
| POS tagging / NER | many-to-many aligned | one output per token (Bi-LSTM) |
| Translation | many-to-many unaligned | encoder–decoder (+ attention) |
| Text generation | autoregressive | next-token softmax at each step |

**Why Bi-LSTM for tagging:** the tag of a word depends on both the words before and after it. **Why not Bi-LSTM for generation:** generation is causal — you cannot see the future.

### 39.2 Architecture selection matrix ★★

| Task | Model | Reason |
|---|---|---|
| Image classification | CNN (ResNet/EfficientNet) | exploits spatial structure and translation |
| Object detection / segmentation | CNN + feature pyramid / U-Net | needs spatial output |
| Mobile / edge inference | depthwise-separable CNN | cheap in FLOPs and memory |
| Real-time sensor forecasting | RNN/LSTM (unidirectional) | causal temporal state, low latency |
| Long-document understanding | Transformer encoder | parallel, long-range |
| Text generation / chat | decoder-only Transformer | causal, scalable |
| Machine translation | encoder–decoder Transformer | full source context + generation |
| Anomaly detection | autoencoder / VAE | reconstruction error |
| Photorealistic generation | GAN / diffusion | sharp samples |
| Smooth latent interpolation | VAE | structured latent space |
| Few labels, similar domain | transfer learning | reuse features |
| Distribution shift at deployment | domain adaptation | align features |

**One-line justifications are what earn the marks** — never just name the model.

**Ethical and practical concerns:** hallucinated or incorrect outputs (confident but wrong), copyright and data-origin issues, biased generation, privacy (training data leakage, deepfakes), misuse for deception, and over-reliance without verification. Always state that generative output needs human verification.

---

# PART X — How to learn and memorise this

## 40. The method

Rote memorisation fails on this syllabus because there are ~80 formulas and no time to drill them all. The syllabus is built from **a small number of patterns**; learn the patterns and the formulas regenerate themselves. Four passes over each topic:

| Pass | What you do | Time per topic |
|---|---|---|
| 1. **Understand** | read the explanation, say *why* the formula must look like that | 10 min |
| 2. **Derive** | reproduce the formula from the pattern with the book closed | 5 min |
| 3. **Recall** | blank page: draw the diagram + write the formula | 3 min |
| 4. **Apply** | one numerical, no notes | 5 min |

If you can do pass 3 and 4, you know it. Reading is not knowing.

### 40.1 Derive-don't-memorise table ★

This is the highest-leverage table in the guide. Each row: the formula you think you must memorise, and the 10-second derivation that regenerates it.

| Formula | Regenerate it by | Check |
|---|---|---|
| **Conv output size** $\lfloor (n+2p-k)/s \rfloor + 1$ | "how many placements of a k-window fit in n+2p when stepping s" | $n=k,\ p=0,\ s=1 \to 1$ ✓ |
| **Conv params** $(K^2C_{\text{in}}+1)C_{\text{out}}$ | each filter is $K^2C_{\text{in}}$ weights + 1 bias; one per output channel | 3×3 on RGB, 64 filters → 1792 ✓ |
| **Depthwise ratio** $1/C_{\text{out}} + 1/K^2$ | divide $K^2C_{\text{in}} + C_{\text{in}}C_{\text{out}}$ by $K^2C_{\text{in}}C_{\text{out}}$ | $K=3,\ C=256 \to 0.115$ ✓ |
| **MLP params** $\sum n_{(l-1)}n_l + \sum n_l$ | every connection + every non-input node | 4-3-2-1 → 26 ✓ |
| **LSTM params** $4[(h+d)h + h]$ | 4 blocks × (input+hidden weights + bias) | $d=3,\ h=4 \to 128$ ✓ |
| **Softmax gradient** $\hat{y} - y$ | differentiate $-\log \hat{y}_c$ and use $d\hat{y}/dz$ | sums to 0 ✓ |
| **Adam first step $\approx \eta$** | bias correction divides by $1-\beta_1^t$, which cancels the $1-\beta_1$ | any $g \to$ step $\eta$ ✓ |
| **KL of Gaussians** $\frac{1}{2}\sum(\mu^2+\sigma^2-1-\log\sigma^2)$ | $E[\log q - \log p]$ for two normals | $\mu=0,\ \sigma=1 \to 0$ ✓ |
| **VAE loss** | $-\log p(x \mid z)$ (reconstruct) + $\mathrm{KL}(q \Vert \text{prior})$ (regularise) | both terms ≥ 0 ✓ |
| **GAN optimum** $D^* = p_{\text{data}}/(p_{\text{data}}+p_g)$ | set $\partial V/\partial D = 0$ | equal distributions → ½ ✓ |
| **Attention scale** $\sqrt{d_k}$ | $\mathrm{Var}(q\cdot k) = d_k$, so divide by the std | $d=512 \to \sim 22$ ✓ |
| **BPTT gradient product** | chain rule over the unrolled graph | $T-k$ factors ✓ |
| **LSTM gradient** $\partial c_t/\partial c_{(t-1)} = \operatorname{diag}(f_t)$ | differentiate the cell equation | $f=1 \to$ unchanged ✓ |
| **XOR impossible** | four inequalities + contradiction | — |
| **Bias–variance** | $E[(y-\hat{h})^2] = \text{bias}^2 + \text{var} + \text{noise}$ | — |

**Rule:** if you cannot regenerate it, you do not know it. A memorised formula fails under exam stress; a derivable one does not.

### 40.2 Mnemonics and hooks ★

| Topic | Hook |
|---|---|
| AI hierarchy | **A-M-R-D** — "A Machine Really Delivers" |
| Perceptron update | **"mistakes only"**: $w \mathrel{+}= \alpha(t-o)x$, zero when right |
| XOR | needs **2 lines** → 2 hidden neurons; AND/OR need 0 |
| Backprop | **"forward to compute, backward to blame, step to improve"** |
| Chain rule order | **error × weight × derivative** (in that order, always) |
| Activations | **"Sigmoid yes/no, softmax which-one, linear how-much"** |
| Sigmoid derivative | **$a(1-a)$** — output times (1 − output) |
| Vanishing gradient | **$0.25^L$** — $0.25^{10} \approx 10^{-6}$ |
| L1 vs L2 | **"L1 kills, L2 shrinks"** (zeros vs small) |
| Regularisers | **"L1 kills, L2 shrinks, dropout ensembles, early stopping saves"** |
| Optimizers | **"Momentum remembers direction, RMSProp magnitude, Adam both"** |
| Init | **Xavier averages fan-in/out; He doubles for ReLU** |
| CNN order | **"conv → nonlin → downsample"**, repeated |
| Pooling | **"conv = what is here; pool = roughly where and how strong"** |
| Equivariance | **"conv is equivariant, pooling makes it invariant"** |
| Depthwise | two stages: **depth (per-channel) then point (1×1 mix)** — "**D then P**" |
| RNN | **"same layer at every tick, plus a memory wire"** |
| BPTT | **"backprop on an unrolled graph"** |
| LSTM gates | **FICO** (like the credit score): Forget, Input, Candidate, Output |
| LSTM cell update | **"keep + write"**: $c = f\odot c_{\text{old}} + i\odot\tilde{c}$; $h = o\odot\tanh(c)$ |
| LSTM gradient | **"f near 1 = gradient highway"** |
| Bi-LSTM | **"read the sentence twice, once backwards"** |
| Recurrent vs recursive | **"line vs tree"** |
| PCA family | **"PCA rotates, PPCA adds noise, FA adds per-feature noise, ICA unmixes"** |
| Autoencoder | **"compress → bottleneck → reconstruct"**, target = input |
| Overcomplete AE | **identity mapping** — perfect reconstruction, zero learning |
| Denoising AE | **"damage the input so it cannot copy"** |
| Contractive AE | **"penalise the Jacobian"** — be flat off the manifold |
| VAE | **"encoder predicts a cloud, not a point"**; reparam = **"move the dice outside"** |
| KL check | $\mu=0,\ \sigma=1 \to 0$ (prior against itself) |
| GAN | **"forger vs detective"**, equilibrium at **0.5** |
| Mode collapse | **"one perfect forgery, repeated"** — sharp but few |
| Transfer learning | **"freeze features, fine-tune specifics"** |
| Domain adaptation | **"same task, different distribution"** |
| Attention | **"Q asks, K answers, V delivers"** |
| Positional encoding | **"attention has no clock"** |
| Gram matrix | **"texture fingerprint"** |
| Temperature | **"creativity dial"**; top-p = **"cut the tail"** |

### 40.3 The pattern tables (compress 80 formulas into 8)

**Pattern 1 — every layer is affine + non-linearity.** $z = Wx + b$, $a = g(z)$. Backprop is always $\delta = (W^\top \delta_{\text{next}}) \odot g'(z)$ and $\partial L/\partial W = \delta a_{\text{prev}}^\top$. MLP, CNN (conv replaces matmul), RNN (shared W), transformer (attention replaces matmul) — all the same skeleton.

**Pattern 2 — every CNN is `conv → nonlin → downsample`.** Differences are only: kernel size, filters, stride, padding, skip connections.

**Pattern 3 — every RNN-family cell is $\text{gate} = \sigma(\text{linear})$.** LSTM has 3 gates + 1 candidate; GRU has 2 gates. Only the *wiring* differs.

**Pattern 4 — every latent model is $x = \text{matrix}\times z + \text{noise}$.** PCA: no noise. PPCA: $\sigma^2 I$. FA: $\Psi$ diagonal. ICA: no noise, non-Gaussian $z$.

**Pattern 5 — every autoencoder is `reconstruction + penalty`.** Vanilla: penalty 0. Sparse: L1/KL on activations. Denoising: noise on input. Contractive: Jacobian norm. VAE: KL on the latent distribution.

**Pattern 6 — every generative model trades likelihood for quality.** AR/flow: exact likelihood, slow/restricted. VAE: approximate, blurry, smooth latent. GAN: implicit, sharp, unstable. Diffusion: approximate, best quality, slowest.

**Pattern 7 — every "why deep" answer has three parts:** parameters (fewer via sharing), structure (locality/hierarchy), optimisation (gradients/initialisation).

**Pattern 8 — every regulariser adds a penalty or noise.** If you forget one, ask "what does it penalise?" — weights, activations, sensitivity, or co-adaptation.

### 40.4 Diagram recall drill ★★★

Draw these **from memory, on blank paper, in under 2 minutes each**. If you cannot, the topic is not learned. Do all seven the night before and again in the morning.

```
1. MLP 2-2-1 with forward arrows and backward δ arrows, labelled with equations
2. CNN pipeline with shapes written at every stage (3×32×32 → …)
3. LSTM cell: c line on top, 3 σ gates, tanh, ⊙/⊕, h_t output
4. VAE: x → encoder → (μ,σ) → z → decoder → x̂, with the KL arrow to N(0,I)
5. GAN: z → G → fake → D ← real, with the two losses written on the arrows
6. Transformer block: MHA → Add&Norm → FFN → Add&Norm, with residuals
7. Autoencoder hourglass with the bottleneck labelled "latent space"
```

**Scoring:** if the diagram is right but labels are missing, you lose half the marks. Label while drawing.

### 40.5 Numerical drills (do these, then check)

1. $n=13$, $k=3$, $p=1$, $s=2$ → output size?
2. 12×12×3 input, 256 filters 3×3, no padding → standard vs depthwise multiplications?
3. MLP 4-3-2-1 → trainable parameters?
4. RNN $d=6$, $h=4$, $o=1$ → parameters?
5. LSTM $d=3$, $h=4$ → parameters?
6. $x=(1,2)$, $w=(1,1)$, $b=1$, sigmoid, $y=1$ → forward pass and one backprop step with $\eta=1$?
7. LSTM with $c_{\text{old}}=0.5$, $z_f=0.4$, $z_i=0.8$, $z_c=0.6$, $z_o=1.0$ → $c_t, h_t$?
8. Softmax of $(2,1,0.1)$?
9. $P_s(x) \neq P_t(x)$ — name the shift and one technique that fixes it?
10. GAN: $D(x)=0.9$, $D(G(z))=0.2$, $m=1$ → $J_G$ and $J_D$?

**Answers.**
1. $\lfloor (13+2-3)/2 \rfloor + 1 = 7$.
2. Standard $10\cdot 10\cdot 256\cdot 9\cdot 3 = 691{,}200$; depthwise $2{,}700 + 76{,}800 = 79{,}500$.
3. $20 + 6 = 26$.
4. $24+16+4+4+1 = 49$.
5. $4[(3+4)\cdot 4+4] = 128$.
6. $z = 4$, $\hat{y} = 0.982$; $\delta = -2(1-0.982)(0.982\cdot 0.018) = -6.4\times 10^{-4}$; $\partial L/\partial w = \delta x = (-6.4\times 10^{-4}, -1.28\times 10^{-3})$; update $w = (1.00064, 1.00128)$.
7. $f=0.5987$, $i=0.6900$, $\tilde{c}=0.5370$ → $c_t = 0.6699$; $o=0.7311$, $h_t = 0.4276$.
8. $(0.659, 0.242, 0.099)$.
9. Covariate shift; feature alignment or adversarial DA.
10. $J_G = -\log 0.2 = 1.609$; $J_D = -\log 0.9 - \log(1-0.2) = 0.105 + 0.223 = 0.328$.

### 40.6 Spaced-repetition schedule

| Day | Session (60–90 min) |
|---|---|
| **D−7** | Parts I–II: read + redraw diagrams 1, 2. Do drills 1–6. |
| **D−5** | Parts III–IV: LSTM cell + PCA family table from memory. Drills 7–10. |
| **D−3** | Parts V–VII: AE family table, VAE/GAN equations, TL/DA diagram. |
| **D−2** | Part VIII + IX: attention formula + transformer block. Full formula sheet read. |
| **D−1** | **Blank-page test:** write the entire formula sheet from memory. Redraw all 7 diagrams. Do the 100 self-tests. |
| **Exam morning** | Last-3-hours sheet only (§44). No new material. |

**The blank-page test is the single best use of the last day.** Write everything you can remember on one sheet, then compare with §41 and mark what you missed in red. Repeat only the red items.

### 40.7 Teach-back (Feynman) prompts

Explain each in 60 seconds without notes; if you stall, that is your gap:

1. Why does a stack of linear layers collapse?
2. Why does the sigmoid cause vanishing gradients, and why does ReLU not?
3. Why is cross-entropy better than MSE for classification?
4. Why does a convolution need fewer parameters than a dense layer?
5. Why does an LSTM remember longer than an RNN?
6. Why does the VAE have a KL term?
7. Why is a GAN unstable while a VAE is not?
8. Why divide attention by $\sqrt{d_k}$?
9. Why freeze early layers in transfer learning?
10. Why does an autoencoder detect anomalies?

### 40.8 If you have one day left

1. **Two hours:** read §41 formula sheet twice; write it from memory once.
2. **Two hours:** redraw the 7 diagrams, twice each.
3. **Two hours:** do the numericals in §40.5 and the self-tests in §42.
4. **One hour:** the trap list (§43) and last-3-hours sheet (§44).
5. **Sleep.** Cramming past midnight costs more marks than it earns.

---

## 41. Master formula sheet

### 41.1 Foundations

$$
\begin{aligned}
\text{Neuron:} &\quad y = f\Big(\sum w_i x_i + b\Big) = f(w\cdot x + b) \\
\text{Perceptron update:} &\quad w \leftarrow w + \alpha(t-o)x,\quad b \leftarrow b + \alpha(t-o) \\
\text{MP neuron:} &\quad y = 1 \text{ if } \sum w_i x_i \geq t \text{ else } 0
\end{aligned}
$$

$$
\begin{aligned}
\text{MLP params:} &\quad \sum n_{(l-1)}\cdot n_l + \sum n_l \\
\text{Forward:} &\quad z^{(l)} = W^{(l)\top} a^{(l-1)} + b^{(l)},\quad a^{(l)} = g(z^{(l)})
\end{aligned}
$$

$$
\begin{aligned}
\text{MSE:} &\quad \frac{1}{N}\sum (y-\hat{y})^2 \qquad \text{MAE: } \frac{1}{N}\sum |y-\hat{y}| \qquad \text{RMSE: } \sqrt{\text{MSE}} \\
\text{BCE:} &\quad -\big[y\log\hat{y} + (1-y)\log(1-\hat{y})\big] \\
\text{Categorical CE:} &\quad -\sum y_k \log \hat{y}_k \to -\log \hat{y}_c \\
\text{Softmax:} &\quad \hat{y}_k = \frac{e^{z_k}}{\sum_j e^{z_j}} \\
\text{Softmax+CE gradient:} &\quad \frac{\partial L}{\partial z} = \hat{y} - y \quad (\text{same for sigmoid+BCE})
\end{aligned}
$$

$$
\begin{aligned}
\text{Gradient descent:} &\quad \theta \leftarrow \theta - \eta \frac{\partial L}{\partial \theta} \\
\text{Mini-batch gradient:} &\quad g = \frac{1}{|B|}\sum_{i\in B}\nabla L_i \quad (\text{unbiased}) \\
\text{Momentum:} &\quad v \leftarrow \beta v + g,\quad \theta \leftarrow \theta - \eta v \\
\text{AdaGrad:} &\quad G \leftarrow G + g^2,\quad \theta \leftarrow \theta - \eta g/(\sqrt{G}+\epsilon) \\
\text{RMSProp:} &\quad s \leftarrow \beta s + (1-\beta)g^2,\quad \theta \leftarrow \theta - \eta g/(\sqrt{s}+\epsilon) \\
\text{Adam:} &\quad m \leftarrow \beta_1 m + (1-\beta_1)g,\quad v \leftarrow \beta_2 v + (1-\beta_2)g^2 \\
&\quad \hat{m} = m/(1-\beta_1^t),\quad \hat{v} = v/(1-\beta_2^t) \\
&\quad \theta \leftarrow \theta - \eta\,\hat{m}/(\sqrt{\hat{v}}+\epsilon) \\
\text{Clip:} &\quad \text{if } \Vert g\Vert_2 > \tau:\ g \leftarrow \tau g/\Vert g\Vert_2
\end{aligned}
$$

$$
\begin{aligned}
\text{Sigmoid:} &\quad \sigma = 1/(1+e^{-z}),\quad \sigma' = \sigma(1-\sigma) \leq 0.25 \\
\text{Tanh:} &\quad (e^z - e^{-z})/(e^z + e^{-z}) = 2\sigma(2z)-1,\quad \tanh' = 1-\tanh^2 \\
\text{ReLU:} &\quad \max(0,z),\quad \text{Leaky: } z \text{ if } z>0 \text{ else } 0.01z \\
\text{SELU:} &\quad \lambda=1.0507,\ \alpha=1.6733
\end{aligned}
$$

### 41.2 Backpropagation

$$
\begin{aligned}
\delta^{(L)} &= \frac{\partial L}{\partial a^{(L)}} \odot g'(z^{(L)}) \quad [\, = \hat{y} - y \text{ for softmax/sigmoid + CE}\,] \\
\delta^{(l)} &= (W^{(l+1)\top}\delta^{(l+1)}) \odot g'(z^{(l)}) \\
\frac{\partial L}{\partial W^{(l)}} &= \delta^{(l)} a^{(l-1)\top}, \qquad \frac{\partial L}{\partial b^{(l)}} = \delta^{(l)} \\
\theta &\leftarrow \theta - \eta \frac{\partial L}{\partial W}
\end{aligned}
$$

### 41.3 Regularisation and training

$$
\begin{aligned}
\text{L1:} &\quad \text{Cost} = \text{Loss} + \lambda\sum|w| \quad \text{gradient } +\lambda\cdot\operatorname{sign}(w) \to \text{sparse} \\
\text{L2:} &\quad \text{Cost} = \text{Loss} + \lambda\sum w^2 \quad \text{gradient } +2\lambda w \to \text{weight decay} \\
\text{Ridge closed form:} &\quad w^* = (X^\top X + \lambda I)^{-1}X^\top y \\
\text{Dropout:} &\quad h' = (m\odot h)/(1-p),\ m \sim \text{Bernoulli}(1-p),\quad E[h'] = h \\
\text{BatchNorm:} &\quad \hat{x} = (x-\mu_B)/\sqrt{\sigma_B^2+\epsilon},\quad y = \gamma\hat{x} + \beta \\
\text{Xavier:} &\quad \text{Var}(w) = 2/(n_{\text{in}}+n_{\text{out}}) \qquad \text{He: } \text{Var}(w) = 2/n_{\text{in}}
\end{aligned}
$$

$$
E[(y-\hat{h})^2] = \text{Bias}^2 + \text{Variance} + \sigma^2
$$

### 41.4 CNN

$$
\begin{aligned}
\text{Convolution:} &\quad S[i,j] = \sum_m\sum_n X[i+m,j+n]K[m,n] \quad (\text{cross-correlation}) \\
\text{Multi-channel:} &\quad S[i,j] = \sum_{m,n,c} X[i+m,j+n,c]K[m,n,c] \\
\text{Output size:} &\quad \lfloor (n + 2p - k)/s \rfloor + 1 \\
\text{Same padding:} &\quad p = (k-1)/2 \\
\text{Dilation:} &\quad k_{\text{eff}} = d(k-1) + 1 \\
\text{Receptive field:} &\quad r_l = r_{(l-1)} + (k_l-1)\cdot\prod_{i<l}s_i \\
\text{Conv params:} &\quad (K^2C_{\text{in}} + 1)\cdot C_{\text{out}} \\
\text{Std multiplications:} &\quad H'W'\cdot C_{\text{out}}\cdot K^2\cdot C_{\text{in}}
\end{aligned}
$$

$$
\begin{aligned}
\text{Pooling output:} &\quad \lfloor (n-f)/s \rfloor + 1 \\
\text{GAP:} &\quad y_c = \frac{1}{HW}\sum X[i,j,c] \\
\text{Depthwise:} &\quad H'W'\cdot C_{\text{in}}\cdot K^2 + H'W'\cdot C_{\text{in}}\cdot C_{\text{out}} \\
\text{Reduction ratio:} &\quad 1/C_{\text{out}} + 1/K^2 \\
\text{Spatially separable:} &\quad K^2 \to 2K \\
\text{FFT cost:} &\quad \mathcal{O}(N^2\log N) \text{ vs direct } \mathcal{O}(N^2K^2) \\
\text{ResNet:} &\quad y = F(x) + x
\end{aligned}
$$

### 41.5 Sequences

$$
\begin{aligned}
\text{RNN:} &\quad h_t = \tanh(W_{xh} x_t + W_{hh} h_{t-1} + b_h) \\
&\quad o_t = W_{hy} h_t + b_y,\quad \hat{y}_t = \psi(o_t) \\
\text{Params:} &\quad d\cdot h + h\cdot h + h + h\cdot o + o \\
\text{Loss:} &\quad L = \sum_t L_t(\hat{y}_t, y_t) \\
\text{BPTT:} &\quad \partial h_t/\partial h_k = \prod_{j=k+1}^{t} \operatorname{diag}(\varphi'(z_j)) W_{hh} \\
&\quad \partial L/\partial W_{hh} = \sum_t \delta_t h_{t-1}^\top \\
&\quad \delta_t = (\partial L_t/\partial h_t) + (W_{hh}^\top \operatorname{diag}(\varphi'))\delta_{t+1}
\end{aligned}
$$

$$
\begin{aligned}
\text{LSTM:} &\quad f_t = \sigma(W_f[h_{t-1};x_t]+b_f) \\
&\quad i_t = \sigma(W_i[\,\cdot\,]+b_i) \\
&\quad \tilde{c}_t = \tanh(W_c[\,\cdot\,]+b_c) \\
&\quad c_t = f_t\odot c_{t-1} + i_t\odot\tilde{c}_t \\
&\quad o_t = \sigma(W_o[\,\cdot\,]+b_o) \\
&\quad h_t = o_t\odot\tanh(c_t) \\
\text{Params:} &\quad 4[(h+d)h + h] \\
\text{Gradient:} &\quad \partial c_t/\partial c_{t-1} = \operatorname{diag}(f_t)
\end{aligned}
$$

$$
\begin{aligned}
\text{GRU:} &\quad z_t = \sigma(W_z[\,\cdot\,]),\quad r_t = \sigma(W_r[\,\cdot\,]) \\
&\quad \tilde{h}_t = \tanh(W_h[r_t\odot h_{t-1}; x_t]) \\
&\quad h_t = (1-z_t)\odot h_{t-1} + z_t\odot\tilde{h}_t
\end{aligned}
$$

$$
\begin{aligned}
\text{Bi-LSTM:} &\quad h_t = [\overrightarrow{h}_t ; \overleftarrow{h}_t] \\
\text{Attention (seq2seq):} &\quad \alpha_{ij} = \operatorname{softmax}_j f(h_{j-1}, s_{i-1}) \\
&\quad C_i = \sum_j \alpha_{ij} h_{j-1}
\end{aligned}
$$

### 41.6 Latent models

$$
\begin{aligned}
\text{PCA:} &\quad C = (1/(N-1))X^\top X,\quad Cv = \lambda v,\quad Z = XW_K \\
&\quad \text{explained variance} = \lambda_i/\sum\lambda_j
\end{aligned}
$$

$$
\begin{aligned}
\text{PPCA:} &\quad z \sim \mathcal{N}(0,I),\quad x = \mu + Wz + \epsilon,\ \epsilon \sim \mathcal{N}(0,\sigma^2 I) \\
&\quad x \sim \mathcal{N}(\mu, WW^\top + \sigma^2 I) \\
&\quad M = W^\top W + \sigma^2 I \\
&\quad E[z \mid x] = M^{-1}W^\top(x-\mu)
\end{aligned}
$$

$$
\begin{aligned}
\text{FA:} &\quad x = \mu + \Lambda z + \epsilon,\ \epsilon \sim \mathcal{N}(0,\Psi),\ \Psi \text{ diagonal} \\
&\quad x \sim \mathcal{N}(\mu, \Lambda\Lambda^\top + \Psi)
\end{aligned}
$$

$$
\begin{aligned}
\text{ICA:} &\quad x = As,\quad y = Wx \approx s \\
&\quad \text{maximise non-Gaussianity (kurtosis, negentropy)} \\
&\quad \text{ambiguity: permutation + scale + sign}
\end{aligned}
$$

### 41.7 Autoencoders and generative models

$$
\begin{aligned}
\text{AE:} &\quad z = E(x),\quad \hat{x} = D(z),\quad L = \Vert x - \hat{x}\Vert^2 \\
\text{BCE recon:} &\quad L = -\sum[x\log\hat{x} + (1-x)\log(1-\hat{x})] \\
\text{Sparse:} &\quad L + \lambda\sum|h_j| \quad \text{or} \quad L + \beta\sum \mathrm{KL}(\rho \Vert \hat{\rho}_j) \\
\text{Contractive:} &\quad L + \lambda\sum_{i,j}(\partial h_j/\partial x_i)^2 \\
\text{Denoise:} &\quad \tilde{x} = x + \epsilon,\quad L = \Vert x - D(E(\tilde{x}))\Vert^2
\end{aligned}
$$

$$
\begin{aligned}
\text{VAE:} &\quad z = \mu + \sigma\odot\epsilon,\ \epsilon \sim \mathcal{N}(0,I) \\
&\quad L = -E[\log p(x \mid z)] + \mathrm{KL}(q(z \mid x) \Vert p(z)) \\
&\quad \mathrm{KL} = \frac{1}{2}\sum(\mu_i^2 + \sigma_i^2 - 1 - \log\sigma_i^2) \\
&\quad \sigma = \exp\big(\frac{1}{2}\log\sigma^2\big)
\end{aligned}
$$

$$
\begin{aligned}
\text{GAN:} &\quad \min_G\max_D E[\log D(x)] + E[\log(1-D(G(z)))] \\
&\quad J_G = -(1/m)\sum\log D(G(z_i)) \\
&\quad J_D = -(1/m)\sum\log D(x_i) - (1/m)\sum\log(1-D(G(z_i))) \\
&\quad D^*(x) = p_{\text{data}}/(p_{\text{data}}+p_g)
\end{aligned}
$$

$$
\begin{aligned}
\text{AR:} &\quad p(x) = \prod p(x_i \mid x_{<i}) \\
\text{Flow:} &\quad \log p_X(x) = \log p_Z(z) + \log\left|\det\frac{\partial f}{\partial x}\right| \\
\text{Diffusion:} &\quad x_t = \sqrt{\bar{\alpha}_t}\,x_0 + \sqrt{1-\bar{\alpha}_t}\,\epsilon \\
&\quad L = E\Vert \epsilon - \epsilon_\theta(x_t,t)\Vert^2
\end{aligned}
$$

### 41.8 Transformers

$$
\begin{aligned}
&\quad Q = XW_Q,\quad K = XW_K,\quad V = XW_V \\
\text{Attention}(Q,K,V) &= \operatorname{softmax}(QK^\top/\sqrt{d_k})V \\
\text{MultiHead} &= \text{Concat}(\text{head}_1\dots\text{head}_H)W^O \\
\text{Block:} &\quad x' = \mathrm{LN}(x + \mathrm{MHA}(x)),\quad y = \mathrm{LN}(x' + \mathrm{FFN}(x')) \\
\text{PE}(pos,2i) &= \sin\big(pos/10000^{2i/d}\big) \\
\text{PE}(pos,2i+1) &= \cos\big(pos/10000^{2i/d}\big) \\
\text{Causal mask:} &\quad S_{ij} = -\infty \text{ for } j > i
\end{aligned}
$$

### 41.9 Practical

$$
\begin{aligned}
\text{Sampling:} &\quad p_T(w_i) = \frac{\exp(z_i/T)}{\sum_j\exp(z_j/T)} \\
\text{Softmax+CE} &\leftrightarrow \text{softmax} + \texttt{categorical\_crossentropy} \\
\text{sigmoid+BCE} &\leftrightarrow \texttt{binary\_crossentropy}
\end{aligned}
$$

### 41.10 Numbers worth remembering

$$
\begin{aligned}
&\quad \sigma(0)=0.5 \quad \sigma(1)=0.731 \quad \sigma(4)=0.982 \quad \tanh(1)=0.762 \quad \tanh(0.5)=0.462 \\
&\quad 0.25^{10} \approx 9.5\times 10^{-7} \qquad 0.5^{10} = 1/1024
\end{aligned}
$$

$$
\begin{aligned}
&\quad \text{LSTM params} \approx 4\times\text{RNN} \qquad \text{depthwise} \approx 8\text{-}9\times\text{cheaper} \\
&\quad \text{ImageNet images } 14\text{M} \qquad \text{AlexNet } 2012 \qquad \text{Transformer } 2017
\end{aligned}
$$

---

## 42. Self-test: 100 questions with answers

**Foundations**

1. **Define deep learning in one line.** A neural network with multiple hidden layers that learns a hierarchy of representations from data.
2. **Why does ML plateau while DL keeps improving?** ML has limited capacity/hypothesis space; DL has millions of parameters and is data-limited, so extra data yields extra accuracy.
3. **State Mitchell's definition of learning.** A program learns from experience E w.r.t. task T and performance P if P on T improves with E.
4. **What is representation learning?** Learning features automatically instead of hand-engineering them.
5. **Biological vs artificial: dendrite, synapse, soma, axon?** Input, weight, summation+activation, output.
6. **Why can't the MP neuron learn?** No learning rule — weights/threshold are fixed by hand.
7. **Write the perceptron learning rule.** $w \leftarrow w + \alpha(t-o)x$.
8. **What is the decision boundary of a 3-input perceptron?** A plane $w\cdot x + b = 0$.
9. **Prove XOR is not linearly separable.** Four inequalities → contradiction (see §2.4).
10. **How many hidden neurons are needed for XOR and why?** Two — XOR is the OR of two linearly separable sub-problems.

**MLP and training**

11. **Why is a non-linearity necessary?** Without it, stacked linear layers collapse to one linear map.
12. **Count parameters for 3-4-4-2.** $12+16+8 = 36$ weights, $4+4+2 = 10$ biases → **46**.
13. **Why not initialise all weights to zero?** Symmetry: all neurons compute and learn the same thing.
14. **Write the MSE and BCE formulas.** See §4.
15. **Why cross-entropy instead of MSE for classification?** It is the NLL; it gives gradient $\hat{y}-y$, no sigmoid saturation in the gradient.
16. **What is the gradient of softmax+CE?** $\hat{y} - y$.
17. **Why is softmax shift-invariant?** The $e^c$ cancels in numerator and denominator.
18. **State the GD update and the role of α.** $\theta \leftarrow \theta - \eta\partial L/\partial\theta$; $\alpha$ is step size — too small slow, too large divergent.
19. **Batch vs SGD vs mini-batch?** Full data / one sample / a small batch per update.
20. **Why is mini-batch gradient unbiased?** $E[g_B] = \nabla J$ for uniformly sampled batches.
21. **What does an epoch mean?** One full pass over the training set.
22. **Derive $\sigma' = \sigma(1-\sigma)$.** See §6.3.
23. **Write the backprop δ recursion.** $\delta^{(l)} = (W^{(l+1)\top}\delta^{(l+1)}) \odot g'(z^{(l)})$.
24. **Gradient w.r.t. a weight in one line.** Error at the receiving node × activation of the sending node.
25. **Why do hidden gradients shrink in the worked example?** Sigmoid saturation: $\sigma'(4) = 0.018$.
26. **What is gradient checking?** Compare analytic gradients with finite differences $(J(\theta+\epsilon)-J(\theta-\epsilon))/2\epsilon$; relative error $< 10^{-7}$ is good.

**Regularisation and optimisation**

27. **Define overfitting and underfitting.** Low train/high test error vs high train/test error.
28. **Bias–variance decomposition?** $\text{bias}^2 + \text{variance} + \text{irreducible noise}$.
29. **Why does L1 produce zeros?** The constraint diamond is touched at a corner (an axis).
30. **Why is L2 called weight decay?** The update multiplies $w$ by $(1-2\eta\lambda)$ each step.
31. **How does dropout rescale at train time?** $h/(1-p)$ (inverted dropout), so $E[h'] = h$.
32. **What is early stopping?** Stop when validation loss stops improving; restore the best weights.
33. **Write the Adam update.** See §9.3.
34. **Why does Adam need bias correction?** $m$ and $v$ start at 0, so early moments are biased toward 0.
35. **Xavier vs He init?** $2/(n_{\text{in}}+n_{\text{out}})$ for sigmoid/tanh; $2/n_{\text{in}}$ for ReLU.
36. **Name three causes of exploding gradients.** Large weights, deep/unrolled recurrence, high LR; fix with clipping.
37. **What does batch norm normalise?** Each feature per mini-batch to mean 0/var 1, then scale/shift.
38. **Which optimiser is the default and why?** Adam — adaptive per-parameter rates, fast, robust to hyperparameters.

**CNN**

39. **Why is a CNN better than a dense net on images?** Local connectivity, weight sharing, hierarchy → far fewer parameters and translation equivariance.
40. **Output size: $n=32$, $k=5$, $p=0$, $s=2$?** $\lfloor (32-5)/2 \rfloor + 1 = 14$.
41. **Conv params: 3×3, $C_{\text{in}}=3$, $C_{\text{out}}=64$?** $(27+1)\cdot 64 = 1792$.
42. **Standard vs depthwise multiplications (12×12×3, 256, 3×3)?** 691,200 vs 79,500.
43. **Depthwise reduction ratio?** $1/C_{\text{out}} + 1/K^2$.
44. **Max vs average pooling?** Strongest response vs average evidence.
45. **What is a receptive field and how does it grow?** The input region affecting one unit; $r_l = r_{(l-1)} + (k-1)\cdot\text{jump}$.
46. **Conv vs cross-correlation?** Cross-correlation does not flip the kernel; CNNs implement it but call it convolution.
47. **Why is GAP better than flatten+dense?** No weights, fewer parameters, any input size, less overfitting.
48. **What does a ResNet skip connection do to the gradient?** $\partial y/\partial x = 1 + \partial F/\partial x$ — an identity highway that prevents vanishing.

**Sequences**

49. **Why can't a dense network handle sequences?** Variable length, order matters, no weight sharing across time.
50. **Write the RNN forward equations and parameter count.** See §16.3.
51. **What is BPTT?** Backprop on the unrolled graph over time, summing shared-weight gradients.
52. **Why do RNN gradients vanish?** The gradient is a product of $t-k$ Jacobians, each with norm < 1.
53. **Two fixes for vanishing and one for exploding.** LSTM/GRU + good init; clipping for exploding.
54. **Write all six LSTM equations.** See §18.2.
55. **Why does LSTM avoid vanishing?** $\partial c_t/\partial c_{t-1} = f_t$; with $f\approx 1$ the gradient passes unchanged.
56. **LSTM parameter count formula?** $4[(h+d)h + h]$.
57. **Why is Bi-LSTM not real-time?** It needs the future tokens for the backward pass.
58. **Name the five seq2seq patterns.** One-to-one, one-to-many, many-to-one, many-to-many aligned/unaligned.
59. **What is the context vector's weakness?** A single fixed vector cannot summarise long inputs.
60. **Write the attention equations.** $\alpha_{ij} = \operatorname{softmax} f(h_{j-1}, s_{i-1})$, $C_i = \sum \alpha_{ij} h_{j-1}$.

**Latent models**

61. **Five PCA steps?** Centre/standardise, covariance, eigen, sort+select, project.
62. **What does an eigenvalue mean?** The variance along its eigenvector.
63. **PPCA model and marginal?** $x = \mu+Wz+\epsilon$; $x \sim \mathcal{N}(\mu, WW^\top+\sigma^2 I)$.
64. **PPCA vs PCA?** Probabilistic/noise/likelihood/missing data vs deterministic projection.
65. **PPCA vs FA?** Isotropic $\sigma^2 I$ vs diagonal $\Psi$.
66. **ICA assumptions?** Independent, non-Gaussian sources, linear mixing, enough mixtures.
67. **Why non-Gaussian?** CLT makes mixtures more Gaussian; maximum non-Gaussianity recovers the sources.
68. **ICA ambiguities?** Permutation, scale, sign.

**Autoencoders and generative**

69. **Define an autoencoder.** A network trained to reconstruct its input through a bottleneck.
70. **Undercomplete vs overcomplete?** $d<n$ compresses; $d\geq n$ can learn the identity.
71. **What is the identity-mapping problem?** A high-capacity AE reconstructs perfectly while learning nothing useful.
72. **Four AE variants and their penalties?** Sparse (L1/KL), denoising (input noise), contractive (Jacobian), VAE (KL).
73. **How does an AE detect anomalies?** Reconstruction error above a threshold.
74. **Reparameterisation trick?** $z = \mu + \sigma\odot\epsilon$, $\epsilon \sim \mathcal{N}(0,I)$ — differentiable sampling.
75. **Write the VAE loss.** $-E[\log p(x \mid z)] + \mathrm{KL}(q(z \mid x) \Vert p(z))$.
76. **Closed-form KL for Gaussians?** $\frac{1}{2}\sum(\mu^2+\sigma^2-1-\log\sigma^2)$.
77. **Why does a VAE produce blurry images?** Gaussian/Bernoulli likelihood averages over uncertainty.
78. **Write the GAN minimax objective.** See §32.2.
79. **Why is mode collapse bad if samples look good?** Diversity is lost; whole classes are missing.
80. **What is a conditional GAN?** Condition $c$ fed to both G and D for controllable generation.

**Transfer, transformers, practical**

81. **What is transfer learning?** Reusing knowledge from a source task on a related target task.
82. **Feature extraction vs fine-tuning?** Freeze everything but the head vs unfreeze top layers with a small LR.
83. **What is negative transfer?** Transfer hurts performance when domains/tasks are too different.
84. **Define a domain.** $D = \{X, P(X)\}$ — feature space and data distribution.
85. **DA vs TL?** Same task, different distribution; DA ⊂ TL.
86. **Name four DA techniques.** Fine-tuning, feature alignment, adversarial DA, data transformation.
87. **Write scaled dot-product attention.** $\operatorname{softmax}(QK^\top/\sqrt{d_k})V$.
88. **Why $\sqrt{d_k}$?** Dot-product variance grows with $d_k$; scaling prevents softmax saturation.
89. **Why positional encoding?** Attention is permutation-invariant — it has no notion of order.
90. **Encoder vs decoder block?** Bidirectional self-attention vs masked self-attention + cross-attention.
91. **Five impacts of transformers?** NLP revolution, democratisation, multimodality, GenAI acceleration, unification.
92. **Why is a transformer faster to train than an RNN?** All positions are processed in parallel; no sequential dependency.
93. **Temperature in sampling?** $\operatorname{softmax}(z/T)$; low T sharpens, high T flattens.
94. **Top-k vs top-p?** Fixed count vs smallest cumulative-probability set (adaptive).
95. **What is DeepDream?** Gradient ascent on the input to maximise a layer's activations.
96. **What does the Gram matrix capture?** Channel correlations → texture/style.
97. **Write the style-transfer total loss.** $\alpha\cdot\text{content} + \beta\cdot\text{style}$.
98. **Which loss for integer labels in Keras?** `sparse_categorical_crossentropy`.
99. **Why GAP before the classifier?** No weights, fewer parameters, input-size agnostic.
100. **Name three GAN failure modes.** Mode collapse, instability, vanishing generator gradient.

---

## 43. Trap list — the mistakes that cost marks

### 43.1 Conceptual traps

| Trap | Correct |
|---|---|
| "Convolution flips the kernel" | CNN convolution is **cross-correlation** (no flip); the term is conventional |
| "Conv output shrinks even with same padding" | $p=(k-1)/2$ keeps it identical |
| "Pooling has learnable weights" | No parameters; only conv/dense do |
| "Depthwise separable mixes channels in both stages" | Only the **pointwise** 1×1 stage mixes channels |
| "LSTM has one more input than RNN for no reason" | $c_{t-1}$ is the long-term memory input |
| "LSTM's hidden state is the long-term memory" | $c_t$ is long-term; $h_t$ is the exposed short-term state |
| "GRU has three gates" | Two: update and reset |
| "Vanishing gradients happen only in RNNs" | Also in deep sigmoid/tanh MLPs; recurrence just makes it worse |
| "Clipping fixes vanishing gradients" | Clipping fixes **exploding** only |
| "A VAE is just an AE with noise" | The KL term is what makes the latent space smooth and samplable |
| "GANs maximise likelihood" | GANs are **implicit** density models — no likelihood at all |
| "Mode-collapsed GANs produce blurry images" | They produce **sharp** but **few** distinct images |
| "PPCA assumes per-feature noise" | PPCA is isotropic $\sigma^2 I$; **FA** has diagonal $\Psi$ |
| "ICA finds uncorrelated components" | ICA finds **independent** components; uncorrelated ≠ independent |
| "Transfer learning always helps" | Negative transfer exists |
| "Domain adaptation changes the task" | Same task, different $P(x)$ |
| "Attention is O(T) in sequence length" | It is **$O(T^2)$** |
| "Transformers don't need position because they see everything" | Without positional encoding they are permutation-invariant |
| "Softmax + binary_crossentropy is fine" | Use `categorical_crossentropy` (or `sparse_`) |
| "Dropout is active at test time" | Disabled at inference; only train-time |

### 43.2 Numerical traps

| Trap | Fix |
|---|---|
| Forgetting $+1$ bias in conv parameter counts | $(K^2C_{\text{in}}+1)C_{\text{out}}$ |
| Counting only weights in MLP/RNN params | Add the biases |
| Using $n-k+1$ with padding/stride | Use $\lfloor (n+2p-k)/s \rfloor + 1$ |
| Rounding output size up | `floor`, always |
| Using $C_{\text{in}}$ instead of $C_{\text{out}}$ in the standard conv cost | $H'W'C_{\text{out}}K^2C_{\text{in}}$ |
| Forgetting to sum over input channels | The $\sum_c$ in the convolution |
| Missing the $1/\sqrt{d_k}$ | It changes the softmax completely |
| Using natural log vs log base 2 in entropy | Both OK in this course — state the base |
| Sign of the gradient update | $\theta - \eta\nabla L$, never $+$ |
| Storing $\sigma$ instead of $\log\sigma^2$ in a VAE | Encoder outputs $\log\sigma^2$ for positivity/stability |
| Forgetting that $J_G$ and $J_D$ are averaged over the batch | Include the $1/m$ |
| Using the same $z$ in both GAN steps | Resample noise for the generator step |

### 43.3 Corrections to the source material (from the mid-sem guide)

| Source says | Correct version |
|---|---|
| $\tanh(z) = (1-e^{-z})/(1+e^{-z})$ | $\tanh(z) = (1-e^{-2z})/(1+e^{-2z}) = 2\sigma(2z)-1$ |
| Lasso uses an L2 penalty | Lasso = **L1**, Ridge = L2 |
| R² sums divided by N | The N cancels: $1 - \sum(y-\hat{y})^2/\sum(y-\bar{y})^2$ |
| F1 example = 75.32% | Exactly **0.75** (rounding artefact) |
| MLP "20 parameters" | 20 weights + 6 biases = **26** |
| RNN "total 44" | 44 weights + 5 biases = **49** |
| Conv feature map "all −6" | Only the top-left is −6; the rest are $12, 12, -6, 8, 8, -6, 2, 12$ |
| Sigmoid range $(-1,1)$ | $(0,1)$ |
| SELU constants 1.05/1.67 | $\lambda\approx 1.0507$, $\alpha\approx 1.6733$ |
| Dying ReLU caused by non-differentiability at 0 | Caused by **zero gradient for negative inputs** |
| Larger α → more epochs | Larger $\alpha$ → fewer, bigger steps (may overshoot) |
| AlexNet "first DL model, 2013" | First **ImageNet winner**, 2012 (LeNet 1998 was the first CNN) |
| "6 decoders" in the Transformer | 6 **encoders and** 6 decoders |
| Attention $h_i$ subscript | $h_{j-1}$ for encoder states; $h_i$ is a hidden-state *vector* of size hidden units |

---

## 44. Last three hours

**Hour 1 — write, don't read.**
Write the full formula sheet (§41) from memory on two A4 pages. Compare, mark misses in red. Re-write only the red items.

**Hour 2 — draw.**
All 7 diagrams from §40.4, twice each, labelled. Then the two "money" diagrams: **LSTM cell** and **transformer block**.

**Hour 3 — answer.**
Do these ten, in order, on paper:

1. Derive backprop for 2-2-1 and do one iteration.
2. Train the XOR MLP (2 hidden neurons).
3. Forward-propagate the RNN for 2 steps.
4. Write and interpret the LSTM equations; compute $c_t, h_t$.
5. Compute conv output size and parameter count for a 3-layer CNN.
6. Standard vs depthwise multiplication count.
7. Derive $\alpha_{ij}$ and $C_{ij}$ for attention.
8. Write the VAE loss and the KL formula.
9. Write the GAN minimax, $J_G$, $J_D$, and explain mode collapse.
10. Differentiate PCA/PPCA/FA/ICA in a table.

**Final 15 minutes.** Read only: the trap list (§43), the mnemonics table (§40.2), and the numbers section (§41.10). Do not learn anything new.

**In the exam.**
1. Read all questions first; start with the highest-marks numerical you know cold.
2. Draw the diagram **before** writing prose — diagrams earn marks and organise your answer.
3. For every numerical, box the formula, substitute, box the answer, then add the one-line interpretation.
4. If you blank on a derivation, write the chain-rule structure and the definitions — partial marks are real.
5. Leave 5 minutes to check units, signs, and whether you added the biases.

---

*End of guide. If you can write §41 from a blank page and draw the seven diagrams without notes, you are ready.*
