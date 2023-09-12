---
layout: post
title: "Comprehending Boxplots in Meteorology"
subtitle: "An Approach to Implement Boxplot Analysis, Especially in Meteorology"
background: '/img/posts/understanding-boxplot/picture2.png'
---

## Background
In this transcript, I’ll explain about the boxplot and how to use in interpret the meteorological data. Some transcript will write in "Bahasa" for educational purpose(to make this article more easy to understand).

Reference:
```
https://www.sciencedirect.com/book/9780128158234/statistical-methods-in-the-atmospheric-sciences
```

## Definition and Basic Description
Boxplot a.k.a box-whisker plot is a simple plot of five sample quantiles: the minimum, lower quartile (q1), median (q2), upper quartile (q3), and the maximum. Using these five numbers, we can make a quick sketch about the distribution of underlying data.

![picture1 image](/img/posts/understanding-boxplot/picture1.png)

The picture above shows the simple boxplot. From the plot, there are some points that we can get, among them are:

1. Maximum temperature is 54 C (>50) which is displayed by edge of upper whisker. Minimum temperature is -10 C which is displayed by edge of lower whisker.
2. its upper quartile is 32, lower quartile is 26 C, and median is 30 C.
q1 and q3 displayed by box, and median is displayed by line in between q1 and q3.
3. from that plot, we can say that main/majority of temperature data have value between 32 C (q3) – 26 C (q1), and in the midle of the data is median, 30 C.

Now, this plot have a problem or shortcoming. Because, from this plot, the distribution of data that have value between q1 and minimum, or q3 and maximum, its not well explained. 

I mean, if we have data, let’s say 45 C. now, of course this value is between q3 and maximum, but how we will know, is this value is like upnormal/anomaly/ or in bahasa “pecicilan” data or not. Because the maximum or minimum data sometimes happen when there are anomaly in there, and this type of data can interfere the process of interpretation, which is also direct to poor interpretation.

Nah, the creator of boxplot, also think about that, so he came with this method, that is “Schematics Plots”.

In simple term, schematics plots is the advance method of boxplot. There are some modification, that answer the question from last paragraph.

![picture2 image](/img/posts/understanding-boxplot/picture2.png)


The schematic plot is identical to the boxplot, except that extreme points deemed to be sufficiently unusual/anomaly are plotted individually. Just how extreme is sufficiently unusual depends on the variability of the data in the central part of the sample, as reflected by the IQR. A given extreme value is regarded as being less unusual if the two quartiles are far apart (i.e., if the IQR is large), and more unusual if the two quartiles are near each other (the IQR is small).

Line that divide less-more unusual the data is called by “fences”. There are 4 fences.
1. upper outer fence
2. upper inner fence
3. lower inner fence
4. lower outer fence

Which is, the value of the fence can be found by formula :

![picture3 image](/img/posts/understanding-boxplot/picture3.png)

The data which exceeds the inner fence called by outsider, and the data which exceeds the outter fence called by far-out.

Now, how we can interpret plot that have more than one boxplot. From reference, said that one important use of this plot is simultaneous graphical comparison of several batches of data.
More detail in page 33 of the reference.

Additional reference "why mean isn’t neccesary to display in boxplot":
```
https://stats.stackexchange.com/questions/269644/why-do-means-appear-outside-the-boxplot
```

## Interpretation Boxplot in Meteorology Field
In interpretation the boxplot in meteorology field, there are few notes that we must consider, that is:
In interpretation the boxplot in meteorology field, there are few notes that we must consider, that is:
1. Paramater value that importans for interpret is IQR, Q1, Q3, Median, and some cases need average*
2. Step one is look for the data distribution. is the data well distributed?. Next, compare with the other data from 
compare to the other data from the same class, is the data more distributed or less distributed. Finally, specifying the range of the data value.
3. Median data is a key for specifying, did the data change, become more or less.
4. Average, in some cases, we should’nt use, because the bias value is very bigger, along the more outsider exist. So, if you wanna use this paramter, make sure the data don’t have a lot of outsider.

The paper that i take for an example intepretation is:
```
https://journal.unesa.ac.id/index.php/jpfa/article/view/22594
```

Example :
in boxplot below, contain information about LFC in Natuna Island when Cold Surge active. If we look from the IQR and Q1 Q3, the data in 12 UTC more distributed compare with the data in 00 UTC, either CS or NS. CS00 data is well distributed (median data (line) is fit on the middle box). Majority of the data distributed on 940 – 960 hPa, with some outsider on 890 – 930 hPa. Compare with NS00, LFC when CS exist IN 00 UTC have decreased in value (Median CS00 = 955 hPa, NS00 = 968 hPa). Although the average NS00 is smaller, but it is can’t represented well because a lot of outsider exist and have an extrem small values (range 840 – 925 hPa) and predispose/affect the average data.
[continue with CS12 NS12, final step is make conclusion, Done]

![picture4 image](/img/posts/understanding-boxplot/picture4.png)

Description of the picture: 
CS00 = BLUE; CS12 = ORANGE;NS00 = GRAY;NS12 =YELLOW
(CS00 : CS event in 00 UTC; NS12 : No CS in 12 UTC)

Thanks for going through this tutorial. If you have any inquiries or require additional help, please don't hesitate to reach out.
