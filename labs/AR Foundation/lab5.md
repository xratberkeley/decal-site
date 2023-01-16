---
layout: page
title: "AR Foundation Lab 5: Image Tracking, Health Bar"
#nav_order: 1
parent: AR Foundation
grand_parent: Labs
---
# AR Foundation Lab 5: Image Tracking, Health Bar
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc}

---

In this lab, we’ll focus on image tracking, particularly tracking an object in the physical space so that a virtual object is overlayed on it. We'll be changing the gun from being static on screen to tracking the gun prefab on the back of a playing card. We will also be adding some more UI elements to the app as well.

Extra UI elements to be added:

- Player health bar that reduces when hit.

# Health Bar

Let's start off with the health bar.

Download this bar [**here**](https://drive.google.com/file/d/1QeEUII5GmR9BPEpLx9a2GrYDhcDINe0e/view). Add it to the textures folder in Unity, like you did with the image that you tracked in the previous section.

Change the **Texture Type** to Sprite (2D and UI).

![image](/assets/images/ar%20foundation/lab5/bar%20inspector.png)

In the textures folder, the Bar should look like this now: 

![image](/assets/images/ar%20foundation/lab5/bar.png)

Create a new **UI > Image** object in the Canvas and rename it to "Border". Take the bar sprite and drag it into the Source Image slot, and click Set Native Size to avoid any stretching.

Then, create an empty game object in Canvas called "Health Bar" and change its width and height to 52 and 9 respectively, so that it matches the size of the border. This way, we can take our border and set it as a child of Health Bar. We will have the border and the fill of the health bar inside of this Health Bar object.

To add the fill of the health bar, we **right click on Health Bar > UI > Image** and rename it to "Fill". Make sure that the Fill is above Border so that the fill shows underneath the border. Change the image color to red. In the Rect Transform anchor presets, select the bottom right option **while also holding down alt** so that it sets the position as well.

![image](/assets/images/ar%20foundation/lab5/rect%20transform%20fill_0.png)

In the Border do the same thing, but DON'T hold down alt this time; we only want to set the anchors for the border. Now when you scale the Health Bar game object, the fill and the border scale with it.

In the Health Bar game object, select the anchor preset, **while holding alt**, that sets the health bar to the top left. We also scaled the health bar so that it would be bigger.

![image](/assets/images/ar%20foundation/lab5/health%20transform.png)

Now that everything is in place, we can implement the health going down! We are going to use a slider component to decrease the fill of the health bar.

In the Health Bar game object, add a Slider component. Uncheck Interactable, set Transition to None, set Navigation to None. Drag in the Fill from the Health Bar to the **Fill Rect** area, and set the **Max Value** to 100.

![image](/assets/images/ar%20foundation/lab5/slider.png)

Try dragging the value slider left and right. You should see the red bar go up and down as you drag it! We are going to write a script to decrease the slider value as the player gets hit.

Create a script and import the UnityEngine.UI library. In this script we only need to reference the slider and write a function that sets the slider's value.

    public Slider slider;

    public void SetHealth(int health)
    {
        slider.value = health;
    }

Every time the monster attacks the player, the player loses health; we want that new health to be reflected in the health bar/slider value. We are leaving this implementation as an exercise to the reader. Think about what you need to add to the Attack() function in Monster.cs so that the slider.value matches the player's health value. You might need to implement a getter function in Player.cs.

When this is all implemented, try getting hit by the monsters! The health bar will go down as you get hit.

# Image Tracking

First off, in the AR Session Origin, add a new component **AR Tracked Image Manager**.

For our tracked image prefab, we want to use the gun, so we need to make the gun into a prefab first.

Go to **Assets > Prefabs** and drag the Gun object from the hierarchy into the project view in the prefabs folder to create a gun prefab. 

![image](/assets/images/ar%20foundation/lab5/Gun%20prefab.png)

In the hierarchy view, the gun will now have the blue box icon next to it, indicating that it is a prefab.

![image](/assets/images/ar%20foundation/lab5/gun%20prefab%20hierarchy.png)

We can delete the Gun game object from the hierarchy now. Drag the Gun prefab into **Tracked Image Prefab** in the AR Tracked Image Manager component.

Now we need to create a reference library for the images we will be tracking. For now, since only the gun will be tracked, we only need one image. For my image, I used the back of a playing card; **the more complex the image is, the better it will be for tracking**. The gun may be a little wonky though, since tracking a moving image while also moving the camera around makes it hard for the camera to properly track. In your final projects, if you are using image tracking, then it is most consistent with images and objects that are static/non-moving.

Take a picture of your card and add it to the assets > textures folder.

![image](/assets/images/ar%20foundation/lab5/texture%20card.png)

Now, to create a reference library, we go back to the prefabs folder. **Right click in the prefabs folder > Create > XR > Reference Image Library**.

![image](/assets/images/ar%20foundation/lab5/reference%20lib.png)

In the ReferenceImageLibrary, click Add Image and select the card. Specify the size as close as possible to the size of the object in the real world.

![image](/assets/images/ar%20foundation/lab5/card%20size.png)

Back to the AR Session Origin, drag the ReferenceImageLibrary into the **Reference Library** variable under the AR Tracked Image Manager component. Change the **Max Number of Moving Images** to 1.

Try running the app on your device now. The gun should appear when you bring up the card now! It should look something like this:

![image](/assets/images/ar%20foundation/lab5/screenshot.jpg)

One last thing. You may notice now that the button doesn't work to fire the gun anymore. That's because the button used to track the GameObject Gun, but now we have a Prefab Gun. Recall in the previous lab that prefabs cannot access or be accessed any gameobjects from a specific scene. This is because prefabs have to work across all scenes, and even across different projects. For that reason, they cannot use anything that's tied to a single scene. Moreover, the Gun gameobject is now being created at runtime instead of already being in the scene, so the fire button component has no way to access the game object before runtime. There are ways around this, but for simplicity's sake, we will just be going back to what we did in Lab 2 to fire - just tapping on the screen (even though we just created the fire button UI in the last lab lol).

    void Update()
        {
            if (Input.touchCount > 0)
            {
                Touch touch = Input.GetTouch(0);
                if (touch.phase == TouchPhase.Began)
                {
                    Fire();
                }
            }
        }

The gun should be tracked by the card now and fire when you tap on the screen!

*This marks the end of lab 5!*

# Final Remarks 

Congratulations! You now have a fully implemented game that you’ve created for your device in AR. There are still a ton of things that could be added and improved on, of course, and we encourage you to add stuff in your own time. We hope that this lab series gave you enough knowledge and familiarity with Unity/AR Foundation to kickstart your own final projects.