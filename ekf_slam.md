# EKF SLAM
## Simultaneous Localisation and Mapping (SLAM)
- Building a map and locating the robot in the map at the same time
- Chicken-or-egg problem

![](slam.png)

## Definition of the SLAM Problem
**Given**
- The robot's controls
$$
u_{1:T} = \{u_1, u_2, u_3, \dots, u_T\}
$$
- Observations
$$
z_{1:T} = \{z_1, z_2, z_3, \dots, z_T\}
$$
**Wanted**
- Map of the environment
$$
m
$$
- Path of the robot
$$
x_{0:T} = \{x_0, x_1, x_2, \dots, x_T\}
$$

## Three Main Paradigms
![](three_main_paradigms.png)

## Bayes Filter
- Recursive filter with prediction and correction step
- **Prediction**
$$
\overline{bel}(x_t) =\int p(x_t|u_t, x_{t-1})bel(x_{t-1})dx_{t-1}
$$
- **Correction**
$$
bel(x_t) = \eta p(z_t|x_t) \overline{bel}(x_t)
$$

## EKF for Online SLAM
- We consider here the Kalman filter as a solution to the online SLAM problem
$$
p(x_t,m|z_{1:t}, u_{1:t})
$$
![](ekf_for_online_slam.png)

## Extended Kalman Filter
![](ekf.png)

## EKF SLAM
- Application of the EKF to SLAM
- Estimate robot's pose and locations of landmarks in the environment
- Assumption: known correspondences
- State space (for the 2D plane) is
$$
x_t = (\underbrace{x, y, \theta}_{\text{robot's pose}}, \underbrace{m_{1,x}, m_{1,y}}_{\text{landmark 1}}, \dots, \underbrace{m_{n,x}, m_{n,y}}_{\underbrace{\text{landmark n}}})^T
$$

## EKF SLAM: State Representation
- Map with $n$ landmarks: $(3+2n)$-dimensional Gaussian
- Belief is represented by
- Yellow represents the covariance matrix of the robot's pose
- Blue represents the covariance matrix of the mapping
- Greeen represents the correlation between the robot pose variance matrix and the mapping covariance matrix
![](ekf_slam_state_representation.png)
- More compactly
![](more_compactly.png)
- Even more compactly (not: $x_R \rightarrow x$)
![](even_more_compactly.png)

## EKF SLAM: Filter Cycle
1. State prediction
2. Measurement prediction
3. Measurement
4. Data association
5. Update

### EKF SLAM: State Prediction
- Below the white circle represents the robot, and the dashed line represents the trajectory of the robot in its environment.
- The red ellipse around the robot represents the level of uncertainty about the robot's pose, which increases as he moves in his environment
- Since the robot's movement in its world (if we don't assume that it doesn't bump and move anything) **will not influence the location of landmarks in the map**, we can say that the robot's movement will **only affect the green-highlighted areas in the below matrices**.
![](state_prediction2.png)

### EKF SLAM: Measurement Prediction
- This stage is basically the equivalent of saying "given my current pose, what sort of landmark should I measure?"
- As can be seen by the highlighted star in the below diagram, the measurement prediction given the robot's pose is the star, with the red ellipse representing its uncertainty level
![](measurement_prediction.png)

### EKF SLAM: Obtained Measurement
- This is the actual measurement that the robot received
![](obtained_measurement.png)

### EKF SLAM: Data Association and Difference Between $h(x)$ and $z$
- We need to find which landmark this measurement belongs to, and the difference between the predicted and measured landmark
![](data_association_and_difference.png)

### EKF SLAM: Update Step
- Once we have completed all the above steps, we can then update the state matrix for correction
![](update_slam.png)

## EKF SLAM: Concrete Example
**Setup**
- Robot moves in the 2D plane
- Velocity-based motion model
- Robot observes point landmark
- Range-bearing sensor
- Known data association
- Known number of landmarks

## Initialisation
- Robot starts in its own reference frame (all landmarks unknown)
- $2N+3$ dimensions
![](initialisation.png)

## Extended Kalman Filter Algorithm
![](ekf_algorithm.png)

## Prediction Step (Motion)
- Goal: Update state space based on the robot's motion
- Robot motion in the plane
$$
\begin{pmatrix}
x' \\
y' \\
\theta'
\end{pmatrix}
=
\underbrace{\begin{pmatrix}
x \\
y \\
\theta
\end{pmatrix}
+
\begin{pmatrix}
-\frac{v_t}{w_t}\sin\theta+\frac{v_t}{w_t}\sin(\theta+w_t\Delta t) \\
\frac{v_t}{w_t}\cos \theta - \frac{v_t}{w_t}\cos(\theta + w_t \Delta t) \\
w_t \Delta t
\end{pmatrix}}_{g_{x,y,\theta}(u_t,(x,y,\theta)^T)}
$$
- How to map that to the $2N+3$ dim space?

## Update the State Space
- From the motion in the plane
$$
\begin{pmatrix}
x' \\
y' \\
\theta'
\end{pmatrix}
=
\begin{pmatrix}
x \\
y \\
\theta
\end{pmatrix}
+
\begin{pmatrix}
-\frac{v_t}{w_t}\sin\theta+\frac{v_t}{w_t}\sin(\theta+w_t\Delta t) \\
\frac{v_t}{w_t}\cos \theta - \frac{v_t}{w_t}\cos(\theta + w_t \Delta t) \\
w_t \Delta t
\end{pmatrix}
$$
- to the $2N+3$ dimensional space
- the first three columns are an identity matrix, and the last $2N$ columns are all zeros
![](update_the_state_space.png)

## Extended Kalman Filter Algorithm
- First line now complete
![](ekf_algorithm_3.png)

## Update Covariance
- The function $g$ only affects the robot's motion and not the landmarks
![](update_covariance.png)

## Jacobian of the Motion
$$
G_t^x = \frac{\partial}{\partial(x, y, \theta)^T}\Bigg[
\begin{pmatrix}
x \\
y \\
\theta
\end{pmatrix}
+
\begin{pmatrix}
-\frac{v_t}{w_t}\sin \theta + \frac{v_t}{w_t} \sin(\theta + w_t \Delta t) \\
\frac{v_t}{w_t} \cos \theta - \frac{v_t}{w_t} \cos (\theta + w_t \Delta t) \\
w_t \Delta t
\end{pmatrix}
\Bigg]
$$
$$
G_t^x = I + \frac{\partial}{\partial (x, y, \theta)^T} 
\begin{pmatrix}
-\frac{v_t}{w_t}\sin \theta + \frac{v_t}{w_t} \sin(\theta + w_t \Delta t) \\
\frac{v_t}{w_t} \cos \theta - \frac{v_t}{w_t} \cos (\theta + w_t \Delta t) \\
w_t \Delta t
\end{pmatrix}
$$
$$
G_t^x = I +
\begin{pmatrix}
0 & 0 & -\frac{v_t}{w_t} \cos \theta + \frac{v_t}{w_t}\cos (\theta + w \Delta t)\\
0 & 0 & -\frac{v_t}{w_t} \sin \theta + \frac{v_t}{w_t} \sin(\theta + w_t \Delta t) \\
0 & 0  & 0 \\
\end{pmatrix}
$$
$$
G_t^x =
\begin{pmatrix}
1 & 0 & -\frac{v_t}{w_t} \cos \theta + \frac{v_t}{w_t} \cos (\theta + w_t \Delta t)\\
0 & 1 & - \frac{v_t}{w_t} \sin \theta + \frac{v_t}{w_t}\sin(\theta + w_t \Delta t)\\
0 & 0 & 1\\
\end{pmatrix}
$$

## This Leads to the Update
![](this_leads_to_the_update.png)

## Extended Kalman Filter Algorithm
![](extended_kalman_filter_algorithm.png)

## EKF SLAM: Prediction Step
![](ekf_slam_prediction_step.png)

## Extended Kalman Filter Algorithm
![](extended_kalman_filter_algorithm.png)

## EKF SLAM: Correction Step
- Known data association
- $c_t^i$: $i$-th measurement at time $t$ observes the landmark with index $j$
- Initialise landmark if unobserved
- Compute the expected observation
- Compute the Jacobian of $h$
- Proceed with computing the Kalman gain

## Range-Bearing Observation
- Range-Bearing observation $z_t^i = (r_t^i, \phi^i_t)^T$
- Range-bearing observation is to do with the relative coordinates in terms of angle and distance from a certain zero coordinate and angle
- If landmark has not been observed
$$
\underset{
	\begin{array}
		\text{observed} \\
		\text{location of} \\
		\text{landmark } j
	\end{array}
}
	{\begin{pmatrix}
		\overline{\mu}_{j,x} \\
		\overline{\mu}_{j,y}
	\end{pmatrix}}
=
\underset{
	\begin{array}
	\text{estimated} \\
	\text{robot's} \\
	\text{location}
	\end{array}
}
{\begin{pmatrix}
	\overline{\mu}_{t,x} \\
	\overline{\mu}_{t,y}
\end{pmatrix}}
+
\underset{
	\begin{array}
		\text{relative} \\
		\text{measurement}
	\end{array}
}
	{\begin{pmatrix}
		r_t^i\cos(\phi_t^i + \overline{\mu}_{t,\theta}) \\
		r_t^i \sin(\phi^i_t + \overline{\mu}_{t,\theta})
	\end{pmatrix}}
$$

## Expected Observation
- Compute expected observation according to the current estimate
$$
\begin{equation}
\begin{split}
\delta &= 
\begin{pmatrix}
\delta_x \\
\delta_y 
\end{pmatrix}
=
\begin{pmatrix}
\overline{\mu}_{j,x} - \overline{\mu}_{t,x} \\
\overline{\mu}_{j,y} - \overline{\mu}_{t,y}
\end{pmatrix} \\
q &= \delta^T\delta \\
\hat{z}^i_t &= 
\begin{pmatrix}
\sqrt{q} \\
\text{atan2}(\delta_y, \delta_x) - \overline{\mu}_{t,\theta}
\end{pmatrix} \\
\hat{z}_t^i &= h(\overline{\mu}_t)
\end{split}
\end{equation}
$$

## Jacobian for the Observation
- Based on:
$$
\begin{equation}
\begin{split}
\delta &= 
\begin{pmatrix}
\delta_x \\
\delta_y 
\end{pmatrix}
=
\begin{pmatrix}
\overline{\mu}_{j,x} - \overline{\mu}_{t,x} \\
\overline{\mu}_{j,y} - \overline{\mu}_{t,y}
\end{pmatrix} \\
q &= \delta^T\delta \\
\hat{z}^i_t &= 
\begin{pmatrix}
\sqrt{q} \\
\text{atan2}(\delta_y, \delta_x) - \overline{\mu}_{t,\theta}
\end{pmatrix} \\
\hat{z}_t^i &= h(\overline{\mu}_t)
\end{split}
\end{equation}
$$
- Compute the Jacobian
![](low-dim-space.png)
$$
\text{low } H_t^i = \frac{\partial h (\bar{\mu}_t)}{\partial \bar{\mu}_t}
= 
\begin{pmatrix}
\frac{\partial \sqrt{q}}{\partial x} & \frac{\partial \sqrt{q}}{\partial y} & \cdots \\[12pt]
\frac{\partial \mathrm{atan2}(\dots)}{\partial x} & \frac{\partial \mathrm{atan2}(\dots)}{\partial y} & \cdots
\end{pmatrix}
	$$

## The First Component
- Based on:
$$
\begin{equation}
\begin{split}
\delta &= 
\begin{pmatrix}
\delta_x \\
\delta_y 
\end{pmatrix}
=
\begin{pmatrix}
\overline{\mu}_{j,x} - \overline{\mu}_{t,x} \\
\overline{\mu}_{j,y} - \overline{\mu}_{t,y}
\end{pmatrix} \\
q &= \delta^T\delta \\
\hat{z}^i_t &= 
\begin{pmatrix}
\sqrt{q} \\
\text{atan2}(\delta_y, \delta_x) - \overline{\mu}_{t,\theta}
\end{pmatrix} \\
\hat{z}_t^i &= h(\overline{\mu}_t)
\end{split}
\end{equation}
$$
- We obtain (by applying the chain rule)
$$
\begin{aligned}
\frac{\partial \sqrt{q}}{\partial x} &=
\frac{1}{2} \frac{1}{\sqrt{q}} 2 \delta_x(-1) \\
\text{} &= \frac{1}{q}(-\sqrt{q}\delta_x)
\end{aligned}
$$
- In-depth derivation:
	- Remember chain rule: $\frac{\partial f(g)}{\partial x} = f'(g) \cdot g'(x)$ 
$$
\begin{aligned}
	\frac{\partial \sqrt{q}}{\partial x} &= \frac{1}{2\sqrt{q}} \cdot \frac{\partial q}{\partial x} \\
	q &= \delta^T \delta\\
	q &= \delta_x^2 + \delta_y^2 \\
	\frac{\partial q}{\partial x} &= 2\delta_x \cdot \frac{\partial \delta_x}{\partial x} \\
	\delta_x & = \overline{\mu}_{j,x} - \overline{\mu}_{t,x} \\
	\frac{\partial \delta_x}{\partial x} &= -1 \\
	\frac{\partial \sqrt{q}}{\partial x} &= \frac{1}{2\sqrt{q}} \cdot 2 \delta_x \cdot -1 \\
	\frac{\partial \sqrt{q}}{\partial x} &= \frac{-1}{\sqrt{q}} \cdot  \delta_x \\
	\frac{1}{\sqrt{q}} \cdot \frac{\sqrt{q}}{\sqrt{q}} &= \frac{\sqrt{q}}{q} \\
	\frac{\partial \sqrt{q}}{\partial x} &= \frac{1}{q}(-\sqrt{q}\delta_x)
\end{aligned}
$$
- This will repeat for all of $\{x, y, \theta, m_{j,x}, m_{j,y}\}$
- Note that for $\{m_{j,x}, m_{j,y}\}$, the $\overline{\mu}_{j,x}$ and $\overline{\mu}_{j,y}$ will be set to 1 rather than 0 like in the above derivation
## Jacobian for the Observation
- Based on
$$
\begin{equation}
\begin{split}
\delta &= 
\begin{pmatrix}
\delta_x \\
\delta_y 
\end{pmatrix}
=
\begin{pmatrix}
\overline{\mu}_{j,x} - \overline{\mu}_{t,x} \\
\overline{\mu}_{j,y} - \overline{\mu}_{t,y}
\end{pmatrix} \\
q &= \delta^T\delta \\
\hat{z}^i_t &= 
\begin{pmatrix}
\sqrt{q} \\
\text{atan2}(\delta_y, \delta_x) - \overline{\mu}_{t,\theta}
\end{pmatrix} \\
\hat{z}_t^i &= h(\overline{\mu}_t)
\end{split}
\end{equation}
$$
- Compute the Jacobian
- This just goes through the process of what we showed above, note that the last two columns are the opposite value, this is because the opposite value in the $\delta_x$ and $\delta_y$ were derived (as shown in the above derivation)
$$
\begin{aligned}
\text{low } H_t^i &= \frac{\partial h (\bar{\mu}_t)}{\partial \bar{\mu}_t} \\
&= \frac{1}{q} 
\begin{pmatrix}
-\sqrt{q} \delta_x & -\sqrt{q} \delta_y & 0 & 
\sqrt{q}\delta_x & \sqrt{q}\delta_y \\
\delta_y & \delta_x & -q & -\delta_y & \delta_x
\end{pmatrix}
\end{aligned}
$$
- Use the computed Jacobian
$$
\begin{aligned}
\text{low } H_t^i &= \frac{\partial h (\bar{\mu}_t)}{\partial \bar{\mu}_t} \\
&= \frac{1}{q} 
\begin{pmatrix}
-\sqrt{q} \delta_x & -\sqrt{q} \delta_y & 0 & 
\sqrt{q}\delta_x & \sqrt{q}\delta_y \\
\delta_y & \delta_x & -q & -\delta_y & \delta_x
\end{pmatrix}
\end{aligned}
$$
- Map it to the high dimensional space
![](map-to-high-dimensional-space.png)

## Next Steps as Specified
![](next_steps_as_specified.png)

## Extended Kalman Filter Algorithn
![](extended_kalman_filter_algorithm.png)

## EKF SLAM - Correction (1/2)
![](ekf_slam_correction_1.png)

## EKF SLAM - Correction (2/2)
![](ekf_slam_correction_2.png)

## Implementation Notes
- Measurement update in a single step requires only one full belief update
- Always normalise the angular components
- You may not neet to create the $F$ matrices explicitly (e.g. in Octave)

## Loop Closing
- Loop closing means recognising an already mapped area
- Data association under
	- High ambiguity
	- Possible environment symmetries
- Uncertainties **collapse** after a loop closure (whether the closure was correct or not)

## Before the Loop Closure
![](before_the_loop_closure.png)

## After the Loop Closure
![](after_the_loop_closure.png)

## Loop Closure in SLAM
- Loop closing **reduces** the uncertainty in robot and landmark estimates
- This can be exploited when exploring an environment for the sake of better (e.g. more accurate) maps
- **Wrong loop closures lead to filter divergence**

## EKF SLAM Correlations
- In the limit, the landmark estimates become **fully correlated**
![](ekf_slam_correlations.png)
![](ekf_slam_correlations2.png)
![](ekf_slam_correlations3.png)
![](ekf_slam_correlations4.png)

## EKF SLAM Correlations
- The correlation between the robot's pose and the landmarks **cannot** be ignored
- Assuming independence generates too optimistic of the uncertainty

## EKF SLAM Uncertainties
- The **determinant** of any sub-matrix of the map covariance matrix **decrease monotonically**
- New landmarks are initialised with **maximum uncertainty**
![](ekf_slam_uncertainties.png)

## EKF SLAM in the Limit
- In the limit, the covariance associated with any single landmark location estimate is determined only by the initial covariance in the vehicle location estimate
![](ekf_slam_in_the_limit.png)

## Example: Victoria Park Dataset
![](victoria_park_dataset.png)

## Victoria Park: Data Acquisition
![](victoria_park_data_acquisition.png)

## Victoria Park: EKF Estimate
![](victoria_park_ekf_estimate.png)

## Victoria Park: Landmarks
![](victoria_park_landmarks.png)

## Example: Tennis Court Dataset
![](example_tennis_court_dataset.png)

## EKF SLAM on a Tennis Court
![](ekf_slam_on_a_tennis_court.png)

## EKF SLAM Complexity
- Cubic complexity depends only on the measurement dimensionality
- Cost per step: dominated by the number of landmarks: $O(n^2)$
- Memory consumption: $O(n^2)$
-  The EKF becomes computationally intractable for large maps!