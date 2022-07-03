# [Private Draft] Low-Diameter Decomposition (LDD) 

By Danupon Nanongkai

**Goal: outline the proof of a theorem about directed Low-Diameter Decomposition (LDD) from [BNW22] (Theorem 1 below).**


Remarks: 
* These notes have not been subjected to the usual scrutiny reserved for formal publications.
* The algorithm presented here is slightly different from [BNW22] but the main ideas are the same. Due to this, there might be some errors in these notes.
* While I tried to make these notes self-contained, the main purpose is to supplement my lecture.
* I usually start with precise mathematical statements, followed by the discussions. If you are lost in the math, look at the discussions for some help. 
* Let me know if more explanations are needed somewhere.


## Standard definitions

Come back to this section if you see some definitions you don't recognize.
* Throughout, we consider directed graphs $G = (V,E,w)$ with integer edge weight function $w$. Let $m:=|E|$ and $n:=|V|$. We also use $V(G)$ to denote the sets of vertices in $G$. 
* Let $G[S]$ denote the subgraph of $G$ induced by $S\subseteq V$.
* For any $V'\subseteq V$ and $E'\subseteq E$, define $G\setminus V'$ and $G\setminus E'$ as the graphs resulting from removing vertices in $V'$ and edges in $E'$, respectively. 
* $dist_G(u,v)$ = the total weight of the shortest path path from $u$ to $v$ in $G$. 
* DAG = Directed Acycle Graph
* W.h.p. = with high probability 
* $\tilde O$ hides $polylog(n)$.


## 1. Main theorem


**Definition (weak/strong diameter):** Given a weighted directed graph $G = (V,E,w)$ and $S\subseteq V$, the **weak diameter** (w.r.t. $G$) of $S$ or $G[S]$ is defined as $\max_{u,v\in S} dist_G(u,v)$. Their **strong diameter** is defined as $\max_{u,v\in S} dist_{G[S]}(u,v)$.


---
**Theorem 1:** There is a randomized algorithm $LDD(G,D)$ with the following guarantees:
* INPUT: an $m$-edge, $n$-vertex graph $G = (V,E,w)$ with non-negative integer edge weight function $w$ and a positive integer $D$.
* OUTPUT: An edge set $E_{Rem}\subseteq E$ such that 
    1. each SCCs of $G\setminus E_{rem}$ has weak diameter $O(D)$, i.e. if $u, v$ are in the same SCC, then $dist_G(u, v) ≤ D$ and $dist_G(v, u) ≤ D,$ and
    3. for every $e\in E$, $Pr[e\in E_{rem}]=O\left(\frac{w(e) \cdot (\log n)^2}{D} + n^{-8}\right)$. 
* RUNTIME: The algorithm has running time $\tilde O(m+n)$ in expectation. 
---

**Remarks:** It is **not** guaranteed that the events $e\in E_{rem}$ and $e'\in E_{rem}$ are independent, for any $e,e'\in E$. In our paper, we use $E_{sep}$ instead of $E_{rem}$. 


**Example (can skip):** If $G$ is a path $v_1, v_2, ..., v_n$, then we can achieve the theorem by randomly selecting $i\in [1, \sqrt{D}]$ and deleting edges $(v_i, v_{i+1})$,$(v_{i+D}, v_{i+D+1})$, ...

## 2. Undirected LDD via ball growing
We start with a classic algorithm for computing a LDD for **undirected** graphs. We will prove output properties similar to Theorem 1. We will not discuss the runtime yet. 


### 2.1 Algorithm

**Definition (Ball):** For any vertex $s$ in an *undirected* graph $G$ and postive integer $R$, let $Ball_G(s,R)=\{v\in V \mid dist_G(s,v)\leq R\}.$ Let $\partial Ball_G(s,R)=\{(x,y)\in E \mid x\in Ball_G(s,R), y\notin Ball_G(s,R)\}$ 

---

**Algorithm** UndiLDD(G, D)  <span style="color:gray">// Explanation  is below</span>
1. $\mathcal{P}\leftarrow \emptyset$,  and $E_{rem}\leftarrow\emptyset$. 
2. While $G$ is not empty:
    2.1 Pick arbitrary vertex $v$ in $G$ 
    2.2 Sample $R_v\sim Geom(p)$ for $p=\min\{1, 20\ln(n)/D\}$. 
    2.3 If $R_v> D/2$, let $R_v=D/2$. 
    2.4 Add edges in $\partial Ball_G(v,R_v)$ to $E_{rem}$. <span style="color:gray">// Make $Ball_G(v,R_v)$ and $SCC$ in $G\setminus E_{rem}$.</span>
    2.5 $G\leftarrow G\setminus Ball_G(v,R_v)$. 

**Return** $E_{rem}$.  <span style="color:gray">// Return vertex partition $\mathcal{P}$ and edge set $E_{rem}$.</span>


---

**Explanation (can skip):** The algorithm carves out a "ball" of some radius around some vertex $v$. It then repeats this until the graph is empty. The balls form a vertex partition and $E_{rem}$ is simply the set of edges between them. 


The key is to pick the raduis according to a "truncated" geometric distribution with parameter $p\approx 1/D$, as in step 2.2 in the pseudocode. This is equivalent to the number of time we need to flip a coin until it turns head, which happens with probability $p\approx 1/D$, but we stop afer $D$ coin flips. 


==Example==

**Ball growing process:** The algorithm can be viewed starting with a ball of radius 0 at some vertex $v$. We flip a coin that turns head with probability $p\approx 1/D$. Every time we get a tail, we increase the radius of the ball by one. We stop when we get a head or the radius is $D$. We cut out the ball and repeat the ball growing process from a new vertex. 

### 2.2 Output Property 2


**Lemma 2.1:** For every $e\in E$, $Pr[e\in E_{rem}]
 = O\left(\frac{w(e) \cdot (\log n)}{D}+n^{-9}\right).$ 
**Proof sketch:**
(I) Consider the algorithm *without* step 2.3, i.e. we allow $R_v>D$. We claim that $Pr[e\in E_{rem}]\leq w(e)\cdot p$$= O\left(\frac{w(e) \cdot (\log n)}{D}\right):$
* Consider any edge $(u,v)$. 
* Consider the first time one of $u$ and $v$ (say $u$) is contained in a ball during the ball-growing process above. After this, either (i) both of $u$ and $v$ are in the ball or (ii) edge $(u,v)$ is added to $E_{rem}$.
* If $w(u,v)=1$, whether (i) or (ii) occurs depends on the next coin flip (what happened before this does not matter). In particular, if we get head (happening w.p. $p$), the process stops and $e\in E_{rem}$; otherwise, both $u$ and $v$ are in the ball so $e$ will never be in $E_{rem}.$ So, $Pr[e\in E_{rem}]\leq p$
* For $w(e)> 1$, the same idea gives $Pr[e\in E_{rem}]\leq w(e)\cdot p.$

(II) Now consider the effect of step 2.3: Some edge $(u,v)$ might be added to $E_{rem}$ when we set $R_v=D$ in Step 2.3. But the probability that step 2.3 is ever executed is very small:
* For any vertex $v$, $Pr[R_v\geq D/2]$$\leq (1-p)^{D/2}\leq e^{-pD/2}$$=e^{-\frac{20 \ln(n) D}{2D}}=n^{-10}$
    * Note: $\forall x>1$, $(1-1/x)^{x-1}>1/e>(1-1/x)^x$.
* By Union Bound, the probability that we set $R_v=D$ in Step 2.3, causing any edge to be added to $E_{rem}$ is $\leq n^{-9}$. 

**QED**


### 2.3 Output Property 1 

Output property 1 in Theorem 1 is easy to prove: Observe that the SCCs (i.e. the connected components) of $G\setminus E_{rem}$ are the balls that we carved out. Since we carved out balls of radius $R_v\leq D$, each SCC has strong(!) diameter $O(D)$.


*Remark:* Observe that we get slightly stronger output properties than stated in Theorem 1: we get a strong diameter of $O(D)$ and $Pr[e\in E_{rem}]\approx \frac{w(e) \cdot (\log n)}{D}$ instead of $\frac{w(e) \cdot (\log n)^2}{D}$.





## 3. Directed LDD via light nodes & recursion

We now extend the above algorithm to directed graphs. We will not discuss the runtime yet. 


### 3.1 Highlights (can skip)
Details are in later sections.

* For digraph, our balls are $Ball^{in}_G(s, R)$ and $Ball^{out}_G(s, R)$. We perform ball carving process like above.
    * Detail: For each carved ball, we include edges only in *one* direction  to $E_{rem}$ (either pointing out of or into the ball). 
* We carve out only balls centered at $v$ with $Ball^{in}_G(v, D)$ or  $Ball^{out}_G(v, D)$ of size $\leq n/2$. Call such $v$ a (in/out-)**light node** (other nodes are **heavy nodes**). 
* **Key lemma:** The set of heavy nodes have weak diameter $O(D)$.
* **Recursion:** We repeat the ball-carving framework on the carved-out balls. Eventually, each component becomes a singleton or has low diameter. Since we recurse on a ball of size $\leq n/2$, the probability that an edge is in $E_{rem}$ is $O(\log(n))$ times more than the undirected case. 


### 3.2 Algorithm

**Definition (in/out-Balls):** For any vertex $s$ in a *directed* graph $G$ and postive integer $R$, let $Ball^{out}_G(s,R)=\{v\in V \mid dist_G(s,v)\leq R\}$ and $Ball^{in}_G(s,R)=\{v\in V \mid dist_G(v,s)\leq R\}.$ Let $\partial Ball_G^{out}(s,R)=\{(x,y)\in E \mid x\in Ball_G(s,R), y\notin Ball_G(s,R)\}$ and $\partial Ball_G^{in}(s,R)=\{(y,x)\in E \mid x\in Ball_G(s,R), y\notin Ball_G(s,R)\}$ 

==PICTURE==


---
**Algorithm** LDD($G$, $D$)  <span style="color:gray">// See explanation below</span>


1. $E_{rem}\leftarrow\emptyset$ and $n\leftarrow |V(G)|$.  <span style="color:gray">// $n$ does not change in the steps below.</span>
2. For every $v\in V$, if $|Ball^{in}_G(v, D)|\leq n/2$, mark $v$ as **in-light**; 
else if $|Ball^{out}_G(v, D)|\leq n/2$, mark $v$ as **out-light**. 
3. While $G$ contains an **in-light** vertex $v$:
    3.1 Sample $R_v\sim Geom(p)$ for $p=\min\{1, 20\log(n)/D\}$. If $R_v> D$, let $R_v=D$. 
    3.2 $E_{rem}'\leftarrow LDD\left(G[Ball_G^{in}(v,R_v)], D\right)$. <span style="color:gray">// Recurse</span>
    3.3 $E_{rem}\gets E_{rem} \cup \partial Ball_G^{in}(v,R_v) \cup E'_{rem}$
    3.4 $G\leftarrow G\setminus Ball_G^{in}(v,R_v)$. 
4. While $G$ contains an **out-light** vertex $v$: 
<span style="color:gray">// Repeat step 3 with "out" instead of "in" </span>
    4.1 Sample $R_v\sim Geom(p)$ for $p=\min\{1, 20\log(n)/D\}$. If $R_v> D$, let $R_v=D$. 
    4.2 $E_{rem}''\leftarrow LDD\left(G[Ball_G^{out}(v,R_v)], D\right)$. <span style="color:gray">// Recurse</span>
    4.3 $E_{rem}\gets E_{rem} \cup \partial Ball_G^{out}(v,R_v) \cup E''_{rem}$
    4.4 $G\leftarrow G\setminus Ball_G^{out}(v,R_v)$. 

**Return** $E_{rem}$ 

---

**Explanation:** Since $G$ is directed, we distinguish the balls to "in-balls" $Ball^{out}_G(s,R)$ and "out-balls" $Ball^{in}_G(s,R)$, where $Ball^{out}_G(s,R)$ include nodes whose distance **from** $s$ is at most $D$ and $Ball^{in}_G(s,R)$ include nodes whose distance **to** $s$ is at most $D$. 

For simplicity, let's assume that $G$ is strongly-connected. If we ignore the recursions (steps 3.2 and 4.2), the algorithm (steps 3-4) simply carves out from $G$ balls of random radius as before. However, it does so only for in/out-**light** nodes defined in step 2. (We will show below that the remaining graph already has $O(D)$ weak diameter.)  Another difference is that it only adds edges in one direction to $E_{rem}$: If it carves out an in-ball $Ball_G^{in}(v,R_v)$, then it adds edges pointing **into** the ball to $E_{rem}$ (step 3.3, where we set $E_{rem}\gets E_{rem} \cup \partial Ball_G^{in}(v,R_v)$).  Similarly, if it carves out an out-ball $Ball_G^{out}(v,R_v)$, then it adds edges pointing **out from** the ball (step 4.3). 

Another difference to the undirected case is the **recursion**: Since we cannot guarantee that the balls we carved out have low diameter, we recursively decompose them (steps 3.2 and 4.2). Since these balls contain $\leq n/2$ nodes, intuitively this increases the probability that an edge is in $E_{rem}$ by a factor of $O(\log n)$. Note that the recursion stops when, e.g., $n=1$ since there is no light node.


### 3.3 Output Property 1 (weak diameter)

In each recursive call of the algorithm, we call nodes marked in- and out-light in step 2 as **light nodes** and call other nodes **heavy**.

**Key lemma:** The set of heavy nodes have weak diameter $\leq 2D$, i.e.  any two heavy nodes $u,v\in V(G)$,  $dist_G(u,v)\leq 2D$
**Proof:** 
* Consider any two heavy nodes $u,v\in V(G)$. 
* There exists $x\in Ball^{out}_G(u, D)\cap Ball^{in}_G(v, D)$
    * since $Ball^{out}_G(u, D)>n/2$ and $Ball^{in}_G(v, D)>n/2.$
* Thus, $dist_G(u,v)\leq dist_G(u,x)+dist_G(x,v)\leq 2D$. 

**QED**


The key lemma easily implies property 1. Detail: 
* Let $B_i$, for $i=1, 2, \ldots,$ be the $i^{th}$ carved-out ball. Let $B_\infty$ be the set of vertices that remain in $G$ after we carve out all balls in Step 3-4.
* Any $x\in B_i$ and $y \in B_j$, for any $i<j$, are **not** in the same SCC in $G\setminus E_{rem}$
    * since we either put to $E_{rem}$ all edges pointing into or out of $B_i$; 
    * e.g. if $B_i=Ball^{out}_G(v, R_v)$ for some vertex $v$, then we put edges in  $\partial Ball^{out}_G(v, R_v)$ to $E_{rem}$ and thus there is no path from $x$ to $y$ in $G\setminus E_{rem}$.
* Thus, each SCC of $G\setminus E_{rem}$ is contained in some $B_i$. SCCs in $B_\infty$ has $O(D)$ weak diameter by the key claim. We can argue by induction that other SCCs has $O(D)$ weak diameter. 


### 3.4 Output Property 2

Output property 2 simply follows from By the analysis from the unweighted case (Lemma 2.1) and the fact that each edge participates in $O(\log n)$ recursive calls. Detail:

* By the analysis from the unweighted case (Lemma 2.1), in each recursive call LDD($G$, $D$), for every edge  $E\in E(G)$, $Pr[e\in E_{rem}]= O\left(\frac{w(e) \cdot (\log n)}{D}+n^{-9}\right)$. 
* Each edge participates in $O(\log n)$ recursive calls; i.e. any edge $e$ in the input graph is contained in $O(\log n)$ among all recursive calls of LDD($G$, $D$).
    * Reason: When we make recursive calls $LDD\left(G[Ball_G^{in}(v,R_v)], D\right)$ and $LDD\left(G[Ball_G^{out}(v,R_v)], D\right)$ in steps 3.2 and 4.2, the graphs $G[Ball_G^{in}(v,R_v)]$ and $G[Ball_G^{out}(v,R_v)]$ contain $\leq n/2$ vertices. (The recursive calls stop when $n=1$ at the latest.) 
* By union bound, for every edge $e$ in the input graph, $Pr[e\in E_{rem}]= O\left(\frac{w(e) \cdot (\log n)^2}{D}+n^{-9}\log n\right)= O\left(\frac{w(e) \cdot (\log n)^2}{D}+n^{-8}\right)$. 


## 4. Runtime


**Highlights (can skip):** The most interesting part is detecting light nodes (step 2). Main ideas:
* We estimate $|Ball^{in}_G(v, D)|$ and $|Ball^{out}_G(v, D)|$ by sampling a set $S$ of $\Theta(\log n)$ nodes and compute $|Ball^{in}_G(v, D)\cap S|$ and $|Ball^{out}_G(v, D)\cap S|.$
* We can compute $|Ball^{in}_G(v, D)\cap S|$ and $|Ball^{out}_G(v, D)\cap S|$ for *all* nodes $v$ in $\tilde O(m)$ time in total by computing  $Ball^{in}_G(s, D)$ and  $Ball^{out}_G(s, D)$ for every $s\in S$. 
* The algorithm and proof in the previous section can be easily modified to handle the fact that we only know the estimates of $|Ball^{in}_G(v, D)|$ and $|Ball^{out}_G(v, D)|$. 

### 4.1 Detecting light nodes (step 2)


#### 4.1.1 Implementing step 2: 

2.1 $k\gets c\ln n$ for a big enough constant $c$. 
2.2 $S\gets \{s_1, s_2, \ldots s_k\}$ where $s_i$ is a random node in $V$ for every $i$.
* Possible: $s_i=s_j$ for some $i\neq j$.

2.3 For each $s\in S$, compute $Ball^{in}_G(s, D)$ and $Ball^{out}_G(s, D)$ <span style="color:gray">// $O(mk)$ time </span>
2.4 For each $v\in V$:  <span style="color:gray">//  $O(nk)$ time </span>

* $|Ball^{in}_G(v, D)\cap S|\gets\{s\in S\mid v\in Ball^{out}_G(s, D)\}$ 
* $|Ball^{out}_G(v, D)\cap S|\gets \{s\in S\mid v\in Ball^{in}_G(s, D)\}$

2.5 For every $v\in V$, if $|Ball^{in}_G(v, D)\cap S|\leq (0.6)k$, mark $v$ as **in-light**; 
else if $|Ball^{out}_G(v, D)\cap S|\leq (0.6) k$, mark $v$ as **out-light**. 

---

#### 4.1.2 Chernoff's bound:

The following lemma easily follows from Chernoff's bound.



**Lemma 4.1:** W.h.p., the following holds for every $v\in V$ and $*\in \{in, out\}$:
$\frac{|Ball^{*}_G(v, D)\cap S|}{k}- 0.1 \leq \frac{|Ball^{*}_G(v, D)|}{n} \leq \frac{|Ball^{*}_G(v, D)\cap S|}{k} + 0.1$


**Proof (tedious):** We use the following form of Chernoff's bound.
* Let $𝑋_1, . . . , 𝑋_k$: independent random variables s.t. $𝟎≤𝑿_𝒊≤𝟏$ for all $𝑖$. 
* Let $𝑋_{𝑎𝑣𝑔}  =(1/k)  \sum_{i=1}^k 𝑋_𝑖$  
* For any $𝜖>0$,
    (a) $Pr⁡[𝑋_{𝑎𝑣𝑔}>𝐸[𝑋_{𝑎𝑣𝑔} ]+𝜖]≤𝑒^{−2k𝜖^2}$
    (b) $Pr⁡[𝑋_{𝑎𝑣𝑔}<𝐸[𝑋_{𝑎𝑣𝑔} ]−𝜖]≤𝑒^{−2k𝜖^2}$


Consider any $v\in V$: 
* For any $i\in [1,k]$, define $X_i=1$ if $s_i\in Ball^{in}_G(v, D)$ and $X_i=0$ otherwise. 
    * $E[X_i]=|Ball^{in}_G(v, D)|/n$ $\forall i$.
* So, $X_{avg}=\frac{|Ball^{in}_G(v, D)\cap S|}{k}$ and $E[X_{avg}]=\frac{|Ball^{in}_G(v, D)|}{n}$. 
* By (a), $Pr\left[\frac{|Ball^{in}_G(v, D)\cap S|}{k}>\frac{|Ball^{in}_G(v, D)|}{n}+0.1\right]\leq 1/n^{10}$. In other words, w.h.p., $\frac{|Ball^{in}_G(v, D)|}{n}\geq \frac{|Ball^{in}_G(v, D)\cap S|}{k}-0.1.$

Other inequalities are proved similarly. The lemma follows by union bound. 

**QED:**


----
#### 4.1.3 Output Properties (sketched): 

Lemma 4.1 implies that the following holds w.h.p.

**(I)** For any $*\in \{in,out\}$, if $|Ball^{*}_G(v, D)\cap S|>0.6k$, then $|Ball^{*}_G(v, D)|>0.5n$. Thus, heavy nodes $v$ according to our new definition are also heavy by the old definition (section 3), i.e. $|Ball^{*}_G(v, D)|>0.5n$. Consequently, we can use the key lemma in the previous section to argue that the weak diameter of every SCC in $G\setminus E_{rem}$ is $O(D)$ w.h.p.


**(II)** For any $*\in \{in,out\}$, if $|Ball^{*}_G(v, D)\cap S|\leq 0.6k$, then $|Ball^{*}_G(v, D)|\leq 0.7n$. This implies that, w.h.p., we only recurses on balls of size $\leq 0.7n$. This implies that, w.h.p., the recursion depth is  $O(\log n)$ like before. Thus the proof in section 3 can still be used to show that $Pr[e\in E_{rem}]=O\left(\frac{w(e) \cdot (\log n)^2}{D} + n^{-8}\right)$. 

**Eliminating "whp":** Above we argue that the output properties  in Theorem 1 hold w.h.p. We can avoid this "w.h.p.":
* We check if the weak diameter of every SCC is $O(D)$ 
    * by computing whether the distances from some node $v$ in the SCC to all other nodes; they must be $O(D)$.  
* We can check if we always recurse on balls of size $\leq 0.7n$.
* If one of the above does not hold, we put every edge in $E_{rem}$. This happens with probability (say) $\leq 1/n^{10}$, so it does not affect our guarantee on $Pr[e\in E_{rem}]$. 


### 4.2 Runtime of steps 3-4

* Generating random variables (steps 3.1 & 4.1):  $O(\log n)$ expected runtime if we use [BF13].  
    * By Theorem 5 in [BF13] with $p=\log(n)/D$ and word size $w=\Omega(\log\log D)$, the expected time is $O(\log(\min\{D/\log n, n\})/\log w)=O(\log n)$
    * There might be other simpler methods since we do not need an exact algorithm. Most programming languages also provide highly optimized methods to do this (e.g. numpy.random.geometric).
* Since we remove $Ball_G^{in}(v,R_v)$ and  $\partial Ball_G^{in}(v,R_v)$  from $G$ after we compute them in steps 3.2-3.3, the total time require by steps 3.2-3.4 is $O(m)$.  The same holds for steps 4.3-4.4. 

So, ignoring the recursion, steps 3-4 require $O(m)$ time in total. 

### 4.3 Summary

The runtime of step 2 is $O(mk)=O(m\log n)$ and steps 3-4 requires $O(m)$ runtime. Since each edge participates in $O(\log n)$ recursions, the total runtime is $O(m(\log n)^2)$.

Finally, for clarity, below is the pseudocode of the fast LDD algorithm we discussed (we combine the previous steps 3-4 for compactness). 

**Algorithm** FastLDD($G$, $D$)  <span style="color:gray">// See explanation below</span>


1. $G_0\gets G$. $E_{rem}\leftarrow\emptyset$ and $n\leftarrow |V(G)|$.  <span style="color:gray">// $n$ does not change in the steps below.</span>
2. For every $v\in V$, if $|Ball^{in}_G(v, D)|\leq n/2$, mark $v$ as **in-light**; 
else if $|Ball^{out}_G(v, D)|\leq n/2$, mark $v$ as **out-light**. 
    2.1 $k\gets c\ln n$ for a big enough constant $c$. 
    2.2 $S\gets \{s_1, s_2, \ldots s_k\}$ where $s_i$ is a random node in $V$ for every $i$. (Possible: $s_i=s_j$ for some $i\neq j$.)
    2.3 For each $s\in S$, compute $Ball^{in}_G(s, D)$ and $Ball^{out}_G(s, D)$ <span style="color:gray">// $O(mk)$ time </span>
    2.4 For each $v\in V$:  <span style="color:gray">//  $O(nk)$ time </span>
    * $|Ball^{in}_G(v, D)\cap S|\gets\{s\in S\mid v\in Ball^{out}_G(s, D)\}$ 
    * $|Ball^{out}_G(v, D)\cap S|\gets \{s\in S\mid v\in Ball^{in}_G(s, D)\}$
    2.5 For every $v\in V$, if $|Ball^{in}_G(v, D)\cap S|\leq (0.6)k$, mark $v$ as **in-light**; 
else if $|Ball^{out}_G(v, D)\cap S|\leq (0.6) k$, mark $v$ as **out-light**. 
3. While $G$ contains an **$*$-light** vertex $v$ for any $*\in \{in,out\}$:
    3.1 Sample $R_v\sim Geom(p)$ for $p=\min\{1, 20\log(n)/D\}$. If $R_v> D$, let $R_v=D$. 
    3.2 Compute $Ball_G^{in}(v,R_v)$. If $|Ball_G^{*}(v,R_v)|>0.7n$, return $E_{rem}=E_{rem}\cup E(G)$ and terminate. <span style="color:gray">// Claim: $Pr[terminate]\leq 1/n^{10}$</span>
    3.3 $E_{rem}'\leftarrow LDD\left(G[Ball_G^{*}(v,R_v)], D\right)$. <span style="color:gray">// Recurse</span>
    3.4 $E_{rem}\gets E_{rem} \cup \partial Ball_G^{*}(v,R_v) \cup E'_{rem}$
    3.4 $G\leftarrow G\setminus Ball_G^{*}(v,R_v)$. 
4. Let $v$ be any node in $G$.  If $dist_{G_0}(v,u)>2D$ or $dist_{G_0}(u,v)>2D$ for any node $u$, return $E_{rem}=E_{rem}\cup E(G)$ and terminate. <span style="color:gray">// Claim: $Pr[terminate]\leq 1/n^{10}$</span>

**Return** $E_{rem}$ 



## Notes
 

 
 
[Linial-Saks](https://dl.acm.org/doi/10.5555/127787.127848)

[Gupta](https://www.cs.cmu.edu/~avrim/Randalgs11/lectures/lect0302.pdf)


## References

[BF13] Karl Bringmann, Tobias Friedrich: Exact and Efficient Generation of Geometric Random Variates and Random Graphs. ICALP (1) 2013: 267-278 [link](https://people.mpi-inf.mpg.de/~kbringma/paper/2013ICALP-1.pdf) 

 [BNW22] Negative-Weight Single-Source Shortest Paths in Near-linear Time. [arXiv](https://arxiv.org/abs/2203.03456)
