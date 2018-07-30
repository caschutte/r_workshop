Introduction to R
================
Charles Schutte
June 29, 2018

This is an [R Markdown](http://rmarkdown.rstudio.com) Notebook. When you execute code within the notebook, the results appear beneath the code.

[R for Data Science](http://r4ds.had.co.nz/) is an excellent resource as you learn to use R for data analysis and visualization.

RStudio [cheat sheets](https://www.rstudio.com/resources/cheatsheets/) are also useful references to keep open while you are coding using a particular package.

For beginners, this [Base R Cheat Sheet](https://www.rstudio.com/wp-content/uploads/2016/10/r-cheat-sheet-3.pdf) will be very helpful.

To get started: 1. Open RStudio. 2. Selcte File -&gt; New Project... 3. Click "New Directory". Click "New Project". Enter a directory name and select its location. Click "Create Project" 4. Copy your data file into the directory you just created.

I prefer to use the [Tidyverse](https://www.tidyverse.org/) collection of packages for my data analysis. Packages are just collections of R functions that people have written and published to expand the capabilities of base R. Let's install the tidyverse.

``` r
# This is a code chunk. It lets you write R code within a Notebook. Commented lines like this one are not run in the code.

# This installs the tidyverse package
# install.packages("tidyverse")

# This loads the package into your workspace so that you can us it
library(tidyverse)
```

Now let's [import your data](http://r4ds.had.co.nz/data-import.html).

``` r
# You will use the read_csv function. Let's learn more about it:
help(read_csv)

# Now import your data
plants <- read_csv("TB_plant_data.csv")
```

    ## Parsed with column specification:
    ## cols(
    ##   Year = col_integer(),
    ##   Month = col_integer(),
    ##   Date = col_date(format = ""),
    ##   Region = col_character(),
    ##   Oil = col_character(),
    ##   Site = col_character(),
    ##   SiteID = col_character(),
    ##   Plot = col_character(),
    ##   Live_Density = col_integer(),
    ##   Live_Height = col_double(),
    ##   Live_Biomass = col_double(),
    ##   Dead_Density = col_integer(),
    ##   Dead_Height = col_double(),
    ##   Dead_Biomass = col_double(),
    ##   Litter_Biomass = col_double(),
    ##   Belowground_Biomass = col_double()
    ## )

``` r
# Open the imported data to take a look at it
View(plants)
```

This output tells you what kind of data you have in each column. Common types are double (number with decimal places), integer, character, and date. Your data now exists as a [tibble](http://r4ds.had.co.nz/tibbles.html) in the Environment tab at the top right of your screen. Click the arrow next to the tibble to get a preview of the columns and values it contains.

Let's try a simple transformation of these data. The tibble contains both live and dead biomass. It might be useful to calculate the total biomass.

``` r
plants$Total_Biomass <- plants$Live_Biomass + plants$Dead_Biomass
```

You might be interested in comparing the average stem densities at all four sites contained here. This can be done by creating a [pipe](http://r4ds.had.co.nz/pipes.html):

``` r
plants %>% 
  group_by(SiteID) %>% 
  summarize(MeanLiveDensity=mean(Live_Density), MeanDeadDensity = mean(Dead_Density))
```

    ## # A tibble: 4 x 3
    ##   SiteID MeanLiveDensity MeanDeadDensity
    ##   <chr>            <dbl>           <dbl>
    ## 1 TB1               203.            109.
    ## 2 TB2               323.            167.
    ## 3 TB3               255.            161.
    ## 4 TB4               231.            217.

Finally, let's [create a plot](http://r4ds.had.co.nz/data-visualisation.html) using this dataset.

``` r
# From "R for Data Science":
# "To make a graph, replace the bracketed sections in the code below with a dataset, a geom function, or a collection of mappings.""

# ggplot(data = <DATA>) + 
#  <GEOM_FUNCTION>(mapping = aes(<MAPPINGS>))

ggplot(data = plants) +
 geom_point(mapping = aes(x = Belowground_Biomass, y = Live_Biomass))
```

    ## Warning: Removed 45 rows containing missing values (geom_point).

![](Introduction_to_R_files/figure-markdown_github/Create%20a%20plot-1.png)

``` r
# Now let's break this down by site
ggplot(data = plants) +
 geom_point(mapping = aes(x = Belowground_Biomass, y = Live_Biomass)) +
 facet_wrap(~SiteID, nrow = 2)
```

    ## Warning: Removed 45 rows containing missing values (geom_point).

![](Introduction_to_R_files/figure-markdown_github/Create%20a%20plot-2.png)

``` r
# We can also use color to help us visualize patterns
plants$Month <- factor(plants$Month)
ggplot(data = plants) +
 geom_point(mapping = aes(x = Belowground_Biomass, y = Live_Biomass, color = Month)) +
 facet_wrap(~SiteID, nrow = 2)
```

    ## Warning: Removed 45 rows containing missing values (geom_point).

![](Introduction_to_R_files/figure-markdown_github/Create%20a%20plot-3.png)

``` r
# Many types of graphs are possible by changes the geom
ggplot(data = plants) +
 geom_boxplot(mapping = aes(x = Month, y = Live_Density, color = SiteID)) +
  labs(x = "Month of Year", y = "Live Plant Stem Density", title = "Best Plot Ever")
```

![](Introduction_to_R_files/figure-markdown_github/Create%20a%20plot-4.png)

A question came up during the workshop after plotting some data: "what to do with outliers?" This is more of a science/philosphy question than an R question. However, there are several ways that we can ignore outliers within R without altering the original data. For the sake of argument, let's decide that every live biomass value over 1000 is an outlier that we want to ignore. We can make that plot like this:

``` r
# One can do this within ggplot by changing the axis limits
ggplot(data = plants) +
 geom_point(mapping = aes(x = Belowground_Biomass, y = Live_Biomass)) +
 ylim(0,1000)
```

    ## Warning: Removed 56 rows containing missing values (geom_point).

![](Introduction_to_R_files/figure-markdown_github/Ignore%20outliers-1.png) Note that you have not altered the dataframe in anyway, the maximum live biomass is still 1659.25. Another solution is to remove the outliers from the dataframe entirely.

``` r
plants$Live_Biomass[plants$Live_Biomass > 1000] <- NA

# Now generate the original plot without setting limits
ggplot(data = plants) +
 geom_point(mapping = aes(x = Belowground_Biomass, y = Live_Biomass))
```

    ## Warning: Removed 56 rows containing missing values (geom_point).

![](Introduction_to_R_files/figure-markdown_github/Remove%20outliers-1.png)

Now the maximum live biomass is 956.02. The outliers are gone entirely. Note that the original data file remains unchanged! If you decide later that values over 1000 are not in fact outliers (which they are not), you can always go back and reload the data frame from the original dat file.

``` r
plants <- read_csv("TB_plant_data.csv")
```

    ## Parsed with column specification:
    ## cols(
    ##   Year = col_integer(),
    ##   Month = col_integer(),
    ##   Date = col_date(format = ""),
    ##   Region = col_character(),
    ##   Oil = col_character(),
    ##   Site = col_character(),
    ##   SiteID = col_character(),
    ##   Plot = col_character(),
    ##   Live_Density = col_integer(),
    ##   Live_Height = col_double(),
    ##   Live_Biomass = col_double(),
    ##   Dead_Density = col_integer(),
    ##   Dead_Height = col_double(),
    ##   Dead_Biomass = col_double(),
    ##   Litter_Biomass = col_double(),
    ##   Belowground_Biomass = col_double()
    ## )
