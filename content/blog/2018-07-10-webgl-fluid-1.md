---
title: "Running Fluid Simulations in WebGL I - Simple Convection"
date: 2018-07-29
tags:
    - mathematics
    - code
    - fluid-dynamics
    - webgl
image: webgl-fluids-1/cover.png
---

Years ago I worked my way through Lorena Barba's [12 steps to Navier-Stokes](http://lorenabarba.com/blog/cfd-python-12-steps-to-navier-stokes/) in Python, but recently I've been getting more and more into GPU programming and figured that it would be an interesting exercise to redo the steps in WebGL. Really when I say GPU programming I mean using general purpose tech like CUDA, but CUDA and WebGL are similar enough (the boilerplate is of course totally different but the idea of writing a kernel to act on many pixels/fluid cells is the same). Plus you get easy, automatic visualisation with WebGL!

If you want to dive straight in, check out the simulation [here](http://jamiejquinn.com/George-GL/01-non-linear-convection/) or the code [here](https://github.com/JamieJQuinn/George-GL).

## Who Is This For?

Just like Barba has for her course, I'm going to assume everyone reading this has a very basic understanding of fluid mechanics, partial differential equations, and numerical methods. By this I mean you should know what a partial derivative is, how you can model fluid behaviour using partial derivatives, and why numerical methods are used to solve fluid equations.

The code will be fairly simple as well, there's not too much software engineering that goes into these small numerical experiments, although I'll say now that the amount of code required to set up even a simple WebGL program is reasonably involved. As a benchmark, as someone who has never used WebGL before, I managed to set all this up in probably around 3 hours. I'll be using a little bit of:

- Javascript - A web programming language. A good basic learning resource is the classic [w3schools](https://www.w3schools.com/jS/default.asp).
- WebGL - A library written for javascript. Check out <https://webgl2fundamentals.org/> for a good tutorial and reference.
- GLSL - This is the shader language used by WebGL, see <https://webgl2fundamentals.org/> again.

By the end of this, you should have the basic understanding of fluid equations, WebGL programming, and numerical methods to create something that looks unfortunately rather boring:

<div style="text-align: center"><video src="/assets/videos/george-gl/simple-convection.webm" width="300" height="300" autoplay loop preload></video></div>

However, don't despair, simulating simple linear convection in 1 dimensions is just the foundation of computational fluid dynamics. We will very quickly and very easily move into more interesting 1D models, and then finally into simulating the fully 2D Navier-Stokes equations, the main set of fluid equations still used today. Beyond that, and beyond Barba's course, we might even end up exploring more complex methods, such as [Stam's classic method](http://www.dgp.toronto.edu/people/stam/reality/Research/pdf/ns.pdf) for quickly and stably simulating real time fluids in real time.

## The Fluid Mechanics

As in Prof. Barba's [first lesson](http://nbviewer.jupyter.org/github/barbagroup/CFDPython/blob/master/lessons/01_Step_1.ipynb) I'll take the 1-dimensional linear convection (or advection) equation, where we have a quantity \(u\) sitting in a fluid that's advected at a speed \(c\):

\(\frac{\partial u}{\partial t} + c\frac{\partial{u}}{\partial x} = 0.\)

For simplicity we'll only deal with this fluid between \(x=0\) and \(x=1\).

The simplest way to transform this equation into a problem solvable by a computer is to **discretise** the derivatives. We start by splitting our 1D space into \(N_x\) points, each separated by \(\Delta x=1/N_x\), and then splitting time in a similar way, stepping forward by a **timestep** \(\Delta t\). The solution \(u\) at a point \(x=i\Delta x\) and a time \(t=n\Delta t\) can then be written as \(u_i^n\). Then, using a **backward difference** formula, the spatial derivative can be approximated as,

$$\frac{\partial{u}}{\partial x} \approx \frac{u_i^n - u_{i-1}^n}{\Delta x},$$

and, using **forward differences**, the time derivative becomes,

$$\frac{\partial{u}}{\partial t} \approx \frac{u_i^{n+1} - u_{i}^{n}}{\Delta t}.$$

It should be fairly obvious why these difference formulae are called forward and backwards differences. Forward differencing calculates derivatives from the \(k\) and \(k+1\) states, while backward differences are calculated from the \(k\) and \(k-1\) states. Central differences are also an option but create strange numerical oscillations when applied to this particular problem, I'll discuss them when dealing with a diffusion problem.

There is an [entire field of mathematics](https://en.wikipedia.org/wiki/Numerical_partial_differential_equations) dedicated to the development and analysis of these approximation of derivatives. This particular technique of combining temporal forward differences with spatial backward differences is a form of the **first-order upwind method**. It's shown to be **stable** when the fluid is moving to the right (and can be modified so the correct spatial difference is chosen based on which direction the fluid is moving) and can be shown to have **first-order accuracy**, meaning the errors that appear will be roughly the same size as \(\Delta t\) and \(\Delta x\). Methods of \(j\)-th order accuracy have errors proportional to \(\Delta x^j\) and/or \(\Delta t^j\) and higher. The upwind method, though not used much in practice due to it not being terribly accurate, is certainly useful for playing about with. After reading the rest of this post, try implementing [the scheme](https://en.wikipedia.org/wiki/Upwind_scheme#First-order_upwind_scheme) for both right-moving and left-moving fluids, or increasing the accuracy using the [second-order upwind scheme](https://en.wikipedia.org/wiki/Upwind_scheme#Second-order_upwind_scheme).

Using these approximations the partial differential equation becomes a finite difference equation,

$$\frac{u_i^{n+1} - u_{i}^{n}}{\Delta t} + c\frac{u_i^n - u_{i-1}^n}{\Delta x} = 0,$$

which can be rearranged to give,

$$u_i^{n+1} = u_i^{n} - c\frac{\Delta t}{\Delta x}(u_i^n - u_{i-1}^n).$$

So, if we know an initial state \(u^0_i\) at every point \(i\), we can work out \(u^1_i\), then \(u^2_i\), and so forth. The upwind scheme is part of a family of methods called **explicit schemes**. This means the future state \(u_i^{n+1}\) can be written **explicitly** in terms of current or past states \(u_i^{n,n-1,n-2,...}\). The alternative is to incorporate future states into the calculation in a more intricate way, producing an **implicit** scheme. These are usually much more stable, often more accurate, but certainly a little harder to understand and code, and can really only be used on **linear** problems. I'll discuss implicit schemes a little more in future posts!

Now it should be quite clear that we've finally found an equation suitable for a computer to solve, an equation that ultimately helps us simulate how a 1D fluid will behave, as long as we know the initial state \(u^0_i\) and the boundary behaviour. It should be said our original PDE does actually have a known **analytical** solution, that is a fully accurate solution we can write as a function \(u(x, t)\) for any time \(t\), at any point \(x\). It can be found using the [method of characteristics](https://services.math.duke.edu/education/joma/sarra/sarra2.html). However, this is a particularly simple case and as we add complexity to the fluid equation, it becomes usually impossible to find anything other than a **numerical** solution through the application of this kind of numerical method.

## The Code

Now technically, I could take that final equation, give myself a starting state, and manually, by hand, work out every little calculation until I run out of time, food or any kind of semblance of sanity, but I won't because computers exists. I'm not going to go through the code in detail because, frankly, it's not very interesting and mostly copied from the [WebGL2 Fundamentals image processing tutorial](https://webgl2fundamentals.org/webgl/lessons/webgl-image-processing.html) anyway.

We're going to represent the grid of points as a texture in WebGL, with one texture representing the state at time \(n-1\) and another at \(n\), so by rendering between them and applying our finite difference formula we simulate the fluid from one time to the next. Of course a texture is a 2D grid of points, good for later when we finally move to interesting 2D simulations, but our problem right now is only 1D, so I'm only going to deal with the \(x\) direction right now.

The only part that differs greatly from a more typical use of textures is that, because we're trying to run a decently accurate simulation, I've instructed the texture to be created with an internal format of `RGBA32I` which assigns 32 bits per colour channel. Not as precise as the standard of 64 bits in true high performance computing, but good enough for us!

Every part of our simulation then uses shaders that act on these textures. The initial conditions are encoded in a shader, the finite difference formula that advances the simulation is a shader, even the boundary conditions are encoded in a shader that acts only at the edges of the texture.

The object that actually gets rendered is just a square of size \(2\times2\), linked to the shaders via a [vertex array object](https://webgl2fundamentals.org/webgl/lessons/webgl-fundamentals.html). Most of the shaders render this square using two triangles that cover the entire square. The odd shader out is the one encoding the boundary conditions, which simply draws lines around the square.

The full pseudocode looks a little like this: 

1. Set up WebGL context
2. Create rendering surfaces:
    1. Create two textures of simulation size
    2. Link to two framebuffers for rendering
3. Compile and link shaders:
    1. Initial condition shader
    2. Simulation shader
    3. Boundary shader
    4. Screen rendering shader
4. Load initial conditions into a texture
5. Run main loop:
    1. Render using main simulation shader from one texture to another
    2. Render boundary conditions
    3. Render result to screen
    4. Swap textures

The actual code can be found on [github](https://github.com/JamieJQuinn/George-GL/tree/master/01-non-linear-convection). Start reading `main.js` and it should be fairly self-explanatory.

### The Shaders

We have 3 interesting shaders and 1 very boring shader. The screen rendering shader simply copies a texture directly to the screen, allowing us to see what state our simulation is in. It's common to visualise the fluid motion using something that flows around with the fluid like ink. In this example I've set the ink to be the variable \(u\). The only thing the screen shader is used for is interpolating from the colour of the background to the colour of this ink.

Since we're constantly rendering a square with four simple vertices, the vertex shader isn't doing anything particularly interesting, the meat is all in the fragment shaders. The only thing the vertex shader is doing is interpolating the texture coordinate to be used in the fragment shader. 

#### Initial Conditions

The first interesting shader is the initial conditions fragment shader:

``` glsl
#version 300 es
precision mediump float;
in vec2 vTextureCoord;
out vec4 outColour;
void main(void) {
  vec2 pos = vTextureCoord.xy;
  if(pos.x > 0.1 && pos.x < 0.3) {
    outColour = vec4(1.0, 0, 1, 0.0);
  } else {
    outColour = vec4(1.0, 0, 0, 0.0);
  }
}
```

Here we can see how the texture is being used to store the state of the system,

$$(R,G,B) = (c, 0, u),$$

where we're using the red, green, blue and alpha channels to store the \(x\)-velocity or \(c\) (set to \(1\) for simplicity), \(y\)-velocity (\(0\) for now), and the ink level, \(u\).

What this shader does is it sets the \(x\)-velocity to be \(1\) everywhere, and creates a little pocket of ink between \(x=0.1\) and \(x=0.3\), letting the ink level be \(0\) everywhere else. That is, it encodes the function

$$u(x, 0) = \begin{cases}
1, & x \in (0.1, 0.3) \\[2ex]
0, & \text{otherwise}
\end{cases}$$

#### Main Simulation

The main simulation fragment shader is written as

```glsl
#version 300 es

precision highp float;

in vec2 vTextureCoord;

uniform sampler2D uSampler;
uniform float dt;
uniform vec2 dxy;

out vec4 outColour;

void main(void) {
  // Get variables
  vec2 u = texture(uSampler, vTextureCoord).xy;
  float ink = texture(uSampler, vTextureCoord).z;
  float inkmx = texture(uSampler, vec2(vTextureCoord.x - dxy.x, vTextureCoord.y)).z;

  // Perform numerical calculation
  ink = ink - u.x * dt / dxy.x * (ink - inkmx);

  // Output results
  float alpha = texture(uSampler, vTextureCoord).w;
  outColour = vec4(u, ink, alpha);
}
```

Recall the difference formula

$$u_i^{n+1} = u_i^{n} - c\frac{\Delta t}{\Delta x}(u_i^n - u_{i-1}^n).$$

The shader encodes this formula, using the previous values `ink` and `inkmx` (as in ink-minus-x) as \(u^n_i\) and \(u^n_{i-1}\) sampled from the texture storing the previous state. As can be seen from the code, we find the \(i-1\) value by  moving the sample coordinate a single \(dx\) to the left, the distance \(dx\) calculated using `1.0/gl.canvas.width`.

#### Boundary Conditions

The boundary shader is a little more interesting because we're not rendering (or simulating) over the full domain, since the boundary conditions should only affect the pixels around the edges. So instead of rendering to a square, we simply render lines around the domain using `gl.LINE_LOOP`. The shader itself sets the velocity and ink level at the boundary to \(0\) by returning `gl_FragColor = vec4(0,0,0,0)`.

## Conclusion

So, to wrap up, we've gone over the basics of fluid mechanics, written through the language of partial differential equations. We've transformed those equations into finite difference formulae that can be readily calculated by a computer. Finally, we've figured out a way we can write those formulae using javascript and WebGL to simulate the fluid in real time on any modern graphics card, all within the browser!

Next up, we'll explore moving to 2D, and simulating a few more interesting 1D fluid equations.
