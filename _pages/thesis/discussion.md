---
layout: page
title: "Discussion
"
permalink: /thesis/discussion/
math: true
nav_order: 
parent: Thesis
hidden: true
---

In this chapter we will discuss about some of the results we have
reached. We will compare the render time, achieved with the methods
explained in the chapter
<a href="#chap:method" data-reference-type="ref"
data-reference="chap:method">4</a>, with a renderer commonly used by the
Computer Graphics research community: the Mitsuba renderer by .
Moreover, we will discuss some correlation between the scenes and the
best performing method.

## Results

In table <a href="/thesis/discussion/#tab:mitsuba1" data-reference-type="ref"
data-reference="tab:mitsuba1">6.1</a> we can see the first comparison of
the Mitsuba renderer with the scenes presented in
figure <a href="/thesis/optimization-methodology/#fig:scenes" data-reference-type="ref"
data-reference="fig:scenes">4.6</a>. There are multiple algorithms that
can be used inside the Mitsuba renderer to render the scene, however the
algorithm that works best in this type of scene is the simple volumetric
path tracing (called volpath_simple). Contrary to our methods, this
algorithm is also using shadow rays. For this reason, the total number
of rays traced for this algorithm is calculated by adding the shadow
rays and the normal rays traced. The results show that the number of
rays traced every second by the Mitsuba volumetric path tracer is much
lower than any other algorithm. In the scenes used in this comparison
the density is constant and the algorithm that is performing better is
the regenerationSK using the single thread regeneration. The reason
being that if the density is constant, also the standard deviation over
the length of a path that traverses the medium will be small. Thus, the
rays are starting and stopping by themselves all together and there is
no need for extra care on synchronizing the threads or compacting them.
The other conclusion that is possible to take is that the GPU is
performing much better than the CPU algorithm also considering the naive
single kernel approach.

<div id="tab:mitsuba1">
<table>
<thead>
</thead>
<tbody>
<tr><td align="center">method</td><td align="right">rays/sec</td><td align="right"></td></tr>
<tr><td align="center"></td><td align="right">cgg-logo</td><td align="right">ad</td></tr>
<tr><td align="center">naiveSK</td><td align="right">3.88</td><td align="right">2.13</td></tr>
<tr><td align="center">regenerationSK (thread)</td><td align="right">81.62</td><td align="right">42.37</td></tr>
<tr><td align="center">regenerationSK (warp)</td><td align="right">4.37</td><td align="right">7.02</td></tr>
<tr><td align="center">streamingSK (compaction)</td><td align="right">52.69</td><td align="right">37.81</td></tr>
<tr><td align="center">streamingSK (sorting)</td><td align="right">20.41</td><td align="right">17.61</td></tr>
<tr><td align="center">sortingSK</td><td align="right">20.90</td><td align="right">17.83</td></tr>
<tr><td align="center">mitsuba (volpath_simple)</td><td align="right">0.003</td><td align="right">17.83</td></tr>
</tbody>
</table>
comparison with Mitsuba renderer. This table shows the comparison of the
algorithm showed in the thesis with a CPU volumetric path tracing
implementation provided by . The table shows that the CPU algorithm is
performing worse in all the case. The two measures provided are the
millions of rays traced every second and the total time for rendering a
400x400 image with the scene</div>

<span id="tab:mitsuba1" label="tab:mitsuba1"></span>

Let’s consider now a scene with varying density but not varying albedo,
like the one in figure <a href="/thesis/discussion/#fig:hetvol" data-reference-type="ref"
data-reference="fig:hetvol">6.1</a>. The results relative to this scene
are presented in the table
<a href="/thesis/discussion/#tab:mitsuba2" data-reference-type="ref"
data-reference="tab:mitsuba2">6.2</a>. The table shows the millions of
rays per second and the total time to render the smoke scene with
density scale factor 800 which corresponds to the scaling factor of the
0 to 1 density volume. The results are showing that in this case the
algorithm that is performing better is the streamingSK. The explanation
can be found in the long duration of some rays inside the medium
compared to other rays that are ending immediately. In this case, the
streaming method is able to group together the long term rays and
decrease the divergence of the warps. This results in a better
efficiency of the streaming algorithm when the scenes present high
varying density. However, the complexity of the density function is not
the only factor to take in consideration.

<figure id="fig:hetvol">
<img src="/assets/images/thesis/tex_img_hetvol.png" style="width:50.0%" />
<figcaption>smoke scene: heterogeneous volume representing smoke. The
file which have grid resolution of (128,128,50) can be found in the
website of the Mitsuba renderer <span class="citation"
data-cites="Mitsuba"></span>. The density scale used in this scene is
800.</figcaption>
</figure>

<div id="tab:mitsuba2">
<table>
<thead>
<tr><th align="center">method</th><th align="right">rays/sec</th><th align="right">time (sec)</th></tr>
</thead>
<tbody>
<tr><td align="center">regenerationSK (thread)</td><td align="right">2.52</td><td align="right">131</td></tr>
<tr><td align="center">regenerationSK (warp)</td><td align="right">7.35</td><td align="right">127</td></tr>
<tr><td align="center">streamingSK (compaction)</td><td align="right">17.41</td><td align="right">53.68</td></tr>
<tr><td align="center">sortingSK</td><td align="right">14.64</td><td align="right">63.83</td></tr>
<tr><td align="center">mitsuba (volpath_simple)</td><td align="right">0.58</td><td align="right">1076</td></tr>
<tr><td align="center">mitsuba (volpath)</td><td align="right">0.54</td><td align="right">1195</td></tr>
</tbody>
</table>
comparison with Mitsuba renderer on the smoke scene. This table shows
the behavior of the algorithms showed in this work in the case the scene
present high varying density and high resolution.</div>

<span id="tab:mitsuba2" label="tab:mitsuba2"></span>

The scene in figure <a href="/thesis/discussion/#fig:bucky" data-reference-type="ref"
data-reference="fig:bucky">6.2</a> presents an high varying density but
a small texture resolution. In this case the methods that are using
compaction are performing worse than the RegenerationSk method with a
single thread regeneration as it is showed in the table
<a href="/thesis/discussion/#tab:comparison3" data-reference-type="ref"
data-reference="tab:comparison3">6.3</a>. A possible reason for this
behavior can be attributed to the ratio between traced paths and grid
resolution. That is, in this type of scenes if the number of rays is
high enough it is more probable that groups of rays access the same
texture data. That happen because the texture resolution is much lower
respect to the previous cases. If threads that are close together access
the same texture values they are behaving similarly to the case of
constant density factor and exactly like in that case a simple method,
like the regenerationSK, that fully utilize the GPU gives the best
performance gain.

<figure id="fig:bucky">
<img src="/assets/images/thesis/tex_img_bucky.png" style="width:50.0%" />
<figcaption>bucky heterogeneous volume with grid resolution of
(32,32,32). The file is usually used for testing volumetric rendering
because it presents an high varying density. In this test the density
values are varying between 0 and 40. For the color a static transfer
function is applied which maps green to low density values, red to
medium density values and blue to high density values. The number of
iterations used for the Monte Carlo estimation is 300.</figcaption>
</figure>

<div id="tab:comparison3">
<table>
<thead>
<tr><th align="center">method</th><th align="right">rays/sec</th><th align="right">time (sec)</th></tr>
</thead>
<tbody>
<tr><td align="center">regenerationSK (thread)</td><td align="right">10.96</td><td align="right">39.71</td></tr>
<tr><td align="center">streamingSK (compaction)</td><td align="right">5.75</td><td align="right">52.83</td></tr>
<tr><td align="center">sortingSK</td><td align="right">5.69</td><td align="right">53.30</td></tr>
<tr><td align="center">naiveSK</td><td align="right">4.99</td><td align="right">60.86</td></tr>
</tbody>
</table>
comparison of the GPU rendering algorithms detailed in the chapter
<a href="#chap:method" data-reference-type="ref"
data-reference="chap:method">4</a> on a high varying density scene in
figure <a href="/thesis/discussion/#fig:bucky" data-reference-type="ref"
data-reference="fig:bucky">6.2</a> with many density holes and small
resolution</div>

<span id="tab:comparison3" label="tab:comparison3"></span>

## Which is the Best?

Looking at the results that we have showed, we can say that the best
algorithm to use for GPU volumetric path tracing closely depends on the
type of scene we want to render. We have seen that for complex scenes
with high varying density and high resolution texture the more
sophisticated algorithm which uses compaction and smart regeneration are
performing better. On the other hand, if our scene is a simple scene,
simple methods which requires less synchronization between threads and
less branch divergence are getting better results.

<div class="page-nav">
  <a href="/thesis/conclusion/" class="next-page">Next chapter →</a>
</div>
