---
title: "Ray Tracing in One Weekend, in Kotlin"
date: 2023-09-27
tags: [kotlin, graphics, side-project]
summary: "Working through Peter Shirley's ray tracing book in Kotlin: small classes, a PPM file as output, and a thread pool to make the final render bearable."
---

There's a particular joy in writing a program whose output is a picture. A ray tracer is one of the best examples: a
few hundred lines of vector maths, and suddenly there are glass spheres reflecting each other on your screen.

I worked through the first book of Peter Shirley's [*Ray Tracing in One Weekend*](https://raytracing.github.io/)
series in Kotlin rather than the book's C++. The code is on GitHub:
[mohaqeq/kotlin-raytracer](https://github.com/mohaqeq/kotlin-raytracer).

![The final scene of the first book](https://github.com/mohaqeq/kotlin-raytracer/blob/8445ffff596c9a8a97093838ba22b4fc34573e8b/FirstBookFinalScene.png?raw=true)

## The shape of the code

The book's structure maps nicely onto small Kotlin files:

| File | What it holds |
|---|---|
| `Ray.kt` | A ray: origin, direction, and the point at distance *t* |
| `Hittable.kt`, `Sphere.kt` | Things a ray can hit; a sphere is the only primitive the first book needs |
| `Collision.kt` | The record of a hit: point, normal, which side of the surface, material |
| `Material.kt` | How light scatters: diffuse (Lambertian), metal and dielectric (glass) |
| `Camera.kt` | Field of view, position and defocus blur |
| `Main.kt` | The scene and the render loop |

Kotlin was a good fit. Data classes and operator overloading make the vector maths read almost like the book's
formulas (`a + b * t` instead of `a.add(b.mul(t))`). Sealed hierarchies keep materials honest.

## Output: the humble PPM

Like the book, the renderer writes a PPM file (`image.ppm`): a header followed by plain RGB triples. It's the simplest
image format there is, so there's no dependency between you and the first picture. Any image viewer, or a one-line
conversion, turns it into a PNG.

## Making the final scene bearable

The final scene of the first book, with many small spheres, antialiasing samples per pixel and several bounces per
ray, is slow when rendered on one core. Rows of pixels are independent, so the render loop hands them to a
`FixedThreadPool` sized to the number of available CPUs, then writes the rows back in order. It is the easiest speed-up
in the whole project, and a nice reminder that "embarrassingly parallel" really is a technical term.

## Running it

```bash
./gradlew run      # writes image.ppm
```

The state at the end of the first book is tagged
[`v1.0`](https://github.com/mohaqeq/kotlin-raytracer/releases/tag/v1.0).

## What I took away

- **Write the maths first, then the abstractions.** The book's order (rays, then spheres, then materials) keeps every
  step visible as a picture. You debug with your eyes.
- **Randomness everywhere.** Diffuse scattering, antialiasing and defocus blur are all random sampling. Seeing noise
  shrink as samples go up is the best intuition for Monte Carlo methods I know.
- **Kotlin is pleasant for numeric code**, as long as you keep allocations in the inner loop in mind.
