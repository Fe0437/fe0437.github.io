---
layout: page
title: "Conclusion
"
permalink: /thesis/conclusion/
math: true
nav_order: 
parent: Thesis
hidden: true
---

In this work we have implemented and analyzed different methods to
optimize a volumetric path tracer. We have seen that, especially on GPU,
the type of path tracing algorithm that performs best is tightly coupled
with the type of scene we want to render. If the scene is simple,
methods that are simple are performing better. If the scene is complex,
it is better to use a different strategy and concentrate on utilization
and data locality. We have proposed new variants of existing path
regeneration algorithms based on different synchronization levels.
Moreover, we have implemented a new sorting strategy which builds upon
the idea of maintaining the ray coherence of the path tracer not only
for the primary generated paths but also during scattering. This new
method is performing as good as the best streaming algorithm but we
believe that, using bigger and different datasets, it can lead to
further improvements in time performance.

## Future Work

There are many possible future directions to take based on the results
showed in this work. The first step is the use of out-of-core textures
and CUDA managed memory. Indeed, our work is currently limited by the
available memory in the GPU. We have implemented a zero copy texture
which however is poor in performance compared to the normal version. The
usage of more memory for testing will permit to test bigger scenes where
we believe the sorting algorithms can perform better. Moreover, the
implementation can be generalized to handle any type of scenes and
geometry using the state of the art intersection framework provided by .
After that the algorithms should be tested on different type of hardware
which can lead to more details about the different use of the cache
between different GPU architectures. Finally, CUDA is not the only
language which permits GPGPU computing and for this reason the
implementation should be ported and tested also with other GPGPU
programming languages like OpenCL.

<div class="page-nav">
  <a href="/thesis/" class="next-page">Back to index →</a>
</div>
