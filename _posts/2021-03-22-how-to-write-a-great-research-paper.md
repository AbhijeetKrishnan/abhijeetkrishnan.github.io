---
title: "Video Summary - *How to write a great research paper* by Simon Peyton Jones (2016)"
tags: video-summary advice
mathjax: true
excerpt: "Summarizing and providing my notes on the talk - How to write a great research paper by Simon Peyton Jones (2016)"
---

Microsoft Research. (2016, July 16). How to Write a Great Research Paper [Video file]. Retrieved
from <https://youtu.be/VK51E3gHENc>
[<i class="fab fa-youtube"></i>](https://youtu.be/VK51E3gHENc)
{: .notice--info}

This is an (almost) hour-long video which contains excellent advice on how to write an academic
research paper. It mainly deals with how to structure the content of your paper, with an emphasis on
keeping the reader in mind. I'll summarize the 7 key points from the talk, with a lengthier
explanation following it.

## Summary

1. **Don't wait: write**


    Don't follow the typical research of process of idea > research > paper. Instead, do idea > paper >
research. Let the process of writing the paper guide your research. Writing your ideas down crystallises
them, forcing you to think more deeply about your idea, and revealing possibilities you did not
consider. It also creates a shareable artifact that you can use to invite collaboration and critique.

2. **Identify your key idea**

    Imagine your paper to be virus whose goal it is to infect the mind of your reader with *one* key idea. This idea should be something useful and re-usable that stays with the reader after they've finished reading your paper.

3. **Tell a story**

    Imagine explaining your research to a friend at a whiteboard. You would probably explain the problem first, describe why it is interesting and unsolved, talk about your idea and how it works (with details and data), then compare it with other approaches. Remember that you start losing readers from the first page onwards, so keep your paper engaging.

4. **Nail your contributions**

    Ensure you focus on your contributions in the 1st page - list them in bullet points and make them refutable. Use forward references to link to where you justify the contributions later in the paper.

5. **Related work: later**

    Related work can act as a barrier between your idea and the reader's mind. It is better to include it at the end after you've already explained your idea simply. Use them earlier if they help explain your solution path better (e.g. if you're building off of prior work), but don't list them for the sake of adding alternative approaches or examples. Give credit freely (it is not a zero-sum game).

6. **Put your readers first (examples)**

    Your problem is a labyrinth; in your research, you've encountered many dead-ends before hitting on the solution. You need to lead your readers comfortably to the solution. Address obvious objections your reader might have. Eschew technical-sounding prose since they will make your reader feel sleepy or stupid. A reader should be able to take something away from your paper even if they skip the details.

7. **Listen to your readers**

    Get your paper read by as many friendly guinea pigs as possible. Explain carefully what you want (confusion is better feedback than spelling errors). Value the first time readers and make it easy for them to give you feedback. Reviews are gold dust; people are donating their time to write a review for you. Take criticism positively.

***

## Don't wait: write

This was one of the best pieces of advice from the talk for me, and is something I've also slowly begun to see the benefit of. When writing my AML paper, I frequently found unjustified claims or methodological shortcomings that provided directions for literature review or software tests. Doing this right from the start of research is invaluable to getting your ideas down on paper, having a living document which you keep updating with your new insights and readings, and being able to have something to show for all your brainstorming.

Dr. Jones also advocates for submitting work in progress to appropriate publication venues. The main thrust behind this advice was to crystallise your ideas by the process of writing them down as a paper.

## Identify your key idea

Dr. Jones advises that your paper should contain one single key idea that you want to communicate to your reader. He uses the analogy of the idea being a virus that your paper should infect your reader's mind with. The main thrust behind this advice is to structure your paper around a single, central idea; one that is useful and re-usable, that will stick with readers after they've finished reading the paper.

He also encourages researchers to not be intimidated about the kind of ideas they feel they need to have to write a paper. You don't need to have an idea you think is "brilliant" before you start writing, it is okay to write a paper about a "simple" idea. If, as part of writing the paper, you realize that the idea is indeed unviable, that's okay. More often than not, you'll find some complexity that you hadn't foreseen, and end up with a decent bit of research. Write and communicate about your ideas no matter how insignificant they seem to you.

## Tell a story

Dr. Jones asks us to visualize explaining the core idea of our paper to a friend at a whiteboard. One could imagine following the same process in the paper as well. He emphasizes the importance of hooking your readers from the first page itself, since the number of readers who will stick with your paper drop off significantly after the first page.

## Nail your contributions

You should begin with introducing your problem and summarizing your contributions towards solving it. Dr. Jones advises against the common trope of explaining a much larger problem than the one you're trying to solve. As an example, don't waste words explaining work done in solving the (very general) problem of bugs in programs. Explain instead the problem of identifying and removing a specific type of bug.

Your contributions are what inform a reader as to whether they want to continue reading your paper. You need to advertise them clearly, in bullet points, with forward references to section numbers where you explain those contributions (or justify the claims you've made). Dr. Jones emphasises the need for these contributions to be *refutable*. For example, don't say that you "described the WizWoz system". Instead, say that you "give the syntax and semantics of a language that supports concurrent processes (Section 3)". Remember that the first page is all the time and words you've got to get your reader hooked.

## Related work: later

Related work is frequently used to pad the paper with the appearance of scholarliness by cramming into it as many examples of alternative approaches as possible. Frequently, these citations are very cursory, serving only to provide examples of something, and don't really indicate that you've read the paper. It is better to first explain the core idea of your paper simply, and include the related work section at the end. Having too many citations in your prose distracts the reader from what you actually want to convey.

The objective of your paper should be to create a path for the reader from their current state of knowledge to your idea. References should be included if they make creating this path easier. For example, if your work builds off of an existing work in the field, it makes sense to use it to aid your explanations. Any citation should be accompanied with a value-judgment, explaining how this work relates to your paper (in a way that helps the reader appreciate your core idea).

Don't include citations to make them look bad, and your work look good by contrast. Giving credit to others is not a zero-sum game where you lose face by giving due credit. Acknowledge inspirations for the approach you're taking. Lastly, acknowledge weaknesses in your paper (towards the end).

## Put your readers first (examples)

Make sure to always keep your reader in mind when writing your paper, and make decisions while asking yourself what would make things more understandable for your readers. Imagine the problem you're trying to solve as a labyrinth. In your research, you've traveled down various promising corridors, only to be met with obstacles or dead ends. After sustained effort, you've managed to hit upon the exit. Your job should be to guide the reader along a much shorter, simpler path directly to the exit.

To do so, ask yourself about potential objections or alternative, seemingly simpler approaches your reader might have and address these pre-emptively. Be careful with using impressive, technical-sounding paragraphs lest you make your reader feel sleepy or stupid. Remember that conveying the intuition behind your paper is primary, and that the reader should be able to take this intuition away even if they skip the details of your paper. Use examples.

In communicating the essence of your paper, don't skip out on the details that make for reproducibility. A potential concern is simplifying your paper to such an extent that a reader begins to doubt the value of your contributions. Balance readability with showing your reader the failed solution pathways that indicate how the problem is more complex than it seems. You can use your own research or rely on the work of others for this.

## Listen to your readers

You should get your paper read by as many "friendly guinea pigs" as possible, and as soon as possible. You want to also explain to them what you actually want as feedback - "I got confused by this paragraph" is much more valuable than "there is a spelling mistake here".

Remember that any reader can only read your paper for the first time once, so getting valuable feedback from them is crucial. Make it as easy as possible for them to give them feedback, especially when they might be uncomfortable. Confusing paragraphs where the reader might feel stupid for not understanding things should be addressed.

Getting feedback from experts is very valuable. Share your work with them when you're nearly done (i.e. you have a completed draft), and especially if you've cited them. Ask them if you've described their work fairly, and you'll very likely receive very helpful responses. These experts are likely to be referees for conferences you might submit your work to anyway, so getting early feedback from them is a big plus.

Lastly, treat every piece of feedback you receive like gold dust. Be truly grateful for every bit of praise and criticism. Remember that readers and reviewers are donating their time on this Earth to you. The hour it took them to read your paper (or write a review) put them literally an hour closer to death. Respect their time and their feedback. Even if the feedback seems hostile or "stupid", ask yourself how you could rewrite your paper so that this seemingly "stupid" reviewer can still understand it.