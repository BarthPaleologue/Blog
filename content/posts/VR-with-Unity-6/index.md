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

As a refresher, I am going to implement grabbable shelves. The user should be able to select a shelf using raycast and have it fly in its hand where manipulation and picking with the other hand will be much easier. Here is a small video of a prototype I made reusing the code from `Roll a ball VR`:

{{< video src="intuition" >}}

As the user can use its hand to rotate the shelves in whatever fashion they want, it solves the issue of objects being occluded by other objects. The user can simply change the perspective to have a clear line of sight to the object they want to select.

In this blog post I will cover the implementation of the technique inside Unity. It will be divided into sections:

- Shelf highlighting
- Attaching items to shelves
- Item highlighting
- Shelf manipulation
- Item picking
- Teleportation

The evaluation of the technique will be done in the next blog post (spoilers, it is really good).

All of my implementation is available in [this repo](https://github.com/BarthPaleologue/SupermarketProject2) which is fork from our teacher's own repo.

For the project, we have access to the paid [Hyper Casual Supermaket](https://assetstore.unity.com/packages/3d/props/hyper-casual-supermarket-177794) assets from the Unity Asset Store. The repo is public so I don't know about the legality of all of this (don't sue me please), but we are using it anyway.

## Shelf highlighting

First, we have to take care of our shelves to make them compatible with the technique. We want a simpler collider so that items can be automatically attached to their shelf using a downward raycast that filters only shelves. For that we will create a layer called `Shelves` (for me it is layer 8).

There are a lot of shelves though, we aren't doing this by hand right?

Normally, we would be able to do it using scripts but the shelves do not have a script attached to them at the moment... Bummer!

Oh boy, well let's select them all by hand then.

![One hour later](image.png)

Okay so now that we have everything selected, we can add a box collider to all of them at once, assign them to the `Shelves` layer and finally attach a `Shelf` script to them.

Now we are pretty much free to do whatever we please! We will start by adding a transparent box around the shelves that will help the user to select them later on.

To make the highlighting work, we will create a new box that we will scale so that in covers the shelf horizontally and englobes the items vertically. We will make it transparent and allow to change the color when the shelf is set to highlighted.

We also need to create another layer just for theses highlights as they will play the role of a large bounding box for the shelf. I called it `ShelvesHighlights` and is my 3rd layer.

```csharp
GameObject highlightBox;
private Color originalHighlightColor = new Color(1, 1, 1, 0.1f);
private Color selectedHighlightColor = new Color(0, 1, 0, 0.5f);    
[SerializeField] private bool isHighlighted = false;

void Start() {
    // get the size of the shelf
    Vector3 shelfSize = this.GetComponent<Renderer>().bounds.size;

    // create a new box
    GameObject box = GameObject.CreatePrimitive(PrimitiveType.Cube);

    // set layer of box to "ShelvesHighlights" (layer 3)
    box.layer = 3;

    // position the box at the same position as the shelf
    box.transform.position = this.transform.position;

    // set the height of the box
    float boxHeight = 2.25f;
    Vector3 boxSize = new Vector3(shelfSize.x, boxHeight, shelfSize.z) + new Vector3(0.01f, 0.01f, 0.01f);
    box.transform.localScale = boxSize;

    // move the box up by half its height
    box.transform.position = new Vector3(box.transform.position.x, box.transform.position.y + boxHeight / 2 - shelfSize.y / 2, box.transform.position.z);

    // set the material of the box to be transparent
    material = new Material(Shader.Find("Transparent/Diffuse"));

    // change the color of the box
    material.color = originalHighlightColor;

    // set the material of the box
    box.GetComponent<Renderer>().material = material;

    // make the box a child of the shelf
    box.transform.parent = this.transform;

    this.highlightBox = box;
}

public void SetHighlighted(bool highlighted) {
    isHighlighted = highlighted;
}

void Update() {
    // change the color of the box based on the highlighted state
    if(isHighlighted) {
        material.color = selectedHighlightColor;
    } else {
        material.color = originalHighlightColor;
    }
}
```

Now we get the following result:

![Shelf highlighting](image-4.png)

You can increase the opacity of the highlights as you wish, but in VR I found this to be enough to tell the boundaries of the shelf.

## Attaching items to the shelves

Next we want to items to stick to their shelf to allow fine manipulation. We will achieve that by using parenting at startup.

All items have attached the `SelectableObject` script that we are going to modify to add the parenting logic.

The idea is that for each object, we shoot a ray downward and filter only the shelves. This should give us the nearest shelf below each object. Then we can simply set the parent of the item to the shelf and we should be good.

```csharp
// raycast downward to set parent to the shelf below (layer 8)
RaycastHit hit;
float distance = 2.0f;
Vector3 dir = Vector3.down;
if (Physics.Raycast(transform.position, dir, out hit, distance, 1 << 8))
{
    transform.parent = hit.transform;
}
```

It is important that you choose the right layer! Mine is 8 but yours could be different.

Now when we start the game in Unity and we edit the position of one shelf, we get the desired result:

{{< video src="parenting" >}}

## Item highlighting

In the very same fashion as we did for the shelves, we will create a highlight box for the items. This is necessary because the selection technique will not use the exact geometry of the items, so it could be disturbing for the user.

More importantly, there is the sandwich problem.

![SCARY SANDWICHES](image-5.png)

You are scared right? No? You should be!

This is a tricky edge case of box colliders, here both sandwich have a box collider that englobes the whole sandwich. Yet, as they are placed in this fashion, their bounding boxes are almost entirely overlapping. This means that one of the 2 sandwiches will be extremy hard to select as the raycast will hit the other one first.

Of solution could be to use a finer collider, but that would increase the complexity of calculations on the CPU and the original Quest doesn't have a lot of lee-way for this supermarket scene.

Instead we will use a highlight so that the user knows which objects he is about to select. This feedback mechanism will help reducing the error rate dramatically.

We will write the highlight logic inside the `SelectableObject` script. As the item already has a box collider, we will need to disable the additional box collider provided by the highlight box.

```csharp
// create a new box to representing the box collider (same scale and orientation as the object, with bounds of the box collider)
GameObject box = GameObject.CreatePrimitive(PrimitiveType.Cube);
box.transform.localScale = this.GetComponent<BoxCollider>().size + new Vector3(0.01f, 0.01f, 0.01f);
box.transform.rotation = this.transform.rotation;
box.transform.position = this.transform.position + this.GetComponent<BoxCollider>().center;

// set the box to be a child of the object
box.transform.parent = this.transform;

// remove the box collider from the box
Destroy(box.GetComponent<BoxCollider>());

// set the material of the box to be transparent
Material material = new Material(Shader.Find("Transparent/Diffuse"));
material.color = new Color(1, 1, 1, 0.3f);
box.GetComponent<Renderer>().material = material;
box.SetActive(false);

this.boundingBoxHelper = box;

public void DisplayBoundingBox(bool display) {
    this.boundingBoxHelper.SetActive(display);
}
```

We get the desired result:

![Item bounding box](image-6.png)

And now we can tell apart the sandwiches!

![The sandwich problem, solved!](image-7.png)

## Selecting the shelves with raycasts

We are now entering the core of the project. We want the user to select the shelf using a raycast.

Thankfully, the raycast selection is already implemented in the `RaycastTechnique.cs` file. We will simply copy its content inside `MyTechnique.cs` and modify it to our needs. 

To make the project use our technique instead of the raycast technique, we will simply change the `Interaction Type` of the `TaskManager` game object to `MyTechnique` in the Unity editor.

We will start easy by highlighting the shelf when clicking on it to check everything is working fine.

Basically, we will check the tag of the raycasted object. If we hit a `shelfHighlight`, then we know its parent gameobject is a shelf. We will then set the `isSelected` boolean of the shelf to `true` to highlight it.

```csharp
// Creating a raycast and storing the first hit if existing
RaycastHit hit;
bool hasHit = Physics.Raycast(rightControllerTransform.position, rightControllerTransform.forward, out hit, Mathf.Infinity);

// Checking that the user pushed the trigger
if (OVRInput.Get(OVRInput.Axis1D.PrimaryIndexTrigger) > 0.1f && hasHit)
{
    GameObject hitObject = hit.collider.gameObject;

    // if hit object has tag "shelfHighlight" then its parent has a Shelf component and must be selected
    if (hitObject.tag == "shelfHighlight")
    {
        GameObject shelf = hitObject.transform.parent.gameObject;
        shelf.GetComponent<Shelf>().isSelected = true;
    }

    // Sending the selected object hit by the raycast
    // currentSelectedObject = hitObject;
}
```

We can test it and see the following result:

{{< video src="shelfSelection1" >}}

Now let's go further and make it highlight the shelf only when the user is hovering over it.

We need another variable to keep track of the currently hovered shelf so that we can disable the shelf highlight when the user is not hovering over it anymore.

```csharp
if(!hasHit) {
    // if we are not hitting anything, we should unselect the shelf we were hovering over
    if(hoveredShelf != null) {
        hoveredShelf.isSelected = false;
        hoveredShelf = null;
    }
} else {
    // if we are hitting something, we should select the shelf we are hovering over
    GameObject hitObject = hit.collider.gameObject;
    if(hitObject.tag == "shelfHighlight") {
        GameObject shelf = hitObject.transform.parent.gameObject;
        if(hoveredShelf != null && hoveredShelf != shelf) {
            hoveredShelf.isSelected = false;
        }
        hoveredShelf = shelf.GetComponent<Shelf>();
        hoveredShelf.isSelected = true;

        // Checking that the user pushed the trigger
        if (OVRInput.Get(OVRInput.Axis1D.PrimaryIndexTrigger) > 0.1f)
        {
            // we will do something here later
        }
    }
}
```

That's much better:

{{< video src="shelfSelection2" >}}

We can do the same with the left controller by duplicating the raycast code and adding the controller prefab to the left anchor of the `OVRCameraRig` game object.

## Shelf manipulation

Now that we have everything in place, we can start manipulating the shelves. When the shelf is hovered and the user presses the trigger, we want the shelf to follow the user's hand.

This means we will also need to save the shelf original position and rotation to put it back when the user releases the trigger.

{{< video src="shelfManipulation1" >}}

{{< video src="shelfManipulation2" >}}

## Item picking

Now that we can handle shelves with one hand, we will use the other hand to pick the wanted item from the shelf. We will use raycasting again to select the item.

One issue is that we want the other hand to be able to select both items on the current shelf or other shelves to change the current shelf.

What we will do is first make a raycast for all shelves highlights. If we hit the one that is currently manipulated, we will then make a raycast for all items on the shelf. If we hit one, we will select it.