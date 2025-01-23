---
layout: page
title: "Attachments
"
permalink: /thesis/attachments/
math: true
nav_order: 
parent: Thesis
hidden: true
---

## Digital content

<div class="description">

implementation/src : source code of the project.

implementation/external : external libraries used inside the project.

implementation/executable : executable of the software for rendering a
volume.

implementation/data : example data which can be used by the renderer

implementation/VisualStudio2015_Cuda9.0 : contains the Visual Studio
project for compiling the code. It requires Cuda9.0 to be installed.

Thesis : pdf of the thesis

scripts : scripts which permits easily to run the rendering on the
different data sets.

</div>

## Minimum System Requirements

To run the program is important to have a CUDA enabled GPU.

## Detailed Hardware Specification

The hardware that we will use for testing is a MacBook Pro with CPU
Intel Core i7 quad-core

- 2,3GHz (Turbo Boost until 3,3GHz).

- cache L3 : 6 MB.

- cache L2 per core: 256 KB.

- cache L1 per core: 32 KB.

- ram : 8 GB DDR3 1600 MHz.

and GPU Nvidia GeForce 650M with the following characteristics:

- CUDA Capability Major/Minor version number: 3.0

- Total amount of global memory: 512 MBytes (536870912 bytes)

- ( 2) Multiprocessors, (192) CUDA Cores/MP: 384 CUDA Cores

- GPU Max Clock rate: 405 MHz (0.41 GHz)

- Memory Clock rate: 2000 Mhz

- Memory Bus Width: 128-bit

- L2 Cache Size: 262144 bytes (number of registers \* 4)

- L1 Cache Size: 64 bytes ( divided between shared memory and cache)

- Maximum Texture Dimension Size (x,y,z) 1D=(65536), 2D=(65536, 65536),
  3D=(4096, 4096, 4096)

- Maximum Layered 1D Texture Size, (num) layers 1D=(16384), 2048 layers

- Maximum Layered 2D Texture Size, (num) layers 2D=(16384, 16384), 2048
  layers

- Total amount of constant memory: 65536 bytes

- Total amount of shared memory per block: 49152 bytes

- Total number of registers available per block: 65536 (equal to
  registers per SM)

- Warp size: 32

- Maximum number of threads per multiprocessor: 2048

- Maximum number of threads per block: 1024

- Max dimension size of a thread block (x,y,z): (1024, 1024, 64)

- Max dimension size of a grid size (x,y,z): (2147483647, 65535, 65535)

- Maximum memory pitch: 2147483647 bytes

- Texture alignment: 512 bytes

- Concurrent copy and kernel execution: Yes with 1 copy engine(s)

- Run time limit on kernels: Yes

- Integrated GPU sharing Host Memory: No

- Support host page-locked memory mapping: Yes

- Alignment requirement for Surfaces: Yes

- Device has ECC (error correction code) support: Disabled

- CUDA Device Driver Mode (TCC or WDDM): WDDM (Windows Display Driver
  Model)

- Device supports Unified Addressing (UVA): Yes

- Device PCI Domain ID / Bus ID / location ID: 0 / 1 / 0
