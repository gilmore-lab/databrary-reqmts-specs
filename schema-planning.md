---
title: "schema-planning"
author: "Rick Gilmore"
date: "2019-10-28 13:11:23"
output:
  html_document:
    keep_md: true
    theme: lumen
    toc: yes
    toc_depth: 3
    toc_float: yes
    code_folding: hide
---



# Purpose

This document contains some work related to planning for the Databrary 2.0 schema.

# Resources

- Ognyanova, K. (2019) Network visualization with R. Retrieved from www.kateto.net/network-visualization. <http://kateto.net/network-visualization>
- ggnet: <https://briatte.github.io/ggnet/>

## Understanding the network data format

Ognyanova 2019 contains several datasets.
I've downloaded them, and will now import them to understand the basic network data format.


```r
nodes <- read.csv("sunbelt2019/Data files/Dataset1-Media-Example-NODES.csv", header=T, as.is=T)
links <- read.csv("sunbelt2019/Data files/Dataset1-Media-Example-EDGES.csv", header=T, as.is=T)

head(nodes)
```

```
##    id               media media.type type.label audience.size
## 1 s01            NY Times          1  Newspaper            20
## 2 s02     Washington Post          1  Newspaper            25
## 3 s03 Wall Street Journal          1  Newspaper            30
## 4 s04           USA Today          1  Newspaper            32
## 5 s05            LA Times          1  Newspaper            20
## 6 s06       New York Post          1  Newspaper            50
```

```r
head(links)
```

```
##   from  to      type weight
## 1  s01 s02 hyperlink     22
## 2  s01 s03 hyperlink     22
## 3  s01 s04 hyperlink     21
## 4  s01 s15   mention     20
## 5  s02 s01 hyperlink     23
## 6  s02 s03 hyperlink     21
```

# Data collection scenarios

## Separate videos for each participant

Let's visualize a scenario where there is a single video for each participant.


```r
grViz("digraph {
	  style=filled
		color=lightgrey
		node [style=filled, color=lightblue]
    video_1 -> {person_1, datavyu_1}
		video_2 -> {person_2, datavyu_2}
    video_3 -> {person_3, datavyu_3}
    videos -> {video_1, video_2, video_3}
    code_1 -> {datavyu_1, datavyu_2, datavyu_3}
    code_2 -> {datavyu_1, datavyu_2, datavyu_3}
    codes -> {code_1, code_2}
    persons -> {person_1, person_2, person_3}
		}")
```

<!--html_preserve--><div id="htmlwidget-370eb29d60a95c2e2244" style="width:672px;height:480px;" class="grViz html-widget"></div>
<script type="application/json" data-for="htmlwidget-370eb29d60a95c2e2244">{"x":{"diagram":"digraph {\n\t  style=filled\n\t\tcolor=lightgrey\n\t\tnode [style=filled, color=lightblue]\n    video_1 -> {person_1, datavyu_1}\n\t\tvideo_2 -> {person_2, datavyu_2}\n    video_3 -> {person_3, datavyu_3}\n    videos -> {video_1, video_2, video_3}\n    code_1 -> {datavyu_1, datavyu_2, datavyu_3}\n    code_2 -> {datavyu_1, datavyu_2, datavyu_3}\n    codes -> {code_1, code_2}\n    persons -> {person_1, person_2, person_3}\n\t\t}","config":{"engine":"dot","options":null}},"evals":[],"jsHooks":[]}</script><!--/html_preserve-->

The `igraph` package seems to be more flexible, and it is still supported.
So, let's try a version using it.


```r
library(igraph)
```

```
## 
## Attaching package: 'igraph'
```

```
## The following objects are masked from 'package:stats':
## 
##     decompose, spectrum
```

```
## The following object is masked from 'package:base':
## 
##     union
```

```r
el <- rbind(c("vid_1","per_1"), c("vid_2","per_2"), c("vid_3","per_3"), c("vids", "vid_1"), c("vids", "vid_2"), c("vids", "vid_3"))
g0 <- graph_from_edgelist(el)
#g0$layout <- layout_in_circle
# l <- layout_with_fr(g0)
# l <- layout_on_grid(g0)
l <- layout_nicely(g0)
plot(g0, vertex.size=40, vertex.color="gray", layout = l)
```

![](schema-planning_files/figure-html/unnamed-chunk-3-1.png)<!-- -->

Ok, so now let's add individual Datavyu and demographic data files to this.


```r
el2 <- rbind(el, c("vid_1", "dv_1"), c("vid_2", "dv_2"), c("vid_3", "dv_3"))
g2 <- graph_from_edgelist(el2)
l <- layout_nicely(g2)
plot(g2, vertex.size=40, vertex.color="gray", layout = l)
```

![](schema-planning_files/figure-html/unnamed-chunk-4-1.png)<!-- -->

Let's now add some codes that are contained within the Datavyu files.


```r
el3 <- rbind(el2, c("dv_1", "c_1"), c("dv_2", "c_1"), c("dv_3", "c_1"))
g3 <- graph_from_edgelist(el3)
l <- layout_nicely(g3)
plot(g3, vertex.size=30, vertex.color="gray", layout = l)
```

![](schema-planning_files/figure-html/unnamed-chunk-5-1.png)<!-- -->

Let's next try to do this using a data frame, and adding demographic data `demo_*`.


```r
edges <- data.frame(from = c("vid_1", "vid_2", "vid_3", "vid_1", "vid_2", "vid_3", "per_1", "per_2", "per_3"), to=c("per_1", "per_2", "per_3", "dv_1", "dv_2", "dv_3", "demo_1", "demo_2", "demo_3"), weight= c(1, 1, 1, 1, 1, 1, 1, 1, 1))
g <- graph_from_data_frame(edges, directed = FALSE)
plot(g, vertex.size=30, vertex.color="gray")
```

![](schema-planning_files/figure-html/unnamed-chunk-6-1.png)<!-- -->

A project links these nodes together.
A project could consist of a single 'wave'.


```r
edges <- data.frame(from = c("vid_1", "vid_2", "vid_3", "vid_1", "vid_2", "vid_3", "per_1", "per_2", "per_3", "proj", "proj", "proj"), to=c("per_1", "per_2", "per_3", "dv_1", "dv_2", "dv_3", "demo_1", "demo_2", "demo_3", "vid_1", "vid_2", "vid_3"), weight= c(1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1))
g <- graph_from_data_frame(edges, directed = FALSE)
plot(g, vertex.size=30, vertex.color="gray")
```

![](schema-planning_files/figure-html/unnamed-chunk-7-1.png)<!-- -->

Or maybe the more natural organization would be at the person level.


```r
edges <- data.frame(from = c("vid_1", "vid_2", "vid_3", "vid_1", "vid_2", "vid_3", "per_1", "per_2", "per_3", "proj", "proj", "proj"), to=c("per_1", "per_2", "per_3", "dv_1", "dv_2", "dv_3", "demo_1", "demo_2", "demo_3", "per_1", "per_2", "per_3"), weight= c(1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1))
g <- graph_from_data_frame(edges, directed = FALSE)
plot(g, vertex.size=30, vertex.color="gray")
```

![](schema-planning_files/figure-html/unnamed-chunk-8-1.png)<!-- -->

## Sample organization for PLAY

Let's try two sites and have indices (subscripts) representing the site and participant number, respectively.


```r
edges <- data.frame(from = c("vid_11", "vid_12", "vid_21", "vid_11", "vid_21", "vid_12", "per_11", "per_12", "per_21", "site_1", "site_1", "site_2"), to=c("per_11", "per_12", "per_21", "dv_11", "dv_12", "dv_21", "demo_11", "demo_12", "demo_21", "per_11", "per_12", "per_21"), weight= c(1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1))
g <- graph_from_data_frame(edges, directed = FALSE)
plot(g, vertex.size=30, vertex.color="white")
```

![](schema-planning_files/figure-html/unnamed-chunk-9-1.png)<!-- -->

We could also add an age group node.


```r
edges <- data.frame(from = c("vid_11", "vid_12", "vid_21", "vid_11", "vid_21", "vid_12", "per_11", "per_12", "per_21", "site_1", "site_1", "site_2", "per_11", "per_12", "per_21"), to=c("per_11", "per_12", "per_21", "dv_11", "dv_12", "dv_21", "demo_11", "demo_12", "demo_21", "per_11", "per_12", "per_21", "12", "18", "18"), weight= c(1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1))
g <- graph_from_data_frame(edges, directed = FALSE)
plot(g, vertex.size=30, vertex.color="white")
```

![](schema-planning_files/figure-html/unnamed-chunk-10-1.png)<!-- -->

This hard-to-read graph gathers age groups across sites.

Let's try some other scenarios.

## One video, multiple people


```r
edges <- data.frame(from = c("v1", "v1", "v1", "v1", "v1"), to=c("p1", "p2", "p3", "p4", "p5"))
g <- graph_from_data_frame(edges, directed = FALSE)
plot(g, vertex.size=30, vertex.color="white")
```

![](schema-planning_files/figure-html/unnamed-chunk-11-1.png)<!-- -->

Let's now add one Datavyu file linked to the video.


```r
edges <- data.frame(from = c("v1", "v1", "v1", "v1", "v1", "v1"), to=c("p1", "p2", "p3", "p4", "p5", "d1"))
g <- graph_from_data_frame(edges, directed = FALSE)
plot(g, vertex.size=30, vertex.color="white")
```

![](schema-planning_files/figure-html/unnamed-chunk-12-1.png)<!-- -->

And now link the Datavyu file (or really segments) to the people.


```r
edges <- data.frame(from = c("v1", "v1", "v1", "v1", "v1", "v1", "d1", "d1", "d1", "d1", "d1", "s1", "s2", "s3", "s4", "s5"), to=c("p1", "p2", "p3", "p4", "p5", "d1", "s1", "s2", "s3", "s4", "s5", "p1", "p2", "p3", "p4", "p5"))
g <- graph_from_data_frame(edges, directed = FALSE)
plot(g, vertex.size=30, vertex.color="white")
```

![](schema-planning_files/figure-html/unnamed-chunk-13-1.png)<!-- -->

## One project, multiple measures per person

Imagine that there are video recordings, a survey, and EEG all collected as part of Wave 1 (`w1`).


```r
edges <- data.frame(from = c("p1", "p2", "p1", "p2", "p1", "p2", "p1", "p2"), to = c("v1", "v2", "s1", "s2", "e1", "e2", "w1", "w1"))
g <- graph_from_data_frame(edges, directed = FALSE)
plot(g, vertex.size=30, vertex.color="white")
```

![](schema-planning_files/figure-html/unnamed-chunk-14-1.png)<!-- -->

Here, the actual assets might be the survey file, the video, and the EEG file.
There is likely to be data associated with the participants and the wave, but that may not need to be stored as specific asset files.

