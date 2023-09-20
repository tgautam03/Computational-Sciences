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

## Circulant Matrix
Consider another matrix similar to $$K_4$$ (discussed in [[Introduction to Four Special Matrices]]), such that the value $$-1$$ wraps around the row, Let's call this matrix a circulant matrix and denote this as, 

$$C_4=\begin{pmatrix}2 & -1 & 0 & -1\\-1 & 2 & -1 & 0 \\0 & -1 & 2 & -1 \\-1 & 0 & -1 & 2 \\\end{pmatrix}$$

Matrix $$C_4$$ shares many properties with $$K_4$$ such as:
- **Symmetric**
- **Sparse (not Tridiagonal though)**
- **Toeplitz**

However, this matrix is not invertible! We can do the row reduction to verify this but there is another way to see this. If we multiply $$C_4$$ with ones vector as shown in equation $$(1)$$, we get all zeros as the resulting vector and by definition a matrix is invertible if and only if it has zero vector in its null space!

$$  
\begin{pmatrix} 2 & -1 & 0 & -1\\ -1 & 2 & -1 & 0 \\ 0 & -1 & 2 & -1 \\ -1 & 0 & -1 & 2 \\ \end{pmatrix} \begin{pmatrix} 1 \\ 1 \\ 1 \\ 1 \\ \end{pmatrix}= \begin{pmatrix} 0 \\ 0 \\ 0 \\ 0 \\ \end{pmatrix} \tag{1} 
$$

We can write the above statement mathematically as well. Consider $$Cu=0$$ and if $$C^{-1}$$ exists, then $$C^{-1}Cu=C^{-1}0$$ and $$u=0$$. However, we have $$u=1$$ hence it's not invertible!

## Practical Origin of Matrices
- Consider a system of $$4$$ masses connected by springs (as shown below), and the matrix generated when trying to solve this system is $$K_4$$.
	<div class="imgcap">
  <img src="https://raw.githubusercontent.com/tgautam03/Computational-Sciences/gh-pages/assets/2023-09-11-Four-Special-Matrices/K4.png" alt="this slowpoke moves"  width="800"/>
  <div class="thecap">Figure 1: Spring Mass System with fixed ends. </div>
  </div>

- Now, if we remove the fixed ends and just tie the masses together, we get matrix $$C_4$$. This is where the term circulant comes from.
	<div class="imgcap">
  <img src="https://raw.githubusercontent.com/tgautam03/Computational-Sciences/gh-pages/assets/2023-09-11-Four-Special-Matrices/C4.png" alt="this slowpoke moves"  width="800"/>
  <div class="thecap">Figure 2: Spring Mass System with ends atached to each other in circular fashion. </div>
  </div>

- Just as an example, take the spring mass system which gives $$K_4$$ and free one end of support.
	<div class="imgcap">
  <img src="https://raw.githubusercontent.com/tgautam03/Computational-Sciences/gh-pages/assets/2023-09-11-Four-Special-Matrices/T4.png" alt="this slowpoke moves"  width="800"/>
  <div class="thecap">Figure 3: Spring Mass System with one free end. </div>
  </div>
	
  In doing so, we get a new matrix $$T_4=\begin{pmatrix}1 & -1 & 0 & 0\\-1 & 2 & -1 & 0 \\0 & -1 & 2 & -1 \\0 & 0 & -1 & 2 \\\end{pmatrix}$$.

- For the sake of completeness, let's free both ends.
	<div class="imgcap">
  <img src="https://raw.githubusercontent.com/tgautam03/Computational-Sciences/gh-pages/assets/2023-09-11-Four-Special-Matrices/B4.png" alt="this slowpoke moves"  width="800"/>
  <div class="thecap">Figure 4: Spring Mass System with both free ends. </div>
  </div>
	
  Now, we get another new matrix $$B_4=\begin{pmatrix}1 & -1 & 0 & 0\\-1 & 2 & -1 & 0 \\0 & -1 & 2 & -1 \\0 & 0 & -1 & 1 \\\end{pmatrix}$$.

All these four special matrices are **Symmetric** and **Sparse**. When it comes to invertibility, $$K$$ and $$T$$ are invertible, but $$C$$ and $$B$$ aren't. A physics intuition is that the systems with some sort of support (fixed end in this case) results in invertible matrices while the unsupported systems give non-invertible matrices.


>$$T$$ and $$B$$ are not Toeplitz!

>Another thing is, $$K$$ and $$T$$ are positive definite (pivots are all positive in upper triangular form) while $$C$$ and $$B$$ are positive semi-definite (pivots are zeros or positive in upper triangular form).

## References
- [MITOCW 18.085 Computational Science and Engineering 1](https://ocw.mit.edu/courses/18-085-computational-science-and-engineering-i-fall-2008/)