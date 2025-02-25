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

## Key Idea of the Hierarchy
- Input is the dense graph
![](Images/key_idea_hierarchy.png)
- Group the nodes of the graph based on their local connectivity
![](Images/key_idea_hierarchy1.png)
- **For each group select one node as a "representative"**
![](Images/key_idea_hierarchy2.png)
- The representatives are the nodes in a new sparsified graph (upper level)
![](Images/key_idea_hierarchy3.png)
- The representatives are the nodes in a new sparsified graph (upper level)
- Edges of the sparse graph are determined by the connectivity of the groups of nodes
- The parameters of the sparse edges are estimated via local optimization
- **Process is repeated recursively**
- Only the upper level of the hierarchy is optimised completely
![](Images/key_idea_hierarchy4.png)
- The changes are propagated to the bottom levels only close to the current robot position
- Only part of the graph is relevant for finding constraints
![](Images/key_idea_hierarchy5.png)

## Construction of the Hierarchy
- When and how to generate a new group?
	- A (simple) distance-based decision
	- The first nodes of a new group is the representative
- When to propagate information downwards?
	- Only when there are inconsistencies?
- How to construct an edge in the sparsified graph?
- How to propagate information downwards?

## Determining Edge Parameters
- Given two connected groups
- How to compute a virtual observation $z$ and the information and the information matrix $\Omega$ for the new edge?
![](Images/determining_edge_parameters.png)
- Optimise the sub-groups independently from the rest
![](Images/determining_edge_parameters1.png)
- The observation is the relative transformation between the two representatives
![](Images/determining_edge_parameters2.png)
- The information matrix is computed from the diagonal block of the matrix $H$ 
![](Images/determining_edge_parameters3.png)

## Propagating Information Downwards
- All representative are nodes from the lower (bottom) level
![](Images/propagating_information_downwards.png)
- Information is propagated downwards by transforming the group at the lower level using a rigid body transformation
![](Images/propagating_information_downwards1d.png)
- Only if the lower level becomes inconsistent, optimise at the lower level
![](Images/propagating_information_downwards2.png)

## For the Best Possible Map...
- Run the optimisation on the lowest level (at the end)
- For offline processing with all constraints, the hierarchy helps convergence faster in case of large errors
- In the case, one pass up the tree (to construct the edges) followed by one pass down the tree is sufficient

## Stanford Garage
![](Images/stanford_garage.png)
- Parking garage at Stanford University
- Nested loops, trajectory of ~7000m

## Stanford Garage Result
![](stanford_garage_result.png)
- Parking garage at Stanford University
- Nested loops, trajectory of ~7000m

## Consistency
- How well does the top level in the hierarchy represent the original input?
- Probability mass of the marginal distribution in the highest level vs. the one of the true estimate (original problem, lowest level)
![](Images/consistency.png)
![](Images/consistency1.png)