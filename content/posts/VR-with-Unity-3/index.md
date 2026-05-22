---
title: "VR with Unity #3"
author: "Barthélemy Paléologue"
type: ""
date: 2023-12-10T16:22:21+01:00
subtitle: "Your first VR game on Oculus Quest"
image: ""
tags: ["VR", "Unity", "C#", "IGD301"]
---

Let's do some real VR development! The school lent me an Oculus Quest, so I will be doing the next tutorials with it.

## What are we doing today?

Okay so last week we created a simple game, following the `Roll a ball` tutorial from Unity. The ball was controlled using the keyboard, but today we will control it by grabbing the game board with our hands, and moving it around to make the ball roll!

I will be going through the entire Unity setup process for Oculus Quest so that you can follow along.

## Developer mode

The first thing to do is to setup the headset in developer mode. I will not detail every step of the setup process as it is well documented online.

To become a developer, you need to create an organization on Meta's website: https://developer.oculus.com/manage/organizations/create/

Then verify your account here: https://developer.oculus.com/manage/verify/

Now, restart your headset and you should have in `Settings` > `System` a new option called `Developer`.

Go in it and enable `USB Connection Dialog`, this will allow us to connect the headset to our computer using the USB-C cable.

Now when you plug the headset to your computer, you should have a popup on the headset asking you to allow the usb debugging. Accept it.

## Install SideQuest

In order to install our apps on the headset, we will use SideQuest. It is by far the largest project for sideloading apps on the Oculus Quest and is cross-platform.

Go to https://sidequestvr.com/setup-howto and download the advanced installer for your OS.

Once that's done, run Sidequest and connect your headset to your computer.

On the top right corner of SideQuest, you should see your headset connected:

{{<figure src="image-3.png" alt="Your headset is connected" caption="Your headset is connected" caption-position="bottom">}}

## Test your setup

Now to check that our config is working, we will upload a test APK on the headset and run it. 

I found this github repo that contains a simple APK to test the setup:

https://github.com/tater-tot25/VR_Projects_Taster

Now in sidequest, click on this little button:

{{<figure src="image.png" alt="Upload APK to headset" caption="Upload APK to headset" caption-position="bottom">}}

Now select the test APK and SideQuest will install it on your headset (this will take a few seconds).

Now in the headeset, go to the app library, select `Unknown Sources` in the top right of the screen. All of our applications will show in this section. For now we only have our test.

When I launched it, I saw this:

{{<figure src="com.Tate.HistProject-20231210-171411.jpg" alt="Alt text" caption="Alt text" caption-position="bottom">}}

If you have something similar, then your setup is working! Let's create our own VR app now!

## Go back to Unity

Let's make ourselves a Unity setup for VR development. We can use the same project as last time, but we will need to change a few things.

### Android build target

As Oculus Quest is an Android device, we will need to change the build target to Android. Go to `File` > `Build Settings` and select `Android` as the build target.

You may need to install it, once that's done click on `Switch Platform`.

### Oculus Integration

Now let's head to the Asset Store [here](https://assetstore.unity.com/packages/tools/integration/oculus-integration-deprecated-82022) and add the Oculus Integration package to your assets. (You will need to login to your Unity account).

It is deprecated, but it works, so we will be using it anyway.

In Unity go to `Window` > `Package Manager` and change the `Packages` tab to `My Assets`. You should see the Oculus Integration package. Click on `Download` then `Import in Project`.

Now, change `My Assets` to `Unity Registry` and search for `Oculus XR Plugin`. Download it and import it in the project as well.

### Project Settings

Go to `Edit` > `Project Settings` > `Player` > `Other Settings` and change the following settings:

- The color space should not be `Gamma` but `Linear`.

- Disable `Auto Graphics API` and remove `Vulkan` from the api list.

In `XR Plugin Management` > `Android` > `Plugin Providers`, check `Oculus`.

This is all the config we will need for now so you can save your project.

## Make a small VR scene

In our project, let's create a new scene called `VRScene`. In this scene, let's create a simple cube and a plane at the origin.

For the plane, in the inspector go to the `Mesh Collider` and check `Convex`.

The plane will be our ground and the cube some kind of desk. We will also wrap them in an empty game object called `Ènvironment`.

Because we imported `Oculus Integration` earlier, we have access to a lot of prefabs that will help us create our scene.

In `Assets` > `Oculus` > `VR` > `Prefabs`, drag and drop the `OVRCameraRig` prefab in the scene.

Now select it and in the inspector, change the `Tracking Origin Type` to `Floor Level`.

In the `display` section, change the color gamut to `Quest 1`.

In the same `Prefabs` folder, drag and drop the `OVRControllerPrefab` prefab in the scene.

Expand the  `OVRCameraRig`, then expand the `TrackingSpace` then expend `LeftHandAnchor` to find `LeftControllerAnchor`. Drag and drop the `OVRControllerPrefab` to the `LeftControllerAnchor` to make it a child of it.

{{<figure src="image-1.png" alt="EXPAND" caption="EXPAND" caption-position="bottom">}}

In the inspector, change the `Controller` to `LTouch`.

Do the same for the right hand, but this time change the `Controller` to `RTouch`.

Save the scene and go to `File` > `Build Settings`. Click on `Add Open Scenes` and then `Build`. You will get an APK that you can install on your headset using SideQuest.

When launching your app, you should see your controllers in your hands and the cube in front of you.

## Import Roll a Ball

Last week, we made a simple game using the Roll a boll tutorial from Unity.

Let's open the Roll a ball scene and put the `Ground`, `Player`, `Walls`, `Collectibles` and `Canvas` in an empty game object called `GameBoard`.

We also need to rename the `PlayerController` script to avoid name collisions with the Oculus Integration package. I renamed it `BallController`. In this script, we must comment everything related to the keyboard input as we will not use it and it would cause errors.

Now we drag it to our assets to make it a prefab that we will use in our VR scene.

In our VR scene, let's add the `GameBoard` prefab and move it to the origin. We will need to scale it down a bit, the size of the box for example (you can adjust the size to your liking by testing it on the headset).

I ended up with something like this:

{{<figure src="com.DefaultCompany.IGD301-20231210-141755.jpg" alt="Game board on desk" caption="Game board on desk" caption-position="bottom">}}

We don't see the UI though, to fix that in the hierachy select the canvas of the `GameBoard` prefab and in the inspector, change the `Render Mode` to `World Space`. Scale it down so that it fits above the board. Now you can see the UI in the scene view. (It might not be super visible because it is white on white), try changing the color of the text to black.

{{<figure src="com.DefaultCompany.IGD301-20231210-143100.jpg" alt="Gameboard with UI" caption="Gameboard with UI" caption-position="bottom">}}

## Grabbing the game board

### Board collider

We would like our player to grab the game board and move it around. To do that, we will need to add a few things to our scene.

Select the `GameBoard` prefab and add a `Box Collider` to it. Edit the collider so that it fits the board. (I put the center to 0 and the size to `21`, `2`, `21`).

As the objects inside the board have their own colliders, we need to use a different layer to avoid collision conflicts.

Next to the `Tag` field in the inspector, we find the `Layer` field. Click on `Add Layer...` and add a new layer called `Selection`.

When prompted, choose `This object only` to apply the layer to the `GameBoard` prefab only.

Add another layer called `rollaball` and apply it to the children of the `GameBoard` prefab (the `Ground`, `Player`, `Walls`, `Collectibles`). When prompted, choose `Yes, Change Children`.

For the `ground` object of the board, we must check `Convex` in the `Mesh Collider` component. By default, the convex collider will be too big, making our ball hover above the ground. To fix that, we need to change the `y scale` of the `ground` object to 0.1 (visually it will not change anything but the convex collider will be thinner).

Back to the `GameBoard` prefab, we add a `Rigidbody` component that will use `gravity` and wont be `kinematic`.

### Hand colliders

Now that we configured the board, we need to add colliders to the hands of the player.

Let's head to the `OVRCameraRig` prefab of our scene and edit `LeftHandAnchor` and `RightHandAnchor`.

We will add `sphere colliders` to the hands (a radius of 0.1 should be enough). Make sure that the colliders are triggers. Also, make sure that the colliders are on the `Selection` layer. When prompted, choose `This object only`.

We also need `rigidbodies` on the hands. Make sure that the rigidbodies are `kinematic` and don't use `gravity`.

### Collision Matrix

We don't want the objects from the `rollaball` layer to collide with the `Selection` layer (they are inside the board). We will edit the layer collision matrix to avoid that.

We will find it in `Edit` > `Project Settings` > `Physics` > `Layer Collision Matrix`.

Uncheck the `Selection` layer for the `rollaball` layer:

{{<figure src="image-2.png" alt="The layer collision matrix" caption="The layer collision matrix" caption-position="bottom">}}

### Selection script

Let's create a new script called `BoardSelect` and attach it to the `LeftHandAnchor` and `RightHandAnchor` of the `OVRCameraRig` prefab.

Let's some variables to the script, one to know if the player selected something, one to store the object that is selected, one to know if the hand is in the collider of the board and one to store the VR controller.

```csharp
private bool isControllerInCollider = false;
private bool isObjectSelected = false;
private GameObject selectedObject = null;

public OVRInput.Controller controller;
private float triggerValue;
```

Then we will add 2 functions `OnTriggerEnter` and `OnTriggerExit` that will be called when the hand enters and exits the collider of the board. We will be using the name of the game object, so be careful to name the game object `GameBoard`.

```csharp
void OnTriggerEnter(Collider other)
{
    if (other.gameObject.name == "GameBoard")
    {
        isInCollider = true;
        selectedObject = other.gameObject;
    }
}

void OnTriggerExit(Collider other)
{
    if (other.gameObject.name == "GameBoard")
    {
        isInCollider = false;
        selectedObject = null;
    }
}
```

*NB: Instead of checking the name of the game object, we could use the layer of the collider and check if it is the `Selection` layer. As layers are integers and might not be the same on different projects, I chose to use the name of the game object instead.*

To properly handle the selected object, we will need to change the `Update` function to check if the player is grabbing something and if so, move it to the position of the hand.

First we get the trigger value from the controller like so:

```csharp
triggerValue = OVRInput.Get(OVRInput.Axis1D.PrimaryIndexTrigger, controller);
```

Then we must handle different cases:
- If the player is not grabbing anything and the trigger is pressed, we select the object and make it follow the hand.
- If the player is grabbing something and the trigger is released, the object is dropped to the ground.

All of this only happens if the hand is in the collider of the board.

```csharp
if (isControllerInCollider) {
    // Select object
    if(!isObjectSelected && triggerValue > 0.95f) {
        isObjectSelected = true;

        // Make the object a child of the controller
        selectedObject.transform.parent = transform;

        // Disable physics to follow the controller
        Rigidbody rb = selectedObject.GetComponent<Rigidbody>();
        rb.isKinematic = true;
        rb.useGravity = false;

        // Reset velocity
        rb.velocity = Vector3.zero;
        rb.angularVelocity = Vector3.zero;
    } 
    // Deselect object
    else if (isObjectSelected && triggerValue < 0.95f) {
        isObjectSelected = false;

        // Remove the object from the controller
        selectedObject.transform.parent = null;

        // Enable back physics
        Rigidbody rb = selectedObject.GetComponent<Rigidbody>();
        rb.isKinematic = false;
        rb.useGravity = true;

        // The object inherits the velocity of the controller (we can throw it)
        rb.velocity = OVRInput.GetLocalControllerVelocity(controller);
        rb.angularVelocity = OVRInput.GetLocalControllerAngularVelocity(controller);
    }
}
```

We can now build the project and upload the APK to the headset using SideQuest.

## Conclusion

If everything went well, you should have something similar to this:

{{< video src="result" >}}

Congratulations on making your first VR game!

All projects files are available on GitHub under a free license at https://github.com/BarthPaleologue/UnityVR
