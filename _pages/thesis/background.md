---
layout: page
title: "Background"
permalink: /thesis/background/
math: true
nav_order: 
parent: Thesis
hidden: true
---

In this chapter, we will look more in depth at the background theory
necessary to discuss this work.

## Volumetric Path Tracing

We have seen in the previous chapter the volumetric rendering equation
<a href="/thesis/problem-statement/#eq:volumetric-rendering-equation" data-reference-type="ref"
data-reference="eq:volumetric-rendering-equation">volumetric-rendering-equation</a>.
Volumetric path tracing is a technique that tries to estimate this
integral. For image synthesis we are usually interested on the radiance
( in equation <a href="/thesis/problem-statement/#eq:Ldef" data-reference-type="ref"
data-reference="eq:Ldef">Ldef</a>) which hits a virtual camera
pointed toward our scene. In order to do that, it uses the Monte Carlo
Integration framework, which we are briefly presenting in the next
paragraph.

### Monte Carlo Methods

Monte Carlo integration is a general tool for estimating integrals by
randomly sampling the domain of integration given a probability
distribution function $p(x)$. More formally, this proposition, which
we are not going to demonstrate, defines the Monte Carlo estimator.

<div class="prop">

<em>Proposition 3.1</em>

<p>
Given $f:\mathbb R^d \rightarrow \mathbb R$ a
function s.t.


$$
I = \int_{\mathcal Q} |f(x)| \,dx < \infty
$$


where 
$$ \mathcal Q \subset \mathbb R^d $$

is a limited set. </p>

<p>
Let $n$ be a random d-dimensional independent vectors $X_1,.....,X_n$ with the same
Probability Distribution Function (here and after PDF),
$p=p_{X_1}=...p_{X_n}$. We define the random variable:

$$
<  I  >_n:= \frac {1}{ n} \sum_{i = 1}^n \frac {f(X_i)}{p(X_i)} \chi_{\mathcal Q} (X_i)
$$

which we call estimator of I.

</p>

<p>
then:

$$
E\big(< I >_n\big)=I
$$

for any $n>0$.
</p>

</div>

The framework works particularly well for complex high-dimensional
integrals and its implementation is very simple, which is perfect for
GPU algorithms. Increasing the number of samples the estimator converges
to the exact solution with a standard deviation which follows
$\sigma = \mathcal{O}(n^{-\frac{1}{2}})$.

### Ray integral estimator

Considering our integral in equation
<a href="/thesis/problem-statement/#eq:volumetric-rendering-equation" data-reference-type="ref"
data-reference="eq:volumetric-rendering-equation">volumetric-rendering-equation</a>
the Monte Carlo primary estimator for the radiance $L$ in the point
$x$ in the direction $\omega$ is:

$$
\begin{aligned}
<  L(x,\omega)  >_1 = \frac{T(x,X_1) \, L_v(X_1, \omega)}{p(X_1)}
\end{aligned}
$$

if we use uniform sampling of the variable $X_1$ respect to distance
along the ray which start at $x$ and ends at $x_s$, representing in
this case the domain $\mathcal Q$ of our integral, we have that

$$
p(X) = \frac{1}{||x-x_s||}
$$

and the estimator will be

$$
\begin{aligned}
<  L(x,\omega)  >_1 \, = \, T(x,X_1) \, L_v(X_1, \omega) \,||x-x_s||
\end{aligned}
$$

However, this simple estimator can lead to high variance when the medium
is particularly dense. To improve the estimator, we can use a different
PDF which simplifies the estimator. This technique is called importance
sampling and in the next paragraph we will see how to sample based on
the transmittance such that, rewriting the transmittance in terms of the
distance along the ray $s'=||x-X_1||$ and considering the
transmittance independent from $x$, we have

<div id="eq:pdf-transmittance"></div>
$$
\begin{aligned}
\label{eq:pdf-transmittance}
p(s') = \sigma_t(s')\, T(s') = \sigma_t(s') \, e^{-\int_0^{s'} \sigma_t(p) dp }
\end{aligned}
$$

where $\sigma_t(s')$ is a normalization term necessary to transform
$T(s')$ in a PDF.

### Sampling distances according to the Transmittance

Woodcock tracking, which can be found in algorithm
<a href="#alg:wood" data-reference-type="ref"
data-reference="alg:wood">wood</a>, is a technique allowing to
sample a point along a ray so that the distribution follows the
transmittance function. The algorithm needs the maximum extinction
coefficient $\sigma_{max}$ inside the medium. This coefficient is used
to sample the distance according to the Beer-Lambert-Law. After that,
the sample is rejected or accepted according to the real extinction
coefficient $\sigma_t(x)$ of the medium. If it is rejected, another
step is taken according to the same law, otherwise the sampled distance
is returned.

<div class="algorithm">
<u>function woodcockTracking$(\mathbf{x},\omega, \sigma_{max}, s_{max})$ </u>
<div class="algorithm-line">$s'=0$</div>
<div class="algorithm-line">while $s' \leq s_{max}$ do</div>
<div class="algorithm-line">  $s' += -\log_{e}(1-\textit{rand()})/\sigma_{max}$</div>
<div class="algorithm-line">  if $\textit{rand()} < \sigma_t(\mathbf{x}- s' \omega ) / \sigma_{max}$ then</div>
<div class="algorithm-line">    break</div>
<div class="algorithm-line">  end if</div>
<div class="algorithm-line">end while</div>
<div class="algorithm-line">return $s'$</div>
</div>

We are not demonstrating here the validity of this method to sample the
distribution $p(S)=\sigma_t(S)T(S)$. For the interested reader we
suggest the original article about the method by .

### Directional integral estimator

Now we can sample from the probability distribution function defined in
<a href="/thesis/background/#eq:pdf-transmittance" data-reference-type="ref"
data-reference="eq:pdf-transmittance">pdf-transmittance</a>. The
primary estimator for the integral inside the VRE in equation
<a href="/thesis/problem-statement/#eq:volumetric-rendering-equation" data-reference-type="ref"
data-reference="eq:volumetric-rendering-equation">volumetric-rendering-equation</a>
in terms of distance along the ray $s' = || x - X_1 ||$ becomes

$$
\begin{aligned}
\begin{split}
<  L(x,\omega) >_1 \, &= \frac{ T(x,x + s'\omega) \, L_v(x+s'\omega, \omega) }{p(s')} \\ \,
&= \frac{ L_v(x+s'\omega, \omega) }{\sigma_t(x+s'\omega)}
\end{split}
\end{aligned}
$$

However, the term $L_v$, which can be found in equation
<a href="/thesis/problem-statement/#eq:volumetric-source" data-reference-type="ref"
data-reference="eq:volumetric-source">volumetric-source</a>, is
constituted by another integral on the set of directions
$\mathcal{S}^2$, which means that after sampling a distance we have
now to sample also a direction $\omega'$. Also in this case we could
just sample according to an uniform distribution, but this will lead
most of the times to an high variance estimator. A much better idea is
to sample according to some factor of the integrand, which is composed
of the phase function $f_p(\omega,x,\omega')$ and the radiance
$L(x, \omega')$. The second term is more difficult to use as it is
exactly what we are searching for, so in this work we will sample
according to the phase function.

### Henyey-Greenstein Function

In the first chapter, we have introduced the phase function as a
characteristic of the medium we are going to render. In our work we are
using as approximation of the Mie Phase functions the Henyey Greenstein
function (abbreviated HG function hereinafter), the reader interested on
other phase functions can read the comprehensive guide about multiple
scattering by . The Mie phase functions describe the scattering behavior
of light interacting with perfectly spherical dielectric particles with
different diameter size. The phase functions can be characterized by the
anisotropy $g$, which is the first angular moment of the function:

$$
\begin{aligned}
g = \int_0^{2\pi}\int_0^{\pi} f_p(\theta',\phi')cos(\theta')sin(\theta')d\theta'd\phi'
\end{aligned}
$$

where $g \in [-1,1]$ and, when it is positive, the light scatters
predominantly into forward directions, while if it negative it scatters
predominantly into backward directions. The HG function is parametrized
by this value and can be written as:

$$
\begin{aligned}
f_{HG}(\theta,\phi,g) = \frac{1}{4\pi} \frac{1-g^2}{(1+g^2-2g\cos\theta)^{3/2}}
\end{aligned}
$$

given this function a spherical direction $(\theta, \phi)$ can be
sampled inverting analytically the function (more information in the
work of ) as

$$
<  I  >_n:= \frac {1}{ n} \sum_{i = 1}^n \frac {f(X_i)}{p(X_i)} \chi_{\mathcal Q} (X_i)
$$

where $\xi_{1,2} \in [0,1]$ are uniformly distributed random samples.

### Volumetric integral estimator

Now we can write the estimator for the term $L_v$, sampling the
direction $\omega'$, with the technique described in the previous
paragraph. We have that the primary estimator for the $L_v$ term in
the volumetric rendering equation
<a href="/thesis/problem-statement/#eq:volumetric-rendering-equation" data-reference-type="ref"
data-reference="eq:volumetric-rendering-equation">volumetric-rendering-equation</a>
is

$$
< L_v(x',\omega) >_1 \, = L_{e,V}(x',\omega') + \sigma_s(x') \frac{ f_p(\omega,x,\omega') L(x', \omega') }{p(\omega')} =  L_{e,V}(x',\omega') + \sigma_s(x') L(x', \omega')
$$

and combining all the results obtained, we have the final estimator

$$
< L(x,\omega) >_1 = \frac{ L_{e,V}(x',\omega') + \sigma_s(x') L(x', \omega')}{\sigma_t(x')} = \frac{ L_{e,V}(x',\omega')}{\sigma_t(x')} + \frac{ \sigma_s(x') L(x', \omega')}{\sigma_t(x')}
$$

where $x' = x+s'\omega$. In the work we are not considering medium
which can emit light, we will have
$L_{e,{\mathcal{V}}}(x',\omega') = 0$ and the equation simplify to

$$
< L(x,\omega) >_1 = \frac{ \sigma_s(x') L(x', \omega')}{\sigma_t(x')} = \alpha(x') L(x', \omega')
$$

where $\alpha(x')$ is the albedo defined in
<a href="/thesis/problem-statement/#eq:albedo" data-reference-type="ref"
data-reference="eq:albedo">albedo</a>. This estimator is very
simple and perfect for GPU implementation, where the albedo can be
easily mapped to an interpolated 3D texture. Using only one sample is
usually not enough and, in its general description, the final estimator
using $n$ samples is

$$
< L(x,\omega) >_n = \frac {1}{n} \sum_{i = 1}^n \alpha(x') L(x', \omega')
$$

### Volumetric Path Tracer Algorithm

The volumetric path tracer uses exactly this estimator on every pixel of
the image sensor. In algorithm
<a href="#alg:volpt" data-reference-type="ref"
data-reference="alg:volpt">volpt</a> a simplified version is
presented only considering background light. The algorithm uses also the
sampling from a Bsdf (Bidirectional scattering distribution function),
that we have not described in the previous paragraphs (the reader can
find its description in the work of ). In this work a physically-based
Bsdf is used, called the GGX. Whenever the point is a surface, the GGX
is sampled instead of the phase function. We sample the GGX using the
distribution of the visible normal. In particular, we are using the
sampling strategy described by which does not require any look up table
to be computed.

<div class="algorithm">
<u>function radiance$(x,\omega)$</u>
<div class="algorithm-line">throughput = Color(1)</div>
<div class="algorithm-line">while true do</div>
<div class="algorithm-line">  hit, $s_{max}$ = intersect$(x,\omega)$</div>
<div class="algorithm-line">  if hit then</div>
<div class="algorithm-line">    $s = \text{woodcockTracking}(\mathbf{x},\omega, \sigma_{max}, s_{max})$</div>
<div class="algorithm-line">    if $s \leq s_{max}$ and insideVolume(x) then</div>
<div class="algorithm-line">      $x = x + \omega \cdot s$</div>
<div class="algorithm-line">      $\omega = \text{samplePhase}(\omega)$</div>
<div class="algorithm-line">      throughput = throughput $\cdot$ albedo(x)</div>
<div class="algorithm-line">    else</div>
<div class="algorithm-line">      $x = x + \omega \cdot s_{max}$</div>
<div class="algorithm-line">      $\omega$, weight = sampleBsdf($\omega$)</div>
<div class="algorithm-line">      throughput = throughput $\cdot$ weight</div>
<div class="algorithm-line">    end if</div>
<div class="algorithm-line">  else</div>
<div class="algorithm-line">    return $L_e(x,\omega) \cdot$ throughput</div>
<div class="algorithm-line">  end if</div>
<div class="algorithm-line">end while</div>
</div>

<div class="algorithm">
<u>function volPT</u>
<div class="algorithm-line">Image = Image($0$)</div>
<div class="algorithm-line">for each pixel p in Image do</div>
<div class="algorithm-line">  for n iterations do</div>
<div class="algorithm-line">    $x, \omega = \text{cameraRay}(p)$</div>
<div class="algorithm-line">    p += radiance($x, \omega$)</div>
<div class="algorithm-line">  end for</div>
<div class="algorithm-line">  p = p/n</div>
<div class="algorithm-line">end for</div>
</div>

### Russian Roulette

Another technique usually implemented with path tracing is the Russian
Roulette. This technique permits to stochastically interrupt a path
before reaching a light source. The probability of interrupting can be
associated with the amount of light transported by the path. In this way
is possible to interrupt with more probability paths which gives smaller
contribution to the overall result. This technique is also unbiased if
we modify our estimator by dividing for the survival probability of the
path. More information about this technique can be found in the work by
. In the software provided it is possible use the Russian Roulette
inside any of the volumetric path tracer showed. However, we didn't
focus the research on this particular aspect.

## GPU Architecture

In this section, we want to review some basics of the GPU architecture,
that we are going to consider as a data parallel computational platform,
and the characteristics of the CUDA platform.

### Design Philosophy and Heterogeneous Programming

One of the most used classifications for computer architecture is the
Flynn's Taxonomy. In this classification, the architectures are divided
into 4 groups:

- Single Instruction Single Data (SISD) : this is the traditional serial
  architecture.

- Single Instruction Multiple Data (SIMD) : multiple cores execute the
  same instruction at the same time.

- Multiple Instruction Single Data (MISD) : multiple cores use separate
  instructions on the same data.

- Multiple Instruction Multiple Data (MIMD) : multiple cores operate on
  multiple data streams.

Another type of classification can be done based on the memory:

- Multi-node with distributed memory: processors are connected by a
  network, each having its own local memory.

- Multiprocessor with shared memory: processors are either connected to
  the same memory or through a low-latency link, e.g. PCI-Express or
  PCIe.

Finally, we will use other two properties to characterize an
architecture:

- throughput (ops/cycle or Tflop/s): rate of operations complete per
  cycle, usually calculated considering floating point operations and
  seconds (it can be sometimes confused with latency, cycle or s, which
  is the waiting time).

- bandwidth (GB/s): rate at which data can be transferred.

In this context, GPU is a multiprocessor with shared memory architecture
that utilizes SIMD groups, called warps. However, some of the features
of the other architectures are also present on NVIDIA GPUs. NVIDIA call
their architecture as Single Instruction, Multiple Thread (SIMT). At the
base of the different performance between CPU and GPU is the different
design philosophy of the architecture. Contrary to CPU, GPU are designed
for high throughput and low latency tasks. GPU is many-core architecture
which contain a high number of cores and high memory bandwidth. Each
core is also very different from a CPU core. CPU cores are heavy-weight
and they are designed to handle complex control logic, instead the GPU
cores are very light-weight and optimized for data-parallel tasks with
simple logic. For this reason, both architectures should be used
together in a heterogeneous system for different scopes. Currently GPU
cannot be used standalone and execution should be initialized by CPU.
Therefore, the CPU is usually called the <em>host</em> and the GPU the
<em>device</em>. In General Purpose GPU programming (GPGPU) execution of code
on the device can be divided in different parts, which we will call
kernels. The host code can call those kernels during its execution and
execute the code of the kernel on the device side. The calls of those
device kernels can also be asynchronous, allowing to the host to perform
more work while the device is running the kernel.

<figure id="fig:gpu">
<embed src="/assets/images/thesis/tex_img_gpucpu.png" width="100%" />
<figcaption>CPU and GPU, heterogeneous programming</figcaption>
</figure>

### CUDA - Compute Unified Device Architecture

CUDA is a GPGPU platform introduced by NVIDIA for their GPU on 2006. It
is designed for scalable parallelism allowing to any CUDA application to
leverage whichever NVIDIA GPU they are running on. This is enabled by
the usage of three main abstractions:

- hierarchy of thread groups

- memory hierarchy

- barrier synchronization

These abstractions are then mapped to concrete implementation on the
specific hardware by CUDA. Figure
<a href="/thesis/background/#fig:memory-hierarchy" data-reference-type="ref"
data-reference="fig:memory-hierarchy">3.2</a> shows the thread and
memory hierarchy and how the memory hierarchy is mapped to the Kepler
GPU architecture. The programmer, to leverage all the parallelism of a
GPU, must partition the multi-threaded program into blocks of threads
which execute independently and in any order.

<figure id="fig:memory-hierarchy" class="figure-large">
<embed class="figure-img" src="/assets/images/thesis/tex_charts_memory_hierarchy.png" class="figure-img"/>
<figcaption>Memory and Thread Hierarchy. All threads can also directly
access the read-only memory using the constant and texture memory. The
concrete Memory hierarchy showed is relative to the Kepler Architecture
(compute capability 3.0)</figcaption>
</figure>

In the concrete, implementation of the CUDA model the GPU uses an array
of Streaming Multiprocessors (SM or SMX). The multiprocessors schedule
threads using SIMD groups of 32 threads called warps. However, threads
in a warp can have their own instructions and they are able to branch
and execute independently (also if this will not exploit the SIMD
capability of the group potentially serializing the program). After the
Kernel launch, a multiprocessor can get a thread block to execute. At
this point, the warp scheduler partitions it into warps with consecutive
thread ID. Program counters and registers of each warps are maintained
on the multiprocessor for the entire life of the warp. For this reason,
the multiprocessor can switch between different warps with no cost. The
warp scheduler can also choose a warp, which active threads are ready to
execute their next instruction, while another warp is waiting (this is
called in CUDA latency hiding).

 The number of block and warps and
consecutively threads, that can be processed inside a multiprocessor,
depends on the number of registers and shared memory used in the kernel,
compared to the resources available on the multiprocessor. We will see
what this means for the performance in
<a href="#chap:method" data-reference-type="ref"
data-reference="chap:method">4</a>. We want also to point out, in this
section, that a CUDA application does not have to be bounded to the data
which resides only on the GPU. The unified memory system gives the
possibility to have a single address space between CPU and GPU memory
(on newer SM architectures this means also on-demand page migration).
Moreover, CUDA's zero-copy memory permits pinned memory location on the
CPU to be accessed on the GPU. This last feature is used inside our
application to give the possibility to load volume 3D textures, which
are larger than the Device Global memory. Another feature of CUDA is the
possibility to perform atomic operations. Those operations allow
concurrent threads to perform read and write operations on data shared
in global memory.

 However, this type of operation should be used
carefully because it can potentially serialize the program in case all
the threads wants to access the same data. For what it concerns shared
memory, the system works differently. To achieve high bandwidth, shared
memory is divided into several memory modules of the same size. Those
modules are able to serve different read and write requests
simultaneously, as long as the address requested resides in different
banks. When two or more requested addresses reside on the same bank it
is called a bank conflict and the accesses are serialized. However,
there is an exception when multiple threads access the same 32-bit
world, in that case the word is broadcast to all the threads requesting
it (if multiple threads request to write, only one will write on the
address, but which one is undefined). The banks are organized such that
consecutive 32-bit words map to successive banks allowing to linear
access to not cause bank conflict.

### Kepler Architecture

In this section we will detail the Kepler architecture (which is
categorized by NVIDIA with compute capability 3). As shown in figure
<a href="/thesis/background/#fig:gpu" data-reference-type="ref"
data-reference="fig:gpu">3.3</a> and
<a href="/thesis/background/#fig:streaming-multiprocessor" data-reference-type="ref"
data-reference="fig:streaming-multiprocessor">3.4</a> in this
architecture a multiprocessor consist of 192 CUDA cores for arithmetic
operations, 64 double‐precision units, 32 special function units, 32
load/store units allowing source and destination addresses to be
calculated for sixteen threads per clock and 4 warp schedulers.
Moreover, two types of cached memory are present: a L2 cache shared by
all the processors, used to cache global and local memory accesses, and
a L1 cache, used to cache access to local memory (including register
spills). The same memory is also used for the shared memory. Each
multiprocessor has also a read-only data cache which is used for
constant or texture cache. The memory transactions are of the size of
128-byte for memory stored both in L1 and L2 cache and 32-byte if the
memory is only cached in L2.

<figure id="fig:gpu" class="figure-large">
<embed class="figure-img" src="/assets/images/thesis/tex_img_gpu.png" class="figure-img"/>
<figcaption>Kepler Architecture (compute capability 3)</figcaption>
</figure>

<figure id="fig:streaming-multiprocessor" class="figure-large">
<embed class="figure-img" src="/assets/images/thesis/tex_img_smx.png" class="figure-img"/>
<figcaption>Streaming Multiprocessor ( Kepler architecture compute
capability 3)</figcaption>
</figure>

<div class="page-nav">
  <a href="/thesis/optimization-methodology/" class="next-page">Next chapter →</a>
</div>
