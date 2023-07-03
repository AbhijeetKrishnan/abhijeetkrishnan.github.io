---
title: Projects
permalink: /projects/
mathjax: true
---

## Fanorona Engine in Rust [<i class="fab fa-github"></i>](https://github.com/AbhijeetKrishnan/rust-fanorona)

`rust-fanorona` is a Fanorona library written in Rust, with move generation capabilities and move validation. It was
written as an exercise in developing a medium-sized codebase in Rust and also to supplement the work done for the
[Fanorona AEC Environment](https://github.com/AbhijeetKrishnan/fanorona-aec), where the Rust-based engine could
potentially be plugged in to speed up execution.

## VSCode Markdown Reddit Spoiler [<i class="fab fa-github"></i>](https://github.com/AbhijeetKrishnan/vscode-markdown-reddit-spoiler) [<i class="fas fa-store-alt"></i>](https://marketplace.visualstudio.com/items?itemName=AbhijeetKrishnan.markdown-reddit-spoiler)

A VS Code extension written in TypeScript that implements the Reddit `>!spoiler!<` syntax to the built-in Markdown
preview. I found it useful to implement Q&A-style notes while studying *Reinforcement Learning: An Introduction (3rd ed.)* by Sutton and Barto.

## Ray Tracing in One Weekend (in Rust) [<i class="fab fa-github"></i>](https://github.com/AbhijeetKrishnan/rtweekend)

A Rust-based implementation of Ray Tracing in One Weekend by Peter Shirley. Aside from the fairly straightforward
translation of the C++ code to Rust, I also managed to parallelize the rendering code, allowing a speed-up of $\approx
72\%$ in rendering the final scene.

## Sokoban Solvability Predictor [<i class="fab fa-github"></i>](https://github.com/AbhijeetKrishnan/sokoban-solvability-predictor)

A Python and TensorFlow-based machine learning project attempting to build a model to predict the solvability of a
Sokoban level. The motivation was to test the capabilities of CNN-based models to solve a longer horizon planning task,
and also to simply develop an end-to-end machine learning application. The dataset was compiled from various collections
of Sokoban levels online, with their pre-processing and cleaning done by me. I used a fairly standard CNN-based model
that's also been used for image recognition tasks. The model is able to achieve an accuracy of $74.69\%$, which is
marginally better than the majority label rate of $72.5\%$ in the dataset.

## Fanorona AEC Environment [<i class="fab fa-github"></i>](https://github.com/AbhijeetKrishnan/fanorona-aec) [<i class="fab fa-python"></i>](https://pypi.org/project/fanorona-aec/)

An implementation of the board game [Fanorona](https://en.wikipedia.org/wiki/Fanorona) as an environment in the
[PettingZoo](https://github.com/PettingZoo-Team/PettingZoo) [AEC](https://arxiv.org/abs/2009.13051) interface. The
project is intended to be a test bed for a research project involving extracting *interpretable strategies* which can be
presented to a human player in order to improve their skill at Fanorona. It is expected that this work will extend to
uncovering similar strategies for any other task on which an RL-agent is trained.

## r/Tekken Bot [<i class="fab fa-github"></i>](https://github.com/AbhijeetKrishnan/r-tekken-bot)

A bot written in Python to provide various useful functionality for [r/Tekken](https://www.reddit.com/r/Tekken/), a subreddit dedicated to the Tekken fighting game
franchise. It automated the updates of the livestream widget that displayed the top Tekken Twitch streamers in real-time,
as well as the Tekken Dojo system that awarded points to helpful users who answered newcomer questions, along with presenting a
tabulated summary of the top-scorers and awarding them with a unique flair.

## Terragen - a procedural terrain generator using Perlin noise [<i class="fab fa-github"></i>](https://github.com/AbhijeetKrishnan/terragen)

A project developed for the final submission of CSC 562: Game Engine Design. It is a Javascript web application which
uses Perlin noise to generate simple 3D heightmaps using WebGL. It adds tree-like objects to the map (also using Perlin
noise) to improve realism.

## WhatsApp Chat Log Parser [<i class="fab fa-github"></i>](https://github.com/AbhijeetKrishnan/whatsapp-parser)

A small, Python-based script to parser a WhatsApp chat log into a csv file to enable subsequent analysis.