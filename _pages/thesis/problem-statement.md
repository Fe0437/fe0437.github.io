---
layout: page
title: "Problem Statement
"
permalink: /thesis/problem-statement/
math: true
nav_order: 
parent: Thesis
hidden: true
---

The chapter describes the problem that the thesis aims to solve and
describe more in detail the different characteristics of Participating
Media.

## Radiative Transport Problem

The problem that the thesis aims to solve is a radiative transport
problem and the radiometric quantity, that we are searching for, is the
radiance $L$. This quantity describes how much light arrives from a
direction $\omega$ on an hypothetical differential area perpendicular
to that direction $dA^\bot$ .

<div id="eq:Ldef"></div>
$$
\begin{aligned}
L=\frac{d^2\Phi}{d\omega\, \,dA^\bot}
=\frac{d^2\Phi}{d\omega\, \,dA\,\cos\theta} \,
\label{eq:Ldef}
\end{aligned}
$$

Where ${\Phi}$, expressed in Watt (Joule/sec), is the radiant flux,
i.e time rate of flow of radiant energy. When this light energy interact
with a participating medium, than it can scatter or be absorbed. In both
cases the light energy is attenuated. This attenuation is usually called
extinction (see ).

$$
\text{Extintion = scattering + absorption}
$$

In this type of problems a distinction is done between solid objects,
which have a well defined boundary, and other media, like gases or
liquids. In this work, we are concentrating on the first type of media,
considering also cases where the object boundary defines a change on the
index of refraction of the medium.

## Participating Media

We can think of a participating media as an agglomerates of small
particles. Each of those particles can be described by the following
characteristics identified by :

- $C_{sca}$ (scattering cross section): is the area of the particle
  which is scattering light;

- $C_{abs}$ (absorption cross section): is the area of the particle
  that is absorbing light;

- $F$ (scattering diagram of the particle): describes the scattering
  behavior and it can be used to obtain the more easily manipulable
  phase function $f_p$.

Given those characteristics, the final medium is described by the
density function $\rho(x)$ of these particles for any point $x$
inside the medium. When a photon arrives to a point $x$, inside a
medium which contains some particles, four types of event can happen:

<div class="description">

<strong>Absorption</strong>: light is absorbed by the particles in that point $x$,
proportionally to their $C_{abs}$; the amount of light absorbed is
defined as the absorption coefficient $\sigma_a(x) [m^{-1}]$.

<div id="eq:absorption"></div>
$$
\begin{aligned}
\label{eq:absorption}
\sigma_a(x) = C_{abs} \cdot \rho(x)
\end{aligned}
$$

<strong>Out-scattering and In-scattering</strong>: light can be out-scattered or
in-scattered in the point $x$ , based on the $C_{scat}$ of the
particles in that point; the amount of light scattered is defined as the
scattering coefficient $\sigma_s(x) [m^{-1}]$.

<div id="eq:scattering"></div>
$$
\begin{aligned}
\label{eq:scattering}
\sigma_s(x) = C_{scat} \cdot \rho(x)
\end{aligned}
$$

<strong>Emission</strong>: finally the medium can emit more energy in the point
$x$; we will call the radiance emitted by the medium in a point $x$
and direction $\omega$, $L_e(x, \omega)$.

</div>

There are also some derived quantities important to describe
interactions with a participating media. Those are the extinction
coefficient $\sigma_t$, defined as

<div id="eq:extinction"></div>
$$
\begin{aligned}
\label{eq:extinction}
\sigma_t(x) = \sigma_a(x) + \sigma_s(x)
\end{aligned}
$$

and the albedo of the medium $\alpha$, defined as

<div id="eq:albedo"></div>
$$
\begin{aligned}
\label{eq:albedo}
\alpha = \frac{\sigma_s}{\sigma_t}.
\end{aligned}
$$

It is important to notice that it is always possible to derive the first
two quantities in equations
<a href="/thesis/problem-statement/#eq:absorption" data-reference-type="ref"
data-reference="eq:absorption">absorption</a> and
<a href="/thesis/problem-statement/#eq:scattering" data-reference-type="ref"
data-reference="eq:scattering">scattering</a> from the last two
equations <a href="/thesis/problem-statement/#eq:extinction" data-reference-type="ref"
data-reference="eq:extinction">extinction</a> and
<a href="/thesis/problem-statement/#eq:albedo" data-reference-type="ref"
data-reference="eq:albedo">albedo</a>, and vice versa. By
integrating the extinction coefficient along a line segment $l$, we
get the optical thickness of our material $\tau$:

$$
\begin{aligned}
\tau(l) = \int_l\sigma_t(x)dx.
\end{aligned}
$$

Now, we have all the elements to completely describe the change of
radiance inside a medium using the radiative transport equation (RTE),
which can be defined with the following equation:

<div id="eq:RTE"></div>
$$
\begin{aligned}
\label{eq:RTE}
(\omega \cdot \nabla) L(x,\omega) = L_e(x,\omega) + \sigma_s(x)L_i(x,\omega) -\sigma_a(x)L(x,\omega) -\sigma_s(x)L(x,\omega).
\end{aligned}
$$

where $L_i(x,\omega)$ is the in-scattered radiance at $x$, through
the direction $\omega$, coming by other scattered radiance. The
equation <a href="/thesis/problem-statement/#eq:RTE" data-reference-type="ref"
data-reference="eq:RTE">RTE</a> includes all the light interactions
that we have previously described, which are in this order: emission,
in-scattering, absorption, out-scattering.

## The Volume Rendering Equation (VRE)

Scenes, containing solid participating media, are usually modeled as a
volume $\mathcal{V}$ and a boundary $\partial \mathcal{V}$, where
$\mathcal{V} \cap \partial \mathcal{V} = \emptyset$. As shown by , the
transport can be described differently, based on the position considered
on the volume. If the point is on the surface $\delta\mathcal{V}$, we
use the classic Rendering Equation

<div id="eq:rendering"></div>
$$
\begin{aligned}
\label{eq:rendering}
L(x,\omega) = L_{e,\delta{\mathcal{V}}}(x,\omega) + \int_{S^2}f_s(\omega,x,\omega')L(x,\omega')|cos\theta_x|d\sigma(\omega'):
\end{aligned}
$$

where $S^2$ is the set of all directions (in this work we are using
only the hemispherical formulation), $f_s$ is the bidirectional
scattering distribution function, which describes the scattering
behavior at a point $x$ on the surface (in particular is the fraction
of incident differential radiation reflected into the direction
$\omega'$). Finally, $cos\theta_x$ is the cosine of the angle
between direction $\omega'$ and the surface normal at $x$ (the
reader interested on more informations about this equation can find its
complete definition in the work of ). On the other hand, if the point is
inside the participating media we have to consider the <em>Volumetric
Rendering Equation</em> (VRE). This equation is obtained integrating the
radiative transport equation, that we have described in the previous
paragraph, along straight light rays until the next intersection point
$x_s$ of the ray with a surface. The equation is:

<div id="eq:volumetric-rendering-equation"></div>
$$
\begin{aligned}
\label{eq:volumetric-rendering-equation}
L(x,\omega) = \int_0^s T(x,x_t) \cdot L_v(x_t, \omega) dt + T(x,x_s) \cdot L(x_s,\omega)
\end{aligned}
$$

where

<div id="eq:volumetric-source"></div>
$$
\begin{aligned}
\label{eq:volumetric-source}
L_v(x, \omega) = L_{e,{\mathcal{V}}}(x,\omega) + \sigma_s(x) \int_{S^2} f_p(\omega,x,\omega')L(x,\omega')d\sigma(\omega')
\end{aligned}
$$

and $T(x,x_t)$ is the transmittance function defined by the
<em>Beer-Lambert-Bougeuer law</em> as

$$
\begin{aligned}
T(x,x_t) = e^{-\tau(||x-x_s||)}
\end{aligned}
$$

with $\tau$ the previously described optical thickness of the medium.

A more general equation, which combines both the equations
<a href="/thesis/problem-statement/#eq:rendering" data-reference-type="ref"
data-reference="eq:rendering">rendering</a> and
<a href="/thesis/problem-statement/#eq:volumetric-rendering-equation" data-reference-type="ref"
data-reference="eq:volumetric-rendering-equation">volumetric-rendering-equation</a>,
is the path integral formulation. In this formulation the space of
integration is the union of spaces containing paths with specific
length, i.e $\mathcal{P} := \bigcup_{k\in\mathbb{N}} \mathcal{P}_k$,
where $P_k$ is the space of paths ${\overline{x}}$ of length $k$.
The reader interested can find its derivation in the work of .

From a mathematical point of view, the volume rendering equation is a
Fredholm integral equation of the 2nd kind. Solving this type of
equation analytically is very difficult, even applying simplifications
on medium and light transport. Many numerical estimations methods have
been studied in the literature. Those methods can be divided in two
groups: deterministic methods and stochastic methods. The first ones are
based on a discretization of the domain of integration and the solution
of large linear systems. However, this type of system is usually
affected by some limitations and the recent research has focused more on
the second types of methods. In the next chapter, we are going to cover
exactly this latter type of methods, giving an overview on some of the
most important ones.

## Porting to GPU

GPU-based programs have a number of limitations on when and how memory
can be accessed. Computation speed increases at a much faster rate than
memory access speeds, which means that, to improve time performance,
special care should be taken to maximize bandwidth usage. Simulating
light transport, inside participating media, can also be not obvious.
For this scope, one of the most used techniques in the industry of
computer graphics is the Monte Carlo path tracing. This is a stochastic
method which uses a random process to correctly estimate the light
interactions in the medium. The random nature of this method makes the
utilization of GPU difficult in this context. GPU is best suited to well
predefined tasks with the minimum control decisions made at runtime.
Random behavior of different GPU threads can lead to decreased
utilization of the device. If a GPU is not utilized at its maximum
processing power, the overhead of transferring data and control to it
can make the GPU implementation slower than a CPU one.

## Limitations

This work is subject to different restrictions which limits the
generality of the results obtained:

- we will not consider possible changes of refraction index inside the
  medium itself;

- our tested objects are rendered in the vacuum, not considering
  participating media outside the volume;

- inherited from the scattering theory used, we will consider only
  independent scattering, meaning that we will consider only
  participating medium which have well-defined separate particles, and
  single scattering, which means that the concentration of particles of
  the medium considered is proportional to the total light intensity
  scattered (further explanation inside the work of  );

- we will not consider wave effects and only consider ray optics;

- we have tested our implementation only on one GPU architecture and we
  address only the usage of CUDA enabled GPU.

<div class="page-nav">
  <a href="/thesis/related-work/" class="next-page">Next chapter →</a>
</div>
