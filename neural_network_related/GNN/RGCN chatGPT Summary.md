- [RGCN Summary](#rgcn-summary)
  - [1. What problem does R-GCN solve?](#1-what-problem-does-r-gcn-solve)
  - [2. The core intuition](#2-the-core-intuition)
  - [3. What exactly is a relational graph?](#3-what-exactly-is-a-relational-graph)
  - [4. R-GCN layer](#4-r-gcn-layer)
  - [5. Why relation-specific transformations matter](#5-why-relation-specific-transformations-matter)
  - [6. Message-passing interpretation](#6-message-passing-interpretation)
  - [7. Matrix formulation](#7-matrix-formulation)
  - [8. The major problem: too many parameters](#8-the-major-problem-too-many-parameters)
  - [9. Basis decomposition](#9-basis-decomposition)
  - [10. Block-diagonal decomposition](#10-block-diagonal-decomposition)
  - [11. Directionality](#11-directionality)
  - [12. Why self-loops are important](#12-why-self-loops-are-important)
  - [13. Multi-layer R-GCN](#13-multi-layer-r-gcn)
  - [14. What does an R-GCN actually learn?](#14-what-does-an-r-gcn-actually-learn)
  - [15. R-GCN for node classification](#15-r-gcn-for-node-classification)
  - [16. R-GCN for link prediction](#16-r-gcn-for-link-prediction)
  - [17. Encoder vs decoder](#17-encoder-vs-decoder)
  - [18. Negative sampling](#18-negative-sampling)
  - [19. R-GCN vs ordinary GCN](#19-r-gcn-vs-ordinary-gcn)
    - [GCN](#gcn)
    - [R-GCN](#r-gcn)
  - [20. R-GCN vs heterogeneous GNN](#20-r-gcn-vs-heterogeneous-gnn)
  - [21. The complete R-GCN pipeline](#21-the-complete-r-gcn-pipeline)
  - [22. A small numerical intuition](#22-a-small-numerical-intuition)
  - [23. What does “heterogeneous” really contribute?](#23-what-does-heterogeneous-really-contribute)
  - [24. Important limitation: over-smoothing](#24-important-limitation-over-smoothing)
  - [25. Important limitation: over-squashing](#25-important-limitation-over-squashing)
  - [26. Relation imbalance](#26-relation-imbalance)
  - [27. Why basis decomposition is conceptually interesting](#27-why-basis-decomposition-is-conceptually-interesting)
  - [28. R-GCN's inductive bias](#28-r-gcns-inductive-bias)
  - [29. Computational complexity](#29-computational-complexity)
  - [30. The most important equations to remember](#30-the-most-important-equations-to-remember)
  - [31. A useful mental model](#31-a-useful-mental-model)
  - [32. R-GCN in one sentence](#32-r-gcn-in-one-sentence)
  - [33. Big-picture comparison](#33-big-picture-comparison)
  - [34. Final conceptual picture](#34-final-conceptual-picture)


# RGCN Summary

Absolutely. Below is a theory-first summary of **Relational Graph Convolutional Networks (R-GCNs)** for heterogeneous/multi-relational graphs, including the intuition, mathematical formulation, architecture, parameterization, normalization, training, and common extensions.

## 1. What problem does R-GCN solve?

A standard Graph Convolutional Network (GCN) assumes that edges are essentially the same type:

> “Node \(u\) is connected to node \(v\).”

But many real-world graphs are **heterogeneous or multi-relational**: the meaning of an edge depends on its relation type.

For example, in a knowledge graph:

* `Alice --works_at--> Company`
* `Alice --lives_in--> City`
* `Alice --friend_of--> Bob`
* `Company --located_in--> City`

The crucial observation is that messages arriving through different relations should generally be processed **differently**.

A normal GCN might aggregate:


$$\mathbf h_v^{(l+1)}=\sigma\left(\sum_{u\in\mathcal N(v)}W^{(l)}\mathbf h_u^{(l)}\right)$$ 

R-GCN instead asks:

> **What type of relationship connects \(u\) and \(v\)?**

and uses a relation-specific transformation:

$$\boxed{\mathbf h_v^{(l+1)}=\sigma\left(W_0^{(l)}\mathbf h_v^{(l)}+\sum_{r\in\mathcal R}\sum_{u\in\mathcal N_v^r}\frac{1}{c_{v,r}}W_r^{(l)}\mathbf h_u^{(l)}\right)}$$

This is the fundamental R-GCN equation.

---

## 2. The core intuition

Imagine a node representing **Alice**.

Suppose Alice has:

```text
Bob ──friend_of──► Alice
Alice ──works_at──► Company
Alice ──lives_in──► Jakarta
```

The information Alice should receive from Bob is different from the information she should receive from a company or city.

Conceptually:

```text
                  relation = friend_of
              ┌─────────────────────────┐
              │                         ▼
           [Bob] ─────── message ───► [Alice]
                                      ▲
                                      │
              relation = works_at     │
          [Company] ─── message ──────┤
                                      │
              relation = lives_in     │
          [Jakarta] ─── message ──────┘
```

R-GCN gives each relation \(r\) its own transformation matrix:

$$
W_{\text{friend}}
$$

$$
W_{\text{works\_at}}
$$

$$
W_{\text{lives\_in}}
$$

so that:

$$\text{message}_{u\rightarrow v}^{(r)}=W_r h_u$$

The model therefore learns:

> “Information coming through a `friend_of` edge should be interpreted differently from information coming through a `works_at` edge.”

That is the key idea behind R-GCN.

---

## 3. What exactly is a relational graph?

A relational graph can be represented as:

$$
G=(V,\mathcal E,\mathcal R)
$$

where:

* \(V\) = set of nodes
* \(\mathcal E\) = set of edges
* \(\mathcal R\) = set of relation types

An edge can be represented as:

$$
(u,r,v)
$$

meaning:

> node \(u\) is connected to node \(v\) through relation \(r\).

For example:

$$
(\text{Alice},\text{works\_at},\text{Google})
$$

or:

$$
(\text{Alice},\text{friend\_of},\text{Bob})
$$

For each relation \(r\), define:

$$\mathcal E_r=\{(u,v):(u,r,v)\in\mathcal E\}$$

and the relation-specific neighborhood:

$$\mathcal N_v^r=\{u:(u,r,v)\in\mathcal E_r\}$$

So instead of having just one neighborhood:

$$\mathcal N_v$$

we have multiple neighborhoods:

$$\mathcal N_v^{r_1},\mathcal N_v^{r_2},\dots,\mathcal N_v^{r_R}.$$

---

## 4. R-GCN layer

The canonical R-GCN layer is:

$$\boxed{h_v^{(l+1)}=\sigma\left(W_0^{(l)}h_v^{(l)}+\sum_{r\in\mathcal R}\sum_{u\in\mathcal N_v^r}\frac{1}{c_{v,r}}W_r^{(l)}h_u^{(l)}\right)}$$

Let's break this equation apart.

**$h_v^{(l)}$**

Representation of node $v$ at layer $l$.

Initially:

$$h_v^{(0)}=x_v$$

where $x_v$ is the node's input feature.

After one layer:

$$
h_v^{(1)}
$$

contains information from its immediate relational neighborhood.

After two layers:

$$
h_v^{(2)}
$$

can contain information from approximately two-hop neighborhoods.

---

**$W_r^{(l)}$**

This is the **relation-specific weight matrix**.

Each relation gets its own transformation:

$$
W_{r_1},W_{r_2},...,W_R
$$

For example:

$$
W_{\text{friend}}
$$

could learn how to transform information associated with friendship, while:

$$
W_{\text{works\_at}}
$$

learns a completely different transformation.

This is what distinguishes R-GCN from an ordinary GCN.

---

**$W_0^{(l)}$**

This is the **self-loop transformation**.

It allows the node to preserve/use its own information:

$$
W_0h_v
$$

Without this term, the new representation would be based exclusively on neighboring nodes.

---

**$c_{v,r}$**

This is a normalization constant.

A common choice is:

$$
c_{v,r}=|\mathcal N_v^r|
$$

so the relation-specific aggregation becomes an average:

$$
\frac{1}{|\mathcal N_v^r|}
\sum_{u\in\mathcal N_v^r}
W_rh_u
$$

This prevents a node with many neighbors from producing disproportionately large messages.

---

**$\sigma$**

An activation function such as:

$$
\text{ReLU}(x)=\max(0,x)
$$

introduces non-linearity.

---

## 5. Why relation-specific transformations matter

Consider:

```text
          friend_of
Bob ─────────────────► Alice

          works_at
Alice ────────────────► Company
```

Suppose Bob's feature vector is:

$$h_{\text{Bob}}=\begin{bmatrix}\text{age}\\\text{occupation}\\\text{location}\end{bmatrix}$$

If Bob is Alice's **friend**, perhaps his age and interests provide useful information.

If another node is Alice's **employer**, its features have completely different semantics.

A single transformation:

$$
Wh_u
$$

would treat both relations identically.

R-GCN instead computes:

$$
W_{\text{friend}}h_{\text{Bob}}
$$

and:

$$
W_{\text{employer}}h_{\text{Company}}
$$

allowing the network to learn different semantic mappings.

---

## 6. Message-passing interpretation

R-GCN is easier to understand using the **message-passing framework**.

Every layer consists conceptually of:

- Step 1 — Message construction

For an edge:

$$u\xrightarrow{r}v$$

construct:

$$m_{u\rightarrow v}^{(r)}=W_rh_u$$

- Step 2 — Aggregation

Collect messages from all neighbors:

$$M_v=\sum_r\sum_{u\in\mathcal N_v^r}\frac{1}{c_{v,r}}m_{u\rightarrow v}^{(r)}$$

- Step 3 — Combine with self-information

$$z_v=W_0h_v+M_v$$

- Step 4 — Nonlinear transformation

$$h_v'=\sigma(z_v)$$

So conceptually:

```text
Neighbor features
      │
      ▼
┌───────────────────┐
│ Relation-specific │
│ transformations   │
│                   │
│ W₁, W₂, ..., W_R  │
└─────────┬─────────┘
          │
          ▼
     Aggregation
          │
          ▼
     + self-loop
          │
          ▼
      Activation
          │
          ▼
    New node embedding
```

---

## 7. Matrix formulation

The layer can also be written using adjacency matrices.

For relation $r$, define:

$$A_r\in\mathbb R^{N\times N}$$

where:

$$(A_r)_{vu}=1$$

if $u$ is connected to $v$ by relation $r$.

Then:

$$H^{(l+1)}=\sigma\left(\sum_{r\in\mathcal R}D_r^{-1}A_rH^{(l)}W_r^{(l)}+H^{(l)}W_0^{(l)}\right)$$

where:

$$D_r(v,v)=|\mathcal N_v^r|.$$

This formulation is useful for understanding the relationship between R-GCN and conventional GCNs.

A normal GCN effectively operates with one adjacency structure, whereas R-GCN decomposes the graph into **relation-specific adjacency matrices**:

$$
A
\rightarrow
\{A_{r_1},A_{r_2},...,A_{r_R}\}.
$$

---

## 8. The major problem: too many parameters

Suppose:

* hidden dimension = \(d\)
* number of relations = \(R\)

Each relation requires:

$$
W_r\in\mathbb R^{d\times d}
$$

Therefore, the number of parameters is approximately:

$$
R d^2.
$$

For example:

$$
R=100,\qquad d=256
$$

gives:

$$100\times256^2=6,553,600$$

parameters **for one R-GCN layer**, before considering other parameters.

For knowledge graphs containing hundreds or thousands of relation types, this becomes problematic.

This motivates one of the most important ideas in R-GCN: **Basis decomposition**.

---

## 9. Basis decomposition

Instead of learning an independent \(W_r\) for every relation, R-GCN can represent each relation matrix as a linear combination of a small number of basis matrices.

$$\boxed{W_r=\sum_{b=1}^{B}a_{rb}V_b}$$

where:

* \(V_b\) = shared basis matrices
* \(a_{rb}\) = relation-specific coefficients
* \(B\ll R\)

For example:

$$W_{\text{friend}}=0.7V_1+0.2V_2-0.1V_3$$

while:

$$W_{\text{works}}=0.1V_1+0.8V_2+0.3V_3.$$

Thus relations share fundamental transformations but combine them differently.

**Intuition**

Think of \(V_1,V_2,V_3\) as reusable “semantic building blocks.”

Each relation says:

> “For my messages, use 70% of basis 1, 20% of basis 2, and −10% of basis 3.”

This dramatically reduces the number of parameters.

Instead of:

$$
R d^2
$$

parameters, approximately:

$$
Bd^2+RB
$$

are required.

---

## 10. Block-diagonal decomposition

Another parameter-reduction approach is **block-diagonal decomposition**.

Split the feature dimension into blocks:

$$
d=d_1+d_2+\cdots+d_B.
$$

Then construct:

$$
W_r=
\begin{bmatrix}
Q_{r1} & 0 & \cdots & 0\\
0 & Q_{r2} & \cdots & 0\\
\vdots & & \ddots & \vdots\\
0 & 0 & \cdots & Q_{rB}
\end{bmatrix}.
$$

Only parameters inside the diagonal blocks are learned.

This reduces computation and parameter count while still allowing relation-specific transformations.

---

## 11. Directionality

Relations in many graphs are directed.

For example:

$$
\text{Alice}
\xrightarrow{\text{works\_at}}
\text{Company}
$$

is not necessarily equivalent to:

$$
\text{Company}
\xrightarrow{\text{works\_at}}
\text{Alice}.
$$

Therefore, R-GCN commonly considers **inverse relations**.

For every relation:

$$
r
$$

one can introduce:

$$
r^{-1}.
$$

For example:

```text
Alice ──works_at──► Google

Google ──works_at⁻¹──► Alice
```

Then information can propagate in both directions while retaining the direction semantics.

This is particularly important for knowledge graphs.

---

## 12. Why self-loops are important

Consider:

```text
Bob ──friend──► Alice
Charlie ──friend──► Alice
```

If Alice updates her representation purely from neighbors:

$$h_A'=W_{\text{friend}}h_B+W_{\text{friend}}h_C$$

her previous information could be lost.

The self-loop gives:

$$h_A'=W_0h_A+W_{\text{friend}}h_B+W_{\text{friend}}h_C.$$

Thus the new representation combines:

$$
\boxed{\text{own information}+\text{neighbor information}}
$$

---

## 13. Multi-layer R-GCN

Suppose we have:

```text
A ──r₁──► B ──r₂──► C
```

With one R-GCN layer, C can directly incorporate information from B.

With two layers:

$$h_C^{(2)}$$

can incorporate information from:

$$A\rightarrow B\rightarrow C.$$

Therefore:

* 1 layer → roughly 1-hop information
* 2 layers → roughly 2-hop information
* \(L\) layers → roughly \(L\)-hop information

Mathematically:

$$H^{(l+1)}=f(H^{(l)},A_{r_1},...,A_{r_R}).$$

Repeated application creates increasingly contextual node embeddings.

---

## 14. What does an R-GCN actually learn?

An important conceptual point is that R-GCN does **not** explicitly learn a symbolic rule such as:

> “If someone works for a company, they probably live in that company's city.”

Instead, it learns continuous representations.

For node \(v\):

$$
h_v^{(l)}
$$

becomes a learned representation containing information accumulated through different relation types.

The model may consequently learn useful patterns such as:

$$
\text{person}
\xrightarrow{\text{works\_at}}
\text{company}
\xrightarrow{\text{located\_in}}
\text{city}.
$$

After enough message-passing layers, the person's embedding can contain information influenced by the city.

This is one reason R-GCN is useful for **knowledge graph reasoning and link prediction**.

---

## 15. R-GCN for node classification

Suppose every node has a label:

$$y_v\in\{1,\ldots,C\}.$$

After the final R-GCN layer:

$$h_v^{(L)}$$

is passed to a classifier:

$$z_v=W_ch_v^{(L)}$$

and:

$$p(y_v=c\mid v)=\operatorname{softmax}(z_v)_c.$$

The training objective is commonly cross-entropy:

$$\mathcal L=-\sum_{v\in V_{\text{train}}}\log p(y_v\mid v).$$

Thus R-GCN can learn node representations that are useful for classification.

---

## 16. R-GCN for link prediction

This is particularly important in knowledge graphs.

Suppose we want to predict whether:

$$(u,r,v)$$

is a valid triple.

R-GCN first produces:

$$h_u,\quad h_v.$$

Then a decoder scores the triple:

$$f(u,r,v).$$

One popular choice is a DistMult-style score:

$$f(u,r,v)=h_u^\top R_rh_v.$$

A higher score means the model believes the relation is more plausible.

The overall architecture becomes:

```text
                 Heterogeneous graph
                         │
                         ▼
                 ┌──────────────┐
                 │    R-GCN     │
                 │   Encoder    │
                 └──────┬───────┘
                        │
                 Node embeddings
                    h_u, h_v
                        │
                        ▼
                 ┌──────────────┐
                 │    Decoder   │
                 │ f(u,r,v)     │
                 └──────┬───────┘
                        │
                        ▼
                 Link prediction
```

---

## 17. Encoder vs decoder

This distinction is extremely important.

In knowledge graph applications:

**R-GCN = encoder**

It converts the graph into embeddings:

$$
G\rightarrow H.
$$

**Decoder = scoring function**

It evaluates whether a relationship is plausible:

$$
(h_u,r,h_v)\rightarrow score.
$$

Therefore:

$$
\boxed{
\text{R-GCN encoder}+\text{link-prediction decoder}
}
$$

is a common architecture.

The R-GCN itself does not necessarily perform the final link prediction.

---

## 18. Negative sampling

Knowledge graphs often contain only observed positive triples.

Suppose:

$$
(Alice,\text{works\_at},Google)
$$

is observed.

We can generate a negative example by corrupting the head or tail:

$$
(Bob,\text{works\_at},Google)
$$

or:

$$
(Alice,\text{works\_at},Microsoft).
$$

Then the model learns:

$$
score(\text{positive})
>
score(\text{negative}).
$$

A common loss is logistic loss:

$$\mathcal L=-\log\sigma(f(u,r,v))-\log\sigma(-f(u',r,v')).$$

Margin-based losses can also be used.

---

## 19. R-GCN vs ordinary GCN

The difference can be summarized elegantly.

### GCN

$$h_v'=\sigma\left(W_0h_v+\sum_{u\in N(v)}W h_u\right)$$

All neighbors are transformed using essentially the same $W$.

### R-GCN

$$h_v'=\sigma\left(W_0h_v+\sum_r\sum_{u\in N_v^r}W_rh_u\right)$$

Neighbors connected by different relations use different $W_r$.

So:

$$\boxed{\text{GCN: one transformation}}$$

$$\boxed{\text{R-GCN: relation-dependent transformations}}$$

---

## 20. R-GCN vs heterogeneous GNN

There is an important terminology distinction.

A **heterogeneous graph** can contain different:

* node types
* edge/relation types
* feature spaces

For example:

```text
Person ──works_at──► Company
Person ──enrolled_in──► University
Company ──located_in──► City
```

R-GCN is fundamentally designed for **multi-relational graphs**, where edge/relation types matter.

However, the original R-GCN formulation is most naturally viewed as a relational GNN over relation types. Modern heterogeneous GNNs may additionally use **node-type-specific transformations**, attention mechanisms, meta-paths, type-specific aggregators, etc.

For example:

$$
W_{\text{Person}\rightarrow\text{Company}}
$$

could explicitly account for source and destination node types.

Therefore, don't assume:

$$
\text{R-GCN}=\text{every possible heterogeneous GNN}.
$$

Rather:

$$
\boxed{\text{R-GCN is one important relational message-passing architecture for heterogeneous/multi-relational graphs.}}
$$

---

## 21. The complete R-GCN pipeline

Putting everything together:

```text
                 Heterogeneous / relational graph
                              │
             ┌────────────────┼────────────────┐
             │                │                │
             ▼                ▼                ▼
          relation 1       relation 2       relation 3
          A₁               A₂               A₃
             │                │                │
             ▼                ▼                ▼
          W₁H             W₂H              W₃H
             │                │                │
             └────────────────┼────────────────┘
                              ▼
                       Relation-wise
                        aggregation
                              │
                              ▼
                        Normalization
                              │
                              +
                              │
                         W₀H (self)
                              │
                              ▼
                         Activation
                              │
                              ▼
                       H^(l+1)
                              │
                         next layer
                              ▼
                       H^(l+2) ...
                              │
                              ▼
                    Final node embeddings
                              │
                    ┌─────────┴─────────┐
                    ▼                   ▼
               Classification      Link prediction
```

---

## 22. A small numerical intuition

Suppose Alice has two neighbors:

$$Bob\xrightarrow{friend}Alice$$

and:

$$Company\xrightarrow{employer}Alice.$$

Assume:

$$h_{Bob}=\begin{bmatrix}1\\2\end{bmatrix},\qquad h_{Company}=\begin{bmatrix}3\\1\end{bmatrix}.
$$

Suppose:

$$W_{friend}=\begin{bmatrix}1&0\\0&2\end{bmatrix}$$

and:

$$W_{employer}=\begin{bmatrix}0&1\\1&0\end{bmatrix}.$$

Then:

$$W_{friend}h_{Bob}=\begin{bmatrix}1\\4\end{bmatrix}$$

while:

$$W_{employer}h_{Company}=\begin{bmatrix}1\\3\end{bmatrix}.$$

The two messages are transformed differently because their **semantics differ**.

R-GCN then aggregates these messages:

$$m_{Alice}=\begin{bmatrix}1\\4\end{bmatrix}+\begin{bmatrix}1\\3\end{bmatrix}=\begin{bmatrix}2\\7\end{bmatrix}.$$

The self-loop contribution is then added before activation.

The exact numbers aren't important—the important concept is:

$$
\boxed{
\text{neighbor feature}
\xrightarrow{\text{relation-specific }W_r}
\text{relation-aware message}
}
$$

---

## 23. What does “heterogeneous” really contribute?

Consider two graphs.

**Homogeneous graph**

```text
A ── B
│    │
C ── D
```

There is essentially one edge semantics.

**Relational graph**

```text
A ──friend──► B
A ──works───► C
A ──lives────► D
```

Now the adjacency structure is not enough.

You need:

$$
A_{friend},A_{works},A_{lives}.
$$

R-GCN effectively learns a transformation for each relation:

$$
W_{friend},W_{works},W_{lives}.
$$

Thus the graph's **topology and semantics** are modeled simultaneously.

---

## 24. Important limitation: over-smoothing

Like other message-passing GNNs, R-GCN can suffer from **over-smoothing**.

As the number of layers increases, node representations can become increasingly similar:

$$
h_u^{(L)}
\approx
h_v^{(L)}.
$$

This is undesirable because nodes lose their individuality.

Therefore, simply making R-GCN deeper does not necessarily improve performance.

This is one reason practical R-GCN models are often relatively shallow.

---

## 25. Important limitation: over-squashing

Another issue is **over-squashing**.

A node may need to receive information from an exponentially growing number of distant nodes.

For example:

```text
many nodes
   \ \ | / /
    \  |  /
     \ | /
      [v]
```

A fixed-size embedding must compress all that information into:

$$
h_v\in\mathbb R^d.
$$

As the receptive field grows, too much information can be forced into too few dimensions.

This is a general message-passing GNN problem and also applies to R-GCN.

---

## 26. Relation imbalance

Another practical problem is that relation frequencies can be highly imbalanced.

For example:

$$
|\mathcal E_{\text{friend}}|
\gg
|\mathcal E_{\text{acquired\_by}}|.
$$

Some relations may occur millions of times while others occur only a few times.

The model may consequently learn much better representations for frequent relations.

Normalization, parameter sharing, regularization, and careful sampling can help.

---

## 27. Why basis decomposition is conceptually interesting

Basis decomposition isn't merely a parameter-saving trick.

It also introduces a form of **parameter sharing across relations**.

Recall:

$$
W_r=\sum_b a_{rb}V_b.
$$

This implies that relations are represented through combinations of common transformations.

You can think of:

$$
V_1,V_2,\ldots,V_B
$$

as a learned relational “dictionary.”

Each relation gets coordinates:

$$
a_r=[a_{r1},...,a_{rB}].
$$

This can be useful when many relation types share underlying patterns.

---

## 28. R-GCN's inductive bias

Every neural architecture has an **inductive bias**—assumptions about what patterns are likely to be useful.

R-GCN assumes:

1. neighboring nodes are informative;
2. relation type determines how information should be transformed;
3. the same relation semantics can be shared across the graph;
4. local message passing can construct useful global representations;
5. relations can often share parameters through decomposition.

This is why R-GCN is particularly appropriate when **edge semantics are fundamental to the problem**.

---

## 29. Computational complexity

Ignoring implementation details, a straightforward R-GCN layer requires relation-specific message transformations.

Conceptually:

$$
O\left(
\sum_{r\in R}
|\mathcal E_r|d_{\text{in}}d_{\text{out}}
\right)
$$

for dense transformations.

With sparse graph operations and efficient implementation, the computation can be considerably more practical.

The important point is that computational cost depends on:

* number of edges,
* number of relations,
* hidden dimension,
* number of layers.

The number of relations also drives parameter growth, which is why basis/block decomposition is important.

---

## 30. The most important equations to remember

If you are studying R-GCN, I would memorize these four.

**① Core R-GCN layer**

$$\boxed{h_v^{(l+1)}=\sigma\left(W_0^{(l)}h_v^{(l)}+\sum_{r\in\mathcal R}\sum_{u\in\mathcal N_v^r}\frac{1}{c_{v,r}}W_r^{(l)}h_u^{(l)}\right)}$$

**② Matrix form**

$$\boxed{H^{(l+1)}=\sigma\left(\sum_rD_r^{-1}A_rH^{(l)}W_r^{(l)}+H^{(l)}W_0^{(l)}\right)}$$

**③ Basis decomposition**

$$
\boxed{
W_r=
\sum_{b=1}^{B}a_{rb}V_b
}
$$

**④ Link-prediction formulation**

$$
\boxed{
score(u,r,v)=f(h_u,r,h_v)
}
$$

These capture most of the fundamental theory.

---

## 31. A useful mental model

If you remember only one conceptual diagram, remember this:

```text
                 RELATIONAL GRAPH
                       │
       ┌───────────────┼────────────────┐
       │               │                │
       ▼               ▼                ▼
    Relation 1      Relation 2       Relation 3
       │               │                │
      W₁              W₂               W₃
       │               │                │
       └───────────────┼────────────────┘
                       ▼
                 Message aggregation
                       │
                       ▼
                  + self message
                       │
                       ▼
                    σ( · )
                       │
                       ▼
                  New embedding
```

The essence is:

$$
\boxed{
\text{Graph structure}
+
\text{relation semantics}
\rightarrow
\text{node representations}
}
$$

---

## 32. R-GCN in one sentence

> **R-GCN is a message-passing GNN that learns a different transformation for each edge/relation type, aggregates relation-specific messages from neighboring nodes, combines them with the node's own representation, and thereby produces embeddings that encode both graph structure and relational semantics.**

---

## 33. Big-picture comparison

| Model     | Relation types       | Main idea                                          |
| --------- | -------------------- | -------------------------------------------------- |
| GCN       | Usually one          | Aggregate neighbors with shared transformation     |
| GraphSAGE | Usually one          | Sample + aggregate neighbors                       |
| GAT       | Usually one/multiple | Attention-weighted neighbors                       |
| **R-GCN** | **Multiple**         | **Relation-specific transformations**              |
| HAN       | Heterogeneous        | Attention over semantic/meta-path structures       |
| HGT       | Heterogeneous        | Type/relation-dependent attention                  |
| R-GAT     | Multiple             | Relation-aware attention                           |
| CompGCN   | Multiple             | Jointly composes node and relation representations |

The key distinction is that **R-GCN explicitly models the semantics of relations through \(W_r\)**.

---

## 34. Final conceptual picture

The whole theory can be compressed into this progression:

$$
\boxed{
\text{Node features}
}
$$

$$
\downarrow
$$

$$
\boxed{
\text{Separate neighbors according to relation}
}
$$

$$
\downarrow
$$

$$
\boxed{
W_{r_1}h_u,\quad W_{r_2}h_u,\quad\ldots,\quad W_{r_R}h_u
}
$$

$$
\downarrow
$$

$$
\boxed{
\text{Normalize + aggregate}
}
$$

$$
\downarrow
$$

$$
\boxed{
+\;W_0h_v
}
$$

$$
\downarrow
$$

$$
\boxed{
\sigma(\cdot)
}
$$

$$
\downarrow
$$

$$
\boxed{
\text{relation-aware node embedding}
}
$$

Repeated across layers:

$$
G
\rightarrow
H^{(1)}
\rightarrow
H^{(2)}
\rightarrow
\cdots
\rightarrow
H^{(L)}.
$$

Those final embeddings can then be used for **node classification, link prediction, entity classification, knowledge graph completion, recommendation, and other relational graph tasks**.

If you're studying R-GCN seriously, the next concepts I would learn in order are 
1. the derivation of the R-GCN equation from ordinary GCN, 
2. basis/block decomposition, 
3. inverse relations, 
4. R-GCN + DistMult/Link Prediction, and (5) how R-GCN differs mathematically from HGT/HAN/CompGCN.
