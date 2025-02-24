# Hierarchical Pose-Graphs for Online Mapping

## Graph-Based SLAM
- Constrainsts connect the poses of the robot while it is moving
- Constraints are inherently uncertain
![](Images/graph_based_slam3.png)
- Observing previously seen areas generates constraints between non-successive poses
![](Images/graph_based_slam2.png)

- Use a **graph** to represent the problem
- Every **node** in the graph corresponds to a pose of the robot during mapping
- Every **Edge** between two nodes corresponds to a spatial constraint between them
- **Graph-Based SLAM**: Build the graph and find a configuration that minimise the error introduced by the constraints

## Front-End and Back-End
- Front-end extracts constraints from the sensor data (data association!)
- Back-end optimises the pose-graph to reduce the error introduced by the constraints
![](Images/frontend_backend.png)
$\rightarrow$ Intermediate solutions are needed to make good data associations

## Hierarchical Pose-Graph
![](Images/hierarchical_pose_graph.png)
"There is no need to optimise the whole graph when a new observation is obtained"

## Motivation
- SLAM front-end seeks for loop-closures
- Requires to compare observations to all previously obtained ones
- In practice, limit search to areas in which the robot is likely to be
- This requires to know **in which part of the graph to search for data associations**
![](Images/motivation1.png)

## Hierarchical Approach
- **Insight**: to find loop closing points, one does not need the perfect global map
- **Idea**: correct only the core structure of the scene, not the overall graph
- The hierarchical pose-graph is a sparse approximation of the original problem
- It exploits the facts that in SLAM
	- Robot moved through the scene and it did not "teleport" to locations
	- Sensors have a limited range
