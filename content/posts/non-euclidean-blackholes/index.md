---
title: "Non Euclidean Black holes"
author: "Barthélémy Paléologue"
type: ""
date: 2024-01-12T09:03:04+01:00
subtitle: "Because why not?"
image: ""
tags: ["cosmos-journeyer", "Shaders"]
bigimg: [{src: "screenshot_24-1-12_9-07.png", desc: "Black hole"}]
---

Okay, I know what you are thinking: "Non Euclidean Black holes? What is this guy talking about?". I know, I told myself so at first, but bear with me, it is actually fun!

## What is a black hole?

A black hole is a region of space where the gravitational field get so strong the bending of spacetime becomes very noticable (and deadly, mom's spaghetti). It becomes so strong that scientists predict the bending becomes infinite at its center at what is called a singularity. This is where the laws of physics breaks down and we don't know what happens.

## Simulating a black hole

Thankfully for us, we don't care about the laws of physics at the center to be able to render a black hole. We only need to simulate the bending of spacetime around it. 

Basically, we discretize the path light takes from the observer to the black hole. At first, it will travel in a straight line, and for each step, we will bend the direction of the lightray a little bit towards the black hole depending on the distance to it. (The closer the light ray is to the black hole, the more it will bend towards it). This technique is called raymarching.

This can done by in realtime by using a raymarching shader! Even though we do a lot of approximations, the result can be quite good. This is [Cosmos Journeyer's](https://cosmosjourneyer.com) version:

{{<figure src="screenshot_24-1-12_9-07.png" alt="Alt text" caption="Alt text" caption-position="bottom">}}

For anyone wondering, I used this shader from shadertoy as a base:

{{< shader id="tsBXW3" >}}

My version basically makes the simulation works inside BabylonJS and is more accurate (I get the ~2x factor for the shadow of the black hole relative to its radius, which is the predicted value for a non rotating black hole).

I will release the code of Cosmos Journeyer around March 2024, so if you are reading this after that date, you can check it out here:

https://github.com/BarthPaleologue/CosmosJourneyer/blob/main/src/shaders/blackhole.glsl

Right now, an online demo is available here:

https://barthpaleologue.github.io/CosmosJourneyer/blackhole.html

To change the parameters, press `U` on your keyboard and click on `blackhole` in the menu that appeared at the top of the screen.

## Non Euclidean Blackholes

So now that we know what we are talking about, what is a non euclidean black hole?

Earlier, I said that for each step of our lightray simulation, we bend it towards the black hole depending on the distance.

Usually when we say "distance", what we really mean is "euclidean distance", the distance we learn in schools. 

But there are more than one distance function! I already covered this quite in depth for p-distances in my square orbit article (https://medium.com/@barth_29567/crazy-orbits-lets-make-squares-c91a427c6b26)

### P-distances (Minkowski distances)

For a refresher, here is the euclidean distance function:

$$
d(x, y) = \sqrt{\sum_{i=1}^n (x_i - y_i)^2}
$$

And here is the p-distance function (also called Minkowski distance):

$$
d(x, y) = \sqrt[p]{\sum_{i=1}^n |x_i - y_i|^p}
$$

Not a lot has changed, we replaced the 2s by ps essentially. But this is enough to change the shape of our black hole!

Here is a black hole with a p-distance of 1 (Manhattan distance):

{{<figure src="screenshot_24-1-12_9-02.png" alt="p=1" caption="p=1" caption-position="bottom">}}

We get the shape of a diamond for the event horizon! This is because the p-distance of 1 is the sum of the absolute values of the differences between the coordinates.

What about p=2?

{{<figure src="screenshot_24-1-12_9-26.png" alt="p=2" caption="p=2" caption-position="bottom">}}

Haha got you, p=2 is just the regular euclidean distance, so we get the same result as before, with a nice circular event horizon.

Okay what about p=5?

{{<figure src="screenshot_24-1-12_9-30.png" alt="p=5" caption="p=5" caption-position="bottom">}}

We get a cubic event horizon! In the same way as increasing p in the orbit articles gave us square orbits, increasing p here gives us a cubic event horizon in the 3rd dimension.

We could actually have predicted this shape by looking at the wikipedia page for Minkowski distances: https://en.wikipedia.org/wiki/Minkowski_distance

We get these shapes in 2D, and we indeed got the same shapes in 3D!

{{<figure src="image.png" alt="Shapes of Minkowski distances" caption="Shapes of Minkowski distances" caption-position="bottom">}}

Let's do one more: p=0.5

{{<figure src="screenshot_24-1-12_9-36.png" alt="p=0.5" caption="p=0.5" caption-position="bottom">}}

The maths checks out!

That's pretty cool, can we find other distances that give us other shapes?

### Signed distance fields

Yeeees! Signed distance fields are functions that give the distance to a certain shape. For example, using the SDF of a capsule, we can bend the rays in a wonky way to get a black hole with a capsule event horizon!

{{<figure src="screenshot_24-1-12_10-13.png" alt="Wtf am I doing" caption="Wtf am I doing" caption-position="bottom">}}

Unfortunately, we are only bending the rays around the capsule and not really raymarching the capsule directly so we don't get a very definite shape.

Indeed, if we use the SDF of a cylinder instead, we get a similar result:

Side view:
{{<figure src="screenshot_24-1-12_10-18.png" alt="Side view" caption="Side view" caption-position="bottom">}} 

Top view:
{{<figure src="screenshot_24-1-12_10-18 (1).png" alt="Top view" caption="Top view" caption-position="bottom">}}

Just for fun, here the result with the SDF of a pyramid:

{{<figure src="screenshot_24-1-12_10-21.png" alt="Big forehead" caption="Big forehead" caption-position="bottom">}}

You probably never thought about a black hole with a huge forehead, but here we are.

For reference I used the SDFs from this shadertoy:

{{< shader id="Xds3zN" >}}

## Conclusion

As we see, we can always get interesting results when we question the assumptions we make (here the definition of the distance). It is a good way to get something that look realistic, and yet feels new and interesting.

There are probably ways to get more definite shapes for the event horizon, and if I have a good idea I will post a follow up to this article. Maybe we can get a mandelbulb black hole? That would be awesome!

That will be all for now, I hope you enjoyed this article! Thanks for reading!

