---
layout: post
title:  "I made: Goethe's Knot"
date:   2025-10-15 14:00:43 +0200
categories: game design 
---

I participated in the [2024 GMTK gamejam](https://itch.io/jam/gmtk-2024) and created [Goethe's knot](https://itch.io/jam/gmtk-2024/rate/2907683). I had some help from a couple of friends for the audio, textures and the title screen backdrop.

The jam's theme was: **built to scale**. The thing that came to mind for me was the environment in [Blame!](https://en.wikipedia.org/wiki/Blame!) created by [Tsutomu Nihei](https://en.wikipedia.org/wiki/Tsutomu_Nihei). It had been on my mind for a while and I took this as an opportunity to create something that at least tried to emulate this world that was **built to scale**.

I used the movement, camera scripts and post processing shader from a previous project. Most of the work I put into the project was creating and scripting the environment and its puzzles.

## Environment

The environment was created using [TrenchBroom](https://github.com/TrenchBroom/TrenchBroom) and the [Qodot](https://github.com/QodotPlugin/Qodot/tree/main) plugin for [Godot](https://godotengine.org/).

The Qodot plugin for Trenchbroom allowed me to iterate quickly and create my environment entirely in Trenchbroom. It supports tagging and adding data to meshes in Trenchbroom, which you can script in Godot. This way, I could set up camera change trigger zones and movable objects fully in Trenchbroom. This is how the map looked with all layers enabled:

![Trenchbroom Map](/assets/GoetheBroom.png)

## Flow

I went for a camera that wasn't controlled by the player but rather controlled by camera areas which set its position and fov. This guides the player in the direction of progress.

There is a lot of walking and a lot of "useless" places to find, with the only reward being some scenery. But I think that captures the lonely feeling I got when reading the **Blame!** manga. ![Scenery](/assets/scenery.png)

## Puzzles

The puzzles are logic puzzles with light parkour elements. Each of the 3 characters can hold one color-coded key. There are 7 keys in total: red, green, blue, yellow, cyan, magenta and white. The 3 characters themselves are color-coded: red, green and blue.

Each character can only see and interact with the objects that contain their own color. This way the red character can only see red, yellow, magenta and white objects. This means the characters can't see each other.

The characters can find 7 color-coded keys throughout the game. Each key controls the state of all objects with the corresponding color. At first I thought about having the keys change the state of all objects containing colors that make up the key's color, but it became a mess. So I decided to keep it simple.

I felt like a lot of people did not fully understand this concept, which is largely due to not having a proper tutorial.

## Feedback

An amazing part of the jam is trying out other people's games and getting feedback on your own! And I love all the feedback we got. You can read all the comments here: [Goethe's knot](https://itch.io/jam/gmtk-2024/rate/2907683).

I actually didn't expect anyone to finish the game, because it can be a chore to figure out how to progress at some points, especially if you haven't understood the mechanics. But this person did, and they left a comment that I'm not going to forget. This fueled my desire to create things for people to experience, this is what makes all the doubts and struggling worth it.

![Feedback00](/assets/GoetheFeedback00.png)

There are many more nice comments. And yes, by nature of the jam, everybody is encouraged to be nice to each other so that they'll also review your game. But I felt this one was heartfelt and really touching.

We got reviewed 30 times and this was the outcome:
![Example](/assets/rating.png)

Not too bad, definitely if you know that there were 7547 entries and the median ratings for a game were 12 if I remember correctly!
