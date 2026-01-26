**Tags:** #ComputerVision #ColorTheory #ColorSpaces #RGB #HSV #HumanVision
![[3-Color.pdf]]
# What is color?
Color is a psychological property of our visual experiences when we look at objects and lights, *not a physical property *of those objects or lights

**Color is the result of interaction between physical light in the environment and our visual system**
### The physics of Light 
##### EM Spectrum
![[2025-10-08-191433_hyprshot.png]]
Why do we see light at these wavelengths?
Because that’s where the sun radiates electromagnetic energy


Any source of light can be completely described physically by its spectrum: the amount of energy emitted (per time unit) at each wavelength 400 - 700 nm.

##### Interaction of light and surfaces

Observed color is the result of interaction of light source spectrum with surface reflectance

Spectral radiometry :
- All definitions and units are now “per unit wavelength”
- All terms are now “spectral”![[2025-10-08-191733_hyprshot.png]]

### The Eye
![[2025-10-08-191814_hyprshot.png]]

##### The human eye is a camera :
- **Iris** - colored annulus with radial muscles
- **Pupil** - the hole (aperture) whose size is controlled by the iris
- **Lens** - changes shape by using ciliary muscles (to focus on objects at different distances)
- The "Film":
	- **Rods** responsible for intensity, **cones** responsible for color
	- **Fovea** - Small region (1 or 2°) at the center of the visual field containing the highest density of cones (and no rods).
	- Less visual acuity in the periphery—many rods wired to the same neuron

##### Color perception
Rods and cones act as filters on the spectrum
- To get the output of a filter, multiply its response curve by the spectrum, integrate over all wavelengths


### Metamers
In colorimetry, metamerism is a perceived matching of colors with different (nonmatching) spectral power distributions. Colors that match this way are called metamers.

# Computer and us 
Our perception of colors is different from the computer
- We usually exploit an entire overview of the scene
- We approximate a lot
- We do not extract the color from the item associated to it
# White balance
- When looking at a picture on screen or print, we adapt to the illuminant of the room, not to that of the scene in the picture
- When the white balance is not correct, the picture will have an unnatural color “cast”![[2025-10-08-192539_hyprshot.png]]

- Film cameras:
	- Different types of film or different filters for different illumination conditions
- Digital cameras
	- Automatic white balance
	- White balance settings corresponding to several common illuminants
	- Custom white balance using a reference object

It can be trick :
- When there are several types of illuminants in the scene, different reference points will yield different results

Without gray cards: we need to “*guess*” which pixels correspond to white objects :
- Gray world assumption :
	- The image average $\hat{r},\hat{g},\hat{b}$ is gray
	- Use weight $\frac{1}{\hat{r}}, \frac{1}{\hat{g}}, \frac{1}{\hat{b}}$ 
- Brightest pixel assumption :
	- Highlights usually have the color of the light source
	- Use weights inversely proportional to the values of the brightest pixels
- Gamut mapping :
	- **Gamut**: convex hull of all pixel colors in an image
	- Find the transformation that matches the gamut of the image to the gamut of a “typical” image under white light
- Use image statistics, learning techniques


# Uses of color in computer vision
- Color histograms for indexing and retrieval
- Skin detection
- Image segmentation and retrieval
- Building appearance models for tracking
- Judging visual realism