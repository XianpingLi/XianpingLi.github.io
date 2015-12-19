---
layout: post
title: Spatial Autocorrelation
---
> Everything is related to everything else, but near things are more related than distant things.--Waldo Tobler<sup>\[1\]</sup>

## What is Spatial Autocorrelation (SA)?

As the first law of geography, SA is a measurement of the degree of similarity between spatial objects. A dependency exists between values of a variable in proximal locations. There are three kinds of SA: positive, negative and random (zero) (Fig. 1).

<!-- more -->

- Positive SA is when spatial data values tend to be clustered (similar values aggregate together in space).
- Negative SA is when spatial data values tend to be dispersed (dissimilar values cluster together).
- Random SA with no clear pattern.

<img src="/images/autocorrelation.jpg" alt = "fig1" />
**Figure 1** Type of spatial autocorrelation<sup>[2]</sup>.

Most spatial data are clustered in ecology. For example, altitude values will always be relatively high near the peak of a mountain. Mean temperature will change smoothly from one place to another. Or there is a significant ecological pattern, but it will not be random. Many people treated SA as a useless parameter and excluded it in their studies when they want to using a spatial variable to reveal an ecological phenomenon. But the existence of SA will violate the independent observations assumption which is a precondition for many statistical methods. Positive dependence makes the effective sample size less than the number of observations. In many ecological examples, SA can arise from the focused variable or other factors that affect this variable. Moreover, positive autocorrelation can indicate different patterns at different scales (Fig. 2), that make it more complicated.

<img src = "/images/SA_patterns.jpg" alt ="fig2" />
**Figure 2** Spatial autocorrelation at different scales<sup>[3]</sup>.

## How can we detect SA in R?
We use the shapefile *eire* included in the package *spdep* as an example to test for SA. Let's load the required packages and data firstly.

{% highlight r linenos %}
library(maptools)
library(rgdal)
library(spdep)

eire <- readShapePoly(system.file("etc/shapes/eire.shp", package="spdep")[1],
    ID="names", proj4string=CRS("+proj=utm +zone=30 +units=km"))
{% endhighlight %}

### 1. Build a neighbors list

{% highlight r linenos %}
eire.nb <- poly2nb(eire)

# Plot the spatial polygons and add the neighbors lists (Fig. 3)
plot(eire)
plot(eire.nb, coordinates(eire), add=TRUE, lwd=2, col='blue')
{% endhighlight %}
<img src="/images/NB.png" alt="fig3" />
**Figure 3** Plot of neighbors list.

### 2. Create a spatial weights matrix for the neighbors lists

{% highlight r linenos %}
# Row-standardized weights matrix
eire.nb.w <- nb2listw(eire.nb)
{% endhighlight %}

### 3. Run statistical test to examine SA
We test SA of the variable **A** (percentage of sample with blood group A) in *eire* data sets.

**Geary's *C***

Computation of squared differences of values that are geographic neighbors, 0-1 for positive, 1 (expected value) for random and 1-2 for negative SA.

{% highlight r linenos %}
geary.test(eire$A,listw=eire.nb.w)

##     Geary's C test under randomisation 

## data:  eire$A 
## weights: eire.nb.w  

## Geary C statistic standard deviate = 4.5146, p-value = 3.172e-06
## alternative hypothesis: Expectation greater than statistic
## sample estimates:
## Geary C statistic       Expectation          Variance 
##        0.38011971        1.00000000        0.01885309     
{% endhighlight %}

**Moran's *I***

Computation of cross-products of mean-adjusted values that are geographic neighbors, -1 to nearly 0 for negative, -1/(n-1) for random and 0-1 for positive.

{% highlight r linenos %}
moran.test(eire$A,listw=eire.nb.w)

##     Moran's I test under randomisation

## data:  eire$A  
## weights: eire.nb.w  

## Moran I statistic standard deviate = 4.6851, p-value = 1.399e-06
## alternative hypothesis: greater
## sample estimates:
## Moran I statistic       Expectation          Variance 
##        0.55412382       -0.04000000        0.01608138 
{% endhighlight %}

Moran's *I* and Geary's *C* are inversely related. The results demonstrate that there is positive SA for variable **A**. But we should be cautious with the results because these tests are highly sensitive to the form of neighbors relationships, the choice of spatial weights and other factors. 

#### References:
<div class="references">
[1]: Tobler WR (1970) A Computer Movie Simulating Urban Growth in the Detroit Region. <em>Economic Geography</em>, <strong>46</strong>, 234-240.
<br>
[2]: http://docs.aurin.org.au/portal-help/analysing-your-data/spatial-statistics-tools/introduction-to-spatial-autocorrelation.
<br>
[3]: http://adegenet.r-forge.r-project.org/files/day3.1.2.pdf.
<br>
</div>
