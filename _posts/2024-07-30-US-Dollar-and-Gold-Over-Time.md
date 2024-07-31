Historical Correlation Between Gold Prices and the U.S. Dollar
================
Edith
2024-07-30

#### Historical Correlation Between Gold Prices and the U.S. Dollar

Is it truly the case that the value of gold and the value of the dollar
have moved in opposite directions throughout history? To investigate
this,I decided to conduct my own analysis using the R programming
language, leveraging daily data for its richness in information.

In order to visually examine the correlation between these two datasets,
I calculated the year-over-year changes for both gold prices and the
U.S. dollar. By plotting these changes on a graph, I aimed to uncover
any apparent trends or correlations that might exist between the two
assets.

``` r
library(quantmod)
library(dplyr)
library(lubridate)
library(xts)
```

``` r
# US dollar index
getSymbols("DX-Y.NYB", verbose = TRUE)
```

    ## downloading  DX-Y.NYB .....

    ## Warning: DX-Y.NYB contains missing values. Some functions will not work if
    ## objects contain missing values in the middle of the series. Consider using
    ## na.omit(), na.approx(), na.fill(), etc to remove or replace them.

    ## [1] "DX-Y.NYB"

``` r
usd <- `DX-Y.NYB`$`DX-Y.NYB.Adjusted`

usd <- data.frame(date = index(usd), price = coredata(usd))
usd$date <- as.Date(usd$date)
colnames(usd) <- c("date", "usd")

# Create a new column for the date one year ago
usd <- usd %>%
  mutate(date_one_year_ago = date - years(1))

# Join the data with itself on the date and date_one_year_ago columns
usd_yoy <- usd %>%
  left_join(usd %>% select(date_one_year_ago = date, usd_one_year_ago = usd), 
            by = "date_one_year_ago") %>%
  filter(!is.na(usd) & !is.na(usd_one_year_ago)) %>%
  mutate(usd_yoy = (log(usd) - log(usd_one_year_ago)) * 100)

usd_yoy <- xts(usd_yoy$usd_yoy, order.by=usd_yoy$date)
names(usd_yoy) <- c("usd_yoy")
```

``` r
# Gold Price; Gold Dec 24 (GC=F)

getSymbols("GC=F", verbose = TRUE)
```

    ## downloading  GC=F .....

    ## Warning: GC=F contains missing values. Some functions will not work if objects
    ## contain missing values in the middle of the series. Consider using na.omit(),
    ## na.approx(), na.fill(), etc to remove or replace them.

    ## [1] "GC=F"

``` r
gold <-`GC=F`$`GC=F.Adjusted`

gold <- data.frame(date = index(gold), price = coredata(gold))
gold$date <- as.Date(gold$date)
colnames(gold) <- c("date", "gold")

# Create a new column for the date one year ago
gold <- gold %>%
  mutate(date_one_year_ago = date - years(1))

# Join the data with itself on the date and date_one_year_ago columns
gold_yoy <- gold %>%
  left_join(gold %>% select(date_one_year_ago = date, gold_one_year_ago = gold), 
            by = "date_one_year_ago") %>%
  filter(!is.na(gold) & !is.na(gold_one_year_ago)) %>%
  mutate(gold_yoy = (log(gold) - log(gold_one_year_ago)) * 100)

gold_yoy <- xts(gold_yoy$gold_yoy, order.by=gold_yoy$date)
names(gold_yoy) <- c("gold_yoy")
```

### Create the plot

``` r
library(ggplot2)
library(reshape2)

# Merge the two xts objects by date
combined <- merge(usd_yoy, gold_yoy, join = "inner")
combined <- data.frame(date = index(combined), coredata(combined))

# Rename columns for clarity
colnames(combined) <- c("date", "usd_yoy", "gold_yoy")

# Melt the data for ggplot
combined <- melt(combined, id.vars = "date")

ggplot(combined, aes(x = date, y = value, color = variable)) +
  geom_line() +
  scale_y_continuous(
    name = "USD YoY",
    sec.axis = sec_axis(~ ., name = "Gold YoY")
  ) +
  labs(title = "USD YoY and Gold YoY Over Time", x = "Date") +
  scale_color_manual(values = c("blue", "red"), labels = c("USD YoY", "Gold YoY")) +
  theme_minimal() +
  theme(
    axis.title.y.right = element_text(color = "red"),
    axis.title.y.left = element_text(color = "blue"),
    legend.position = c(0.95, 0.95), # Position legend at the top right
    legend.justification = c("right", "top") # Adjust legend justification
  )
```

![](2024-07-30-USD-and-Gold-Over-Time_files/figure-gfm/library%20for%20plot-1.png)<!-- -->

The results suggest a clear inverse relationship between the two assets.
When the value of the dollar decreases, the value of gold tends to
increase, and vice versa. This inverse correlation appears to be quite
pronounced when examining the plotted data. Understanding this
relationship can be particularly beneficial for risk management
strategies, as it highlights the potential for gold to act as a hedge
against fluctuations in the dollar’s value. Such insights are valuable
for investors and policymakers alike, as they navigate the complexities
of financial markets and seek to optimize asset allocation and risk
mitigation strategies.
