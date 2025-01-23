---
layout: page
title: "Related Work
"
permalink: /thesis/related-work/
math: true
nav_order: 
parent: Thesis
hidden: true
---

In this section, we want to briefly overview some of the well
established methods for rendering participating media solving the
volumetric rendering equation
<a href="/thesis/problem-statement/#eq:volumetric-rendering-equation" data-reference-type="ref"
data-reference="eq:volumetric-rendering-equation">volumetric-rendering-equation</a>
and the modifications proposed to port them on the GPU. After that, we
will focus on one of those techniques and cover all the work that has
been done on optimizing its implementation on GPU.

## Solving the Volumetric Rendering Equation

We have seen in the previous chapter that our problem can be formulates
as the solution of an integral, the volumetric rendering equation in
<a href="/thesis/problem-statement/#eq:volumetric-rendering-equation" data-reference-type="ref"
data-reference="eq:volumetric-rendering-equation">volumetric-rendering-equation</a>.
There are many possibility to estimate the value of this integral but
all the algorithms, that we will treat here, are stochastic and
<em>unbiased</em>, which means that the expected value of the stochastic
estimator is equal to the value of the integral.

$$
E[ \langle I \rangle] = I
$$

We are not going to cover in the next paragraphs the photon-mapping
approaches, which are particularly good on rendering some difficult
light effects, e.g reflected caustics. A GPU implementation of this type
of algorithms can be found in the article <a href="/thesis/problem-statement/#eq:volumetric-rendering-equation" data-reference-type="ref"
data-reference="eq:volumetric-rendering-equation">(ref)</a>. The article describes GPU
implementations of Progressive Photon Mapping, Stochastic Progressive
Photon Mapping, Progressive Bidirectional Photon Mapping and Vertex
Connection and Merging, which combine photon mapping approach to
bidirectional path tracing, using multiple importance sampling.
Moreover, a comparison between all the algorithm is presented which
shows the advantage of using Path Tracing, for scenes which present no
complex lighting.

### Path Tracing (PT)

Path tracing (developed by ) is one of the most used algorithms in
physically-based rendering and it is also the one that we decided to use
for our GPU implementation. The reason being that GPU benefits from
simple with high arithmetic intensity code. Path tracing is the most
simple of all the presented algorithms even using participating media,
which only requires distance sampling inside the media to extend the
classic surface path tracer. Path tracing can also be coupled with next
event estimation, a technique that allows direct connection of the paths
with the light sources in the scene, by considering direct illumination
from light sources and indirect illumination two separate Monte Carlo
processes. This can really improve the converging time of the method in
some scenes, for example when the light sources are very small. 

In this
work, we decided to not include the next event estimations technique. In
the context of GPU, using this technique means following different
execution paths for threads that need to compute shadow rays and the
others. Moreover, in the context of solid volumetric media this added
contribution will be very small in most of the cases, because the
contribution must be also attenuated by the transmittance function.
Different authors have proposed efficient implementation of path tracing
on GPU in the context of surface rendering, we are going to cover those
methods in the next section after discussing other alternatives.

### Bidirectional Path Tracing (BPT)

Bidirectional Path Tracing ( developed by ) is a more sophisticated
approach combining the advantages of path tracing with the dual method,
which starts from the light source and light tracing, using multiple
importance sampling for the final Monte Carlo estimator. 

In this technique, each approach is weighted and summed together and it can
yield to unbiased estimations of the integral. The method can be easily
generalized for participating media, but special care should be taken
for the weights, which can be only approximated. Many authors have
ported this algorithm to GPU without considering volumetric media. The
first implementation was introduced by , then in a successive work
improved it. The implementation increment the SIMD efficiency compared
to a naive implementation. During a first phase of path generation,
active threads are compacted together (we will add more details about
this method in the chapter
<a href="#chap:method" data-reference-type="ref"
data-reference="chap:method">4</a>). During the connection phase, each
GPU thread is used to evaluate every bidirectional connection. The
method requires a higher memory consumption in comparison to the Path
Tracing implementation. Moreover, in the work it is showed that even if
the CPU implementation performs better than the classic Path Tracer,
this is not the same for GPU, where the path tracer outperforms BPT in
most of the scenes. In a successive work proposed Light Vertex Cache
BPT; the key idea of this algorithm is that only a certain number of
randomly chosen vertex are connected in the connection phase.

This allows to store all the vertex in a single global cache of size given,
by the average path length and to simplify the algorithm. The
implementation offers a considerable speed up compared to the previous
one and shows better performance than Path Tracing in scenes with
complex lighting.

### Metropolis Light Transport (MLT)

Metropolis Light Transport (MLT) is a method which leverages Metropolis
sampling to sample the path space. This allows to generate paths
according to the type of function we are integrating (that, in our case,
is the radiance coming from the rendered scene):

$$
p(\overline{x}) = \frac{f(\overline{x})}{b}
$$

where b is equal to the value that we want to estimate
$b= \int_\mathcal{P}f(\overline{y}d\sigma(\overline{y})$. The value of
the latter can be estimated rendering the scene at low number of
samples. One of the samples is then used as first state with a
probability equal to $\frac{f}{p}$. The next states are then generated
using the tentative transition function
$t(\overline{x} \to \overline{y})$, which proposes a modification of
the previous path. The modification is accepted or rejected based on the
acceptance function. The method is highly affected by the proposal
strategy and has also been explored in the participating media context.


The implementation is based on identifying a path by the random number
used to create it. The integral over the path space is then transformed
to an integral over the space of those numbers (the primary sample
space). New paths in this method are created by perturbing the sample.
However, both the algorithms have the risks to continue the computation
over and over on the same area of the scene. To avoid that to happen,
new random samples have to be generated after random intervals of time.
Also this algorithm was addressed by Dietger.
In its implementation,the algorithm runs many MLT samplers in parallel and mutates the random
numbers at the base of the method during the path generation phase.
Moreover, the implementation builds on top of the BPT implementation
previously described and inherits many of the improvements shown in that
case. Also this method requires a high memory consumption respect to the
Path Tracer implementation and the improvement achieved, using this
technique, depends strongly on the type of scene. 

Furthermore, also in this case it is shown that, even if the CPU implementation performs
better than Path Tracing, this is not true for what concerns GPU.

## Efficient Implementations of Path Tracing on GPU

Path tracing is the core at the base of all the algorithms previously
described. Consequently, an efficient implementation of this building
block can have a positive effect also on the method previously
described. We are not going to cover in this section the work done to
reach high GPU performances ray casting large scenes. The reason being
that our objective is to focus on a single volume, which requires only
the intersection with a simple bounding box. The reader interested can
look into the work done by , which describes the use of Spatial Bounding
Volume Hierarchies (SBVH) on GPU. For the same reason we decided to not
use any ray-shooting solutions, as NVIDIA’s OptiX ( more informations in
the work of ) or other frameworks, but rather to construct a new GPU
solution from the ground up. In the introduction, we have talked about
the challenges on porting a CPU algorithm to GPU. To build an efficient
version of Path Tracing on GPU, it is essential to address full
utilization of the available processing power.

 However, Path Tracing is
a stochastic method, where each path traced can follow a different
direction. The termination of some paths, before others, can reduce the
full utilization of GPU. Nova’ak et al. have addressed this problem by
regenerating terminated paths. This method uses persistent threads,
which access a global pool of paths when the paths that are computed
terminate. The method is improved by , who considered also compacting
the active paths before regenerating new paths for the idle threads (
see chapter <a href="#chap:method-overview" data-reference-type="ref"
data-reference="chap:method-overview">method-overview</a> for
more explanation on this). 

A single kernel version of the described
algorithm is given by , who proposed a tiled compaction rather than a
global device compaction similar to the method we will show. However,
the algorithm showed in the article does not give the expected results
due to the hardware limitation in the number of registers available for
a single kernel. propose a different type of path tracing which is best
suited for scenes with complex materials requiring a lot of computation
and thus leading to thread divergence. The methods use multiple
lightweight kernels calls which allow to utilize all the resources
present on the GPU for the specific task. The paths are stored in a
large pool of paths, which are sorted in chunks based on the specialized
kernel they have to be used on. proposed a single kernel version of path
tracing with regeneration, where regeneration is done singularly by each
thread. However, in this implementation the regeneration is causing code
divergence, which means that while the path is regenerating the other
threads in the SIMD group (called warp in CUDA) have to wait before
continuing. 

In this work, a different single kernel regeneration is used
that prevents this divergence by regenerating the paths only when all
the paths inside the warp have finished. provide also a performance
comparison between the available algorithms with the result of having
best comparable performances in the regeneration path tracer method with
a single kernel and the streaming path tracer with multiple kernels.
proposed a different implementation of the path regeneration technique
with the objective of lower self cost and avoiding moving ray data on
different memory locations. They use tile-based work distribution and
regenerate entire tiles instead of single threads only if the number of
tiles to regenerate is greater than a certain threshold. Moreover, they
apply thread compaction using shared memory for the threads associated
to a tile..

<div class="page-nav">
  <a href="/thesis/background/" class="next-page">Next chapter →</a>
</div>
