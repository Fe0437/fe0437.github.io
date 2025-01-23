---
layout: page
title: "Optimization Methodology
"
permalink: /thesis/optimization-methodology/
math: true
nav_order: 
parent: Thesis
hidden: true
---

In this section we are going to present an overview of the method used
to optimize the volumetric path tracer. The chapter follows the design
decisions that someone should evaluate when approaching this type of
problem. Different design paths have different advantages and
disadvantages that we are going to cover. 

In
figure <a href="/thesis/optimization-methodology/#fig:design-decisions" data-reference-type="ref"
data-reference="fig:design-decisions">4.1</a> it is possible to see the
exact design decisions tree. At the beginning, we are going to examine
how to use GPU and CPU together and this we lead us to a decision on a
single versus multi kernel approach. After that, we are going to
evaluate how to maximize the utilization of GPU regenerating threads,
which become idles during the computation. Finally, we are going to see
how to maximize the memory throughput and the data locality compacting
together the threads that are still active. 

In
figure <a href="/thesis/optimization-methodology/#fig:scenes" data-reference-type="ref"
data-reference="fig:scenes">4.6</a> there are some scenes that we are
going to use for the tests: two of them from the application field of 3D
printing and the other two from visualization. The volumetric path
tracer tests showed in this section are all running without Russian
Roulette <a href="#russian-roulette" data-reference-type="ref"
data-reference="russian-roulette">russian-roulette</a>.

## Testing Hardware

We will use for testing a MacBook Pro with CPU Intel Core i7 quad-core
and GPU Nvidia GeForce 650M with the following characteristics:

- Kepler architecture, compute capability 3.0

- Total amount of global memory: 512 MB

- Total number of registers available per block: 65536 (equal to
  registers per SM)

- Maximum number of threads per block: 1024

- Total amount of shared memory per block: 49152 bytes

more details about the hardware are provided in the Appendix.

<figure id="fig:design-decisions" class="figure-full">
<embed class="figure-img" src="/assets/images/thesis/tex_charts_design-decisions.png" />
<figcaption>Design decision tree. This tree represent the design
decisions made to optimize a GPU path tracer. In the first level, the
first decision is regarding the use of a single or a multi kernel and
more in general, giving more control to the device or to the host. The
second level contains all the possibility presented to optimize the
utilization of the GPU. Finally, the last level contains the possibility
to decrease data access latencies in a volumetric path
tracer.</figcaption>
</figure>
<figure id="fig:scenes" class="figure-full">

<div style="display: grid; grid-template-columns: 1fr 1fr; gap: 10px;">
  <figure id="fig:ad" class="figure-big">
    <img src="/assets/images/thesis/tex_img_ad.png" />
    <figcaption>ad scene: rendering of a slab with grid resolution
    (320,320,100). The texture "printed" on the surface has a depth equal to
    10 (10% of the total resolution depth) and it is assigned to the albedo
    volume, the rest of the albedo volume is white. The density volume is
    always constant and equal to 100.</figcaption>
  </figure>

  <figure id="fig:cgg-logo" class="figure-big">
    <img src="/assets/images/thesis/tex_img_cgg-logo.png" />
    <figcaption>cgg-logo scene: rendering of a solid unit cube with a grid
    resolution of (100,100,100). The texture "printed" on the surface has a
    depth equal to 10 (10% of the total resolution depth) and it is assigned
    to the albedo volume, the rest of the albedo volume is white. The
    density volume is always constant and equal to 100.</figcaption>
  </figure>

  <figure id="fig:body" class="figure-big">
    <img src="/assets/images/thesis/tex_img_artifix.png" />
    <figcaption>artifix scene : rendering of the publicly available artifix
    Data set (Osirix) which have grid resolution of (128, 87, 128). The data
    has been used to create the density volume, the albedo has been created
    from the density using a transfer function which maps the density from
    gray to red to blue based on the density value. The maximum density
    value is 100.</figcaption>
  </figure>

  <figure id="fig:head" class="figure-big">
    <img src="/assets/images/thesis/tex_img_manix.png" />
    <figcaption>manix scene: rendering of the publicly available Manix Data
    set (Osirix) which have grid resolution of (128, 115, 128). The data has
    been used to create the density volume, the albedo has been created from
    the density using a transfer function which maps the density from gray
    to red to blue based on the density value. The maximum density value is
    100.</figcaption>
  </figure>
</div>
<figcaption>Test scenes. top: scenes for 3D printing application,
bottom: scenes for data visualization.</figcaption>
</figure>

## Host Control or Device Control

### Single Kernel versus Multi Kernel

The topic of using a single "megakernel" versus multiple lighter kernels
has already been widely covered in the literature. A naive single kernel
implementation holds all the algorithm and requires all the resources
associated with all the different parts of it. Nvidia OptiX () utilizes
a similar but more sophisticated approach. In this framework, a
Just-In-Time compiler combines all the different stages of the path
tracer, in the form of PTX (assembly language for NVIDIA GPU), into a
single kernel. The kernel is formed as a state machine where, to
minimize execution divergence, a scheduler selects a single state for an
entire SIMT unit. If a thread is not requiring that state, it remains
idle during that iteration. compared the traditional single kernel
approach to a wavefront formulation, which uses different kernels for
different control flows. This method allows to completely avoid code
divergence, i.e. sorting the paths based on their next interaction,
dividing them into different bins and launching a kernel only for the
paths in the same bin. Moreover, the kernels launched are perfectly
specialized for the task and they can utilize the GPU resources
completely. However, this work is not containing this possibility. There
are different reasons for that:

- Dynamic parallelism and global device synchronization: supported only
  by the newer CUDA GPU (compute capability 3.5 and higher), dynamic
  parallelism gives the ability to a single CUDA thread of launching a
  new kernels with its own configuration. This technique reduces the
  need to transfer execution control and data between host and device
  (). Moreover, the global device synchronization introduced with CUDA
  9.0 (using special cooperative groups) removes the need of launching a
  new kernel only for synchronizing all the threads running on the
  device.

- The type of scenes that we are analyzing in this work contains just
  one object: the volume that we want to render. For this reason there
  are not many different type of interactions. The only two cases are if
  the analyzed point is inside the volume or on the boundary (exactly
  how it is explained in the section
  <a href="#sec:VRE" data-reference-type="ref"
  data-reference="sec:VRE">1.3</a>). However, the wavefront formulation
  is best suited for scenes with many different and complex materials.

Inspired by results showed by others like and the only multi kernel
approaches that we are going to analyze are composed by a maximum of 2
kernels. The table <a href="/thesis/optimization-methodology/#table:sk-vs-mk" data-reference-type="ref"
data-reference="table:sk-vs-mk">4.1</a> shows the difference on using a
naive approach with a single kernel call or multiple kernel calls. The
naiveSK method (naive with a single kernel) consists of a simple
volumetric path tracer implementation where each thread computes only
one path and all the data is stored on local memory (the algorithm can
be found in the work of ). The naiveMK method (naive with multiple
kernels) consists of a simple volumetric path tracer divided in two
kernels. The first kernel is generating the paths and computing the
first intersection with the object, while the second kernel extends the
active paths computing a new intersection with the scene. It is clear
that the single kernel approach wins over the multi kernel one in terms
of speed. However, there are also advantages on using multiple kernel
calls. From a user perspective, using smaller kernels allows the CPU to
take the control of the application and, in case only one GPU is
available in the system, it avoids to freeze the user interface for too
much time.

<div id="table:sk-vs-mk">
<table>
<thead>
<tr><th align="center">Method</th><th align="center">rays/sec</th></tr>
</thead>
<tbody>
<tr><td align="center">naiveSK</td><td align="center">3.91</td></tr>
<tr><td align="center">naiveMK</td><td align="center">1.19</td></tr>
</tbody>
</table>
Single Kernel versus Multi Kernel (naive). The speed of the methods is
analyzed in terms of millions of traced rays per second (rays/sec). The
test has been done using the scene in figure
<a href="/thesis/optimization-methodology/#fig:cgg-logo" data-reference-type="ref"
data-reference="fig:cgg-logo">4.3</a> rendering a 400x400 image for 100
iterations (number of samples for the Monte Carlo estimation)
</div>

<span id="table:sk-vs-mk" label="table:sk-vs-mk"></span>

### Image Tiling

Another possibility for interleaving the control between host and device
is using image tiles. That is, the image is divided into equal tiles and
the device can work on each of them separately at the same time. This
technique allows to lower the memory requirements for running the
software, because we need only to store the size of one tile. This also
means that the rendering is divided into multiple kernel calls. Each of
this kernels have to render a single tile and for this reason is faster.
In this perspective, it is important to not use tiles that are too
small, because this will decrease the utilization of the hardware and
therefore the performance (we will talk more about utilization in the
next section). The consequence is that, in order to obtain the maximum
performance, the size of the tile should depend on the type of hardware.
The tile size is also affecting the specific algorithm in use. Most of
the algorithms that we are showing in this work have different memory
requirement and different behaviors depending on the tile size. However,
from our tests it emerges that the behavior of each algorithm using
different tile sizes is comparable with the use of different number of
samples. Potentially, a tiled image allows also to decrease the time to
transfer the image from the device to the host. In the practical case,
however, the size of the image is usually small and the time to transfer
it is not comparable with the time for computation. For this reason,
also using a second buffer for the image tile (this technique is usually
called Double Buffering) and the CUDA asynchronous memory copy to
overlap computation and image transfers, the performance improvement is
negligible. Table
<a href="/thesis/optimization-methodology/#tab:tiled-rendering" data-reference-type="ref"
data-reference="tab:tiled-rendering">4.2</a> shows the results of using
different image tiles. From this table it is possible to see that
launching multiple kernels on different image tiles is not affecting the
performance as far as the number of tiles is not becoming too high.
Indeed, the computation remains stable until the number of pixel
processed by the algorithm becomes too small, decreasing the utilization
of the device for the kernel call. This result shows also that the real
performance bottleneck on dividing the path tracing task is not
represented by the kernel launch overhead but from the repetitive
loading and storing of the same data.

<div id="tab:tiled-rendering">
<table>
<thead>
</thead>
<tbody>
<tr><td align="center"></td><td align="right">tiling setting</td><td align="right"></td><td align="right"></td><td align="right"></td><td align="right"></td><td align="right"></td></tr>
<tr><td align="center"></td><td align="right">(1,1)</td><td align="right">(2,2)</td><td align="right">(4,4)</td><td align="right">(8,8)</td><td align="right">(32,32)</td><td align="right">(64, 64)</td></tr>
<tr><td align="center">n\. paths per tile</td><td align="right">18432000</td><td align="right">4608000</td><td align="right">1152000</td><td align="right">288000</td><td align="right">18000</td><td align="right">4500</td></tr>
<tr><td align="center">StreamingSK</td><td align="right">61.27</td><td align="right">67.98</td><td align="right">72.72</td><td align="right">76.13</td><td align="right">90.66</td><td align="right">224.04</td></tr>
<tr><td align="center">regenerationSK</td><td align="right">7.99</td><td align="right">10.02</td><td align="right">9.44</td><td align="right">9.48</td><td align="right">34.11</td><td align="right">98.40</td></tr>
<tr><td align="center">naiveSK</td><td align="right">219.47</td><td align="right">223.65</td><td align="right">221.93</td><td align="right">224.59</td><td align="right">224.85</td><td align="right">123.32</td></tr>
</tbody>
</table>
Different image tiling settings used for rendering a 1920x1920 image
with the scene in figure
<a href="/thesis/optimization-methodology/#fig:cgg-logo" data-reference-type="ref"
data-reference="fig:cgg-logo">4.3</a>. In the table the number of paths
processed per tile is compared to the time to render the all image. Note
that, opposed to the naiveSK, the streamingSK kernel launching
configuration does not depend on the number of paths processed but
rather on the threads available on the GPU. For this reason when the
number of paths processed becomes low the naive method improve its
performance while the streamingSK decrease in performance.</div>

<span id="tab:tiled-rendering" label="tab:tiled-rendering"></span>

### Summary

In this chapter we have seen the first design decision that should be
made when approaching this problem: giving the control of the program to
the host or to the device. In the first case, the advantage is to have a
more responsive UI and, if the kernels are well divided into different
tasks, it allows to use efficiently the device resources for the task.
In our case, most of the computation concerns the same simple task: the
interaction with the medium. For this reason, dividing the computation
will not improve the performance. We have also seen another way to
divide the computation by rendering separate image tiles. This allows to
use the same algorithms on a different hardware, whose memory
requirements depend on the image tile size. Moreover, overlapping
communication and computation decreases the latency on getting the
resulting image. The tile size is affecting also the utilization of the
GPU which is upper bounded by the number of paths to compute. Indeed,
the number of paths to compute an image tile is equal to

$$
\text{paths} = \text{tile.width} \cdot \text{tile.height} \cdot \text{iterations}.
$$

While the number of iterations is given by the user, the tile width and
height can be specified according to the hardware used, which allows to
improve the utilization for the specific hardware. In the next chapter
we will see which other factors can decrease the utilization of the GPU
and how to prevent it.

## Maximizing Utilization and Hiding Multiprocessor Latency

The objective of maximizing the utilization of the device is achieved
when the device is always active until the end of the computation. In a
path tracer, the problem is given by the different lengths of the paths
computed. When a path in a thread has terminated, that thread will not
be utilized until the termination of all the others. To overcome this
inefficiency   have proposed to use path regeneration. This technique
consists of using a persistent thread system (more information about
persistent thread programming can be found in the work of ) where new
paths, taken from a global pool of paths, are assigned to idle threads
(the path is therefore regenerated).

Another problem that can affect the utilization on the system is the
number of registers and the shared memory necessary for the kernel. Most
of the time those resources are stored in a fast memory (L1 cache in
devices with compute capability 3 or higher) that is limited. When the
kernel reaches this limit the maximum number of resident blocks in a
multiprocessor is decreased. If the kernel requires more registers than
available on the multiprocessor, then the compiler will attempt to
minimize register usage, or it can be forced to do it, while keeping
them stored in the local memory (register spilling). On the other hand,
if the kernel requires too much shared memory the only solution is to
decrease the number of resident block in a multiprocessor. When the
number of threads resident on the same multiprocessor is high then the
latency of a inactive warps, which cannot perform their next
instruction, can be hidden efficiently. The full utilization in this
case is achieved when the warp scheduler can always issue some
instruction for some warp during the latency period.

### Persistent Thread

The persistent thread style of programming helps a developer to separate
the task to compute from the hardware that is running it. The threads
are active for the entire duration of a kernel and every thread gets new
work from a work queue when it finishes its current task. In our case
this means regenerating the terminated paths and tracing new ones. A new
path can be created by using an identification number like in algorithm
<a href="#alg:regenerate" data-reference-type="ref"
data-reference="alg:regenerate">regenerate</a>. For this reason,
the work queue is represented by one global value that is atomically
incremented during regeneration. In this paragraph, we analyze the
different approach that we may take for regenerations:

- Thread regeneration: a single kernel is launched and new paths are
  assigned to every thread. When the thread becomes inactive the path is
  regenerated.

- Warp regeneration: a single kernel is launched and new paths are
  assigned to every warp. When all the threads in a warp become
  inactive, all the warp is regenerated incrementing the atomic counter
  only once per warp by the size of the warp.

- Block regeneration: a single kernel is launched and new paths are
  assigned to every block. At every path extension, if the path queue is
  not empty, all the threads in a block are synchronized and all the
  inactive ones are regenerated.

- Device regeneration: multiple kernel are launched, one kernel is
  performing regeneration and another one path extension. If the path
  queue is not empty, after each extension all the inactive threads are
  regenerated.

Those regeneration techniques require different types of synchronization
points: the first one is the classic implementation where if a thread
becomes inactive it is immediately regenerated, i.e if only one thread
is regenerating the rest of the threads in the warp, we have to wait for
it before continuing. The second one is using the warp vote functions
(which can be found in the documentation by ) to decide weather to
regenerate or not the warp. In CUDA 9 and on devices with compute
capability higher than 3.0 this can be efficiently done using the warp
shuffle functions, which allows to exchange values between threads in
the same warp, to get the right path for each thread. The third one
requires synchronization of all the threads in a block and it
regenerates all the block all together. This latter is similar to what
is done by . Finally, the last one is regenerating all the processed
paths and requires to synchronize all the devices. We did not include
this last one in our study because we have already tested that the
multi-kernel approach is not favoring our settings. In the
implementation presented the synchronization points are performed after
each path extension (unless the path queue is empty). This can be not
optimal, as the synchronization point for regeneration should be placed
exactly when the performance risks to decrease for not having enough
active threads (in the next paragraph we will see what exactly this
means). However, this is not only hardware dependent but also scene
dependent. For this reason, we did not include this variation in this
work. The table in <a href="/thesis/optimization-methodology/#tab:regeneration" data-reference-type="ref"
data-reference="tab:regeneration">4.6</a> shows the results of the
different methods on our scenes. We can see that the regenerationSK
(regeneration with a single kernel) on a single thread works best on the
scenes with constant density (cgg-logo and ad) for which the warps are
almost always synchronized. In the other two scenes the warp
regeneration is performing better. When the paths start to
differentiate, this method allows to maintain the coherence inside a
warp by restarting all the warp at once.

<div class="algorithm">
<u>function regenerate</u>
<div class="algorithm-line"><strong>Input:</strong> thread: thread to regenerate, paths\_head: global counter for the paths</div>
<div class="algorithm-line"><strong>Output:</strong> thread: regenerated thread</div>
<div class="algorithm-line">path_id = paths_head $++$</div>
<div class="algorithm-line">if path_id $>$ total_paths then</div>
<div class="algorithm-line">  thread.active = false</div>
<div class="algorithm-line">end if</div>
<div class="algorithm-line">pixel = getPixel(path_id)</div>
<div class="algorithm-line">thread.path.ray = cameraRay(pixel)</div>
<div class="algorithm-line">thread.path.throughput = Color(1)</div>
<div class="algorithm-line">thread.active = true</div>
<div class="algorithm-line">return thread</div>
</div>

<div id="tab:regeneration">
<table>
<thead>
</thead>
<tbody>
<tr><td align="center">method</td><td align="right">rays/sec</td><td align="right"></td><td align="right"></td><td align="right"></td></tr>
<tr><td align="center"></td><td align="right">cgg-logo</td><td align="right">ad</td><td align="right">artifix</td><td align="right">manix</td></tr>
<tr><td align="center">naiveSK</td><td align="right">3.88</td><td align="right">2.13</td><td align="right">8.34</td><td align="right">8.45</td></tr>
<tr><td align="center">regenerationSK (thread)</td><td align="right">81.62</td><td align="right">42.37</td><td align="right">11.80</td><td align="right">11.42</td></tr>
<tr><td align="center">regenerationSK (warp)</td><td align="right">17.13</td><td align="right">17.05</td><td align="right">12.11</td><td align="right">14.82</td></tr>
<tr><td align="center">regenerationSK (block)</td><td align="right">3.52</td><td align="right">3.21</td><td align="right">7.35</td><td align="right">7.41</td></tr>
</tbody>
</table>
comparison of different types of regeneration on the scenes presented in
figure <a href="/thesis/optimization-methodology/#fig:scenes" data-reference-type="ref"
data-reference="fig:scenes">4.6</a>. Three regeneration approach are
taken in consideration: regenerationSK (thread) doesn’t require any type
of synchronization and regenerate a thread immediately after it becomes
idle. regenerationSK (warp) require all the warp to be idle before
regenerating. regenerationSK (block) is synchronizing all the block and
regenerating only when all the block is idle. Finally, the naiveSK
approach is a simple volumetric path tracer which is not performing any
regeneration.</div>

<span id="tab:regeneration" label="tab:regeneration"></span>

### Occupancy

The occupancy of a kernel consists of the number of active warps (SIMD
group) on a multiprocessor when launching the kernel. If the occupancy
is maximal, then all the warps that can simultaneously reside on a
multiprocessor are active and the warp scheduler can hide the latency of
the warps which cannot execute their next instruction. However, if the
kernel requires too many registers or too much shared memory, it have to
limit the number of threads active on a single multiprocessor. If the
kernel requires too much shared memory, there is nothing that the
compiler could do to increase the occupancy. However, if the problem is
the number of registers used, then one possibility is to limit this
number using register spilling. In our case, the kernel of our
volumetric path tracer requires approximately 64 registers for the
compiler, which limits the occupancy to 50%. To achieve the 100% of
occupancy we could force the compiler to reduce the number of registers
used by our path tracer.

#### Bounding Register Usage

Bounding the number of registers used by a kernel is not always a good
idea. The compiler is usually limiting the number of register used by
itself and tries to make the best decision between increasing the
occupancy and efficiently storing memory on registers. Sometimes,
however, it is possible that this decision is not the one that gives the
best performance. For this reason, another possibility is to force a
kernel to use a minimum number of blocks per multiprocessor by limiting
even more the number of registers used. In the table
<a href="/thesis/optimization-methodology/#tab:bounded-registers" data-reference-type="ref"
data-reference="tab:bounded-registers">4.4</a> we can see the effect of
achieving maximum occupancy using this technique. All the results are
worse respect to the version with half the occupancy in table
<a href="/thesis/optimization-methodology/#tab:regeneration" data-reference-type="ref"
data-reference="tab:regeneration">4.6</a>. By analyzing the process, it
is possible to see that also if the number of warps in the
multiprocessor is maximum, the latency is higher than before. The time
required to get the spilled registers is therefore higher than the
latency hiding capability of the device.

<div id="tab:bounded-registers">
<table>
<thead>
</thead>
<tbody>
<tr><td align="center">method</td><td align="right">rays/sec</td><td align="right"></td><td align="right"></td><td align="right"></td></tr>
<tr><td align="center"></td><td align="right">cgg-logo</td><td align="right">ad</td><td align="right">artifix</td><td align="right">manix</td></tr>
<tr><td align="center">regenerationSK (thread)</td><td align="right">39.04</td><td align="right">32.81</td><td align="right">12.66</td><td align="right">11.37</td></tr>
<tr><td align="center">regenerationSK (warp)</td><td align="right">4.37</td><td align="right">7.02</td><td align="right">11.92</td><td align="right">10.95</td></tr>
</tbody>
</table>
maximize occupancy decreasing registers usage. In those results the
number of register used by the kernels is bounded so that the device can
achieve maximum occupancy. However, comparing those results with the
ones in table <a href="/thesis/optimization-methodology/#tab:regeneration" data-reference-type="ref"
data-reference="tab:regeneration">4.6</a> is possible to see that this
method is actually performing much worse than the previous one with only
50% of occupancy</div>

<span id="tab:bounded-registers" label="tab:bounded-registers"></span>

#### Maximize Register Usage

Increasing the occupancy is not the only way to hide threads latency;
another possibility is to leverage the instruction-level parallelism.
Indeed, a warp that has subsequent independent instructions can hide the
latency of an instruction performing the next one. This technique allows
to lower the occupancy needed to obtain the device peak of performance,
which can be calculated with Little’s law:

$$
\text{parallelism} = \text{latency} \cdot \text{throughput}
$$

We have used this technique in our volumetric path tracer by assigning
more than one path to each thread. This allows to decrease the
dependency among subsequent instructions and, therefore, to hide the
latency inside a single thread using the instruction-level parallelism.
Recent tests  have demonstrated how the use of more registers per
thread, using less threads per multiprocessor, can lead to better
performance with smaller occupancy. We have also tested our volumetric
path tracing with a smaller occupancy, but without any visible
improvements. The conclusion that we may take is that in this case the
compiler is doing an optimal choice about the number of registers and
threads to use.

### Summary

We have seen in this section different application of the persistent
thread techniques, which differ on synchronization points. Moreover, we
have seen some techniques to achieve better performances at lower
occupancy utilizing instruction-level parallelism and more registers per
thread. However, the difference in latency between arithmetic operations
and memory operations is high and the best way to improve the
performance is decreasing this gap. For this reasons, in the next
section, we will discuss about localizing memory access inside the
kernels to maximize the cache usage and decrease the memory access
latency.

## Data Locality and Code Divergence

We have discussed until now all the performance limiting factors
regarding the full utilization of the GPU and the logic communication
between host and device. In this chapter, we are going to discuss about
one of the most important limiting factor in GPGPU: the bandwidth. In
the section <a href="#sec:gpu-background" data-reference-type="ref"
data-reference="sec:gpu-background">3.2</a> we have seen that access on
every level of GPU memory hierarchy have different bandwidth. Localizing
the data access inside a kernel is a key factor for decreasing the
number of low bandwidth data transfers. Code divergence is another
limiting factor which can drop the performance causing the instruction
throughput to decrease. When a warp executing a kernel meets a flow
control instruction, it diverges if the threads in the warp follow
different flows of execution and if the different instructions are
substantial (otherwise predication is used). The hardware maintains a
bit vector of active threads and executes the code once for the active
and then for the inactive. When all the executions paths are complete
the warp re-converge to the original path. All those limitations are
present in a Monte Carlo Path Tracer that is based on a random control
flow and thus random memory access. It is not straightforward to create
a predetermined data access pattern that can be exploited in this case.
Dietger et al.  have proposed a solution based on stream compaction of
the active threads at every path extension. In this solution, paths that
are still active are compacted together and stored in the global memory
of the device. In this section, we will propose a new method that aims
to merge the benefits of compacting with the reordering of the rays to
exploit warp and data coherence.

### Compaction

As we have already discussed in the introduction of the chapter, active
thread compaction allows to separate the threads into active and idle.
There are two main advantages on using this approach for a MC path
tracer:

1.  SIMD efficiency increase: the compaction of the active threads
    allows to have warps fully active or fully inactive.

2.  Primary ray coherence is maintained: the regenerated paths usually
    follow the same control flow path during the first iteration without
    any code divergence and access the same GPU caches exploiting data
    locality.

Also in this case we want to analyze the different approaches to
compaction:

- Device Compaction (<em>StreamingMK</em>): all the active threads inside the
  device are compacted together. The methods requires two streams of
  global memory for path data one in input and one in output. The
  compaction is efficiently performed using an atomic operation which
  tracks the number of elements written to the output stream. This
  method requires two kernels, one for regenerating inactive paths and
  one for extending the paths.

- Tiled Block Compaction (<em>StreamingSK</em>): the active threads are
  compacted only inside a block. The path data is still stored in the
  global memory, but the compaction of the active threads is performed
  only on a tile of this data, large as much as the number of threads
  inside a block. The graphical output of the compaction can be seen in
  figure <a href="/thesis/optimization-methodology/#fig:block-compaction" data-reference-type="ref"
  data-reference="fig:block-compaction">4.8</a> and pseudo-code in
  algorithm <a href="#alg:streamingSK" data-reference-type="ref"
  data-reference="alg:streamingSK">streamingSK</a>.

In both cases the compaction is performed by the following steps (which
are graphically showed in figure
<a href="/thesis/optimization-methodology/#fig:compaction" data-reference-type="ref"
data-reference="fig:compaction">4.7</a>):

1.  an exclusive sum on the active labels of the thread allows to find
    the position of each active thread inside the output stream (which
    is the same as the input for the tiled block compaction)

2.  the number of active threads is updated (the Device compaction
    requires an atomic operation).

3.  the active threads write their data on the output stream.

<figure id="fig:compaction" class="figure-full">
<embed class="figure-img" src="/assets/images/thesis/tex_charts_Compaction.png" />
<figcaption>steps for compacting a group of threads. In the top active
threads are colored in orange and associated with the label 1 inside a
row of blocks representing a linear memory of threads. In the second row
of threads from the top, an exclusive sum operation is performed on the
active labels. In the third row the computed sums are used for
identifying the closest empty thread in the memory. The atomic counter
is incremented with the last sum computed and the threads which reside
after the position indicated by this counter are
regenerated</figcaption>
</figure>

In our tests we have seen a performance boost of the block compaction
versus the device compaction. We think that most of the performance gain
is given by the use of a single kernel. However, compared to the
<em>RegenerationSK</em> on a warp the <em>StreamingVolPTsk</em> is getting better
result, demonstrating the improvement given by the compaction.

<div id="tab:regeneration">
<table>
<thead>
</thead>
<tbody>
<tr><td align="center">method</td><td align="right">rays/sec</td><td align="right"></td><td align="right"></td><td align="right"></td></tr>
<tr><td align="center"></td><td align="right">cgg-logo</td><td align="right">ad</td><td align="right">artifix</td><td align="right">manix</td></tr>
<tr><td align="center">regenerationSK (thread)</td><td align="right">81.62</td><td align="right">42.37</td><td align="right">11.80</td><td align="right">11.42</td></tr>
<tr><td align="center">regenerationSK (warp)</td><td align="right">4.37</td><td align="right">7.02</td><td align="right">11.92</td><td align="right">10.95</td></tr>
<tr><td align="center">streamingSK</td><td align="right">52.69</td><td align="right">37.81</td><td align="right">7.42</td><td align="right">8.02</td></tr>
<tr><td align="center">streamingMK</td><td align="right">6.59</td><td align="right">4.70</td><td align="right">5.44</td><td align="right">5.18</td></tr>
</tbody>
</table>
comparison of different types of compaction. The streamingSK method is
compacting all the active threads that are in the same block, whereas
the streamingMK is compacting all the active threads in all the device.
Those two new methods are compared with the regeneration methods which
are not using compaction</div>

<span id="tab:regeneration" label="tab:regeneration"></span>

<figure id="fig:block-compaction" class="figure-medium">
<embed class="figure-img" src="/assets/images/thesis/tex_charts_block_compaction.png" />
<figcaption>Block Compaction. This figure shows how the active threads
are compacted using the StreamingSK algorithm. The memory containing the
threads is divided in blocks and only inside each block the compaction
is performed.</figcaption>
</figure>

<div class="algorithm">
<u>kernel streamingSK</u>
<div class="algorithm-line">shared total_active = 0</div>
<div class="algorithm-line">do</div>
<div class="algorithm-line">  thread = loadThread(threads)</div>
<div class="algorithm-line">  if thread.id $\geq$ total_active then</div>
<div class="algorithm-line">    paths_head = regenerate(thread, paths_head)</div>
<div class="algorithm-line">  end if</div>
<div class="algorithm-line">  synchronizeBlock()</div>
<div class="algorithm-line">  if thread.active then</div>
<div class="algorithm-line">    thread, image = extend(thread, image, scene)</div>
<div class="algorithm-line">  end if</div>
<div class="algorithm-line">  total_active, compacted_position = exclusiveSum(thread)</div>
<div class="algorithm-line">  if thread.active then</div>
<div class="algorithm-line">    threads[compacted_position] = thread</div>
<div class="algorithm-line">  end if</div>
<div class="algorithm-line">while total_active $>$ 0 or paths_head $<$ total_paths</div>
</div>

### Reordering

In this section we are going to discuss a new method which is based on
the previously detailed <em>StreamingVolPTsk</em>. We have seen in the previous
chapter that compaction is helping on maintaining high SIMD efficiency
and primary ray coherence. However, the secondary rays are usually not
coherent and they are compacted to the others active rays independently
on their position or direction. The method that we propose consists in
using the <em>Morton order</em>, also called Z-order, to reorder the active
rays while compacting. Moon et al.  have already used this technique to
achieve a cache-oblivious ray reordering for path tracing; they achieved
more than an order of magnitude performance for big models that cannot
fit into main memory. Our scope is extending this work to GPU and
integrating it with the already discussed streaming compaction.

### Morton Order

The Morton order (also called Z-order for its characteristic shape as in
figure <a href="/thesis/optimization-methodology/#fig:z-order" data-reference-type="ref"
data-reference="fig:z-order">4.9</a>) is based on a space filling curve
which is a function that maps multidimensional data to one dimension,
preserving locality of the data points. In our implementation, the
z-value of a point is calculated by interleaving bits of the quantized
coordinates of the multidimensional data  . The quantized coordinates,
which are lying between 0 and 1, are transformed linearly into 10-bit
integers. Those integers are then expanded using zeros between the bits
of each coordinate, so that the different coordinates can be combined
together, interleaving their bits, into a single value. This value
represent the z-value of the coordinate (graphical structure of the
method can be found in figure
<a href="/thesis/optimization-methodology/#fig:morton" data-reference-type="ref"
data-reference="fig:morton">4.10</a>). The resulting ordering correspond
of a depth-first traversal of a commonly used 3D data structure called
Octree.

<figure id="fig:z-order">
<table style="border: none; background: none;">
<tr>
<td style="border: none;"><figure class="figure-medium">
<img src="/assets/images/thesis/tex_img_MortonCurve-2x2x2.png" />
</figure></td>
<td style="border: none;"><figure class="figure-medium">
<img src="/assets/images/thesis/tex_img_MortonCurve-4x4x4.png" />
</figure></td>
<td style="border: none;"><figure class="figure-medium">
<img src="/assets/images/thesis/tex_img_MortonCurve-8x8x8.png" />
</figure></td>
</tr>
</table>
<figcaption> Z-Order. from the left to the right we have: 3D z-curve
with resolution 2X2X2, 3D z-curve with resolution 4X4X4, 3D z-curve withß
resolution 8X8X8. It is important to notice how the 3D points are mapped
into a linear curve so that points that are near in the 3D space are
near also in the curve (the image is taken from the website
http://asgerhoedt.dk) </figcaption>
</figure>

<figure id="fig:morton" class="figure-full">
<embed class="figure-img" src="/assets/images/thesis/tex_charts_morton_code.png" />
<figcaption> Morton code generation: this chart explain the algorithm
used to calculate the Morton code (z-value). First the fractional part
of the coordinates is taken. Then those coordinates are expanded using 0
values. Finally the coordinates are combined together to create the
final code. </figcaption>
</figure>

### Z-Sorting

In our contest, the main source of data is given by the volumetric data
that is accessed through the texture memory of the device. Texture
accesses in CUDA give best performances in applications where memory
access patterns exhibit spatial locality. For this reason reordering
rays using the z-order should improve the access pattern on the texture.
The coordinate that have to be used for that is the position of the ray
inside the volume. Thus, the bounding box of the volume is used to
quantize the ray position. The quantized position are then transformed
as we have previously seen into z-values. Finally, we are reordering the
rays using the block radix sort, where the keys used for reordering are
the z-values of the ray position (as shown in figure
<a href="/thesis/optimization-methodology/#fig:sorting" data-reference-type="ref"
data-reference="fig:sorting">4.11</a>. If the thread is not active the
maximum z-value is associated with the ray. This method allows to sort
and compact at the same time the threads inside a single block. We have
analyzed two possible implementation of the z-sorting:

1.  Sorting and compacting before texture access (<em>StreamingSK</em> with
    sorting variant): consists in using the same tiled block compaction
    already seen before, but this time, instead of using a scan
    operation to compact the active rays, we are using Morton sorting.

2.  Sorting and compacting after texture access (<em>SortingSK</em>) : consists
    in delaying the texture access for the albedo volume of the
    volumetric path tracer after reordering and compaction of the rays
    (a pseudo code of the algorithm can be found in
    <a href="#alg:sortingSK" data-reference-type="ref"
    data-reference="alg:sortingSK">sortingSK</a>).

The first option should help to create more coherent rays, which in the
next path extension will have more probability to access data that is
near in memory. The second option uses a shared array of labels
describing if the albedo texture is accessed by the thread during the
last extension. This array is then used after reordering and compaction
to decide which of the reordered paths need to access the albedo
texture. Finally, the texture is accessed by the reordered threads. That
is, the last option allows to access the texture with the more coherent
way given random access positions.

<div class="algorithm">
<u>kernel sortingSK</u>
<div class="algorithm-line">shared total_active = 0</div>
<div class="algorithm-line">shared tex_access[n_threads] = false</div>
<div class="algorithm-line">do</div>
<div class="algorithm-line">  thread = loadThread(threads)</div>
<div class="algorithm-line">  if thread.id $\geq$ total_active then</div>
<div class="algorithm-line">    paths_head = regenerate(thread, paths_head)</div>
<div class="algorithm-line">  end if</div>
<div class="algorithm-line">  synchronizeBlock()</div>
<div class="algorithm-line">  tex_access, thread, image = extend(thread, image, scene, tex_access)</div>
<div class="algorithm-line">  sorted_position = blockZSorting(thread)</div>
<div class="algorithm-line">  total_active = sum(thread)</div>
<div class="algorithm-line">  thread = swapThreads(thread, sorted_position)</div>
<div class="algorithm-line">  if tex_access[sorted_position] then</div>
<div class="algorithm-line">    thread = accessTexture(thread)</div>
<div class="algorithm-line">  end if</div>
<div class="algorithm-line">while total_active $>$ 0 or paths_head $<$ total_paths</div>
</div>

<div id="tab:regeneration">
<table>
<thead>
</thead>
<tbody>
<tr><td align="center">method</td><td align="right">rays/sec</td><td align="right"></td><td align="right"></td><td align="right"></td></tr>
<tr><td align="center"></td><td align="right">cgg-logo</td><td align="right">ad</td><td align="right">artifix</td><td align="right">manix</td></tr>
<tr><td align="center">regenerationSK (thread)</td><td align="right">81.62</td><td align="right">42.37</td><td align="right">11.80</td><td align="right">11.42</td></tr>
<tr><td align="center">regenerationSK (warp)</td><td align="right">4.37</td><td align="right">7.02</td><td align="right">11.92</td><td align="right">10.95</td></tr>
<tr><td align="center">streamingSK (compaction)</td><td align="right">52.69</td><td align="right">37.81</td><td align="right">7.42</td><td align="right">8.02</td></tr>
<tr><td align="center">streamingSK (sorting)</td><td align="right">20.41</td><td align="right">17.61</td><td align="right">6.78</td><td align="right">6.74</td></tr>
<tr><td align="center">sortingSK</td><td align="right">20.90</td><td align="right">17.83</td><td align="right">7.02</td><td align="right">6.87</td></tr>
</tbody>
</table>
sorting comparison: in this table two different sorting methodologies
are adopted. the first one, streamingSK (sorting) is behaving exactly
like the streamingSK with the only difference that the sorting with the
z-order based on the ray position is used instead of the compaction. The
second one, sortingSK, have the only difference of postponing the
texture access after the reordering like in algorithm
<a href="#alg:sortingSK" data-reference-type="ref"
data-reference="alg:sortingSK">sortingSK</a>. It is clear from the
table that the sorting methods are not performing better than the other
ones on those scenes.</div>

<span id="tab:regeneration" label="tab:regeneration"></span>

<figure id="fig:sorting" class="figure-full">
<embed class="figure-img" src="/assets/images/thesis/tex_charts_sorting.png" />
<figcaption>z-reordering of the data. In this image is explained the
reordering methods used in this work. At the top the points inside the
volume (orange blocks inside the cube) that correspond to a thread in
the global memory (represented by a row of blocks) are placed
independently on their position inside the volume. In the bottom the
threads corresponding to the points in the volume are reordered based on
the z-value of those points and compacted.</figcaption>
</figure>

### Summary

In this section we have discussed different techniques to improve data
locality and thread divergence. Compaction allows to create coherent
primary rays and to improve SIMD efficiency, while reordering of rays
tries to improve secondary rays coherency inside the volume.
Unfortunately, the results show that the increased data coherence are
not affecting the performance of the algorithm. One of the reasons for
this behavior could be the too high texture resolution compared to the
number of sample points inside the volume, represented by the texture
origins. If the ratio of those two values is too big, then the method
cannot explore data coherence by reordering the rays. Indeed, in this
case, considering the random position of the rays inside the volume it
is improbable that their position will be close. Therefore, even if the
method is accessing the texture in a coherent order, the texture cache
is not big enough to store the values between one texture access and the
next one.

<div class="page-nav">
  <a href="/thesis/implementation-details/" class="next-page">Next chapter →</a>
</div>
