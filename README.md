# Online Dictionary Learning demo

This noteboook is intended to provide a simple pytorch implementation for dictionary learning (DL) based on stochastic gradient descent. This is in essence a multi-purpose DL method that I put together in the midst of my own work, and I frequently use to carry out simple experiments and tests.
This implementation does not follow any particular publication, though it is perhaps closest to the OSDL work on sparse dictionaries (without the double-sparisy component), or to the Online Dictionary Learning algorithm by Mairal (though without their convex surrogate function approach).

Look at Description.ipynb for the general description of the algorithm.

------

### Basic algorithm description
The Dictionary Learning problem is concerned with
\begin{equation}
\min_{\gamma_i,D} \sum_{i=1}^N \|y_i - D\gamma_i\|_2^2 + \lambda\ g(\gamma_i), \quad s.t. \quad \ \|d_j\|_2 = 1, \forall j
\end{equation}
where $g(\gamma_i)$ is a spase-enforcing penalty term such as the $\ell_1$ norm or the $\ell_0$ pseudo-norm. Both will be considered in this implementation. 

Note that the sum of the reconstruction error per sample can be denoted in matrix form as $\|Y - D\Gamma\|^2_F$, where matrices $Y$ and $\Gamma$ have the vectors $y_i$ and $\gamma_i$ in their columns, respectively.

Most dictionary learning methods employ an alternating minimization approach to address the above non-convex problem, by alternating between:
* Minimizing the objective w.r.t. $\gamma_i$ while keeping the dictionary fixed, termed **Sparse Coding**, and
* Minimizing the objective w.r.t. $D$, while keeping the representations $\gamma_i$ fixed, termed **Dictionary Update**.

We will employ such alternating approach as well. 

More broadly, one could apply a **batch** scheme: perform sparse coding _on all_ training examples, and then update $D$ accordingly (and iterate these steps). Alternatively, one might employ a **stochastic optimization** approach, and minimize the loss above one sample (or one mini-batch) at a time. We will employ this latter online implementation.

#### Sparse Coding
When minimizing for every representation $\gamma_i$ (with the dictionary $D$ being fixed), this implementation allows for two sparse-enforcing penalties:
* When the sparsity penalty function is the $\ell_1$ norm, the problem to be minimized is
\begin{equation}
\min_{\gamma_i} \|y_i - D\gamma_i\|_2^2 + \lambda \ \|\gamma_i\|_1 \ \forall i
\end{equation}
and we employ the Fast Iterative Threhsolding Algorithm from Beck and Teboulle, or FISTA for short.

* When the sparsity penalty funcition is the non-convex and non-smooth $\ell_0$ pseudo-norm, we opt for a constraint formulation and we minimize:
\begin{equation}
\min_{\gamma_i} \|y_i - D\gamma_i\|_2^2 \quad s.t. \quad \|\gamma_i\|_0 \leq k \ \forall i,
\end{equation}
and we employ the Iterative Hard Tresholding method. Generally speaking, this $\ell_0$ approach may lead to higher number of "dead" filters (atoms that are not used nor trained), which is typically solved by introducing other simple exteneral regularization techniques (replacing unused and repeated atoms, progressively reducing the target cardinality through training, etc).

#### Dictionary Update
After having found all $\gamma_i$ for each $y_i$ in the mini-batch, the dictionary update problem is concerned with
\begin{equation}
\min_{D} \|Y_i - D\Gamma_i\|_2^2 \quad s.t. \quad \ \|d_j\|_2 = 1, \forall j.
\end{equation}
Ignoring for a moment the $\ell_2$ constraint on the dictionary atoms, one could minimize this $\ell_2$ loss with the Least Squares solution. In favor or a less severe minimization (note, this is still an alternating minimization approach for a non-convex problem), we simply perform a grandient step so as to minimize this norm, followed by a renormalization of the atoms to unit norm.


This Sparse Coding and Dictionary Update steps are iterated, every time for a different mini-batch, with stochastic gradient descent (with momentum).
