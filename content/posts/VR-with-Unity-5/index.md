---
title: "VR with Unity #5"
author: "Barthélémy Paleologue"
type: ""
date: 2023-12-18T09:31:06+01:00
subtitle: "Daunting supermarkets"
image: ""
tags: ["VR", "Unity", "IGD301", "Poking around"]
---

Have you ever been in a supermarket and felt overwhelmed by the task of grabing objects from the shelves? I know, i know.

{{<figure src="image.png" alt="The supermarket of dooooom" caption="The supermarket of dooooom">}}

Of course you have! This feeling of frustration is not uncommon for people not using advanced VR selection techniques in real supermarkets.

That is why we will explore some techniques that could be used to make your VR experience more enjoyable while shopping for groceries.

## Use camera FOV-based zoomimg

*Disclaimer: this method is hard to pull off using Unity's oculus integration. See NB at the end of the section for more information.*

Objects can be far away from the user. When that happens, it is difficult to be precise enough to grab them. We don't want to move because that would induce motion thickness though.

What if we zoomed in the camera? That would make the object appear bigger on the screen and make it easy to grab using the controllers.

We would try to implement something like the begining of this video from Cyberpunk 2077 when the eyes zoom on the washing machine:

{{< youtube s0kw5wirXWw >}}

To create this effect, we can play with the field of view (FOV) of the camera. The FOV is the angle between the left and right edges of the camera's view. The bigger the FOV, the more we can see on the screen. On the contrary, the less we can see on the screen, the bigger the remaining objects appear.

Here is an image with a FOV of 60 degrees:

{{<figure src="image-2.png" alt="FOV=60" caption="FOV=60">}} 

As you can see the items of the shelf are far away and small on the screen. Now without moving at all, by just lowering the FOV to 23 degrees, we can see the items are bigger:

{{<figure src="image-1.png" alt="FOV=23" caption="FOV=23">}}

Once the objects are magnified, it is easy to pick them using a raycast from a controller or from your eyes. Once the object is selected, it will fly right into the user's hand like you some kind of Jedi.

This solution solves picking for far away or clustered objects and is easy to implement, we just need to listen to a user input and change the FOV accordingly.

However in the case of an object being occluded, this will not be enough. Moreover, it does not allow for multiple object selection as well. That's why I have two other solutions.

To evaluate this technique, I would test accuracy, speed and motion thickness. The accuracy is straightforward, we want to see if the user if successful to select the object. As zooming might not be instantaneous depending on the implementation, speed of selection is also a concern. If zooming takes a long time it is detrimental to the whole experience. 

The motion thickness is a bit more complicated, changing the FOV of the camera does work in games like Cyberpunk 2077, but it VR it might be thickenning.

*NB: after further research, it looks like changing the FOV of the VR camera is very hard to do using Unity's Oculus integration. Even though changing the FOV directly is not possible, it can still be achievable by rendering the scene to an offscreen rendertexture and copying it to the screen. Anyhow it is way harder to implement that just changing the FOV. This limitation feels a bit strange to me, why not simply provide a default value? Using Unity's built in XR support, it is possible to use `XRDevice.fovZoomFactor`*

## Use ChatGPT

What is the solution when we have a problem in 2023? We ask ChatGPT of course! The problem is that ChatGPT (in its basic version) only understands text.

To circumvent this issue, we can listen to the user's voice and convert it to text using something like OpenAI's Whisper. But this will not be enough, we have to give ChatGPT an understanding of its environment. This can be achieved by prompt engineering where we give it positions of objects and the format we want for input and output.

In this case, we want to know what are the objects in front of the user and what are their positions.

To achieve this, we can check for the intersection of all the items with the Frustrum of the camera, this will give us only the items that are in front of the user. Using the view transformation matrix of the camera, we can also get the position of the items in the camera's reference frame.

The next step if to perform K-Clustering of the positions to group the objects in space to make them easier to understand to ChatGPT. As a matter of fact, the user may use the clustering information in its voice command, like "Give me the bottle next to the sandwiches". The clustering process ensures that ChatGPT will understand the user's own clustering. 

Once this is done, we can send the voice command and the informations we got, like this:

```
Hey ChatGPT, give me the bottle behind on the left, next to the sandwiches.

[ENV] (positions are in the camera's reference frame, depth proportional to z)
[
    clusters: [
        {
            name: bottles,
            position: [0.5, 0.5, 0.5],  
            items: [
                {
                    name: bottle1,
                    position: [0.4, 0.5, 0.5]
                },
                {
                    name: bottle2,
                    position: [0.5, 0.5, 0.5]
                },
                {
                    name: bottle3,
                    position: [0.6, 0.5, 0.5]
                },
                {
                    name: bottle4,
                    position: [0.4, 0.5, 0.6]
                },
                {
                    name: bottle5,
                    position: [0.5, 0.5, 0.6]
                },
                {
                    name: bottle6,
                    position: [0.6, 0.5, 0.6]
                }
            ]
        },
        {
            name: sandwiches,
            position: [0.3, 0.5, 0.5],
            items: [
                {
                    name: sandwich1,
                    position: [0.3, 0.5, 0.5]
                },
                {
                    name: sandwich2,
                    position: [0.3, 0.5, 0.6]
                }
            ]
        },
        ...other clusters
    ]
]
```

And then ChatGPT would answer the name of the game objects that it thinks fits the description:

```
bottle4
```

It would be wild if it actually worked right?

{{<figure src="image-6.png" alt="Wow" caption="Wow">}}

Well well, if it isn't the exact bottle we wanted! ChatGPT never fails to amaze me.

This method is not limited to single object selection, we could return different names, separated by a comma. Parsing the output of ChatGPT is then easy, as we only need to split the string on the comma to get the different names.

The selected object would then fly to your cart effortlessly. To make it feel more immersive, we could add a small drone that flies to the object and brings it back to the user when the answer is received and parsed.

Note that in order for ChatGPT to understand our formatting, we need to use a custom system prompt to prime its behavior. In this system prompt, we describe the format of the input and the output.

To evaluate this method, we need to measure accuracy and latency. ChatGPT is powerful but it is not perfect. Even though I get a few correct answers, the model would be slightly wrong most of the time. The accuracy might be better using GPT4, but I couldn't test it.

Using this kind of AI is not possible locally on a VR headset, so we would need to connect to the internet. Depending of the connection speed and the speed of the model we use, the latency might be too high to be usable for real-time interaction.

## Dynamic supermarket

All this time we have been fighting the limitations of the real world: objects can be far away, clustered and occluded. Why should we cope with our real world problems when VR gives us limitless ways to craft a better experience.

My last idea is to make the supermarket dynamic in itself. Get rid of the static shelves to make something better.

One example of dynamic shelves can be found in the home of Wall-E the robot from Pixar. In this video, look at 1:26 (I don't know how to add timings in hugo markdown):

{{< youtube IJ07DcGGmMg >}}

Wall-E compresses space efficiently by having shelves that can move in 3D space. The shelves he doesn't need are out of his reach, and he can cycle through the shelves using buttons.

This alone is a great way to remove the need for locomotion and grabbing far away objects. But we still have to deal with occlusion and clustering.

The solution to this problem is to let the user grab sub-shelves with one hand. In this system, the items are attached to their sub-shelves using parent-child relationships. When the user grabs a sub-shelf, the items are grabbed as well. 

Once the shelf is in hand, the user can easily rotate it in the same way we did for the [Roll a Ball VR](https://barthpaleologue.github.io/Blog/posts/vr-with-unity-3/) example from last time. This solves the occlusion problem by letting the user choose its perspective on the items. If the user wants an item from the back of the shelf, he can just rotate it to make the back items easily accesible to grab with the other hand.

For example this occluded chips cylinder behind the magenta bottles:

{{<figure src="image-4.png" alt="Occluded object" caption="Occluded object">}}

If we simply rotate the shelf:

{{<figure src="image-5.png" alt="No longer occluded" caption="No longer occluded">}}

Now the cylinder is perfectly visible and easy to grab for a user in VR.

In addition to that, the user can scale the sub-shelf with its items by using both hands. This lets the user pick small or clustered items more easily.

When the shelf is released, it simply goes back to its original scaling and position. This is a great way to make the supermarket dynamic and solve all the problems we had before.

This solution also works in multiplayer, as instead of moving the original shelf, we can move a copy of it. This way, the other players can still see the original shelf in its original position and use it as well.

## Conclusion

As we can see, something as trivial as selecting groceries in a supermarket is not that straightforward in VR. We need to think about the user experience and how to make it as enjoyable as possible. In the next tutorials, I will be implementing the dynamic supermarket solution and maybe combine it with the zooming camera as well if I can find a way to do it with Unity's Oculus integration.

The assets used in this article are from https://assetstore.unity.com/packages/3d/props/food/food-grocery-items-low-poly-75494