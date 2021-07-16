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
                                      format <span class='op'>=</span> <span class='st'>"%Y%m%d %H%m%s"</span><span class='op'>)</span>
</code></pre></div>

</div>


<div class="layout-chunk" data-layout="l-body">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='fu'>glimpse</span><span class='op'>(</span><span class='va'>data</span><span class='op'>$</span><span class='va'>date</span><span class='op'>)</span>
</code></pre></div>

```
 POSIXct[1:4063], format: NA NA NA NA NA NA NA NA NA NA NA NA NA NA NA NA NA NA NA NA ...
```

</div>

# 2.3.2 Text proprecessing

In this step, we would like to clean text, including punctuation removal, converting lower case, stop word removal, extra white space removal and stemming.

Cleaning the text data starts with making transformations like removing special characters from the text. This is done using the tm_map() function to replace special characters like / and | with a space. The next step is to remove the unnecessary whitespace and convert the text to lower case.

<div class="layout-chunk" data-layout="l-body">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='va'>data</span><span class='op'>$</span><span class='va'>message</span> <span class='op'>&lt;-</span> <span class='fu'>str_replace_all</span><span class='op'>(</span><span class='va'>data</span><span class='op'>$</span><span class='va'>message</span>,<span class='st'>'[[:punct:]]+'</span>, <span class='st'>""</span><span class='op'>)</span>

<span class='va'>data</span><span class='op'>$</span><span class='va'>message</span> <span class='op'>&lt;-</span> <span class='fu'>str_replace_all</span><span class='op'>(</span><span class='va'>data</span><span class='op'>$</span><span class='va'>message</span>,<span class='fu'>fixed</span><span class='op'>(</span><span class='st'>"@"</span><span class='op'>)</span>,<span class='st'>""</span><span class='op'>)</span>

<span class='co'>#data$message &lt;- str_replace_all(data$message,fixed("#"),"")</span>

<span class='va'>data</span><span class='op'>$</span><span class='va'>message</span> <span class='op'>&lt;-</span> <span class='fu'>str_replace_all</span><span class='op'>(</span><span class='va'>data</span><span class='op'>$</span><span class='va'>message</span>,<span class='fu'>fixed</span><span class='op'>(</span><span class='st'>"&lt;"</span><span class='op'>)</span>,<span class='st'>""</span><span class='op'>)</span>
<span class='va'>data</span><span class='op'>$</span><span class='va'>message</span> <span class='op'>&lt;-</span> <span class='fu'>str_replace_all</span><span class='op'>(</span><span class='va'>data</span><span class='op'>$</span><span class='va'>message</span>,<span class='fu'>fixed</span><span class='op'>(</span><span class='st'>"˜"</span><span class='op'>)</span>,<span class='st'>""</span><span class='op'>)</span>

<span class='va'>data</span><span class='op'>$</span><span class='va'>message</span> <span class='op'>&lt;-</span> <span class='fu'>str_replace_all</span><span class='op'>(</span><span class='va'>data</span><span class='op'>$</span><span class='va'>message</span>,<span class='st'>"[\u4e00-\u9fa5]+"</span>, <span class='st'>""</span><span class='op'>)</span>
</code></pre></div>

</div>


### 2.3.3 Classify Record

Junk referred to advertisements or financial purpose reports. The below code chunk is used to identify the spam reports.

<div class="layout-chunk" data-layout="l-body">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='va'>junk</span> <span class='op'>&lt;-</span> <span class='va'>data</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span><span class='op'>(</span><span class='fu'>str_detect</span><span class='op'>(</span><span class='va'>message</span>,<span class='st'>"#artists|#badcreadit|#cars|#followme|#nobanks|# nobank|#meds|#cancer|#bestfood|#workAtHome|#gettingFired|#pharmacyripoff|#iThing
                    |homeworkers.kronos|#abilasFinest|#hungry|Easy make.money|
                    #iThing|visit this|#eastAbila link|#abilajobs|clickhere|
                    #welksFurniture| #swat |#bugs|visit this link|this site"</span><span class='op'>)</span><span class='op'>)</span> 

<span class='va'>junk</span> <span class='op'>&lt;-</span> <span class='va'>data</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span><span class='op'>(</span><span class='fu'>str_detect</span><span class='op'>(</span><span class='va'>author</span>,<span class='st'>"junkman|carjunkers|eazymoney|junk99902|junkieduck113
  |junkman377|junkman995"</span><span class='op'>)</span><span class='op'>)</span>
</code></pre></div>

</div>



Spam represents the no meaningful and irrelevant or inappropriate messages post online. The below code chunk is used to identify the spam reports.

<div class="layout-chunk" data-layout="l-body">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='va'>spam</span> <span class='op'>&lt;-</span> <span class='va'>data</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span><span class='op'>(</span><span class='fu'>str_detect</span><span class='op'>(</span><span class='va'>message</span>, <span class='st'>"Grammar||#POKlove#Lucio|#getoverit|#baa-baa| #HomelessAwareness|#trylove|#pictures|#nobodycares|#abilafire|#wishfulthinking|
                    Viktor-E|#hogwash|#RememberElian|#standoff|work from home|#blackvansrules|#schaber|#abilacityPark|#downwithkronos"</span><span class='op'>)</span><span class='op'>)</span>

<span class='va'>spam</span> <span class='op'>&lt;-</span> <span class='va'>data</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span><span class='op'>(</span><span class='fu'>str_detect</span><span class='op'>(</span><span class='va'>author</span>,<span class='st'>"KronosQuoth|Clevvah4Evah|FriendsOfKronos|
                    jaquesjoyce101|klingon4real|michelleR|SaveOurWildlands|
                    AbilaAllFaith|footfingers|GreyCatCollectibles|luvwool"</span><span class='op'>)</span><span class='op'>)</span>
</code></pre></div>

</div>



Apart from the Spam and Junk, other emails would be  classified into meaningful records, which could be received by the below code chunk.

<div class="layout-chunk" data-layout="l-body">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='va'>meaningful1</span> <span class='op'>&lt;-</span> <span class='fu'>anti_join</span><span class='op'>(</span><span class='va'>data</span>,<span class='va'>junk</span>,by<span class='op'>=</span><span class='st'>"message"</span><span class='op'>)</span> 
<span class='va'>meaningful</span> <span class='op'>&lt;-</span> <span class='fu'>anti_join</span><span class='op'>(</span><span class='va'>meaningful1</span>,<span class='va'>spam</span>,by<span class='op'>=</span><span class='st'>"message"</span><span class='op'>)</span>
</code></pre></div>

</div>



Next, we further process "#" in the tag which leave in the before pre-processing code chunk.

<div class="layout-chunk" data-layout="l-body">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='va'>junk</span><span class='op'>$</span><span class='va'>class</span> <span class='op'>&lt;-</span> <span class='st'>"junk"</span>
<span class='va'>spam</span><span class='op'>$</span><span class='va'>class</span> <span class='op'>&lt;-</span> <span class='st'>"spam"</span>
<span class='va'>meaningful</span><span class='op'>$</span><span class='va'>class</span> <span class='op'>&lt;-</span> <span class='st'>"meaningful"</span>
</code></pre></div>

</div>



<div class="layout-chunk" data-layout="l-body">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='va'>data_classed</span> <span class='op'>&lt;-</span> <span class='fu'><a href='https://rdrr.io/r/base/cbind.html'>rbind</a></span><span class='op'>(</span><span class='va'>spam</span>, <span class='va'>junk</span>,<span class='va'>meaningful</span><span class='op'>)</span>
</code></pre></div>

</div>


<div class="layout-chunk" data-layout="l-body">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='va'>data_classed</span><span class='op'>$</span><span class='va'>message</span> <span class='op'>&lt;-</span> <span class='fu'>str_replace_all</span><span class='op'>(</span><span class='va'>data_classed</span><span class='op'>$</span><span class='va'>message</span>,<span class='fu'>fixed</span><span class='op'>(</span><span class='st'>"#"</span><span class='op'>)</span>,<span class='st'>""</span><span class='op'>)</span>
</code></pre></div>

</div>


2.3.4 Data Tokenization 

In this code chunk below, unnest_tokens() of tidytext package is used to split the dataset into tokens, while stop_words() is used to remove stop-words.

Tokenization step
<div class="layout-chunk" data-layout="l-body">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='va'>data_classed_token</span> <span class='op'>&lt;-</span> <span class='va'>data_classed</span> <span class='op'>%&gt;%</span>
  <span class='fu'>unnest_tokens</span><span class='op'>(</span><span class='va'>word</span>, <span class='va'>message</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'><a href='https://rdrr.io/r/stats/filter.html'>filter</a></span><span class='op'>(</span><span class='fu'>str_detect</span><span class='op'>(</span><span class='va'>word</span>, <span class='st'>"[a-z']$"</span><span class='op'>)</span>,
         <span class='op'>!</span><span class='va'>word</span> <span class='op'>%in%</span> <span class='va'>stop_words</span><span class='op'>$</span><span class='va'>word</span><span class='op'>)</span>
</code></pre></div>

</div>



Count the frequent word in all data_class
<div class="layout-chunk" data-layout="l-body">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='va'>data_classed_token</span> <span class='op'>%&gt;%</span>
  <span class='fu'>count</span><span class='op'>(</span><span class='va'>word</span>, sort <span class='op'>=</span> <span class='cn'>TRUE</span><span class='op'>)</span>
</code></pre></div>

```
                                          word    n
1                                     pokrally 1370
2                                           rt  942
3                                   kronosstar  905
4                                          pok  480
5                                        abila  449
6                                         fire  389
7                                    abilapost  384
8                                       police  294
9                                        rally  292
10                                        life  210
11                                      people  192
12                             centralbulletin  176
13                       homelandilluminations  159
14                                         apd  155
15                                     grammar  154
16                                        dont  143
17                                     success  141
18                                     dancing  131
19                                         van  131
20                                     dolphin  118
21                                         tag  118
22                                     viktore  105
23                                    standoff  103
24                                        time  100
25                                        stop   98
26                                    building   92
27                                       scene   92
28                                     arrived   83
29                                          dr   75
30                                          im   75
31                                       units   75
32                                        park   74
33                                     control   71
34                                        cops   71
35                                  additional   69
36                          dancingdolphinfire   68
37                                      newman   68
38                                gelatogalore   67
39                                     reports   66
40                                      person   63
41                                    shooting   61
42                                         cop   60
43                                   residents   60
44                                     traffic   59
45                                       world   59
46                                      sylvia   57
47                                         afd   56
48                                         guy   56
49                                     intnews   55
50                                     failure   53
51                                      gelato   53
52                                      report   53
53                                  successful   52
54                                         day   51
55                                   apartment   50
56                                      living   50
57                                       crowd   49
58                                  department   49
59                                       youre   49
60                                         run   47
61                                        swat   47
62                                       stand   46
63                                        fear   45
64                                       lucio   45
65                                       dream   44
66                                    hostages   44
67                                       jakab   44
68                                        real   44
69                                     ithakis   43
70                                        wait   43
71                                        city   41
72                             newsonlinetoday   41
73                                    progress   41
74                                       black   40
75                                   officials   40
76                                     stefano   40
77                                     succeed   40
78                                     trapped   40
79                                      change   39
80                                  dispatched   39
81                                      galore   39
82                                         hit   39
83                                       madeg   39
84                                  terrorists   39
85                           internationalnews   38
86                                     officer   38
87                                   ambulance   37
88                                     hostage   37
89                                      public   37
90                                 alexandrias   36
91                                          di   36
92                                     megaman   36
93                                  newsonline   36
94                                        stay   36
95                                      trucks   36
96                                  businesses   35
97                                        fail   35
98                                       floor   35
99                                     injured   35
100                                       isnt   35
101                                      marek   35
102                                   peaceful   35
103                                    achieve   34
104                                       door   34
105                                 evacuation   34
106                                      minds   34
107                                      shots   34
108                                surrounding   34
109                                  achilleos   33
110                                       live   33
111                                       guys   31
112                                 paramedics   31
113                                   starting   31
114                                  continues   30
115                                   evacuate   30
116                                firefighter   30
117                                  afdheroes   29
118                                      crime   29
119                                     dreams   29
120                                        hes   29
121                                      start   29
122                                      wrong   29
123                                  evacuated   28
124                                      fired   28
125                                       shot   28
126                                       word   28
127                                     audrey   27
128                                  buildings   27
129                                    courage   27
130                                     desire   27
131                                  followers   27
132                                        gun   27
133                                       home   27
134                                 motivation   27
135                                   resident   27
136                                       safe   27
137                                       site   27
138                                        top   27
139                                      avoid   26
140                                 evacuating   26
141                                       mind   26
142                                    product   26
143                                      youve   26
144                                   hospital   25
145                                       idea   25
146                                     kronos   25
147                                   presence   25
148                                   remember   25
149                                    lorenzo   24
150                                      music   24
151                                        omg   24
152                                    remarks   24
153                                      trust   24
154                                     artist   23
155                                      check   23
156                                 difference   23
157                                     doesnt   23
158                                evacuations   23
159                                     inside   23
160                                    rescued   23
161                               truccotrucco   23
162                                      whats   23
163                                 apartments   22
164                                     coming   22
165                                    complex   22
166                                      crazy   22
167                                    firemen   22
168                                       song   22
169                                 suspicious   22
170                                     terror   22
171                                      waste   22
172                                     accept   21
173                                     afraid   21
174                                        bad   21
175                                        ben   21
176                                      block   21
177                                     corner   21
178                                     leader   21
179                                     matter   21
180                                      reach   21
181                                      tense   21
182                                   violence   21
183                                      worth   21
184                                     giving   20
185                                  professor   20
186                                         st   20
187                             dancingdolphin   19
188                                      found   19
189                                 government   19
190                                       hear   19
191                                       hope   19
192                                        lot   19
193                                       luck   19
194                                       move   19
195                                   overcome   19
196                                       risk   19
197                                    schaber   19
198                                     secret   19
199                                      takes   19
200                                     tunnel   19
201                                       unit   19
202                                       wind   19
203                                  abilafire   18
204                                     action   18
205                                   arrested   18
206                                       cars   18
207                                 choconibbs   18
208                                     coffee   18
209                                       dude   18
210                              environmental   18
211                                       feel   18
212                                     floors   18
213                            friendsofkronos   18
214                                    gastech   18
215                                   injuries   18
216                                       lose   18
217                                   measured   18
218                                     moving   18
219                                    playing   18
220                                     street   18
221                                    support   18
222                                     taking   18
223                                    vehicle   18
224                                    windows   18
225                                        act   17
226                                        ago   17
227                                      gonna   17
228                                       lots   17
229                                      river   17
230                                      smoke   17
231                                     strong   17
232                                       team   17
233                                    appears   16
234                                       band   16
235                                       bert   16
236                                     called   16
237                                     closed   16
238                                       firm   16
239                                     gunmen   16
240                                       lift   16
241                                    moments   16
242                                    parking   16
243                                    purpose   16
244                                  recommend   16
245                                 responders   16
246                                       road   16
247                                   soldiers   16
248                                      teach   16
249                                      usual   16
250                                    winning   16
251                                    witness   16
252                                  witnesses   16
253                                alarmsecure   15
254                                        bed   15
255                                  bicyclist   15
256                                   business   15
257                                  butterfly   15
258                                       call   15
259                                   children   15
260                                      count   15
261                                    develop   15
262                             discouragement   15
263                                     driver   15
264                                environment   15
265                                   failures   15
266                                     flames   15
267                                      hands   15
268                                    improve   15
269                                 incomplete   15
270                                       join   15
271                                       left   15
272                                      light   15
273                                     liking   15
274                                      names   15
275                                     nearby   15
276                                       news   15
277                                    pursued   15
278                                     remain   15
279                                 rogerroger   15
280                                     scared   15
281                                    special   15
282                                      stage   15
283                                   stepping   15
284                                     stones   15
285                                     surest   15
286                                    bathing   14
287                                    blessed   14
288                                      child   14
289                                    connect   14
290                                      daily   14
291                                    destiny   14
292                              determination   14
293                                      didnt   14
294                                        die   14
295                                     energy   14
296                                      fresh   14
297                                     future   14
298                                     genius   14
299                                      guest   14
300                                       hand   14
301                                  happiness   14
302                                      heard   14
303                                        ive   14
304                                    leaders   14
305                                      leave   14
306                                    morning   14
307                                   occupied   14
308                                    pursuit   14
309                               satisfaction   14
310                                       size   14
311                                   speaking   14
312                                    subject   14
313                                   suspects   14
314                                  wildlands   14
315                                  wonderful   14
316                                      arent   13
317                                      begun   13
318                                      build   13
319                                  collapsed   13
320                                  condition   13
321                                       dark   13
322                                       fall   13
323                                      ideas   13
324                                  mcconnell   13
325                                      money   13
326                                      power   13
327                                   protoguy   13
328                                   released   13
329                                    resolve   13
330                                    shooter   13
331                                  situation   13
332                                      sleep   13
333                                    talking   13
334                                     theyre   13
335                                   underway   13
336                                      upper   13
337                                    waiting   13
338                                achievement   12
339                                   breaking   12
340                                        car   12
341                                     carpet   12
342                                 challenges   12
343                                     chance   12
344                                   charging   12
345                                    command   12
346                                   concerns   12
347                                    curious   12
348                                   decision   12
349                           disturbancenoise   12
350                                  education   12
351                              entrepreneurs   12
352                                  expansion   12
353                                      fears   12
354                                     givers   12
355                                      hurts   12
356                               intersection   12
357                                    mistake   12
358                                      movie   12
359                          pokrallyinthepark   12
360                                   positive   12
361                                       save   12
362                                       seat   12
363                                      serve   12
364                                        set   12
365                                     takers   12
366                                    talents   12
367                                   tomorrow   12
368                                    unclear   12
369                               unsuccessful   12
370                               abilasfinest   11
371                                  assaultin   11
372                                       cast   11
373                                      catch   11
374                              circumstances   11
375                                  community   11
376                                    consent   11
377                                     decide   11
378                                  decisions   11
379                               difficulties   11
380                                       gift   11
381                                      guess   11
382                                       hell   11
383                                  imaginary   11
384                                   inferior   11
385                                        job   11
386                                       late   11
387                                       lost   11
388                                misdemeanor   11
389                                  perimeter   11
390                                       prof   11
391                                     result   11
392                                     return   11
393                                       rise   11
394                                    running   11
395                                        sin   11
396                                   speakers   11
397                                     stable   11
398                                       step   11
399                                      stone   11
400                     subjectcircumstancesin   11
401                                 succeeding   11
402                                       talk   11
403                              unconquerable   11
404                               vehicleblack   11
405                                      voice   11
406                                       whos   11
407                                        win   11
408                                    yelling   11
409                                      begin   10
410                                    bullets   10
411                                    captive   10
412                                     common   10
413                                  continued   10
414                                  corporate   10
415                                   destined   10
416                                   distance   10
417                                    driving   10
418                                         en   10
419                                   expanded   10
420                                   expected   10
421                                      extra   10
422                               firefighters   10
423                                      flush   10
424                                       free   10
425                                     ground   10
426                                       hour   10
427                                 identified   10
428                                   insanity   10
429                                      issue   10
430                                       jams   10
431                                    knowing   10
432                                   location   10
433                                     looked   10
434                                    meaning   10
435                                       mile   10
436                                       oars   10
437                                  occupants   10
438                                    prayers   10
439                                     pursue   10
440                                     refuse   10
441                                       rest   10
442                                      route   10
443                                       runs   10
444                                     safety   10
445                           saveourwildlands   10
446                                      shoot   10
447                                     sounds   10
448                                      speak   10
449                                   tactical   10
450                                    telling   10
451                                  terrorist   10
452                                   thinking   10
453                                    tonight   10
454                                      truck   10
455                                 understand   10
456                                     undone   10
457                                   vicinity   10
458                                   airplane    9
459                                     assist    9
460                                      aware    9
461                                     begins    9
462                                       boss    9
463                                     bricks    9
464                                     calmer    9
465                                      climb    9
466                                    comfort    9
467                                     create    9
468                                       dead    9
469                                    discuss    9
470                              disinterested    9
471                                   diverted    9
472                                      doors    9
473                                      doubt    9
474                                       draw    9
475                                      earth    9
476                                     eighty    9
477                                     excess    9
478                                 excitement    9
479                                 experience    9
480                                  explosion    9
481                                 foundation    9
482                                       gold    9
483                                     grocer    9
484                                      hanns    9
485                                     harder    9
486                                      heavy    9
487                                     heroes    9
488                                hospitalafd    9
489                                       hurt    9
490                                        ill    9
491                                inspiration    9
492                                   inspired    9
493                                        joy    9
494                                  knowledge    9
495                                        lay    9
496                                       love    9
497                               mahahomeland    9
498                                       male    9
499                                   mcconnel    9
500                                 misfortune    9
501                                     moment    9
502                               neighborhood    9
503                                     peaked    9
504                                    percent    9
505                                   pictures    9
506                                  providing    9
507                                       punt    9
508                                   searched    9
509                                   shooters    9
510                                     slowly    9
511                                     speech    9
512                                  strangest    9
513                                    subdued    9
514                                      tamed    9
515                                     thrown    9
516                                     tonite    9
517                            troubleatgelato    9
518                                    victims    9
519                                       wild    9
520                                     wisdom    9
521                                       wont    9
522                                    wounded    9
523                                        wow    9
524                                       zone    9
525                             accidentreport    8
526                                 accomplish    8
527                             accomplishment    8
528                                    actions    8
529                                        add    8
530                                    advised    8
531                                      ahead    8
532                                  atrotious    8
533                                 attendance    8
534                                    average    8
535                                    battles    8
536                                       bike    8
537                                     breath    8
538                                    breaths    8
539                                caterpillar    8
540                                    century    8
541                                     choice    8
542                                    concert    8
543                                    created    8
544                                 creativity    8
545                                      cross    8
546                                        doc    8
547                                      donot    8
548                                     dophin    8
549                                    edition    8
550                                      egeou    8
551                                      elses    8
552                                    empower    8
553                                 enthusiasm    8
554                                  estimates    8
555                                 excellence    8
556                                    expands    8
557                                     expect    8
558                                    failing    8
559                                     female    8
560                                      fight    8
561                                     fiscal    8
562                                        fun    8
563                                  genuinely    8
564                                    glories    8
565                                      goals    8
566                                    grammer    8
567                        greycatcollectibles    8
568                                    growing    8
569                                  happening    8
570                                      heart    8
571                                    helping    8
572                               ideapokrally    8
573                                         ii    8
574                                  imitation    8
575                                  impatient    8
576                                       joni    8
577                                 kidnapping    8
578                                    killing    8
579                                       lazy    8
580                          lessthanexcellent    8
581                                    limited    8
582                                       loss    8
583                                      makes    8
584                                masterpiece    8
585                                    measure    8
586                                     medals    8
587                                     mental    8
588                                  miserable    8
589                                   motivate    8
590                                   multiply    8
591                                 negotiated    8
592                                  obstacles    8
593                                      ocean    8
594                                   officers    8
595                                     online    8
596                               onlythetruth    8
597                                   ordinary    8
598                                   original    8
599                                originality    8
600                        ourcountryourrights    8
601                                        pin    8
602                                       plan    8
603                                       pour    8
604                                    private    8
605                                 proportion    8
606                                     pulled    8
607                                       quit    8
608                                      raise    8
609                                        ran    8
610                                     reason    8
611                                    refrain    8
612                                    replace    8
613                                  requested    8
614                                 sacrifices    8
615                                  screaming    8
616                                     settle    8
617                                      shalt    8
618                                      shore    8
619                                    shrinks    8
620                                      sight    8
621                                     single    8
622                                     sirens    8
623                                   spelling    8
624                                    started    8
625                                  struggles    8
626                                     talent    8
627                                      thous    8
628                                        thy    8
629                                     tongue    8
630                                   traveled    8
631                                      types    8
632                                       view    8
633                                   vigilant    8
634                                    walking    8
635                                   activism    7
636                                   argument    7
637                                      asset    7
638                                  attribute    7
639                                     bitter    7
640                                  blessings    7
641                                    blowing    7
642                                     cancer    7
643                                    capital    7
644                                    carpool    7
645                                      click    7
646                                   conceive    7
647                                    corrupt    7
648                                 corruption    7
649                                  criticism    7
650                                       cure    7
651                                 dangermice    7
652                                      dayin    7
653                                     dayout    7
654                                    decided    7
655                                 determined    7
656                                 dictionary    7
657                               disappointed    7
658                     diseasefundkronospayus    7
659                                   diseases    7
660                                   disguise    7
661                                   diverged    7
662                                     dolphn    7
663                                     doomed    7
664                                     easily    7
665                                    efforts    7
666                                    erratic    7
667                                     excuse    7
668                                    explain    7
669                                       fate    7
670                                    forgive    7
671                                    forward    7
672                                    gawking    7
673                       goodstuffkronoslinks    7
674                                      grasp    7
675                                  greatness    7
676                                       grow    7
677                                      grows    7
678                                     happen    7
679                                       head    7
680                                     hiding    7
681                                     hinder    7
682                                       hire    7
683                                 impossible    7
684                                    inherit    7
685                                     killed    7
686                                    license    7
687                                  literally    7
688                                     losing    7
689                                       meek    7
690                                   mitchell    7
691                                       nice    7
692                                nonjudgment    7
693                                 panopticon    7
694                                      parla    7
695                                         pd    7
696                                 permission    7
697                                      plant    7
698                                     poeple    7
699                                    poklove    7
700                                       post    7
701                                    praying    7
702                                 prettyrain    7
703                                  principle    7
704                                  questions    7
705                                     rarely    7
706                                   reaction    7
707                                      ready    7
708                                  readymade    7
709                                    reasons    7
710                                   reckless    7
711                                  relieving    7
712                                   repeated    7
713                          residentscontinue    7
714                                  responded    7
715                             responsibility    7
716                                      roads    7
717                                       sara    7
718                                     scares    7
719                              selfknowledge    7
720                                     shapes    7
721                                       sick    7
722                                     simply    7
723                                   sofitees    7
724                                   solution    7
725                                   someones    7
726                                       soul    7
727                                    streets    7
728                                     stupid    7
729                                        sum    7
730                                   terrible    7
731                                    tragedy    7
732                                       tree    7
733                                     trials    7
734                              trollingsnark    7
735                                unconfirmed    7
736                                 unexamined    7
737                                        ups    7
738                                    visible    7
739                                       week    7
740                                       wood    7
741                                     yellow    7
742                      abilapolicedepartment    6
743                                   activist    6
744                                alexandiras    6
745                                 anaregents    6
746                                      april    6
747                                   arrivals    6
748                                     arrive    6
749                                   attained    6
750                                  backwards    6
751                                      blame    6
752                                      bunch    6
753                                    chasing    6
754                                      cheap    6
755                                      close    6
756                                 complement    6
757                                  condiment    6
758                                confidently    6
759                               corporations    6
760                                       cost    6
761                                   creating    6
762                                     credit    6
763                                      dares    6
764                                       date    6
765                                       days    6
766                               descriptions    6
767                                       diff    6
768                                  difficult    6
769                                  direction    6
770                                       dots    6
771                                      elian    6
772                                      empty    6
773                                   figthers    6
774                                      filay    6
775                                    finding    6
776                                 firemedics    6
777                                     flavor    6
778                                      focus    6
779                                     follow    6
780                                 forgetting    6
781                                    fortune    6
782                                   function    6
783                                       gain    6
784                                     gifted    6
785                                       guns    6
786                                      habit    6
787                                       half    6
788                                    halfway    6
789                                    heading    6
790                                      heals    6
791                                    hearing    6
792                                        hey    6
793                                      house    6
794                                        hut    6
795                                   imagined    6
796                                 influences    6
797                                  inspiring    6
798                                 introduces    6
799                                  intuition    6
800                                       lame    6
801                                 leadership    6
802                                      loose    6
803                                   majority    6
804                                     marvel    6
805                                    massive    6
806                                    matters    6
807                                 meaningful    6
808                                     minion    6
809                               negotiations    6
810                                  neighbors    6
811                                      night    6
812                               observations    6
813                                    offered    6
814                          officia1abilapost    6
815                                 overcoming    6
816                                    patient    6
817                                      pause    6
818                                   planning    6
819                                      plate    6
820                                       poor    6
821                                      prize    6
822                                    produce    6
823                                      react    6
824                                    reading    6
825                                        red    6
826                                     refill    6
827                                    reflect    6
828                                remembering    6
829                                 requesting    6
830                                   resolves    6
831                                     rocket    6
832                                      saved    6
833                                      scary    6
834                                    science    6
835                                    service    6
836                                    serving    6
837                                    setting    6
838                                       ship    6
839                                  siaradsea    6
840                                     stroke    6
841                                      stuck    6
842                             sustainability    6
843                                       taxi    6
844                                   tenacity    6
845                                       test    6
846                                      timmy    6
847                                      watch    6
848                                      weary    6
849                                    weekend    6
850                                    wouldve    6
851                                      write    6
852                                    writing    6
853                                       yeah    6
854                              abilaallfaith    5
855                                abilafinest    5
856                              abilafiredept    5
857                                    absense    5
858                                   accident    5
859                                        alm    5
860                                        apb    5
861                               appreciation    5
862                                   arriving    5
863                                    article    5
864                                    awesome    5
865                                    benefit    5
866                                    blocked    5
867                                      bonus    5
868                                       book    5
869                                       born    5
870                                      break    5
871                                     brings    5
872                                       burn    5
873                                    capture    5
874                                     caught    5
875                                     church    5
876                                     closer    5
877                                coincidence    5
878                                   creative    5
879                               definiteness    5
880                                     demand    5
881                                     dodont    5
882                                      drive    5
883                                        due    5
884                                     easier    5
885                                  escalated    5
886                                        eye    5
887                                     failed    5
888                                   favorite    5
889                                   fighting    5
890                                     flares    5
891                             floorabilapost    5
892                                   followme    5
893                                footfingers    5
894                                    friends    5
895                       getemkronosfollowers    5
896                                      ghost    5
897                                       girl    5
898                                    gunfire    5
899                                     hazard    5
900                                   homeland    5
901                                       hows    5
902                               imaginations    5
903                                   incident    5
904                                 individual    5
905                                 innovation    5
906                                  interfere    5
907                            internationally    5
908                                   involved    5
909                                  isknowing    5
910                                jerkdrivers    5
911                                    keeping    5
912                                        kid    5
913                                  kidnapped    5
914                                       kill    5
915                                  laserlike    5
916                                    leaving    5
917                                     legacy    5
918                                limitations    5
919                                  limitless    5
920                                     listen    5
921                                      local    5
922                                        low    5
923                                        mad    5
924                                    marshal    5
925                                     martao    5
926                                    mastery    5
927                                    maximum    5
928                                       meet    5
929                                    meeting    5
930                                       miss    5
931                                    missing    5
932                                   multiple    5
933                                negotiating    5
934                                 negotiator    5
935                                      north    5
936                                     office    5
937                              opportunities    5
938                               organization    5
939                                       past    5
940                                      peace    5
941                                        pls    5
942                              pokkronosjoin    5
943                                       pole    5
944                                      polvo    5
945                              possibilities    5
946                                  potential    5
947                                    program    5
948                                 protection    5
949                                    quality    5
950                                   question    5
951                              rememberelian    5
952                            rememberjuliana    5
953                                   reporter    5
954                                 resistance    5
955                                    revenge    5
956                                        sad    5
957                                      seeds    5
958                                       shes    5
959                                       shop    5
960                                      shown    5
961                               simonhamaeth    5
962                                       sing    5
963                                      sings    5
964                                     social    5
965                                        sow    5
966                                     sowing    5
967                           sowkronoshelpnow    5
968                                    species    5
969                                      spend    5
970                                  spreading    5
971                                      staff    5
972                                      store    5
973                                     strike    5
974                                     strive    5
975                                surrendered    5
976                                   survival    5
977                                      swear    5
978                                     tethys    5
979                                     tomish    5
980                                       town    5
981                                       true    5
982                              truthforcadau    5
983                                    unknown    5
984                                    warrior    5
985                                     wheres    5
986                                     window    5
987                                      woman    5
988                                    worried    5
989                                      youll    5
990                                       100s    4
991                              abilacitypark    4
992                                     albert    4
993                                      alive    4
994                                     answer    4
995                                       arms    4
996                                   assisted    4
997                                  badcredit    4
998                                     beauty    4
999                                   blackvan    4
1000                                  boldness    4
1001                                     boost    4
1002                                      care    4
1003                                    carlys    4
1004                              carlyscoffee    4
1005                                       cat    4
1006                                     clean    4
1007                                   clothes    4
1008                                collection    4
1009                                continuing    4
1010                                controlled    4
1011                                     costs    4
1012                                      crap    4
1013                                   creator    4
1014                                   crucial    4
1015                                    cruise    4
1016                       dancingdolphinsfire    4
1017                                      dang    4
1018                                    danger    4
1019                                      dare    4
1020                                        de    4
1021                                     death    4
1022                                diminishes    4
1023                                 displaced    4
1024                             distinguishes    4
1025                                 donations    4
1026                                  dreaming    4
1027                                      east    4
1028                                      easy    4
1029                                   edessis    4
1030                                  einstein    4
1031                                      eira    4
1032                                    elodis    4
1033                                        em    4
1034                                 evolution    4
1035                                      exit    4
1036                                     faith    4
1037                                    family    4
1038                                   finally    4
1039                  fixitkronoscreditratings    4
1040                                  follower    4
1041                  followerskronosgetpeople    4
1042                                      food    4
1043                                      form    4
1044                                     front    4
1045                                getviolent    4
1046                                   goldman    4
1047                                     gotta    4
1048                                      hate    4
1049                           heroesabilapost    4
1050                                     hoses    4
1051                                 ignorance    4
1052                                illumation    4
1053                                incredible    4
1054                                inhalation    4
1055                                  inspires    4
1056                                    ithing    4
1057                                   juliana    4
1058                                   justice    4
1059                                       key    4
1060                                      kids    4
1061                                   learned    4
1062                                      link    4
1063                                     magic    4
1064                                   mailbox    4
1065                                      meds    4
1066                                   message    4
1067                                  millions    4
1068                                    missed    4
1069                                       mom    4
1070                                  movement    4
1071                                    nature    4
1072                                      nuts    4
1073                                     paint    4
1074                                   partial    4
1075                                     party    4
1076                                   patrons    4
1077                                 phenomena    4
1078                                      play    4
1079                                      poks    4
1080                                   popping    4
1081                                precaution    4
1082                              prescription    4
1083                                 president    4
1084                                  pressure    4
1085                                    rachel    4
1086                                  radicals    4
1087                                    rating    4
1088                                      rave    4
1089                                      read    4
1090                                  redisrad    4
1091                                   release    4
1092                                  reported    4
1093                                      rich    4
1094                                     rules    4
1095                                      sale    4
1096                               saranespola    4
1097                                   scandal    4
1098                                      send    4
1099                                     sheep    4
1100                                somethings    4
1101                              standoffover    4
1102                                    stores    4
1103                                     stuff    4
1104                                    system    4
1105                                     tells    4
1106                                    tethan    4
1107                                     times    4
1108                                    todays    4
1109                                  tomatoma    4
1110                                   totally    4
1111                                   treated    4
1112                                  treating    4
1113                                 trickling    4
1114                                understood    4
1115                           vandalismreport    4
1116                                viktortree    4
1117                                      warm    4
1118                                  watching    4
1119                                     water    4
1120                                    weight    4
1121                                      weve    4
1122                                       won    4
1123                                       wth    4
1124                                     yikes    4
1125                                       yup    4
1126                                abilaprays    3
1127                            activityreport    3
1128                                addressing    3
1129                                    agents    3
1130                                 allfaiths    3
1131                                  american    3
1132                                   anarchy    3
1133                                 announced    3
1134                                    arrest    3
1135                                    author    3
1136                                     award    3
1137                                     awful    3
1138                    badprofileskronostacky    3
1139                                     bated    3
1140                                  bestfood    3
1141                                      blah    3
1142                                   breadth    3
1143                          brewvebeenserved    3
1144                                  bringing    3
1145                               brontesriff    3
1146                                  brothers    3
1147                                     buddy    3
1148                        carskronoschoppers    3
1149                                    carson    3
1150                                    cheers    3
1151                              cleaningfish    3
1152                                   climate    3
1153                                   coffeee    3
1154                                   consist    3
1155                                   contact    3
1156                                   cookies    3
1157                                counseling    3
1158                                    couple    3
1159                                   credits    3
1160                                       cut    3
1161                                    dancin    3
1162                                      deal    3
1163                                      dear    3
1164                                   depends    3
1165                                      derp    3
1166                       dietkronosstarvenow    3
1167                                 distefano    3
1168                                  downtown    3
1169                                  dragging    3
1170                                  drnewman    3
1171                     drugskronoscheappills    3
1172                                     ellen    3
1173                                 employees    3
1174                                     enemy    3
1175                              entrepreneur    3
1176                                      erik    3
1177                                     event    3
1178                                    events    3
1179                                     evvah    3
1180                                      fast    3
1181                                    feelin    3
1182                                    felony    3
1183                           firstresponders    3
1184                                     folks    3
1185                                    forget    3
1186                                   founder    3
1187                                    france    3
1188                                      fund    3
1189                                    george    3
1190                                girlfriend    3
1191                                      govt    3
1192                                     greed    3
1193                                     green    3
1194                                   grocery    3
1195                                      haha    3
1196                                      hang    3
1197                                     happy    3
1198                                      hard    3
1199                                    havent    3
1200                                 hemprules    3
1201                                     henri    3
1202                                      hold    3
1203                                   holding    3
1204                                     homes    3
1205                                 impactful    3
1206                                   include    3
1207                               instruments    3
1208                                 interrupt    3
1209                                 intrinsic    3
1210                               introducing    3
1211                                    island    3
1212                           justiceforelian    3
1213                                   kapelou    3
1214                                    kidnap    3
1215                                      land    3
1216                                      lero    3
1217                                    lights    3
1218                                    lovely    3
1219                                    manage    3
1220                                    market    3
1221                                     means    3
1222                                medication    3
1223                       medskronoscheapmeds    3
1224                                      milk    3
1225                                     minor    3
1226                                  mistakes    3
1227                                 mitchells    3
1228                                  momentum    3
1229                                     month    3
1230                                    movies    3
1231                                  narcotic    3
1232                                   natural    3
1233                               neighboring    3
1234                                       net    3
1235                                  octavios    3
1236                                       ode    3
1237                               ouzerielian    3
1238                                     paris    3
1239                                peacefully    3
1240                              peacefullyat    3
1241                               penultimate    3
1242                            pharmacyripoff    3
1243                                     phone    3
1244                                     plans    3
1245                                     plays    3
1246                                     poked    3
1247                                   pollute    3
1248                                 pollution    3
1249                                      pray    3
1250                                    pretty    3
1251                                   pulling    3
1252                                    quotes    3
1253                                      raly    3
1254                                      rank    3
1255                             reggiewassali    3
1256                                   reminds    3
1257                                 reporting    3
1258                             responsibilty    3
1259                               responsible    3
1260                                   robeson    3
1261                                     rocks    3
1262                                    rotter    3
1263                                     salad    3
1264                                     sales    3
1265                                      saul    3
1266                                    saving    3
1267                                   scanner    3
1268                                   seizure    3
1269                               shoplifting    3
1270                                      sign    3
1271                                   singing    3
1272                                   sisters    3
1273                                     songs    3
1274                    sowkronosclimatechange    3
1275                                   speaker    3
1276                                  standing    3
1277                                 survivial    3
1278                                     swift    3
1279                                   teaches    3
1280                                 terrorism    3
1281                                    therye    3
1282                                      tied    3
1283                              timelessness    3
1284                                      told    3
1285                                underneath    3
1286                                 unfounded    3
1287                                 uninjured    3
1288                                  universe    3
1289                                      uofa    3
1290                                  upstairs    3
1291                                     urges    3
1292                                      vans    3
1293                                    waving    3
1294                                     words    3
1295                                     worst    3
1296                                       2nd    2
1297                                   500week    2
1298                                        5k    2
1299                                 abilajobs    2
1300                                    abilas    2
1301                              abilawatcher    2
1302                                  accepted    2
1303                                   account    2
1304                            accountability    2
1305                               accountable    2
1306                                 actingfor    2
1307                                 activists    2
1308                                   advises    2
1309                                  afdheros    2
1310                                       air    2
1311                                    aliens    2
1312                             alinskyauthor    2
1313                        allergictofurballs    2
1314                                 allruined    2
1315                                      ally    2
1316                                   amatuer    2
1317                                     angry    2
1318                                    animal    2
1319                                  animated    2
1320                                 announces    2
1321                                announcing    2
1322                                  anooying    2
1323                                  answered    2
1324                                   anthing    2
1325                                       apa    2
1326                                 apathetic    2
1327                                  applause    2
1328                                     arise    2
1329                                    arises    2
1330                                     armed    2
1331                                      army    2
1332                                 arrivedat    2
1333                                    ashtma    2
1334                               atachilleos    2
1335                                    attack    2
1336                                 attackers    2
1337                                 attention    2
1338                                  audience    2
1339                         audreynewmanworld    2
1340                              audreyrotter    2
1341                                      aura    2
1342                               authorities    2
1343                                        aw    2
1344                              awardwinnign    2
1345                                    baabaa    2
1346                                       bac    2
1347                                  backdoor    2
1348                                    barely    2
1349                                    battle    2
1350                                      bean    2
1351                                      beat    2
1352                                    beaten    2
1353                                 beautiful    2
1354                                   bedtime    2
1355                                    belief    2
1356                                  believes    2
1357                                     biker    2
1358                                   biology    2
1359                               bittersweet    2
1360                            blackvansrules    2
1361                                   blather    2
1362                     blatherblatherblather    2
1363                                     blaze    2
1364                                     blend    2
1365                                  blocking    2
1366                                     board    2
1367                                   booking    2
1368                                      boom    2
1369                                     bored    2
1370                                       bot    2
1371                                    bowing    2
1372                                   boycott    2
1373                                      boys    2
1374                                     brave    2
1375                        bravesthusbandever    2
1376                                     bring    2
1377                                 burnabila    2
1378                             burnabilaburn    2
1379                               businessman    2
1380                                      busy    2
1381                                       buy    2
1382                                    calmed    2
1383                                capitalist    2
1384                                  car4sale    2
1385                                    carbon    2
1386                                     cards    2
1387                                     cares    2
1388                                   carreer    2
1389                                  carrying    2
1390                                 cartridge    2
1391                                 celebrate    2
1392                                  chanters    2
1393                                   checkin    2
1394                                  cheering    2
1395                                  chekcing    2
1396                                 childless    2
1397                                   circles    2
1398                                  citizens    2
1399                                   classic    2
1400                                   closing    2
1401                               collaborate    2
1402                                comeondown    2
1403                                 concludes    2
1404                                 conclused    2
1405                                connection    2
1406                             consciousness    2
1407                              consequences    2
1408                           conservationist    2
1409                                   contest    2
1410                   contestkronosfreeithing    2
1411                                     copsr    2
1412                                 cormorant    2
1413                                  cornered    2
1414                              corperations    2
1415                         couldthisgetworse    2
1416                                   counter    2
1417                                   country    2
1418                                     cover    2
1419                                 crackdown    2
1420                                 craziness    2
1421                                      cred    2
1422                                    crying    2
1423                                    cuaght    2
1424                                       cuz    2
1425                        dancingdophinsfire    2
1426                         dansingdolfinfire    2
1427                        dansingdolphinfire    2
1428                      dateskronosclickhere    2
1429                                  debating    2
1430                                  declined    2
1431                     definitelygonnabelate    2
1432                             deludednation    2
1433                               description    2
1434                              destinations    2
1435                              destinyarise    2
1436                                  detailed    2
1437                               devastation    2
1438                                    device    2
1439                                 didnthave    2
1440                                differnece    2
1441                                      dire    2
1442                                  directly    2
1443                                discovered    2
1444                                 discredit    2
1445                                discussion    2
1446                               distraction    2
1447                                  distress    2
1448                                       dog    2
1449                                      doin    2
1450                                      doll    2
1451                                     downn    2
1452                            downwithkronos    2
1453                                     draws    2
1454                                   dropped    2
1455                                  drrotter    2
1456                                 dstrangeo    2
1457                                     dudes    2
1458                easycreditkronosmorecredit    2
1459                                ecological    2
1460                                 economics    2
1461                                 effective    2
1462                                  egeouapd    2
1463                                  emerging    2
1464                                engagement    2
1465                                    ernieo    2
1466                                   etheres    2
1467                                  evacuees    2
1468                                everybodys    2
1469                                  evidence    2
1470                                  exciting    2
1471                                    exotic    2
1472                                    extort    2
1473                                 extorting    2
1474                                      eyed    2
1475                                    fairly    2
1476                                  families    2
1477                                    famous    2
1478                                      fans    2
1479                                      farm    2
1480                                     farms    2
1481                                    feeble    2
1482                                   figured    2
1483                                firereport    2
1484                                  firewins    2
1485                                firghtened    2
1486                                      fist    2
1487                                     flits    2
1488                                   focused    2
1489                                  focusing    2
1490                                   foreign    2
1491                                     frame    2
1492                                     freak    2
1493                                   freezer    2
1494                                   funnrun    2
1495                                     funny    2
1496                                       gag    2
1497                                    gandhi    2
1498                                    gastov    2
1499                                  gathered    2
1500                      gelatogalorestandoff    2
1501                               gelatorocks    2
1502                                generation    2
1503                             generousabila    2
1504                                 getoverit    2
1505                                        gg    2
1506                                    ghandi    2
1507                                    glassy    2
1508                                    global    2
1509                                 goonsquad    2
1510                              governmental    2
1511                                   gowaway    2
1512                                  grandmas    2
1513                                   greeddy    2
1514                                      grew    2
1515                                    guests    2
1516                                  gunshots    2
1517                                        ha    2
1518                                      haev    2
1519                                   happend    2
1520                                  happened    2
1521                               harpsichord    2
1522                                  haunting    2
1523                                    headed    2
1524                                headlights    2
1525                                   heavily    2
1526                                      heck    2
1527                                  helpless    2
1528                                    helpus    2
1529                           hennyhenhendrix    2
1530                                     heres    2
1531                                     hippy    2
1532                                historical    2
1533                                   hogwash    2
1534                                  homeless    2
1535                                  hometown    2
1536                                       hop    2
1537                                    hoping    2
1538                                       hot    2
1539                                hourofneed    2
1540                                   howling    2
1541                   howworkathomekronosscam    2
1542                                     human    2
1543                                   hundred    2
1544                                    hungry    2
1545                                     hurry    2
1546                           hurtfirefighter    2
1547                                     idiot    2
1548               imreallypromptmostofthetime    2
1549                                       ims    2
1550                                 including    2
1551                           infernaltraffic    2
1552                               information    2
1553                                    injury    2
1554                                 injustice    2
1555                                       ink    2
1556                                   inspect    2
1557                             inspirational    2
1558                                 insurance    2
1559                                   intense    2
1560                           interdependence    2
1561                                interviews    2
1562                             investigation    2
1563                                   invites    2
1564                                  ivolence    2
1565                                       jax    2
1566                                   jobmake    2
1567                                     johnp    2
1568                                      joke    2
1569                                     jujst    2
1570                                     julie    2
1571                                    keping    2
1572                                     kicks    2
1573                                     kinda    2
1574                                     knock    2
1575                                   knocked    2
1576                                    leaves    2
1577                                legitimate    2
1578                                     lived    2
1579                                     lives    2
1580                                     lomax    2
1581                                     lucky    2
1582                                    lyrics    2
1583                                  magazine    2
1584                       makemoneykronoslink    2
1585                                     march    2
1586                             marcusdrymiau    2
1587                                      mars    2
1588                                     marta    2
1589                                   masters    2
1590                                  measures    2
1591                                   medical    2
1592                                 micheller    2
1593                                  midnight    2
1594                                    minded    2
1595                              mindofkronos    2
1596                                        mo    2
1597                                      moar    2
1598                                    mobile    2
1599                             moneygrubbers    2
1600                      moneykronossellstuff    2
1601                                    mortal    2
1602                                  mountain    2
1603                                     moved    2
1604                                     movig    2
1605                                    murder    2
1606                                  musician    2
1607                                   mystery    2
1608                                  national    2
1609                                neededfire    2
1610                             neededofficer    2
1611                                 negotiate    2
1612                                   newmans    2
1613                                      nite    2
1614                                   nobanks    2
1615                                    norest    2
1616                                    notios    2
1617                                     noway    2
1618                                  offering    2
1619                                  official    2
1620                                 omgomgomg    2
1621                           omgopmgomgthank    2
1622                                oppression    2
1623                                 organizer    2
1624                                       oso    2
1625                                    otters    2
1626                                   outsdie    2
1627                                      owed    2
1628                                     owner    2
1629                                   pahetic    2
1630                                      paid    2
1631                              particpation    2
1632                                   partner    2
1633                                   passion    2
1634                                   peeking    2
1635                                     peeks    2
1636                            peoplepokrally    2
1637                                   persons    2
1638                                       phd    2
1639                                photoalbum    2
1640                                    photos    2
1641                                      pics    2
1642                                      pigs    2
1643                                     pilau    2
1644                                pillowcase    2
1645                                    pissed    2
1646                      pleasedontgiveuponme    2
1647                                    podium    2
1648                                    pokers    2
1649                           pokkronosevents    2
1650                          pokliesinthepark    2
1651                                     pople    2
1652                                   povided    2
1653                                  powerful    2
1654                                 practices    2
1655                                    prayer    2
1656                                     press    2
1657                            princecharming    2
1658                              progresswith    2
1659                                propaganda    2
1660                                    proved    2
1661                                   prowler    2
1662                                  putitout    2
1663                                   quoting    2
1664                                    racing    2
1665                                     radio    2
1666                                    raiser    2
1667                                     range    2
1668                                      reap    2
1669                                  receives    2
1670                                   recipes    2
1671                           recommendations    2
1672                               reeducation    2
1673                                   reenter    2
1674                                  releases    2
1675                                   relieve    2
1676                                remarkable    2
1677                                remarksand    2
1678                                 renewable    2
1679                                renovation    2
1680                                 reporters    2
1681                                  rescuing    2
1682                                resolution    2
1683                                  response    2
1684                                   returns    2
1685                                   rewards    2
1686                                ridiculous    2
1687                                    rights    2
1688                                      riot    2
1689                                    rising    2
1690                                    robots    2
1691                                   rockers    2
1692                                   rotters    2
1693                                      ruin    2
1694                                    safely    2
1695                                savepeople    2
1696                                    scarey    2
1697                                   scarlet    2
1698                                    school    2
1699                                  sciences    2
1700                                scientists    2
1701                                    search    2
1702                                   section    2
1703                                   seesome    2
1704                                   sending    2
1705                                    served    2
1706                                   shaking    2
1707                                   shelter    2
1708                                     shift    2
1709                                      shut    2
1710                                  silenced    2
1711                                    silent    2
1712                                       sit    2
1713                                   sitting    2
1714                                  slippery    2
1715                                  slowdown    2
1716                      smadcokronosb33nk4xs    2
1717                       smadcokronoscbcje9q    2
1718                                     smart    2
1719                                     smell    2
1720                             smokygridlock    2
1721                    smushcomkronos154xu5xi    2
1722                                   society    2
1723                                  software    2
1724                     somebodywantgelatobad    2
1725                                   souliou    2
1726                                   sounded    2
1727                          sowkronosclasses    2
1728  sowkronosclimatechangedistafanointerview    2
1729                      sowkronosgreenliving    2
1730                         sowkronoswhoweare    2
1731                                  speeches    2
1732                                     srsly    2
1733                                statements    2
1734                                    status    2
1735                    stayingbehindmymailbox    2
1736                   stickyrheadinsomegelato    2
1737                                 stillonit    2
1738                              stillshaking    2
1739                                  stinking    2
1740                                   stoodup    2
1741                                    stooge    2
1742                                     story    2
1743                                    stress    2
1744                                    struck    2
1745                                     study    2
1746                                   stuffed    2
1747                                   stufpid    2
1748                                  subsides    2
1749                                   suddeny    2
1750                                    suffer    2
1751                                  summoned    2
1752                                 supported    2
1753                                supporting    2
1754                                 surrended    2
1755                                 surrender    2
1756                                  surround    2
1757                                 suspected    2
1758                               sustainable    2
1759                                      tada    2
1760                                 telephone    2
1761                                    tellme    2
1762                                 terorists    2
1763                                 terorrist    2
1764                                   terrror    2
1765                            thanks4nothing    2
1766                                 thanksafd    2
1767                                     thatz    2
1768                                  themself    2
1769                                    threat    2
1770                                thundering    2
1771                                      tint    2
1772                                     tired    2
1773                                  titanium    2
1774                                  tonights    2
1775                               trafficking    2
1776                             transitioning    2
1777                             trapanitweets    2
1778                                   trashed    2
1779                                   trouble    2
1780                                   trustus    2
1781                                   trylove    2
1782                                   tuesday    2
1783                                        tv    2
1784                                      type    2
1785                                typinghard    2
1786                                       ugh    2
1787                                      ugly    2
1788                                undercover    2
1789                                university    2
1790                                    untold    2
1791                                    unwind    2
1792                                    upyeah    2
1793                                        ur    2
1794                                       usa    2
1795                                  vacation    2
1796                                vegetables    2
1797                                    victim    2
1798                             viktorekronos    2
1799       viktorekronosourmusicstandupspeakup    2
1800            viktorekronosthebandinterviews    2
1801                                      walk    2
1802                                       wan    2
1803                                     wanna    2
1804                                      warn    2
1805                               watermelons    2
1806                                     waved    2
1807                                  welcomes    2
1808                                 welcoming    2
1809                                      west    2
1810                                  wetlands    2
1811                                    wheels    2
1812                                      wide    2
1813                           wishfulthinking    2
1814                                  withsome    2
1815                                      wnat    2
1816                                    worlds    2
1817                                     worry    2
1818                                       wtf    2
1819                                       wuz    2
1820                                 yeahright    2
1821                                  yeelling    2
1822                                   yesssss    2
1823                                  12string    1
1824                                 357second    1
1825                                     4ever    1
1826                                 abalckvan    1
1827                             abilaparadise    1
1828                                    abilia    1
1829                                accidental    1
1830                                  activity    1
1831                                     actor    1
1832                                 addtional    1
1833                                     admit    1
1834                                    advise    1
1835                                   affects    1
1836                                       age    1
1837                                    agenda    1
1838                                    aiding    1
1839                                alexandria    1
1840                       alienskronosamongus    1
1841                                   alinsky    1
1842                        almostcreamedbyvan    1
1843                                alzawahiri    1
1844                                  amblance    1
1845                                    amount    1
1846                                 anarchist    1
1847                                   animals    1
1848                                 answering    1
1849                                  anytbody    1
1850                                    apdare    1
1851                           apdfailpokrules    1
1852                                 apdheroes    1
1853                                   apdswat    1
1854                               appartments    1
1855                                       apt    1
1856                                      apts    1
1857                                 aptsearch    1
1858                                    arentt    1
1859                                areslittle    1
1860                                armageddon    1
1861                                   armored    1
1862                                 arresting    1
1863                                   arrests    1
1864                                   arrives    1
1865                                      arse    1
1866                                       art    1
1867                                   artists    1
1868                           assaultweaponin    1
1869                                 assessing    1
1870                                 assisting    1
1871                                  asterian    1
1872                         athenamusiconline    1
1873                                    awards    1
1874                                 awareness    1
1875                                background    1
1876                                   backway    1
1877                                    badges    1
1878                                      bads    1
1879                                     bands    1
1880                                      barf    1
1881                         beantheredonethat    1
1882                                      bear    1
1883                                     becuz    1
1884                                      beer    1
1885                                 beginning    1
1886                                   beleive    1
1887                                    believ    1
1888                                  believed    1
1889                                    belive    1
1890                                   belivef    1
1891                                       bet    1
1892                                      bets    1
1893                                   blasted    1
1894                                     bless    1
1895                                      blog    1
1896                                     blogs    1
1897                                     blood    1
1898                                     boast    1
1899                                      body    1
1900                                   bonfire    1
1901                                   boredom    1
1902                                    bottom    1
1903                                   bravely    1
1904                                     bribe    1
1905                         broadcastbuilding    1
1906                           broadcastfelony    1
1907                                        bs    1
1908                                       bug    1
1909                                      bugs    1
1910                                  buidling    1
1911                                    bummer    1
1912                                    buying    1
1913                                      buys    1
1914                                 bystander    1
1915                                bystanders    1
1916                                      cafe    1
1917                                      calm    1
1918                                cantbegood    1
1919                                  canthear    1
1920                               capitalists    1
1921                                  captives    1
1922                                      card    1
1923                                    career    1
1924                                   carrier    1
1925                                casualties    1
1926                                    caused    1
1927                                    chased    1
1928                      cheapdrugskronosmeds    1
1929                                   cheered    1
1930                                 chemicals    1
1931                                   chicken    1
1932                   chirpwatchkronoswatchme    1
1933                                  chouting    1
1934                                      chuc    1
1935                                   citizen    1
1936                               citizenship    1
1937                                   classis    1
1938                                  clearing    1
1939                                    clicke    1
1940                                   closure    1
1941                                      clue    1
1942                                      cnat    1
1943                                   comeing    1
1944                                    comeon    1
1945                                  comments    1
1946                            communications    1
1947                                   company    1
1948                                completely    1
1949                                congestion    1
1950                              construction    1
1951                                  continue    1
1952                           continuespolice    1
1953                              controlabila    1
1954                             controlstreet    1
1955                                 copkiller    1
1956                                   coppers    1
1957                                  cordoned    1
1958                                cormorants    1
1959                                 couchsurf    1
1960                                  courtesy    1
1961                                    cousin    1
1962                                  coverage    1
1963                                   covered    1
1964                                    covers    1
1965                                       cow    1
1966                                    craazy    1
1967                                     crash    1
1968                                    crimes    1
1969                                  criminal    1
1970                                 criminals    1
1971                                    custom    1
1972                                   cycling    1
1973                                   cyclist    1
1974                                     dance    1
1975                                   dancind    1
1976                         dancingdolfinfire    1
1977                               dancingfire    1
1978                              danzamacabra    1
1979                                    ddfire    1
1980                                 dedicated    1
1981                                   degrees    1
1982                              deportations    1
1983                                  deported    1
1984                                   deserve    1
1985                                destroying    1
1986                                  destroys    1
1987                                   details    1
1988                                    didhnt    1
1989                                     didtn    1
1990                                 differnce    1
1991                                    dinner    1
1992                                       dis    1
1993                               disappeared    1
1994                                  discount    1
1995                               discoveries    1
1996                                disgusting    1
1997                           dispatchedcrowd    1
1998                      dispatchedpedestrian    1
1999                              distinguised    1
2000                                  distract    1
2001                                      docs    1
2002                                      dogs    1
2003                                  dolphins    1
2004                                     dorel    1
2005                                  downsend    1
2006                                     drama    1
2007                                     drink    1
2008                                  drinking    1
2009                                    drinks    1
2010                                  driverin    1
2011                                   drivers    1
2012                                    drives    1
2013                                  driveway    1
2014                                     drugs    1
2015                                     drunk    1
2016                                   dtrejos    1
2017                                   dummies    1
2018                                      duty    1
2019                                 eastabila    1
2020                                       eat    1
2021                                    eating    1
2022                                  economic    1
2023                                   egghead    1
2024                                     elder    1
2025                                electrical    1
2026                                 emergency    1
2027                           emergencypolice    1
2028                             emergencyswat    1
2029                                    empire    1
2030                                  employer    1
2031                                endangered    1
2032                                   enemies    1
2033                                   enfuego    1
2034                                    engage    1
2035                                   engines    1
2036                                  engulfed    1
2037                                  entering    1
2038                              enthusiastic    1
2039                             enviroharmful    1
2040                                        er    1
2041                                     eriks    1
2042                                     ermou    1
2043                                 esitmated    1
2044                                 essential    1
2045                              establishing    1
2046                                   estefan    1
2047                       evacuationabilapost    1
2048                                   evening    1
2049                                  everyday    1
2050                                      evil    1
2051                                excietemtn    1
2052                                 exercises    1
2053                                  exerting    1
2054                                   exiting    1
2055                                    expand    1
2056                                 expanding    1
2057                                explosions    1
2058                                   exposed    1
2059                                    extent    1
2060                                  extorted    1
2061                                 extortion    1
2062                                   faculty    1
2063                                     false    1
2064                                 falseflag    1
2065                                 fantastic    1
2066                      fastjobskronostricks    1
2067                                       fat    1
2068                                 favorites    1
2069                                        fd    1
2070                                  featured    1
2071                                   feeling    1
2072                                  feelings    1
2073                                    fellow    1
2074                                   fighter    1
2075                                    figure    1
2076                                   figures    1
2077                                   filling    1
2078                                  finallly    1
2079                               finallyhere    1
2080                               fingerboard    1
2081                                  finished    1
2082                                fireiscool    1
2083                             fireisscience    1
2084                              firerefugees    1
2085                                     fires    1
2086                               firestarter    1
2087                         firstdatedisaster    1
2088                                      flag    1
2089                                  flashing    1
2090                                  flooding    1
2091                                    flurry    1
2092                                       fly    1
2093                                    flying    1
2094                                      folk    1
2095                 followerskronosgetpopular    1
2096                     followmekronosartists    1
2097                                     force    1
2098                                    forgot    1
2099                                  freebird    1
2100                                    friday    1
2101                                    frites    1
2102                                 fromparla    1
2103                               fulfillment    1
2104                                 furniture    1
2105                                       fyi    1
2106                                     galor    1
2107                                    garden    1
2108                                gastechpok    1
2109                                    getout    1
2110                              gettingfired    1
2111                                     givin    1
2112                         gladitsnotmyhouse    1
2113                                    gladly    1
2114                                    glaore    1
2115                              gleatogalore    1
2116                                     glory    1
2117                                     gonig    1
2118                               gonnabelate    1
2119                                gonnagetit    1
2120                                       goo    1
2121                                  goodness    1
2122                                 gooooooal    1
2123                                    gowawy    1
2124                                   grabbed    1
2125                                     grace    1
2126                                 graduated    1
2127                               grandmother    1
2128                                    greedy    1
2129                                   greenie    1
2130                                 greenlife    1
2131                                 greenwash    1
2132                              greenwashing    1
2133                                  gridlock    1
2134                                gridlocked    1
2135                             griefstricken    1
2136                                    guilty    1
2137                                    guitar    1
2138                                    gunman    1
2139                                   gunshot    1
2140                                    gurbuz    1
2141                                     hadnt    1
2142                                    handle    1
2143                                happeningn    1
2144                               happyending    1
2145                                   hasbeen    1
2146                                  headline    1
2147                                    hearts    1
2148                               helicoptors    1
2149                                    helpme    1
2150                                herratomas    1
2151                                 hesitates    1
2152                                     hilel    1
2153                                      hink    1
2154                                      hirt    1
2155                                  historic    1
2156                         hngoheboabilapost    1
2157                                      holy    1
2158  homelandilluminationkronosmarekinterview    1
2159                         homelessawareness    1
2160                              homelessness    1
2161                         homeworkerskronos    1
2162                   hotchickskronospictures    1
2163                      hottipskronostrading    1
2164                                    houses    1
2165                                       hte    1
2166                                      huge    1
2167                                  humanity    1
2168                                        id    1
2169                                       idb    1
2170                                  identity    1
2171                                    idiots    1
2172                                  ignorant    1
2173                       illiteratecriminals    1
2174                                   imagine    1
2175                               immediately    1
2176                                    impact    1
2177                             inconsiderate    1
2178                                   indoors    1
2179                                injurieshi    1
2180                                  innocent    1
2181                     inspirationalpokrally    1
2182                                 institute    1
2183                               institution    1
2184                               interviewed    1
2185                              interviewing    1
2186                                 interwebs    1
2187                      investigationrequest    1
2188            investigationrequestadditional    1
2189 investigationrequestlocationpersonvehicle    1
2190                                irrational    1
2191                                    issues    1
2192                          ithakisabilapost    1
2193                                  itolduso    1
2194                                     iwait    1
2195                                      iwll    1
2196                                     jacob    1
2197                                      jeez    1
2198                                    jewlry    1
2199                                   joining    1
2200                                     joins    1
2201                                    jojods    1
2202                                    joking    1
2203                                joybubbles    1
2204                            juliannabanana    1
2205                                     jumpy    1
2206                                   justhit    1
2207                                  justlike    1
2208                                 katerinas    1
2209                                  keyboard    1
2210                                      kick    1
2211                                    kicked    1
2212                                kidnappers    1
2213                                  kindafun    1
2214                                 kingwilly    1
2215                         klinekronossignup    1
2216                                  knocking    1
2217                                      konw    1
2218                                kronosians    1
2219                          kronoskidnapping    1
2220                                    laouto    1
2221                                     laugh    1
2222                                 lawmakers    1
2223                                    lawyer    1
2224                                 layinglow    1
2225                                     learn    1
2226                                   letting    1
2227                                     level    1
2228                                    ligths    1
2229                                      line    1
2230                            linkcitykronos    1
2231                                    linked    1
2232                               livecasting    1
2233                                      logo    1
2234                                      lola    1
2235                       lookingforsomething    1
2236                                    loooks    1
2237                                    losere    1
2238                                     loves    1
2239                                      lute    1
2240                                   luvwool    1
2241                                  madegafd    1
2242                                      maha    1
2243                                     mamin    1
2244                                   managed    1
2245                                  managing    1
2246                                   maniacs    1
2247                                     maple    1
2248                                     marak    1
2249                                    marine    1
2250                          marriagematerial    1
2251                               martaflores    1
2252                                      mass    1
2253                                    messed    1
2254                                microblogs    1
2255                                  miilling    1
2256                                     mikal    1
2257                                  mindless    1
2258                                   minutes    1
2259                             missingtethys    1
2260                                       mix    1
2261                                 mmmtastes    1
2262                     moneykronosworkathome    1
2263                                      mood    1
2264                                     moron    1
2265                                    mounts    1
2266                                        ms    1
2267                                       mtg    1
2268                                  murdered    1
2269                                   murders    1
2270                                 musicians    1
2271                                      mute    1
2272                                  mutually    1
2273                            nashvilletuned    1
2274                                     nasty    1
2275                           neededambulance    1
2276                           neededamublance    1
2277                              neededpolice    1
2278                              negoitations    1
2279                                negoitator    1
2280                             neighborhoods    1
2281                                   nervous    1
2282                                  neverwas    1
2283                                 newspaper    1
2284                                      nico    1
2285                                nightmares    1
2286                               nobodycares    1
2287                                      noes    1
2288                                      nooo    1
2289                                    nooooo    1
2290                                nooooooooo    1
2291                                   nopeace    1
2292                             noresalevalue    1
2293                                    norris    1
2294                                      note    1
2295                                    notice    1
2296                                  notified    1
2297                                 notmoving    1
2298                             notthatintome    1
2299                                  nowguess    1
2300                                  numorous    1
2301                                       nut    1
2302                                officially    1
2303                                   offsaid    1
2304  ohnoohnoohnoohnoohnoohnoohnoohnoohnoohno    1
2305                                       omc    1
2306                                 omgponies    1
2307                                 omgwtfbbq    1
2308                              ominousknock    1
2309                                 onethanks    1
2310                                    onfire    1
2311                             onl1nerecords    1
2312                                 operation    1
2313                              orchestrated    1
2314                                     outta    1
2315                                   outwant    1
2316                                  overlord    1
2317                              overreaction    1
2318                                     panel    1
2319                               panickattak    1
2320                                  paradise    1
2321                                    parlor    1
2322                                  partners    1
2323                               partnership    1
2324                                 passenger    1
2325                                passionate    1
2326                                peacecrowd    1
2327                                peacerules    1
2328                                    peeked    1
2329                         peoplearehorrible    1
2330                                   peoples    1
2331                                   peppers    1
2332                                   perform    1
2333                                performing    1
2334                                 permanent    1
2335                                 persevere    1
2336                                     photo    1
2337                                       pic    1
2338                                   picking    1
2339                            picskronospics    1
2340                                      plea    1
2341                                    poison    1
2342                                 poisoning    1
2343                                 pokkronos    1
2344                  pokkronosmeetourpartners    1
2345         pokkronosourcommunitybeansnthings    1
2346                   pokkronosrallyinthepark    1
2347                       pokkronoswaterfacts    1
2348                                  pokpeace    1
2349                                 pokpokpok    1
2350                          policenegotiator    1
2351                                 political    1
2352                                    pommes    1
2353                               poorpuppies    1
2354                                  possibly    1
2355                                    poster    1
2356                                       pot    1
2357                             powercrystals    1
2358                                      powr    1
2359                               practically    1
2360                                  practice    1
2361                           prayersanswered    1
2362                                    preety    1
2363                                   prepare    1
2364                             presentations    1
2365                                prestgious    1
2366                                   prevail    1
2367                              prizewinning    1
2368                               problemcall    1
2369                                proffessor    1
2370                                   profile    1
2371                                   profits    1
2372                                protecting    1
2373                                    proven    1
2374                                     punch    1
2375                                  punished    1
2376                    pursuitcontinuespolice    1
2377                             pursuitpolice    1
2378                                   putting    1
2379                               pyroatheart    1
2380                                   quickly    1
2381                                radicalism    1
2382                                   raising    1
2383                           rallybusinesses    1
2384                   rallykronosfriendsofllm    1
2385                                 rallypark    1
2386                                    rallys    1
2387                                      ramp    1
2388                                    random    1
2389                                    ransom    1
2390                                   reached    1
2391                                  realcity    1
2392                                  realized    1
2393                                  received    1
2394                        recklesshazaradous    1
2395                                recognized    1
2396                                 recycling    1
2397                          referkronosphish    1
2398                              relationship    1
2399                             relationships    1
2400                                  reminder    1
2401                                   renewal    1
2402                                    renown    1
2403                                    repair    1
2404                                    repent    1
2405                   requestlocationdwelling    1
2406             requestlocationpersonincident    1
2407                                  required    1
2408                                  research    1
2409                                  resolved    1
2410                                responding    1
2411                                restaurant    1
2412                                      ride    1
2413                                   rinaldo    1
2414                                   rioting    1
2415                                   ripping    1
2416                                   robbery    1
2417                                     roofs    1
2418                                    rootin    1
2419                                  rosewood    1
2420                                   roughly    1
2421                                     round    1
2422                                   roundup    1
2423                                     rubab    1
2424                                   ruining    1
2425                                     ruled    1
2426                                  runblack    1
2427                                   rundown    1
2428                                     runin    1
2429                                      sais    1
2430                                      salo    1
2431                                  sanjorge    1
2432                                       sap    1
2433                                   sarcasm    1
2434                               saulalinsky    1
2435                           scaredmetodeath    1
2436                                      scen    1
2437              scenedancingdolphinabilapost    1
2438                             schadenfreude    1
2439                                  schedule    1
2440                                 scientist    1
2441                                     score    1
2442                                 secondary    1
2443                                  security    1
2444                                      sell    1
2445                                  serivice    1
2446                                    serves    1
2447                                   servide    1
2448                                    shaken    1
2449                                   sharing    1
2450                                      shed    1
2451                                   shifted    1
2452                                  shocking    1
2453                                    shoddy    1
2454                                      shoe    1
2455                                  shootout    1
2456                                    shoots    1
2457                                   shopped    1
2458                                  shouting    1
2459                                   shoving    1
2460                                   showeed    1
2461                                  shuoldve    1
2462                                     silly    1
2463                                    sittin    1
2464                                 situatoin    1
2465                                     sixof    1
2466                                  slightly    1
2467                       smadcokronos383n28q    1
2468                        smadcokronos383xaw    1
2469                      smadcokronos39cni39s    1
2470                      smadcokronos39vn811v    1
2471                        smadcokronos39xhke    1
2472                       smadcokronosb39x02r    1
2473                                    smells    1
2474                             smokefreehome    1
2475                                  snatched    1
2476                                      snek    1
2477                                     solve    1
2478                                    somany    1
2479                                somanyguns    1
2480                                   sombody    1
2481                                   someday    1
2482                                   somedby    1
2483                                someothing    1
2484                                sometthing    1
2485                               songwriters    1
2486                             sophisticated    1
2487                                   sososad    1
2488                                 soulofshi    1
2489                                     souls    1
2490                                     sound    1
2491                           sowkronosactnow    1
2492           sowkronosclimatechangepollution    1
2493               sowkronosdistafanos30127pdf    1
2494               sowkronosenvironmentalheros    1
2495                     sowkronosgreengardens    1
2496                  sowkronosmakeadifference    1
2497                          sowkronosrecycle    1
2498                                      sows    1
2499                                   spandex    1
2500                              speakingshes    1
2501                                     spill    1
2502                                    spoken    1
2503                                  spooking    1
2504                                    spread    1
2505                                  staffing    1
2506                              stakeholders    1
2507                                 stalemate    1
2508                           startseomething    1
2509                                    starve    1
2510                                    stayed    1
2511                                     stays    1
2512                                steakhouse    1
2513                                  sticking    1
2514                                    stinks    1
2515                                    stopby    1
2516                                  stopping    1
2517                                   stories    1
2518                               strengthens    1
2519                                 stretcher    1
2520                                  stringed    1
2521                                   strings    1
2522                                  stripped    1
2523                                  stronger    1
2524                                 structure    1
2525                                    studio    1
2526                               stuffnstuff    1
2527                                     stupd    1
2528                                stupidfire    1
2529                                  suitable    1
2530                                    suport    1
2531                                    supply    1
2532                                supporters    1
2533                                  supports    1
2534                                  supposed    1
2535                                 suppposed    1
2536                                 surferman    1
2537                                 surprised    1
2538                                surprising    1
2539                               sustability    1
2540                                     swats    1
2541                                      sway    1
2542                                    switch    1
2543                                     swoop    1
2544                                   sylvias    1
2545                                 symbiotic    1
2546                            sympathesizers    1
2547                                   syruppy    1
2548                                    syvlia    1
2549                                     table    1
2550                                      taco    1
2551                                    tahnks    1
2552                                    takign    1
2553                                     talks    1
2554                                      tear    1
2555                                       ten    1
2556                                   tension    1
2557                                      term    1
2558                                     teror    1
2559                                 terrified    1
2560             terroristsnotasdumbastheylook    1
2561                         terrorsympathizer    1
2562                                       tha    1
2563                                     theyd    1
2564                                    theyve    1
2565                                    thinkt    1
2566                                  thinkthe    1
2567                                 thirdfire    1
2568                                   thistag    1
2569                                       tho    1
2570                                      thot    1
2571                                 thousands    1
2572                                 threating    1
2573                                     threw    1
2574                                      thsi    1
2575                                     thtat    1
2576                                 time4wine    1
2577                                    timmys    1
2578                                   tiskele    1
2579                                     toget    1
2580                                      toll    1
2581                                       ton    1
2582                                      tooo    1
2583                                    tootin    1
2584                                    torban    1
2585                                    totaly    1
2586                                     tough    1
2587                                tournament    1
2588                                     toxic    1
2589                                  training    1
2590                            trappedhostage    1
2591                                  trashbin    1
2592                                trenchcoat    1
2593                                   trickle    1
2594                                     tring    1
2595                                  troubles    1
2596                                       ttl    1
2597                                     tturn    1
2598                                      tugs    1
2599                                     tunes    1
2600                                   turnout    1
2601                                     tweet    1
2602                                   tyranny    1
2603                                    unfold    1
2604                                  uniforms    1
2605                                    united    1
2606                     unitslocationdwelling    1
2607                                      univ    1
2608                                unoccupied    1
2609                                   updates    1
2610                                     urban    1
2611                                      vaca    1
2612                                    vacate    1
2613                                vanpartial    1
2614                                    victor    1
2615                                   victore    1
2616                                    viewed    1
2617                  viktorekronosmusicvideos    1
2618                      viktorekronosourfans    1
2619           viktorekronosourlistenerstories    1
2620  viktorekronosourmusicstandupspeakupworld    1
2621                     viktorekronosschedule    1
2622     viktorekronosthebandlomaxgastovathena    1
2623                                  viktores    1
2624                                     visit    1
2625                                    visits    1
2626                                vivasylvia    1
2627                                volunteers    1
2628                                     vomit    1
2629                                 waitforit    1
2630                                    walked    1
2631                                      wall    1
2632                                 wandering    1
2633                                       war    1
2634                                   warming    1
2635                                  warnings    1
2636                                     wasnt    1
2637                                    wasted    1
2638                      watcherkronosfilters    1
2639                                wealthiest    1
2640                          weaponslivetests    1
2641                                   website    1
2642                                     weeks    1
2643                     weightlosskronosphish    1
2644                            welksfurniture    1
2645                                    wheezy    1
2646                                 whereareu    1
2647                                     whiny    1
2648                                 whoaminow    1
2649                                  wildlife    1
2650                                    winner    1
2651                                     wnats    1
2652                                     women    1
2653                                  wonderig    1
2654                             wonderinglime    1
2655                                workathome    1
2656                                     worse    1
2657                       worstblockpartyever    1
2658                        worstfirstdateever    1
2659                                   wouldnt    1
2660                                     wraps    1
2661                                writinlazy    1
2662                                        ya    1
2663                                       yay    1
2664                                      yehu    1
2665                                     yells    1
2666                                    yessir    1
2667                                      youd    1
2668                     youtubekronosdt83neln    1
2669                                        yr    1
2670                                      yuck    1
```

</div>


Instead of counting individual word, you can also count words within by newsgroup by using the code chunk below.
<div class="layout-chunk" data-layout="l-body">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='va'>token_by_class</span> <span class='op'>&lt;-</span> <span class='va'>data_classed_token</span> <span class='op'>%&gt;%</span>
  <span class='fu'>count</span><span class='op'>(</span><span class='va'>class</span>, <span class='va'>word</span>, sort <span class='op'>=</span> <span class='cn'>TRUE</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>ungroup</span><span class='op'>(</span><span class='op'>)</span>
</code></pre></div>

</div>



<div class="layout-chunk" data-layout="l-body">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='fu'>wordcloud</span><span class='op'>(</span><span class='va'>data_classed_token</span><span class='op'>$</span><span class='va'>word</span>,
          <span class='co'>#words_by_author$n,</span>
          max.words <span class='op'>=</span> <span class='fl'>200</span><span class='op'>)</span>
</code></pre></div>
<img src="the-sharpe-ratio_files/figure-html5/unnamed-chunk-17-1.png" width="624" />

</div>



<div class="layout-chunk" data-layout="l-body">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='fu'>wordcloud</span><span class='op'>(</span><span class='va'>token_by_class</span><span class='op'>$</span><span class='va'>word</span>,
          <span class='va'>token_by_class</span><span class='op'>$</span><span class='va'>n</span>,
         max.words <span class='op'>=</span> <span class='fl'>200</span><span class='op'>)</span>
</code></pre></div>
<img src="the-sharpe-ratio_files/figure-html5/unnamed-chunk-18-1.png" width="624" />

</div>


The code chunk below uses bind_tf_idf() of tidytext to compute and bind the term frequency, inverse document frequency and ti-idf of a tidy text dataset to the dataset.


<div class="layout-chunk" data-layout="l-body">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='va'>tf_idf</span> <span class='op'>&lt;-</span> <span class='va'>token_by_class</span> <span class='op'>%&gt;%</span>
  <span class='fu'>bind_tf_idf</span><span class='op'>(</span><span class='va'>word</span>,<span class='va'>class</span>, <span class='va'>n</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>arrange</span><span class='op'>(</span><span class='fu'>desc</span><span class='op'>(</span><span class='va'>tf_idf</span><span class='op'>)</span><span class='op'>)</span>
</code></pre></div>

</div>



*Visualising tf-idf as interactive table*


Table below is an interactive table created by using datatable().

<div class="layout-chunk" data-layout="l-body">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='fu'>DT</span><span class='fu'>::</span><span class='fu'><a href='https://rdrr.io/pkg/DT/man/datatable.html'>datatable</a></span><span class='op'>(</span><span class='va'>tf_idf</span>, filter <span class='op'>=</span> <span class='st'>'top'</span><span class='op'>)</span> <span class='op'>%&gt;%</span> 
  <span class='fu'>formatRound</span><span class='op'>(</span>columns <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/base/c.html'>c</a></span><span class='op'>(</span><span class='st'>'tf'</span>, <span class='st'>'idf'</span>, 
                          <span class='st'>'tf_idf'</span><span class='op'>)</span>, 
              digits <span class='op'>=</span> <span class='fl'>3</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>formatStyle</span><span class='op'>(</span><span class='fl'>0</span>, 
              target <span class='op'>=</span> <span class='st'>'row'</span>, 
              lineHeight<span class='op'>=</span><span class='st'>'25%'</span><span class='op'>)</span>
</code></pre></div>
preserveb131eb302f8b1525

</div>




<div class="layout-chunk" data-layout="l-body">
<div class="sourceCode"><pre class="sourceCode r"><code class="sourceCode r"><span class='va'>tf_idf</span> <span class='op'>%&gt;%</span>
  <span class='co'>#filter(str_detect(class, "^sci\\.")) %&gt;%</span>
  <span class='fu'>group_by</span><span class='op'>(</span><span class='va'>class</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>slice_max</span><span class='op'>(</span><span class='va'>tf_idf</span>, 
            n <span class='op'>=</span> <span class='fl'>12</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>ungroup</span><span class='op'>(</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>mutate</span><span class='op'>(</span>word <span class='op'>=</span> <span class='fu'><a href='https://rdrr.io/r/stats/reorder.factor.html'>reorder</a></span><span class='op'>(</span><span class='va'>word</span>, 
                        <span class='va'>tf_idf</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>%&gt;%</span>
  <span class='fu'>ggplot</span><span class='op'>(</span><span class='fu'>aes</span><span class='op'>(</span><span class='va'>tf_idf</span>, 
             <span class='va'>word</span>, 
             fill <span class='op'>=</span> <span class='va'>class</span><span class='op'>)</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>geom_col</span><span class='op'>(</span>show.legend <span class='op'>=</span> <span class='cn'>FALSE</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>facet_wrap</span><span class='op'>(</span><span class='op'>~</span> <span class='va'>class</span>, 
             ncol <span class='op'>=</span> <span class='fl'>2</span>,
             scales <span class='op'>=</span> <span class='st'>"free"</span><span class='op'>)</span> <span class='op'>+</span>
  <span class='fu'>labs</span><span class='op'>(</span>x <span class='op'>=</span> <span class='st'>"tf-idf"</span>, 
       y <span class='op'>=</span> <span class='cn'>NULL</span><span class='op'>)</span>
</code></pre></div>
<img src="the-sharpe-ratio_files/figure-html5/unnamed-chunk-21-1.png" width="624" />

</div>

```{.r .distill-force-highlighting-css}
```
