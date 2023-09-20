---
layout: post
comments: false
title:  "Four Special Matrices"
excerpt: "Matrices are the basic data structures for computational sciences, and there are a few special matrices that pop in very often in several applications."
date:   2023-09-11 10:00:00

---

## Introduction
Consider a matrix, $$K_4=\begin{pmatrix}2 & -1 & 0 & 0\\\ -1 & 2 & -1 & 0 \\\ 0 & -1 & 2 & -1 \\\ 0 & 0 & -1 & 2 \\\ \end{pmatrix}$$. 

What are the special properties of this matrix?
- **Symmetric**: $$K=K^T$$
- **Sparse (Tridiagonal)**: The point on sparsity is especially true when matrix K is something like $$100 \times 100$$
- **Toeplitz**: Constant diagonals, i.e. we have $$2$$ and $$-1$$ across the diagonals
- **Invertible**: Identifying that a matrix is invertible, is crucial to solving a system of equations. The standard way to find out if a matrix is invertible, is to row reduce it and represent it in upper triangular form. Let's see how to do that.


## Row Reduction
Using the same $$K_4$$ matrix discussed in the previous section,

- Picking the first column, where the pivot is $$k_{11}$$ element. We make all the elements below the pivot, zero by performing row reduction operation $$R_2 \rightarrow R_2 - (k_{11}/k_{21})R_1$$.

$$K_4=\begin{pmatrix} 2 & -1 & 0 & 0\\ 0 & 3/2 & -1 & 0 \\ 0 & -1 & 2 & -1 \\ 0 & 0 & -1 & 2 \\ \end{pmatrix}$$

- Next, move onto the 2nd column and again ensure that all elements below the pivot $$k_{22}$$ are zero by performing the row reduction operation $$R_3 \rightarrow R_3 - (k_{22}/k_{32})R_2$$.

  $$K_4= \begin{pmatrix} 2 & -1 & 0 & 0\\ 0 & 3/2 & -1 & 0 \\ 0 & 0 & 4/3 & -1 \\ 0 & 0 & -1 & 2 \\ \end{pmatrix}$$

- Do the same thing for the 3rd column, $$R_4 \rightarrow R_4 - (k_{33}/k_{43})R_3$$.

  $$K_4= \begin{pmatrix} 2 & -1 & 0 & 0\\ 0 & 3/2 & -1 & 0 \\ 0 & 0 & 4/3 & -1 \\ 0 & 0 & 0 & 5/4 \\ \end{pmatrix}$$

- For the final column, there's nothing below the pivot value.

We now have $$K_4$$ reduced to the upper triangular form and the simple test of invertibility is to check if all pivots are non-zero. If they're non-zero, then the matrix is invertible.

<!-- {: .note-title } -->
<!-- {: .info } -->
{: .warn }
> Don't touch determinants when dealing with invertibility! Any numerical methods library will evaluate determinants this way by first converting the matrix into an upper triangular form and then taking the product of the diagonal elements. In this case, the determinant is $$5$$!




## References
- [Jaan's blog post](https://jaantollander.com/post/how-to-create-software-packages-with-julia-language/)