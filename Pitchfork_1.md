---
title: "Pitchfork_1"
author: "Drew Whitehead"
date: "7/22/2018"
output: 
  html_document:
    keep_md: true
---



## Pitchfork Analysis Part 1

Project Overview:
Pitchfork is a music website out of Chicago, which is primarily known for their comprehensive reviews recently released music. While Pitchfork started off as an indie music publication (indie music stands for independent music, which is produced independently of commercial record label), it has morphed into an industry wide publication over the years. Critiques from Pitchfork are given in the form of a detailed evaluation, an album rating and an assigned ‘Best New Music’ award if the rating is high enough. These reviews are often considered the final word in the critique of music, however, despite shifting to an industry wide focus they’re still considered biased at times against non-indie artists. This analysis will look to examine which factors are indicative of positively (and negatively) reviewed albums and determine if Pitchfork has any noticeable biases in their critique of the music industry. 

Preliminary EDA:
The dataset utilized contains over 18,000 reviews ranging in years from 1999 to 2017. Bias can be studied by examining changes in the ratings assigned for the years from 2000 thru 2016 (which contain complete records YTD). In order to determine evidence of bias against commercial artists, artists needed to be segmented into three artist classes; Established (artists with eight or more reviews since 2000), Semi-Established (artists with four or more reviews, but less than eight, since 2000), and New (artists with less than four reviews since 2000).

The first analysis (below) of potential bias is an examination average ratings assigned and number of best new music awards given by artist class, year over year. The top panel of plot one shows that while Established artists consistently scored higher than New and Semi-Established artists throughout the 00’s, the average ratings of New artists have over taken Established artists by 2016. The bottom panel of plot one shows that while the best new music awards were distributed equally initially, the awards to New artists outpace existing artists by 650% by 2016. While average rating could provide some evidence of bias, best new music ratings are a clear indicator of bias.



```r
# libraries
library(ggplot2)
library(RSQLite)
library(xlsx)
```

```
## Loading required package: rJava
```

```
## Loading required package: xlsxjars
```

```r
library(dplyr)
```

```
## 
## Attaching package: 'dplyr'
```

```
## The following objects are masked from 'package:stats':
## 
##     filter, lag
```

```
## The following objects are masked from 'package:base':
## 
##     intersect, setdiff, setequal, union
```

```r
library(plyr)
```

```
## -------------------------------------------------------------------------
```

```
## You have loaded plyr after dplyr - this is likely to cause problems.
## If you need functions from both plyr and dplyr, please load plyr first, then dplyr:
## library(plyr); library(dplyr)
```

```
## -------------------------------------------------------------------------
```

```
## 
## Attaching package: 'plyr'
```

```
## The following objects are masked from 'package:dplyr':
## 
##     arrange, count, desc, failwith, id, mutate, rename, summarise,
##     summarize
```

```r
  # set working directory to sqlite db and connect
setwd("/Users/drewwhitehead/sqlite/")
con <- dbConnect(SQLite(), dbname="pitchfork.sqlite")

  # return table names
alltables <- dbListTables(con)

  # qurey each table name and assign to object
artists <- dbGetQuery(con, 'select * from artists')
genres <- dbGetQuery(con, 'select * from genres')
reviews <- dbGetQuery(con, 'select * from reviews')
labels <- dbGetQuery(con, 'select * from labels')
years <- dbGetQuery(con, 'select * from years')
dbDisconnect(con)
#content <- read.csv("Solo Projects/Exported CSV Files/content.csv", header = T)

  # create master dataframe from sqlite output
pitchfork_master <- merge(artists, genres)
pitchfork_master <- merge(pitchfork_master, reviews)
pitchfork_master <- merge(pitchfork_master, labels)
pitchfork_master <- merge(pitchfork_master, years)

  # set working directory back to normal
setwd("/Users/drewwhitehead/Documents/Northwestern/R-Studio/")

  # write cluster results to excel file
#write.xlsx(pitchfork_master, file = "Solo Projects/Pitchfork Master.xlsx")

# remove all duplicates from master file
pitchfork_master <- pitchfork_master[!duplicated(pitchfork_master$reviewid),]
nrow(pitchfork_master)
```

```
## [1] 18002
```

```r
  # create subset df of all reviews past '99 called mod_era
mod_era <- pitchfork_master[which(pitchfork_master[,'year']>1999 & pitchfork_master[,'year']<2017),]

  # remove global as it does not have enough records to be considered for average
mod_era <- mod_era[which(mod_era[,'genre']!='global'),]

  # create summary table of average score by genre ~ year
avg_rating_year <- as.data.frame(aggregate(mod_era[, "score"], list(mod_era$genre, mod_era$year), mean))

  # graph of avg score by genre YoY
p0 <- ggplot(data = avg_rating_year, aes(x = Group.2, y = x, colour = Group.1)) +       
  geom_line(aes(group = Group.1)) + geom_point() + xlab("Year") + ylab("Score") + 
  ggtitle("Avg Score by Genre YoY") + theme(plot.title = element_text(hjust = 0.5))

  # create list of artists with over 10 reviews
artist_count_df <- as.data.frame(table(mod_era$artist))
colnames(artist_count_df)[1] <- 'artist'
artist_count_df <- artist_count_df[artist_count_df$Freq >= 4,]
artist_count_df$established.rating <- NA
artist_count_df[artist_count_df$Freq >= 8, 3] <- 'Established'
artist_count_df$established.rating[is.na(artist_count_df$established.rating)] <- 'Semi-Established'
artist_count_df <- artist_count_df[-2]

  # join list of established artists
mod_era <- join(mod_era, artist_count_df, by = 'artist', type='left')
mod_era$established.rating[is.na(mod_era$established.rating)] <- 'New'

  # create summary table of average score by genre ~ author
avg_rating_year_established <- as.data.frame(aggregate(mod_era[, "score"], list(mod_era$year, 
                                                                                          mod_era$established.rating), mean))

  # create plot of avg score by establishment rating YoY
p1 <- ggplot(data = avg_rating_year_established, aes(x = Group.1, y = x, colour = Group.2)) +       
  geom_line(aes(group = Group.2)) + geom_point() + xlab("Year") + ylab("Score") + 
  ggtitle("Avg Score by Establishment Rating YoY") + theme(plot.title = element_text(hjust = 0.5))

  # create summary table of average score by genre ~ author
bnm_count_year_established <- as.data.frame(aggregate(mod_era[, "best_new_music"], list(mod_era$year, 
                                                                                mod_era$established.rating), sum))

  # create plot of avg score by establishment rating YoY
p2 <- ggplot(data = bnm_count_year_established, aes(x = Group.1, y = x, colour = Group.2)) +       
  geom_line(aes(group = Group.2)) + geom_point() + xlab("Year") + ylab("Count of Best New Music") + 
  ggtitle("Count of Best New Music by Establishment Rating YoY") + theme(plot.title = element_text(hjust = 0.5))

  # display plots 1 and 2
multiplot <- function(..., plotlist=NULL, file, cols=1, layout=NULL) {
  library(grid)
  plots <- c(list(...), plotlist)
  numPlots = length(plots)
  if (is.null(layout)) {
    layout <- matrix(seq(1, cols * ceiling(numPlots/cols)),
                     ncol = cols, nrow = ceiling(numPlots/cols))
  }
  
  if (numPlots==1) {
    print(plots[[1]])
    
  } else {
    # Set up the page
    grid.newpage()
    pushViewport(viewport(layout = grid.layout(nrow(layout), ncol(layout))))
    for (i in 1:numPlots) {
      matchidx <- as.data.frame(which(layout == i, arr.ind = TRUE))
      print(plots[[i]], vp = viewport(layout.pos.row = matchidx$row,
                                      layout.pos.col = matchidx$col))
    }
  }
}
multiplot(p1, p2, cols=1)
```

![](Pitchfork_1_files/figure-html/cars-1.png)<!-- -->