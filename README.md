# README

## Data

1. data.csv	    : full data (I am not sure where I got this, but nowadays this is not difficult to find)
2. data_mex.csv	    : Mexico data (created by `grep Mexico data.csv> data_mex.csv`)
3. smalldf.csv	    : Last 25 days for Mexico (a simple `tail` in `R`)
   - col1	date as numbers
   - col2	cumulative deaths
   - col3	deaths per day
4. plot.Rmd	    : The R markdown file 
5. plot.html	    : Generated from `plot.Rmd`

## Problem

[Here](https://stackoverflow.com/questions/15102254/how-do-i-add-different-trend-lines-in-r) are two methods to add multiple fits to a plot.

Trying to add an exponential fit to my data was useless. Using the `ggplot2` approach my code was very similar:

```R
p + stat_smooth(method = 'nls', formula = y ~ a * exp(b * x), se = FALSE, method.args = list(start = list(a = 1, b = 1)))
```

But still nothing ... the error was:

```
Warning message:
Computation failed in `stat_smooth()`:
Missing value or an infinity produced when evaluating the model
```

As I did replicate the examples in the above link. The problem was either my data or the arguments `a` and `b`. I tried all combinations for my arguments, but no luck. Next I thought that maybe the first days (those without deaths) were making the algorithm that finds `a` and `b` to fail. That is why I get the first version of `smalldf`. But again the same error. So it was something else...

## Solution
So the next weird thing in line about my data was the date. So if I change the date to numbers everything runs. 

