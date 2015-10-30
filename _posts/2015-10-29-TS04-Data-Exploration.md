---
layout: post
title: "Lesson 04 Data Exploration"
date:   2015-10-24
authors: "Marisa Guarinello, Megan Jones, Courtney Soderberg"
dateCreated: 2015-10-22
lastModified: 2015-10-29
tags: [module-1]
description: "This lesson will teach individuals how to plot subsetted timeseries
data (e.g. plot by season) and to plot time series data with NDVI."
code1:
image:
  feature: NEONCarpentryHeader_2.png
  credit: A collaboration between the National Ecological Observatory Network (NEON) and Data Carpentry
  creditlink: http://www.neoninc.org
permalink: m

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


    # output will have width of 80 max
    options(width=80)

##About
This activity will walk you through the fundamental principles of working 
with raster data in R.

**R Skill Level:** Intermediate - you've got the basics of `R` down.


###Goals / Objectives
After completing this activity, you will know:
* How to use facets in ggplot
* How to combine different types of data into one plot.


###hings You'll Need To Complete This Lesson
Please be sure you have the most current version of `R` and preferably
R studio to write your code.

####R Libraries to Install
<li><strong>ggplot:</strong> <code> install.packages("ggplot2")</code></li>


#Graph precip by total PAR across all seasons

    part.prec <- ggplot(harMet.daily,aes(prec, part)) +
               geom_point() +
               ggtitle("Relationship between precipitation and PART") +
               theme(plot.title = element_text(lineheight=.8, face="bold",size = 20)) +
               theme(text = element_text(size=20)) +
               xlab("Total Precipitation") + ylab("Mean Total PAR")

#Use daily data and subset by seasons (retaining PAR, prec, & temp variables)

    #subset by season using month (12-2 is winter, 3-5 spring, 6-8 summer, 9-11 fall)

#Use facets in ggplot to create the same graph for each season and display them in a grid

    #part.prec + facet_grid(. ~ season)
    #part.prec + facet_grid(season ~ .)
    
    #part.prect + facet_wrap(~season, ncol = 2)

#Graph one of those variables and NDVI data together