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

> [!WARNING]
> Don't touch determinants when dealing with invertibility! Any numerical methods library will evaluate determinants this way by first converting the matrix into an upper triangular form and then taking the product of the diagonal elements. In this case, the determinant is $$5$$!


## GitHub Actions
**GitHub Actions** can automatically run tests and documenter everytime I push code to the GitHub Repository. 

To do this, first create *.github/* folder with *Documenter.yml* and *Runtests.yml* files (these are the config files for GitHub actions):

*Documenter.yml*
```yml
name: Documenter
on:
  push:
    branches:
      - master
    tags: '*'
  pull_request:
jobs:
  build:
    runs-on: ubuntu-latest # OS Support
    steps:
      - uses: actions/checkout@v2
      - uses: julia-actions/setup-julia@latest
        with:
          version: '1.8.3' # Julia Version
      - name: Install dependencies
        run: julia --project=docs/ -e 'using Pkg; Pkg.develop(PackageSpec(path=pwd())); Pkg.instantiate()'
      - name: Build and deploy
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }} # For authentication with GitHub Actions token
          DOCUMENTER_KEY: ${{ secrets.DOCUMENTER_KEY }} # For authentication with SSH deploy key
        run: julia --project=docs/ docs/make.jl
```

*Runtests.yml*
```yml
name: Runtests
on: [push, pull_request]
jobs:
  test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        julia-version: ['1.8.3'] # Julia Version
        julia-arch: [x64] # 64 bit Architecture Support
        os: [macOS-latest, ubuntu-latest] # OS Support
    steps:
      - uses: actions/checkout@v2
      - uses: julia-actions/setup-julia@latest
        with:
          version: ${{ matrix.julia-version }}
      - uses: julia-actions/julia-buildpkg@latest
      - uses: julia-actions/julia-runtest@latest
```

Package Structure:
```
jac/
├─ .git/
├─ .github/
│  └─ workflows/
│     ├─ Documenter.yml
│     └─ Runtests.yml
├─ docs/
│  ├─ src/
│  │  ├─ Development
│  │  |     ├─
│  |  ├─ Examples
│  │  |     ├─
│  │  └─ index.md
│  ├─ Project.toml
│  └─ make.jl
├─ src/
│  └─ jac.jl
├─ test/
│  └─ runtests.jl
├─ .gitignore
├─ LICENSE
├─ Project.toml
└─ README.md
```

Now, I have to deploy keys for GitHub Actions. First, generate the keys using `DocumenterTools` package:
```sh
julia> DocumenterTools.genkeys()
┌ Info: add the public key below to https://github.com/$USER/$REPO/settings/keys with read and write access
└ the title can be left empty as GitHub can infer it from the key comment

ssh-rsa WILL HAVE KEY HERE Documenter

[ Info: add a secure environment variable named 'DOCUMENTER_KEY' to https://travis-ci.com/$USER/$REPO/settings (if you deploy using Travis CI) or https://github.com/$USER/$REPO/settings/secrets (if you deploy using GitHub Actions) with value:

WILL HAVE KEY HERE
```

Follows the next steps:
- Copy the ssh key (starting with `ssh-rsa`) in **Settings/Deploy keys** on the GitHub repository's website (figure below for details)

    <div class="imgcap">
    <img src="https://raw.githubusercontent.com/tgautam03/tgautam03.github.io/master/assets/2022-12-23-Julia_packages/DocumenterKey.png" alt="this slowpoke moves"  width="800"/>
    <div class="thecap">Figure 1: Adding Documenter Key to GitHub.</div>
    </div>

- Copy the secret key in **Settings/Secrets/Actions** on the GitHub repository's website (figure below for details)

    <div class="imgcap">
    <img src="https://raw.githubusercontent.com/tgautam03/tgautam03.github.io/master/assets/2022-12-23-Julia_packages/Secret.png" alt="this slowpoke moves"  width="800"/>
    <div class="thecap">Figure 2: Adding Secret Key to GitHub.</div>
    </div>

One last thing before pushing the changes to remote repository, we have to modify *docs/make.jl* file to include the URL to the GitHub repository:
```julia
using Documenter
using jac

makedocs(
    sitename = "jac",
    format = Documenter.HTML(),
    modules = [jac]
)

# Documenter can also automatically deploy documentation to gh-pages.
# See "Hosting Documentation" and deploydocs() in the Documenter manual
# for more information.
deploydocs(
    repo = "github.com/tgautam03/jac.git" # URL to Repo
)
```

Now, push the changes to GitHub.

## Final Touches
In the remote repository, I can now see the Tests and Documenter running inside *Actions* tab. I can also add **Badges** so that I don't have to look inside *Actions* tab every time to check the status (this is done inside *Actions/Documenter* and *Actions/Runtests* using triple dots on the top right corner).

Once the documentation is deployed, I can check the URL on GitHub Repo itself (in my case it's https://tgautam03.github.io/jac/dev/).


## References
- [Jaan's blog post](https://jaantollander.com/post/how-to-create-software-packages-with-julia-language/)