---
layout: page
title: "Implementation Details
"
permalink: /thesis/implementation-details/
math: true
nav_order: 
parent: Thesis
hidden: true
---

In this chapter we are going to cover some details on the implementation
of the previously described algorithms. Software design is particularly
difficult in CUDA, as to achieve the maximum performance all the
branching decision must be made as early as possible. Furthermore, there
is no possibility to use virtual functions, abstraction and classic
polymorphism. We will see in the next section how we can deal with those
constraints and how to create a as generic as possible volumetric path
tracer in CUDA. The solution proposed is the base of the implementation
provided in the attachment
<a href="#attach:digital-content" data-reference-type="ref"
data-reference="attach:digital-content">digital-content</a>.

## Generic Programming

To improve code re-usability and flexibility many of the most important
CUDA libraries like CUB and thrust, which we are also employing inside
the project, use Generic Programming. This technique leverages the power
of C++ templates to create a compile-time abstraction layer. That is,
all the callback or virtual classes are substituted by a template. This
template will represent a data structure which must contain all the
functions and members requested. In the case of virtual functions this
is done with the use of Functors: data structure which define the
operator `()`. In all the implementations of the volumetric path tracing
provided this technique is used. The only template which must be defined
is the Scene. This template requires to define an intersection
structure, called `SceneIsect`, which contains all the information about
the intersection with the scene. From the scene should be also possible
to access the medium and the BSDF based on the information on the
intersection structure. Sampling of the distance with the woodcock
tracking <a href="#chap:theory" data-reference-type="ref"
data-reference="chap:theory">3</a> and of the albedo is also done using
a template Medium. For this reason, the implementation of the kernels
are completely general and re-usable. Every type of scene can be
substituted to the template without requiring any change for the kernel.
Moreover, this system allows to create custom device scene and to
improve the performance of the method based on the type of scene. In
figure <a href="/thesis/implementation-details/#fig:sw_design" data-reference-type="ref"
data-reference="fig:sw_design">5.1</a> is shown a part of the software
architecture relative to the kernel composition. The right kernel with
the right scene is selected at run-time by a `RenderFactory` which
creates the renderer based on a configuration object.

<figure id="fig:sw_design" >

<embed class="figure-img" src="/assets/images/thesis/tex_charts_sw_design.png" />

<figcaption> kernel launch software architecture. The figure shows the
method used to create generic implementation of the volumetric path
tracing without affecting the performance. Concretely, different
implementation are compiled for each different variant of the algorithm.
The template interfaces permits to do that without affecting
re-usability of the code.</figcaption>
</figure>

## Scene Assembler

The software provides a configuration system that is flexible and easy
to extend. All the configurations for all the algorithms are stored
inside a single `Config` object, which is used to initialize all the
renderer. This configuration object, however, can be populated using
different systems. In this moment, the only interface with a user
wanting to use the software is represented by the command line. The
commands provided by the user are analyzed by a `ConfigParser` object
which has the ability to parse and create a `Config` object. The command
line is used principally for configurations relative to the algorithm
used for rendering. For the configurations relative to the scene,
another object is used: the `SceneBuilder`. The `SceneBuilder` is an
interface which allows to load different scene types without affecting
the normal work-flow of the renderer. In this moment, there are two
implementations of this interface: the `XmlSceneBuilder` and the
`MhaSceneBuilder`. The first one allows to load the medium from a
Mitsuba Xml scene file, while the second one from a VTK mha volume
format. Which one of those builders to use is chosen by the
`ConfigParser` object which selects the right one based on the command
line argument provided by the user. Finally, the `SceneAssembler`
provides an interface between the `SceneBuilder` and the `ConfigParser`
and is also responsible for the creation of the `Scene` object, which
will be placed inside the final configuration object of the renderer.
The figure <a href="/thesis/implementation-details/#fig:assembler" data-reference-type="ref"
data-reference="fig:assembler">5.2</a> shows a charts of the system just
explained.

<figure id="fig:assembler">

<embed src="/assets/images/thesis/tex_charts_sceneassembler.png" width="60%" />

<figcaption> Configuration Architecture. In this image is shown the
system used to permit the loading of different scene formats inside the
software. Moreover, the SceneBuilder interface provide an extension
point for the loading of even more formats.</figcaption>
</figure>

## Interactive Renderer and Transfer Delegation

There are two main configurations to run the software: testing mode and
interactive mode. The testing configuration works only by command line
and allows to benchmark one of the algorithms presented in the previous
chapter. The user can specify the number of trials and the software will
run the algorithm the number of times specified, returning the the mean
time, standard deviation and rays/sec for the algorithm on the given
scene. Instead, the interactive mode uses the GLFW library, which is an
OpenGL multi-platform library, to create a new window where the scene is
rendered one iteration for every frame. The interactive renderer works
as an additional layer upon the normal renderer. There are different
objects which allow to decouple the different elements of the system:

- `GLViewController`: allows to create the main window. By running the
  OpenGL main loop, it draws the pixel buffer object on the window and
  controls the `InputController` and the `BufferProcessorDelegate`.

- `InputController`: is an interface which allows to use the GLFW events
  to modify the scene. An implementation is the `CameraController` which
  allows to orbit the camera around the center of the object during the
  rendering.

- `BufferProcessorDelegate`: is an interface which allows to modify the
  OpenGL pixel buffer object. The only implementation provided is the
  `CudaInteractiveRenderer` which takes the normal renderer and uses it
  to render the next frame on the OpenGL pixel buffer object. To make
  this possible, the object leverages the power of the interoperability
  between CUDA and OpenGL.

However, there is a difference between the normal renderer and
interactive renderer and it depends on the output of the renderer. In
one case the output must be transfered in the host memory, while in the
other one it should be transfered in another buffer always inside the
device memory, so to use the interoperability between CUDA and OpenGL.
For this reason, the `CudaVolPath` uses a delegate object to transfer
the output buffer from the source to its destination. The interface of
this delegate is called `Buffer2DTransferDelegate` and three
implementations are provided in the software:

- `HostImageBufferTansferDelegate`: it is the basic method which
  transfers the buffer from the device memory to the host memory.

- `DeviceImageBufferTansferDelegate`: this transfer delegate transfers
  the buffer from the device memory to device memory.

- `DeviceTiledImageBufferTansferDelegate`: works exactly like the
  previous one but it allows to update the image one tile per frame.

Those transfer delegates can take as an argument a Functor, which must
be used on the output image before the transfer is complete. In the
`CudaVolPath` this transformation allows to scale the rendering output
by the number of iterations of the Monte Carlo estimation. Also the
transfer delegate is chosen by the `RenderFactory` at the beginning,
based on the launching configuration provided by the user. All the
system is shown in figure
<a href="/thesis/implementation-details/#fig:interactive" data-reference-type="ref"
data-reference="fig:interactive">5.3</a>.

<figure id="fig:interactive" >

<embed class="figure-img" src="/assets/images/thesis/tex_charts_outputdelegate.png" />

<figcaption> Interactive Renderer Architecture. The interactive renderer
has been created without compromising the generality of the algorithm.
The algorithm created can be used inside a BufferProcessorDelegate which
is providing the data necessary to the GLViewController for rendering
the image frame by frame.</figcaption>
</figure>

## Zero-Copy Volume

Storing all the volume texture inside the device memory is not always
possible. Sometimes the volume is too big and it cannot fit inside the
memory available in the hardware. For this reason, the software allows
to store the volume data inside the host memory. It is clear that this
is not a good idea considering that the bandwidth between CPU and GPU is
much lower than the bandwidth with the DRAM. The render will be, in this
case, much slower on accessing the volume, while with the use of the
texture cache this will happen only if there is a cache miss. In the
newer devices (compute capability 6.x or higher) this can be implemented
more efficiently using the managed memory, that has inside a page fault
mechanism, for which memory pages can be stored inside the device local
memory. However, our target architecture is the Kepler (compute
capability 3.0), so the software does not use this technique. Instead,
the zero copy memory is used, this technique allows to use a page locked
memory on the host directly inside the device. Unfortunately, it is not
possible at the moment to create a CudaArray, meaning memory layouts
that are optimized for local texture fetching, with this type of memory.
Therefore, we create the texture using the linear memory pointer. This
has the disadvantage of not exploiting the three-dimensional data
locality, inside the texture making useless algorithms, like the
previously described sortingSK.

<div class="page-nav">
  <a href="/thesis/discussion/" class="next-page">Next chapter →</a>
</div>
