**Tags:** #ComputerVision #ImageFormation #Sensors #Digitalization #Sampling #Quantization
![[2-Aquisition.pdf]]
# Image Formation

| Light Source            | i(candela) |
| ----------------------- | ---------- |
| Solar sky               | 9000       |
| Cloudy sky              | 1000       |
| Moon light              | 0.01       |
| Internal Light (Office) | 100        |

| Object                  | r    |
| ----------------------- | ---- |
| Black velvet            | 0.01 |
| White wall              | 0.8  |
| Silver and other metals | 0.9  |
| Snow                    | 0.93 |
# Surface Properties

- Absorption
- Diffusion
- Reflex
- Transparency
- Refraction
# Image Acquisition 
![[2025-10-08-142851_hyprshot.png]]

# Mathematically
### Image Formation

**Light source** : $$E(x,y,z,\lambda)$$
**Reflective function for each point** $$r(x,y,z,\lambda)$$
**Reflected light is captures**$$c(x,y,z,\lambda) = E(x,y,z,\lambda) \times r(x,y,z,\lambda)$$
Inside the camera there is a projection of the image to a plane
**Projection $\mathcal{P}$ from world coordinate $x,y,z$ to image coordinate $x',y'$** $$c_{p}(x',y',\lambda) = \mathcal{P}(c(x,y,z,\lambda))$$
### Projections
Two types:
-  *Perspective projection*![[2025-10-08-143703_hyprshot.png]]
	-  Closer is bigger.
	- Human eye and camera under this model
	-  natural model, but mathematically more complicate
 

- *Ortographic projection*![[2025-10-08-143710_hyprshot.png]]
	-  Object size independent of the distance from the capturing device
	-  unnatural, but mathematically easier

Once we have $c_{p}(x',y',\lambda)$, the characteristic of the capture device takes over.

### Sensitivity

$V(\lambda)$ is the **sensitivity function**, it determine how sensitive the sensor is to capturing the range of wavelength present in $c_{p}(x',y',\lambda)$ 

The results is an *image function*  which determine the amount of reflected light that is captured at the camera coordinate $x',y'$ 
$$f(x',y') = \int_{\lambda} c_{p}(x',y',\lambda) \cdot V(\lambda)\ d\lambda $$

### Colors

For a color camera they are 3 Sensitivity function (R,G,B) and thus 3 image functions :
$$f_{R}(x',y') = \int_{\lambda} c_{p}(x',y',\lambda) \cdot V_{R}(\lambda)\ d\lambda$$$$f_{G}(x',y') = \int_{\lambda} c_{p}(x',y',\lambda) \cdot V_{G}(\lambda)\ d\lambda$$$$f_{B}(x',y') = \int_{\lambda} c_{p}(x',y',\lambda) \cdot V_{B}(\lambda)\ d\lambda$$
The image function is the result of :
- *Incident light* $E(x,y,z,\lambda)$ at the point $x,y,z$
- The *reflectivity function* $r(x,y,z,\lambda)$ of this point
- The formation of the *reflected light* $c(x,y,z,\lambda)$ 
- The *projection* of the reflected light from the 3D space to the 2D camera sensor $\mathcal{P}$
- The *sensitivity* of the camera $V(\lambda)$

### Digitalization

##### Fundamentals of Digital Images
- An image: *a multidimensional function of spatial coordinates*
- Spatial coordinate: 
	- $(x,y)$ for 2D case such as photograph,
	- $(x,y,z)$ for 3D case such as CT scan images
	- $(x,y,t)$ for movies
- The function $f$ may represent intensity (for monochrome images) or color (for color images) or other associated values.

The projected image is in a continuum domain and co-domain:
$$(x',y') \in \mathbb{R}^{2} \text{ and } f(x',y')$$
Digital computers cannot process parameters/functions that vary in a
continuum
We need to *discretize*:
- Sampling : $$(x',y') \to (x_{i}',y_{j}') \in \mathbb{N}^{2} \text{ with } (i=1, \dots , N-1; j=0, \dots M-1)$$
- Quantization : $$f(x_{i}', y_{i}' \to \hat{f}(x_{i}', y_{i}') \in \mathbb{N})$$
##### Spatial Sampling 1D
![[2025-10-08-182237_hyprshot.png]]
We can think as the multiplication between the signal and a *comb function*

##### Spatial Sampling 2D

Same think but with this comb function
![[2025-10-08-182342_hyprshot.png]]
$$comb(x',y') = \sum_{i=0}^{N-1} \sum_{j=0}^{M-1}\delta(x' -i\Delta_{x}, y' - j\Delta_{y})$$
The sampling is $$f_{c}(x',y') \times comb(x',y')$$
$$f(x_{i}',y_{j}') = f(x',y') \times comb(x',y')$$
For simplicity $f(x_{i}', y_{j}')= f(i,j)$

### Image Quantization
**Image quantization**: discretize continuous pixel values into discrete numbers

**Color resolution**/ color depth/ levels:
- No. of colors or gray levels or
- No. of bits representing each pixel value
- No. of colors or gray levels Nc is given by $N_{c} = 2^{b}$ where b = no. of bits
### Images as Discrete Functions

After spatial sampling and quantization, an image is a discrete function, The image domain $\Omega$ is now discrete, $\Omega \in \mathbb{N}$ and on is the image range :
$I : \Omega \to \{1,\dots,P\}$ where $p \in \mathbb{N}$. Typically, $P = 2^{8}$ *8 bit quantization*

The data structure for an image is simply a 2D array of values

# Image Types

**Intensity image or monochrome image** : each pixel corresponds to light intensity normally represented in gray scale (graylevel)

**Color image or RGB image**: each pixel contains a vector representing red, green and blue components

**Binary image or black and white image** : Each pixel contains one bit : 
	-  1 represent white 
	- 0 represents black

**Index image** : Each pixel contains index number pointing to a color in a color table

##### Pixel
-  single value on the sampled grid 
-  It does not correspond to a point but rather to an area, the smallest treatable
##### Memory 
-  Pixel number: $\#_{rows} \times \#_{collumns} = M\times N$
-  k: image resolution : $k = log(L)$, (L: intensity levels)
-  N. bit per img: $b = M \times N \times k$

##### Spatial resolution
The smallest distinguishable detail in an image
Expressed as :
	– ppi : Pixel Per Inch
	– dpi : Dot Per Inch
	– $M\times N$ : does not express the relationship between the number of pixels and the detail level of the original scene


Resolution, what's the best?
- it depends on the application:
	- visualization/web: 72 dpi (dot per inch), that is the monitor resolution
	- printing: up to 1200 dpi…
- If I need to pass from low resolution to high resolution? *Interpolation*, even if it is just an approximation…

# Interpolation or Zooming

### Zooming, Nearest-Neighbor interpolation

The value of a pixel in the output image (D) is set equal to the value of the closest pixel in the input image ($S$):
- $(i_{d}, j_{d})$ = position of the pixels in output (integer)
- $(x_{s}, y_{s})$ = position of the pixels in input (real)
$D(i_{d}, j_{d}) = S(round(x_{s}), round(y_{s}))$
- n. of input pixel used for the interpolation: 1
- *Fastest method, but less precise*

**Conventional indexing method**

| $x-1,y-1$ | $x,y-1$ | $x+1,y-1$ |
| --------- | ------- | --------- |
| $x-1,y$   | $x,y$   | $x+1, y$  |
| $x+1,y+1$ | $x,y+1$ | $x+1,y+1$ |

**Neighbors of a Pixel**
Neighborhood relation is used to tell adjacent pixels. It is useful for analyzing regions
*4-neighbors of p* :
$$N_{4}(p) = \{(x-1,y), (x+1,y), (x, y-1), (x,y+1)\}$$
4-neighborhood relation considers only vertical and horizontal neighbors.
>[!NOTE]  $q \in N_{4}(p) \implies p \in N_{4}(q)$

*8-neighbors of p* : $$N_{8}(p) = \{(x-1,y-1), (x, y-1),(x+1, y-1), (x-1, y), (x+1, y), (x-1, y+1), (x, y+1), (x+1, y+1)\}$$
8-neighborhood relation considers *all neighbor pixels*.

*Diagonal neighbors of p* :$$N_{D}(p) = \{(x-1,y-1), ,(x+1, y-1), (x-1, y+1), (x+1, y+1)\}$$Diagonal -neighborhood relation considers only diagonal neighbor pixels.

### Connectivity
Connectivity is adapted from neighborhood relation. Two pixels are connected if they are in the same class (i.e. the same color or the same range of intensity) and they are neighbors of one another.

For p and q from the same class
- 4-connectivity : p and q are 4-connected if $q \in N_{4}(p)$
- 8-connectivity : p and q are 8-connected if $q \in N_{8}(p)$
- mixed-connectivity (m-connectivity): p and q are m-connected if $q \in N_{4}(p)\vee (q\in N_{D}(p) \wedge N_{4}(p) \cap N_{4}(q) = \varnothing)$
##### Adjacency
A pixel p is *adjacent* to pixel q is they are connected. Two image subsets $S_{1}$ and $S_{2}$ are adjacent if some pixel in $S_{1}$ is adjacent to some pixel in $S_{2}$
![[2025-10-08-185655_hyprshot.png]]
We can define type of adjacency: 4-adjacency, 8-adjacency or m-adjacency depending on type of connectivity.

##### Path
A *path* from pixel p at $(x,y)$ to pixel q at $(s,t)$ is a sequence of distinct pixels :
$$(x_{0}, y_{0}),(x_{1}, y_{1}), \dots (x_{n}, y_{n})$$
such that $(x_{0}, y_{0})= (x,y)$ and $(x_{n}, y_{n}) = (s,t)$
and $\forall i \in [|2,n|], (x_{i},y_{i}) \text{is adjacent to } (x_{i-1}, y_{i-1})$![[2025-10-08-190117_hyprshot.png]]
We can define type of path: 4-path, 8-path or m-path depending on type of adjacency
![[2025-10-08-190142_hyprshot.png]]

##### Distance 
For pixel p, q, and z with coordinates $(x,y)$, $(s,t)$ and $(u,v)$, D is a *distance function* or metric if : 
- $D(p,q) \geq 0$ with $D(p,q) = 0 \Leftrightarrow p=q$
- $D(p,q) = D(q,p)$
- $D(p,z) \le D(p,q) + D(q,z)$

**Example** : 
Euclidean distance $$D_{e}(p,q) = \sqrt{ (x-s)^{2} + (y-t)^{2}}$$
$D_{4}$-distance (city-block distance) is defined as $$D_{4}(p,q) = |x-s|+|y-t|$$
$D_{8}$-distance (chessboard distance) is defined as $$D_{8}(p,q) = max(|x-s|,|y-t|)$$
