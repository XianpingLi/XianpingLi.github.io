---
layout: post
title: Parallel computing in R
---
**R** was born to be single threaded, so it will be slow for big data sets or heavy time consuming computations. We can use some third-party packages to parallelize our task on multiple cores or machines. Normally, we define the number of cores, and initialize the workers which computations can be sent to first. Then split the task into pieces and distribute the sub-tasks to workers. Shut down the cluster and do more analysis on the results at last. Here, I use the functions in package *snowfall* which depends on *snow* to give an example.

<!-- more -->

{% highlight r linenos %}
# Assume I want to calculate the mean value of a 
# sample with a fixed sample size from a population.

# A simulated population with 1000 individuals
set.seed(1234)
pop <- rnorm(1000)

mean(pop)
## [1] -0.0265972

# Number of repetitions, sample size and the result  
times <- 50000
size <- 50
result <- c()

# A function for calculating the mean value 
# of a sample with a fixed sample size
sampleMean <- function(.) {
    samples <- sample(pop, size)
    return(mean(samples))
}

# Time cost if we use for loop
system.time(
    for (i in 1:times) {
        result[i] <- sampleMean()
    }
)
##   user  system elapsed 
##   5.63    0.01    5.71 

mean(result)
## [1] -0.02673376

# Compute in parallel
# Load package
library(snowfall)

# Initialisation of cluster usage
sfInit(            # see help file for more information
    parallel=TRUE, # parallel or sequential
    cpus=4         # number of CPUs
    )

# Export the global values to workers, 
# you can use "sfExportAll" to export all.
# See help file for more information
sfExport("pop","size")

# Execute the function in parallel.
# See help file for other functions
system.time (
    parResult <- sfLapply(1:times, sampleMean)
)
##   user  system elapsed 
##   0.02    0.00    0.43

# Stop cluster
sfStop()

mean(unlist(parResult))
## [1] -0.02693764

{% endhighlight %}
Some packages have build-in functions can do parallel computations. For example, *pdredge* (MuMIn) can do model selection in parallel. 