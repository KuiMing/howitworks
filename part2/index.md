---
title       : rCharts
author      : Ramnath Vaidyanathan
framework   : minimal
highlighter : prettify
hitheme     : twitter-bootstrap
mode        : selfcontained
github      : {user: rcharts, repo: howitworks, branch: gh-pages}
widgets     : [disqus, ganalytics]
assets:
  css: 
    - "http://fonts.googleapis.com/css?family=PT+Sans"
    - "../assets/css/app.css"
    - "../assets/css/gh-buttons.css"
    - "http://odyniec.net/articles/turning-lists-into-trees/css/tree.css"
url: {lib: ../libraries}
---

# How rCharts Works?
#### Part 2

<!-- AddThis Smart Layers BEGIN -->
<!-- Go to http://www.addthis.com/get/smart-layers to customize -->
<script type="text/javascript" src="//s7.addthis.com/js/300/addthis_widget.js#pubid=ra-4fdfcfd4773d48d3"></script>
<script type="text/javascript">
  addthis.layers({
    'theme' : 'transparent',
    'share' : {
      'position' : 'left',
      'numPreferredServices' : 5
    }   
  });
</script>
<!-- AddThis Smart Layers END -->




<a href="http://prose.io/#{{page.github.user}}/{{page.github.repo}}/edit/gh-pages/index.Rmd" class="button icon edit">Edit Page</a>

This is a multi-part series on how rCharts works. My objective is to show you how easy it is to integrate a javascript visualization library into rCharts, and take advantage of a single-unified interface and other functionalities.

In the [first part](http://rcharts.io/howitworks), I showed you how to integrate [uvCharts](http://imaginea.github.io/uvCharts/) with rCharts, and create a simple barchart, straight from R. In this second part, I will take you through the steps required to expose the full API of uvCharts, so that all customization features are directly accessible from R.

## uvCharts

From the [documentation page](http://imaginea.github.io/uvCharts/documentation.html) of uvCharts, we observe that all charts are specified using a single function that accepts three arguments.

```js
var chartObject = uv.chart(chartType, graphDefinition, optionalConfiguration)
```


<iframe
  style="width: 100%; height: 650px"
  src="http://jsfiddle.net/ramnathv/w669V/2/embedded/js,result">
</iframe>

`chartType` is a string that specifies the type of the chart (e.g. BarChart). `graphDefinition` is a json object consisting of `categories` and `dataset`, that specify the different groups and the dataset (name-value pairs) corresponding to each group. `optionalConfiguration` is a json object consisting of different configuration elements (e.g. `meta`, `margin` etc), each of which consists of key-value pairs.

Our objective now is to translate this javascript into a mustache layout and populate it with data and parameters from R

--- .RAW

## Layout

The mustache layout shown below is a modified version of what we developed in Part 1. The basic idea is to bundle the entire payload into a single json object `chartParams` (which is returned by default), and recover `graphdef`, `config` and `type` from it. Just as a reminder `foo.bar` in javascript is equivalent to `foo$bar` in R.

```html
<script>
  var chartParams = {{{ chartParams }}},
      graphdef = chartParams.graphdef,
      config = chartParams.config
  
  config.meta.position = '#{{chartId}}'
  
  var chart = uv.chart(chartParams.type, graphdef, config);
</script>
```

---

## Reference Class

`rCharts` uses an object-oriented approach called reference classes, that allows common functionality to be bundled into a parent class, and library specific functionality to be defined or overridden in the child class.

In Part 1 of this series, we had directly used the `rCharts` base class and the generic `set` and `setLib` methods to integrate `uvCharts`. However, for greater flexibility, it is better to implement `uvCharts` as a separate class.

```r
Uvcharts <- setRefClass('Uvcharts', contains = 'rCharts', 
  methods = list(
    initialize = function(){
      callSuper()
      params$config <<- list(meta = list(position = "#uv-div"))
    },
    config = function(..., replace = F){
      params$config <<- setSpec(params$config, ..., replace = replace)
    }
  )
)
```

We define the `Uvcharts` class to be a child class that inherits from the parent `rCharts` class. The `initialize` method executes the default initialization code for all `rCharts` classes using `callSuper()`, and then sets the `meta` parameter of `config` to '#uv-div'.

The `config` method allows different configuration elements to be specified easily without having to resort to a convoluted sequence of nested lists. Note that these methods allows users to build a visualization in steps, rather than having to specify everything in a single function call.

Our final step is to wrap all this into a `uPlot` function. To keep things flexible, we use the `...` parameter to allow a user to specify configuration related parameters directly using `uPlot`.


```r
uPlot <- function(x, y, data, group = NULL, type, ...){
  dataset = make_dataset(x = x, y = y, data = data, group = group)
  u1 <- Uvcharts$new()
  u1$set(graphdef = list(
    categories = names(dataset),
    dataset = dataset
  ))
  u1$set(type = type)
  if (length(list(...)) > 0){
   u1$config(...)
  }
  return(u1)
}
```


Now, it is time to recreate the chart


```r
hair_eye_male <- subset(as.data.frame(HairEyeColor), Sex == "Male")
dataset = make_dataset('Hair', 'Freq', hair_eye_male, group = 'Eye')
u1 <- uPlot("Hair", "Freq", 
  data = hair_eye_male, 
  group = "Eye",
  type = 'StackedBar'
)
u1$config(meta = list(
  caption = "Hair vs. Eye Colors",
  vlabel  = "Hair Color"
))
u1$config(graph = list(
  palette = "Olive"  
))
u1
```

<iframe src=assets/fig/myplot.html seamless></iframe>





### Acknowledgements

I would like to thank [TimelyPortfolio](http://github.com/timelyportfolio) for his valuable suggestions and comments, which helped improve this post.


<div id='disqus_thread'></div>
