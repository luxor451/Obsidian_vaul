**Tags:** #ComputerVision #OpticalFlow #MotionEstimation #LucasKanade #VideoProcessing
![[13-Optical-Flow.pdf]]
# Video
- A video is a sequence of frames captured over time
- Now our image data is a function of space $(x, y)$ and time $(t)$

# Motion Applications: Segmentation of video
- Background subtraction
	- A static camera is observing a scene
	- Goal: separate the static background from the moving foreground
	- Form an initial background estimate
		- For each frame:
			- Update estimate using a moving average
			- Subtract the background estimate from the frame
			- Label as foreground each pixel where the magnitude of the difference is greater than some threshold
			- Use median filtering to “clean up” the results
	- Shot boundary detection
		- For each frame
			- Compute the distance between the current frame and the previous one
				- Pixel-by-pixel differences
				- Differences of color histograms
				- Block comparison
			- If the distance is greater than some threshold, classify the frame as a shot boundary
- Motion segmentation
- Segment the video into multiple coherently moving objects

### More applications of motion
- Segmentation of objects in space or time
- Estimating 3D structure
- Learning dynamical models – how things move
- Recognizing events and activities
- Improving video quality (motion stabilization)
	- Feature-based methods
	- Extract visual features (corners, textured areas) and track them over multiple frames
	- Sparse motion fields, but more robust tracking
	- Suitable when image motion is large (10s of pixels)
- Direct, dense methods
	- Directly recover image motion at each pixel from spatio-temporal image brightness variations
	- Dense motion fields, but sensitive to appearance variations
	- Suitable for video and when image motion is small

# Motion estimation: Optical flow
**Optic flow** is the **apparent** motion of objects or surface

Will start by estimating motion of each pixel separately
Then will consider motion of entire image

### Problem definition: 
How to estimate pixel motion from image $I(x,y,t)$ to $I(x,y,t+1)$ ?
- Solve pixel correspondence problem
	- Given a pixel in $I(x,y,t)$, look for *nearby* pixels of the same color in $I(x,y,t+1)$

Key assumptions
- **Small motion**: Points do not move very far
- **Color constancy**: A point in $I(x,y,t)$ looks the same in $I(x,y,t+1)$
	- For grayscale images, this is brightness constancy

##### Constraints (grayscale images)
![[Pasted image 20251203160825.png]]
- Let’s look at these constraints more closely
	- Brightness constancy constraint (equation) $$I(x,y,t) = I(x+u, y+v, t+1)$$
	- Small motion: (u and v are less than 1 pixel, or smooth) Taylor series expension of $I$ : $$I(x+u, y+v) \approx I(x,y) + \frac{\partial I}{\partial x} u + \frac{\partial I}{\partial y} v$$

- Combining these 2 equations : $$
\begin{align} \\
&\text{Short hand : } I_{x} = \frac{\partial I}{\partial x} \\
0 &= I(x+u,y+v,t+1) - I(x,y,t) \\
&\approx I(x,y,t+1) + I_{x}u + I_{y}v-I(x,y,t) \\
&\approx[I(x,y,t+1) - I(x,y,t)] + I_{x}u + I_{y}v \\
&\approx I_{t} + I_{x}u + I_{y}v \\
&\approx I_{t}+ \nabla  I\cdot<u,v>
\end{align}$$

-  As $u$ and $v$ tend to zero, this become exact : $$0 = I_{t}+\nabla I \cdot <u,v> $$
- **Brightness constancy constraint equation** $$I_{x}u+I_{y}v+I_{t} = 0$$
### How does this make sense?
Can we use this equation to recover image motion (u,v) at each pixel?
- How many equations and unknowns per pixel?
	- One equation (this is a scalar equation!), two unknowns (u,v)

The component of the motion perpendicular to the gradient (i.e., parallel to the edge) cannot be measured

If $(u, v)$ satisfies the equation, so does $(u+u’, v+v’)$ if $\nabla I \cdot[u\text' \ \  v']^{T} = 0$
![[Pasted image 20251203162404.png]]

### Solving the ambiguity…
- How to get more equations for a pixel?
- Spatial coherence constraint
- Assume the pixel’s neighbors have the same $(u,v)$
	- If we use a 5x5 window, that gives us 25 equations per pixel $$0 = I_{t}p_{i} + \nabla I(p_{i})\cdot \begin{bmatrix}
u & v
\end{bmatrix}$$
$$\begin{bmatrix}
I_{x}(p_{1}) & I_{y}(p_{1}) \\
I_{x}(p_{2}) & I_{y}(p_{2}) \\
\vdots  &  \vdots \\
I_{x}(p_{25}) & I_{y}(p_{25}) \\
\end{bmatrix}
\begin{bmatrix}
u \\
v
\end{bmatrix} = - \begin{bmatrix}
I_{t}(p_{1}) \\
I_{t}(p_{2}) \\
\vdots \\
I_{t}(p_{25})
\end{bmatrix}$$
- Least squares problem with $$A\cdot d = b$$
- Least squares solution for d given by $(A^{T} A)d = A^{T}b$
$$\begin{bmatrix}
\sum I_{x}I_{x}  & \sum I_{x}I_{y} \\
\sum I_{x}I_{y}  & \sum I_{y}I_{y}
\end{bmatrix} \begin{bmatrix}
u \\
v
\end{bmatrix} = - \begin{bmatrix}
\sum I_{x}I_{t} \\
\sum I_{y}I_{t}
\end{bmatrix}
$$
>[!NOTE] The summations are over all pixels in the K x K window

When is this solvable? I.e., what are good points to track?
- $A^{T}A$ should be invertible
- $A^{T}A$ should not be too small due to noise
	- eigenvalues $\lambda_{1}$ and $\lambda_{2}$ of $A^{T}A$ should not be too small
- $A^{T}A$ should be well-conditioned
	- $\frac{\lambda_{1}}{\lambda_{2}}$should not be too large ($\lambda_{1} =$ larger eigenvalue)

>[!NOTE] Same Criteria for [[Blobs#Harris Detector: Mathematics|Harris corner detector]]

##### Low texture region
![[Pasted image 20251203164727.png]]
– gradients have small magnitude
– small $\lambda_{1}$, small $\lambda_{2}$
**BAD**
##### Edge
![[Pasted image 20251203164738.png]]
– large gradients, all the same
– large $\lambda_{1}$, small $\lambda_{2}$
**BAD**
##### High textured region
![[Pasted image 20251203164811.png]]
- gradients are different, large magnitudes
- large $\lambda_{1}$, large $\lambda_{2}$
**GOOD**
# Errors in Lucas-Kanade
- A point does not move like its neighbors
	- Motion segmentation
- Brightness constancy does not hold
	- Do exhaustive neighborhood search with normalized correlation - tracking features – maybe [[Alignment#SIFT|SIFT]] – more later….
- The motion is large (larger than a pixel)
	1. Not-linear: Iterative refinement
	2. Local minima: coarse-to-fine estimation

# Not tangent: Iterative Refinement
![[Pasted image 20251203165004.png]]
Iterative Lucas-Kanade Algorithm
1. Estimate velocity at each pixel by solving Lucas-Kanade equations
2. Warp It towards $I_{t+1}$ using the estimated flow field
	- use image warping techniques (image transformations)
3. Repeat until convergence


# Revisiting the small motion assumption
- Is this motion small enough?
	- Probably not—it’s much larger than one pixel
	- How might we solve this problem?
### Reduce the resolution!
Coarse-to-fine optical flow estimation
 ![[Pasted image 20251203165212.png]]![[Pasted image 20251203165226.png]]

# State-of-the-art optical flow
### 2009
Start with something similar to Lucas-Kanade
+ gradient constancy
+ energy minimization with smoothing term
+ region matching
+ keypoint matching (long-range)
### 2015
Deep convolutional network which accepts a pair of input frames and upsamples the estimated flow back to input resolution. Very fast because of deep network, near the state-of-the-art in terms of end-point-error.![[Pasted image 20251203165311.png]]

# Optical flow
- Definition: optical flow is the apparent motion of brightness patterns in the image
- Ideally, optical flow would be the same as the motion field
- Have to be careful: apparent motion can be caused by lighting changes without any actual motion
	- Think of a uniform rotating sphere under fixed lighting vs. a stationary sphere under moving illumination