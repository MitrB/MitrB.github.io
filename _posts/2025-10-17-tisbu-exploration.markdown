---
layout: post
title:  "How to traverse a megastructure: Don't blame yourself too much."
date:   2025-10-17 14:00:43 +0200
categories: game design 
---

For a while I had been dreaming about creating a [Blame!](https://en.wikipedia.org/wiki/Blame!) inspired game. **Blame!'s** environment and others like it have been described as indifferent to the people (or other creatures) that inhabit it. The environments are often megastructures, build for a bigger purpose than housing life. But when developing games, we often build our environments to suit the traversal needs of our protagonists. There can not be any dead ends that softlock you after 10 minutes of playtime. How do you manage to build a game that feels hostile, while keeping an engaging player experience?

## DBYTM

I'm creating a game under the working name "Don't blame yourself too much". It is a 3D exploration-platformer that takes place at the end of the universe, right before the beginning of the heath death, right before the last star has burned out. The game takes place in a world very much inspired by Tsutomu Nihei's work, a megastructure that is being expanded by automata. The story follow Cece and Mur, 2 siblings that are searching for an observatory.

Today we will be looking in to how you will be controlling Cece and Mur as you explore this world. I want to create a seemless experience without fail states, while giving the player as much freedom to explore as I could. Let's build up to the core traversal design idea.

Cece is be the primary character you will be controll. She controls much like other protagonists from platfromers. They can walk around, run and jump. Most importantly is that she is only impeded by the collision of the world. She is able to walk of cliffs and jump up to higher platforms. Let's look at a scene that represents the game world and possible traversal problems you could run in to.

![DBYTM00](/assets/DBYTM00.png)

Cece is represented by the red bean. You can imagine walking down that slope and wanting to explore the surrounding area.

![DBYTM01](/assets/DBYTM01.png)

You could jump off to a lower platfrom (1). Or try to make the jump (2). Maybe you just want to see what's down below (3). These 3 actions are troublesome. If you jump off at (1), then you can't get back up unless there are some platforms to get you back up. If this is an inhospitable area of the game, then this might not make sense. Those platforms might look a bit too convenient to the player, resulting in the breaking of their suspencion of belief. What if you fail jump (2)? Do we just magically teleport the player back? Do we kill Cece?! These things are expected from games, we can call them fail states. And most would not mind. But what if we could let the player just fall? What will they find there? Not sure, probably an empty surface. I'll lay out the tradeoffs later.

For now let's focus on the solution. We introduce Mur. While Cece is controlled with yout tyipcal WASD + SPACEBAR combo. Mur is controlled RTS style. With the click of a mouse button. If Cece's movement is prohibited and enabled by the environments collisions, Mur's movement is decided fully by a navmesh. He is stuck on it.

This means that there is no way for him to get stuck somewhere, as long the navmesh is continuous of course. There are some details that I will skip for now, especially with some of the puzzles in mind. But he is our anchor. He is our savepoint. Let's take a look at his playing field.

![DBYTM02](/assets/DBYTM02.png)

Mur is represented by the green bean and his navmesh is the area surrounded by green. We can clearly see that he can freely walk around the area after exiting the hallway. Mur and Cece are a clever pair. They both have half a part of a tool. A teleporation device that let's one wearer teleport to the other. I assume you understand the implications. Cece can at any moment, teleport to Mur's location (represented by the red arrows). 

This unlocks a level of freedom to the movement of Cece. As our progress is tracked by Mur's position in the world, we can explore every corner of every environment we find ourselves in. This has it's drawbacks. Creating puzzles with this in mind is tricky, at some point we might want to take away Cece's powers. But I won't focuss too much on the problems in this post.

There is a last problem that we have to solve: if the navmesh has to be continuous, then realisticly Cece could also just walk anywhere, defeating the purpose of any platforming challenge. Unless a switch for a door was unreachable, which is a decent solution. But I also introduce conditional teleportation fro Mur.

![DBYTM03](/assets/DBYTM03.png)

There are objects in the game (represented by purple blob) where Cece can summon Mur. This let's Mur move to places he can't reach.
 