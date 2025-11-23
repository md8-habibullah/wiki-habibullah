---
title: Differential Equations (ODE) Reference
date:
description: Advanced notes on First/Second order ODEs, Laplace Transforms, and Numerical Methods.
tags:
  - math
  - calculus
  - university
  - latex
  - cs-theory
  - LLM
---

> [!abstract] The Language of Change
> A Differential Equation (DE) describes how a state changes over time.
> * **Algebra** solves for unknown numbers: $x^2 + 2x = 0$
> * **Differential Equations** solve for unknown *functions*: $\frac{dy}{dx} = ky$
>
> In Computer Science, we rarely solve these analytically (by hand). We solve them numerically using algorithms like **Euler's Method** or **Runge-Kutta**.

## 1. Classification & Definitions

**Order:** The highest derivative present in the equation.
* $y' + y = 0$ (First Order)
* $y'' + 3y' + 2y = 0$ (Second Order)

**Linearity:** An ODE is **linear** if the dependent variable $y$ and its derivatives $y', y''$ appear to the first power and are not multiplied together.
* Linear: $y'' + \sin(t)y = e^t$
* Non-Linear: $y'' + y^2 = 0$ (Because of $y^2$)

---

## 2. First Order ODEs ($n=1$)

### A. Separable Equations
The simplest form. We move all $y$'s to one side and all $x$'s to the other.
$$
\frac{dy}{dx} = g(x)h(y) \implies \int \frac{1}{h(y)} dy = \int g(x) dx
$$

### B. Linear Equations (Integrating Factor)
Standard form:
$$
\frac{dy}{dx} + P(x)y = Q(x)
$$

**Algorithm:**
1.  Calculate the **Integrating Factor**: $I(x) = e^{\int P(x) dx}$
2.  Multiply the entire equation by $I(x)$.
3.  The Left Hand Side (LHS) collapses via the Product Rule:
    $$
    \frac{d}{dx}[I(x)y] = I(x)Q(x)
    $$
4.  Integrate both sides and solve for $y$.

### C. Exact Equations
If $M(x, y)dx + N(x, y)dy = 0$ is "exact", then there exists a potential function $F(x,y)$ such that:
$$
\frac{\partial M}{\partial y} = \frac{\partial N}{\partial x}
$$
**Solution:** $F(x, y) = C$.

---

## 3. Second Order Linear ODEs ($n=2$)

Typically used to model springs, circuits, and vibrations.
Standard Form:
$$
ay'' + by' + cy = f(t)
$$

### Part 1: Homogeneous ($f(t) = 0$)
We assume the solution is of the form $y = e^{rt}$. This leads to the **Characteristic Equation**:
$$
ar^2 + br + c = 0
$$

Solve for roots $r_1, r_2$:
1.  **Distinct Real Roots** ($b^2 - 4ac > 0$):
    $$y(t) = c_1 e^{r_1 t} + c_2 e^{r_2 t}$$
2.  **Repeated Real Root** ($b^2 - 4ac = 0$):
    $$y(t) = c_1 e^{rt} + c_2 t e^{rt}$$
3.  **Complex Roots** ($r = \lambda \pm \mu i$):
    $$y(t) = e^{\lambda t} (c_1 \cos(\mu t) + c_2 \sin(\mu t))$$

### Part 2: Non-Homogeneous ($f(t) \neq 0$)
The general solution is $y(t) = y_h(t) + y_p(t)$.
* $y_h$: The homogeneous solution (from Part 1).
* $y_p$: The "Particular" solution.

**Method of Undetermined Coefficients:**
Guess the form of $y_p$ based on $f(t)$:
| Term in $f(t)$ | Guess for $y_p(t)$ |
| :--- | :--- |
| $Ae^{kt}$ | $Ce^{kt}$ |
| $\sin(kt)$ or $\cos(kt)$ | $A\cos(kt) + B\sin(kt)$ |
| Polynomial $t^n$ | $A_n t^n + \dots + A_0$ |

---

## 4. The Laplace Transform ($\mathcal{L}$)
The engineer's "Hack". It turns **Calculus** problems (derivatives) into **Algebra** problems.

**Definition:**
$$
F(s) = \mathcal{L}\{f(t)\} = \int_{0}^{\infty} e^{-st} f(t) dt
$$

**Key Properties:**
1.  **Linearity:** $\mathcal{L}\{af + bg\} = a\mathcal{L}\{f\} + b\mathcal{L}\{g\}$
2.  **Differentiation:** Turns derivatives into multiplication by $s$.
    $$\mathcal{L}\{y'(t)\} = sY(s) - y(0)$$
    $$\mathcal{L}\{y''(t)\} = s^2Y(s) - sy(0) - y'(0)$$

**Common Transforms Table:**
| $f(t)$ | $F(s)$ |
| :--- | :--- |
| $1$ | $1/s$ |
| $e^{at}$ | $1/(s-a)$ |
| $\sin(kt)$ | $k/(s^2 + k^2)$ |
| $\cos(kt)$ | $s/(s^2 + k^2)$ |
| $t^n$ | $n! / s^{n+1}$ |

> [!tip] Solving IVPs with Laplace
> 1. Take $\mathcal{L}$ of both sides of the ODE.
> 2. Solve algebraically for $Y(s)$.
> 3. Use Partial Fractions to decompose $Y(s)$.
> 4. Take the Inverse Laplace $\mathcal{L}^{-1}$ to find $y(t)$.

---

## 5. Numerical Methods (CS Approach) 💻

When an ODE is impossible to solve by hand (which is 99% of real-world cases), we approximate.

### Euler's Method
The simplest approach. We use the tangent line to step forward in time.
$$
y_{n+1} = y_n + h \cdot f(t_n, y_n)
$$
* $h$: Step size (e.g., 0.01).
* **Error:** $O(h)$ (Global error is proportional to step size).

### Runge-Kutta 4 (RK4)
The industry standard for simulations (Game physics, orbital mechanics). It samples the slope at 4 points to predict the next step.

$$
y_{n+1} = y_n + \frac{h}{6}(k_1 + 2k_2 + 2k_3 + k_4)
$$
Where:
* $k_1 = f(t_n, y_n)$
* $k_2 = f(t_n + \frac{h}{2}, y_n + \frac{h}{2}k_1)$
* $k_3 = f(t_n + \frac{h}{2}, y_n + \frac{h}{2}k_2)$
* $k_4 = f(t_n + h, y_n + hk_3)$

**Error:** $O(h^4)$. Much more precise than Euler.

---

## Linked Notes
* [[CPP-Ultimate-Guide]] - Implementing RK4 in C++.
* [[Linear-Algebra]] - Eigenvalues for Systems of ODEs.
* [[Physics-Simulation]] - Using ODEs for game engines.