---
title: "R Graphics Cookbook Notes"
author: "David D. Lee"
date: "May 3, 2016"
output: html_document
---



## barplot
### - grouped bar

```r
ggplot(cabbage_exp, aes(x = Date, y = Weight, fill = Cultivar)) + 
  geom_bar(position = position_dodge(0.7), width = 0.5, stat = "identity") +
  geom_text(aes(y = Weight + 0.1, label = Weight), size = 3, 
            position = position_dodge(1.2))
```

```
## Warning: position_dodge requires non-overlapping x intervals
```

![plot of chunk grouped bar](figure/grouped bar-1.png)

```r
# position_dodge to set interval length between groups
# aes(y=Weight+1) means the label always above the top of bar
```

- reorder, fill with variable

```r
upc <- subset(uspopchange, rank(Change) > 40)

# reorder Abb by Change
ggplot(upc, aes(x = reorder(Abb, Change), y = Change, fill = Region)) +
  geom_bar(stat = "identity", colour = "black") +
  scale_fill_manual(values = c("#669933", "#FFCC66"))
```

![plot of chunk reorder, fill with variable](figure/reorder, fill with variable-1.png)

### - stack bar plot

```r
# calculate the height of label cumutively
ce <- arrange(cabbage_exp, Date, Cultivar)
ce <- ddply(ce, "Date", transform, label_y=cumsum(Weight) - Weight*0.5)

ggplot(ce, aes(x=Date, y=Weight, fill=Cultivar)) +
  geom_bar(stat="identity", colour="black") +
  # reverse the order of legend
  guides(fill=guide_legend(reverse = TRUE)) +
  geom_text(aes(y=label_y, label=paste(format(Weight, nsamll=2), "kg")), 
            size=4, colour = "blue") +
  scale_fill_brewer(palette="Pastel2")
```

![plot of chunk stack bar plot](figure/stack bar plot-1.png)

### - Cleveland plot

```r
tophit <- tophitters2001[1:25, ]

# reorder name by lg, and avg. reorder() function only by one order varible
name_order <- tophit$name[order(tophit$lg, tophit$avg)]
tophit$name <- factor(tophit$name, levels=name_order)

ggplot(tophit, aes(x=avg, y=name)) +
  # add line startwith 0 and end with name
  geom_segment(aes(yend=name), xend=0, colour="grey50") +
  geom_point(size=3, aes(colour=lg)) +
  scale_colour_brewer(palette="Set2", limits=c("NL", "AL")) +
  theme_bw() +
  theme(panel.grid.major.y = element_blank(), # delete horizonal grid
        legend.position = c(1, 0.55), # put legend into the plot area
        legend.justification = c(1, 0.5))
```

![plot of chunk cleveland plot](figure/cleveland plot-1.png)

## lineplot

### - point on line

```r
ggplot(worldpop, aes(x=Year, y=Population)) + geom_line() + 
  geom_point() + scale_y_log10()
```

![plot of chunk point on line](figure/point on line-1.png)

### - area plot

```r
sunspotyear <- data.frame(
  Year = as.numeric(time(sunspot.year)),
  Sunspots = as.numeric(sunspot.year)
)

ggplot(sunspotyear, aes(x=Year, y=Sunspots)) + 
  geom_area(fill="blue", alpha=.2) +
  geom_line()
```

![plot of chunk area plot](figure/area plot-1.png)

### - stack area plot

```r
ggplot(uspopage, aes(x=Year, y=Thousands, fill=AgeGroup, order=desc(AgeGroup))) +
  geom_area(colour=NA, alpha=.4) + # fill colour to NA to eleminate frames
  scale_fill_brewer(palette="Blues") +
  geom_line(position="stack", size=.2)
```

![plot of chunk stack area plot](figure/stack area plot-1.png)

### - add confidence interval

```r
clim <- subset(climate, Source == "Berkeley", select=c("Year","Anomaly10y","Unc10y"))
ggplot(clim, aes(x=Year, y=Anomaly10y)) +
  geom_ribbon(aes(ymin=Anomaly10y-Unc10y, ymax=Anomaly10y+Unc10y), alpha=.2) +
  # note that ribbon() goes ahead line()
  geom_line()
```

![plot of chunk confidence interval](figure/confidence interval-1.png)

## Scatter plot

### grouped scatter

```r
hw <- heightweight
hw$weightGroup <- cut(hw$weightLb, breaks=c(-Inf, 100, Inf),
                      labels=c("< 100", "> 100"))
ggplot(hw, aes(x=ageYear, y=heightIn, shape=sex, fill=weightGroup)) +
  geom_point(size=2.5) +
  scale_shape_manual(values=c(21, 24)) +
  scale_fill_manual(values=c(NA, "black"),
                    guide=guide_legend(override.aes = list(shape=21))) # modigy legend
```

![plot of chunk grouped scatter](figure/grouped scatter-1.png)

### map continuns variable to plot attributes(colour or size)

```r
ggplot(heightweight, aes(x=ageYear, y=heightIn, size=weightLb, colour=sex)) +
  geom_point(alpha=.5) +
  scale_size_area() + # use area, map size is using distance(will be squared in area)
  scale_color_brewer(palette="Set2")
```

![plot of chunk mapped size](figure/mapped size-1.png)

### marginal rugs

```r
ggplot(faithful, aes(x=eruptions, y=waiting)) +
  geom_point() +
  geom_rug(position = "jitter", size=.2) # add jitter to avoid too much 
```

![plot of chunk marginal rugs](figure/marginal rugs-1.png)

### add anominations to scatter plot

```r
cdat<-subset(countries, Year==2009 & healthexp>2000)
cdat$Name1 <- cdat$Name
idx <- cdat$Name1 %in% c("Canada", "Ireland", "United Kingdom", "United States",
                         "New Zealand", "Iceland", "Japan", "Luxembourg",
                         "Netherlands", "Switzerland")
cdat$Name1[!idx] <- NA
ggplot(cdat, aes(x=healthexp, y=infmortality)) +
  geom_point() +
  geom_text(aes(x=healthexp+100, label=Name1), size=4, hjust=0) +
  xlim(2000, 10000)
```

```
## Warning: Removed 17 rows containing missing values (geom_text).
```

![plot of chunk add anominations](figure/add anominations-1.png)

### bubble plot with grids
both axis are factors

```r
hec <- HairEyeColor[,,"Male"] + HairEyeColor[,,"Female"]
# convert to long format
hec <- melt(hec, value.name="count")

ggplot(hec, aes(x=Eye, y=Hair)) +
  geom_point(aes(size=count), shape=21, colour="black", fill="cornsilk") +
  scale_size_area(max_size=20, guide=FALSE) +
  geom_text(aes(y=as.numeric(Hair)-sqrt(count)/22, label=count), vjust=1,
            colour="grey60", size=4)
```

![plot of chunk bubble plot](figure/bubble plot-1.png)

## Data Distribution
### histogram grouped

```r
birthwt$smoke<-factor(birthwt$smoke)
ggplot(birthwt, aes(x=bwt, fill=smoke)) +
  geom_histogram(position="identity", alpha=.4)
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![plot of chunk histogram grouped](figure/histogram grouped-1.png)

### kernel density with histogram

```r
ggplot(faithful, aes(x=waiting, y=..density..)) + # convert to proper scales
  geom_histogram(fill="cornsilk", colour="grey60", size=.2) +
  geom_density() +
  xlim(35, 105)
```

```
## `stat_bin()` using `bins = 30`. Pick better value with `binwidth`.
```

![plot of chunk kernel density](figure/kernel density-1.png)

### box plot

```r
ggplot(birthwt, aes(x=factor(race), y=bwt)) +
  geom_boxplot(notch=TRUE) # add notches
```

```
## notch went outside hinges. Try setting notch=FALSE.
```

![plot of chunk box plot](figure/box plot-1.png)

```r
# add mean values
ggplot(birthwt, aes(x=factor(race), y=bwt)) +
  geom_boxplot() +
  stat_summary(fun.y="mean", geom="point", shape=23, size=3, fill="white")
```

![plot of chunk box plot](figure/box plot-2.png)

### violin plot

```r
ggplot(heightweight, aes(x=sex, y=heightIn)) +
  geom_violin(adjust=.5, trim=FALSE) + # smooth degree
  geom_boxplot(width=.1, fill="black", outlier.color = NA) +
  stat_summary(fun.y=median, geom="point", fill="white", shape=21, size=2.5)
```

![plot of chunk violin plot](figure/violin plot-1.png)

### Wikinson dotplot

```r
countries2009<-subset(countries, Year==2009 & healthexp>=2009)
ggplot(countries2009, aes(x=infmortality)) +
  geom_dotplot(method="histodot", binwidth=.25) +
  geom_rug() +
  scale_y_continuous(breaks=NULL) +
  theme(axis.title.y=element_blank())
```

![plot of chunk dotplot](figure/dotplot-1.png)

### Grouped dotplot with boxplot

```r
ggplot(heightweight, aes(x=sex, y=heightIn)) +
  geom_boxplot(aes(x=as.numeric(sex) + .2, group=sex), width=.25) + # adjust locations
  geom_dotplot(aes(x=as.numeric(sex) - .2, group=sex), binaxis="y",
               binwidth=.5, stackdir="center") +
  scale_x_continuous(breaks=1:nlevels(heightweight$sex),
                     labels=levels(heightweight$sex))
```

![plot of chunk grouped dotplot](figure/grouped dotplot-1.png)

### 2D density plot

```r
ggplot(faithful, aes(x=eruptions, y=waiting)) +
  geom_point() +
  stat_density2d()
```

![plot of chunk 2D denstity plot](figure/2D denstity plot-1.png)


```r
ggplot(faithful, aes(x=eruptions, y=waiting)) +
  stat_density2d(aes(fill=..density..), geom="raster",
                 contour=FALSE, h=c(.5, 5)) # control the scale
```

![plot of chunk density to fill](figure/density to fill-1.png)


```r
ggplot(faithful, aes(x=eruptions, y=waiting)) +
  stat_density2d(aes(alpha=..density..), geom="tile",# tile plot
                 contour=FALSE) +
  geom_point() +
  annotate("text", x=-Inf, y=Inf, label="Upper left", hjust=-.2, vjust=2) +
  annotate("text", x=mean(range(faithful$eruptions)), y=-Inf, vjust=-0.4,
           label="Bottom middle")
```

![plot of chunk density to alpha](figure/density to alpha-1.png)

## Annomation
### Math formula in annomations

```r
ggplot(data.frame(x=c(-3,3)), aes(x=x)) +
  stat_function(fun=dnorm) +
  annotate("text", x=2, y=0.3, parse=TRUE, 
           label=" 'Function:' * frac(1, sqrt(2*pi)) * e ^ {-x^2 / 2}")
```

![plot of chunk math formula](figure/math formula-1.png)

```r
# single quote to show text
```

### Error Bar

```r
pd <- position_dodge(0.3)

ggplot(cabbage_exp, aes(x=Date, y=Weight, colour=Cultivar, group=Cultivar)) +
  geom_errorbar(aes(ymin=Weight-se, ymax=Weight+se),
                width=.2, size=0.25, colour="black", position=pd) +
  geom_line(position=pd) +
  geom_point(position=pd, size=2.5)
```

![plot of chunk error bar](figure/error bar-1.png)


## Coordinate Adjustment
### Reverse x and y axis

```r
# discrete
ggplot(PlantGrowth, aes(x=group, y=weight)) +
  geom_boxplot() +
  coord_flip() +
  scale_x_discrete(limits=rev(levels(PlantGrowth$group)))
```

![plot of chunk reverse axis](figure/reverse axis-1.png)

```r
# continuous
ggplot(PlantGrowth, aes(x=group, y=weight)) +
  geom_boxplot() +
  scale_y_reverse(limits=c(8,0))
```

![plot of chunk reverse axis](figure/reverse axis-2.png)

### Set the bound of axis

```r
p <- ggplot(PlantGrowth, aes(x=group, y=weight)) +
  geom_boxplot()

# adjust scales
p + scale_y_continuous(limits=c(5,6.5))
```

```
## Warning: Removed 13 rows containing non-finite values (stat_boxplot).
```

![plot of chunk set bound](figure/set bound-1.png)

```r
# coordinate transform
p + coord_cartesian(ylim=c(5,6.5))
```

![plot of chunk set bound](figure/set bound-2.png)

### Control the ratio of x and y axis

```r
ggplot(marathon, aes(x=Half, y=Full)) +
  geom
```

```
## Error in eval(expr, envir, enclos): object 'geom' not found
```
