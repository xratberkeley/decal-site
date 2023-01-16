---
layout: page
title: "AR Foundation Lab 4: Putting It All Together"
#nav_order: 1
parent: AR Foundation
grand_parent: Labs
---
# AR Foundation Lab 4: Putting It All Together
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc}

---

In this lab, we’ll be taking all of the components you learned about and created thus far and put them together into a single cohesive experience. We’ll also take care of some odds and ends that we didn’t get to in the previous labs.

Imagine the flow of the game, from start to finish. Here’s everything that needs to happen:

- Player can see through the camera on their device and have a working gun. Done.

- Monsters start spawning periodically from preset locations and walk towards the player. We’ve done the latter, but not the former.

- Players can shoot and kill monsters. Done.

- Upon reaching the player, monsters will attack, causing the player to take damage. We’ve done the former, but not the latter.

- Upon losing all health, the game restarts. Not done.

# Spawning Monsters

To start things off, let’s update our monster prefab so that when we spawn it via script it has all the functionality the monster in our scene has. Select the monster gameobject and click Overrides > Apply All in its prefab options. If you made all the edits to the prefab directly you can skip this step.

![image](/assets/images/ar%20foundation/lab4/lab4fix.png)

After you’ve done this, you can remove the monster within the scene - we don’t want to get assassinated the instant we start the game.

Next, create a script called SpawnManager. This script will manage the spawning process for the monsters. Declare in it the following public variables and stub in the Spawn() function.

    public class SpawnManager : MonoBehaviour {
        public Transform[] spawnLocations;
        public float spawnTime;
        public GameObject monsterPrefab;

        ...

        public void Spawn() {

        }
    }

The variables are pretty self explanatory - spawnLocations will contain a list of locations (transforms) where monsters could spawn from, spawnTime is the time interval between spawns, and monsterPrefab is the gameobject that gets spawned.

Let’s fill out Spawn(). Every time this function gets called, we’ll want to choose a random transform to spawn it at. We’ll instantiate the monster, then position and rotate it accordingly.

    public void Spawn() {
        Transform spawn = spawnLocations[Random.Range(0, spawnLocations.Length)];
        GameObject monster = Instantiate(monsterPrefab, spawn.position, spawn.rotation);
    }

Random.Range(x, y) returns a random integer between x inclusive and y exclusive. As a side note, if you feed it two floats instead, it’ll return you a random float instead.

Now we have to repeatedly call this function every couple seconds. While we could write a timer system in Update() using GetTime(), Unity actually has a function for this particular use case. Put the following line in the Start() function.

    void Start () {
        InvokeRepeating("Spawn", spawnTime, spawnTime);
    }

InvokeRepeating is a Unity function that will call the function in the first argument after a certain amount of time, dictated by the second argument. It will then continue to call this function at set intervals (the third argument). In our case, Unity will call Spawn() after spawnTime seconds and continue to do so endlessly.

Let’s add the SpawnManager script to our game. In the Prefabs folder, double click on the **Environment** prefab to open up this menu.

![image](/assets/images/ar%20foundation/lab4/environment_1.png)

In this, create a new empty gameobject called “ScriptManager”, and attach this script to it. Set Spawn Time to 5 and set the monster prefab correctly.

![image](/assets/images/ar%20foundation/lab4/2_2.png)

To fill in Spawn Locations, we’re going to create empty gameobjects (which contain only transforms) and position them around the map where we want monsters to spawn.

But where do we want to spawn them? The locations should be out of the player’s sight, so that we can’t see them pop into existence. They also have to be on the nav mesh, otherwise the spawned monster wouldn’t be able to navigate properly. To make sure we’re meeting these requirements, open up the navigation window, which should cause the blue navmesh to appear.

Create a couple empty gameobjects and call them Spawn (you can number them if you wish). Position them around the map such that they’re on the navmesh and out of the player’s line of sight.

![image](/assets/images/ar%20foundation/lab4/3_1.png)
![image](/assets/images/ar%20foundation/lab4/4_1.png)
![image](/assets/images/ar%20foundation/lab4/5_0.png)

Once you’re satisfied with your spawns, select ScriptManager and drag the spawns onto the Spawn Locations array to add to the list.

![image](/assets/images/ar%20foundation/lab4/6_1.png)

Give it a shot! Starting the game now should cause a monster to spawn once every 5 seconds.

# The Player

We’re now going to create a script to deal with player specific logic. Create and begin editing a new script called “Player”. Similar to the monster, we’ll want to track the player’s state (either alive or dead) with enums and also store the player’s health.

    public enum State {
        ALIVE, DEAD
    }

    public State playerState = State.ALIVE;

    public int maxHealth;

    private int health;

    void Start () {
        health = maxHealth;
    }

There’s only one interaction we haven’t implemented yet for the player: getting hurt by monsters. Stub in the public Hurt() function.

    public void Hurt(int damage) {

    }

This function is nearly identical to the Hurt() function in Monster.cs. Fill it out and stub in a Die() function as well.

    public void Hurt(int damage) {
        if (playerState == State.ALIVE) {
            health -= damage;
            if (health <= 0) {
                Die();
            }
        }
    }

    void Die() {

    }

What happens when the player dies? There’s a lot of things we could do, but for simplicity we’ll just restart the level. Unity gives you the ability to load up a new scene in its library UnityEngine.SceneManagement, so import it at the top of the file. Then add a line in Die() to reload the scene.

    using UnityEngine.SceneManagement;

In Unity, add the player script to your ScriptManager gameobject and set Max Health to 100.

All that’s left is to call Hurt() whenever the player gets attacked by a monster. Switch to editing Monster.cs, and in it declare a damage variable and a reference to the Player script you just wrote.

    public int damage;

Finally, modify the Attack() function (and potentially other parts of the code) so that it calls Hurt() appropriately to damage the player. **We have left implementing this final part as an exercise to the reader**. Again, here are some tips:

- For monsters to be able to hurt the player, there needs to be some link between the monster script on on a spawned monster and the player script in the scene that you just added.
- Remember that all components are scripts and vice-versa, from the ones default to Unity like Rigidbody or AudioSource, to the ones you created and added to a gameobject yourself.
  
Try it out! When monsters attack you up to 5 times now, the scene should restart (indicating that you’ve lost).

# The Canvas

By now you might be wondering about how to add UI to your game. Fortunately, Unity provides us the Canvas system that can handle UI interactions apart from the game. You can read more about it [here](https://docs.unity3d.com/Packages/com.unity.ugui@1.0/manual/UICanvas.html).

Create a canvas **(right click hierarchy > UI > Canvas)**.

First adjust the canvas scaler to **Scale with Screen Size**. 

![image](/assets/images/ar%20foundation/lab4/Canvas%20scaler.png)

Now add a button for firing the gun. **(right click hierarchy > UI > Button)**  Move the button to the bottom right and anchor it to the corner. Adjust the button size to your liking and change the text to Fire (adjust the font size too). Also make the background for the button transparent. 

![image](/assets/images/ar%20foundation/lab4/button%20position.png)

Now let's link the button to the fire function of the gun. Drag the gun object from under AR Camera to the onclick function of button. Choose the Fire function from the gun. Also, remove the update function from Gun.cs so only clicking the fire button with fire the gun.

![image](/assets/images/ar%20foundation/lab4/onclick.png)

# Main Menu

Most games have some sort of main menu so we'll also make a rudimentary one to get the concept. We will take the following steps to make this work.

- Create a new scene called MainMenu
- Make a start button to enter the game
- Make a back button to return to the main menu
- Return to the main menu when the player dies

Start by creating a new Scene in the scenes folder. Create a canvas and place a Start button in the center. Feel free to customize the size, font, color, etc to you liking. You can also set the background of the scene to a solid color if you do not want the skybox as the background for your menu.

To change the background, select the main camera and change clear flags from skybox to solid color, then change the color.

Next, make a new script called SceneSelect and create a StartButtonPressed() function. Fill in the function to load the "lab" scene when called. We won't provide the code for this since you can figure it out by now (hint: look at SceneManager documentation)

We created a new MenuManager gameobject undercanvas to attach this script to and link with the Start button. There are other ways to do it, so on your own link the new script you created to the start button.

You should now be greeted with your MainMenu and clicking Start should launch the game!

 

Okay two more things to fix.

1. We should make a back button in the game to return to the main menu

2. Return to the main menu once you die.

For the back button, create a script called BackButton.cs. 

    public class BackButton : MonoBehaviour
    {
        [SerializeField]
        GameObject m_BackButton;
        public GameObject backButton
        {
            get { return m_BackButton; }
            set { m_BackButton = value; }
        }

        void Start()
        {
            if (Application.CanStreamedLevelBeLoaded("MainMenu"))
            {
                m_BackButton.SetActive(true);
            }
        }

        public void BackButtonPressed()
        {
            // YOUR CODE HERE (launch the main menu)
        }
    }

Now in the lab scene create a BackButtonManager gameobject under the canvas hierarchy. Create a button as a child of the BackButtonManager. (ignore the RuntimeHierarchy and RuntimeInspector)

![image](/assets/images/ar%20foundation/lab4/Backbuttonmanager.png)

Attach your BackButton script to the BackButtonManager and link the two together. Link the button with the script and link the button to the function.

Lastly, make it so you return to the main menu when the player dies. (where did we code this in?)

 

*Congratulations, you have completed the basic lab! You have also learned to use the canvas system in Unity and make UI elements for you game.*