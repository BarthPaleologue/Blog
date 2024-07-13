---
title: "Making Shaders Faster #1"
author: "Barthélemy Paléologue"
type: ""
date: 2023-12-18T18:48:24+01:00
subtitle: "Planetary rings"
image: ""
tags: ["Cosmos Journeyer", "Shaders", "BabylonJS"]
bigimg: [{src: "rings.png", desc: "Planetary rings"}]
---

Realtime shader programming is challenging. Although the GPU is very fast, when adding more and more shaders, you always reach a point where your GPU is no longer fast enough. This is where optimization comes into play.

Shaders can be optimized in 2 ways: improve the shader code itself, which can give decent improvements, and precomputing data, which can make the difference between 20fps and 500fps.

In this blog post series, I will be presenting how I optimized the shaders of [Cosmos Journeyer](https://cosmosjourneyer.com): a procedural universe running in the web browser.

## Noise is expensive

To perform many effects in computer graphics, we rely on noise functions:  randomness with smooth transitions. Here is what it can look like:

{{<figure src="./noise.png" alt="Noise" caption="Noise" caption-position="bottom">}}

src: https://barthpaleologue.github.io/Noise-Engine/dist/

The issue is that we use A LOT of noise, and it is very costly to generate at runtime. That is why we will see how to precompute it in various contexts.

## Planetary rings

{{<figure src="./rings.png" alt="Planetary rings" caption="Planetary rings" caption-position="bottom">}}

Let's start slowly our optimization journey with the planetary rings. To create the ring effect, we use a noise function that computes the ring density for every pixel visible of the rings. Although I made it using a postprocess, you could use a real mesh with a shader material that performs the same thing inside the fragment shader.

### Finding symmetries

Symmetries are our friends when it comes to conquering optimization problems. In a more general way, the more assumptions you can make about a problem, the more you can optimize it.

In the case of the planetary rings, you may notice that the density function has a radial symmetry. If we take 2 small slices of our rings (like we would do with a pie), we would see that they are identical. The density function has no dependency on the angle around the pllanet, only the distance to its center.

As our density function is just a 1D function, it is possible to store the density values in an array with the index of the array representing the distance to the center of the planet.

In computer graphics, we do not use plain arrays, but textures (which are just a 3D array that are mostly used for images: width x height x color channels). If we create a texture of dimensions width x 1 x 1, we have something equivalent to the 1D array I mentionned earlier.

After precomputing the rings look up texture, we get something like this:

{{<figure src="./ringsLUT.png" alt="rings lut" caption="rings lut" caption-position="bottom">}}

We can intuitively see that rotating this texture around the planet will indeed give us the same ring system as before.

### How to generate the texture

We will now dive inside the actual code implementation that I made to generate the picture above.

First problem with the implementation: ditching our density function that uses the distance to the center of the planet to using a texture that only takes a sampling coordinate between 0 and 1 is not straightforward. We will need to do some remapping.

As the rings can be defined by the distance where they start, and the distance where they end. We can remap this distance range to [0;1] easily using a remapping function:

```glsl
// remap a value comprised between low1 and high1 to a value between low2 and high2
float remap(float value, float low1, float high1, float low2, float high2) {
    return low2 + (value - low1) * (high2 - low2) / (high1 - low1);
}
```

The second issue is not specific to the lookup texture but to any noise sampling function in general: we can't use the raw distance to the center of the planet to sample our noise function. Noise functions in computer graphics tend to give better results when the sampling coordinates are small. Going too far from the origin results in artefacts related to the function itself, and plain floating point imprecisions.

This is an issue because working on planetary scales means having to deal with distances in the order of millions of meters (even billions sometimes).

One solution to this problem is to use the distance to the center of the planet, divided by the radius of the planet, which gives us dimensionless values that are not too big (it probably won't exceed 20).

In the main ring shader, we convert the raw distance to the relative distance then to [0;1], and in the texture generation code, we can get back our relative distances by using the same remapping but in reverse.

This allows us to reuse the existing density function without changing anything:

```glsl
precision highp float;

// texture coordinates
varying vec2 vUV;

uniform float seed; // noise offset for unique rings
uniform float frequency; // noise frequency
uniform float ringStart; // relative distance where the ring starts
uniform float ringEnd; // relative distance where the ring ends

// you can use any 1D noise function here with Fractal Brownian Motion
#include "../utils/noise1D.glsl";

#include "../utils/remap.glsl";

void main() {
    // reversed remapping
    float relativeDistance = remap(vUV.x, 0.0, 1.0, ringStart, ringEnd);

    // layer noise to get a more interesting result
    float macroRingDensity = noise(fract(seed) + relativeDistance * frequency / 10.0, 1, 2.0, 2.0);
    float ringDensity = noise(fract(seed) + relativeDistance * frequency, 5, 2.0, 2.0);
    ringDensity = mix(ringDensity, macroRingDensity, 0.5);

    // fade out the ring at the start and the end
    ringDensity *= smoothstep(ringStart, ringStart + 0.03, relativeDistance);
    ringDensity *= smoothstep(ringEnd, ringEnd - 0.03, relativeDistance);

    // accentuate density gradient
    ringDensity *= ringDensity;

    gl_FragColor = vec4(ringDensity, 0.0, 0.0, 0.0);
}
```

Here is a live version in shadertoy:

{{< shader id="McsGD4" >}}

To use the texture to compute the density at any point we can simply convert the raw distance to the relative distance, then to the texture coordinate:

```glsl
float ringDensityAtPoint(vec3 samplePoint, vec3 object_position, float object_radius, float rings_start, float rings_end) {
    vec3 samplePointPlanetSpace = samplePoint - object_position;

    float distanceToPlanet = length(samplePointPlanetSpace);
    float relativeDistance = distanceToPlanet / object_radius;

    // out if not intersecting with rings and interpolation area
    if (relativeDistance < rings_start || relativeDistance > rings_end) return 0.0;

    float uvX = remap(relativeDistance, rings_start, rings_end, 0.0, 1.0);
    float lutDensity = texture2D(rings_lut, vec2(uvX, 0.0)).x;

    return lutDensity;
}
```

## Conclusion

And that's pretty much it for the rings. The same look up texture can be used to make the rings' shadows faster. Next up will be the clouds which have a little bit more going on.