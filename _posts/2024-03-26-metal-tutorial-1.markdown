---
layout: post
title:  "Metal Tutorial 1"
date:   2024-03-26 19:04:14 +0100
categories: metal tutorial swift
---

# Metal Tutorial 1 

In this tutorial, we will learn how to set up a Metal project and render a simple triangle on the screen.

## Prerequisites

A basic understanding of swift, and graphics programming is required.
There are already a lot of resources that can help to start with the basics, here some suggestions:

- [Swift Programming Language](https://docs.swift.org/swift-book/)
- [Learn OpenGL](https://learnopengl.com/)
- [Metal by Example](https://metalbyexample.com/)
- [Metal Programming Guide](https://developer.apple.com/documentation/metal)
- [scratchapixel](https://www.scratchapixel.com/)
- [Ray Tracing in One Weekend](https://raytracing.github.io/books/RayTracingInOneWeekend.html)
- [Physically Based Rendering](http://www.pbr-book.org/)
- [Real-Time Rendering](http://www.realtimerendering.com/)
- [Computer Graphics: Principles and Practice](https://www.elsevier.com/books/computer-graphics/hearn/978-0-12-415791-3)

If you are still here 🤗, clone the repository from the [GitHub repository](https://github.com/Fe0437/MetalTutorials).

## Setting up the project

Open the file MTMetalTutorialsApp.swift and ensure that the view is set up correctly with the MT1ContentView.

```swift
@main
struct MetalTutorialsApp: App {
    var body: some Scene {
        WindowGroup {
            // substitute here to choose the tutorial
            MT1ContentView()
        }
    }
}
```

now run the project, and you should see this window:

![MT1ContentView](/assets/2024-03-26-metal-tutorial-1/MT1ContentView.png)
