---
layout: post
comments: false
title:  "Differential and Difference Equations"
excerpt: "Differential Equations at the heart of Computational Sciences and to solve them on a computer, we need to understand Difference Equations."
date:   2023-09-20 10:00:00

---

## Introduction
Consider a scenario where a *thin, insulated* metal rod is put on an *infinite block of ice at both ends* and a *blowtorch* is heating it along it's length (see Figure 1). The task is to find the temperature at several locations along the bar when it reaches steady state (i.e. when there's no change in temperature of the rod with time).

<div class="imgcap">
  <img src="https://raw.githubusercontent.com/tgautam03/Computational-Sciences/gh-pages/assets/2023-09-20-Differential-Difference-Equations/Figure1.png" alt="this slowpoke moves"  width="800"/>
  <div class="thecap">Figure 1: A thin metal rod with both ends attached to ice blocks and blowtorch heating it. </div>
</div>

> In the above description, we actually received almost all the assumptions for the problem:
> - *Thin*: It's just a 1D problem with probably $$x$$ used to denote the location along the length of the rod.
> - *Insulated*: No energy loss due to radiation from the surface of the rod to the surroundings.

The *Partial Differential Equation* (PDE) with boundary conditions explaining this scenario is known as the **Heat Equation** (Steady-State in this case). 

$$
-\frac{d^2u}{dx^2}=f(x) \ \ \forall \ 0 \le x \le 1; \ \ \ \ \ \ \ \ \ \ u(0)=u(1)=0 \tag{1}
$$

> Note that 
> - The temperature distribution along the length of the rod which ranges between 0 and 1 is given as $$u(x)$$,
> - The source term (*blowtorch*) is defined as $$f(x)$$ and,
> - The boundary conditions $$u(x)=u(1)=0$$ represent the *ice block* at the ends. 

> The complete Heat Equation is derived using conservation of heat energy and Fourier's Law of Heat Conduction, details to which can be found in my [YouTube Video]() or the [Blog Post]() on this topic.

### Why are *Boundary Conditions* so important?
For the time being, set $$f(x)=1$$. Now looking at the differential equation, what we need is some function $$u(x)$$ that satisfies $$-\frac{d^2u}{dx^2}=1$$. For this simple problem, we can integrate both sides twice and get a solution $$u(x)=-\frac{1}{2}x^2 + Cx + D$$ where $$C$$ and $$D$$ can be any constants! This is where boundary conditions $$u(0)=u(1)=0$$ fix the values of $$C$$, $$D$$ and gives us a unique solution $$u(x)=-\frac{1}{2}x^2 + \frac{1}{2}x$$.

### Finite Differences
In order to solve the PDE on a computer, we need to find a way to represent or approximate the derivatives numerically, which is done using Finite Differences. From *Taylor Series*, we know that 

$$
u(x+h) = u(x) + h u'(x) + \frac{h^2}{2!}u''(x) + \frac{h^3}{3!}u'''(x) + \cdots \tag{2}
$$

and 

$$
u(x-h) = u(x) - h u'(x) + \frac{h^2}{2!}u''(x) - \frac{h^3}{3!}u'''(x) + \cdots \tag{3}
$$

Rearranging Equation $$(2)$$, we can get the *Forward Difference Approximation* to the 1st order derivative

$$
\begin{aligned}
\frac{u(x+h) -  u(x)}{h} & = u'(x) + \frac{h}{2!}u''(x) + \frac{h^2}{3!}u'''(x) + \cdots \\
                        & = u'(x) + \mathcal{O}(h) 
\end{aligned} \tag{4}
$$

Similarly from Equation $$(3)$$, we can get the *Backward Difference Approximation* to the 1st order derivative

$$
\begin{aligned}
\frac{u(x) -  u(x-h)}{h} & = u'(x) - \frac{h}{2!}u''(x) + \frac{h^2}{3!}u'''(x) + \cdots \\
                        & = u'(x) + \mathcal{O}(h) 
\end{aligned} \tag{5}
$$

Finally, subtracting Equation $$(3)$$ from Equation $$(2)$$, we can get the *Centered Difference Approximation* to the 1st order derivative

$$
\begin{aligned}
\frac{u(x+h) -  u(x-h)}{2h} & = u'(x) + \frac{2h^2}{3!}u'''(x) + \cdots \\
                        & = u'(x) + \mathcal{O}(h^2) 
\end{aligned} \tag{6}
$$

> *Centered Difference* is 2nd order Accurate in $$h$$ while others are 1st order Accurate in $$h$$!

We can also add Equations $$(2)$$ and $$(3)$$, to get an approximation to 2nd order derivative

$$
\begin{aligned}
\frac{u(x+h) - 2u(x) +  u(x-h)}{h^2} & = u''(x) + \frac{2h^2}{4!}u''''(x) + \cdots \\
                        & = u''(x) + \mathcal{O}(h^2) 
\end{aligned} \tag{7}
$$

> 2nd order derivative is by default 2nd order accurate in $$h$$!

## Difference Equation
The next step is to use the above defined equations $$(2)$$ through $$(7)$$ and convert the *Differential Equation* to a *Difference Equation*. For this, we need **numerical descretisation**. 

### Numerical Discretisation
Considering the PDE in Equation $$(1)$$, where the domain lies between 0 and 1. What discretisation does is represent this continuous domain using a finite set of points. As shown in Figure 2, I'm discretising the domain into 5 equispaced points with distance $$h$$ between the two adjacent points. 

### Solution