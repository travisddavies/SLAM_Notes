# Sparse Extended Information Filter for SLAM
## Reminder: Parameterisations for the Gaussian Distribution
![](Images/reminder-eif.png)

## Motivation
![](Images/seif-motivation.png)

## Motivation
![](Images/seif-motivation-2.png)

## Most Features Have Only a Small Number of Strong Links
![](Images/features-small-links.png)

## Information Matrix
- Information matrix can be interpreted as a graph of constraints/links between nodes (variables)
- Can be interpreted as a MRF
- Missing links indicate conditional independence of the random variables
- $\Omega_{i,j}$  tells us the strength of a link
- Larger values for nearby features
- Most off-diagonal elements in the information are close to 0 (but $\neq$ 0)

## Create Sparsity
- "Set" most links to zero/avoid fill-in
- Exploit sparseness of $\Omega$ in the computations
- **Sparse** = finite number of non-zero off-diagonals, independent of the matrix size

## Effect of Measurement Update on the Information Matrix
![](Images/effect-information-matrix.png)
![](Images/effect-information-matrix.png)
![](Images/effect-information-matrix3.png)
- Adds information between the robot's pose and the observed feature
![](Images/effect-information-matrix4.png)

## Effect of Motion Update on the Information Matrix
![](Images/effect-information-matrix5.png)
![](Images/effect-information-matrix6.png)
![](Images/effect-information-matrix7.png)
- Weakens the links between the robot's pose and the landmarks
- Add links between landmarks

## Sparsification
![](Images/sparsification.png)
![](Images/sparsification1.png)
![](Images/sparsification2.png)
![](Images/sparsification3.png)
- Sparsification means "ignoring" links (assuming conditional independence)
- Here: links between the robot's pose and some of the features
![](Images/sparsification4.png)

## Active and Passive Landmarks
Key element of SEIF SLAM to obtain an efficient algorithm

**Active Landmarks**
- A subset of all landmarks
- Includes the currently observed ones
**Passive Landmarks**
- All others

## Active vs. Passive Landmarks
![](Images/active_vs_passive_landmarks.png)

## Sparsification in Every Step
- SEIF SLAM conducts a **sparsification** step **in each iteration**

**Effect**
- The robot's pose is linked to the active landmarks only
- Landmarks have only links to nearby landmarks (landmarks that have been active at the same time)

## Four Steps of SEIF SLAM
![](Images/four_steps_seif_slam.png)
![](Images/four_steps_seif_slam2.png)
![](Images/four_steps_seif_slam3.png)
![](Images/four_steps_seif_slam4.png)

## Matrix Inversion Lemma
- Before we start, let us re-visit the matrix inversion lemma
- For any invertible quadratic matrices $R$ and $Q$ and any matrix $P$, the following holds:
$$
(R+PQP^T)^{-1} = R^{-1}-R^{-1}P(Q^{-1}+P^T R^{-1} P)^{-1}P^T R^{-1}
$$

## SEIF SLAM - Prediction Step
- Goal: Compute $\overline{\xi}_t, \overline{\Omega}_t, \overline{\mu}_t$ from motion and the previous estimate $\xi_{t-1}, \Omega_{t-1}, \mu_{t-1}$ 
- Efficiency by exploiting sparseness of the information matrix

## Let us start from EKF SLAM...
![](Images/let_us_start.png)
![](Images/let_us_start2.png)
![](Images/let_us_start3.png)

## SEIF - Prediction Step (1/3)
![](Images/seif-prediction.png)

### Compute the Information Matrix
- Computing the information matrix
$$
\begin{align}
	\overline{\Omega}_t &= \overline{\Sigma}_t^{-1} \\
	&= [G_t \Omega_{t-1}^{-1}G_t^T + R_t]^{-1} \\
	&= [\Phi^{-1}_t+R_t]^{-1}
\end{align}
$$
- with the term $\Phi_t$ defined as
$$
\begin{align}
\overline{\Omega}_t &= [\Phi_t^{-1}+R_t]^{-1} \\
&= [\Phi^{-1}_t+F^T_x R_t^x F_x]^{-1}
\end{align}
$$
- Apply the matrix inversion lemma
![](Images/apply_matrix.png)
![](Images/apply_matrix1.png)
![](Images/apply_matrix2.png)
- This can be written as
$$
\begin{align}
\overline{\Omega}_t &= [\Phi_t^{-1}+R_t]^{-1} \\
&= [\Phi^{-1}_t+F^T_x R_t^x F_x]^{-1} \\
&= \Phi_t-\underbrace{\Phi_t F_x^T(R_t^{x-1}+F_x\Phi_tF_x^T)^{-1}F_x\Phi_t}_{\kappa_t} \\
&= \Phi_t - \kappa_t
\end{align}
$$
- Question: Can we compute $\Phi_t$ efficiently ($\Phi_t = [G_t^T]^{-1}\Omega_{t-1}G_t^{-1}$)?

### Computing $\Phi_t = [G_t^T]^{-1} \Omega_{t-1} G_t^{-1}$ 
- Goal: constant time if $\Omega_{t-1}$ is sparse
![](Images/computing.png)
![](Images/computing_2.png)
![](Images/computing_3.png)
- We have
$$
\begin{align}
G_t^{-1} = I + \Psi_t \quad\quad\quad [G_t^T]^{-1} = I + \Psi_t^T \\
\end{align}
$$
- with
$$
\Psi_t = F_x^T\underbrace{[(I + \Delta)^{-1}-I]}_{3\times3 \text{ matrix}}F_x
$$
- $\Phi_t$ is zero except of a 3x3 block
- $G_t^{-1}$ is an identity except of a 3x3 block

Given that:
- $G_t^{-1}$ and $[G_t^T]^{-1}$ are identity matrices except of a 3x3 block
- The information matrix is sparse
- This implies that
$$
\Phi_t = [G_t^T]^{-1}\Omega_{t-1}G_t^{-1}
$$
- can be computed in constant time

- Given that $\Omega_{t-1}$ is sparse, the constant time update can be seen by
$$
\begin{align}
	\Phi_t &= [G_t^T]^{-1}\Omega_{t-1}G_t^{-1} \\
	&= (I + \Psi_t^T)\Omega_{t-1}(I+\Psi_t) \\
	&= \Omega_{t-1}+\underbrace{\Psi_t^T \Omega_{t-1}+\Omega_{t-1} \Psi_t^T + \Psi_t^T \Omega_{t-1} \Psi_t}_{\lambda_t} \\
	&= \Omega_{t-1} + \underline{\lambda_t} \\
	& \text{all elements zero except a constant number of entries}
\end{align}
$$

### Prediction Step in Brief
- Compute $\Psi_t$
- Compute $\lambda_t$ using $\Psi_t$
- Compute $\Phi_t$ using $\lambda_t$
- Compute $\kappa_t$ using $\Phi_t$
- Compute $\overline{\Omega}_t$ using $\Phi_t$ and $\kappa_t$

## SEIF - Prediction Step (2/3)
![](Images/seif-prediction2.png)
Information matrix is computed, now do the same for the information vector and the mean

### Compute the Mean
- The mean is computed as in the EKF
$$
\overline{\mu}_t = \mu_{t-1} + F_x^T \delta
$$
- Reminder (from SEIF motion update)
![](Images/seif-compute-the-mean.png)

### Compute the Information Vector
- We obtain the information vector by 
$$
\overline{\xi}_t
$$