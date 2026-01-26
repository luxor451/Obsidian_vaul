**Tags:** #ComputerVision #Histograms #ImageEnhancement #HistogramEqualization #Contrast
# Definition
**The (intensity or brightness) histogram shows how many times a particular grey level (intensity) appears in an image**

An image has low contrast when the complete range of possible values is not used. Inspection of the histogram shows this lack of contrast
![[2025-10-08-193602_hyprshot.png]]

# Colors
### Grayscale
RGB color can be converted to a gray scale value by $$Y = 0.299R+0.587G+0.114B$$
>[!NOTE] Y: the grayscale component in the YIQ color space used in NTSC television.

The weights reflect the eye's brightness sensitivity to the color primaries

### Histogram of color images
- individual histograms of red, green and blue![[2025-10-08-193836_hyprshot.png]]
or
- a 3-D histogram can be produced, with the three axes representing the red, blue and green channels, and brightness at each point representing the pixel count

# Transformation 
- Point operation $T(r_{k}) = s_{k}$![[2025-10-08-193940_hyprshot.png]]

Properties of $T$ : keeps the original range of grey values monoton increasing

### Histogram equalization (HE)
Transforms the intensity values so that the histogram of the output image approximately matches the flat (uniform) histogram. As for the discrete case the following formula applies: 
$$s_{k} = T(r_{k}) = \sum ^{k}_{j=0} \frac{n_{j}}{n} \cdot (L-1)$$
with $k = 0,1,2,\dots,L-1$
L: number of grey levels in image (e.g., 255)
$n_j$: number of times j-th grey level appears in image
n: total number of pixels in the image
![[2025-10-08-194303_hyprshot.png]]
### Histogram projection (HP)
Assigns equal display space to every occupied raw signal level, regardless of how many pixels are at that same level. In effect, the raw signal histogram is "projected" into a similar-looking display histogram$$s_{k} = 255 \cdot B(k)$$
Occupied (used) grey level: there is at least one pixel with that grey level
$B(k)$: the fraction of occupied grey levels at or below grey level k
B(k) rises from 0 to 1 in discrete uniform steps of $\frac{1}{n}$, where n is the total number of occupied levels
![[2025-10-08-194635_hyprshot.png]]
### Plateau equalization
By clipping the histogram count at a saturation or plateau value, one can produce display allocations intermediate in character between those of HP and HE.$$s_{k} = 255 \cdot P(k)$$
The PE algorithm computes the distribution not for the full image histogram but for the histogram clipped at a plateau (or saturation) value in the count. When that plateau value is set at 1, we generate $B(k)$ and so perform HP; When it is set above the histogram peak, we generate $F(k)$ and so perform HE. At intermediate values, we generate an intermediate distribution which we denote by $P(k)$.


![[2025-10-08-194616_hyprshot.png]]
### Histogram specification (HS)

An image's histogram is transformed according to a desired function Transforming the intensity values so that the histogram of the output image approximately matches a specified histogram.![[2025-10-08-194918_hyprshot.png]]

### Contrast stretching (CS)
By stretching the histogram we attempt to use the available full grey level range. The appropriate CS transformation : $$s_{k} = 255 \cdot \frac{r_{k}-min}{max-min}$$
It can also map an intensity range to another intensity range.![[2025-10-08-195053_hyprshot.png]]

# Application
- CT lung studies
- Thresholding:
	- Converting a greyscale image to a binary one
- Normalization :
	- When one wishes to compare two or more images on a specific basis, such as texture, it is common to first normalize their histograms to a "standard" histogram. This can be especially useful when the images have been acquired under different circumstances. Such a normalization is, for example, HE.
	- Histogram matching takes into account the shape of the histogram of the original image and the one being matched.
- Normalization of MRI images
	- MRI intensities do not have a fixed meaning, not even within the same protocol for the same body region obtained on the same scanner for the same patient.
- Presentation of high dynamic images (IR, CT)
- The HP algorithm is widely used by infrared (IR) camera manufacturers as a real-time automated image display.
- The PE algorithm is used in the B-52 IR navigation and targeting sensor