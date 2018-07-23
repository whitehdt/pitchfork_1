---
title: "Pitchfork_1"
author: "Drew Whitehead"
date: "7/20/2018"
output: 
  html_document:
    keep_md: true
---



## Pitchfork Analysis: Preliminary EDA (part 1):

The dataset utilized contains over 18,000 reviews ranging in years from 1999 to 2017. Changes in bias can be studied by examining changes in the ratings assigned for the years from 2000 thru 2016 (which contain complete records YTD). In order to determine evidence of bias against commercial artists, artists needed to be segmented into three artist classes; Established (artists with eight or more reviews since 2000), Semi-Established (artists with four or more reviews, but less than eight, since 2000), and New (artists with less than four reviews since 2000).

The first analysis of potential bias is an examination average rating assigned and number of best new music awards given by artist class, year over year. The top panel of plot one is an examination of the average rating, which shows that while Established artists consistently scored higher than New and Semi-Established artists throughout the 00’s, the average ratings of New artists have overtaken Established artists by 2016. The bottom panel of plot one is an examination of the number of best new music awards assigned, which shows that while the best new music awards were distributed equally initially, the awards to New artists outpace existing artists by 650% by 2016. While average rating could provide some evidence of bias, best new music ratings are a clear indicator of bias.



![](Pitchfork_1_files/figure-html/cars-1.png)<!-- -->
