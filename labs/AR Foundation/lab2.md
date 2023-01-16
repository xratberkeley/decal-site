---
layout: page
title: "AR Foundation Lab 2: Building the Gun"
#nav_order: 1
parent: AR Foundation
grand_parent: Labs
---
# AR Foundation Lab 2: Building the Gun
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc}

---

In this lab, we’ll focus completely on the gun. By the end of the lab, you should be able to fire the gun by pressing on the screen, and when the gun fires, there is an animation, sound effects, and particles coming from the gun.

As an extra lab, we also change the gun from being static on screen to being tracked on the back of a playing card. If you’re interested in experimenting with image tracking, we encourage you to do the extra lab as well.

# Adding the Gun

Now we can create the gun. We want the gun to be statically on our screen as we move around our device. As such, we want the gun to be tied to the AR Camera.

Start by right-clicking the AR Camera game object (under AR Session Origin) in the hierarchy and clicking “Create Empty”. This will create an object with nothing but a transform component, which we’ll add to and flesh out over the course of these labs. Rename the empty GameObject to “Gun”, and reset its transform (right click on the transform component > reset) if it is not in default position - we’ll move it later.

Go into the Models folder and find the object called “makarov”. This is the 3D model for our gun. Go ahead and drag it onto our newly created empty object, which will parent it under Gun.

![image](/assets/images/ar%20foundation/lab2/gun%20hierarchy.png)

Parenting is a concept in Unity that allows us to construct complex objects out of simpler ones and have them all move together as one. An object (call it the child) parented under another object (call it the parent) will have all its transform values defined as relative to that of its parent. This means when the parent moves, scales, or rotates, its child will change along with it.

Let’s rename this newly childed object to “Model”. It’ll start out being far, far too large, so we select its parent, Gun, and scale each of its axes to 0.025. We can move the Gun object around in the scene, which should now move the 3D model around as well. Position/rotate gameobject Gun so it’s about here on the game screen:

![image](/assets/images/ar%20foundation/lab2/gun%20game%20scene.png)

We used this transform on the “Gun” game object:

![image](/assets/images/ar%20foundation/lab2/gun%20transform.png)

Depending on your device’s camera/screen, the best gun positioning will vary. It is also possible to script the gun object so that it is scaled in relation to the device’s view so that the gun, on any device, will be in the same position, but we are not going to bother with that in these labs.

Let’s try to build the project on our device now.

Do you notice anything a little off? You may notice that the camera is zoomed in and the gun is closer than it is on the Game scene in Unity. That is why we have the gun more close to the center than to the right of the screen.

The "zoomed" look of the camera image might be the result of the AR Camera Manager. AR Foundation’s camera rendering applies additional image scaling so that the camera image can entirely fill the device screen. This scaling is necessary (and mathematically correct) in order to avoid image distortion due to the different aspect ratio of cameras with respect to the screen. It also avoids black side bands (i.e. letterboxing) from appearing at the top / bottom (or left/right) of the rendered image.

But otherwise, the scene on your device should have the gun in it now!

# Firing the Gun

We’ve finally reached the most exciting part of the lab: firing the gun. In a future lab, we will add a button so that when the button is pressed, it will fire; but for now we will program it so that the gun fires when we tap on the screen.

Create a new script called "Gun" and attach it to your Gun gameobject. We want to fire the gun every time we tap on the screen.

    public class Gun : MonoBehaviour {
        ... 

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
    }

Now, we have to write our Fire() function. Let’s list out all the things that need to happen when we fire the gun:

- Gunshot sound plays.
- Recoil animation plays.
- Gunshot explosion visual effect (VFX) plays.

All three of these things will require use of different Unity components.

# Sound

In Unity, add an AudioSource component to gameobject Gun. AudioSources are what play sounds in Unity. Turn off “Play On Awake” since we don’t want to hear a gunshot every time we start the game. Then assign the clip “Gunshot” to the Audio Clip field (you can either drag it in from the Sounds folder or click the circle on the right).

![image](/assets/images/ar%20foundation/lab2/gun%20inspector.png)

Next, in Gun.cs, create a private variable of type AudioSource called audioSource. Then in Start(), initialize that value to reference the component we just created.

    public class Gun : MonoBehaviour {
        ...
    
        AudioSource audioSource;
    
        // Use this for initialization
        void Start () {
            audioSource = GetComponent<AudioSource>();
        }
    ...

Now that we have a reference to the component, we can use it in our Fire() function. To play the clip attached to an AudioSource, call audioSource.PlayOneShot(). Unity also has a function called audioSource.Play(), but it can’t play multiple sounds simultaneously. This would be problematic as we can fire the gun faster than the length of the gunshot clip.

    public void Fire() {
        audioSource.PlayOneShot(audioSource.clip);
    }

We can try to build the app now. Firing the gun now should give you a satisfying sound effect.

# Animation

We won’t be getting into the details of animation in this class. If you wish to learn more about it, I recommend first going through some of Unity’s videos on the topic [here](https://unity3d.com/learn/tutorials/topics/animation). Instead I’ll give a quick overview of the system.

On a high level, Unity’s animation system consists of two parts: controllers and motions. Motions (also called animations) are pieces of data that show how a particular object changes over time; they’re made in either Unity or some other software like Maya or 3DS Max. Controllers, on the other hand, are animation state machines; they contain the logic that dictates what animation plays when, based off of a set of external flags called parameters.

Parameters are defined in the controller and can be accessed/modified via script. This is mostly how we’ll be interacting with the animation system during the labs.

Take a look at the Animations folder and double-click the “Gun” controller to open up the Animator window.

![image](/assets/images/ar%20foundation/lab2/9.png)

Note there are only two “real” states that are connected, Idle and Gun-Anim, and if you click on the Idle state you’ll see it doesn’t have a motion attached in the inspector. The Gun-Anim state does, however: the recoil animation. The green “Entry” state points to what state we start off on.

Next, look at the arrow going from Idle to Gun-Anim. This is a transition, and tells us that our Gun could possibly move from the Idle to Gun-Anim state.

In the inspector, you’ll see a box labeled “Conditions”. Conditions list out all the parameter requirements that need to be met for a transition to be taken. This box contains only a single parameter: “Fire”. You can view all possible parameters by clicking the word “Parameters” on the left side of the window.

The open circle next to the Fire parameter indicates that parameter Fire is a trigger type parameter. When it gets set to true (via script or some other means), it’ll stay true for one frame and then automatically set itself to false again. There are other types of parameters too, such as int, float, or bool.

What this means is that if our Gun is in the Idle state and we set the Fire parameter to true, it’ll move to the Gun-Anim state and start playing the motion stored there (the recoil animation).

If you now click on the arrow going from Gun-Anim to Idle, you’ll notice there’s no conditions listed. Does this mean we’re stuck in the state once we get there? Not quite. The checked box “Has Exit Time” above tells Unity that when the motion in the starting state is finished, it should automatically take this arrow. So no other condition is needed to return to Idle.

Let’s put all of this together now. Our gun starts off in the Idle state, and when we set parameter Fire to true, it moves to the Gun-Anim state and plays the recoil animation. Once the recoil animation is finished, it automatically moves back to the Idle state.

First off, add an Animator component to Gun’s child, Model, and set its controller to Gun (you could also just drag Gun.controller in the animations folder into Model’s inspector view). The animator must go on Model since that’s the 3D model that actually gets animated.

![image](/assets/images/ar%20foundation/lab2/10.png)

Now in order to start the recoil animation, we need to set the “Fire” parameter in the Animator component, which we can do in code. Switch to editing Gun.cs. We’ll start off by creating a private variable that holds our animator and initializing it in Start(). Since our animator component is on Model, not Gun, we’ll need to get a reference to Model. transform.Find() looks for a specifically named object in all of the base object’s children.

        ...

        private AudioSource audioSource;
        private Animator animator;

        // Use this for initialization
        void Start () {
            audioSource = GetComponent<AudioSource>();
            animator = transform.Find("Model").GetComponent<Animator>();
        }

        ...

Then in our Fire() function, we’ll set the parameter to trigger the animation. Unity’s Animator class has a different set function for each parameter type. In this case, we’re setting a trigger type parameter so we’ll use SetTrigger() and feed it the name of the parameter we want set.

    public void Fire() {
        audioSource.PlayOneShot(audioSource.clip);
        animator.SetTrigger("Fire");
    }

Firing the gun now should play a short recoil animation.

# Visual Effect

Just like with animation, Unity has its own complex system for creating and manipulating visual effects that won’t be in scope for this class. If you wish to learn more about it, I recommend starting with Unity’s [video](https://unity3d.com/learn/tutorials/topics/graphics/particle-system) on the topic.

In folder Prefabs/Particle Systems, we’ve provided a gunshot VFX for you called “MuzzleFlashEffect”. Drag it into the editor as a child of Gun. Position the MuzzleFlashEffect in front of the gun barrel (where the VFX will play). The size of the particle effect is also extremely small, so we scale the size to 50.

![image](/assets/images/ar%20foundation/lab2/MuzzleFlashEffect.png)

The MuzzleFlashEffect should be around here in front of the gun barrel:

![image](/assets/images/ar%20foundation/lab2/EffectLocation.png)

All that’s left is to play the VFX here in script. Switch to editing Gun.cs. Like the previous two sections, we’ll start by creating and initializing a variable that’ll reference the Particle System.

        ...

        private AudioSource audioSource;
        private Animator animator;
        private ParticleSystem particleSystem;

        // Use this for initialization
        void Start () {
            audioSource = GetComponent<AudioSource>();
            animator = transform.Find("Model").GetComponent<Animator>();
            particleSystem = transform.Find("MuzzleFlashEffect").GetComponent<ParticleSystem>();
        }

        ...

Then in our function Fire() we’ll make the VFX play.

    public void Fire() {
        audioSource.PlayOneShot(audioSource.clip);
        animator.SetTrigger("Fire");
        particleSystem.Play();
    }

Firing the gun should now create a spark of muzzle flash at the front of the gun.

*This marks the end of lab 2!*