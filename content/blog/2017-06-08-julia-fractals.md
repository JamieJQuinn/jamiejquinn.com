---
title: "Mr. Julia"
date: 2017-06-08
tags:
    - code
    - mathematics
    - art
    - javascript
    - webgl
description: "Creating art from the mathematics of Julia fractals."
image: julia-fractals/cover.png
---

Going on my theme of wonderfully fractal images, I wrote a little simulation to introduce myself to webGL. Go have a wee play about with it [here](http://jamiejquinn.com/webGL-Julia-Fractal/).

### The Maths
You can find lots of information about Julia fractals all around the web so I won't go into much detail at all here. All I'll say is that the fractals, named for Gaston Julia, come about by iterating a complex number through the formula
$$z_{n+1} = z_n^2 + c,$$
where \(c\) is some complex number. There's no need to take the square but it produces some very nice images without going into the complexity of trying to take powers of complex numbers.

The pictures are produced by taking each pixel, turning its location (\(x, y\)) into a location in the complex plane \(z = x+ iy\) and using that as the starting point for the iteration. The location of the mouse on the screen gives the value of \(c\) and changes the entire nature of the fractal. Take a look at some of the fascinating patterns you can get below.

### Examples

![](/assets/img/julia-fractals/1.png)

![](/assets/img/julia-fractals/3.png)

![](/assets/img/julia-fractals/4.png)

![](/assets/img/julia-fractals/5.png)
