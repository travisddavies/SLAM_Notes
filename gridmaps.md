# Grid Maps
## Features vs. Volumetric Maps
![](Images/features-vs-volumetric-maps.png)

## Features
- So far, we only used feature maps
- Natural choice for Kalman filter-based SLAM systems
- Compact representation
- Multiple feature observations improve the landmark position estimate (EKF)

## Grid Maps
- Discretise the world into cells
- Grid structure is rigid
- Each cell is assumed to be occupied or free space
- Non-parametric model
- Large maps require substantial memory resources
- Do not rely on a feature detector

## Example
![](Images/grid-maps-feature.png)

## Assumption 1
- The area that corresponds to a cell is either completely free or occupied
![](Images/assumption-1.png)

## Representation
- Each cell is a **binary random variable** that models the occupancy
![](Images/representation-gridmaps.png)

## Occupancy Probability
- Each cell is a **binary random variable** that models the occupancy
- Cell is occupied: $p(m_i)=1$ 
- Cell is not occupied: $p(m_i)=0$
- No knowledge: $p(m_i)=0.5$

## Occupancy Probability Example
- Each cell is a **binary random variable** that models the occupancy
![](Images/assumption-2.png)

## Assumption 2
- The world is **static** (most mapping systems make this assumption)
![](Images/assumption21.png)

## Assumption 3
- The cells (the random variables) are **independent** of each other
![](Images/assumption2.png)

## Joint Distribution
![](Images/joint-distribution.png)

## Representation
- The probability distribution of the map is given by the product over the cells
![](Images/representation2.png)

## Example A
![](Images/example-a.png)

## Example B
![](Images/example-b.png)

## Estimating a Map From Data
- Given sensor data $z_{1:t}$ and the poses $x_{1:t}$ of the sensor, estimate the map
![](Images/estimating-from-sensor-data.png)

## Static State Binary Bayes Filter
![](Images/state-static-bayes-filter.png)
![](Images/state-static-bayes-filter-1.png)
![](Images/state-static-bayes-filter-2.png)
![](Images/state-static-bayes-filter-3.png)
![](Images/state-static-bayes-filter-4.png)
![](Images/state-static-bayes-filter-5.png)
![](Images/state-static-bayes-filter-6.png)
![](Images/state-static-bayes-filter-6.png)
![](Images/state-static-bayes-filter-8.png)
