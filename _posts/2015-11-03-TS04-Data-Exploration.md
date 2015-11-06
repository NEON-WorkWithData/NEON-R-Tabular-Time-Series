---
layout: post
title: "Lesson 04: Data Exploration"
date:   2015-10-21
authors: "Megan A. Jones, Marisa Guarinello, Courtney Soderberg"
dateCreated: 2015-10-22
lastModified: 2015-10-29
tags: [module-1]
description: "This lesson will teach individuals how to plot subsetted timeseries data (e.g. plot by season) and to plot time series data with NDVI."
code1:
image:
  feature: NEONCarpentryHeader_2.png
  credit: A collaboration between the National Ecological Observatory Network (NEON) and Data Carpentry
  creditlink: http://www.neoninc.org
comments: false
---

<section id="table-of-contents" class="toc">
  <header>
    <h3>Contents</h3>
  </header>
<div id="drawer" markdown="1">
*  Auto generated table of contents
{:toc}
</div>
</section><!-- /#table-of-contents -->



##About
This lesson will teach individuals how to plot subsetted timeseries data (e.g. plot by season) and to plot time series data with NDVI.

<div id="objectives" markdown="1">

**R Skill Level:** Intermediate - you've got the basics of `R` down.

###Goals / Objectives
After completing this activity, you will know:
 * How to use facets in ggplot,
 * How to combine different types of data into one plot.


###Things You'll Need To Complete This Lesson
Please be sure you have the most current version of `R` and preferably
R studio to write your code.

####R Libraries to Install
<li><strong>ggplot</strong> <code> install.packages("ggplot2")</code></li>
<li><strong>scales:</strong> <code> install.packages("scales")</code></li>
<li><strong>gridExtra:</strong> <code> install.packages("gridExtra")</code></li>

####Data to Download
Make sure you have downloaded the AtmosData folder from
http://figshare.com/articles/NEON_Spatio_Temporal_Teaching_Dataset/1580068

####Recommended Pre-Lesson Reading
Lessons 00-03 in this Time Series learning module
</div>

#Lesson 04: Further Exploration of the Data
Up until this point in the lesson we have been looking at how a single variable 
changes across time. In the interest of studying phenology it is likely that 
some combination of precipitation, temperature, and solar radiation (PAR) plays
a role in the annual greening and browning of plants.  

Let's look at interactions between several of the variables to each other across
the seasons before adding the NDVI data into the mix.  Using the NDVI data we
can finally directly compare the observed plant phenology with patterns we've 
already been looking at.  

##Graph precip by total PAR across all seasons
PAR a measure of solar radiation is less on cloudy days and precipitation is
also more likely when clouds are present.  We will use `ggplot` to graph PAR
and precipitation from the daily Harvard Meterological data.  We can simply do
this using the code we previously learned and substituatuing precipitation
(prec) in for time on the x axis.  

    #PAR v precip 
    par.precip <- ggplot(harMet.daily,aes(prec, part)) +
               geom_point(na.rm=TRUE) +    #removing the NA values
               ggtitle("Daily Precipitation and PAR at Harvard Forest") +
               theme(plot.title = element_text(lineheight=.8, face="bold",size = 20)) +
               theme(text = element_text(size=20)) +
               xlab("Total Precipitation (mm)") + ylab("Mean Total PAR")
    par.precip

![ ]({{ site.baseurl }}/images/rfigs/TS04-Data-Exploration/PAR-v-precip-1.png) 

While there is a lot of noise in the data, there does seem to be a trend of 
lower PAR when precipitation is high. Yet in this data we don't tease apart any 
of annual patterns as date isn't any part of this figure.  

##Use daily data and subset by seasons (retaining PAR, precip, & temp variables)
One way to add a time component to this figure is to subset the data by season 
and tehn to plot a facetted graph showing the data by season.  

The first step to doing this is to subset by season.  First, we have to switch 
away from coding and data analysis and think about ecology.  What is the best
way to break a year apart into seasons?  

The divisions will change depending on on the geography and climate where the
data were collected.  We are using data from Harvard Forest in Massachusettes in
in the northeastern United States.  Given the strong seasonal affects in this 
area dividing the year into 4 seasons is ecologically relevant.  Based on
general knowledge of the area it is also plausible to group the months as 

 * Winter: December - February
 * Spring: March - May 
 * Summer: June - August
 * Fall: September - November 
 
In order to subset the data by season we will again use the `dplyr` package.


    #subset by season using month (12-2 is winter, 3-5 spring, 6-8 summer, 9-11 fall)
    
    #Inelegant and not working
    winter<-harMet.daily%>%      
    mutate(month=format(date, "%m")) %>%
      subset(month == 12 )
    
    summer<-harMet.daily%>%      
    mutate(month=format(date, "%m")) %>%
      subset(month==c(06,07,08))

    ## Warning in month == c(6, 7, 8): longer object length is not a multiple of
    ## shorter object length

##Use facets in ggplot to create the same graph for each season and
#display them in a grid

    #part.prec + facet_grid(. ~ season)
    #part.prec + facet_grid(season ~ .)
    
    #part.prect + facet_wrap(~season, ncol = 2)

#Graph one of those variables and NDVI data together


#Polishing your ggPlot graphs
Throughout this module we have been creating plots using ggplot.  We've covered
the basics of adding axis labels and titles but using ggplot you can do so much 
more.  If you are interested in making your graphs look better consider reading

1) ggplot2 Cheatsheet from Zev Ross:  http://neondataskills.org/cheatsheets/R-GGPLOT2/
2) ggplot 2 documentation index: http://docs.ggplot2.org/current/index.html#