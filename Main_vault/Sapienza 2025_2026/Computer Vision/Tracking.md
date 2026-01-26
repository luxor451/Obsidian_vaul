**Tags:** #ComputerVision #ObjectTracking #KalmanFilter #ParticleFilter #MotionAnalysis
![[14-Tracking.pdf]]

# Feature tracking
- So far, we have only considered [[Optical Flow#Motion estimation: Optical flow|optical flow]] estimation in a pair of images
- If we have more than two images, we can compute the optical flow from each frame to the next
- Given a point in the first image, we can in principle reconstruct its path by simply “following the arrows
### Tracking challenges
- Ambiguity of optical flow
	- Find good features to track
- Large motions
	- Discrete search instead of Lucas-Kanade
- Changes in shape, orientation, color
	- Allow some matching flexibility
- Occlusions, disocclusions
	- Need mechanism for deleting, adding new features
- Drift – errors may accumulate over time
	- Need to know when to terminate a track

### Handling large displacements
- Define a small area around a pixel as the template
- Match the template against each pixel within a search area in next image – just like stereo matching!
- Use a match measure such as SSD or correlation
- After finding the best discrete location, can use Lucas-Kanade to get sub-pixel estimate

### Tracking over many frames

- Select features in first frame
- For each frame:
	- Update positions of tracked features
		- Discrete search or Lucas-Kanade
	- Terminate inconsistent tracks
		- Compute similarity with corresponding feature in the previous frame or in the first frame where it’s visible
	- Start new tracks if needed
### Shi-Tomasi feature tracker
- Find good features using eigenvalues of second- moment matrix
	- Key idea: “good” features to track are the ones that can be tracked reliably
- From frame to frame, track with Lucas-Kanade and a pure translation model
	- More robust for small displacements, can be estimated from smaller neighborhoods
- Check consistency of tracks by *affine* registration to the first observed instance of the feature
	- Affine model is more accurate for larger displacements
	- Comparing to the first frame helps to minimize drift

### Tracking with dynamics

- Key idea: Given a model of expected motion, predict where objects will occur in next frame, even before seeing the image
	- Restrict search for the object
	- Improved estimates since measurement noise is reduced by trajectory smoothness
# General model for tracking
- The moving object of interest is characterized by an underlying state $X$
- State $X$ gives rise to *measurements* or *observations* Y
- At each time $t$, the state changes to $X_t$ and we get a new observation $Y$
![[Pasted image 20251203170615.png]]
### Steps of tracking
- **Prediction**: What is the next state of the object given past measurements ?$$P(X_{t}|Y_{0} = y_{0}, \dots, Y_{t-1} = y_{t-1})$$
- **Correction**: Compute an updated estimate of the state from prediction and measurements $$P(X_{t}|Y_{0} = y_{0}, \dots, Y_{t-1} = y_{t-1}, Y_{t} = y_{t})$$
- Tracking can be seen as the process of propagating the posterior distribution of state given measurements across time

### Simplifying assumptions
- Only the immediate past matters $$P(X_{t}|X_{0}, \dots, X_{t-1}) = P(X_{t}|X_{t-1})$$
- Measurements depend only on the current state$$P(Y_{t}|X_{0}, Y_{0}, \dots, X_{t-1}, Y_{t-1}, X_{t}) = P(Y_{t}|X_{t})$$
### Tracking as induction
- Base case:
	- Assume we have initial prior that predicts state in absence of any evidence: $P(X_0)$
	- At the first frame, correct this given the value of $Y_0=y_0$ $$P(X_{0}|Y_{0} = y_{0}) = \frac{P(y_{0}|X_{0})P(X_{0})}{P(y_{0})} \propto P(y_{0}|X_{0})P(X_{0})$$
- Given corrected estimate for frame $t$:
	- Predict for frame $t+1$
	- Correct for frame $t+1$
### Induction step: Prediction
-  Prediction involves representing$P(X_{t}|y_{0},\dots,y_{t-1})$ given $P(X_{t-1}|y_{0},\dots,y_{t-1})$ :$$
\begin{align}
P(X_{t}|y_{0},\dots,y_{t-1} ) &= \int P(X_{t},X_{t-1}|y_{0},\dots,y_{t-1}) dX_{t-1} \text{  (Law of total probability)} \\ 
 &= \int P(X_{t}|X_{t-1},y_{0},\dots,y_{t-1}) P(X_{t-1}| y_{0},\dots,y_{t-1})dX_{t-1} \text{ (Conditioning on } X_{t-1} )\\ 

&= \int P(X_{t}|X_{t-1}) P(X_{t-1}| y_{0},\dots,y_{t-1})dX_{t-1} \text{ (Independence assumption)}
\end{align}$$

### Induction step: Correction
-  Correction involves computing$P(X_{t}|y_{0},\dots,y_{t})$ given $P(X_{t}|y_{0},\dots,y_{t-1})$ :$$
\begin{align}
P(X_{t}|y_{0},\dots,y_{t}) &= \frac{P(y_{t}|X_{t},y_{0},\dots,y_{t-1}) P(X_{t}|y_{0},\dots,y_{t-1})}{P(y_{t}|y_{0},\dots,y_{t-1})} \text{ (Bayes rule)}\\
&= \frac{P(y_{t}|X_{t}) P(X_{t}|y_{0},\dots,y_{t-1})}{P(y_{t}|y_{0},\dots,y_{t-1})} \text{ (Independence assumption (observation} y_{t} \text{depends only on state } X_t))  \\
&= \frac{P(y_{t}|X_{t}) P(X_{t}|y_{0},\dots,y_{t-1})}{\int P(y_{t}|X-t)P(X_{t}|y_{0}, \dots, y_{t-1})dX_{t}} \text{ (Conditioning on } X_{t} )
\end{align}$$

### Summary: Prediction and correction

**Prediction** :
$$P(X_{t}|y_{0},\dots,y_{t-1} ) = \int P(X_{t}|X_{t-1}) P(X_{t-1}| y_{0},\dots,y_{t-1})dX_{t-1} $$


**Correction** :
$$P(X_{t}|y_{0},\dots,y_{t}) = \frac{P(y_{t}|X_{t}) P(X_{t}|y_{0},\dots,y_{t-1})}{\int P(y_{t}|X_t)P(X_{t}|y_{0}, \dots, y_{t-1})dX_{t}}$$
# Linear Dynamic Models
- Dynamics model: state undergoes linear transformation plus Gaussian noise$$X_{t} \sim N\left( D_{t}x_{t-1}, \Sigma_{d_{t}} \right)$$
- Observation model: measurement is linearly transformed state plus Gaussian noise$$X_{t} \sim N\left( M_{t}x_{t}, \Sigma_{m_{t}} \right)$$
### Example: Constant velocity (1D)
- State vector is position and velocity (greek letters denote noise terms)
$$x_{t} = \begin{bmatrix}
p_{t} \\
v_{t}
\end{bmatrix} \text{ with } \begin{align}
p_{t} &= p_{t-1} +(\Delta t)v_{t-1} + \varepsilon \\
v_{t} &=v_{t-1} + \xi
\end{align}
$$

$$x_{t} = D_{t}x_{t-1} + \text{noise} = \begin{bmatrix}
1  & \Delta t \\
0 & 1
\end{bmatrix} \begin{bmatrix}
p_{t-1} \\
v_{t-1}
\end{bmatrix} + \text{noise}$$
- Measurement is position only $$y_{t} = Mx_{t} + \text{noise} = \begin{bmatrix}
1  & 0
\end{bmatrix} \begin{bmatrix}
p_{t} \\
v_{t}
\end{bmatrix} + \text{noise}$$
### Example: Constant acceleration (1D)
- State vector is position, velocity, and acceleration (greek letters denote noise terms)
$$x_{t} = \begin{bmatrix}
p_{t} \\
v_{t} \\
a_{t}
\end{bmatrix} \text{ with } \begin{align}
p_{t} &= p_{t-1} +(\Delta t)v_{t-1} + \varepsilon \\
v_{t} &=v_{t-1} + (\Delta t)a_{t-1} + \xi \\
a_{t} &= a_{t-1} + \zeta
\end{align}$$
$$x_{t} = D_{t}x_{t-1} + \text{noise} = \begin{bmatrix}
1  & \Delta t  & 0\\
0 & 1 & \Delta t \\
0 & 0 & 1
\end{bmatrix} \begin{bmatrix}
p_{t-1} \\
v_{t-1} \\
a_{t-1}
\end{bmatrix} + \text{noise}$$
- Measurement is position only $$y_{t} = Mx_{t} + \text{noise} = \begin{bmatrix}
1  & 0  & 0
\end{bmatrix} \begin{bmatrix}
p_{t} \\
v_{t} \\
a_{t}
\end{bmatrix} + \text{noise}$$
# The Kalman filter
- Method for tracking linear dynamical models in Gaussian noise
- The predicted/corrected state distributions are Gaussian
	- You only need to maintain the mean and covariance
	- The calculations are easy (all the integrals can be done in closed form)

### 1D state
1. Make **measurement**
2. **Correct** : Given prediction of state and current measurement, update prediction of state $P(X_{t}|y_{0},\dots,y_{t} )$
	- Mean and std. dev. of corrected state $\mu_{t}^{+}, \sigma_{t}^{+}$
3. **Time advance** from $t-1$ to $t$
4. **Predict** : Given corrected state from previous time step and all the measurements up to the current one, predict the distribution over the current step $P(X_{t}|y_{0},\dots,y_{t-1} )$
	- Mean and std. dev. of predicted state $\mu_{t}^{-}, \sigma_{t}^{-}$
5. **Repeat**

### General case
 - Predict $$\begin{align}
x_{t}^{-} &= D_{t}x_{t-1}^{+} \\
\Sigma_{t}^{-} &= D_{t}\Sigma_{t-1}^{+}D_{t}^{T} + \Sigma_{d_{t}}
\end{align}
 $$
 - Correct $$\begin{align}
K_{t} &= \Sigma_{t}^{-} M_{t}^{T}(M_{t}\Sigma_{t}^{-}M_{t}^{T}+\Sigma_{m-t})^{-1} \\
x_{t}^{+} &= x_{t}^{-} + K_{t}(y_{t}-M_{t}x_{t}^{-}) \\
\Sigma_{t}^{+} &= (I-K_{t}M_{t})\Sigma_{t}^{-}
\end{align}$$
### Pros and Cons 
- Pros
	- Simple updates, compact and efficient
- Cons
	- Unimodal distribution, only single hypothesis
	- Restricted class of motions defined by linear model

# Factored sampling
- Represent the state distribution non-parametrically
	- Prediction: Sample points from prior density for the state, $P(X)$
	- Correction: Weight the samples according to $P(Y|X)$
$$P(X_{t}|y_{0},\dots,y_{t}) = \frac{P(y_{t}|X_{t}) P(X_{t}|y_{0},\dots,y_{t-1})}{\int P(y_{t}|X_t)P(X_{t}|y_{0}, \dots, y_{t-1})dX_{t}}$$
### Particle filtering
- We want to use sampling to propagate densities over time (i.e., across frames in a video sequence)
- At each time step, represent posterior $P(X_t|Y_t)$ with weighted sample set
- Previous time step’s sample set $P(X_{t-1}|Y_{t-1})$ is passed to next time step as the effective prior
![[Pasted image 20251204180003.png]]
- Start with weighted samples from previous time step 
- Sample and shift according to dynamics model 
- Spread due to randomness; this is predicted density $P(X_t|Y_{t-1})$ 
- Weight the samples according to observation density 
- Arrive at corrected density estimate $P(X_t|Y_t)$

### Results 
![[Pasted image 20251204180027.png]]

# Tracking issues
- Initialization
	- Manual
	- Background subtraction
	- Detection
- Obtaining observation and dynamics model
	- Generative observation model: “render” the state on top of the image and compare
	- Discriminative observation model: classifier or detector score
	- Dynamics model: learn (very difficult) or specify using domain knowledge
- Prediction vs. correction
	- If the dynamics model is too strong, will end up ignoring the data
	- If the observation model is too strong, tracking is reduced to repeated detection
- Data association
	- What if we don’t know which measurements to associate with which tracks?
- Drift
	- Errors caused by dynamical model, observation model, and data association tend to accumulate over time

### Drift
![[Pasted image 20251204180307.png]]
### Data association
- So far, we’ve assumed the entire measurement to be relevant to determining the state
- In reality, there may be uninformative measurements (clutter) or measurements may belong to different tracked objects
- **Data association**: task of determining which measurements go with which tracks
![[Pasted image 20251204180408.png]]
- Simple strategy: only pay attention to the measurement that is “closest” to the prediction![[Pasted image 20251204180440.png]]
- Doesn’t always work…![[Pasted image 20251204180454.png]]
- More sophisticated strategy: keep track of multiple state/observation hypotheses
	- Can be done with particle filtering
- This is a general problem in computer vision, there is *no easy solution*