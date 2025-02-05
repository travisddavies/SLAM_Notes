# Grid-Based FastSLAM
## Motivation
- So far, we addressed landmark-based SLAM (KF-based SLAM, FastSLAM)
- We learned how to build grid maps assuming "known poses"

**Today: SLAM for building grid maps**

## Mapping With Raw Odometry
![](Images/mapping_raw_odometry.png)

## Observation
- **Assuming known poses fails!**

#### Questions
- Can we solve the SLAM problem if no pre-defined landmarks are available?
- Can we use the ideas of FastSLAM to build grid maps?

## Rao-Blackwellisation for SLAM
- Factorisation of the SLAM posterior
![](Images/rao_blackwellisation_for_slam.png)

## Grid-Based SLAM
- As with landmarks, the map depends on the poses of the robot during data acquisition
- If the poses are known, grid-based mapping is easy ("mapping with known poses")

## A Graphical Model for Grid-Based SLAM
![](Images/graphical_model_grid_slam.png)

## Grid-Based Mapping with Rao-Blackwellisation Particle Filters
- Each particle represents a possible trajectory of the robot
- Each particle maintains its own map
- Each particle updates it upon "mapping with known poses"

## Particle Filter Example
![](Images/particle_filter_example.png)

## Performance of Grid-Based FastSLAM 1.0
![](Images/performance_fastslam.png)

## Problem
- Too many samples are needed to sufficiently model the motion noise
- Increasing the number of samples is difficult as each map is quite large

- **Idea**: Improve the pose estimate **before** applying the particle filter

## Pose Correction Using Scan-Matching
- Maximise the likelihood of the **current** pose and map relative to the **previous** pose and map

![](Images/pose_correction_scan_matching.png)

## Mapping Using Scan Matching
- We can see below that the map's errors has improved greatly with this new method
![](Images/mapping_scan_matching.png)

## Grid-Based FastSLAM with Improved Odometry
- Scan-matching provides a **locally consistent** pose correction
- Pre-correct short odometry sequences using scan-matching and use them as input to FastSLAM
- Fewer particles are needed, since the error in the input is smaller

## Graphical Model for Mapping with Improved Odometry
- The concept is that we will scan-match chunks of poses, i.e. 100
- This will be done in succession, i.e. 100 poses, then another 100 poses, then another 100 poses
- Then use a particle filter to make the jump in the map from chunk $i$ to chunk $i+1$ using the standard FastSLAM approach
- As shown in the diagram below, you take the poses estimated from scan matching $k$ observations and measurements to jump to $x_k$
- We then do 1 step by a real odometry measurement and we add it on top of the observations estimated from the scan-matching
- We then take the observation of the last pose and apply to particle filter
- This can be considered like taking many observations to jump from one local map to the next local map
- Mathematically this works since you are applying the particle filter at pose $x_k$ with respect to $x_0$, then $x_{2k}$ with respect to $x_k$ and so on
![](Images/improved_odometry_graphical_model.png)

## Grid-Based FastSLAM with Scan-Matching
- As shown below, the performance is very good, particularly for loop closures
![](Images/grid_slam_scan_matching.png)
![](Images/grid_slam_scan_matching2.png)
![](Images/grid_slam_scan_matching3.png)

## Summary so far...
- Approach to SLAM that combines scan matching and FastSLAM
- Scan matching to generate virtual 'high quality' motion commands
- Can be seen as an ad-hoc solution to an improved proposal distribution

## What's Next?
- Compute an improved proposal that considers the most recent observation
$$
x_t^{[k]} \sim p(x_t | x_{1:t-1}^{[k]}, u_{1:t}, z_{1:t})
$$
**Goals:**
- More precise sampling
- More accurate maps
- Less particles needed

## The Optimal Proposal Distribution
- Now our proposal distribution can be broken down into smaller pieces using Bayes' Rule, with the first being the observation model and the second being the odometry model (blue v red lines)
- As shown below, we can actually see that combining poses via scan-matching actually has very good, peaked results. However for the odometry model, we can see that the results are not as good.
- We can therefore say that the first term actually dominates this product
- We thus want to exploit this feature in the distribution
![](Images/optimal_proposal_distribution.png)

## Proposal Distribution
- We can first just say that the top product can be called $\tau(x_t)$
![](Images/proposal_distribution0.png)
- What we then do is focus on the bottom expression, which doesn't contain the pose of the current timestep
- What we can do then is differentiate over all possible poses multiplied by the likelihood to be in the current pose (i.e. $\tau$)
![](Images/proposal_distribution1.png)
- We can therefore say then that the formula can be simplied as follows
![](Images/proposal_distribution2.png)
- What it means by locally limiting the area over which to integrate means that we will have very peaked distributions when we scan-match (as shown above)
- What it means to globally limit the area over which we integrate means that we will limit our poses to a reasonable distance (i.e. 5m), and we can be sure that our robot in the SLAM results won't look like it good teleported away
![](Images/proposal_distribution3.png)
- A demonstration is shown below, the first term will have several peaks but many peaks are not important, while the global peak will limit the distribution to only the local peaks within its own peak
![](Images/proposal_distribution4.png)
- So this is just the finalisation of what we were discussing above, with an introduction of how we sample from it
![](Images/proposal_distribution5.png)

## Gaussian Proposal Distribution
![](Images/gaussian_proposal_distribution.png)

## Estimating the Parameters of the Gaussian for Each Particle
$$
\begin{align}
\mu^{[i]} &= \frac{1}{\eta}\sum^K_{j=1}x_j \tau(x_j) \\
\Sigma^{[i]} &= \frac{1}{\eta}\sum^K_{j=1}(x_j - \mu^{[i]})(x_j - \mu^{[i]})^T \tau(x_j)
\end{align}
$$
$x_j$ are the points sampled around the result of the scan matcher

## Gaussian Proposal Distribution
![](Images/gaussian_proposal_distribution1.png)

## The Importance Weight
![](Images/importance_weight10.png)
![](Images/importance_weight11.png)
![](Images/importance_weight12.png)
![](Images/importance_weight13.png)
![](Images/importance_weight14.png)
![](Images/importance_weight15.png)
![](Images/importance_weight16.png)

## Improved Proposal
- The proposal adapts to the structure of the environment
- As shown below, if we have an open field we'll generally have poses spread out over the area. If we have a corridor, the model recognises the walls and spreads out poses along the axis. If the is a wall in front too, then the poses will be very concentrated.
![](Images/improved_proposal.png)

## Resampling
- Resampling at each step limits the "memory" of our filter
- Suppose we lose each time 25% of the particles, this may lead to:
![](Images/resampling1.png)
- Goal: Reduce the resampling actions

## Selective Resampling
- Resampling is necessary to achieve convergence
- Resampling is dangerous, since important samples might get lost ("particle depletion")
- Resampling makes only sense if particle weights differ significantly
- **Key question: When to resample?**

## Number of Effective Particles
- Empirical measure of how well the target distribution is approximately by samples drawn from the proposal
$$
n_{eff} = \frac{1}{\sum_i(w_t^{[i]})^2}
$$
- $n_{eff}$ describes "the inverse variance of the **normalized** particle weights"
- For equal weights, the sample approximation is close to the target

## Resampling with $n_{eff}$
- If our approximation is close to the target, no resampling is needed
- We only resample when $n_{eff}$ drops below a  given threshold ($N/2$)
$$
\frac{1}{\sum_i(w_t^{[i]})^2} \overset{?}{<} N / 2
$$
- Note: weights need to be normalized 

## Typical Evolution of $n_{eff}$
![](Images/typical_evolution_neff.png)

## Intel Lab
- **15 particles**
- Four times faster than real-time P4, 2.8GHz
- 5cm resolution during scan matching
- 1cm resolution in final map
![](Images/intel_lab.png)
![](Images/intel_lab1.png)

## Outdoor Campus Map
- **30 particles**
- $250 \times 250m^2$
- 1.75km (odometry)
- 30cm resolution in final map
![](Images/outdoor_campus_map.png)

## MIT Killian Court
![](Images/mit_killian_court.png)
- The **"infinite-corridor-dataset"** at MIT
![](Images/mit_killian_court1.png)

## Real World Application
- This guy uses a similar technique
![](Images/real_world_application.png)

## Problems of Gaussian Proposals
- Gaussians are uni-model distributions
- In case of loop-closures, the likelihood function might be multi-modal
![](Images/problem_gaussian_proposals.png)

## Gaussian or Non-Gaussian?
- Statistical test to check whether or not sample a generated from a Gaussian
- Anderson-Darling test (based on the cumulative density function)
- Difference between the Gaussian and the optimal proposal via KLD

## Is a Gaussian an Accurate Choice for the Proposal?
![](Images/gaussian_accurate.png)

## Problems of Gaussian Proposals
- Multi-modal likelihood function can cause filter divergence
![](Images/problem_gaussian_proposals1.png)

## Efficient Multi-Modal Sampling
- Approximate the likelihood in a better way!
![](Images/efficient_multimodal_sampling.png)
- Sample from odometry first and the use this as the start point for scan matching

## The Two-Step Sampling Works!
![](Images/two_step_sampling.png)

## Proposal Error Evaluation
![](Images/proposal_error_evalution.png)

## Effect of Two-Step Sampling
- Allows for better modeling multi-modal likelihood functions (high KLD values do not occur)
- For uni-modal cases, identical results
- Minimal computational overhead

## Gaussian Proposal: Yes or No?
- Gaussian allow for efficient sampling 
- Problematic in multi-modal cases
- Laser-Based SLAM: 3-6% multi-modal distribution (for the datasets here)
- Gaussian proposals can lead to divergence
- Two-step sampling process overcomes this problem effectively and efficiently

## Conclusion 
- The ideas of FastSLAM can also be applied in the context of grid maps
- Improved proposals are essential 
- Similar to scan-matching on a per-particle base
- Selective resamples reduce the risk of particle depletion
- Substantial reduction of the required number of particles