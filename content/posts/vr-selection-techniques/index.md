---
title: "VR Selection Techniques"
author: "Barthélemy Paléologue"
type: ""
date: 2023-12-10T18:22:38+01:00
subtitle: "An overview of 3 selection techniques in VR"
image: ""
tags: ["VR", "IGD301"]
---

## Raycasting from the eyes

In mainstream VR, the most common selection technique is raycasting from the controller. It is simple and cheap to implement, but having to move the hands around can be tiring.

{{<figure src="image.png" alt="https://journals.sagepub.com/doi/10.1177/21695067231192429" caption="https://journals.sagepub.com/doi/10.1177/21695067231192429" caption-position="bottom">}}

Using your gaze allows for a very natural way to select distant object (the reach is infinite). It is used in newer headsets such as the Apple Vision Pro that take advantage of cutting edge hardware.

It is used for single object selection as you can't select multiple objects at once with your eyes (its cardinality is single). Moreover, you can't refine your selection by only using your eyes. This can be combined with hand tracking to allow progressive refinement.

## Flashlight

An evolution of raycasting is "cone casting" or "flashlight". It is a raycast with a cone shape. This is very different from raycasting as we have now a selection volume instead of a point. However, the further away objects are, the larger the cone will be, making it harder to only select the object you want.

As the flashlight is a variant of the raycast, its reach is infinite. However, contrary to the raycast, it has a volume allowing for multiple object selection (its cardinality is plural). As you can't control the angle of the cone, there is no progressive refinement. This is why Aperture was created.

## Aperture

The aperture is an evolution of the flashlight. The main difference being that you can adjust the angle of the cone to refine your selection (progressive refinement). This allows for a very precise selection of objects.

{{<figure src="image-1.png" alt="Design and evaluation of a novel out-of-reach selection technique for VR using iterative refinement" caption="Design and evaluation of a novel out-of-reach selection technique for VR using iterative refinement" caption-position="bottom">}}

In the same way as the flashlight, the aperture has an infinite reach and a plural cardinality. However as you can adjust the angle of the cone, you can select objects one by one as well with a small enough angle. Therefore the cardinality can be both plural and single.