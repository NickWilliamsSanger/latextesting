Introduction to rtreefit
================
29/01/2025

<!-- README.md is generated from README.Rmd. Please edit that file -->

## Installation

<!-- badges: start -->
<!-- badges: end -->

You can install rtreefit like so:

``` r
devtools::install_github("nangalialab/rtreefit",build_vignettes=TRUE)
```

## Introduction

This package estimates time-based (“ultrametric”) trees, wherein the
y-axis of phylogenetic somatic mutation trees is converted from
mutations to time. The method jointly fits wild type rates, mutant rates
and absolute time branch lengths using a Bayesian per individual
tree-based model under the assumption that the observed branch lengths
are Poisson or Negative Binomial distributed with

Mean = Duration × Sensitivity × Mutation Rate

The method works with at most one change point per branch and supports
heterochronous sampling. See the rtreefit vignette for slightly fuller
mathematical details:

``` r
browseVignettes("rtreefit") 
```

## Branch Timings and Per Driver Clade Mutation Rate

We consider a rooted tree where each edge $i$ consists of an observed
mutation count $m_i$ and a true duration $t_i$. We refer to a given edge
and its child node interchangeably by the same label. Now let $D(i)$ be
the set of terminal nodes (tips) that descend from node $i$ and let
$A(i)$ be its corresponding set of ancestral nodes excluding the root.
We assume that each tip of the tree $k$ has a known corresponding time
$T_k$ (e.g. the post conception age in years of the patient at sampling
of the cell) and so we therefore have the following constraint:

$$ T_k=\sum_{i \in A(k)}t_i $$

and

$$ T_k> t_i > 0 $$

We incorporate this constraint by performing the optimisation over the
interior branches of the tree with reparameterised branch durations
$x_i$ transformed to be in the range $0< x_i <1$. If $j$ is an edge
whose parent node is the root then:

$$ t_j=x_j  \text{min}({T_k:k \in D(j)}) $$

For other interior edges, $i$, we have

$$ t_i=\left(\text{min}\left(T_k:k \in D(i)\right) -\sum_{j\in A(i)} t_j\right)x_i $$

The duration of the terminal edges is fixed by the values of $t_i$ on
the interior edges and the overall duration constraint:

$$ t_i=\text{min}\left(T_k:k \in D(i)\right)-\sum_{j\in A(i)} t_j $$

We assume that there are $p-1$ change points in the tree corresponding
to the acquisition of driver mutations. This results in $p$ mutation
rates $\lambda_j$ applying throughout the tree where we allow at most
one change point per branch and the initial ancestral (or wild type)
rate is $\lambda_0$ and additional rate change points occur a fraction
$\alpha_j$ along branch $j$ and descendent branches have the rate
$\lambda_j$ unless there are additional change points in descendant
branches. The effective rate on branches with a change point going from
$\lambda_l$ to $\lambda_j$ is just the weighted average
$\alpha_j \lambda_l+(1-\alpha_j)\lambda_j$ where we use a uniform unit
interval prior for the $\alpha$’s.

### Negative Binomial Model

We assume the underlying mutation process follows a Negative Binomial
Distribution with the above piecewise constant driver specific mutation
rates, the number of mutations accrued on branch $i$ in time $t_i$
measured in years:

$$ M_i \sim \text{NB}\left(\lambda\times t_i,\lambda\times t_i\times \phi\right) $$

The number of observed mutations is:

$$ m_i \sim \text{Binomial}(M_i,s_i) $$

Where we are using a per-branch estimated sensitivity $s_i$ that
indirectly depends on the depth of sample and the number of samples
sharing a branch (see ?). This is equivalent too:

$$ m_i \sim \text{NB}\left(\lambda\times t_i \times s_i,\lambda\times t_i\times \phi\right) $$

with priors $1/\phi \sim \text{HalfNormal}(0,10)$,
$\lambda \sim \mathcal{N}(\Lambda,0.25 \Lambda)$ where $\Lambda$ is the
naive estimation of a single rate $\lambda$ as the per patient median of
the ratio of the root to tip mutation count and the tip sampling age,
and finally we use the weakly informative prior for the stick breaking
fractions:

$$x_i \sim \text{Beta}(\alpha=p_i/(1-\sum_{j\in A(i)}p_j),\beta=1)$$

where the $p_i$ is an initial approximation of the duration of the
branch length expressed as a fraction of the sampling time:

$$ p_i=\text{min}\left(j\in D(i)\right) \frac{m_j+1}{\Sigma_{k\in A(j)}(m_k+1)} $$

Note that the overdispersion parameter is rescaled so that it is
comparable across branches with different mutation burden.

### Poisson Model

Here we assume the underlying mutation process follows a Poisson
Distribution again with the above piecewise constant driver specific
mutation rates, the number of observed mutations accrued on branch $i$
in time $t_i$ measured in years:

$$ m_i \sim \text{Poisson}(\lambda\times t_i\times S_i) $$

where

$$ S_i \sim \text{Beta}\left(\alpha=c,\beta=c(1-s_i)/s_i\right)$$

Where we have chosen the concentration parameter $c=100$. This reflects
only modest uncertainty in our estimates in sensitivity and also allows
the model to mitigate larger than expected variability in the branch
lengths. In other respects the priors are the same as for the Negative
Binomial Model.
