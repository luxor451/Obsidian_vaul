**Tags:** #ComputerVision #EdgeDetection #Canny #Sobel #Gradients #ImageProcessing
![[6-Edge.pdf]]
# Introduction
**Goal**: Identify sudden changes (discontinuities) in an image
- Intuitively, most semantic and shape information from the image can be encoded in the edges
- More compact than pixels

**Ideal**: artist’s line drawing (but artist is also using object-level knowledge)

### Origin of Edges
Edges are caused by a variety of factor![[2025-10-16-174642_hyprshot.png]]

### Characterizing edges
**An edge is a place of rapid change in the image intensity function**![[2025-10-16-174737_hyprshot.png]]
# Image gradient
The gradient of an image: $$\nabla f = \left( \frac{\partial f}{\partial x}, \frac{\partial f}{\partial y} \right)$$![[2025-10-16-174853_hyprshot.png]]
The gradient points in the direction of most rapid increase in intensity

The **gradient direction** is given by : $$\theta = \arctan(\frac{\partial f}{\partial x} / \frac{\partial f}{\partial y})$$
The edge strength is given by the gradient magnitude $$||\nabla f|| = \sqrt{ \left( \frac{\partial f}{\partial x} \right)^{2} +\left( \frac{\partial f}{\partial y} \right)^{2}  }$$
### Finite difference filters
![[2025-10-16-175242_hyprshot.png]]
##### Exemple
![[2025-10-16-175302_hyprshot.png]]
### Effects of noise
Consider a single row or column of the image
- Plotting intensity as a function of position gives a signal![[2025-10-16-175354_hyprshot.png]]


- Finite difference filters respond strongly to noise
	- Image noise results in pixels that look very different from their neighbors
	- Generally, the larger the noise the stronger the response
- What is to be done?
	- Smoothing the image should help, by forcing pixels different to their neighbors (=noise pixels?) to look more like neighbors

##### Solution: smooth first
![[2025-10-16-175620_hyprshot.png]]
> [!NOTE] To find edges, look for peaks in $\frac{d}{dx}(f\circledast g)$

##### Derivative theorem of convolution
- Differentiation is convolution, and convolution is associative, This saves us one operation: $$\frac{d}{dx}(f\circledast g) = f\circledast \frac{d}{dx}g$$![[2025-10-16-180005_hyprshot.png]]

# Derivative of Gaussian filter
![[2025-10-16-180040_hyprshot.png]]
![[2025-10-16-180104_hyprshot.png]]
### Summary: Filter mask properties
- Filters act as templates
	- Highest response for regions that “look the most like the filter”
	- Dot product as correlation
- Smoothing masks
	- Values positive
	- Sum to 1 → constant regions are unchanged
	- Amount of smoothing proportional to mask size
- Derivative masks
	- Opposite signs used to get high response in regions of high contrast
	- Sum to 0 → no response in constant regions
	- High absolute value at points of high contrast
# Tradeoff between smoothing and localization

Smoothed derivative removes noise, but blurs edge. Also finds edges at different “scales”.![[2025-10-16-180254_hyprshot.png]]
### Implementation issues
- The gradient magnitude is large along a thick “trail” or “ridge,” so how do we identify the actual edge points?
- How do we link the edge points to form curves?![[2025-10-16-180334_hyprshot.png]]

# Edge finding
![[2025-10-16-180509_hyprshot.png]]

We wish to mark points along the curve where the magnitude is biggest.
We can do this by looking for a maximum along a slice normal to the curve (non- maximum suppression). These points should form a curve. There are then two algorithmic issues: at which point is the maximum, and where is the next one?

### Non-maximum suppression
![[2025-10-16-180539_hyprshot.png]]
At q, we have a maximum if the value is larger than those at both p and at r. Interpolate to get these values.![[2025-10-16-180613_hyprshot.png]]

##### Predicting the next edge point
![[2025-10-16-180648_hyprshot.png]]
Assume the marked point is an edge point. Then we construct the tangent to the edge curve (which is normal to the gradient at that point) and use this to predict the next points (here either r or s).

### Designing an edge detector

Criteria for an “optimal” edge detector:
- Good detection: the optimal detector must minimize the probability of false positives (detecting spurious edges caused by noise), as well as that of false negatives (missing real edges)
- Good localization: the edges detected must be as close as possible to the true edges
- Single response: the detector must return one point only for each true edge point; that is, minimize the number of local maxima around the true edge![[2025-10-16-180733_hyprshot.png]]

### Canny edge detector
- This is probably the most widely used edge detector in computer vision
- Theoretical model: step-edges corrupted by additive Gaussian noise
- Canny has shown that the first derivative of the Gaussian closely approximates the operator that optimizes the product of signal-to-noise ratio and localization
##### How it works ?
1. Filter image with derivative of Gaussian
2. Find magnitude and orientation of gradient
3. Non-maximum suppression:
	- Thin multi-pixel wide “ridges” down to single pixel width
4. Linking and thresholding (hysteresis):
	- Define two thresholds: low and high
	- Use the high threshold to start edge curves and the low threshold to continue them
##### Effect of $\sigma$ (Gaussian kernel spread/size)

The choice of $\sigma$ depends on desired behavior
- large $\sigma$ detects large scale edges
- small $\sigma$ detects fine features