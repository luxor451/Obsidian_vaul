**Tags:** #ComputerVision #FeatureDetection #Blobs #Corners #Harris #SIFT #ScaleInvariant
![[7-Blobs.pdf]]
# Introduction
### Why extract features?
Motivation: panorama stitching
- We have two images – how do we combine them?![[2025-10-16-181417_hyprshot.png]]
1. Extract features
2. Match features
3. Align images
### Characteristics of good features

Characteristics of good features
- **Repeatability**
	- The same feature can be found in several images despite geometric and photometric transformations
- **Saliency**
	- Each feature has a distinctive description
- **Compactness and efficiency**
	- Many fewer features than image pixels
- **Locality**
	- A feature occupies a relatively small area of the image; robust to clutter and occlusion

### Applications
Feature points are used for:
- Motion tracking
- Image alignment
- 3D reconstruction
- Object recognition
- Indexing and database retrieval
- Robot navigation

# Finding Corners

-  Key property: in the region around a corner, image gradient has two or more dominant directions
-  Corners are repeatable and distinctive

### The Basic Idea
- We should easily recognize the point by looking through a small window
- Shifting a window in any direction should give a large change in intensity![[2025-10-16-181721_hyprshot.png]]

### Harris Detector: Mathematics
Change of intensity for the shift $[u,v]$
$$E(u,v) = \sum_{x,y}w(x,y) \cdot[I(x+u,y+v)-I(x,y)]^{2}$$
Window function : $w$, 1 in window and 0 outside
Shifted intensity : $I(x+u,y+v)$
Intensity : $I(x,y)$

##### Bilinear Approximation for small shifts:

 Second-order Taylor expansion of $E(u,v)$ about $(0,0)$  :
$$E(u,v) \approx E(0,0) + \begin{bmatrix} u & v \end{bmatrix}  \begin{bmatrix}
E_{u}(0,0) \\
E_{v}(0,0)
\end{bmatrix}
+ \frac{1}{2} \begin{bmatrix} u & v \end{bmatrix}
\begin{bmatrix}
E_{uu}(0,0) & E_{uv}(0,0) \\
E_{uv}(0,0) & E_{vv}(0,0)
\end{bmatrix}
\begin{bmatrix}
u \\
v
\end{bmatrix}
$$
The bilinear approximation simplifies to:$$E(u,v) \approx \begin{bmatrix} u & v \end{bmatrix} M \begin{bmatrix} u \\ v \end{bmatrix}$$
With
$$M = \sum_{x,y}w(x,y) \begin{bmatrix}
I_{x}^{2} & I_{x}I_{y} \\
I_{x}I_{y} & I_{y}^{2}
\end{bmatrix}$$
$$M = \begin{bmatrix}
\sum I_{x}I_{x} & \sum I_{x}I_{y} \\
\sum I_{x}I_{y} & \sum I_{y}I_{y}
\end{bmatrix}
=\sum \begin{bmatrix}
I_{x} \\ I_{y}
\end{bmatrix}
\begin{bmatrix}
I_{x} & I_{y}
\end{bmatrix}
= \sum \nabla I(\nabla I)^{T}
$$
##### Interpreting the second moment matrix
First, let's consider a axis-aligned corner :![[2025-10-17-120640_hyprshot.png]]
$$M = \begin{bmatrix}
\sum I_{x}^{2} & \sum I_{x}I_{y} \\
\sum I_{x}I_{y} & \sum I_{y}^{2}
\end{bmatrix} = \begin{bmatrix}
\lambda_{1} & 0 \\
0 & \lambda_{2}
\end{bmatrix}$$
This means dominant gradient directions align with x or y axis
If either $\lambda$ is close to 0, then this is not a corner, so look for locations where both are large.

**General Case :**
Since $M \in \mathcal{S}_{n}(\mathbb{R})$, we have $$M = R^{-1} \begin{bmatrix}
\lambda_{1} & 0 \\
0 & \lambda_{2}
\end{bmatrix} 
R$$We can visualize $M$ as an ellipse with axis lengths determined by the eigenvalues and orientation determined by $R$

**Ellipse equation :** $$\begin{bmatrix} u & v \end{bmatrix} M \begin{bmatrix} u \\ v \end{bmatrix} = \text{const}$$
![[2025-10-17-121300_hyprshot.png]]
##### Interpreting the eigenvalues
Classification of image points using eigenvalues of M: 
![[2025-10-17-121345_hyprshot.png]]
**Corner response function :**
$$R = \det(M) - \alpha \cdot\mathrm{Tr}(M)^{2} = \lambda_{1} \lambda_{2} - \alpha(\lambda_{1}+\lambda_{2})^{2}$$
$\alpha \in [0.04,0.06]$![[2025-10-17-121638_hyprshot.png]]
##### Summary of steps
![[2025-10-17-121835_hyprshot 1.png]]
Compute corner response $R$ ![[2025-10-17-121846_hyprshot.png]]
Find point where $R > \text{threshold}$![[2025-10-17-121854_hyprshot.png]]
Take only the point of local maxima of $R$![[2025-10-17-121903_hyprshot.png]]
![[2025-10-17-121910_hyprshot 1.png]]

1. Compute Gaussian derivatives at each pixel
2. Compute second moment matrix $M$ in a Gaussian window around each pixel
3. Compute corner response function $R$
4. Threshold $R$
5. Find local maxima of response function (non-maximum suppression)
# Invariance
We want features to be detected despite geometric or photometric changes in the image: 
- if we have two transformed versions of the same image, features should be detected in corresponding locations

### Models of Image Change : 
- Geometric :
	- Rotation
	- Scale
	- Affine (valid for: orthographic camera, locally planar object)
- Photometric
	- Affine intensity change ($I \rightarrow aI+b$)
### Harris Detector: Invariance Properties
##### Rotation
Ellipse rotates but its shape (i.e. eigenvalues) remains the same
Corner response $R$ is invariant to image rotation
##### Affine intensity change
- Only derivatives are used => invariance to intensity shift $I \rightarrow I+b$
- Intensity scale: $I \rightarrow aI$![[2025-10-17-122524_hyprshot.png]]
*Partially* invariant to affine intensity change

##### Scaling
![[2025-10-17-122608_hyprshot.png]]
*Not invariant* to scaling
### Scale-invariant feature detection

- Goal : independently detect corresponding regions in scaled versions of the same image
-  Need *scale selection* mechanism for finding characteristic region size that is *covariant* with the image transformation

# Scale-invariant features: Blobs
[[Egde#Image gradient#Effects of noise|Recall Edge detection]]
### From edges to blobs
- Edge = ripple
- Blob = superposition of two ripples
![[2025-10-17-123117_hyprshot.png]]
**Spatial selection**: the magnitude of the Laplacian response will achieve a maximum at the center of the blob, provided the scale of the Laplacian is “matched” to the scale of the blob

### Scale selection
- We want to find the characteristic scale of the blob by convolving it with Laplacians at several scales and looking for the maximum response
- However, Laplacian response decays as scale increases: (original radius = 8)
![[2025-10-17-123228_hyprshot.png]]
##### Scale normalization 
The response of a derivative of Gaussian filter to a perfect step edge decreases as $\sigma$ increases![[2025-10-17-123326_hyprshot.png]]
- The response of a derivative of Gaussian filter to a perfect step edge decreases as $\sigma$ increases
- To keep response the same (scale-invariant), must multiply Gaussian derivative by $\sigma$
- Laplacian is the second Gaussian derivative, so it must be multiplied by $\sigma^{2}$
##### Effect of scale normalization![[2025-10-17-123506_hyprshot.png]]
# Blob detection in 2D

Laplacian of Gaussian: Circularly symmetric operator for blob detection in 2D
$$\nabla^{2}g = \frac{\partial^{2} g}{\partial x^{2}} + \frac{\partial^{2} g}{\partial y^{2}}$$
**Scale-normalized :** $$\nabla_{\text{norm}}g = \sigma^{2} \left ( \frac{\partial^{2} g}{\partial x^{2}} + \frac{\partial^{2} g}{\partial y^{2}} \right )$$
### Scale selection
The 2D Laplacian is given by : $$(x^{2}+y^{2}-2\sigma^{2})e^{\frac{-(x^{2}+y^{2})}{2\sigma^{2}}}$$
Therefore, for a binary circle of radius r, the Laplacian achieves a maximum at $$\sigma = \frac{r}{\sqrt{ 2 }}$$
### Characteristic scale
We define the **characteristic scale** as the scale that produces peak of Laplacian response![[2025-10-17-123908_hyprshot.png]]
### Scale-space blob detector
1. Convolve image with scale-normalized Laplacian at several scales
2. Find maxima of squared Laplacian response in scale-space
### Efficient implementation
Approximating the Laplacian with a difference of Gaussians , $L$ is the Laplacian and $\text{DoG}$ is the Difference of Gaussians
$$L = \sigma \left ( G_{xx}(x,y,\sigma) + G_{yy} (x,y,\sigma) \right )$$ $$\text{DoG} = G(x,y,k\sigma) - G(s,y,\sigma)$$
### Affine adaptation
 [[Blobs#Finding Corners#Bilinear Approximation for small shifts|Recall]] 
 - The second moment ellipse can be viewed as the “characteristic shape” of a region
- We can normalize the region by transforming the ellipse into a unit circle

**Orientation ambiguity**
-  There is no unique transformation from an ellipse to a unit circle
	- We can rotate or flip a unit circle, and it still stays a unit circle
- So, to assign a unique orientation to key points:
	- Create histogram of local gradient directions in the patch
	- Assign canonical orientation at peak of smoothed histogram

- Problem: the second moment “window” determined by weights $w(x,y)$ must match the characteristic shape of the region
- Solution: iterative approach
	- Use a circular window to compute second moment matrix
	- Perform affine adaptation to find an ellipse-shaped window
	- Recompute second moment matrix using new window and iterate

### Invariance vs. covariance
**Invariance :**
- features(transform(image)) = features(image)
 **Covariance :**
- features(transform(image)) = transform(features(image)
