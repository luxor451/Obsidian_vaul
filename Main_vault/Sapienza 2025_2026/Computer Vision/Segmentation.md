**Tags:** #ComputerVision #ImageSegmentation #Clustering #KMeans #GraphCut #MeanShift
![[12-Segmentation.pdf]]# Terminology
- **Segmentation, grouping, perceptual organization**: gathering features that belong together
- **Fitting**: associating a model with observed features
-  **Top-down segmentation**: pixels belong together because they come from the same object
- **Bottom-up segmentation**: pixels belong together because they look similar
# Segmentation
### The goals
![[2025-12-02-174918_hyprshot.png]]
- Separate image into coherent “objects”
	- Top-down or bottom-up process?
	- Supervised or unsupervised?
- Group together similar-looking pixels for efficiency of further processing
	- Related to image compression
	- Measure of success is often application-dependent
### Outline
- Inspiration from psychology
- Segmentation as clustering
	- K-means
	- Mean shift
- Segmentation as partitioning
	- Graph-based segmentation, normalized cuts
- Integrating top-down and bottom-up segmentation for recognition

# The Gestalt school
- Grouping is key to visual perception
- Elements in a collection can have properties that result from relationships
	- “The whole is greater than the sum of its parts
### Gestalt factors
![[2025-12-02-175711_hyprshot.png]]
-  These factors make intuitive sense, but are very difficult to translate into algorithms
- They may be hard to put into algorithms, but understanding them can come in useful for interface design

# Different clustering strategies
- Agglomerative clustering
	- Start with each point in a separate cluster
	- At each iteration, merge two of the “closest” clusters
- Divisive clustering
	- Start with all points grouped into a single cluster
	- At each iteration, split the “largest” cluster
- K-means clustering
	- Iterate: assign points to clusters, compute means
- K-medoids
	- Same as k-means, only cluster center cannot be computed by averaging
	- The “medoid” of each cluster is the most centrally located point in that cluster (i.e., point with lowest average distance to the other points)
### K-Means clustering
- based on [K-means](https://en.wikipedia.org/wiki/K-means_clustering)
-  K-means clustering based on intensity or color is essentially vector quantization of the image attributes
- Clusters don’t have to be spatially coherent
- Clustering based on $(r,g,b,x,y)$ values enforces more spatial coherence
##### pros and cons
- Pros
	- Simple and fast
	- Converges to a local minimum of the error function
- Cons
	- Need to pick K
	- Sensitive to initialization
	- Sensitive to outliers
	- Only finds “spherical” clusters

### Mean shift segmentation
[An advanced and versatile technique for clustering-based segmentation](https://en.wikipedia.org/wiki/Mean_shift)
-  The mean shift algorithm seeks a mode or local maximum of density of a given distribution
	- Choose a search window (width and location)
	- Compute the mean of the data in the search window
	- Center the search window at the new mean location
	- Repeat until convergence
-  **Cluster**: all data points in the attraction basin of a mode
- **Attraction basin**: the region for which all trajectories lead to the same mode
##### Mean shift clustering/segmentation
- Find features (color, gradients, texture, etc)
- Initialize windows at individual pixel locations
- Perform mean shift for each window until convergence
- Merge windows that end up near the same “peak” or mode

##### Results 
![[Pasted image 20251202181314.png]]
##### Pros and cons
- Pros
	- Does not assume spherical clusters
	- Just a single parameter (window size)
	- Finds variable number of modes
	- Robust to outliers
- Cons
	- Output depends on window size
	- Computationally expensive
	- Does not scale well with dimension of feature space

### Graph-based segmentation
- Represent features and their relationships using a graph
- Cut the graph to get subgraphs with strong interior links and weaker exterior links
##### Images as graphs
- Node for every pixel
- Edge between every pair of pixels (or every pair of “sufficiently close” pixels)
- Each edge is weighted by the **affinity** or similarity of the two nodes
Partitioning :
- Break Graph into Segments
	- Delete links that cross between segments
	- Easiest to break links that have low affinity
		- similar pixels should be in the same segments
		- dissimilar pixels should be in different segments

##### Measuring Affinity
**Distance** : $$\text{aff}(x,y) = \exp\left( -\frac{\lvert x-y \rvert^{2} }{2\sigma_{d}^{2}}  \right)$$
**Intensity** : $$\text{aff}(x,y) = \exp\left( -\frac{\lvert I(x)-I(y) \rvert^{2} }{2\sigma_{d}^{2}}  \right)$$**Color** : $$\text{aff}(x,y) = \exp\left( -\frac{\lvert c(x)-c(y) \rvert^{2} }{2\sigma_{d}^{2}}  \right)$$
##### Scale affects affinity
- Small $\sigma$: group only nearby points
- Large $\sigma$: group far-away points
### Graph cut
- Set of edges whose removal makes a graph disconnected
- Cost of a cut: sum of weights of cut edges
- A graph cut gives us a segmentation
	- What is a “good” graph cut and how do we find one?
![[Pasted image 20251202182608.png]]
![[Pasted image 20251202182617.png]]
##### Minimum cut
- We can do segmentation by finding the **minimum cut** in a graph
	- Efficient algorithms exist for doing this
- Drawback: minimum cut tends to cut off very small, isolated components
##### Normalized cut
- A minimum cut penalizes large segments
- This can be fixed by normalizing the cut by component size
- The normalized cut cost is:$$\frac{\text{cut}(A,B)}{\text{assoc(A,V)}} +\frac{\text{cut}(A,B)}{\text{assoc(B,V)}} $$with $\text{assoc}(A, V) =$ sum of weights of all edges in $V$ that touch $A$
- The exact solution is NP-hard but an approximation can be computed by solving a *generalized eigenvalue* problem
##### Pros and cons
- Pros
	- Generic framework, can be used with many different features and affinity formulations
- Cons
	- High storage requirement and time complexity
	- Bias towards partitioning into equal segments

### Using texture features for segmentation
- Texture descriptor is vector of filter bank outputs
- Textons are found by clustering
- Affinities are given by similarities of texton histograms over windows given by the “local scale” of the texture

# Image parsing
- Define generative models for text and faces
	- Deformable spline-based templates for characters![[Pasted image 20251202184226.png]]
	- PCA model for faces![[Pasted image 20251202184246.png]]
- **Top-down**: propose a model for a given region
- **Bottom-up**: verify the consistency of the model with image features

# Example results 
![[Pasted image 20251202184445.png]]![[Pasted image 20251202184455.png]]

# Cleaning up the result
- Problem:
	- Histogram-based segmentation can produce messy regions
		- segments do not have to be connected
		- may contain holes
- How can these be fixed?

### Dilation operator:
$$G = H \oplus F $$$F[x,y]$ :
![[Pasted image 20251202184634.png]]
$H[u,v]$ :
![[Pasted image 20251202184705.png]]
**Dilation** : does H “overlap” F around $[x,y]$ ? 
- $G[x,y] = 1$ if $H[u,v]$ and $F[x+u-1,y+v-1]$ are both $1$ somewhere $0$ otherwise
- Written $G = H \oplus F$

### Erosion operator:
$$G = H \ominus F$$
**Erosion**: is H “contained in” F around $[x,y]$
- $G[x,y] = 1$ if $F[x+u-1,y+v-1]$ is $1$ everywhere that $H[u,v]$ is $1$, $0$ otherwise
- Written $G = H \ominus F$
### Nested dilations and erosions
**Closing** : $G = H \ominus (H \oplus F)$

You can clean up binary pictures by applying combinations of dilations and erosions. Dilations, erosions, opening, and closing operations are
known as **morphological operations**.