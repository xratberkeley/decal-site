---
layout: page
title: "Homework 3: Fractal Generation"
nav_order: 3
parent: Homework
---
# Homework 3: Fractal Generation
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc}

---

This homework assignment is to create your own fractal generator. It will look something like this:

![image](/assets/images/hw3/2.png)

In this assignment, we will explore recursion within Unity by writing scripts that’ll build fractals for us! Fractals pair great with recursion because recursion can exploit and represent its hierarchical and repetitive characteristic very naturally.

The fractal we want to construct for this assignment is fairly simple. Consider a single cube as the starting point. Our fractal will spawn new cubes half its size on five of its faces. Each of those cubes will then do the same thing, and so on until we reach the maximum recursion depth. Finally, every cube will slowly rotate on its y-axis, and thus children need to be oriented such that they do not clip significantly into their parent.

Consider a top down view of a parent and its direct children:

![image](/assets/images/hw3/1_1.png)

As you can see, the light grey children are half the size as the dark grey parent. Note that each child is right next to the parent - its face touches the parent’s face, but the two don’t overlap. The child in the center is no exception; it sits on top of the parent.

# Getting Started 

First off, download the skeleton asset package [**here**](https://drive.google.com/file/d/1XDZzyjZIlU7TJ6J9rWNkQZCH-DNdbPeh/view?usp=sharing) and import it into a new project. Create a new scene called HW3 to work in.

For testing purposes, we’ve provided you with a script to let you to zoom/pan your camera via mouse while in Game Mode. Attach the CameraController script to your Main Camera to do so.

Within the Main Camera’s inspector, we would also recommend that you change the Clear Flags option from “Skybox” to “Solid Color”. This will make your fractal easier to see.

![image](/assets/images/hw3/3.png)

You’ll also want to enable automatic lighting so your scene gets proper lighting. Do so through Windows > Lighting > Settings > Auto Generate.

Next, create an empty gameobject called “Fractal”, center it at the origin, and attach the Fractal script to it. You’ll notice there are some unpopulated fields in the Fractal component. Populate the fields with the following:

- **Mesh**: Cube (standard Unity asset, click the circle next to the field to select it)

- **Material**: Fractal (included in the package)

- **MaxDepth**: 1

- **Child Scale**: 0.5

- **Max Rotation Speed**: 15

Before jumping into the code writing parts of the homework, we recommend reading through Fractal.cs and understanding what the script is trying to do.

# Creating the Children

In the first section, we’ll be filling in the CreateChildren() function of Fractal.cs.

The CreateChildren() function currently runs a for-loop through the length of the childDirections array, which contain all the directions in which the children of this fractal will be growing. We’ll use this for-loop to spawn our children.

In your code, you’ll want to do the following:

Create a new gameobject.

Add a Fractal instance to the new gameobject.

Call the Initialize() function on the new Fractal.

Once you’ve successfully done this, press play and look at your hierarchy - you should see five additional gameobjects with Fractal components attached. You'll want to look into [creating new gameobjects](https://docs.unity3d.com/ScriptReference/GameObject-ctor.html) and the [AddComponent()](https://docs.unity3d.com/ScriptReference/GameObject.AddComponent.html) function.

Additional hints:

- You’d want to use the generic version of the AddComponent function.

- Pay careful attention to what’s being passed into Initialize().

- It’s possible to do all this in one line!

# Setting Children Transforms

In this section, we’ll be working on modifying the transforms of our fractal children within the Initialize() function of Fractal.cs.

In particular, you’ll have to modify the gameobject’s transform such that it matches the requirements described at the beginning of the homework. Specifically, there are three attributes you’ll want to change:

- **transform.localScale**. Use the childScale variable.

- **transform.localPosition**. Use the childDirections array and childIndex variable.

- **transform.localRotation**. Use the childOrientations array and childIndex variable. You will not be able to tell whether your rotation is correct until the next section.

Once you’ve successfully completed this section, you’ll see something very similar to the image below. Notice how there’s no visible gaps between the children and parent.

![image](/assets/images/hw3/4.png)

# Recursive Attributes

In the first section, you wrote the recursive call. You might not have realized it back then, but you did! Now, we will complete the recursive structure. Since we’re building fractals, the children will need to inherit attributes from their parents to enforce the recursive, repetitive nature of fractals.

We’ve already completed three of the attributes for you: the mesh, materials, and childScale. You must figure out the values for:

- **maxDepth**. This should be the same as its parent.

- **maxRotationSpeed**. This should be the same as its parent.

- **depth**. This should be one higher than its parent.

- **transform.parent**. This should be the transform associated with the Fractal parent.

Once you’ve done so, start by verifying that your implementation for transform.parent is correct. If you’ve implemented it correctly, your children will correctly “nest” within your parent in the hierarchy. Your hierarchy should look something like the one below!

![image](/assets/images/hw3/5.png)

Now, we can test everything else. Change the maxDepth in scene’s Fractal gameobject from 1 to 5. Assuming everything was implemented correctly, you should see something very similar to the image below.

![image](/assets/images/hw3/6.png)

Notice the beautiful nesting structure that we’ve created!

Finally, verify that your implementation for localRotation is correct. As you can observe from the image above, the children are rotating along the face of the parent. Make sure your children are doing so as well, and not rotating “into” the parent. To make it more clear, here is an example where the localRotation was set **incorrectly**:

![image](/assets/images/hw3/7.png)

Congratulations! You now have a fully operating fractal!

# Further Exploration

Now that you’ve completed the assignment, there’s lots of cool things that you can add to it. I’ve provided a few suggestions below!

- Adding support for spawning different sorts of meshes.

- Irregular spawning of meshes? (i.e. only spawns if it’s greater than a certain number?)

- Right now, our fractal has an upwards growing pattern. We can add support for it to grow towards the bottom as well.

Something cool to both try and be wary for: increasing the maxDepth! Due to how this fractal grows exponentially in size, increasing maxDepth can easily cause Unity to overload and crash. See how far you can push it!

**IMPORTANT NOTE**: You should **SAVE** before attempting to do this experiment. You may be unlucky and lose all of your work.

# Submission and Grading

Use simmer.io to submit your work to bCourses. Completion of the “operational fractal” that we guided you in creating will get 4 points (out of 5), and adding your own improvements will get you the last point. There will be extra credit given to those whom we deemed went above and beyond with their improvements.

## Credits
{:.no_toc}
Original assignment concept taken from Jasper Flick from catlikecoding.