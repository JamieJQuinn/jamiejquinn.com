---
title: Regularly Expressional Fractalations
date: 2017-06-01
tags:
    - mathematics
    - art
    - javascript
image: regex-fractals/cover.png
---

There's something about fractals that humans find fascinating. They manage to contain a beautiful impression of infinity despite being not very difficult to create. These fractals have been produced by a very simple recipe:

![quadrant_numbering2](/assets/img/regex-fractals/quadrant_numbering2.png)

1. Split a big square of pixels into 4 quadrants and label them 1 to 4
2. Repeat this process for each of the smaller squares and add the quadrant number to the label
3. Keep cutting the squares into 4 until you've gone as small as you want
4. Now we can use regular expressions to find and mark boxes with certain labels

I wrote a little bit of javascript that creates such fractals from regular expressions involving the digits 1, 2, 3 or 4. Have a play [here](http://jamiejquinn.com/regex-fractals/) and check out some examples below.

### `1`
![1](/assets/img/regex-fractals/1.png)

### `[13][24][13]`
![1](/assets/img/regex-fractals/132413.png)

### `12|21|34|43`
![1](/assets/img/regex-fractals/12or21or34or43.png)

### `13|24`
![1](/assets/img/regex-fractals/13or24.png)

### `13|31|24|42`
![1](/assets/img/regex-fractals/13or31or24or42.png)

### `[34]+2`
![1](/assets/img/regex-fractals/34plus2.png)
