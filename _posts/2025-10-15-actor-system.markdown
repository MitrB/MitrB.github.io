---
layout: post
title:  "How not to write 455 dialogues for every cinematic : The Actor System"
date:   2025-10-15 14:00:43 +0200
categories: game design 
---

Imagine you are building a game where you have 15 characters you could possibly unlock. From these 15 characters, 3 characters could be taken on a "mission". A game with narrative moments where these characters have dialogue scenes with each other and NPCs. A game without a protagonist that does the heavy lifting by doing most of the talking. You have to account for any combination of 3 characters out of 15. This means 455 possible combinations for any given dialogue sequence! How do you solve this?

I came up with the Actor System. A compromise to make this possible.

The core idea is very simple: when encountering a dialogue scene, we assign a role to every character from the party. They perform the dialogue that was written for the role. Writing good and creative dialogue will do a lot of the heavy lifting. But we can be flexible and extend this core premise.

To make my implementation simple, I held to one important rule: there must be a fixed number of characters participating in any dialogue scene. In my case this was 3. This means that at any point in the game the player had a team of 3 characters. This way I could write dialogue for any party of 3, while not having to write 455 different possibilities.

I will also show how I extended my simple premise, giving myself more dialogue to write, but there is an important tradeoff between having to write less and being able to express a character in a way that still feels believable.

In this writeup I'll try to lay down a generic framework anyone could use for such a system.

## What is an Actor?

An actor is an archetype. It's a role that a character will take during a dialogue. This means that we have to be able to define such a role. I decided to define an Actor by an Actor Vector. You could call it a Personality Vector, but you can really put any qualities in it. That's the cool thing about this system, it's a generic framework that you can mold to your needs.

For this example, let's take 5 personality quality ranges that speak to our imaginations.

1. Passive      <->     Active
2. Nihilist     <->     Existentialist 
3. Stoic        <->     Emotional
4. Optimist     <->     Pessimist 
5. Introvert    <->     Extrovert

For each quality I define 0 to be the middle point between the two extremes, the upper and lower limit depends on needs for the game system but it will become clear why it wouldn't be optimal to have a big range. This means that if I want to represent a **Stoic** Actor that is somewhat **Optimistic**, despite their **Nihilist** views, but prefer to be **Active** during interactions, I would represent them with such a vector: [0.8, -1.0, -0.9, -0.35, 0].

### Character Actor Vector

Each of the characters in your game is assigned their own personal Actor Vector, which we'll call the Character Actor Vector. This distills their personality into an N-dimensional vector, with N being the number of "personality" traits you account for. These could also be physical traits. For example, you could define the physical strength of your characters.

### Dialogue Actor Vector

Each dialogue sequence will require 3 Actors, which might differ greatly and be less nuanced than the personal Actor vectors assigned to characters themselves. For example, if we have a dialogue sequence where we need an **Optimist**, someone **Active** and an **Introvert**, we would call these Dialogue Actor vectors:

1. Optimist: [0, 0, 0, -1, 0]
2. Active: [1, 0, 0, 0, 0]
3. Introvert [0, 0, 0, 0, -1]

The Character Actor Vectors will have to be mapped to these Dialogue Actor Vectors. Let's explain with what mindset we will be writing dialogue sequences and after that how we will be mapping Characters to Actors.

## Writing a dialogue sequence

When writing a dialogue sequence we write dialogues for the Dialogue Actors, not specific characters. Imagine that our characters find themselves in a precarious situation and are now stuck in a dark room which they need to escape from. We start this sequence where we want a character that will take the lead: an **Active**, **Optimist** so we assign them the vector [1, 0, 0, -1, 0]. Someone else that is **Pessimistic** about the situation: [0, 0, 0, 1, 0]. And an **Extroverted** diplomat: [0.5, 0.5, 0.2, -0.5, 0.5]. Notice that I define these actors as I see fit and I should probably define the Dialogue Actor Vectors after I have written the dialogue for this dialogue sequence.

We can write the scene with these 3 Actors in mind. I imagine the leader figure wanting to take charge, the pessimist being stubborn about their hopelessness and the diplomat trying to mediate between them.

Some of you might have noticed one big issue with this system: it has the downside that we lose all unique personalities for individual characters. Some of you may have already thought of a solution: let's also introduce conditional dialogue if one of the Characters is assigned to a certain Dialogue Actor. This gives us the freedom to add personal quirks from a character to a certain line. The idea is to convey the same phrase, but in a way that is more in line with what that specific character would say.

We can go even further and have different scenarios for a given dialogue sequence. Imagine that for the same dialogue sequence, like the example given before, we might have a situation where we foresee another group of Actors. Maybe we have a much more subdued or active group, or maybe we have a completely different scenario if a specific character is in the group.

Let's get into the technical stuff by looking at some methods of achieving such an Actor system. Starting with mapping the Character Actors to Dialogue Actors.

## Mapping Characters to Actors

When triggering a dialogue sequence we have to properly map each Character Actor bijectively to another Dialogue Actor. Yes of course we are going to compare Euclidean distances and assign vectors by smallest distance between them!

But before doing this calculation we will consult the Actor System to only give us possible combinations of mappings. For example, if we have actors [A, B, C] we could define that if a given character is present for a dialogue sequence, they always should play actor A. It should also be possible for us to forbid them from playing actors B and C for example. Let me show with this simple example.

Imagine we have 3 characters (from the 15 total) that trigger a dialogue scene:

1. Afikka   | Character Actor Vector : [0.5, 0.8, 0.3, 0.35, -0.45]
2. Solevius | Character Actor Vector : [-0.6, 0, -0.2, -0.67, 0.9]
3. Caligula | Character Actor Vector : [1.0, -0.78, -0.68, 0.8, -0.8]

and we have a need for these 3 Actors defined by their Dialogue Actor Vectors:

1. Leader   | Dialogue Actor Vector : [1, 0, 0, -1, 0]
2. Pessimist| Dialogue Actor Vector : [0, 0, 0, 1, 0]
3. Mediator | Dialogue Actor Vector : [0.5, 0.5, 0.2, -0.5, 0.5]

We have 6 possible combinations of bijections. I'll be shortening their names to A S C for Afikka Solevius and Caligula respectively and the actors to L P M for Leader Pessimist and Mediator. We can calculate their distances.

1. A=L, S=P, C=M, distance = 6.1914
2. A=L, S=M, C=P, distance = 4.7370
3. A=P, S=L, C=M, distance = 5.6010
4. A=P, S=M, C=L, distance = 4.8369
5. A=M, S=L, C=P, distance = 4.8478
6. A=M, S=P, C=L, distance = 5.5382

It's clear that combination 2 is the best fitting one, meaning that Afikka will be playing the Leader, Solevius the Mediator and Caligula the Pessimist. Now imagine that we bar Afikka from playing the leader, then we don't calculate those possibilities:

1. A=P, S=L, C=M, distance = 5.6010
2. A=P, S=M, C=L, distance = 4.8369
3. A=M, S=L, C=P, distance = 4.8478
4. A=M, S=P, C=L, distance = 5.5382

Now the best fit is Afikka playing the Pessimist, Solevius the Mediator and Caligula the Leader.

So, in the most basic version of the Actor system you create a dialogue sequence for N characters. You trigger this sequence, which maps each character's Character Actor Vector to its closest Dialogue Actor Vector and the scene plays out with the characters speaking the lines of their respective actor. You might even have some conditions for when a certain character gets matched to a certain actor in a scene, letting them say a more fitting variation of their dialogue.

This is a very limiting way of using this system, but it will work if you keep to these conditions. To give ourselves more freedom (and work) we must admit that one group of characters might have a very different dynamic from another group of characters. That's why we won't just be defining N Dialogue Actor Vectors, but also defining multiple scenarios by defining multiple groups of Actor Vectors. Instead of doing a bijection on one group, we do the bijection on each group and then take the best fitting one.

Here's an example for 2 different scenarios for the same scene, which funnel in a common part. ![Example](/assets/ActorSystemExample.png)

Given any combination of characters in this dialogue sequence, they will be mapped to either scenario 1 or 2. These scenarios have different party dynamics, giving a different feel to the conversation. In this dialogue sequence, we also decided to let these 2 scenarios converge to a generic dialogue sequence. I hope this shows what the system is trying to achieve.

## Closing thoughts

This system tries to make sure that characters act in a way that feels natural during a dialogue sequence. We're essentially creating an illusion of depth. The system works best if the player is wondering (at least on first impression) if we really did write dialogue for every possible party combination.

If at any point in reading this you got excited to play this game I am talking about... I'd have to disappoint you as it is neither finished nor in production. I am sure though that I will return to this concept. This was the first "big" game I was working on and at some point I decided to quit it, pursuing other challenges. I basically chickened out, but I'm slowly gearing up to be capable of finishing such a project. That I am sure of.

But I wanted to share this system! Maybe it inspires you or maybe you can implement it in your own game...
