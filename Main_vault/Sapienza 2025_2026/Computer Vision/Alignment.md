**Tags:** #ComputerVision #ImageAlignment #Transformations #Homography #RANSAC #Features
![[9-Alignment.pdf]]

# Introduction 
- Two broad approaches:
	-  Direct (pixel-based) alignment
		- Search for alignment where most pixels agree
	- Feature-based alignment
		- Search for alignment where extracted features agree
		- Can be verified using pixel-based alignment

### Fitting a model to features in one image
![[2025-11-22-183302_hyprshot.png]]
We need to find a model that minimize $$\sum_{i}\text{residual}(x_{i}, M)$$
### Fitting a model to a transformation between pairs of features (matches) in two images
![[2025-11-22-183450_hyprshot.png]]
Find transformation $T$ that minimize $$\sum_{i} \text{residual}(T(x_{i}),x_{i}')$$

# Feature-based alignment outline
- Extract features
- Compute putative matches
- Loop:
	- Hypothesize transformation T (small group of putative matches that are related by T)
	-  Verify transformation (search for other matches consistent with T)
![[2025-11-22-183650_hyprshot.png]]![[2025-11-22-183707_hyprshot.png]]
# 2D transformation models
- Similarity (translation, scale, rotation)
![[2025-11-22-183831_hyprshot.png]]
- Affine
![[2025-11-22-183836_hyprshot.png]]
- Projective (homography)
 ![[2025-11-22-183840_hyprshot.png]]

### Affine transformations
- Simple fitting procedure (linear least squares)
- Approximates viewpoint changes for roughly planar objects and roughly orthographic cameras
- Can be used to initialize fitting for more complex models

##### Basic 2D transformation
- Scale $$\begin{bmatrix}
x' \\
y'
\end{bmatrix} = \begin{bmatrix}
s_{x} & 0  \\
0 & s_{y}
\end{bmatrix} \begin{bmatrix}
x \\
y
\end{bmatrix}$$
- Shear$$\begin{bmatrix}
x' \\
y'
\end{bmatrix} = \begin{bmatrix}
1 & \alpha_{x}  \\
\alpha_{y} & 1
\end{bmatrix} \begin{bmatrix}
x \\
y
\end{bmatrix}$$
- Rotate $$\begin{bmatrix}
x' \\
y'
\end{bmatrix} = \begin{bmatrix}
\cos(\theta) & -\sin(\theta)  \\
\sin(\theta) & \cos(\theta)
\end{bmatrix} \begin{bmatrix}
x \\
y
\end{bmatrix}$$
- Translate$$\begin{bmatrix}
x' \\
y'
\end{bmatrix} = \begin{bmatrix}
1 & 0 & t_{x} \\
0 & 1 & t_{y}
\end{bmatrix} \begin{bmatrix}
x \\
y \\
1
\end{bmatrix}$$
- **Affine** : it is any combination of the previous $$\begin{bmatrix}
x' \\
y'
\end{bmatrix} = \begin{bmatrix}
a & b & c \\
d & e & f
\end{bmatrix} \begin{bmatrix}
x \\
y \\
1
\end{bmatrix}$$
### Feature descriptors
- What if we don’t know the correspondences?
	-  Need to compare feature descriptors of local patches surrounding interest points


- Assuming the patches are already normalized (i.e., the local effect of the geometric transformation is factored out), how do we compute their similarity?
- Want invariance to intensity changes, noise, perceptually insignificant changes of the pixel pattern


- Simplest descriptor: vector of raw intensity values
- How to compare two such vectors?
	- Sum of squared differences (SSD)$$\text{SSD}(u,v) = \sum_{i}(u_{i}-v_{i})^{2}$$ *(Not invariant to intensity change)*
	-  Normalized correlation
		- Invariant to affine intensity change
		- Zero-Mean Normalized Sum of Squared Differences (ZNSSD)
		- Zero-Mean Normalized Sum of Absolute Differences (ZNSAD)

- Disadvantage of patches as descriptors:
	- Small shifts can affect matching score a lot
- Solution: histograms

##### SIFT 
- Descriptor computation:
	- Divide patch into $4\times4$ sub-patches
	- Compute histogram of gradient orientations (8 reference angles) inside each sub-patch
	- Resulting descriptor: $4\times4\times8 = 128$ dimensions
- Advantage over raw vectors of pixel values :
	- Gradients less sensitive to illumination change
	- “Subdivide and disorder” strategy achieves robustness to small shifts, but still preserves some spatial information

### Feature matching
- Generating *putative matches*: for each patch in one image, find a short list of patches in the other image that could match it based solely on appearance
	 - Exhaustive search
		- For each feature in one image, compute the distance to all features in the other image and find the “closest” ones (threshold or fixed number of top matches)
	- Fast approximate nearest neighbor search
		- Hierarchical spatial data structures (kd-trees, vocabulary trees)
		- Hashing
##### Feature space outlier rejection
- How can we tell which putative matches are more reliable?
- Heuristic: compare distance of nearest neighbor to that of second nearest neighbor
	- Ratio will be high for features that are not distinctive
	- Threshold of 0.8 provides good separation
##### Dealing with outliers
- The set of putative matches still contains a very high percentage of outliers
- How do we fit a geometric transformation to a small subset of all possible matches?
- Possible strategies:
	- RANSAC
	- Incremental alignment
	- Hough transform
	- Hashing
### Strategy 1 : [[Fitting#RANSAC|RANSAC]]

- RANSAC loop:
	1. Randomly select a seed group of matches
	2. Compute transformation from seed group
	3. Find inliers to this transformation
	4. If the number of inliers is sufficiently large, re- compute least-squares estimate of transformation on all of the inliers
- Keep the transformation with the largest number of inliers

##### Problem
- In many practical situations, the percentage of outliers (incorrect putative matches) is often-very high (90% or above)
- Alternative strategy: restrict search space by-using strong locality constraints on seed groups and inliers
	- Incremental alignment

### Strategy 2: Incremental alignment
- Take advantage of strong locality constraints: only pick close-by matches to start with, and gradually add more matches in the same neighborhood 
##### Detail:
- Generating seed Group :
	- Identify triples of neighboring features $(i, j, k)$ in first image
	- Find all triples $(i', j', k')$ in the second image such that $i'$ (resp. $j', k'$) is a putative match of i (resp. $j, k$), and $j', k'$ are neighbors of $i'$![[2025-11-22-190802_hyprshot.png]]

- Beginning with each seed triple, repeat:
	- Estimate the aligning transformation between corresponding features in current group of matches
	- Grow the group by adding other consistent matches in the neighborhood
- Until the transformation is no longer consistent or no more matches can be found

### Strategy 3: [[Fitting#Hough transform|Hough Transform]]
- Suppose our features are scale- and rotation-invariant
	- Then a single feature match provides an alignment hypothesis (translation, scale, orientation)
	- Of course, a hypothesis obtained from a single match is unreliable
	- Solution: let each match vote for its hypothesis in a Hough space with very coarse bins
##### Details : 
- Training phase: For each model feature, record 2D location, scale, and orientation of model (relative to normalized feature frame)
- Test phase: Let each match between a test and a model feature vote in a 4D Hough space
	- Use broad bin sizes of 30 degrees for orientation, a factor of 2 for scale, and 0.25 times image size for location
	- Vote for two closest bins in each dimension
- Find all bins with at least three votes and perform geometric verification
	- Estimate least squares affine transformation
	- Use stricter thresholds on transformation residual
	- Search for additional features that agree with the alignment

### Strategy 4: Hashing
- Make each invariant image feature into a low-dimensional “key” that indexes into a table of hypotheses
- Given a new test image, compute the hash keys for all features found in that image, access the table, and look for consistent hypotheses
- This can even work when we don’t have any feature descriptors: we can take n-tuples of neighboring features and compute invariant hash codes from their geometric configurations

# Beyond affine transformations
-  What is the transformation between two views of a planar surface?
- What is the transformation between images from two cameras that share the same center?
**Homography**: plane projective transformation (transformation taking a quad to another arbitrary quad)![[2025-11-22-191252_hyprshot.png]]

### Fitting a homography
-  Recall: homogenenous coordinates
	- Homogeneous image coordinates$$(x,y) \implies \begin{bmatrix}
x \\
y \\
1
\end{bmatrix}$$
	- Homogeneous image coordinates$$(x,y,z) \implies \begin{bmatrix}
x \\
y \\
z \\
1
\end{bmatrix}$$
Converting from homogenenous image coordinates : $$\begin{bmatrix}
x \\
y \\
w
\end{bmatrix} \implies \left( \frac{x}{w}, \frac{y}{w} \right) \text{ and }\begin{bmatrix}
x \\
y \\
z \\
w
\end{bmatrix} \implies\left( \frac{x}{w}, \frac{y}{w}, \frac{z}{w} \right)$$
Equation for homography: $$\lambda \begin{bmatrix}
x' \\
y' \\
1
\end{bmatrix} = \begin{bmatrix}
h_{11} & h_{12} & h_{13} \\
h_{21} & h_{22} & h_{23} \\
h_{31} & h_{32} & h_{33}
\end{bmatrix} \begin{bmatrix}
x \\
y \\
1
\end{bmatrix}$$
*9 entries, 8 degrees of freedom (scale is arbitrary)*

# Issues in alignment-based applications

- Choosing the geometric alignment model
	- Tradeoff between “correctness” and robustness (also, efficiency)
- Choosing the descriptor
	- “Rich” imagery (natural images): high-dimensional patch-based descriptors (e.g., SIFT)
	- “Impoverished” imagery (e.g., star fields): need to create invariant geometric descriptors from k-tuples of point-based features
- Strategy for finding putative matches
	- Small number of images, one-time computation (e.g., panorama stitching): brute force search
	- Large database of model images, frequent queries: indexing or hashing
	- Heuristics for feature-space pruning of putative matches
-  Hypothesis generation strategy
	- Relatively large inlier ratio: RANSAC
	- Small inlier ratio: locality constraints, Hough transform
- Hypothesis verification strategy
	- Size of consensus set, residual tolerance depend on inlier ratio and expected accuracy of the model
	- Possible refinement of geometric model
	- Dense verification