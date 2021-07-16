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
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='va'>packages</span> <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='st'>'tidyverse'</span>,<span class='st'>'dplyr'</span>,<span class='st'>'readr'</span>,<span class='st'>'tm'</span>,<span class='st'>'wordcloud'</span>,<span class='st'>'SnowballC'</span>,
             <span class='st'>'UpSetR'</span>,<span class='st'>'ggplot2'</span>,<span class='st'>'topicmodels'</span>,<span class='st'>'stringr'</span>,<span class='st'>'clock'</span>, <span class='st'>'tidytext'</span>,<span class='st'>'tokenizers'</span>,<span class='st'>'DT'</span><span class='op'>)</span>
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

1[](image/1.files.PNG)


Firstly, we need to combine 3 files into one consolidated file, which is necessary for the following analysis. In this step, package 'tidyverse' would be used.

The following R code shows the process of data consolidation, then three dataset with different have been integrated into one file.

<div class="layout-chunk" data-layout="l-body">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='kw'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='op'>(</span><span class='va'><a href='http://tidyverse.tidyverse.org'>tidyverse</a></span><span class='op'>)</span>
<span class='va'>table1</span> <span class='op'>&lt;-</span> <span class='fu'><a href='https://rdrr.io/r/utils/read.table.html'>read.csv</a></span><span class='op'>(</span><span class='st'>"csv-1700-1830.csv"</span><span class='op'>)</span>
<span class='va'>table2</span> <span class='op'>&lt;-</span> <span class='fu'><a href='https://rdrr.io/r/utils/read.table.html'>read.csv</a></span><span class='op'>(</span><span class='st'>"csv-1831-2000.csv"</span><span class='op'>)</span>
<span class='va'>table3</span> <span class='op'>&lt;-</span> <span class='fu'><a href='https://rdrr.io/r/utils/read.table.html'>read.csv</a></span><span class='op'>(</span><span class='st'>"csv-2001-2131.csv"</span><span class='op'>)</span>
<span class='va'>data</span> <span class='op'>&lt;-</span> <span class='fu'><a href='https://rdrr.io/r/base/cbind.html'>rbind</a></span><span class='op'>(</span><span class='va'>table1</span>, <span class='va'>table2</span>,<span class='va'>table3</span><span class='op'>)</span>
</code></pre></div>

</div>



Next, we would separate 'ccdata' and 'mbdata', they represent microblog record and emergency call center data collected by  the Abila, Kronos local police and fire departments.


<div class="layout-chunk" data-layout="l-body">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='va'>ccdata</span> <span class='op'>&lt;-</span> <span class='fu'><a href='https://rdrr.io/r/base/subset.html'>subset</a></span><span class='op'>(</span><span class='va'>data</span>, <span class='va'>type</span> <span class='op'>==</span> <span class='st'>"ccdata"</span><span class='op'>)</span>
<span class='va'>mbdata</span> <span class='op'>&lt;-</span> <span class='fu'><a href='https://rdrr.io/r/base/subset.html'>subset</a></span><span class='op'>(</span><span class='va'>data</span>, <span class='va'>type</span> <span class='op'>==</span> <span class='st'>"mbdata"</span><span class='op'>)</span>
</code></pre></div>

</div>



### 2.3 Data preprocessing

### 2.3.1 Modifying Date formate  

Date data in table were shown as scientific notation, so we firstly disabling scientific notation in R for further date processing.

<div class="layout-chunk" data-layout="l-body">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='kw'><a href='https://rdrr.io/r/base/library.html'>library</a></span><span class='op'>(</span><span class='va'><a href='http://lubridate.tidyverse.org'>lubridate</a></span><span class='op'>)</span>

<span class='fu'><a href='https://rdrr.io/r/base/options.html'>options</a></span><span class='op'>(</span>scipen <span class='op'>=</span> <span class='fl'>999</span><span class='op'>)</span>
</code></pre></div>

</div>


Next, package 'lubridate' would be used to convert data type from 'yyyymmddhhmmss' to 'yyyy-mm-dd hh:mm:ss', and create a new column 'date' in data, and the code chunk could be seen below.


<div class="layout-chunk" data-layout="l-body">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='va'>data</span><span class='op'>$</span><span class='va'>date.yyyyMMddHHmmss.</span><span class='op'>&lt;-</span> <span class='fu'><a href='https://rdrr.io/r/base/character.html'>as.character</a></span><span class='op'>(</span><span class='va'>data</span><span class='op'>$</span><span class='va'>date.yyyyMMddHHmmss.</span><span class='op'>)</span>
<span class='va'>data</span><span class='op'>$</span><span class='va'>date</span> <span class='op'>&lt;-</span>  <span class='fu'>date_time_parse</span><span class='op'>(</span><span class='va'>data</span><span class='op'>$</span><span class='va'>date.yyyyMMddHHmmss.</span>,
                                      zone <span class='op'>=</span> <span class='st'>""</span>,
                                      format <span class='op'>=</span> <span class='st'>"%Y%m%d %H%M%s"</span><span class='op'>)</span>
</code></pre></div>

</div>


<div class="layout-chunk" data-layout="l-body">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='fu'>glimpse</span><span class='op'>(</span><span class='va'>data</span><span class='op'>$</span><span class='va'>date</span><span class='op'>)</span>
</code></pre></div>

```
 POSIXct[1:4063], format: NA NA NA NA NA NA NA NA NA NA NA NA NA NA NA NA NA NA NA NA ...
```

</div>



### 2.3.1 Text proprecessing

In this step, we would like to clean text, including punctuation removal, conveting lowe case, stop word removal, extra white space removal and stemming.

Cleaning the text data starts with making transformations like removing special characters from the text. This is done using the tm_map() function to replace special characters like / and | with a space. The next step is to remove the unnecessary whitespace and convert the text to lower case.



### 2.4 Classify Record

Junk referred to advertisements or financial purpose reports. The below code chunk is used to identify the spam reports.

<div class="layout-chunk" data-layout="l-body">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='va'>junk</span> <span class='op'>&lt;-</span> <span class='va'>data</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span><span class='op'>(</span><span class='fu'>str_detect</span><span class='op'>(</span><span class='va'>message</span>,<span class='st'>"#artists|#badcreadit|#cars|#followme|#nobanks|# nobank|#meds"</span><span class='op'>)</span><span class='op'>)</span> 

<span class='va'>junk</span> <span class='op'>&lt;-</span> <span class='va'>data</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span><span class='op'>(</span><span class='fu'>str_detect</span><span class='op'>(</span><span class='va'>author</span>,<span class='st'>"junkman|carjunkers|eazymoney"</span><span class='op'>)</span><span class='op'>)</span>
</code></pre></div>

</div>


<div class="layout-chunk" data-layout="l-body">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='va'>meaningful</span> <span class='op'>&lt;-</span> <span class='va'>data</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span><span class='op'>(</span><span class='fu'>str_detect</span><span class='op'>(</span><span class='va'>message</span>,<span class='st'>"#gowaway|#POKrally|#POK|#AlibaPost|#CentralBulletin|#POKRallyinthePark|#IntNews|#nopeace"</span><span class='op'>)</span><span class='op'>)</span> 

<span class='va'>meaningful</span> <span class='op'>&lt;-</span> <span class='va'>data</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span><span class='op'>(</span><span class='fu'>str_detect</span><span class='op'>(</span><span class='va'>author</span>,<span class='st'>"AbilaPost|anaregents"</span><span class='op'>)</span><span class='op'>)</span>
</code></pre></div>

</div>



Spam represents the no meaningful and irrelevant or inappropriate messages post online. The below code chunk is used to identify the spam reports.

<div class="layout-chunk" data-layout="l-body">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='va'>spam</span> <span class='op'>&lt;-</span> <span class='va'>data</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span><span class='op'>(</span><span class='fu'>str_detect</span><span class='op'>(</span><span class='va'>message</span>, <span class='st'>"Grammar||#POKlove#Lucio|#getoverit|#baa-baa| #HomelessAwareness|#trylove"</span><span class='op'>)</span><span class='op'>)</span>

<span class='va'>spam</span> <span class='op'>&lt;-</span> <span class='va'>data</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span><span class='op'>(</span><span class='fu'>str_detect</span><span class='op'>(</span><span class='va'>author</span>,<span class='st'>"KronosQuoth|Clevvah4Evah|FriendsOfKronos"</span><span class='op'>)</span><span class='op'>)</span>
</code></pre></div>

</div>


<div class="layout-chunk" data-layout="l-body">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'>#data$message &lt;- str_replace_all(data$message,'[[:punct:]]+', "")</span>

<span class='co'>#data$message &lt;- str_replace_all(data$message,fixed("@"),"")</span>

<span class='co'>#data$message &lt;- str_replace_all(data$message,fixed("#"),"")</span>

<span class='co'>#data$message &lt;- str_replace_all(data$message,fixed("&lt;"),"")</span>

<span class='co'>#data$message &lt;- str_replace_all(data$message,"[\u4e00-\u9fa5]+", "")</span>
</code></pre></div>

</div>





```





Next, with tidy text framework, we need to both break the text into individual tokens and transform it to a tidy data structure, in this step, we use tidytext's unnest_tokens() function.

<div class="layout-chunk" data-layout="l-body">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'>#author_count &lt;- data %&gt;%</span>
<span class='co'># count(author,sort = TRUE)</span>
<span class='co'>#author_count</span>
</code></pre></div>

</div>




<div class="layout-chunk" data-layout="l-body">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='co'>#usenet_words &lt;- data %&gt;%</span>
<span class='co'>#unnest_tokens(word, message) %&gt;%</span>
</code></pre></div>

</div>



<div class="layout-chunk" data-layout="l-body">


</div>



```{.r .distill-force-highlighting-css}
```
