**Tags:** #ComputerVision #NeuralNetworks #CNN #DeepLearning #MachineLearning #Backpropagation
![[15-Neural-Networks.pdf]]
# What are CNNs?
- [[Neural Network#Convolutional Neural Networks|Convolutional neural networks]] are a type of *neural network*
- The neural network includes layers that perform special operations
- Used in vision, but to a lesser extent also in NLP, biomedical, etc.
- Often they are *deep*

# Deep neural network
![[Pasted image 20251204181404.png]]

# Traditional Recognition Approach
![[Pasted image 20251204181448.png]]
- Features are key to recent progress in recognition, but research shows they’re flawed…
-  Where next? Better classifiers? Or keep building more features?

### What about learning the features?
- Learn a feature hierarchy all the way from pixels to classifier
- Each layer extracts features from the output of previous layer
- Train all layers jointly
![[Pasted image 20251204181535.png]]
### “Shallow” vs. “deep” architectures
- Traditional recognition: “Shallow” architecture ![[Pasted image 20251204181604.png]]
- Deep learning: “Deep” architecture![[Pasted image 20251204181626.png]]

# Biological neuron and Perceptron
- A biological neuron ![[Pasted image 20251204181655.png]]
- An artificial neuron (Perceptron) a linear classifier![[Pasted image 20251204181713.png]]
### Inspiration: Neuron cells
- Neurons
	- accept information from multiple inputs,
	- transmit information to other neurons.
- Multiply inputs by weights along edges
- Apply some function to the set of inputs at each node
- If output of function over threshold, neuron “fires”![[Pasted image 20251204181811.png]]![[Pasted image 20251204181820.png]]

# Multi-layer Neural Network

- A non-linear classifier
- **Training** : find network weights $w$ to minimize the error between true training labels $y_{i}$ and estimated labels $f_{w}(x_{i})$ $$E(w) = \sum_{i=1}^{N}(y_{i} - f_{w}(x_{i}))^{2}$$
- If $f$ i s *differentiable*, the minimization can be done by gradient descent
- This training method is called **back-propagation**

### A neuron 
- Inputs : $(x_{1}, x_{2}, \dots, x_{d})$
- Weights : $(w_{1}, w_{2}, \dots, w_{d})$
- Output : $\sigma(w\cdot x+b)$ with $\sigma_{t} = \frac{1}{1+e^{-t}}$

### Multilayer networks
- Cascade neurons together
- Output from one layer is the input to the next
- Each layer has its own sets of weights
![[Pasted image 20251204182330.png]]
- Predictions are fed forward through the network to classify

### Deep neural networks
- Lots of hidden layers
- Depth = power (usually)
![[Pasted image 20251204182451.png]]
### Activation functions
- Sigmoid : $$\sigma_{x} = \frac{1}{1+e^{-x}}$$
- tanh : $$\tanh(x)$$
- ReLU : $$\max(0,x)$$
- Leaky ReLU : $$\max(0.1x,x)$$
- Maxout : $$\max(w_{1}^{T}x + b_{1}, w_{2}^{T}x + b_{2})$$
- ELU :$$ f(x) = 
\begin{cases} 
      x & x > 0 \\
      \alpha (e^{x}-1) & x \leq 0 \\
   \end{cases}
$$

# How do we train them?
- The goal is to iteratively find such a set of weights that allow the activations/outputs to match the desired output
- We want to minimize a **loss function**
- The loss function is a function of the weights in the network
- For now let’s simplify and assume there’s a single layer of weights in the network
### Comments on training algorithm
- Not guaranteed to converge to zero training error, may converge to local optima or oscillate indefinitely.
- However, in practice, does converge to low error for many large networks on real data.
- Thousands of epochs (epoch = network sees all training data once) may be required, hours or days to train.
- To avoid local-minima problems, run several trials starting with different random weights (random restarts), and take results of trial with lowest training set error.
- May be hard to set learning rate and to select number of hidden units and layers.
- Neural networks had fallen out of fashion in 90s, early 2000s; back with a new name and significantly improved performance (deep networks trained with dropout and lots of data).

### Over-training prevention
- Running too many epochs can result in over-fitting.![[Pasted image 20251204183129.png]]
-  Keep a hold-out validation set and test accuracy on it after every epoch. Stop training when additional epochs actually increase validation error.

### Determining best number of hidden units
- Too few hidden units prevents the network from adequately fitting the data.
- Too many hidden units can result in over-fitting.![[Pasted image 20251204183219.png]]
- Use internal cross-validation to empirically determine an optimal number of hidden unit

### A note on training
- The more weights you need to learn, the more data you need
- That’s why with a deeper network, you need more data for training than for a shallower network
- That’s why if you have sparse data, you only train the last few layers of a deep net![[Pasted image 20251204183330.png]]
### Effect of number of neurons
![[Pasted image 20251204183354.png]]
- More neurons = more capacity

### Hidden unit interpretation
- Trained hidden units can be seen as newly constructed features that make the target concept linearly separable in the transformed space.
- On many real domains, hidden units can be interpreted as representing meaningful features such as vowel detectors or edge detectors, etc.
- However, the hidden layer can also become a distributed representation of the input in which each individual unit is not easily interpretable as a meaningful feature
### Summary
- We use deep neural networks because of their strong performance in practice
- Feed-forward network architecture
- Training deep neural nets
	- We need an objective function that measures and guides us towards good performance
	- We need a way to minimize the loss function: stochastic gradient descent
	- We need backpropagation to propagate error towards all layers and change weights at those layers
- Practices for preventing overfitting

# Convolutional Neural Networks
- Also known as CNN, ConvNet, DCN
- CNN = a multi-layer neural network with
1. Local connectivity
2. Weight sharing

### Local Connectivity
![[Pasted image 20251204183833.png]]
- \# input units (neurons): 7
- \# hidden units: 3
- Number of parameters
	- Global connectivity: 3 x 7 = 21
	- Local connectivity: 3 x 3 = 9

### Weight Sharing
![[Pasted image 20251204183923.png]]
- \# input units (neurons): 7
- \# hidden units: 3
- Number of parameters
	- Without weight sharing: 3 x 3 = 9
	- With weight sharing : 3 x 1 = 3
### Multiple input channels
![[Pasted image 20251204184026.png]]
### Multiple output maps
![[Pasted image 20251204184043.png]]
### Putting them together
- Local connectivity
- Weight sharing
- Handling multiple input channels
- Handling multiple output mapsPutting them togethe
![[Pasted image 20251204184121.png]]

### What is a Convolution?
-  Weighted moving sum![[Pasted image 20251204184157.png]]
### Convolutional Neural Networks
1. Input image!
![[Pasted image 20251204184231.png]]
2. Convolution (Learned)
![[Pasted image 20251204184306.png]]
3. Non-linearity. ReLU
 ![[Pasted image 20251204184334.png]]
 4. Spatial pooling 
	- Max-pooling: a non-linear down-sampling
	- Provide translation invariance
![[Pasted image 20251204184429.png]]
5. Normalization 
![[Pasted image 20251204184510.png]]
### Training Convolutional Neura Networks
- Backpropagation + stochastic gradient descent with momentum
- Dropout
- Data augmentation
- Batch normalization
- Initialization
	- Transfer learning
### Training CNN with gradient descent
 - A CNN as composition of functions $$f_{w}(x) = f_{L}(\dots(f_{2}(f_{1}(x,w_{1}),w_{2}) \dots), w_{L})$$![[Pasted image 20251204181404.png]]
 - Parameters $$w = (w_{1}, w_{2}, \dots, w_{L} )$$
 - Empirical loss function $$L_{w} = \frac{1}{n}\sum_{i}l(z_{i}, f_{w}(x_{i}))$$
 - Gradient descent $$w^{t+1} = w^{t} - \eta_{t}\frac{\partial f}{\partial w} (w^{t})$$
	with $$\begin{align}
w^{t+1} &\text{ : New weight} \\
w^{t} &\text{ : olg weight} \\
\eta_{t }&\text{ : Learning rate} \\ \\
\frac{\partial f}{\partial w} (w^{t}) &:\text{ Gradient}
\end{align}$$
>[!NOTE]
>[Gradient descent](https://en.wikipedia.org/wiki/Gradient_descent) is an optimization algorithm that’s used when training a machine learning model. It’s based on a convex function and tweaks its parameters iteratively to minimize a given function to its local minimum.

### Backpropagation (recursive chain rule)
![[Pasted image 20251210165828.png]]
### Dropout
Intuition: successful conspiracies
- 50 people planning a conspiracy
- Strategy A: plan a big conspiracy involving 50 people
	- Likely to fail. 50 people need to play their parts correctly.
- Strategy B: plan 10 conspiracies each involving 5 people
	- Likely to succeed!
![[Pasted image 20251210165923.png]]
**Main Idea**: approximately combining exponentially many different neural network architectures efficiently

### Data Augmentation (Jittering)
- Create virtual training samples
	- Horizontal flip
	- Random crop
	- Color casting
	- Geometric distortion

### Batch Normalization
Batch normalization can be implemented during training by calculating the mean and standard deviation of each input variable to a layer per mini-batch and using these statistics to perform the standardization.

**Input** : Values of $x$ over a mini-batch : $\mathbb{B} = \{x_{1\dots m}\}$;
Parameters to be learned:  $\gamma, \beta$
**Output** : $\{ y_{i} = \text{BN}_{\gamma,\beta}(x_{i})\}$
- $\mu_{\mathbb{B}} \leftarrow \frac{1}{m}\sum_{i=1}^{m}x_{i}$ *Mini-batch mean*
- $\sigma_{\mathbb{B}}^{2} \leftarrow \frac{1}{m}\sum_{i=1}^{m}(x_{i} - \mu_{\mathbb{B}})^{2}$ *Mini-batch variance*
- $\hat{x_{i}} \leftarrow \frac{x_{i} - \mu_{\mathbb{B}}}{\sqrt{ \sigma_{\mathbb{B}}^{2}+\varepsilon}}$ *Normalize*
- $y_{i} \leftarrow \gamma \hat{x_{{i}}} + \beta = \text{BN}_{\gamma,\beta}(x_{i})$ *Scale and shift*


# Deep learning library
- [TensorFlow](https://www.tensorflow.org/)
	- Research + Production
- [PyTorch](https://pytorch.org/)
	- Research
- [Caffe2](https://caffe2.ai/)
	- Production

# Things to remember
- Convolutional neural networks
	- A cascade of conv + ReLU + pool
	- Representation learning
	- Advanced architectures
	- Tricks for training CNN
- Visualizing CNN
	- Activation
	- Dissection
- Overview
	- Neuroscience, Perceptron, multi-layer neural networks
- Convolutional neural network (CNN)
	- [[Filters#Correlation & Convolution|Convolution]], nonlinearity, max pooling
	- CNN for classification and beyond
- Understanding and visualizing CNN
	- Find images that maximize some class scores; visualize individual neuron activation, input pattern and images; breaking CNNs
- Training CNN
	- Dropout; data augmentation; batch normalization; transfer learning