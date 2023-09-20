---
layout: post
comments: false
title:  "Automatic differentiation (forward mode)"
excerpt: "Forward mode Automatic differentiation from scratch."
date:   2021-11-20 23:00:00
---

## Introduction
From Taylor Series we know that $$f(x+h)=f(x)+hf^{'}(x)$$ (setting $$h^2=0$$). This statement gives us a way to represent a function by it's value and derivative:

$$f(x)\rightarrow f(x)+\epsilon f^{'}(x)$$

> The idea is very similar to the one of **Complex Step Finite Difference** where the value and the chance is stored separately, i.e. *Dual Numbers*.

## Dual Numbers
Thus, to extend the idea of complex step differentiation beyond complex analytic functions, we define a new number type, the dual number. A dual number is a multidimensional number where the sensitivity of the function is propagated along the dual portion.

A struct can be used to track both value and the gradient.

```julia
struct Dual{T}
    val::T
    grad::T
end
```

Let's consider two functions:

$$f(x)\rightarrow f(x)+\epsilon f^{'}(x)$$

$$g(x)\rightarrow g(x)+\epsilon g^{'}(x)$$

### Addition Rule
Adding the above two functions give:

$$(f+g)(x)\rightarrow f(x)+g(x)+\epsilon \big[f^{'}(x)+g^{'}(x)\big]$$

This can be easily coded by overriding `+` operator in julia such that it collects gradients as well whenever `Dual` type is used.

```julia
Base.:+(f::Dual, g::Dual) = Dual(f.val+g.val, f.grad+g.grad)
Base.:+(f::Dual, g::Real) = Dual(f.val+g, f.grad)
Base.:+(f::Real, g::Dual) = Dual(f+g.val, g.grad)
```

> Similar rules can be applied to other basic operations.

### Subtraction Rule

$$(f-g)(x)\rightarrow f(x)-g(x)+\epsilon \big[f^{'}(x)-g^{'}(x)\big]$$

```julia
Base.:-(f::Dual, g::Dual) = Dual(f.val-g.val, f.grad-g.grad)
Base.:-(f::Dual, g::Real) = Dual(f.val-g, f.grad)
Base.:-(f::Real, g::Dual) = Dual(f-g.val, g.grad)
```

### Multiplication Rule

$$(f \cdot g)(x)\rightarrow f(x)g(x)+\epsilon \big[f(x)g^{'}(x)+f^{'}(x)g(x)\big]$$

```julia
Base.:*(f::Dual, g::Dual) = Dual(f.val*g.val, f.val*g.grad+f.grad*g.val)
Base.:*(f::Dual, g::Real) = Dual(f.val*g, f.grad*g)
Base.:*(f::Real, g::Dual) = Dual(f*g.val, f*g.grad)
```

> Note that $$\epsilon^2=0$$

### Division Rule

$$(\frac{f}{g})(x)\rightarrow \frac{f(x)}{g(x)}+\epsilon \bigg[\frac{f^{'}(x)g(x)-f(x)g^{'}(x)}{g(x)^2}\bigg]$$

```julia
Base.:/(f::Dual, g::Dual) = Dual(f.val/g.val, (f.grad*g.val-f.val*g.grad)/g.val^2)
Base.:/(f::Dual, g::Real) = Dual(f.val/g, (f.grad*g)/g^2)
Base.:/(f::Real, g::Dual) = Dual(f/g.val, (-f*g.grad)/g.val^2)
```

> If we have been paying attention, the term inside the $$\epsilon$$ basically contains the derivative rules that we know from our childhood!!

### Exponential Operator
We know that $$x^3$$ basically means $$((x\times x)\times x)$$, so we can exploit the **multiplication rule** to define this.

```julia
Base.:^(f::Dual, g::Real) = Base.power_by_squaring(f, g)
```

## Differential Equations
Suppose we have a function $$f(x)=x^2+2x$$. From chain rule, we can write $$\frac{\partial f}{\partial x}=\frac{\partial f}{\partial x} \times \frac{\partial x}{\partial x}$$ where $$\frac{\partial x}{\partial x}=1$$. Hence to get the *automatic differentiation* to supply $$\frac{\partial f}{\partial x}$$, we need to define the input $$x$$ as a `Dual` number, i.e. `Dual(x, 1)`, where `x` is the value at which we want to compute gradient.

Let's evaluate gradient at $$x=2$$.

```julia
h(x) = x^2 + 2*x
x = Dual(2,1)
println("Function Value: ", h(x).val)
println("Function Gradient: ", h(x).grad)
```
Above code returns:
```text
Function Value: 8
Function Gradient: 6
```

### What's the catch?
We're getting both the function value and the gradient together. There has to be a hit on performance? 

Using `BenchmarkTools` and evaluating for the scalar input, we complete the execution in 0.016 ns.
```julia
# Normal Function eval
@btime _ = h(2);
```
```text
0.016 ns (0 allocations: 0 bytes)
```

Let's see what we get when `Dual` number is given as the input.
```julia
# Val + grad eval
@btime _ = h(Dual(2,1));
```
```text
0.016 ns (0 allocations: 0 bytes)
```

It's exactly the same and **Vectorisation** can explain these findings. 

> Modern CPUs have Vector Registers hence `value` and `grad` are evaluated in parallel. This can be easily seen by looking at the assembly code (notice commands like `vmovdqu` and `vpaddq`).
```assembly
.text
; ┌ @ In[8]:1 within `h'
	pushq	%r14
	pushq	%rbx
	subq	$24, %rsp
	movq	%rsi, %rbx
	movq	%rdi, %r14
; │┌ @ none within `literal_pow'
; ││┌ @ none within `macro expansion'
; │││┌ @ In[6]:1 within `^'
	movabsq	$power_by_squaring, %rax
	movq	%rsp, %rdi
	movl	$2, %edx
	callq	*%rax
; │└└└
; │┌ @ In[4]:3 within `*' @ int.jl:88
	vmovdqu	(%rbx), %xmm0
	vpaddq	%xmm0, %xmm0, %xmm0
; │└
; │┌ @ In[2]:1 within `+' @ int.jl:87
	vpaddq	(%rsp), %xmm0, %xmm0
; │└
	vmovdqu	%xmm0, (%r14)
	movq	%r14, %rax
	addq	$24, %rsp
	popq	%rbx
	popq	%r14
	retq
	nop
; └
```

## Higher Dimensions
Suppose we have functions $$f(x, y) = x^2 + xy$$ and $$g(x, y) = y^3 + x$$ and we want to compute jacobian $$J = 
\begin{bmatrix} 
\frac{\partial f}{\partial x} & \frac{\partial f}{\partial y}\\
\frac{\partial g}{\partial x} & \frac{\partial g}{\partial y}
\end{bmatrix}$$ at $$ $x=3, y=4$$.

This can be done by evaluating all four elements separately as follows:
```julia
ff(x, y) = x^2 + x*y
gg(x, y) = y^3 + x

df_dx = ff(Dual(3, 1), Dual(4, 0)).grad
df_dy = ff(Dual(3, 0), Dual(4, 1)).grad

dg_dx = gg(Dual(3, 1), Dual(4, 0)).grad
dg_dy = gg(Dual(3, 0), Dual(4, 1)).grad


J = zeros(2, 2)
J[1,1] = df_dx
J[1,2] = df_dy
J[2,1] = dg_dx
J[2,2] = dg_dy

println("Jacobian: ")
show(stdout, "text/plain", J)
```
```text
Jacobian: 
2×2 Matrix{Float64}:
 10.0   3.0
  1.0  48.0
```

Hence, for $$f(x_1, x_2, \cdots, x_n)$$, we'll have to do differentiation `n` times. Let's see if we can relax this condition a bit.

We know that $$\nabla f = \left[\begin{array}{ccc}
\dfrac{\partial f(x,y)}{\partial x} & \dfrac{\partial f(x,y)}{\partial y}
\end{array}\right]$$, hence what if we could do something like `df = ff(X).grads where (X=[x y])` and `df=[ff(Dual(3, 1), Dual(4, 0)).grad, ff(Dual(3, 0), Dual(4, 1)).grad]`, i.e. utilise the **vector instructions** to calculate each partial derivative in parallel?

This is where we need to define a new type, **MultiDual**.

### MultiDual

```julia
struct MultiDual{N,T}
	val::T
	# SVector is static vector which lives on the stack
	grads::SVector{N,T} 
end
```
In `MultiDual`, `N` defines the number of variables in the function and `T` defines the type (`Int`, `Float`, etc. etc.). 

Various rules can be defined using Taylor Series (like before), i.e. if $$x$$ is a vector then Taylor Series is defined as 

$$f(x+\epsilon)=f(x)+\epsilon \nabla f(x)+O(\epsilon)$$

> Only change is that $$f^{'}$$ is replaced by $$\nabla f$$. Same thing goes for various differentiation rules.

#### **Addition Rule**
$$(f+g)(x)\rightarrow f(x)+g(x)+\epsilon \big[\nabla f(x)+\nabla g(x)\big]$$

```julia
function +(f::MultiDual{N,T}, g::MultiDual{N,T}) where {N,T}
    return MultiDual{N,T}(f.val+g.val, f.grads+g.grads)
end
```

#### **Subtraction Rule**
$$(f-g)(x)\rightarrow f(x)-g(x)+\epsilon \big[\nabla f(x)-\nabla g(x)\big]$$

```julia
function -(f::MultiDual{N,T}, g::MultiDual{N,T}) where {N,T}
    return MultiDual{N,T}(f.val-g.val, f.grads-g.grads)
end
```

#### **Multiplication Rule**
$$(f \cdot g)(x)\rightarrow f(x)g(x)+\epsilon \big[f(x)\nabla g(x)+\nabla f(x)g(x)\big]$$

```julia
function *(f::MultiDual{N,T}, g::MultiDual{N,T}) where {N,T}
    return MultiDual{N,T}(f.val*g.val, f.val*g.grads+f.grads*g.val)
end
```

> Note we set $$\epsilon^2=0$$

> The term in the $$\epsilon\big[\cdot \cdot \cdot\big]$$ contains the calculus rules that we remember from our childhood!!

#### **Division Rule**
$$(\frac{f}{g})(x)\rightarrow \frac{f(x)}{g(x)}+\epsilon \bigg[\frac{\nabla f(x)g(x)-f(x)\nabla g(x)}{g(x)^2}\bigg]$$

```julia
function /(f::MultiDual{N,T}, g::MultiDual{N,T}) where {N,T}
    return MultiDual{N,T}(f.val/g.val, (f.grads*g.val-f.val*g.grads)/g.val^2)
end
```

#### **Exponential operator**

```julia
function ^(f::MultiDual{N,T}, g::Real) where {N,T}
    return Base.power_by_squaring(f,g)
end
```

### Calculating efficient Jacobian
Let's now compute the Jacobian using `MultiDual`. The two functions are 

$$f(x, y) = x^2 + xy$$ 

$$g(x, y) = y^3 + x$$

The next step is to define `MultiDual` functions for `x` and `y`. 
- Let a function $$xx(x,y)=x + 0 \times y$$. `MultiDual` `xx` can be written as `xx = MultiDual(x, SVector(1.,0.))`. `SVector(1.,0.)` is written because $$\frac{\partial xx}{x} = 1$$ and $$\frac{\partial xx}{y} = 0$$, hence these are set as the first and second elements of the vector.
- Similar for $$yy(x,y)=y + 0 \times x$$, `yy=MultiDual(y, SVector(0.,1.))`.

Jacobian can then easily be calculated. Complete code shown below.
```julia
ff(x, y) = x^2 + x*y
gg(x, y) = y^3 + x

# Jacobian at x=3 and y=4
xx = MultiDual(3., SVector(1.,0.))
yy = MultiDual(4., SVector(0.,1.))

println("Jacobian: ", ff(xx, yy).grads, gg(xx, yy).grads)
```
```text
Jacobian: [10.0, 3.0][1.0, 48.0]
```

### Performace

Let's verify if this actually makes out calculations fast.

- **`Dual`**
	```julia
	# Jacobian using Dual
	function jacobian(ff, gg, x::Real, y::Real)
		return ff(Dual(x, 1), Dual(y, 0)).grad, 
		ff(Dual(x, 0), Dual(y, 1)).grad,
		gg(Dual(x, 1), Dual(y, 0)).grad,
		gg(Dual(x, 0), Dual(y, 1)).grad
	end

	@btime _ = jacobian(ff, gg, 3, 4);
	```
	```text
	11.298 ns (0 allocations: 0 bytes)
	```

- **`MultiDual`** 
	```julia
	# Jacobian using MultiDual
	function jacobianVec(ff, gg, x::Real, y::Real)
		return ff(MultiDual(x, SVector(1.,0.)), MultiDual(y, SVector(0.,1.))).grads,
		gg(MultiDual(x, SVector(1.,0.)), MultiDual(y, SVector(0.,1.))).grads
	end

	@btime _ = jacobianVec(ff, gg, 3., 4.);
	```
	```text
	6.530 ns (0 allocations: 0 bytes)
	```

> As expected, `MultiDual` is almost twice as fast as `Dual`.

## Conclusion
- Forward mode AutoDiff calculates derivatives along with the function values independently.
- Vectorisation is crucial to get high performance from the AutoDiff code.

## References
- Programming Language: [Julia Programming Language](https://julialang.org/)
- Example Code: [Jupyter Notebook](https://github.com/tgautam03/lucidgrad/blob/main/nb/01_AutomaticDifferentiation.ipynb)
- Awesome Youtube Lecture: [Forward-Mode Automatic Differentiation (AD) via High Dimensional Algebras](https://www.youtube.com/watch?v=zHPXGBiTM5A&list=PLCAl7tjCwWyGjdzOOnlbGnVNZk0kB8VSa&index=13&t=1312s)



