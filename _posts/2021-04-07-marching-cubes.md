---
layout: post
title:  "The Marching Cubes Algorithm"
date:   2021-04-07 18:30:00 +0200
categories: algorithms 3d
---

{% include mathjs.html %}

# The marching cubes algorithm

## Introduction
During the holydays, since ETH wasn't giving me enough work to do, I decided to learn how to use [Unity][unity].
While deciding the project to implement, I stumbled upon the marching cube algorithms, so now here we are!

## The problem
The main need the Marching Cube Algorithm tries to satisfy, is the need to form a facet approximation to an isosurface through a scalar field, sampled on a rectangle grid. A lot of big words huh? Let's make this simple: imagine to have a lot of points nicely distributed on a section of space. We now decide which of these points we want to be contained in our 3d surface, and which not. There we are! We can now pass our "marching cube" over all the points and make it draw triangles to "exclude" external vertices: each triangle will have his vertices between an excluded cube vertex and an included one (see Figure 1 for a reference).

## The algorithm
The main problem of this algorithm is the amount of data we'd need to compute, but luckily someone has already done its for us!
As everyone knows a cube has 8 vertices. Since each one of those vertices can be either inside the isosurface or outisde, there are $$ 2^{8} = 256 $$ possible combinations of vertices. Luckily (again!) only $$ 14 $$ of those are actually relevant, since all the other are just rotations or mirrorings of these "base" cases.

{% include image.html 
    url="assets\img\cube-combinations.jpg" 
    description="FIgure 1: The fundamental cases of the marching cubes algorithm" 
    width="300"%}



 [unity]: https://unity.com/
