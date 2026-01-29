**Tags:** #ComputerVision #ModelFitting #HoughTransform #RANSAC #LeastSquares #LineFitting
![[8-Fitting.pdf]]
# Introduction
- Choose a parametric model to represent a set of features
-  Membership criterion is not local
	- Can’t tell whether a point belongs to a given model just by looking at that point
- Three main questions:
	-  What model represents this set of features best?
	- Which of several model instances gets which feature?
	- How many model instances are there?
-  Computational complexity is important-
	- It is infeasible to examine every possible set of parameters and every possible combination of features

### Issues
- Noise in the measured feature locations
-  Extraneous data: clutter (outliers), multiple lines
-  Missing data: occlusions
### Voting schemes
-  Let each feature vote for all the models that are compatible with it
-  Hopefully the noise features will not vote consistently for any single model
- Missing data doesn’t matter as long as there are enough features remaining to agree on a good model

# Hough transform

-  An early type of voting scheme
- General outline:
	- Discretize parameter space into bins
	- For each feature point in the image, put a vote in every bin in the parameter space that could have generated this point
	-  Find bins that have the most votes 
![[2025-11-20-165248_hyprshot.png]]

### Parameter space representation
- A line in the image corresponds to a point in Hough space 
![[2025-11-20-165354_hyprshot.png]]
- What does a point $(x_{0}, y_{0})$ in the image space map to in the Hough space?
	- Answer: the solutions of $b = –x_{0}m + y_{0}$
	- This is a line in Hough space 
![[2025-11-20-165530_hyprshot.png]]

- Where is the line that contains both $(x_{0}, y_{0})$ and $(x_1, y_1)$?
- It is the intersection of the lines $b = –x_0m + y_0$ and $b = –x_1m + y_1$
![[2025-11-20-165649_hyprshot.png]]
##### Polar representation for lines
- Problems with the $(m,b)$ space:
	- Unbounded parameter domain
	- Vertical lines require infinite $m$
- Alternative: polar representation 
![[2025-11-20-165946_hyprshot.png]]
**Each point will add a sinusoid in the $(\theta,\rho)$ parameter space**
### Algorithm outline
```
Initialize accumulator H to all zeros

For each edge point (x,y) in the image
	For θ = 0 to 180
		ρ = x cos θ + y sin θ
		H(θ, ρ) = H(θ, ρ) + 1
	end
end

Find the value(s) of (θ, ρ) where H(θ, ρ) is a local maximum

The detected line in the image is given by ρ = x cos θ + y sin θ
```
### Practical details

- Try to get rid of irrelevant features
	- Take edge points with significant gradient magnitude
- Choose a good grid / discretization
	- Too coarse: large votes obtained when too many different lines correspond to a single bucket
	-  Too fine: miss lines because some points that are not exactly collinear cast votes for different buckets
-  Increment neighboring bins (smoothing in accumulator array)
-  Who belongs to which line?
	-  Tag the votes
### Pros
- All points are processed independently, so can cope with occlusion
- Some robustness to noise: noise points unlikely to contribute consistently to any single bin
- Can deal with non-locality and occlusion
- Can detect multiple instances of a model in a single pass
### Cons
-  Complexity of search time increases exponentially with the number of model parameters
-  Non-target shapes can produce spurious peaks in parameter space
-  It’s hard to pick a good grid size

# A “grab-bag” of techniques

-  If we know which points belong to the line, how do we find the “optimal” line parameters?
	-  Least squares
	- [Probabilistic fitting](https://en.wikipedia.org/wiki/Probability_distribution_fitting) 
- What if there are outliers?
	- Robust fitting, RANSAC
- What if there are many lines?
	- Incremental fitting, K-lines
- What if we’re not even sure it’s a line?
	- Model selection
- Our main case study remains line fitting, but most ofthe concepts are very generic and widely applicable

# Least squares line fitting
- Data : $(x1, y1),\dots, (xn, yn)$
- Line equation: $y_i = m x_i + b$
- Find (m, b) to minimize
$$\begin{align}
E &= \sum_{i=1}^n(y_{i}-mx_{i}-b)^{2} \\
&=\sum_{i=1}^n(y_{i}-\begin{bmatrix}x_{i} & 1\end{bmatrix} \begin{bmatrix}m \\ b\end{bmatrix})^{2} \\
&= \begin{vmatrix}
\begin{bmatrix}
y_{1} \\ \vdots \\ y_{n}
\end{bmatrix}  - \begin{bmatrix}
x_{1} & 1 \\
\vdots & \vdots \\
x_{n} & 1
\end{bmatrix}  
 \begin{bmatrix}
m \\
b
\end{bmatrix}
\end{vmatrix} ^{2} \\
&=\lvert Y-XB \rvert^{2}  \\
&=(Y-XB)^{T} (Y-XB) = Y^{T}Y - 2(XB)^{T}Y+(XB)^{T}(XB)
\end{align}
$$

$$
\begin{align}
\frac{dE}{dB} &= 2X^{T}XB-2X^{T}Y =0 \\
X^{T}XB &= X^{T}Y
\end{align}
$$
This give us the Normal equation least squares solution to $$XB=Y$$
>[!WARNING] Fails completely for vertical lines

### Total least squares
- Distance between point $(x_n, y_n)$ and line $ax+by=d (a^2+b^2=1): |ax + by – d|$
- Find $(a, b, d)$ to minimize the sum of squared perpendicular distance
$$E = \sum_{i=1}^{n}(ax_{i}+by_{i}-d)^{2}$$

$$ \frac{\partial E}{\partial d} = \sum-2(ax_{i}+by_{i}-d) = 0$$
$$d=\frac{a}{n}\sum_{i=1}^{n}x_{i} + \frac{b}{n}\sum_{i=1}^{n} = a\bar{x}+b\bar{y}$$
$$E=\sum_{i=1}^{n}(a(x_{i}-\bar{x})+b(y_{i}-\bar{y}))^{2} = \begin{vmatrix}
\begin{bmatrix}
x_{1} -\bar{x} & y_{1} -\bar{y} \\
\vdots & \vdots \\
x_{n}-\bar{x} & y_{n}-\bar{y} 
\end{bmatrix}
\begin{bmatrix}
a  \\
b
\end{bmatrix}
\end{vmatrix}^{2} = 
(UN)^{T}UN$$
$$\frac{dE}{dN} = 2(U^{T}U)N = 0$$
Solution to $(U^{T}U)N = 0$, subject to $\lvert N \rvert^{2} = 1$: eigenvector of $U^{T}U$ associated with the smallest eigenvalue (least squares solution to homogeneous linear system $UN = 0$)

$$U = \begin{bmatrix}
x_{1} - \bar{x} & y_{1} - \bar{y} \\
\vdots & \vdots \\
x_{n} -\bar{x} &y_{n} -\bar{y}
\end{bmatrix}$$
Second moment matrix :
$$U^{T}U = \begin{bmatrix}
\sum_{i=1}^{n} (x_{i}-\bar{x})^{2} & \sum_{i=1}^{n} (x_{i}-\bar{x})(y_{i} -\bar{y}) \\
\sum_{i=1}^{n} (x_{i}-\bar{x})(y_{i} -\bar{y}) & \sum_{i=1}^{n} (y_{i}-\bar{y})^{2}
\end{bmatrix}$$![[2025-11-21-155646_hyprshot.png]]

### Least squares as likelihood maximization
Generative model : line points are corrupted by Gaussian noise in the direction perpendicular to the line
$$\begin{bmatrix}
x  \\
y
\end{bmatrix} = \begin{bmatrix}
u \\
v
\end{bmatrix} + \varepsilon \begin{bmatrix}
a \\
b
\end{bmatrix}$$
$$\begin{align}

&\begin{bmatrix}
u \\
v
\end{bmatrix} \text{ being the point of the line, } \\&\varepsilon \text{ the noise zero-mean
Gaussian with
std. dev.}\sigma \\
&\begin{bmatrix}
a \\
b
\end{bmatrix} \text{ the normal direction}
\end{align}
$$
### Least squares for general curves
-  We would like to minimize the sum of squared geometric distances between the data points and the curve
![[2025-11-21-160604_hyprshot.png]]
##### Calculating geometric distance

![[2025-11-21-160857_hyprshot.png]]

Curve tangent $$\left( \frac{\partial C}{\partial v} (u_{0}, v_{0}), -\frac{\partial C}{\partial u} (u_{0}, v_{0})\right)$$
The curve tangent must be orthogonal to the vector connecting $(x, y)$ with the closest point on the curve, $(u_0, v_0)$ :
$$
\begin{equation}
    \begin{cases}
      \frac{\partial C}{\partial v} (u_{0}, v_{0}) [x -u_{0}] - \frac{\partial C}{\partial v} (u_{0}, v_{0}) [y -u_{0}] = 0 \\
      C(u_{0}, v_{0}) =0
    \end{cases}\,
\end{equation}
$$
Must solve system of equations for $(u_0, v_0)$

### Least squares for conics
- Equation of a general conic: 
$$
\begin{align}
C(a, x) &= a · x = ax^2 + bxy + cy^2 + dx + ey + f = 0 \\
a &= [a, b, c, d, e, f] \\
x &= [x2, xy, y2, x, y, 1] \\
\end{align}
$$
-  Minimizing the geometric distance is non-linear even for a conic
- Algebraic distance: $C(a, x)$
- Algebraic distance minimization by linear least squares : 
$$\begin{bmatrix}
x_{1}^{2} & x_{1}y_{1} & y_{1}^{2} & x_{1} & y_{1} & 1 \\
x_{2}^{2} & x_{2}y_{2} & y_{2}^{2} & x_{2} & y_{2} & 1 \\
\vdots & \vdots & \vdots & \vdots & \vdots & \vdots \\
x_{n}^{2} & x_{n}y_{n} & y_{n}^{2} & x_{n} & y_{n} & 1
\end{bmatrix} \begin{bmatrix}
a  \\
b \\
c \\
d \\
e \\
f
\end{bmatrix} = 0$$

> [!TIP] Implementation
> In C++, linear systems like this are efficiently solved using the [[../Robot Programming/Eigen|Eigen Library]].

### Remarks
Least square is robust to noise but not to outliers

# Robust estimators

- General approach: minimize $\sum_{i} \rho(r_{i}(x_{i},\theta); \sigma)$
- $r_i (x_i, \theta)$ – residual of i-th point w.r.t. model parameters $\theta$
- $\rho$ – robust function with scale parameter $\sigma$
![[2025-11-21-162200_hyprshot.png]]
The robust function $\rho$ behaves like squared distance for small values of the residual $u$ but saturates for larger values of $u$

### Choosing the scale
- Just right : The effect of the outlier is eliminated
- Too small : The error value is almost the same for every point and the fit is very poor
- Too large: Behaves much the same as least squares

# RANSAC

- Robust fitting can deal with a few outliers – what if we have very many?
- Random sample consensus (RANSAC): Very general framework for model fitting in the presence of outliers
- Outline
	- Choose a small subset uniformly at random
	- Fit a model to that subset
	- Find all remaining points that are “close” to the model and reject the rest as outliers
	- Do this many times and choose the best model

### RANSAC for line fitting

-  Repeat $N$ times:
	- Draw $s$ points uniformly at random
	- Fit line to these $s$ points
	- Find inliers to this line among the remaining points (i.e., points whose distance from the line is less than $t$)
	- If there are $d$ or more inliers, accept the line and refit using all inliers
### Choosing the parameters
- Initial number of points $s$
	- Typically minimum number needed to fit the model
- Distance threshold $t$
	- Choose t so probability for inlier is p (e.g. 0.95)
	- Zero-mean Gaussian noise with std. dev. $σ: t^2=3.84σ^2$
- Number of samples $N$
	- Choose N so that, with probability p, at least one random sample is free from outliers (e.g. p=0.99) (outlier ratio: e)
-  Consensus set size $d$
	- Should match expected inlier ratio
### Pros
- Simple and general
- Applicable to many different problems
- Often works well in practice
### Cons
- Lots of parameters to tune
- Can’t always get a good initialization of the model based on the minimum number of samples
- Sometimes too many iterations are required
- Can fail for extremely low inlier ratios
- We can often do better than brute-force sampling

# Fitting multiple lines
- Voting strategies
	- Hough transform
	- RANSAC
- Other approaches
	- Incremental line fitting
	- K-lines
	- Expectation maximization

# Incremental line fitting

- Examine edge points in their order along an edge chain
- Fit line to the first s points
- While line fitting residual is small enough, continue adding points to the current line and refitting
- When residual exceeds a threshold, break off current line and start a new one with the next s “unassigned” points
### Pros
- Exploits locality
- Adaptively determines the number of lines
### Cons
- Needs sequential ordering of features
- Can’t cope with occlusion
- Sensitive to noise and choice of threshold

# K-Lines

-  Initialize k lines
	- Option 1: Randomly initialize k sets of parameters
	- Option 2: Randomly partition points into k sets and fit lines to them
- Iterate until convergence:
	- Assign each point to the nearest line
	- Refit parameters for each line
### Pros
- Guaranteed to reduce line fitting residual at each iteration
- Can cope with occlusion
### Cons
- Need to know k
- Can get stuck in local minima
- Sensitive to initialization

# Model selection
- Should we prefer a simpler or a more complex model?
-  Two issues
	-  Which model fits the observed data best?
	- Generalization performance: how well will a model predict points we haven’t seen before?

### Bias-variance tradeoff
- Models with too many parameters may fit a given sample better, but have high variance
- Generalization error is due to overfitting
- Models with too few parameters may not fit a given sample well because of high bias
- Generalization error is due to underfitting
### Occam’s razor
- Given several models that describe the data equally well, the simpler one should be preferred
- There should be some tradeoff between error and model complexity
- This is rarely done rigorously, but is a powerful “rule of thumb”
- Simpler models are often preferred because of their robustness (= low variance)
# Review of key concepts
- Least squares fitting
- Probabilistic fitting (we didn’t see in class, but you can do it)
	- The likelihood function
	- Estimates: maximum likelihood, MAP
- Dealing with outliers
	- Robust fitting
	- RANSAC
- Fitting multiple lines
	- Incremental fitting
	- Voting: Hough transform, RANSAC
	- Alternating minimization with “missing data”: K-lines
- Model selection
	- Bias vs. variance
	- Occam’s razor