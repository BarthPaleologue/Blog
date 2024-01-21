---
title: "VR With Unity #6"
author: "Barthélémy Paléologue"
type: ""
date: 2024-01-21T15:35:05+01:00
subtitle: "The final project"
image: ""
tags: ["IGD301", "VR", "Unity", "C#"]
draft: true
---

Okay the course is coming to a close and now we have to make a final project. The task is quite straightforward: select grocery items in a supermarket.

The idea is to implement one of the selection techniques of last time and test it against simple raycasting selection.

## The selection technique

As a refresher, I am going to implement the grabbable shelves idea. Here is a small video of a prototype I made reusing the code from `Roll a ball VR`:

{{< video src="intuition" >}}

The user can use its hands to rotate the shelves in whatever fashion they want. This solves the issue of objects being occluded by other objects. The user can simply change the perspective to have a clear line of sight to the object they want to select.

## The implementation

All of my implementation is available in [this repo](https://github.com/BarthPaleologue/SupermarketProject2) which is fork from our teacher's own repo.

For the project, we have access to the paid [Hyper Casual Supermaket](https://assetstore.unity.com/packages/3d/props/hyper-casual-supermarket-177794) assets from the Unity Asset Store. The repo is public so I don't know about the legality of all of this (don't sue me please), but we are using it anyway.

### Attaching items to the shelves

The first step to implement the selection technique is to attach the items to the shelves so that we can grab the shelf with the items on it.

To achieve this, we will create a grocery item script that will shoot a raycast downwards to find the shelf below it. If it finds one, it will attach itself to it.

Wait the shelves don't have colliders?

![Alt text](blinking-eyes-white-guy.gif)

Oh boy, well let's add some then.

![One hour later](image.png)

Thankfully, we can add colliders to multiple shelves at once! The only difficulty was to select all shelves without selecting any walls but it was decently fast.

I noticed that the `SelectableObject` script was automatically added to grocery items when starting the game. Therefore we can inject our attachment logic in there. Here is what I added to the `Start` method:

```csharp
// raycast downward to set parent to the shelf below
RaycastHit hit;
float distance = 1.0f;
Vector3 dir = Vector3.down;
if (Physics.Raycast(transform.position, dir, out hit, distance))
{
    transform.parent = hit.transform;
}
```

Simple enough! Now when we start the game in Unity and we edit the scene, we get the desired result:

{{< video src="parenting" >}}

### Adding highlight to shelves