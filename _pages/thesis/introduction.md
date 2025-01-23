---
layout: page
title: "Introduction
"
permalink: /thesis/introduction/
math: true
nav_order: 
parent: Thesis
hidden: true
---

## Motivation

In the last decade, graphics processor units have emerged as a cheap and
powerful data parallel computational platforms. The architectural
innovations, in such processors, have transformed the GPU from
fixed-function hardware blocks to programmable units for general purpose
computations (GPGPU). The gap between floating-point capability of GPU
versus CPU is increasing on a year base and other hardware trends
encourages to parallelize the existing serial algorithms.

 The evolution of new programming languages, like CUDA- Compute Unified
Device Architecture- and OpenCL, gives to the programmer more
flexibility in the usage of the GPU processors for general purpose
computing. This processing power can be used today to address many of
the computational problems, not solvable in adequate time with the
previously available resources. The task we are going to focus on is
realistic image synthesis of participating media. Most of the light
effects, visible in nature, can be described by this type of media.
Indeed, those are usually used to describe the property of a generic
material. Realistic rendering of those materials requires physical
simulations of the light interacting with the media. Those interactions
can be thousands or millions based on the type of participating media we
are going to simulate. For this reason, simulation of the light can take
a long time. Efficiently, using GPU to compute in parallel, those light
interactions can improve the time performance to get the final render.

## Possible Applications

Realistic image synthesis of solid object interests a wide range of
areas. In architecture and product visualization can be used to present
a building or a product. In virtual prototyping can be used to predict
the appearance of an object before creating it and modifying it in
according to that. Realistic Real time rendering is used inside many of
the most famous games. Another possible application comes from volume
visualization, where realistic rendering plays a central role in 3D
shape perception. Most of the available implementation, in this sense,
focuses more on performance, rather than perfect light transport
simulation. The original idea for this project comes from the 3D
printing field. In a recent paper   have used volumetric rendering for
accurate prediction of texture colors in 3D printing. This prediction
stage is the biggest performance bottleneck of all the pipeline. The
reason is that print materials have usually high optical density and
simulating the appearance of those materials requires computing
thousands of scattering interactions. 

This work aims to give a
comprehensive view on how to optimize a volumetric path tracing for the
GPU architecture, taking in consideration the advantages and
disadvantages of different design choices. The solutions found can be
used inside a completely GPU based renderer or as a stage of longer
pipeline for rendering.

## Thesis Structure

The thesis has six chapters:

- the first chapter describes the problem that the thesis aims to solve.

- The second chapter shows the work that has already been done on
  solving this problem.

- The third chapter describe the theoretical background necessary for
  the methods that the thesis implements.

- The fourth chapter shows the method implemented and the different
  behavior of each method.

- The fifth chapter describes some implementation details on the
  solution adopted and the software created.

- Finally, the sixth chapter shows some results obtained with the GPU
  methods implemented and a comparison with a standard CPU algorithm.

<div class="page-nav">
  <a href="/thesis/problem-statement/" class="next-page">Next chapter →</a>
</div>
