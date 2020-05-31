---
layout: post
title: Analyzing Snare and tanner data
date: 2020-05-31 4:16 PM
---

<style type="text/css">

    body .gist .gist-file {
        margin-bottom: 0;
        border: 1px dashed #adb5bd;
        border-radius: 0;
    }

    body .gist .gist-data {
        border-bottom: none;
        border-radius: 0;
        background-color: #c8cbcf;
    }

    body .gist .blob-wrapper {
        border-radius: 0;
    }

    body .gist .highlight {
        background-color: transparent;
        font-size: 14px;
    }

    body .gist .highlight td {
        padding: 5px 15px !important;
        line-height: 1;
        font-family: inherit;
        font-size: inherit;
    }

    body .gist tr:first-child td {
        padding-top: 15px !important;
    }

    body .gist tr:last-child td {
        padding-bottom: 15px !important;
    }

    body .gist .blob-num {
        color: #ced4da;
        background-color: #495057;
        pointer-events: none;
    }

    body .gist .gist-meta {
        display: none;
    }
    
</style>

I had [Snare](https://github.com/mushorg/snare) & [Tanner](https://github.com/mushorg/tanner) running on two different digital ocean's droplets. I just wanted to see what juicy data I can get using this honeypot. How I deployed it is a different story and I'll write a different blog post for that. In this post, I just wanted to share small code I wrote as well as some graphs I was able to generate using them.

## Downloading data using Tanner API

Tanner doesn't support any inbuilt export option(yet) but it has an API so I wrote a small python script to download data from my tanner API and store it in a JSON file.

{% gist 47cf9dca249818d2e72833c9622efe7d %}


## Visualizing data

Ok, so now I had all the data from the tanner server in a file named `tanner_9f7d7dd3-ac6b-468b-8cee-ce3e352eff6e.json`. Then I used a library like pandas, matplotlib to generate some graphs. You can see the gist below that contains the code that I used to get some graphs. And before you scroll down just want to clear one thing, I am not an expert all this visualization with matplotlib and pandas(like [@Ashish](https://github.com/Ashish-012)) so it's possible that my method of cleaning the data, making data frames looks very odd to you. If you have any suggestions then please let me know.

{% gist 8b97889bb7893320b57579c70d7fa08e %}


### Graphs

* Simple line graph showing number of requests made from various countries.

![Line graph](images/graphs/line.png)

* Bar graphs on the same data set

![](images/graphs/bar.png)

* Bar graph on sorted data

![](images/graphs/sorted_bar.png)

* Pie chart on country vs number of requests

![](images/graphs/pie.png)

* Scattered graphs

![](images/graphs/scatter.png)

__Sorted data set__

![](images/graphs/sorted_scattered.png)

* Types of attack 

![](images/graphs/attack_type.png)


***

That's all, I think I learned a few new things today. oh and one of the cool things that I learned today was  `how to include gist in your blog posts`. If you are using Jekyll for your blog then check out [jekyll-gist](https://github.com/jekyll/jekyll-gist). This plugin helps you embed gist in your GitHub pages easily.

***

Thanks for reading and if you have any suggestions about the code or anything else then let me know.
