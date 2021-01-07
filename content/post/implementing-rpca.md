---
# Documentation: https://sourcethemes.com/academic/docs/managing-content/

title: "Implementing Robust PCA in Python"
subtitle: ""
summary: ""
authors:
- admin

tags: []
# categories: []
date: "2019-10-25T12:35:53-07:00"
lastmod: "2019-10-25T12:35:53-07:00"
featured: false
draft: false
toc: true
math: true

# Featured image
# To use, add an image named `featured.jpg/png` to your page's folder.
# Focal points: Smart, Center, TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight.
image:
  caption: ""
  focal_point: ""
  preview_only: false

# Projects (optional).
#   Associate this post with one or more of your projects.
#   Simply enter your project's folder or file name without extension.
#   E.g. `projects = ["internal-project"]` references `content/project/deep-learning/index.md`.
#   Otherwise, set `projects = []`.
projects: []

---

In [a previous post]({{< ref "/post/pca-to-rpca.md">}}), I introduce robust PCA, the math behind and an example where I put the model in action.
This post I will share my Python implementation of robust PCA.
If you don't have any background in convex optimization, some of the discussions here might be boring or irrelevant.
If you really just need an implementation of robust PCA, skip the background section and you'll find the code below.

## First Some Background

### Alternating Direction Method of Multipliers

Say you have a convex optimization problem that looks like this:
$$
\begin{aligned}   
& \min_{X, Z}    & & f(X) + g(Z) \\\\\\   & \text{subject to}    & & AX + BZ = C \\
\end{aligned}
$$
where $f$ and $g$ are convex.
The augmented Lagrangian looks like this:
$$
\mathcal{L}(X, Z, Y) = f(X) + g(Z) + \langle Y,C-AX-BZ\rangle + \frac{\mu}{2}\|C-AX-BZ\|_F^2
$$
 If you feel the need to understand how we got here, I suggest reading the second chapter of [Boyd *et al*'s classic report](https://stanford.edu/~boyd/papers/pdf/admm_distr_stats.pdf) on this.

ADMM propses an iterative approach using the Lagrangian.
Say we are at iteration $k$ and we are given from previous iteration $X^k$, $Z^k$, $Y^k$.
The required iterations for step $k+1$ will become:
$$
\begin{aligned}
	X^{k+1} &= \underset{X}{\mathrm{argmin}}\mathcal{L}(X, Z^k, Y^k) \\\\\\
	Z^{k+1} &= \underset{Z}{\mathrm{argmin}}\mathcal{L}(X^{k+1}, Z, Y^k) \\\\\\
	Y^{k+1} &= Y^k + \mu(C-AX^{k+1}-BZ^{k+1}) 
\end{aligned}
$$

If you have a closed form solution for the updates of $X^{k+1}$ and $Z^{k+1}$, you're in good luck.

ADMM can be thought of as a tug-of-war between optimality gap and feasibility of the primal problem.
This can be used to set up a systematic way to early stop the algorithm and/or play around with $\mu$ over iterations.
According to [Boyd *et al*](https://stanford.edu/~boyd/papers/pdf/admm_distr_stats.pdf), the primal residual and dual residuals of the problem can be quantified as $r^k = \|AX^k-BZ^k-C\|_F^2$ and $h^k = \|\mu_k A^\top B(Z^k - Z^{k+1})\|_F^2$, respectively.
Algorithm can be terminated when both of these quantities are below a certain tolerance and advise on choosing these tolerance values also mentioned in the same report.
To bias the algorithm towards reaching primal feasibility one can dial up $\mu$ (thus increasing the penalty on primal residuals) or dial it down for speeding up closing the optimality gap.
The $\mu_k$ update then becomes:


$$
\mu_{k+1} = \begin{cases} 
\rho\mu_k & \text{if } r^k > \tau s^k \\\\\\ 
\mu_k\mathbin{/}\rho & \text{if } s^k > \tau r^k \\\\\\
\mu_k & \text{o.w.}
		\end{cases}
$$
This gives us an almost complete look at ADMM.
Now we will see how we can write a special case of ADMM for robust PCA.

### Robust PCA via ADMM

Let's refresh our memories.
The optimization problem for robust PCA was:

$$\begin{aligned}   
& \min_{L}    & & \|L\|\_* + \lambda\|S\|\_1 \\\\\\   
& \text{subject to}    & & L + S = M 
\end{aligned}$$

Then we can make following analogies:
$$
\begin{aligned}
f(L) &= \|L\|\_* \\\\\\
g(S) &= \|S\|\_1 \\\\\\
A &= I \\\\\\
B &= I \\\\\\
C &= M \\\\\\
\end{aligned}
$$
I'm sorry about the change of notation here but I feel that's the best way.

What about minimization steps?
I won't go into the details of their derivation but I hope to point you in the right direction if you want to do it on your own.

$$\begin{aligned}
L^{k+1} &= \underset{L}{\mathrm{argmin}}\mathcal{L}(L, S^k, Y^k) \\\\\\
& =\|L\|\_* + \langle Y^k,M-L-S\rangle + \frac{\mu\_k}{2}\|M-L-S\|\_F^2 \\\\\\
& \propto  (1/\mu_k)\|L\|\_* + \|M-S^k+Y^k/\mu^k\|\_F^2 \\\\\\
& = \mathcal{D}\_{1\mathbin{/}\mu\_k}(M-S^k+Y^k/\mu^k)
\end{aligned}$$

where $\mathcal{D}_{1\mathbin{/}\mu_k}$ is the singular value thresholding operator. 
Please refer to Section 2.1 of [Cai *et al.*](https://statweb.stanford.edu/~candes/papers/SVT.pdf) for detailed explanation of it.

Then

$$\begin{aligned}
S^{k+1} &= \underset{S}{\mathrm{argmin}}\mathcal{L}(L^{k+1}, S, Y^k) \\\\\\
&= \lambda\|S\|\_1 + \langle Y^k,M-L-S\rangle + \frac{\mu\_k}{2}\|M-L-S\|\_F^2 \\\\\\
&\propto (\lambda/\mu\_k)\|S\|\_1 + \|X-L-Y^k/\mu^k\|\_F^2 \\\\\\
&= \mathcal{P}_{(\lambda/\mu\_k)}(X-L-Y^k/\mu^k)
\end{aligned}$$

where $\mathcal{P}_{1\mathbin{/}\mu_k}$ is the soft thresholding operator.
Section 6.5.2 of [Boyd *et al.*](https://web.stanford.edu/~boyd/papers/pdf/prox_algs.pdf) is a good reference for the soft thresholding operator.
It is commonly used to solve lasso regression.
It is also a subroutine of the singular value thresholding operator.

## Implementation in Python with Numpy

Now that we have all the steps ready, we can start implementing. 
We will only need a Python environment with version 3.6+ (just because I like static type hinting feature) and numpy.
You can easily set this up with conda or Pipenv.

```python
import numpy as np
import numpy.linalg as la
```

Let's first start by defining the proximal operators: 

```python
def soft_thresholding(y: np.ndarray, mu: float):
    """
    Soft thresholding operator as explained in Section 6.5.2 of https://web.stanford.edu/~boyd/papers/pdf/prox_algs.pdf
    Solves the following problem:
    argmin_x (1/2)*||x-y||_F^2 + lmb*||x||_1

    Parameters
    ----------
        y : np.ndarray
            Target vector/matrix
        lmb : float
            Penalty parameter
    Returns
    -------
        x : np.ndarray
            argmin solution
    """
    return np.sign(y) * np.clip(np.abs(y) - mu, a_min=0, a_max=None)

def svd_shrinkage(y: np.ndarray, tau: float):
    """
    SVD shrinakge operator as explained in Theorem 2.1 of https://statweb.stanford.edu/~candes/papers/SVT.pdf
    Solves the following problem:
    argmin_x (1/2)*||x-y||_F^2 + tau*||x||_*
    
    Parameters
    ----------
        y : np.ndarray
            Target vector/matrix
        tau : float
            Penalty parameter
    Returns
    -------
        x : np.ndarray
            argmin solution
    
    """
    U, s, Vh = np.linalg.svd(y, full_matrices=False)
    s_t = soft_thresholding(s, tau)
    return U.dot(np.diag(s_t)).dot(Vh)
  
```

I want my API to follow a `scikit-learn`-like design so here's how it should like at the end:

```python
rpca = RobustPCA(lmb=4e-3, max_iter=100)
L, S = rpca.fit(X)

```

The only difference is that `.fit` function doesn't normally return anything in `scikit-learn` but I want it to return the low-rank and sparse components for the sake of simplicity.

Then I start building the class with my main method `.fit`. Starting with `.fit` gives me a good idea about what other parameters I have to initialize or what submethods I'll end up having to implement:

```python
def fit(self, X: np.ndarray):
    mu = self.mu_0_
    Y = X / self._J(X, mu)
    S = np.zeros_like(X)
    S_last = np.empty_like(S)
    for k in range(self.max_iter_):
        # Solve argmin_L ||X - (L + S) + Y/mu||_F^2 + (lmb/mu)*||L||_*
        L = svd_shrinkage(X - S + Y/mu, 1/mu)

        # Solve argmin_S ||X - (L + S) + Y/mu||_F^2 + (lmb/mu)*||S||_1
        S_last = S.copy()
        S = soft_thresholding(X - L + Y/mu, self.lmb_/mu)

        # Update dual variables Y <- Y + mu * (X - S - L)
        Y += mu*(X - S - L)
        r, h = self._get_residuals(X, S, L, S_last, mu)

        # Check stopping cirteria
        tol_r, tol_h = self._update_tols(X, L, S, Y)
        if r < tol_r and h < tol_h:
            break

        # Update mu
        mu = self._update_mu(mu, r, h)

    return L, S

```

The private methods I need, `._J`, `._get_residuals`,`._update_tols` and `._update_mu` are implemented as follows.

```python
def _get_residuals(X: np.ndarray, S: np.ndarray, L: np.ndarray, S_last: np.ndarray, mu: float):
    primal_residual = la.norm(X - S - L, ord="fro")
    dual_residual = mu*la.norm(S - S_last, ord="fro")
    return primal_residual, dual_residual

def _update_mu(self, mu: float, r: float, h: float):
    if r > self.tau_ * h:
        return mu * self.rho_
    elif h > self.tau_ * r:
        return mu / self.rho_
    else:
        return mu

def _update_tols(self, X, S, L, Y):
    tol_primal = self.tol_rel_ * max(la.norm(X), la.norm(S), la.norm(L))
    tol_dual = self.tol_rel_ * la.norm(Y)
    return tol_primal, tol_dual

def _J(self, X: np.ndarray, lmb: float):
    return max(np.linalg.norm(X), np.max(np.abs(X))/lmb)
```

We haven't talked about `._J`. It's a dual variable initialization technqiue discussed in Section 3.1 of [Lin *et al.*](https://people.eecs.berkeley.edu/~yima/matrix-rank/Files/rpca_algorithms.pdf).

The last part is to write an `__init__` function, add some docstrings and wrap everything up in a class.

```python
class RobustPCA:
    """
    Solves robust PCA using Inexact ALM as explained in Algorithm 5 of https://arxiv.org/pdf/1009.5055.pdf
    Parameters
    ----------
        lmb: 
            penalty on sparse errors
        mu_0: 
            initial lagrangian penalty
        rho: 
            learning rate
        tau:
            mu update criterion parameter
        max_iter:
            max number of iterations for the algorithm to run
        tol_rel:
            relative tolerance
        
    """
    def __init__(self, lmb: float, mu_0: float=1e-5, rho: float=2, tau: float=10, 
                 max_iter: int=10, tol_rel: float=1e-3):
        assert mu_0 > 0
        assert lmb > 0
        assert rho > 1
        assert tau > 1
        assert max_iter > 0
        assert tol_rel > 0
        self.mu_0_ = mu_0
        self.lmb_ = lmb
        self.rho_ = rho
        self.tau_ = tau
        self.max_iter_ = max_iter
        self.tol_rel_ = tol_rel
        
    def fit(self, X: np.ndarray):
        """
        Fits robust PCA to X and returns the low-rank and sparse components
        Parameters
        ----------
            X:
                Original data matrix

        Returns
        -------
            L:
                Low rank component of X
            S:
                Sparse error component of X
        """
        assert X.ndim == 2
        mu = self.mu_0_
        Y = X / self._J(X, mu)
        S = np.zeros_like(X)
        S_last = np.empty_like(S)
        for k in range(self.max_iter_):
            # Solve argmin_L ||X - (L + S) + Y/mu||_F^2 + (lmb/mu)*||L||_*
            L = svd_shrinkage(X - S + Y/mu, 1/mu)
            
            # Solve argmin_S ||X - (L + S) + Y/mu||_F^2 + (lmb/mu)*||S||_1
            S_last = S.copy()
            S = soft_thresholding(X - L + Y/mu, self.lmb_/mu)
            
            # Update dual variables Y <- Y + mu * (X - S - L)
            Y += mu*(X - S - L)
            r, h = self._get_residuals(X, S, L, S_last, mu)
            
            # Check stopping cirteria
            tol_r, tol_h = self._update_tols(X, L, S, Y)
            if r < tol_r and h < tol_h:
                break
                
            # Update mu
            mu = self._update_mu(mu, r, h)
            
        return L, S
            
    def _J(self, X: np.ndarray, lmb: float):
        """
        The function J() required for initialization of dual variables as advised in Section 3.1 of 
        https://people.eecs.berkeley.edu/~yima/matrix-rank/Files/rpca_algorithms.pdf            
        """
        return max(np.linalg.norm(X), np.max(np.abs(X))/lmb)
    
    @staticmethod
    def _get_residuals(X: np.ndarray, S: np.ndarray, L: np.ndarray, S_last: np.ndarray, mu: float):
        primal_residual = la.norm(X - S - L, ord="fro")
        dual_residual = mu*la.norm(S - S_last, ord="fro")
        return primal_residual, dual_residual
    
    def _update_mu(self, mu: float, r: float, h: float):
        if r > self.tau_ * h:
            return mu * self.rho_
        elif h > self.tau_ * r:
            return mu / self.rho_
        else:
            return mu
        
    def _update_tols(self, X, S, L, Y):
        tol_primal = self.tol_rel_ * max(la.norm(X), la.norm(S), la.norm(L))
        tol_dual = self.tol_rel_ * la.norm(Y)
        return tol_primal, tol_dual
        
```

## Complexity Analysis

All the norms and matrix additions/summations/multiplications are elementwise operations so that they're $\mathcal{O}(np)$ given that our matrix $X$ is an $n\times p$ matrix.
The major bottleneck of algorithms involving nuclear norm is that they typically require singular value thresholding which reqiuires SVD.
Since we're computing a skinny SVD, the complexity will be $\mathcal{O}(np\min(n,p))$.
SVD also requires to fit the entire data into the memory so it's inefficient in that sense too.
This can become a huge issue if you want to scale this algorithm and the literature has addressed this issue in certain ways which I hope to discuss in another post.

## Things to Try

The algorithm is ready to use, but here are a few suggestions I have for you to play around with the code a little and interact with it:

* Plot the interplay of `r`, `h` and `mu` over iterations to see the tug-of-war I mentioned earlier in action.
* Take one frame and record its evolution over the course of the algorithm. Especially observe how the sparse component for that frame `S[frame,:]` changes over time.
* You can estimate the rank of `L`

A very accesible dataset is the [cropped Yale B](http://vision.ucsd.edu/~leekc/ExtYaleDatabase/ExtYaleB.html) dataset where you have faces of different people taken under various lighting conditions.
Just pick one or two people and incldue all the illumination conditions they have to see if you can extract their clean face in the low-rank component.

