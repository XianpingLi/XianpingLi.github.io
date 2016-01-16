---
layout: post
title: Web Scraping with R
---
## Web scraping
So many data are on the web nowadays, and this situation will go on. There is a technology called web scraping letting us obtain data from websites repeatedly and automatically. Moreover, this procedure can transform the "unstructured" data (actually many of the content in the web page is structured, so that you can scrape it for subsequent analysis) to "structured" data (more cleaned data), see [Wikipedia](https://en.wikipedia.org/wiki/Web_scraping) for more details. There are some examples scraping data with **R** ([this](http://blog.rstudio.org/2014/11/24/rvest-easy-web-scraping-with-r/), [this](http://educate-r.org//2015/12/04/centraliowaruser/) and [this](http://fredheir.github.io/WebScraping/))

<!-- more -->

## Specific task: scrape the information of the packages referring to spatial analysis from CRAN

We will use the powerful package *rvest* created by Hadley Wickham (yes, that prolific one who is also the author of *ggplot2*, *dplyr*, *devtools*...). 

###1. We will get the web sites of the packages listed under the CRAN Task View for "Analysis of Spatial Data" firstly

{% highlight r linenos %}
library(rvest)

# Parse the web page (task view of spatial)
taskView <- read_html("https://cran.r-project.org/web/views/Spatial.html")

# Get the links of the packages listed under the title "CRAN packages"
urls <- taskView %>%
            html_nodes("ul:nth-child(5) a") %>% # I locate the css selector with SelectorGadget (a Chrome extension, Google it)
            html_attr('href')

# Package names
pkName <- taskView %>%
            html_nodes("ul:nth-child(5) a") %>% 
            html_text()  

head(urls)
## [1] "../packages/ade4/index.html"        
## [2] "../packages/adehabitat/index.html"  
## [3] "../packages/adehabitatHR/index.html"
## [4] "../packages/adehabitatHS/index.html"
## [5] "../packages/adehabitatLT/index.html"
## [6] "../packages/adehabitatMA/index.html"

# Substitute the ".." with "https://cran.r-project.org/web" for the real urls
urls <- gsub("\\.\\.","https://cran.r-project.org/web",urls)

{% endhighlight %}

###2. Execute a *for* loop to download the information of each package 

{% highlight r linenos %}
# I select the attributes, "Version", "Published" and "Maintainer"
result <- data.frame(package_name=pkName,
            version=NA,
            published=NA,
            maintainer=NA)

for (i in 1:length(urls)) {
    inf <- read_html(urls[i]) %>%
                html_nodes("table") %>%  
                .[[1]] %>% # the first table
                html_table()

    rownames(inf) <- inf[,1]
    inf <- as.vector(inf[c("Version:","Published:","Maintainer:"),2]) # the three rows I need 
    inf[3] <- gsub("\\s+<.*>","",inf[3]) # remove the email address

    result[i,c("version","published","maintainer")] <- inf
}

# Done. You can do more analyses on the data frame
head(result)
##  package_name version  published           maintainer
## 1         ade4   1.7-3 2015-11-22  Aurélie Siberchicot
## 2   adehabitat  1.8.18 2015-07-22      Clement Calenge
## 3 adehabitatHR  0.4.14 2015-07-22      Clement Calenge
## 4 adehabitatHS  0.3.12 2015-07-22      Clement Calenge
## 5 adehabitatLT  0.3.20 2015-07-22      Clement Calenge
## 6 adehabitatMA  0.3.10 2015-07-22      Clement Calenge

{% endhighlight %}

## Cautions
* Refresh you scripts when the websites were updated.
* Keep in mind that many of the content on the web are **copyrighted**.
* Please do not put too much pressure on the web servers.


