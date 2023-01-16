---
layout: page
title: "Vive Lab 2: Creating the Gun"
#nav_order: 1
parent: HTC Vive
grand_parent: Labs
---
# Vive Lab 2: Creating the Gun
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc}

---

In this lab, we’ll focus completely on the gun. By the end of the lab, you should be able to pick up the gun, drop it, and fire it with the appropriate effects attached.

How can we interact with our gun, anyway? If you hold an HTC Vive controller in your hand, you’ll notice that there are two main triggers that we can press. One is with the middle finger, called the Grip Trigger, and the other is with the index finger, called the Hair Trigger.

![image](/assets/images/htc%20vive/lab2/Vive%20Input.png)
[*Image Source*](https://www.raywenderlich.com/149239/htc-vive-tutorial-unity)

For our gun, we want to implement the following functionality:

- Hovering over the gun and holding down the grip trigger picks up the gun.

- While picked up, pressing the hair trigger fires the gun.

- When the grip trigger is released, the gun is dropped and becomes a physics object.

You can download the skeleton for this lab [**here**](https://drive.google.com/open?id=1KKam_FTVJbKOZo66NfSXPcGsk7DLwx8_) - alternatively, you can directly start working off a completed Lab 1! **This is a long lab, so please don’t hesitate to ask the facilitators questions or just reach out if you get stuck. We’re here to help!**

# Reading In Controller Input

The triggers on the Touch controllers are like buttons, which only have an on and off state. Because of this, input for the triggers comes as a boolean that is either true or false, where the meaning of the values depend on which function from the Steam SDK is being used. In our case, true will mean we are holding the trigger down, and false will mean the opposite. We’re going to read in this data from Steam’s SDK and store it, once per frame.

Let’s start this off by creating our first script. Go into the Scripts folder, right-click an empty space within, select Create > C# Script, and name it “Hand”. Double-click the file to open it (likely in Visual Studio).

![image](/assets/images/htc%20vive/lab2/1.PNG)

*Unity automatically starts you out with some boilerplate code.*

You can delete the Start function, as we won’t be using it. We’ll put in a private variable that represents the controller this hand is attached to. It will be initialized using the Awake() function. We’ll also use a variable to store the hair trigger value of the previous frame, for reasons we’ll explain later.

    public class Hand : MonoBehaviour {

        // Reference to trackObj script. Keeps track of controllers

        private SteamVR_TrackedObject trackedObj;

        // Keeps track of controllers. Also which controller (L/R)

        private SteamVR_Controller.Device Controller

        {

            get { return SteamVR_Controller.Input((int)trackedObj.index); }

        }


        private bool indexTriggerState = false;

        private bool handTriggerState = false;

        private bool oldIndexTriggerState = false;

    void Awake() {
        trackedObj = GetComponent<SteamVR_TrackedObject> ();
    }

    ...

    }

The ellipses represent omitted code. The public and private keywords have the same meanings they do in other programmings languages, like Java. However, public variables can be seen and set in the Unity editor, while private ones cannot.

Next, using the Update() function, every frame we’ll read controller input and store it in the variables we just defined for later use.

    void Update() {

    oldIndexTriggerState = indexTriggerState;

    indexTriggerState = Controller.GetHairTriggerDown ();

    handTriggerState = Controller.GetPressDown (SteamVR_Controller.ButtonMask.Grip);

    }

The magic happens in the second and third lines, where we use Steam’s API to query for controller state using the GetHairTriggerDown() and GetPressDown() functions. The parameters should be fairly intuitive, but you can get extra practice with SteamVR input with this tutorial [here](https://www.raywenderlich.com/149239/htc-vive-tutorial-unity).

Next, add the following lines to the end of the Update() function so we can later test our code.

    print("Hand-Trigger State: " + indexTriggerState);

    print("Grip State: " + handTriggerState);

Switch back to Unity now. We’re going to add an instance of our new Hand script to each of our hands. To do so, look for CameraRig’s children, Controller (left) and Controller (right) These represent your controllers in the game. Add the script as a component to both of them (you can use the same Add Component button to search for “Hand”). Your Inspector view for these objects should now look like this:

![image](/assets/images/htc%20vive/lab2/Controller%20Left%20Lab%202.png)
![image](/assets/images/htc%20vive/lab2/Controller%20Right%20Lab%202.png)

Finally, run your project but don’t put on the headset, and open the Console tab (next to Project). You should be able to play with the controller triggers and see the values change on the console.

# Picking Up the Gun (Part 1)

Now that we’re successfully reading in input, let’s implement our first bit of functionality: picking up the gun. Before we do anything, we’ll add two variables to our Hand class: boolean holdingGun, which stores whether or not we’re holding a gun, and GameObject gun, which stores the gun if we’re holding one.

    private bool holdingGun = false;

    private GameObject gun = null;

There are three requirements that must be met for us to try and pick up the gun:

1. Our hand must be touching/inside the gun.

2. We must hold down on the grip trigger for that grip’s controller

3. We must not already be holding a gun.

Let’s start with the first requirement. How can we detect whether or not our hand touching the gun? The answer is with [triggers](https://unity3d.com/learn/tutorials/topics/physics/colliders-triggers).

You might remember from the roll-a-ball tutorial that triggers are a type of collider. Objects with trigger colliders do not collide with other things normally. They freely pass through them and this can be detected via code. For our purposes, we’ll attach a trigger collider to our hands, then detect collisions in script and check if the other collider is a gun.

In Unity, shift-select both Controller (left) and Controller (right) so that we can edit them simultaneously. Add a Cube Collider to the front of them - they’ll represent our hands in physics calculations. Change their side lengths to 0.075 so they more accurately bound to the tip of the controllers.

Switch back to Hand.cs in Visual Studio. Add the following function to the class:

    void OnTriggerStay(Collider other) {

    }

This [function](https://docs.unity3d.com/ScriptReference/Collider.OnTriggerStay.html) automatically gets called every frame our hand touches a valid object. To delve into what counts as a “valid” object, see the chart below:

![image](/assets/images/htc%20vive/lab2/Lab%202%20Trigger%20Chart.png)

Our hand has no rigidbody (making it static) and has a trigger collider, so it counts as a Static Trigger Collider. By the chart, it’ll send trigger messages upon colliding with our gun, which is a Rigidbody Collider.

So now we have a function that gets called when we touch the gun. However, this function could get called when it collides with other things as well, so we need a way to identify whether parameter other is actually the collider attached to our gun. We’ll use [tags](https://docs.unity3d.com/Manual/Tags.html) to do so.

    void OnTriggerStay(Collider other) {

        if (other.CompareTag("Gun")) {

        }

    }

Now that we know we’re dealing with a gun, let’s check if requirements two and three are met. If they are, we can grab the gun! You’ll also want to comment out or delete the print statements in Update() so that your print message can be seen.

    void OnTriggerStay(Collider other) {

        if (other.CompareTag("Gun")) {

            if (handTriggerState && !holdingGun) {

                print("Grabbing gun.");

            }

        }

    }

Our code is now ready to test, but there’s one thing we need to do in Unity first: set the tag on the gun. Select your Gun object in the hierarchy, and in Inspector view go to *Tag > Add Tag*. Press the plus button and create a new tag called “Gun”. Then select your Gun again and tag it as “Gun”.

![image](/assets/images/htc%20vive/lab2/Image%20Lab%202.png)

Try it out! Stick your hand into the gun and hold down the hand trigger, and check that you’re repeatedly printing out “Grabbing gun.” when you do so.

# Picking Up the Gun (Part 2)

The next thing to do is replace that print statement with actual functionality. Let’s remove that line and instead stub in the Grab() function.

    void OnTriggerStay(Collider other) {

        if (other.CompareTag("Gun")) {

            if (handTriggerState && !holdingGun) {

                Grab(other.gameObject);

            }

        }

    }

    void Grab(GameObject obj) {

    }

other.gameObject simply returns the GameObject attached to the collider - in this case, our gun. Let’s now consider everything that needs to happen when you grab the gun.

1. Variables holdingGun and gun need to be updated appropriately.

2. The gun should be parented under our hand so that the two will move together.

3. The position and rotation of the gun should snap our hand’s grip.

4. The gun should no longer get influenced by gravity or other outside forces, so that it doesn’t get knocked out of our hand somehow.

Item 1 and 2 are easy enough. transform by itself refers to the transform of whatever gameobject this script is attached to, which in this case is our hand. The reason we’re referring to the transform of each gameobject (as opposed to the gameobject itself) is because parenting only operates between transforms.

    void Grab(GameObject obj) {

        holdingGun = true;

        gun = obj;

        gun.transform.parent = transform;

    }

To do item 3, we’re going to introduce two new public Vector3 variables, holdPosition and holdRotation. Vector3s are just lists of length 3, and can hold x/y/z or yaw/pitch/roll values. These will determine the position and rotation of the gun, relative to our hand. We’ve experimentally determined these values for you already.

    private Vector3 holdPosition = new Vector3(0, -0.05f, -0.03f);

    private Vector3 holdRotation = new Vector3(-45, 180, 0);

Now we’ll use those values to position/rotate the gun properly when we grab it.

    void Grab(GameObject obj) {

        ...

        gun.transform.localPosition = holdPosition;

        gun.transform.localEulerAngles = holdRotation;

    }

localPosition and localEulerAngles represent the position and yaw/pitch/roll respectively of the object (the gun), relative to that of its parent (the hand).

And finally, for item 4 we simply have to modify the gun’s rigidbody (which as you recall defines how our object gets influenced by physics forces). We’ll disable gravity and enable IsKinematic, which will prevent our gun from getting influenced by collisions or normal physics forces.

    void Grab(GameObject obj) {

        ...

        gun.GetComponent<Rigidbody>().useGravity = false;

        gun.GetComponent<Rigidbody>().isKinematic = true;

    }

Note the syntax for getting a specific component attached to an object (here the rigidbody attached to the gun). We’ll be using this a lot in the future.

Go ahead and try it now! You should be able to reach over and, holding down the hand trigger on the controller, get the gun to snap to your hand.

As you may have noticed, when picking up the gun, the HTC Vive controller doesn’t disappear, and as a consequence, clashes with the gun model. We don’t want that. In order to make the controller graphically disappear when we pick up the gun, add the following code to the Hand script and appropriately reference it.

    //Should ideally make the HTC Vive controller visible or not depending on input

    // @source     //https://answers.unity.com/questions/1177557/hiding-steamvr-vive-controller-models.html

    void SetControllerVisible(GameObject controller, bool visible)  {

        foreach (SteamVR_RenderModel model in controller.GetComponentsInChildren<SteamVR_RenderModel>()) {

            foreach (var child in model.GetComponentsInChildren<MeshRenderer>())

                child.enabled = visible;

        }

    }

    void Grab(GameObject obj) {

        ...

        SetControllerVisible (this.gameObject, false);

    }

# Dropping the Gun

Now that we can pick up the gun, let’s implement the ability to release it. Unlike before, the requirements for us to drop the gun are much looser:

1. We must be holding the gun.

2. We must release the grip trigger..

Since these requirements could happen at any time, we’ll put a check for it in our Update() function so it gets checked every frame and stub in Release(). We’re also going to divide the checks into separate if cases as it’ll come in handy later.

    void Update() {

    ...
    
    if (holdingGun) {

        if (handTriggerState) {
        
            Release ();

        } else {
        
        } 

    }

    ...
    }

    void Release() {

    }

Now, what happens when you release the gun? Basically, undoing everything we did before.

1. The gun should be unparented from our hand so the two no longer move together.

2. The gun should get influenced by gravity and outside forces.

3. The gun should be given the same velocity that our hand had when it got released (so we can throw it).

4. Variables holdingGun and gun need to be updated appropriately.

5. After the gun is completely disconnected, we want to be able to see our controllers again.

Item 1 is simple enough.

    void Release() {
        handTriggerState = false;
        gun.transform.parent = null;

    }

Items 2 and 3 will require the rigidbody, so we’ll turn that into its own local variable for easy access. While we’re at it, we’ll complete item 2 by reverting the flags on the rigidbody we had set before.

    void Release() {
        handTriggerState = false;

        gun.transform.parent = null;

        Rigidbody rigidbody = gun.GetComponent<Rigidbody>();

        rigidbody.useGravity = true;

        rigidbody.isKinematic = false;

    }

To do item 3, we need to know the velocity and rotation our hand is moving the moment we release the gun. We could do this with a bit of math, but there’s an easier method: SteamVR’s SDK SteamVR_Controller.Device tracks the velocities and rotations of the physical controllers for you. We’ll take that value and assign it to our gun’s rigidbody.

    void Release() {

        ...

        rigidbody.velocity = Controller.velocity;

        rigidbody.angularVelocity = Controller.angularVelocity;

    }

We’ll update our variables to complete item 4. Note that setting gun to null had to be the last step; otherwise we wouldn’t have had a reference to our gun for the other items.

    void Release() {

        ...

        rigidbody.velocity = Controller.velocity;

        holdingGun = false;

        gun = null;

    }

Finally, to complete item 5, we simply add the appropriate reference to SetControllerVisible.

    void Release() {

        ...

        SetControllerVisible (this.gameObject, true);

    }

Give it a shot! You should now be able to drop or throw a held gun by simply releasing the hand trigger. Be careful not to hit anyone or anything while doing this.

# Firing the Gun

We’ve finally reached the most exciting part of the lab: firing the gun. As always, let’s first consider the requirements that need to be met for the gun to fire:

1. We must be holding the gun.

2. We must press down on the hair trigger
   
   - But it shouldn’t constantly fire as we hold it down.

We’re already checking for item 1 in Update() of Hand.cs, so we’ll move on to item 2.

Since don’t want the gun to fire constantly while we hold the button down, we have to check that our hair trigger state in the previous frame was false. This will force us to release the trigger in order to fire the gun again.

This is why we put in oldIndexTriggerState at the very beginning of the lab. Since oldIndexTriggerState represents the index trigger value of the previous frame, we can just check if it’s false and if indexTriggerState is greater than true. In Hand.cs, do the proper checks.

    ...

    private GameObject gun = null;

    void Update() {

        ...

        if (holdingGun) {
                    
            if (handTriggerState) {
                Release ();
            } else {
                Gun gunScript = gun.GetComponent<Gun> ();

                gunScript.SetTriggerRotation (indexTriggerState);

                if (indexTriggerState && !oldIndexTriggerState) {

                    gunScript.Fire ();
                }

            }
                        
        }

    }

And in Gun.cs, stub in the new public function Fire():

    public class Gun : MonoBehaviour {

        ...

        public void Fire() {

            print("Pew!");

        }

    }

If you try the project now, you should be able to print “Pew!” by pressing the index trigger.

The only thing left is to replace this print statement with proper behaviour. Let’s list out all the things that need to happen when we fire the gun:

- Gunshot sound plays.

- Recoil animation plays.

- Gunshot explosion visual effect (VFX) plays.

All three of these things will require use of different Unity components.

# Sound

In Unity, add an AudioSource component to gameobject Gun. AudioSources are what play sounds in Unity. Turn off “Play On Awake” since we don’t want to hear a gunshot every time we start the game. Then assign the clip “Gunshot” to the Audio Clip field (you can either drag it in from the Sounds folder or click the circle on the right).

![image](/assets/images/htc%20vive/lab2/8.png)

Next in Gun.cs, create a private variable of type AudioSource called audioSource. Then in Start(), initialize that value to reference the component we just created.

    public class Gun : MonoBehaviour {

        ...

        AudioSource audioSource;

        // Use this for initialization

        void Start () {

            audioSource = GetComponent<AudioSource>();

        }

        ...
    }

Now that we have a reference to the component, we can use it in our Fire() function. To play the clip attached to an AudioSource, call audioSource.PlayOneShot(). Unity also has a function called audioSource.Play(), but it can’t play multiple sounds simultaneously. This would be problematic as we can fire the gun faster than the length of the gunshot clip.

    public void Fire() {

        audioSource.PlayOneShot(audioSource.clip);

    }

Firing the gun now should give you a satisfying sound effect.

# Animation

We won’t be getting into the details of animation in this class. If you wish to learn more about it, I recommend first going through some of Unity’s videos on the topic here. Instead I’ll give a quick overview of the system. If you wish to skip this part and go straight to the next instruction, just look for the next bolded sentence.

On a high level, Unity’s animation system consists of two parts: controllers and motions. Motions (also called animations) are pieces of data that show how a particular object changes over time; they’re made in either Unity or some other software like Maya or 3DS Max. Controllers, on the other hand, are animation state machines; they contain the logic that dictates what animation plays when, based off of a set of external flags called parameters.

Parameters are defined in the controller and can be accessed/modified via script. This is mostly how we’ll be interacting with the animation system during the labs.

Take a look at the Animations folder and double-click the “Gun” controller to open up the Animator window.

![image](/assets/images/htc%20vive/lab2/9.png)

Note there are only two “real” states that are connected, Idle and Gun-Anim, and if you click on the Idle state you’ll see it doesn’t have a motion attached in the inspector. The Gun-Anim state does, however: the recoil animation. The green “Entry” state points to what state we start off on.

Next, look at the arrow going from Idle to Gun-Anim. This is a transition, and tells us that our Gun could possibly move from the Idle to Gun-Anim state.

In the inspector, you’ll see a box labeled “Conditions”. Conditions list out all the parameter requirements that need to be met for a transition to be taken. This box contains only a single parameter: “Fire”. You can view all possible parameters by clicking the word “Parameters” on the left side of the window.

The open circle next to the Fire parameter indicates that parameter Fire is a trigger type parameter. When it gets set to true (via script or some other means), it’ll stay true for one frame and then automatically set itself to false again. There are other types of parameters too, such as int, float, or bool.

What this means is that if our Gun is in the Idle state and we set the Fire parameter to true, it’ll move to the Gun-Anim state and start playing the motion stored there (the recoil animation).

If you now click on the arrow going from Gun-Anim to Idle, you’ll notice there’s no conditions listed. Does this mean we’re stuck in the state once we get there? Not quite. The checked box “Has Exit Time” above tells Unity that when the motion in the starting state is finished, it should automatically take this arrow. So no other condition is needed to return to Idle.

Let’s put all of this together now. Our gun starts off in the Idle state, and when we set parameter Fire to true, it moves to the Gun-Anim state and plays the recoil animation. Once the recoil animation is finished, it automatically moves back to the Idle state.

Instruction starts again here. First off, add an Animator component to Gun’s child, Model, and set its controller to Gun (you could also just drag Gun.controller in the animations folder into Model’s inspector view). The animator must go on Model since that’s the 3D model that actually gets animated.

![image](/assets/images/htc%20vive/lab2/10.png)

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

In folder Prefabs/Particle Systems, we’ve provided a gunshot VFX for you called “MuzzleFlashEffect”. Drag it into the editor as a child of Gun. It should already be positioned correctly in front of the gun barrel (where the VFX will play). If not, change its transform to what’s shown below.

![image](/assets/images/htc%20vive/lab2/11.png)

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

*This marks the end of lab 2! To check-off successfully, let a facilitator see your current project and show them that you can pick up, drop/throw, and fire the gun properly.*