---
title: "Video Summary - *Human-AI Co-creation for Games and Game Creators* by Mikhail Jacob (2021)"
tags: video-summary research
mathjax: true
excerpt: "Summarizing and providing my notes on the research talk - Human-AI Co-creation for Games
and Game Creators by Mikhail Jacob (2021)"
---

Jacob, M. (2021, February 23-24). *Human-AI Co-creation for Games and Game Creators* [Research
talk]. AI and Gaming Research Summit Day 1 Track 2 Part 2, Virtual. <https://www.microsoft.com/en-us/research/event/aiandgaming2021/>
[<i class="fab fa-youtube"></i>](https://youtu.be/brnuD-eO8-Y)
{: .notice--info}

The theme of this talk was regarding how AI can be used to co-create with humans. The first half of
the talk demonstrated this with two co-creative experiences developed by the speaker to improvise
dance and theatre. The second half explored how AI co-creation could be applied to game design, with
the speaker addressing what challenges there are in the application of RL to it, and how a work of
his addresses a particular challenge.

AI is already used as part of creative processes to -

- provide inspiration (e.g., [PowerPoint Designer](https://www.microsoft.com/en-us/microsoft-365/blog/2015/11/13/the-evolution-of-powerpoint-introducing-designer-and-morph/)
which automatically generates slide templates based on the content in them)
- automate tedium (e.g., Adobe Photoshop's [content-aware fill](https://helpx.adobe.com/photoshop/using/content-aware-fill.html) 
which can automatically and accurately fill in blank spaces in images based on the surrounding
content)
- creative collaboration (e.g., [Drawing Apprentice](https://adam.cc.gatech.edu/DrawingApprentice/)
uses an AI to co-create art with you that wouldn't be possible for a single artist)

It can be applied to games by -

- generating emergent gameplay with the game characters
- co-creating the games themselves

The first possibility has been explored using two creative installations which use improv to create
emergent dance and theatre.

## [LuminAI](https://expressivemachinery.gatech.edu/projects/luminai/) - Improv Dance

LuminAI does partner dance. Based on a user's dance moves, it will generate gestures that are
contextual and appropriate to the human performer. A challenge in implementing it was the
knowledge-authoring bottleneck of designing various gestures. The solution to it was to learn them
from users, learn the sequencing of these actions, and use procedural generation to generate variants
of these actions when none are available. The takeaway is to learn from users to avoid the
knowledge-authoring bottleneck.

## [Robot Improv Circus](https://expressivemachinery.gatech.edu/projects/robot-improv-circus/) - Improv Theatre

The Robot Improv Circus is a VR game where a participant plays the [Props game](http://www.improvgames.com/props/)
\- they are given a prop and must use it in a way that is surprising and humorous, drawing on the
abstract shape of the prop and various shared knowledge between the performer and the audience.

The challenge with this is that the action-set is very open-ended. Selecting an action in a time-sensitive
manner from this large pool is tricky even for humans. The speaker's group solved this challenge by
first creating a gesture dataset from humans interacting with a fixed set of props. They trained a
conditional VAE to generate variants of these actions. They used computational models of creativity
drawn from literature to evaluate each action's novelty, unexpectedness and quality. They then
selected actions based on how well they fit a predetermined *creative arc* through the dimension-space
defined by the aforementioned metrics.

The second half of the talk focused on how human-AI co-creation could be applied to game design and
development. The speaker first addressed the challenges that real-world game AI developers face when
using ML and RL techniques in their games. In the accompanying AIIDE 2020 [paper](https://www.microsoft.com/en-us/research/publication/its-unwieldy-and-it-takes-a-lot-of-time-challenges-and-opportunities-for-creating-agents-in-commercial-games/), the speaker
interviewed multiple such developers and thematically analyzed their survey responses. A subset of
these challenges are -

1. balancing designer control vs. agent autonomy
2. adapting agents at scale (generalizing across games)
3. automated game testing at scale

The speaker explored the first challenge in a [work done](https://www.microsoft.com/en-us/research/blog/designer-centered-reinforcement-learning/)
by a mentee of theirs. They explored how a designer could provide style-rewards to an agent to get
it to obey a desired style in its behaviour. Issues with reward hacking were solved using potential-
based reward shaping. This enabled expressive control over RL agents.

The speaker concluding with presenting the biggest challenge of this area as doing more human-centered
research into these techniques. The biggest opportunity is to get co-creation tools out of labs and
into the hands of creators.
