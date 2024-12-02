## What is Robot Mapping?
- **Robot** - a device, that moves through the environment
- **Mapping** - modelling the environment

## Related Terms
![](Images/related-terms.png)

## What is SLAM?
- Computing the robot's poses and the map of the environment at the same time

- **Localization**: estimating the robot's location
- **Mapping**: building a map
- **SLAM**: building a map and localising the robot simultaneously

## Localisation Example
- Estimate the robot's poses given landmarks
![](Images/localisation-example.png)

## Mapping Example
- Estimate the landmarks given the robot's poses
![](Images/mapping-example.png)

## SLAM Example
- Estimate the robot's poses and the landmarks at the same time
![](Images/slam-example.png)

## The SLAM Problem
- SLAM is a **chicken-or-egg** problem:
	- A map is needed for localisation and
	- A pose estimate is needed for mapping
![](Images/slam-problem.png)

## SLAM is Relevant
- It is considered a fundamental problem for truly autonomous robots
- SLAM is the basis for most navigation systems
![](Images/slam-is-relevant.png)

## SLAM Applications
- SLAM is central to a range of indoor, outdoor, air and underwater applications for both manned and autonomous vehicles.

**Examples**:
- At home: vacuum cleaner, lawn mower
- Air: surveillance with unmanned air vehicle
- Underwater: reef monitoring
- Underground: exploration of mines
- Space: terrain mapping for localisation

## SLAM Applications
![](Images/slam-applications.png)

## Definition of the SLAM Problem
**Given**
- The robot's controls
	$u_{1:T} = \{u_1, u_2, u_3, ..., u_T\}$
- Observations
	$z_{1:T} = \{z_1, z_2, z_3, ..., z_T\}$
**Wanted**
- Map of the environment
	$m$
- Path of the robot
	$x_{0:T} = \{x_0, x_1, x_2, ..., x_T\}$

## Probabilistic Approaches
- Uncertainty in the robot's motions and observations
- Use the probability theory to explicitly represent the uncertainty
![](Images/probabilistic-approaches.png)

## In the Probabilistic World
Estimate the robot's path and the map
![](Images/in-the-probabilistic-world.png)

## Graphical Model
![](Images/graphical-model.png)

## Full SLAM vs. Online SLAM
- Full SLAM estimates the entire path
$$p(x_{0:T},m|z_{1:T},u_{1:T})$$
- Online SLAM seeks to recover only the most recent pose
$$p(x_t,m|z_{1:t}.u_{1:t})$$
## Graphical Model of Online SLAM
![](Images/graphical-model-of-online-slam.png)

### Online SLAM
- Online SLAM means marginalising out the previous poses
$$p(x_t,m|z_{1:t},u_{1:t}) = \int ... \int p(x_{0:t}, m|z_{1:t},u_{1:t})dx_{t-1}...dx_0$$
- Integrals are typically solved recursively, one at a time

![](Images/graphical-model-of-online-slam2.png)
$$p(x_{t+1}, m|z_{1:t+1},u_{1:t}) = \int ... \int p(x_{0:t+1},m|z_{1:t+1},u_{t:t+1})dx_t ... dx_0$$
## Why is SLAM a Hard Problem?
1. Robot path and map are both **unknown**
![](Images/unknown-map.png)
2. Map and pose estimates correlated

- The **mapping between observations and the map is unknown**
- Picking **wrong** data associations can have **catastrophic** consequences (divergence)
![](Images/divergence.png)

## Taxonomy of the SLAM Problem
### Volumetric vs. feature-based SLAM
![](Images/taxonomy-of-the-slam-problem.png)

### Topologic vs. Geometric Maps
![](Images/taxonomy-of-the-slam-problem2.png)

### Known vs. Unknown Correspondence
![](Images/known-correspondence.png)

### Static vs. Dynamic Environments
![](Images/static-dynamic-environments.png)

### Small vs. Large Uncertainty
![](Images/small-large-uncertainty.png)

### Active vs. Passive SLAM
![](Images/active-passive-slam.png)

### Anytime and Any-Space SLAM
![](Images/anytime-anyspace-slam.png)

### Single-Robot vs. Multi-Robot SLAM
![](Images/single-multi-robot-slam.png)

## Approaches to SLAM
- Large variety of different SLAM approaches have been proposed
- Most robotics conferences dedicate multiple tracks to SLAM
- The majority of techniques use probabilistic concepts
- History of SLAM dates back to the mid-eighties
- Related problems in geodesy and photogrammetry

SLAM History by Durrant-Whyte
- 1985/86: Smith et al. and Durrant-Whyte describe geometric uncertainty and relationships between features and relationships between features and relationships between features or landmarks
- 1986: Discussion at ICRA on how to solve the SLAM problem followed by the key paper by Smith, Self and Cheeseman
- 1990-95: Kalman-filter based approaches
- 1995: SLAM acronym coined at ISRR'95
- 1995-19999: Convergence proofs & first demonstrations of real-systems 
- 2000: Wide interest in SLAM started

## Three Main Paradigm
![](Images/three-main-paradigms.png)

## Motion and Observation Model
![](Images/motion-and-observation-model.png)

## Motion Model
- The motion model describes the relative motion of the robot
![](Images/motion-model.png)

## Motion Model Example
- Gaussian model
![](Images/gaussian-model.png)
- Non-Gaussian model
![](Images/non-gaussian-model.png)

## Standard Odometry Model
- Robot moves from $(\bar{x}, \bar{y}, \bar{\theta})$, to $(\bar{x}', \bar{y}', \bar{\theta}')$
- Odometry information $y=(\delta_{rot1}, \delta_{trans}, \delta_{rot2})$ 
$$
\delta_{trans} = \sqrt{(\bar{x}'-\bar{x})^2 - (\bar{y}'-\bar{y})^2}
$$
$$
\delta_{rot1} = \text{atan2}(\bar{y}'-\bar{y}, \bar{x}'-\bar{x}) - \bar{\theta}
$$
$$
\delta_{rot2} = \bar{\theta}' - \bar{\theta} - \delta_{rot1}
$$
![](Images/standard-odometry-model.png)

