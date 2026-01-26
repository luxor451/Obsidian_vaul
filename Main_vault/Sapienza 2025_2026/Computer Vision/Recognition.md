**Tags:** #ComputerVision #ObjectRecognition #Classification #SVM #MachineLearning #GenerativeModels
![[10-Recognition.pdf]]![[11-Recognition-2.pdf]]

# Outline
- What is object category recognition?
- History of object representation
	- Geometric
	- Global appearance-based
	- Sliding window
	- Indexing with local features
	- Constellation models
	- Bags of features
- Issues in statistical recognition
	- Generative vs. discriminative models
	- Supervised vs. unsupervised
	- Different kinds of recognition tasks
	- Datasets


How many object categories are there? 
**10000 to 30000**


### Modeling variability
Variability: 
- Camera position
- Illumination
- Internal parameters
- Within-class variations (2 chairs are not always the same)

### Timeline of recognition

- 1965-late 1980s: alignment, geometric primitives
- Early 1990s: invariants, appearance-based methods
- Mid-late 1990s: sliding window approaches
- Late 1990s: feature-based methods
- Early 2000s: parts-and-shape models
- 2003 – present: bags of features
- Present trends: combination of local and global methods, modeling context, integrating recognition and segmentation

### What works today 
- Reading license plates, zip codes, checks
- Fingerprint recognition
- Face detection
- Recognition of flat textured objects (CD covers, book covers, etc.)

# Issues in recognition
- The statistical viewpoint
 - Generative vs. discriminative methods
- Model representation
- Supervised vs. unsupervised methods
- Different recognition tasks
- Dataset

# Object categorization: the statistical viewpoint
![[2025-11-22-194125_hyprshot.png]]
- MAP decision : $$p(\text{zebra} | \text{image} ) \text{ vs } p(\text{no zebra}|\text{image)}$$
-  Bayes rule: $$p(\text{zebra} | \text{image} ) \propto p(\text{image} | \text{zebra}) \cdot p(\text{zebra})$$ 
![[2025-11-22-194406_hyprshot.png]]

• **Discriminative methods** : model posterior
• **Generative methods** : model likelihood and prior

### Discriminative methods

- Direct modeling of $p(\text{zebra} | \text{image} )$. 
- A model of the conditional probability of the target Y, given an observation x
![[2025-11-22-194955_hyprshot.png]]
- Learn a decision rule (classifier) assigning bag-of-features representations of images to different classes
### Generative methods

- Model of $p(\text{image} | \text{zebra}) \text{ and } p(\text{zebra})$
- A model of the conditional probability of the observable X, given a target y![[2025-11-22-195107_hyprshot.png]]
- Model the probability of a bag of featuresgiven a class
### Comparaison 
- Generative methods
	- + Interpretable
	- + Can be learned using images from just a single category
	- - Sometimes we don’t need to model the likelihood when all we want is to make a decision
- Discriminative methods
	- + Efficient
	+ + Often produce better classification rates
	- - Can be hard to interpret
	- - Require positive and negative training data


# Discriminative methods
### Classification
- Assign input vector to one of two or more classes
- Any decision rule divides input space into decision regions separated by decision boundaries
##### Nearest Neighbor Classifier
- Assign label of nearest training data point to each test data point

##### K-Nearest Neighbors
- For a new point, find the k closest points from training data
- Labels of the k points “vote” to classify
- Works well provided there is lots of data and the distance function is good K-Nearest Neighbors **k = 5**
### Functions for comparing histograms
- L1 distance $$D(h_{1}, h_{2}) = \sum_{i=1}^{N}\lvert h_{1}(i) - h_{2}(i) \rvert $$
- $\chi^{2}$ distance $$D(h_{1}, h_{2}) = \sum_{i=1}^{N}
\frac{(h_{1}(i) - h_{2}(i))^{2}}{h_{1}(i) + h_{2}(i)} $$
- Quadratic distance $$D(h_{1}, h_{2}) = \sum_{i,j} A_{i,j} (h_{1}(i) - h_{2}(j))^{2}$$
### Linear classifiers
- Find linear function (hyperplane) to separate positive and negative examples
![[2025-11-22-200332_hyprshot.png]]

### Support vector machines
- Find hyperplane that maximizes the margin between the positive and negative examples![[2025-11-22-200406_hyprshot.png]]
### Nonlinear SVMs
-  Datasets that are linearly separable work out great:![[2025-11-22-200450_hyprshot.png]]
- But what if the dataset is just too hard?![[2025-11-22-200454_hyprshot.png]]
- We can map it to a higher-dimensional space:![[2025-11-22-200458_hyprshot.png]]

-  General idea: the original input space can always be mapped to some higher-dimensional feature space where the training set is separable:![[2025-11-22-200528_hyprshot.png]]
### Review: Discriminative methods
- Nearest-neighbor and k-nearest-neighbor classifiers
	- L1 distance, χ2 distance, quadratic distance, Earth Mover’s Distance
- Support vector machines
	- Linear classifiers
	- Margin maximization
- Of course, there are many other classifiers out there
	- The kernel trick
	- Kernel functions: histogram intersection, generalized Gaussian, pyramid match
	- Neural networks, boosting, decision trees, …

### Summary: SVMs for image classification
1. Pick an image representation (in our case, bag of features)
2. Pick a kernel function for that representation
3. Compute the matrix of kernel values between every pair of training examples
4. Feed the kernel matrix into your favorite SVM solver to obtain support vectors and weights
5. At test time: compute kernel values for your test example and each support vector, and combine them with the learned weights to get the value of the decision function
##### What about multi-class SVMs?
- Unfortunately, there is no “definitive” multi-class SVM formulation
-  In practice, we have to obtain a multi-class SVM by combining multiple two-class SVMs
-  One vs. others
	- Traning: learn an SVM for each class vs. the others
	- Testing: apply each SVM to test example and assign to it the class of the SVM that returns the highest decision value
- One vs. one
	- Training: learn an SVM for each pair of classes
	- Testing: each learned SVM “votes” for a class to assign to the test example

## SVMs: Pros and cons
-  Pros
	- Many publicly available [SVM packages](http://www.kernel-machines.org/software)
	- Kernel-based framework is very powerful, flexible
	- SVMs work very well in practice, even with very small training sample sizes
- Cons
	- No “direct” multi-class SVM, must combine two-class SVMs
	- Computation, memory
	- During training time, must compute matrix of kernel values for every pair of examples
	- Learning can take a very long time for large-scale problems

# Generative methods
They model the probability distribution that produced
a given bag of features

### Naïve Bayes Model
- Assume that each feature is conditionally independent *given the class*$$p(w_{1}, \dots,w_{N} | c) = \prod_{i=1}^{N} p(w_{i}|c)$$
$$c^{*} = \text{argmax}_{c}\ p(c) \prod_{i=1}^{N}p(w_{i}|c)$$
$c^{*}$ : MAP decision
$p(c)$ : Prior prob. of the object classes
$p(w_{i}|c)$ : Likelihood of ith visual word given the class. Estimated by empirical frequencies of visual words in images from a given class

### Probabilistic Latent Semantic Analysis
- Unsupervised technique
- Two-level generative model: a document is a mixture of topics, and each topic has its own characteristic word distribution![[2025-12-02-172541_hyprshot.png]]

$$p(w_{i}|d_{j}) = \sum_{k=1}^{K} p(w_{i}|z_{k}) p(z_{k}|d_{j})$$
$p(w_{i}|d_{j})$ : Probability of word i in document j (known)
$p(w_{i}|z_{k})$ Probability of word i given topic k (unknown)
$p(z_{k}|d_{j})$ : Probability of topic k given document j (unknown)

##### Recognition
- Finding the most likely topic (class) for an image:
$$z^{*} =\text{argmax}_{z} p(z|d) $$
- Finding the most likely topic (class) for a visual word in a given image:
$$z^{*} =\text{argmax}_{z} p(z|w,d) = \text{argmax}_{z} \frac{p(w|z)p(z|d)}{\sum_{z'}p(w|z')p(z'|d)} $$
### Summary
 - **Naïve Bayes**
	- *Unigram* models in document analysis
	- Assumes conditional independence of words given class
	- Parameter estimation: frequency counting
- **Probabilistic Latent Semantic Analysis**
	- Unsupervised technique
	- Each document is a mixture of topics (image is a mixture of classes)
	- Can be thought of as matrix decomposition
	- What else to study? Parameter estimation: EM

# Issues for statistical recognition
- Representation
	- How to model an object category
- Learning
	- How to find the parameters of the model, given training data
- Recognition
	- How the model is to be used on novel data

### Representation
-  Generative / discriminative / hybrid
-  Appearance only or location and appearance
- Invariances
	- View point
	- Illumination
	- Occlusion
	- Scale
	- Deformation
	- Clutter
	- etc.
- Part-based, global, sliding window
- Use set of features or each pixel in image

### Learning
-  Unclear how to model categories, so we learn what distinguishes them rather than manually specify the difference -- hence current interest in machine learning)
- Methods of training: generative vs. discriminative
- Generalization, overfitting, bias vs. variance
- Level of supervision
	- Manual segmentation; bounding box; image labels; noisy labels
	- Task-dependent
# Dataset issues
- How large is the degree of intra-class variability?
- How “confusable” are the classes?
- Is there bias introduced by the background? I.e., can we “cheat” just by looking at the background and not the object?
- Care about **DATASET BIAS**!
# Summary
 Recognition is the “grand challenge” of computer vision
- History
	- Geometric methods
	- Appearance-based methods
	- Sliding window approaches
	- Local features
	- Parts-and-shape approaches
	- Bag-of-features approaches
- Issues
	- Generative vs. discriminative models
	- Supervised vs. unsupervised methods
	- Tasks, datasets