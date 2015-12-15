---
layout: post
title: "Lesson 06: Plot using Facets and Plot Time Series with NDVI data"
date:   2015-10-19
authors: [Megan A. Jones, Marisa Guarinello, Courtney Soderberg]
contributors: [Leah Wasser]
dateCreated: 2015-10-22
lastModified: 2015-12-14
category:  
tags: [module-1]
mainTag: 
description: "This lesson teaches how to plot subsetted time series data using
the `facets()` function (e.g., plot by season) and to plot time series data with
NDVI."
code1: TS06-Facets-And-NDVI-Plotting-In-R.R
image:
  feature: NEONCarpentryHeader_2.png
  credit: A collaboration between the National Ecological Observatory Network (NEON) and Data Carpentry
  creditlink: http://www.neoninc.org
permalink: /R/Time-Series-Plot-Facets-NDVI
comments: false
---

{% include _toc.html %}

##About
This lesson teaches how to plot subsetted time series data using
the `facets()` function (e.g., plot by season) and to plot time series data with
NDVI.

**R Skill Level:** Intermediate - you've got the basics of `R` down.

<div id="objectives" markdown="1">

###Goals / Objectives
After completing this activity, you will:

 * Know how to use `facets()` in the ggplot2 package.
 * Be able to combine different types of data into one plot.

###Challenge Code
Throughout the lesson we have Challenges that reinforce learned skills. Possible
solutions to the challenges are not posted on this page, however, the code for
each challenge is in the `R` code that can be downloaded for this lesson (see
footer on this page).

###Things You'll Need To Complete This Lesson
Please be sure you have the most current version of `R` and, preferably,
RStudio to write your code.

####R Libraries to Install
<li><strong>ggplot:</strong> <code> install.packages("ggplot2")</code></li>
<li><strong>scales:</strong> <code> install.packages("scales")</code></li>
<li><strong>gridExtra:</strong> <code> install.packages("gridExtra")</code></li>
<li><strong>grid:</strong> <code> install.packages("grid")</code></li>
<li><strong>dplyr:</strong> <code> install.packages("dplyr")</code></li>
<li><strong>reshape2:</strong> <code> install.packages("reshape2")</code></li>

####Data to Download
<a href="https://ndownloader.figshare.com/files/3579861" class="btn btn-success"> Download Atmospheric Data</a>

The data used in this lesson were collected at Harvard Forest which is
an National Ecological Observatory Network  
<a href="http://www.neoninc.org/science-design/field-sites/harvard-forest" target="_blank"> field site</a>. 
These data are proxy data for what will be available for 30 years
on the [NEON data portal](http://data.neoninc.org/ "NEON data")
for both Harvard Forest and other field sites located across the United States.

####Setting the Working Directory
The code in this lesson assumes that you have set your working directory to the
location of the unzipped file of data downloaded above.  If you would like a
refresher on setting the working directory, please view the [Setting A Working Directory In R]({{site.baseurl}}/R/Set-Working-Directory/ "R Working Directory Lesson") 
lesson prior to beginning this lesson.

####Skills Needed
This lessons assumes familiarity with both the `dplyr` package and `ggplot()` in
the `ggplot2` package.  If you are not comfortable with either of these we
recommend starting with the [Subset & Manipulate Time Series Data with `dplyr` lesson]({{site.baseurl}}/R/Time-Series-Subset-dplyr/ "Learn dplyr") 
and the [Plotting Time Series with ggplot in R lesson]({{site.baseurl}}/R/Time-Series-Plot-ggplot/ "Learn ggplot")  
respectively, to gain familiarity.

###Time Series Lesson Series 
This lesson is a part of a series of lessons on tabular time series data in `R`:

* [ Brief Time Series in R - Simple Plots with qplot & as.Date Conversion]({{ site.baseurl}}/R/Brief-Tabular-Time-Series-qplot/)
* [Understanding Time Series Metadata]({{ site.baseurl}}/R/Time-Series-Metadata/)
* [Convert Date & Time Data from Character Class to Date-Time Class (POSIX) in R]({{ site.baseurl}}R/Time-Series-Convert-Date-Time-Class-POSIX/)
* [Refining Time Series Data - NoData Values and Subsetting Data by Date in R]({{ site.baseurl}}/R/Subset-Data-and-No-Data-Values/)
* [Subset and Manipulate Time Series Data with dplyr]({{ site.baseurl}}/R/Time-Series-Subset-dplyr/)
* [Plotting Time Series with ggplot in R]({{ site.baseurl}}/R/Time-Series-Plot-ggplot/)
* [Plot uisng Facets and Plot Time Sereis with NDVI data]({{ site.baseurl}}/R/Time-Series-Plot-Facets-NDVI/)
* [Converting to Julian Day]({{ site.baseurl}}/R/julian-day-conversion/)

###Sources of Additional Information

</div>

#The Data Approach
What we want to do with ecological time series data often extends beyond
plotting a single variable as it changes across time. For example, when studying
phenology it is likely that a combination of precipitation, temperature, and
solar radiation (PAR) plays a role in the annual greening and browning of 
plants.  

In this lesson we will learn how to plot several multiple variables variables
together across the seasons, prior to adding in NDVI data.  Using the NDVI data 
we can finally directly compare the observed plant phenology with patterns we've
already been exploring.  

###Load the Data
We will need the daily micro-meteorology data for 2009-2011 from the Harvard
Forest. If you do not have these data sets as `R` objects, please load them from
the .csv files in the downloaded data and convert date-time variables to 
POSIXct date-time class. 


    #Remember it is good coding technique to add additional libraries to the top of
      #your script 
    library(lubridate) #for working with dates
    library(ggplot2)  #for creating graphs
    library(scales)   #to access breaks/formatting functions
    library(gridExtra) #for arranging plots
    library(grid)   #for arrangeing plots
    library(dplyr)  #for subsetting by season
    
    #set working directory to ensure R can find the file we wish to import
    #setwd("working-dir-path-here")
    
    #daily HARV met data, 2009-2011
    harMetDaily.09.11 <- read.csv(file="AtmosData/HARV/Met_HARV_Daily_2009_2011.csv",
                         stringsAsFactors = FALSE)
    
    #covert date to POSIXct date-time class
    harMetDaily.09.11$date <- as.POSIXct(harMetDaily.09.11$date)
    
    #monthly HARV temperature data, 2009-2011
    harTemp.monthly.09.11<-read.csv(file="AtmosData/HARV/Temp_HARV_Monthly_09_11.csv",
                                    stringsAsFactors=FALSE)
    
    #convert datetime from chr to datetime class & rename date for clarification
    harTemp.monthly.09.11$date <- as.POSIXct(harTemp.monthly.09.11$datetime)

##Graph Two Variables on One Plot
Is there a relationship between PAR, a measure of solar radiation, and
precipiation.  It makes sense there would be as solar radiation is lower on 
cloudy days and precipitation is also more likely when clouds are present. 

We will use `ggplot()` to graph PAR and precipitation from the daily Harvard
Meteorological data.  We can simply do this by in putting both variables into
the aesthetics (`aes`) argument of `ggplot()`.  Unless we specify, the first 
variable will be the x-axis and the second the y-axis.  


    #PAR v precip 
    par.precip <- ggplot(harMetDaily.09.11,aes(prec, part)) +
               geom_point(na.rm=TRUE) +    #removing the NA values
               ggtitle("Daily Precipitation and PAR at Harvard Forest") +
               theme(plot.title = element_text(lineheight=.8, face="bold",size = 20)) +
               theme(text = element_text(size=20)) +
               xlab("Total Precipitation (mm)") + ylab("Mean Total PAR")
    par.precip

![ ]({{ site.baseurl }}/images/rfigs/TS06-Facets-And-NDVI-Plotting-In-R/PAR-v-precip-1.png) 

While there is a lot of noise in the data, there does seem to be a trend of 
lower PAR when precipitation is high. Yet in this data we don't tease apart any 
of annual patterns as date isn't part of this figure.  


##Subset by Season
One way to add a time component to this figure is to subset the data by season 
and then to plot a facetted graph showing the data by season.  The initial step
to do this is to subset by season.  

First, we have to switch 
away from coding and data analysis to think about ecology.  What is the best
way to break a year apart into seasons?  

The divisions will change depending on the geographic location and climate where
the data were collected.  We are using data from Harvard Forest in Massachusetts
in the northeastern United States.  Given the strong seasonal affects in this 
area dividing the year into 4 seasons is ecologically relevant.  Based on
general knowledge of the area it is also plausible to group the months as 

 * Winter: December - February
 * Spring: March - May 
 * Summer: June - August
 * Fall: September - November 
 
In order to subset the data by season we will again use the `dplyr` package.  


    #create a column of only the month
    harMetDaily.09.11 <- harMetDaily.09.11  %>% 
      mutate(month=format(date,"%m"))
    
    #check structure of this variable
    str(harMetDaily.09.11$month)

    ##  chr [1:1095] "01" "01" "01" "01" "01" "01" "01" "01" ...

Notice that the new column `month` is recognized as character class.  

We can now use `mutate()` and an `ifelse` statement to create a new variable
called `season` by grouping three months together. Because month is a character
variable we need to put the month number in quotes. 

Within `dplyr` `%in%` is short-hand for "or"; the 3rd line of code essentially
says "If the month variable is equal to 12 or  01 or  02, set the season
variable to winter".  The "Error" at the end is what will be printed in the 
column if the month does not equal any of the month dates specified.  


    harMetDaily.09.11 <- harMetDaily.09.11 %>% 
      mutate(season = 
               ifelse(month %in% c("12", "01", "02"), "winter",
               ifelse(month %in% c("03", "04", "05"), "spring",
               ifelse(month %in% c("06", "07", "08"), "summer",
               ifelse(month %in% c("09", "10", "11"), "fall", "Error")))))
    
    #check to see if this worked
    head(harMetDaily.09.11$month)

    ## [1] "01" "01" "01" "01" "01" "01"

    head(harMetDaily.09.11$season)

    ## [1] "winter" "winter" "winter" "winter" "winter" "winter"

    tail(harMetDaily.09.11$month)

    ## [1] "12" "12" "12" "12" "12" "12"

    tail(harMetDaily.09.11$season)

    ## [1] "winter" "winter" "winter" "winter" "winter" "winter"

##Use Facets in ggplot
Facets allow us to plot subsets of data together.  Using `facet_grid()` we can
create the same plot of PAR and precipiation for each season and display them in
a grid.


    #run this code to plot the same plot as before but with one plot per season
    par.precip + facet_grid(. ~ season)

    ## Error in layout_base(data, cols, drop = drop): At least one layer must contain all variables used for facetting

It didn't work!  Why? 

We added the season variable to `harMetDaily.09.11` after we created the
original `par.precip` plot. We need to re-run the `par.precip` code again so 
that `season` is now included in the data (go back and re-run that code). 



Now that we've done that we can again try the facet code. 


    par.precip + facet_grid(. ~ season)

![ ]({{ site.baseurl }}/images/rfigs/TS06-Facets-And-NDVI-Plotting-In-R/plot-by-season2-1.png) 

    # for a landscape orientation of the plots we change the order of arguments in
        #facet_grid():
    par.precip + facet_grid(season ~ .)

![ ]({{ site.baseurl }}/images/rfigs/TS06-Facets-And-NDVI-Plotting-In-R/plot-by-season2-2.png) 

    #and another arrangement of plots:
    par.precip + facet_wrap(~season, ncol = 2)

![ ]({{ site.baseurl }}/images/rfigs/TS06-Facets-And-NDVI-Plotting-In-R/plot-by-season2-3.png) 

How does our ability to see patterns in the data vary depending on which
arrangement the graphs are in? 

##Defining the Order of Facetted Plots
Notice how our facetted plots are currently show up as "fall", "spring",
"summer", "winter".  As no level is assigned the data are class character, `R`
simply arranges them in alphabetical order.  However, "fall", "spring", 
"summer", "winter" do have a specified order to them.  It doesn't mater which
season starts the cycle but they should be in order to ease our understanding
of annual cycles. 

In order to create a logical order we need to assign levels to the seasons. We
can do this by assigning the seasons as a factor with a defined order (or 
level).  


    harMetDaily.09.11$season<- factor(harMetDaily.09.11$season, level=c("spring",
                                                        "summer","fall","winter")) 
    
    #check to make sure it worked
    str(harMetDaily.09.11$season)

    ##  Factor w/ 4 levels "spring","summer",..: 4 4 4 4 4 4 4 4 4 4 ...

    #rerun original par.precip code to incorporate the levels. 
    par.precip <- ggplot(harMetDaily.09.11,aes(prec, part)) +
             geom_point(na.rm=TRUE) +    #removing the NA values
            ggtitle("Daily Precipitation and PAR at Harvard Forest") +
             theme(plot.title = element_text(lineheight=.8, face="bold",size = 20)) +
             theme(text = element_text(size=20)) +
             xlab("Total Precipitation (mm)") + ylab("Mean Total PAR")
               
    #new facetted plots
    par.precip + facet_grid(. ~ season)

![ ]({{ site.baseurl }}/images/rfigs/TS06-Facets-And-NDVI-Plotting-In-R/assigning-level-to-season-1.png) 

We now how a plot with the seasons in a logical order. 

#Challenge: Compare Precipitation and Air Temperature Across Seasons
1) Using the same data create a faceted plot showing the relationship between 
precipitation and air temperature across the seasons.  Create the figure with
"winter" as the first facetted plot.   
2) Compare how different orientations of the plots highlight different patterns.

![ ]({{ site.baseurl }}/images/rfigs/TS06-Facets-And-NDVI-Plotting-In-R/challenge-code-prec.airtemp-1.png) ![ ]({{ site.baseurl }}/images/rfigs/TS06-Facets-And-NDVI-Plotting-In-R/challenge-code-prec.airtemp-2.png) 

#Plot NDVI & PAR using Daily Data

##NDVI Data
Normalized Difference Vegetation Index (NDVI) is an indicator of how green vegetation is.  NDVI is derrived from remote sensing data based on a ratio the reflectance of visible red spectra and near-infrared specra.  The NDVI values vary from -1.0 to 1.0.  

The imagery data used to create this NDVI data were collected over
the National Ecological Observatory Network's
<a href="http://www.neoninc.org/science-design/field-sites/harvard-forest" target="_blank" >Harvard Forest</a>
field site.  
The imagery was created by the U.S. Geological Survey (USGS) using a 
<a href="http://eros.usgs.gov/#/Find_Data/Products_and_Data_Available/MSS" target="_blank" >  multispectral scanner</a>
on a <a href="http://landsat.usgs.gov" target="_blank" > Landsat Satellite </a>.
The data files are Geographic Tagged Image-File Format (GeoTIFF).  
A tutorial,
[Extract NDVI Summary Values from a Raster Time Series]({{ site.baseurl}}/R/Extract-NDVI-From-Rasters-In-R/), 
explains how to create this NDVI file based on Raster data. 

###Read In NDVI Data
We need to read in the 2011 NDVI data for the Harvard Forest. A .csv file is
within `NDVI_ForAtmosData` subdirectory in the `AtmosData` directory.

    #first read in the NDVI CSV data
    NDVI.2011 <- read.csv(file="AtmosData/NDVI_ForAtmosData/meanNDVI_HARV_2011.csv"
                          , stringsAsFactors = FALSE)
    #check out the data
    str(NDVI.2011)

    ## 'data.frame':	11 obs. of  6 variables:
    ##  $ X        : int  1 2 3 4 5 6 7 8 9 10 ...
    ##  $ meanNDVI : num  0.365 0.243 0.251 0.599 0.879 ...
    ##  $ site     : chr  "HARV" "HARV" "HARV" "HARV" ...
    ##  $ year     : int  2011 2011 2011 2011 2011 2011 2011 2011 2011 2011 ...
    ##  $ julianDay: int  5 37 85 133 181 197 213 229 245 261 ...
    ##  $ Date     : chr  "2011-01-05" "2011-02-06" "2011-03-26" "2011-05-13" ...

    head(NDVI.2011)

    ##   X meanNDVI site year julianDay       Date
    ## 1 1 0.365150 HARV 2011         5 2011-01-05
    ## 2 2 0.242645 HARV 2011        37 2011-02-06
    ## 3 3 0.251390 HARV 2011        85 2011-03-26
    ## 4 4 0.599300 HARV 2011       133 2011-05-13
    ## 5 5 0.878725 HARV 2011       181 2011-06-30
    ## 6 6 0.893250 HARV 2011       197 2011-07-16

#Challenge: Convert Date in Character Class to a Date-Time Class
The Date data is currently in the character class convert the data to a date
class. 



##Subset Only the 2011 Micrometeology Data
First lets get just 2011 from the `harmet.Daily` data since that is the only
year for which we have NDVI data.


    #Use dplyr to subset only 2011 data
    harMet.daily2011 <- harMetDaily.09.11 %>% 
      mutate(year = year(date)) %>%   #need to create a year only column first
      filter(year == "2011")
    
    #convert data from POSIX class to Date class; both "date" vars. now Date class
    harMet.daily2011$date<-as.Date(harMet.daily2011$date)

In this data set we have the following variables:  

* 'X': an integer representing each row
* meanNDVI: the daily total NDVI for that area.  ("Mean" comes from the fact that the value is a mean of all pixels in the original raster).
* site: all NDVI values are from the Harvard Forest
* year: all values are from 2011
* julianDay: the numeric day of the year
* Date: a date in format "YYYY-MM-DD".

## Two y-axes or Side-by-Side Plots?
When we have different types of data like NDVI (scale: 0-1 index units) and PAR
(scale: 0-65.8 mole per meter squared) that we want to plot over time, we cannot
simply plot them on the same plot as they have different y-axes.  

One option, would be to plot both data types in the same plot space but each
having it's own axis.  However, there is a line of graphical representation 
thought that this is not a good practice.  The creator of `ggplot2` ascribes to
this dislike of different y-axes and so neither `qplot` nor `ggplot` have this
functionality.  

Instead, plots of different types of data can be plotted next to each other to 
allow for comparison.  Depending on how the plots are being viewed they can have
a verticle or horizontal arrangement.  

Following this second option, we will create a plot for each variable using the
same time variable (Julian day) as our x-axis.

Then we will plot the two plots in the same viewer so we can more easily compare
them.  


    #create plot of julian day vs. PAR
    plot.par.2011 <- ggplot(harMet.daily2011, aes(date, part))+
      geom_point(na.rm=TRUE) +
      ggtitle("Daily PAR at Harvard Forest, 2011")+
      theme(legend.position = "none",
            plot.title = element_text(lineheight=.8, face="bold",size = 20),
            text = element_text(size=20))
    
    plot.NDVI.2011 <- ggplot(NDVI.2011, aes(Date, meanNDVI))+
      geom_point(colour = "forestgreen", size = 4) +
      ggtitle("Daily NDVI at Harvard Forest, 2011")+
      theme(legend.position = "none",
            plot.title = element_text(lineheight=.8, face="bold",size = 20),
            text = element_text(size=20))
     
    #display the plots together
    grid.arrange(plot.par.2011, plot.NDVI.2011) 

![ ]({{ site.baseurl }}/images/rfigs/TS06-Facets-And-NDVI-Plotting-In-R/plot-PAR-NDVI-1.png) 

This is nice but a bit confusing as the time on our x-axis doesn't fully line
up.  To fix this we can assign the same min and max to both x-axes so that
they align.  We can also label the axes 


    plot2.par.2011 <- plot.par.2011 +
      scale_x_date(labels = date_format("%b %d"),
                   breaks = "3 months", minor_breaks= "1 week",
                   limits=c(min(NDVI.2011$Date),max=max(NDVI.2011$Date)))+
      ylab("Total PAR") + xlab ("")
     
    
    plot2.NDVI.2011 <- plot.NDVI.2011 +
      scale_x_date(labels = date_format("%b %d"),
                   breaks = "3 months", minor_breaks= "1 week",
                   limits=c(min(NDVI.2011$Date),max=max(NDVI.2011$Date)))+
      ylab("Total NDVI") + xlab ("Date")
    
    
    grid.arrange(plot2.par.2011, plot2.NDVI.2011) 

![ ]({{ site.baseurl }}/images/rfigs/TS06-Facets-And-NDVI-Plotting-In-R/plot-same-xaxis-1.png) 

#Challenge: Plot Air Temperature and NDVI
Create a complementary plot pairing with Air Temperateure and NDVI.  Choose
colors and symobls that show the data well.  Finally, plot air temperature, PAR,
and NDVI in a single window to ease comparisons.  

![ ]({{ site.baseurl }}/images/rfigs/TS06-Facets-And-NDVI-Plotting-In-R/challengeplot-same-xaxis-1.png) ![ ]({{ site.baseurl }}/images/rfigs/TS06-Facets-And-NDVI-Plotting-In-R/challengeplot-same-xaxis-2.png) 


##Two plots with One x-axis.  
We are able to nicely see the three different variables.  However, we waste a
lot of space to repeating the x-axes and the titles.  Instead of three seperate
plots we could alternatively, use facets and plot all the variables of interests
(NDVI, air temperature, precipiation, and PAR) on a single x-axis.  To do this
we'll 
use `melt()` from the `reshape2` package. `melt` tranforms the data from a wide
format (columns of different variables with values in them), to a long format 
(column of variables with a column of values) organized according to a specified
variable.  


    library(reshape2)  #allows us to "melt" dataframes from "wide" to "long"
    
    #merge the two data frames by date and retain all 'harMet.daily
    harMetNDVIall.daily.2011<- merge(harMet.daily2011, NDVI.2011, by.x = "date", 
                                  by.y = "Date", all.x=TRUE)
    
    #convert from "wide" form to "long" form
    harMetNDVI.daily.2011.long<-melt(harMetNDVIall.daily.2011, id ="date")

    ## Warning: attributes are not identical across measure variables; they will
    ## be dropped

Once we have the data in the long form we can subset and then plot the data. 


    #subset to retain just the variables of interest.  The vertical bar character
    # means "OR".
    harMetNDVI.daily.2011.select<-subset(harMetNDVI.daily.2011.long,
                                    variable=="meanNDVI"| variable== "prec"|
                                    variable == "airt" | variable == "part")
    
    NDVI.harMet.facet.plot<-ggplot(harMetNDVI.daily.2011.select,
                                   aes(date, value), group=variable) +
      geom_point() +
      facet_grid(variable~., scales="free") +   #specify facets & y-axis can vary
      ggtitle("Harvard Forest 2011") +
      scale_x_date(labels = date_format("%b %d"),  #abbreviated month & day
                   breaks = "3 months", minor_breaks= "1 month") +  #where grid is
      xlab ("Date") + ylab ("Value") +
      theme(legend.position = "none",
            plot.title = element_text(lineheight=.8, face="bold",size = 20),
            text = element_text(size=10)) 
    
    NDVI.harMet.facet.plot

![ ]({{ site.baseurl }}/images/rfigs/TS06-Facets-And-NDVI-Plotting-In-R/plot-same-x-axis-2-1.png) 
