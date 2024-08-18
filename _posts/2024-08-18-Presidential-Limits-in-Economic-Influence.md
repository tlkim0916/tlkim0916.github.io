2024-08-18-Presidential Limits in Economic Influence
================
Edith
2024-08-18

## Presidential Limits in Economic Influence

It seems the main point of this news is that a president alone has
limited impact on the economy’s performance, whereas the federal
reserve’s decisions are far more influential. This is a point I
completely agree with.

What directly affects the stock, loan, and bond markets are actions
taken by the federal reserve, such as changes in interest rates, which
influence how much people borrow and invest, and how the federal reserve
assesses the economy, which shapes overall market sentiment.

I believe the president’s role should be focused on setting policy, not
interfering with purely economic processes. While this approach might
seem like a slight drag on the economy in the short-term, it is crucial
for long-term stability.

If a president intervenes in these processes to create favorable
impressions during their term - just to claim the economy was strong
under their leadership - it could ultimately undermine the economy and
lead to long-term instability.

``` r
library(ggplot2)
library(fredr)

# Fetching the data
inf <- fredr(series_id = "CPIAUCNS", frequency = "m", units = "pc1")
rate <- fredr(series_id = "DFEDTARU", frequency = "m")

# Define the shading periods
trump_start <- as.Date("2017-02-01")
trump_end <- as.Date("2021-01-20")
biden_start <- as.Date("2021-01-20")
biden_end <- Sys.Date()

ggplot() +
  # Shading for Trump's presidency
  geom_rect(aes(xmin = trump_start, xmax = trump_end, ymin = -Inf, ymax = Inf),
            fill = "lightblue", alpha = 0.3) +
  # Shading for Biden's presidency
  geom_rect(aes(xmin = biden_start, xmax = biden_end, ymin = -Inf, ymax = Inf),
            fill = "lightgreen", alpha = 0.3) +
  # Line for US CPI Inflation
  geom_line(data = inf[inf$date >= as.Date("2017-01-01"), ], 
            mapping = aes(x = date, y = value, color = "US CPI Inflation"), linewidth = 1) +
  # Line for Federal Funds Rate
  geom_line(data = rate[rate$date >= as.Date("2017-01-01"), ], 
            mapping = aes(x = date, y = value, color = "Federal Funds Rate"), linewidth = 1) +
  # Add custom annotations for the presidencies
  annotate("text", x = as.Date("2018-06-01"), y = max(inf$value, na.rm = TRUE), 
           label = "Trump's Presidency", color = "blue", size = 4, hjust = 0) +
  annotate("text", x = as.Date("2022-06-01"), y = max(inf$value, na.rm = TRUE), 
           label = "Biden's Presidency", color = "green", size = 4, hjust = 0) +
  # Title and labels
  ggtitle("US CPI Inflation and Federal Funds Rate") +
  xlab("Year") +
  ylab("%") +
  # Custom colors for lines
  scale_color_manual(values = c("US CPI Inflation" = "red", "Federal Funds Rate" = "blue")) +
  theme_minimal()
```
![plot-1](../images/2024-08-18-Presidential-Limits-in-Economic-Influence/plot-1.png)<!-- -->
