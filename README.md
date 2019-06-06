
<!-- README.md is generated from README.Rmd. Please edit that file -->

# AFremover

Autofluorescence is a long-standing problem that has hindered
fluorescence microscopy image analysis. To address this, we have
developed a method that identifies and removes autofluorescent signals
from multi-channel images post acquisition.

## Installation

``` r
library(devtools)
devtools::install_github("ellispatrick/AFremover")
```

## A quick example

``` r
library(AFremover)
set.seed(51773)
## Read in images.
imageFile1 = system.file("extdata","ImageB.CD3.tif", package = "AFremover")
imageFile2 = system.file("extdata","ImageB.CD11c.tif", package = "AFremover")
im1 <- EBImage::readImage(imageFile1)
im2 <- EBImage::readImage(imageFile2)

## Rescale the images.
im1 = im1/max(im1)
im2 = im2/max(im2)

combined <- EBImage::rgbImage(green=sqrt(im1), red=sqrt(im2))
EBImage::display(combined, all = TRUE, method = 'raster')
```

![](man/figures/README-unnamed-chunk-2-1.png)<!-- -->

``` r


## Create masks using EBImage.

# Find tissue area
tissue1 = im1 > 2*min(im1)
tissue2 = im2 > 2*min(im2)

# Calculate thresholds
imThreshold1 <- mean(im1[tissue1]) + 2*sd(im1[tissue1])
imThreshold2 <- mean(im2[tissue2]) + 2*sd(im2[tissue2])

# Calculate masks.
mask1 <- EBImage::bwlabel(im1 > imThreshold1)
mask2 <- EBImage::bwlabel(im2 > imThreshold2)

## Calculate intersection mask
mask <- intMask(mask1,mask2)


## Calculate textural features.
df <- afMeasure(im1, im2, mask)
#> Warning in cor(im1Pixels, im2Pixels): the standard deviation is zero

#> Warning in cor(im1Pixels, im2Pixels): the standard deviation is zero

#> Warning in cor(im1Pixels, im2Pixels): the standard deviation is zero

#> Warning in cor(im1Pixels, im2Pixels): the standard deviation is zero

#> Warning in cor(im1Pixels, im2Pixels): the standard deviation is zero

## Alternatively
## Correlation only
# afMask <- afIdentify(mask, df, minSize = 100, maxSize = Inf, corr = 0.6)

## Clustering with given k
# afMask <- afIdentify(mask, df, minSize = 100, maxSize = Inf, k = 6)

## Clustering with estimated k.
afMask <- afIdentify(mask, df, minSize = 100, maxSize = Inf, k = 20, kAuto = TRUE)


## Remove autofluorescence from images
im1AFRemoved <- im1
im2AFRemoved <- im2
im1AFRemoved[afMask != 0] <- 0
im2AFRemoved[afMask != 0] <- 0

combinedRemoved <- EBImage::rgbImage(green = sqrt(im1AFRemoved), red = sqrt(im2AFRemoved))
img_comb = EBImage::combine(combined, combinedRemoved)
EBImage::display(img_comb, all = TRUE, method = 'raster',nx = 2)
```

![](man/figures/README-unnamed-chunk-2-2.png)<!-- -->

``` r

##Or
##Exclude AF ROIs

exclude1 = unique(mask1[afMask>0])
mask1Removed = mask1
mask1Removed[mask1Removed%in%exclude1] = 0
exclude2 = unique(mask2[afMask>0])
mask2Removed = mask2
mask2Removed[mask2Removed%in%exclude2] = 0
```
