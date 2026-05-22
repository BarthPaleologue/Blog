---
title: "Planetary Level of Detail #1"
author: "Barthélemy Paléologue"
type: ""
date: 2024-04-07T18:46:59+02:00
subtitle: "A technical breakdown"
image: ""
tags: ["cosmos-journeyer", "LoD"]
bigimg: [{src: "banner.png", desc: "Planetary detail at every scale"}]
draft: true
---

## Introduction

Planetary rendering is hard. The scales at play can go from the small butterfly in a patch of grass to an entire continent viewed from space. As always, computer graphics is all about finding tricks to fake a realistic look to the player.

The gist of it is that we want a lot of details where the player is looking, and save as much resources everywhere else.

I have been developing [Cosmos Journeyer](https://cosmosjourneyer.com)'s planet renderer for more than 3 years now, and I will share some of the insights I have found while doing it.

The most important trick, the one we will talk about today, is the terrain dynamic level of detail (LoD): a way to render fine details close to the player's eyes, and reducing the quality the further away we look.

## Chunking

In order to have this dynamic level of detail, we have to split our planet surface into smaller chunks that we will be able to control separately: we don't want to have the same level of detail for the entire planet. If the player lands on one side of the planet, it should not affect the terrain on the other side of said planet. 

When working on a flat surface, the most straightforward chunking method is the grid: cut the terrain into smaller squares, which can then be subdivided further in 4 new squares, and so on recursively.

TODO: an illustration of a grid chunking

As far as we know, planets are not flat (although the flat earth society has members all around the globe): the simple 2D grid won't be enough. There are many kinds of spheres, and choosing the right one will be important. Let me guide you through the 3D sphere shop!

## Finding the best sphere

How can there be multiple kind of spheres? A sphere is a sphere, that's all, why should we care?

Well the thing is, spheres do not exist in this part of computer graphics, we only do triangles! And when we stick many triangles together, we can make any shape: spheres, cubes, spacestations...

And there are multiple ways to assemble the triangles to get the shape we desire. Some will be more useful than others depending on your usecase.

### UV Sphere

The most common way to create a sphere is to take inspiration from world maps: a 2d plane that can be wrapped around a sphere like shown on this image from Blender's documentation:

{{<figure src="image-1.png" alt="UV Sphere" caption="UV Sphere" caption-position="bottom">}}

This is called a UV sphere. The mapping with the 2D surface is really useful when using textures, we can simply take the map of the earth and wrap it around like a gift. As there is no free lunch, we have some downsides:

The sphere does not have a uniform distribution of vertices, they get crowded at the poles and sparse at the equator. If we used this sphere for planet rendering, the poles would get more details than the equator. Moreover, subdividing locally the sphere's surface is not trivial.

### Icosphere

Another popular way is to start with an icosahedron:

{{<figure src="image-3.png" alt="From Wikipedia" caption="From Wikipedia" caption-position="bottom">}}

And then we subdivide each triangle into 4 smaller triangles, like a Triforce or a Sierpinski triangle depending on your cultural preferences:

{{<figure src="image-4.png" alt="https://sinestesia.co/wp/wp-content/uploads/2017/09/icos.png" caption="https://sinestesia.co/wp/wp-content/uploads/2017/09/icos.png" caption-position="bottom">}}

You can tell that the vertex distribution is much more uniform than the UV sphere, and subdivision is easy. The problem is that we subdivide the entire sphere, there are no individual chunks.

### Cube Sphere

The third popular option is to start with a cube! Quite counter intuitive I know. 

The thing is, subdividing a cube's surface is easy: 6 squares for the faces, meaning each face can be subdivided in 4 new squares, nice!

Here is our base cube:

TODO: illustration of a cube

That's nice but planets are not cubes, I hear you say. And you would be right, even though the cubic earth society as members all around the globe!

Fortunately for us, mapping the cube on the sphere can be done in one step: for each vertex, we translate it toward the center of the cube so that each vertex is at the same distance from the center:

TODO: illustration of a cube sphere

Well its not perfect, we can see the vertex density tends to be higher at the corners of the cube. The vertex distribution can be improved using some fancy math that I won't be covering today, but go take a look at [cat like coding's explanation](https://catlikecoding.com/unity/tutorials/cube-sphere/) if you are curious.

Cube spheres are a solid foundation for planetary rendering, let's get our hands dirty with the actual LoD implementation!

## Quadtrees

### Back to 2D

Let's go back to our 2D example from earlier. We can divide a square into more squares and get this kind of results:

TODO: a nice kind of result

We can use a tree-like structure to represent it in a more abstract way that will be easier to implement. We can consider the base state to be the first node of our tree (the root).

If we subdivide it one, our root gives birth to 4 child nodes that we can connect that way:

TODO: a nice illustration

And we can keep doing this multiple times:

TODO: another illustration

In a tree, the deepest nodes (those that have no children) are called the leaves. In our case, the leaves are the actual chunks of terrain that are used.

It is easy merge chunks together: take 4 children on one node, remove them so that their parent becomes a leaf: we have effectively reduced the level of detail.

### Back to 3D

Now that we understand how it works in 2D, we can go back to our cube sphere. We will not be using a single tree, but 6: one for each side of the cube. What will matter now is to choose subdivision and merging rules for our chunks.

## My mistakes

As always the devil lies in the details, and I have encountered a ton of edge cases while implementing this piece of software. I will know tell you about some of them, so that my failures can be your successes.

### LoD Oscillation

Let's say we have a distance threshold at which we decide wether to subdivide or merge chunks (let's say twice the size of the current chunk).

Let's start with a chunk of LoD `L`:

TODO: illustration of a chunk of LoD L

As the camera is close enough (below the threshold), we will now subdivide this chunk into 4 smaller ones of LoD `L+1`:

TODO: illustration of a chunk of LoD L+1

But now, the threshold has changed, and the camera is far enough (above the threshold), we will merge the 4 chunks into a single one of LoD `L`:

TODO: illustration of a chunk of LoD L

And we are back to our starting point. We can keep doing this until infinity, I call it LoD oscillation. The lesson from this mistake is that using the same distance threshold for subdivision and merging is not a good idea.

### Distance to a chunk

The next one is more subtle. How do we compute the distance to a chunk?

Well if you are like me you will just get the distance between the camera and the position of the chunk mesh and we are done.

While this works really well for very large chunks, something annoying happens when we get to a higher LoD with smaller and smaller chunk:

TODO: illustration of the discrepancy between the distance to the chunk and the distance to the surface

As you can see, at low LoD, the distance between the chunk position and the actual surface made of vertices is quite negligible compared to the size of the chunk itself. However, if you take a small chunk of mount Everest, your vertices will be displaced upward quite above the chunk base position, to a point where this distance becomes larger than the chunk size itself.

This is very problematic because you physically cannot get close enough to the chunk's actual position to trigger a subdivision, which will cause the terrain to be stuck on a coarser level and look bad.

The lesson from this mistake is that using the distance between the camera and the chunk's position is not a good idea.

## A more robust system

I spent a few hours trying to make the original system work, by computing the average height of each chunk for example, but I convinced myself that it was not worth it: I threw it all to the trash and started anew. When you cannot get rid of all the edge cases, it is a sign that your system is fundamentally flawed.

The new system creates a score for each chunk based on its distance to the camera, and the distance of the camera to the closest chunk. This score is then directly used to compute the LoD of the surface.

This new method is much more robust and I am quite happy with it.

### Great circle distances

The first part of my new system is to express that chunks further away from the camera should have a lower LoD. For this we will need a way to compute the distance between the chunk and the camera.

For this part of the algorithm, we don't care about the distance of the camera to the planet, so we will use its projected position on the sphere.

Computing distances on the surface of a sphere is different as when we are doing it on a 2d plane. The shortest path is no longer as straight line, which means the distance between two points is no longer the euclidean distance.

Instead we use the great circle distance, which is the length of the shortest arc connecting two points on a sphere. 

{{<figure src="image.png" alt="Wikipedia Image" caption="Wikipedia Image" caption-position="bottom">}}

The formula to compute it is as follow for a point `p` and a point `q` on a sphere of radius `r`:

$$
D_{c} = r \times \arccos (\frac{p}{r} \cdot \frac{q}{r})
$$

Now we can use this formula to get the distance between any chunk, and the projected position of the camera of the sphere.

The idea is that a chunk that is twice as far away from the camera should be twice as coarse. As our subdivision algorithm is based on powers of two, it means we want the LoD to be one less for each doubling of the distance.

We can express that using the powerful logarithm function: (here we use the base 2 logarithm)

$$
\text{LoD} = \text{lod}_{max} - \log_2(D_c)
$$

### Distance to the planet

We ignored the distance to the planet until now but it is very important. In fact we want our LoD to be twice as coarsed when the camera is twice as far away to the planet surface (we will be using logs again!).

But first how do we compute the distance to the planet?

We cannot choose the distance to the sphere, as the terrain surface is displayed vertically. We want to avoid the second mistake from before.

The simplest solution here is to compute the distance between the camera and the sphere which has a radius of `r` + `max_terrain_height`. This way, we can ensure the camera will always be able to get close enough to the terrain to trigger a subdivision.

When the camera is below the max terrain height, we will simply clamp the distance to 0.

This has one negative side effect: all finest chunks are loaded at the max elevation, which is technically a waste of resources as the camera is still too far to get the full advantage of the higher resolution. This is a tradeoff I am willing to make for now, but I definitely want to improve on this in the future. 

We can update our LoD formula to take into account the distance to the planet:

$$
\text{LoD} = \text{lod}_{max} - \log_2(D_c) - \log_2(D_p)
$$
