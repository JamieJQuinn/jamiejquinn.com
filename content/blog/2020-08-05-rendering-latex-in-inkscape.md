---
layout: post
title: "How to Render Latex in Inkscape"
date: "2020-08-05 16:45:05 +0100"
tags:
  - mathematics
  - latex
---

Rendering latex equations or text in the graphics package [Inkscape](https://inkscape.org/) is not an intuitive task, so I've decided to create an extremely short tutorial on doing exactly that using Inkscape's in-built latex extesion. It's not difficult but the menus are quite cluttered and there's a quirk to rendering to equations that's helpful to know.

I've always found the default extension sufficient but if you're looking for a more polished experience, check out [TexText](https://textext.github.io/textext/index.html).

Inkscape does not include its own latex renderer so **you must have latex installed on your local machine. **

**Step 1: Opening the latex dialogue**

With Inkscape open, navigate to the menu `Extensions > Render > Mathematics > Latex (pdflatex)...`.

This is what it looks like on linux with pdflatex installed. The final option may look slightly different on other platforms.

![](/assets/img/inkscape_tutorial/1.jpg)

**Step 2: Rendering a formula**

The text you type into the dialogue that appears is rendered by latex in text-mode, so in order to render an equation, you must add the equation mode dollar signs around it, like `$e=mc^2$`. After hitting apply, your equation should appear as an object you can move and resize.

![step 2](/assets/img/inkscape_tutorial/2.png)

This method is fairly simple and works for my purposes, however if you have more complex formulae or a need to edit latex objects after they've been rendered, I would recommend using TexText instead.
