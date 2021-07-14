---
title: "VAST Challenge 2021 - Mini-Challenge 3 "
description: |
  A short description of the post.
author:
  - name: Tang Yue
    url: {}
date: 07-14-2021
output:
  distill::distill_article:
    self_contained: false
---




# 1. Overview

In the roughly twenty years that Tethys-based GAStech has been operating a natural gas production site in the island country of Kronos, it has produced remarkable profits and developed strong relationships with the government of Kronos. However, GAStech has not been as successful in demonstrating environmental stewardship.

In January, 2014, the leaders of GAStech are celebrating their new-found fortune as a result of the initial public offering of their very successful company. In the midst of this celebration, several employees of GAStech go missing. An organization known as the Protectors of Kronos (POK) is suspected in the disappearance, but things may not be what they seem.

On January 23, 2014, multiple events unfolded in Abila. You’ve been asked to come in to perform a retrospective analysis based on limited information about what took place. Your goal is to identify risks and how they could have been mitigated more effectively.

in this assignment we would provide some insights to help law enforcement from Kronos and Tethys.


# 2. Exploration and preparation of Data

There are two dataset provides:

(1) Microblog records that have been identified by automated filters as being potentially relevant to the ongoing incident

(2) Text transcripts of emergency dispatches by the Abila, Kronos local police and fire departments.

(3) maps of Abila and background documents.


Two dataset are stored in 3 csv files, they are csv-1700-1830.csv", "csv-1831-2000.csv' and  "csv-2001-2131.csv":

The data for Mini-Challenge 3 is found in three segments.

Segment 1 covers the time period from 1700 to 1830 Abila time on January 23. 

Segment 2 covers the time period from 1830 to 2000 Abila time on January 23. 

Segment 3 covers the time period from 2000 to shortly after 2130 Abila time on January 23. 

### 2.1 Install and load R package

First, we run the below code to set the environment.


In this assignment, the tidyverse, ggforce, GGally, plotly R and parcoords packages will be used, which could be seen from below code chunk.


<div class="layout-chunk" data-layout="l-body">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='va'>packages</span> <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='st'>'tidyverse'</span>,<span class='st'>'dplyr'</span>,<span class='st'>'readr'</span>,
             <span class='st'>'UpSetR'</span>,<span class='st'>'ggplot2'</span>,<span class='st'>'topicmodels'</span><span class='op'>)</span>
<span class='kw'>for</span> <span class='op'>(</span><span class='va'>p</span> <span class='kw'>in</span> <span class='va'>packages</span><span class='op'>)</span><span class='op'>{</span>
  <span class='kw'>if</span><span class='op'>(</span><span class='op'>!</span><span class='kw'><a href='https://rdrr.io/r/base/library.html'>require</a></span><span class='op'>(</span><span class='va'>p</span>, character.only <span class='op'>=</span> <span class='cn'>T</span><span class='op'>)</span><span class='op'>)</span><span class='op'>{</span>
  <span class='fu'><a href='https://rdrr.io/r/utils/install.packages.html'>install.packages</a></span><span class='op'>(</span><span class='va'>p</span><span class='op'>)</span>
  <span class='op'>}</span>
  <span class='kw'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='op'>(</span><span class='va'>p</span>,character.only <span class='op'>=</span> <span class='cn'>T</span><span class='op'>)</span>
<span class='op'>}</span>
</code></pre></div>

</div>



### 2.2 Import dataset and combine data

The data of Microblog records and emergency calls are stored in separate csv files. And three csv files share same columns but data generate from different date, which is shown below.

![](image/1.files.PNG)

Firstly, we need to combine 3 files into one consolidated file, which is necessary for the following analysis. In this step, package 'tidyverse' would be used.
The following R code shows the process of data consolidation, then three dataset with different have been integrated into one file.


```{.r .distill-force-highlighting-css}
```
