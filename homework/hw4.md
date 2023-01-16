---
layout: page
title: "Homework 4: Pong"
nav_order: 4
parent: Homework
---
# Homework 4: Pong
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc}

---

By now, you should have a pretty good grasp on Unity from the labs and the other homework assignments. Unlike these past assignments, however, your final project is completely free-form, with no set instructions to follow. As a way to bridge this gap, this final assignment will be a lot less guided than the previous ones.

You'll be building a clone of Pong, the classic Atari game. Download the skeleton package [**here**](https://drive.google.com/file/d/1c7m916mECxBUbsGPNjTeQAhDPE2XL1l5/view?usp=sharing).

![image](/assets/images/hw4/1_5.png)

# Your Task

Everyone has heard of [Pong](https://www.youtube.com/watch?v=fiShX2pTz9A), and for good reason - it was the first financially successful video game of all time and kick-started the industry as we know it today. It is also an extremely simple game (out of necessity, considering it had to fit on 128 *bytes* of ram). The combination of these two factors makes it a traditional programming exercise for anyone learning a new language or tool.

Your task is to implement a game of Pong such that it meets the specifications, which are described below. If you look at the skeleton package, you'll notice that while we've built the scene for you, there's almost no starter code. You recommend using the scene provided, but you are free to start from scratch.

Now, let's break down a game of Pong. What are the components that work together to make the game tick?

# Puck Physics

![image](/assets/images/hw4/2_3.png)

The puck moves across the table, bouncing off of the walls and paddles. With respect to movement, we expect the following from your puck:

- It moves in straight lines at constant velocity, and doesn't change speed after bouncing off an object.
- The angle it reflects to after bouncing off an object is the same angle as when it came in (imagine a laser bouncing off a mirror).

*This isn't faithful to the original games. In other versions of Pong, the puck would speed up after every collision until somebody lost. The angle that which the puck would reflect off the paddle would also change depending on which part of the paddle was hit. You are free to implement this more advanced version for extra credit.*

## Implementation Hints 

In the scene provided, we've taken advantage of Unity physics to both specifications by attaching a [physics material](https://docs.unity3d.com/Manual/class-PhysicMaterial.html) with full bounciness to the puck gameobject. We've also changed its rigidbody's collision detection to "Continuous Dynamic", which tells Unity that this object is going to move around a lot and should be treated accordingly.

# Player Interactions

![image](/assets/images/hw4/3_2.png)

The player controls the blue paddle on the left side of the screen. Pressing the **up** or **w** keys on the keyboard should move the paddle up, while pressing **down** or **s** should move it down. The paddle should move at a constant, reasonable speed, and shouldn't clip into or move through the walls.

## Implementation Hints 

Utilize Unity's [input manager](https://docs.unity3d.com/Manual/ConventionalGameInput.html) to access keyboard input (you used it in roll-a-ball to move the ball). Instead of using physics to move the paddle around, it'd be easier to directly modify its transform.

# Artificial Intelligence

![image](/assets/images/hw4/4_2.png)

An AI agent controls the red paddle on the right side of the screen. The movement of this paddle is subject to the same constraints as the player - it must move at the same speed as the player, cannot clip or pass through walls, and cannot accelerate.

You must script an AI for this paddle so that one person can play against the computer. The AI doesn't have to be perfect by any means - it's fine if it's able to hit the puck roughly ~50% of the time.

## Implementation Hints 

There are many possible solutions for this part. One simple approach is comparing the paddle's z position with that of the puck's, and choosing a direction to move in off of that. Since this is Berkeley though, for full credit we require that AI agents be run by multi-layer convolutional neural nets fed by the data of every pong, air hockey, and beer pong game ever played*.

**This is a joke. But we will give extra credit to anyone who manages to implement an ML powered pong AI into this assignment. To get started, you could look into Unity's recently released [ML Agents](https://github.com/Unity-Technologies/ml-agents). Want to go even further? Do it off of purely pixel values with [OpenAI Universe](https://github.com/openai/universe) with [deep reinforcement learning](http://www.nature.com/nature/journal/v518/n7540/abs/nature14236.html?foxtrotcallback=true).*

# Game Logic 

When the game begins, the puck should receive an impulse that gets it moving across the table. This can be either hard-coded to move towards a certain player or completely randomized. It must, however, have a decent amount of movement on the x-axis so that the game doesn't get stuck. At this time, the score will be 0 to 0. This is reflected in the ScoreUI object which we've scripted and put into the scene for you already.

Changes in game logic occur when the puck passes by a paddle and reaches the goal behind it. The following things occur:

- The scoring player receives a point (update ScoreUI accordingly).
- The puck is moved back to the center of the table.
- The puck receives an impulse that gets it moving towards the *player who was scored on*.

When the winning score of 5 is reached, a message must be displayed on screen indicating who won. ScoreUI does this for you already.

## Implementation Hints 

There are many ways you could detect a goal being scored: triggers, [collision detection](https://unity3d.com/learn/tutorials/topics/physics/detecting-collisions-oncollisionenter) with the border, or a simple distance check on the x-axis. Since the puck has a rigidbody, you can use [AddForce](https://docs.unity3d.com/ScriptReference/Rigidbody.AddForce.html) to apply an initial impulse to it.

# Submission and Grading

Use simmer.io to submit your work to bCourses.

Grading will be done by whether or not the submission meets the described specifications. For full credit, a facilitator must be able to actually play the game. Partial credit will be given if some components (puck, player, AI, logic) work but others don't. Unlike HW2 and HW3, making additional improvements is not required for this assignment. We will, however, still give extra credit to those we deem went above and beyond with their submission.