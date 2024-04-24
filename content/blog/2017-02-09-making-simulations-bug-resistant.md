---
title: 4 Tips on Making Simulations Bug Resistant
date: 2017-05-19
tags:
    - code
    - mathematics
    - debugging
    - hpc
    - simulations
image: making-simulations-bug-resistant/cover.jpg
cover_author: Alain Wong
cover_link: https://unsplash.com/@alainwong?utm_medium=referral&utm_campaign=photographer-credit
---

Having written and used a decent number of simulations over the past few years I've come to understand that preventing bugs in scientific software is just a wee bit different from how it's usually done in more standard software development.

For one thing, many of the simulations come under the category of high performance computing (HPC) simulations, so it can take a long time to build and run a test case, leading to iteration speeds that are *painfully* slow. Another feature of simulations is that sometimes the code isn't that complex in software terms, the programs simply iterates over a large number of equations, which itself comes with a pitfalls. Debugging is also a nightmare, not because something goes terribly wrong and the program crashes, that's not too hard to fix, but what are you meant to do when a graph your simulation produces is just a little bit too tall? Or the fluid you're simulating inexplicably only goes one way? Is the problem in your theory or in your code?

In light of this, here are a few tips that have saved me a lot of bother in the past, and might save you in the future.

## 1. Unit Test
I'm going to get this one out of the way quickly. I know unit testing is an obvious one to most software developers, but I also know that many scientists who program do not consider themselves developers. At any rate, unit testing is kind of hard, especially when common HPC languages like fortran and C aren't exactly the most accommodating of languages for unit testing, and even more so when you consider how many parts of a simulation just aren't unit testable.

Taking that into account my advice is this:

- Set up a unit testing library in your language of choice
- Learn how to write a basic test in it
- Test the living crap out of every bit you can, eg
  - Derivatives and mathematical helper functions
  - System of equation solvers
  - Functions that deal with data in/out

Not only will you be able to show that much of your code works fine, but you'll hopefully write better, simpler functions.

Here's a reasonably simple example from a small 1D fluid simulation that I wrote. It tests the ability of a variable structure, `ModelVariables`, to save and load itself. It's just a simple, effective test, and it came in useful a few times when I made some breaking code change and knew about it *immediately* when this test failed.

``` cpp
TEST_CASE( "ModelVariables save and load correctly", "[variables]" ) {
  // Setup
  int N = 10;
  std::string filePath = "ModelVariableSaveTest.dat";
  const Constants c(0.0001f, 2.0f, N, 10, 2.0f, 3.0f);

  ModelVariables vars(c);
  ModelVariables vars2(c);

  // Load in some test data
  real* data = vars.pressure.get();
  for(int i=0; i<5; ++i) {
    data[i] = 4.0f;
  }
  
  // Save it
  vars.save(filePath.c_str());

  // Load it
  vars2.load(filePath.c_str(), c);

  // Check it
  for(int i=0; i<vars.len(); ++i) {
    CHECK(vars.pressure[i] == vars2.pressure[i]);
    CHECK(vars.density[i] == vars2.density[i]);
    CHECK(vars.velocity[i] == vars2.velocity[i]);
  }
}
```

## 2. Line Equations Up
Say you've got multiple equations that have a symmetry to them, for example if you're working in 3D and you're calculating a similar thing in all three dimensions. Make it easy for your eyes to spot an error and line up your equations.

The following three lines of almost language-agnostic code should all be identical except from any `x` being replaced with `y` and `z`, a reasonably typical situation in a fluid dynamics simulation. There are 3 errors. Can you see them?

``` cpp
bxx = c*bxx - a*bxx + 2*x**2*pbxx
byy = a*byy + 2*x**2*pdyy - c*byy
bzz = 2*z**2 * pbzz + o * bzz - a * bzz 
```

What if I do this?

``` cpp
bxx = -a*bxx + 2*x**2*pbxx + c*bxx 
byy =  a*byy + 2*s**2*pdyy - c*byy
bzz = -a*bzz + 2*z**2*pbzz + o*bzz  
```

Immediately you can see most of the errors clearly! The minus sign is in the wrong place in the `yy` equation, there's an `s` instead of an `y` too (woops, copy and paste is the devils work), there's an `o` instead of a `c` and a `d` instead of a `b`. You'll note that was actually 4 errors, because how often do you know how many errors you have in a piece of code?

Now certainly this is bad code as well because the names don't make immediate sense (see tip #3) and perhaps I should have brackets around the power, and yes the compiler might pick up some of these errors for you, but just look how much easier it is to spot little formatting errors that *could* cause you major headaches when a little time is taken to properly format the code. 

## 3. Name Your Goddamn Variables
I did a maths degree at undergraduate level so I totally understand that when maths is written down it looks better as \\(a = b_n + c\\) than \\(ants = bread_n + corn\\). It's mainly because when you ram a bunch of symbols together like \\(xyz\\) you know that just means \\(x\\), \\(y\\) and \\(z\\) all multiplied together. This falls a bit short when an equation is translated into code. It looks like incomprehensible garbage unless you have a symbol chart in a comment somewhere (which will end up being wrong some time down the line) or you actually have the book or paper with the equation in it in front of you. Plus having one letter names make refactoring or even just searching for a variable *extremely* difficult.

Here's a real life example of when this habit of short variable names lead me to spend nearly an entire week debugging a small 2D fluid simulation.

``` cpp
for(int n=1; n<Nn; ++n) {
  for(int k=1; k<Nn-1; ++k) {
    ...
  }
}
```

In line 2, that's not meant to be an `Nn`. That's meant to be an `Nz`, but it's nearly impossible to see!

A better way to do this is to name those variables something a little more meaningful. The variable `n` is actually counting the number of spectral modes in the code, so why not call `Nn` something better like `n_modes`. It's readable and its not going to get mixed up with `Nz`, especially if `Nz` is named something more readable like `n_points`, now a meaningful name since `k` is actually counting through the grid points. 

Sometimes it can go too far however, especially when transcribing equations into code. Check out the next tip.

## 4. Do as Little as Possible

> Every step between your equations and code is a source of error.

As in many things, when writing simulations there is little scope for error, especially considering the results of the simulation are often untestable or the code takes so long to run that it's just impractical to test against real data. Even then, with complex simulations it's difficult to know whether differences between your simulation results and your experimental data are due to numerical inaccuracy, experimental issues, problems with the theory (but not the code) or straight up bugs in the code.

If you've ever done any kind of mathematical derivation or proof or just some long-winded calculation, you know that you're going to make a mistake somewhere which means in turning any equations or theory into code you want to do it in as few steps as possible. That means, at least for a first version, the equations should be written in code as close to the originals as possible. There are two ways to do this and pros and cons to each.

1. Write using the same (or close to the same) symbols. \\(a  = b \times c\\) becomes `a = b*c`.
2. Write the equation in terms of what it means. \\(a = b \times c\\) becomes `apples = bananas*cantaloupe`.

I tend to use the second style now, although I was very much a fan of the first style for years. The second style feels more natural to program in. Not only is it easier to search for `apples` than `a` but you can spot errors much more easily (see tip #3). The issue with it is that it's slightly more difficult to check against the original equation and formatting can get a little harder.

But you can make your own decision. I'm not your mother.

# The Round Up
In short,

1. Unit test what you can
2. Use good formatting
3. Name your variables well
4. Don't over-complicate your equations

Hopefully I've presented some food for thought here. If you disagree with anything or just have any examples or comments, let me know!

Many of these ideas are inspired by Joel Spolsky's philosophy of making bad code look wrong. Do check out his [blog post](https://www.joelonsoftware.com/2005/05/11/making-wrong-code-look-wrong/) for more ideas and philosophy on writing code that just looks wrong when it is. Code Complete is also a fantastic tome with some empirical evidence on how to improve your code style.
