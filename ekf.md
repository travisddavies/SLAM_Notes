# Extended Kalman Filter
## SLAM is a State Estimation Problem
- Estimate the map and robot's pose
- Bayes filter is one tool for state estimation
- **Prediction**
$$
\overline{bel}(x_t) = \int p(x_t | u_t, x_{t-1})bel(x_{t-1})dx_{t-1}
$$
- **Correction**
$$
bel(x_t) = \eta p(z_t|x_t)\overline{bel}(x_t)
$$

## Kalman Filter
- It is a Bayes filter
- Estimator for the linear Gaussian case
- Optimal solution for linear models and Gaussian distributions

## Kalman Filter Distribution
- Everything is Gaussian
$$
p(x) = \det(2\pi \Sigma)^{-\frac{1}{2}} \exp(-\frac{1}{2}(x - \mu)^T \Sigma^{-1}(x-\mu))
$$
![](kalman-filter-distribution.png)

## Properties: Marginalisatin and Conditioning
- Given
$$
x =
\begin{pmatrix}
x_a \\
x_b
\end{pmatrix}
\quad
p(x) = \mathcal{N}
$$
- The marginals are Gaussians
$$
p(x_a) = \mathcal{N} \quad p(x_b)= \mathcal{N}
$$
- As well as the conditionals
$$
p(x_a | x_b) = \mathcal{N} \quad p(x_b | x_a) = \mathcal{N}
$$

## Marginalisation
- Given 
$$
p(x) = p(x_a, x_b) = \mathcal{N}(\mu, \Sigma)
$$
with
$$
\mu = \begin{pmatrix}\mu_a \\ \mu_b \end{pmatrix} \quad \Sigma = \begin{pmatrix} \Sigma_{aa} & \Sigma_{ab} \\ \Sigma_{ba} & \Sigma_{bb} \end{pmatrix}
$$
- The marginal distribution is 
$$
p(x_a) = \int p(x_a, x_b) dx_b = \mathcal{N}(\mu, \Sigma)
$$
with
$$
\mu = \mu_a \quad \Sigma = \Sigma_{aa}
$$
- This is just your simple marginalisation operation
## Conditioning
- Given 
$$
p(x) = p(x_a, x_b) = \mathcal{N}(\mu, \Sigma)
$$
with
$$
\mu = \begin{pmatrix}\mu_a \\ \mu_b \end{pmatrix} \quad \Sigma = \begin{pmatrix} \Sigma_{aa} & \Sigma_{ab} \\ \Sigma_{ba} & \Sigma_{bb} \end{pmatrix}
$$
- The conditional distribution is
$$
p(x_a|x_b) = \frac{x_a, x_b}{p(x_b)} = \mathcal{N}(\mu, \Sigma)
$$
with
$$
\mu = \mu_a + \Sigma_{ab}\Sigma^{-1}_{bb}(b-\mu_b)
$$
$$
\Sigma = \Sigma_{aa} - \Sigma_{ab}\Sigma_{bb}^{-1}\Sigma_{ba}
$$
- Note the complicated matrix operations for the last two expressions, these inverse operations are expensive to calculate on a computer
- Also note the $\Sigma$ values, as $\Sigma_{*b}$ and $\Sigma_{b*}$ approach infinity (i.e. they have very low confidence in these recorded landmarks by not being seen very much), the influence that $b$ has over $a$ approaches zero

## Linear Model
- The Kalman filter assumes a linear transition and observation model
- Zero mean Gaussian noise
$$
x_t = A_tx_{t-1} +B_tu_t + \epsilon_t
$$
$$
z_t = C_tx_t + \delta_t
$$
## Component of a Kalman Filter
### $A_t$
Matrix $(n \times n)$ that describes how the state evolves from $t-1$ to $t$ without controls or noise

### $B_t$
Matrix $(n \times l)$ that describes how the control $u_t$ changes the state from $t-1$ to $t$

### $C_t$
Matrix $(k \times n)$ that describes how to map the state $x_t$ to an observation $z_t$

### $\epsilon_t$ & $\delta_t$
Random variables representing the process and measurement noise that are assumed to be independent and normally distributed with covariance $R_t$ and $Q_t$ respectively

## Linear Motion Model
- Motion under Gaussian noise leads to
$$
p(x_t|u_t, x_{t-1}) = \det(2\pi R_t)^{-\frac{1}{2}}\exp \bigg(-\frac{1}{2}(x_t - A_tx_{t-1} - B_tu_t)^T(x_t - A_tx_{t-1} - B_tu_t)\bigg)
$$
- $R_t$ describes the noise of the motion 

## Linear Observation Model
- Measuring under Gaussian noise leads to
$$
p(z_t|x_t) = \det(2\pi Q_t)^{-\frac{1}{2}}\exp \bigg(-\frac{1}{2}(z_t-C_tx_t)^TQt^{-1}(z_t-C_tx_t)\bigg)
$$
- $Q_t$ describes the measurement noise

## Everything Stays Gaussian
- Given an initial Gaussian belief, the belief is always Gaussian
$$
\overline{bel}(x_t) = \int \underline{p(x_t | u_t,x_{t-1})}\underline{bel(x_{t-1})}dx_{t-1}
$$
$$
bel(x_t) = \eta p(z_t|x_t)\overline{bel}(x_t)
$$
- Proof is non-trivial

## Kalman Filter Algorithm
![](kalman-filter-algorithm.png)

## 1D Kalman Filter Example (1)
![](1d-kalman-filter-example.png)

## 1D Kalman Filter Example (2)
![](1d-kalman-filter-example-2.png)

## Kalman Filter Assumptions
- Gaussian distributions and noise
- Linear motion and observation model
![](kalman-filter-assumptions.png)

## Non-Linear Dynamic Systems
- Most realistic problems (in robotics) involve nonlinear functions
![](non-linear-dynamic-systems.png)

## Linearity Assumption Revisited
- We can use the RHS linear function to express the parameters $a$ for gradient and $b$ for bias for a Gaussian distribution
![](linear-assumption-revisited.png)

## Non-Linear Function
- In a non-linear function, we do not simply have an $a$ and $b$ parameter that we can use to express a Gaussian distribution
- This leads to non-Gaussian distributions, as can be seen on the left
- However a single point on the plot does have an $a$ and $b$ if we find the derivative
![](non-linear-function.png)

## Non-Gaussian Distributions
- The non-linear functions lead to non-Gaussian distributions
- Kalman filter is not applicable anymore!

**What can be done to resolve this?**
**Local linearisation!**

## EKF Linearisation: First Order Taylor Expansion
![](ekf-linearisation-first-order-taylor-expansion.png)

## Reminder: Jacobian Matrix
- It is a **non-square matrix** $m \times n$ in general
- Given a vector-valued function
$$
g(x) = 
\begin{pmatrix}
g_1(x) \\
g_2(x) \\
\vdots \\
g_m(x)
\end{pmatrix}
$$
- The **Jacobian matrix** is defined as
$$
G_x = 
\begin{pmatrix}
\frac{\partial g_1}{\partial x_1} & \frac{\partial g_1}{\partial x_2} &  ... & \frac{\partial g_1}{\partial x_n} \\
\frac{\partial g_2}{\partial x_1} & \frac{\partial g_2}{\partial x_2} & ... & \frac{\partial g_2}{\partial x_n} \\
\vdots & \vdots & \dots & \vdots \\
\frac{\partial g_m}{\partial x_1} & \frac{\partial g_m}{\partial x_2} & \dots & \frac{\partial g_m}{\partial x_n}
\end{pmatrix}
$$

## Reminder: Jacobian Matrix
- It is the orientation of the tangent plane to the vector-valued function at a given point
![](reminder-jacobian-matrix.png)
- Generalises the gradient of a scalar valued function

## Linearity Assumption Revisited
![](linearity-assumption-revisited.png)

### Non-Linear Function
![](non-linear-function.png)

### EKF Linearisation (1)
- Below shows the difference between the gaussian of the function and the EKF Gaussian through linearisation via Taylor Approximation
![](ekf-linearisation-1.png)

### EKF Linearisation (2)
- However if the Gaussian distribution's covariance increases (i.e. the points recorded are far away from each other), then the difference between the EKF Gaussian distribution and the actual Gaussian distribution increases.
![](ekf-linearisation-2.png)

### EKF Linearisation (3)
- If the covariance is very low (i.e. we have tightly bound recorded points), then the difference between the EKF Gaussian distribution and the actual Gaussian distribution is also very low.
![](ekf-linearisation-3.png)

## Linearised Motion Model
- The linearised model leads to
$$
p(x_t|u_t,x_{t-1}) \approx \det (2 \pi R_t)^{-\frac{1}{2}} \exp \bigg(-\frac{1}{2}(x_t - g(u_t, \mu_{t-1})- G_t(x_{t-1} - \mu_{t-1}))^TR_t^{-1}(x_t - \underbrace{g(u_t, \mu_{t-1})-G_t(x_{t-1}-\mu_{t-1})}_{\text{linearised model}})\bigg)
$$
- $R_t$ describes the noise of the motion

## Linearised Observation Model
$$
p(z_t|x_t) =\det (2 \pi Q_t)^{-\frac{1}{2}} \exp \bigg(-\frac{1}{2}(z_t - h(\overline\mu_t)- H_t(x_t - \overline{\mu}_t))^TQ_t^{-1}(z_t - \underbrace{h(\overline{\mu}_t)-H_t(x_t-\overline{\mu}_t)}_{\text{linearised model}})\bigg)
$$
- $Q_t$ describes the measurement noise

## Extended Kalman Filter Algorithm
![](ekf-algorithm.png)