% Image analysis in R
% Tommaso Leonardi
% tl344@ebi.ac.uk





# A couple of covenient packages for image manipulation in R
1. EBImage

```r
source("http://bioconductor.org/biocLite.R")
biocLite("EBImage")
```




# The simplest image
An image in R is just a matrix where 0 is black and 1 is white

```r
simple_image <- matrix(seq(0, 1, by = 0.2), nrow = 6, ncol = 6)
simple_image
```

```
##      [,1] [,2] [,3] [,4] [,5] [,6]
## [1,]  0.0  0.0  0.0  0.0  0.0  0.0
## [2,]  0.2  0.2  0.2  0.2  0.2  0.2
## [3,]  0.4  0.4  0.4  0.4  0.4  0.4
## [4,]  0.6  0.6  0.6  0.6  0.6  0.6
## [5,]  0.8  0.8  0.8  0.8  0.8  0.8
## [6,]  1.0  1.0  1.0  1.0  1.0  1.0
```


The _display_ function in EBImage makes it easy to visualise a matrix

```r
display(simple_image, method = "raster")
```

![](knit_figure/figunnamed-chunk-3.png) 


# Loading external images
Reading an image from a file is as easy as:

```r
queen <- readImage("HRH_ERII.tif")
queen <- resize(queen, w = 200, h = 260)
```



```r
queen
```

```
## Image
##   colormode: Grayscale 
##   storage.mode: double 
##   dim: 200 260 
##   nb.total.frames: 1 
##   nb.render.frames: 1 
## 
## imageData(object)[1:5,1:6]:
##        [,1]   [,2]   [,3]   [,4]   [,5]   [,6]
## [1,] 0.6000 0.5681 0.5703 0.5708 0.5670 0.5380
## [2,] 0.5922 0.5756 0.5647 0.5538 0.5838 0.5647
## [3,] 0.6039 0.5877 0.5641 0.5490 0.5681 0.5854
## [4,] 0.5804 0.5807 0.5837 0.5748 0.5619 0.5554
## [5,] 0.5765 0.5958 0.5843 0.5695 0.5776 0.5625
```

```r
writeImage(queen, "img/queen_1.jpg", quality = 85)
```

![](img/queen_1.jpg)



# Modifying the brightness

Adding a constant to every pixel increses the brightness

```r
newQueen <- queen + 0.2
plot(as.numeric(queen), as.numeric(newQueen), pch = 19, ylim = c(0, 1), xlim = c(0, 1))
```

![](knit_figure/figunnamed-chunk-6.png) 

```r
writeImage(newQueen, "img/brightQueen.jpg", quality = 85)
```

![Original](img/queen_1.jpg) ![Modified](img/brightQueen.jpg)


# Modifying the contrast

Multiplying a constant to every pixel increses the constrast

```r
newQueen <- queen * 2
plot(as.numeric(queen), as.numeric(newQueen), pch = 19, ylim = c(0, 1), xlim = c(0, 1))
```

![](knit_figure/figunnamed-chunk-7.png) 

```r
writeImage(newQueen, "img/contrastQueen.jpg", quality = 85)
```

![Original](img/queen_1.jpg) ![Modified](img/contrastQueen.jpg)



# Modifying the gamma

Exponentiating every pixel increses the gamma

```r
newQueen <- queen^2
plot(as.numeric(queen), as.numeric(newQueen), pch = 19, ylim = c(0, 1), xlim = c(0, 1))
```

![](knit_figure/figunnamed-chunk-8.png) 

```r
writeImage(newQueen, "img/gammaQueen.jpg", quality = 85)
```

![Original](img/queen_1.jpg) ![Modified](img/gammaQueen.jpg)


# Other operations on the matrix

## Transposition

```r
newQueen <- t(queen)
writeImage(newQueen, "img/transQueen.jpg", quality = 85)
```

![](img/transQueen.jpg)

## Thresholding

```r
newQueen <- queen > 0.5
writeImage(newQueen, "img/thrQueen.jpg", quality = 85)
```

![](img/thrQueen.jpg)


# Filtering
Filtering allows to apply various effects to an image. In its simplest form, you center a filter matrix on each pixel of the image and you reset the value of the pixel to the sum of pixel-wise products between the image and the filter matrix.
This is a filter matrix:

```r
filt <- matrix(1, nc = 5, nr = 5)
filt <- filt/sum(filt)
filt
```

```
##      [,1] [,2] [,3] [,4] [,5]
## [1,] 0.04 0.04 0.04 0.04 0.04
## [2,] 0.04 0.04 0.04 0.04 0.04
## [3,] 0.04 0.04 0.04 0.04 0.04
## [4,] 0.04 0.04 0.04 0.04 0.04
## [5,] 0.04 0.04 0.04 0.04 0.04
```

Now imagine to position the central pixel of this filter on one pixel of the image. Then you multiply one by one the pixels of the filter with the pixels of the image covered by the filter. The new value for the central pixel will be the sum of all these products. You then repeat the same procedure after positioning the filter on every pixel in the image.
We divide the filter matrix by its sum so that the sum of the matrix is always 1 (if we didn't we would increase the overall intensity if the image).

The function _filter2_ of EBImage does this procedure for us:

```r
newQueen <- filter2(queen, filt)
writeImage(newQueen, "img/lofiltQueen.jpg", quality = 85)
```

![Original](img/queen_1.jpg) ![](img/lofiltQueen.jpg)



# Sharpening
Sharpening an image is the result of applying yet another filter on it.

This is a filter matrix that would sharpen the image:

```r
filt <- matrix(-1, nc = 3, nr = 3)
filt[2, 2] <- 9
filt
```

```
##      [,1] [,2] [,3]
## [1,]   -1   -1   -1
## [2,]   -1    9   -1
## [3,]   -1   -1   -1
```

The pixel intensity has a strong positive weight, while its neighbours have a small negative one. The sum of all pixels is 1, so that the overall intensity of the image doesn't change.


```r
newQueen <- filter2(queen, filt)
writeImage(newQueen, "img/sharpQueen.jpg", quality = 85)
```

![Original](img/queen_1.jpg) ![](img/sharpQueen.jpg)



# Can we use R to make something useful?
Given an image of this type:

```r
tif_files <- as.Image(readTIFF("astrocytes.tiff"))
colorMode(tif_files) <- "color"
nuc <- tif_files[, , 1]
cel <- tif_files[, , 3]
img <- rgbImage(green = cel, blue = nuc)
writeImage(resize(img, w = 347, h = 464), "img/img.jpg", quality = 86)
```

![](img/img.jpg)

Can you tell me how many cells there are, what is the cell size distribution and what is the bistribution of signal intensities cell-by-cell?







