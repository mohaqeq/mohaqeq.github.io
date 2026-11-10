---
title: "Teaching kids to make games on a phone (in German)"
date: 2026-01-19
tags: [python, teaching, games, pygame-zero, android]
summary: "A German-language tutorial for children: thirteen small Pygame Zero games that run on an Android phone with Pydroid 3, from drawing shapes to a space shooter."
---

Children today usually have a phone or tablet long before they have a laptop. So instead of asking "do you have a
computer?", I wrote a game-programming tutorial that runs where they already are: on Android, in
[Pydroid 3](https://play.google.com/store/apps/details?id=ru.iiec.pydroid3), with
[Pygame Zero](https://pygame-zero.readthedocs.io/).

It's in German, because it was written for German-speaking kids:
[mohaqeq/pygame-zero-tutorial](https://github.com/mohaqeq/pygame-zero-tutorial).

## Why Pygame Zero

Pygame Zero removes almost all boilerplate. You define `draw()` and `update()`, and the framework runs the game loop.
A child's first program draws something within minutes, which is what keeps them going. Touch input maps onto the mouse
events, so the same code works on a phone screen.

## Two parts, thirteen games

**Part 1: Basics, no images needed.** Every game is drawn with shapes, so there are no assets to download or lose:

1. Drawing shapes
2. Understanding touch input
3. *Fang den Ball* (catch the ball)
4. A jumping box, like the Chrome dinosaur
5. *Flappy Box*, like Flappy Bird
6. Ping pong against the computer
7. Breakout
8. On-screen buttons for control

**Part 2: Images and sound.** Once the logic makes sense, sprites and sounds make it feel like a "real" game:

1. Flappy Bird with images
2. Collecting coins
3. A space shooter
4. Ping pong with effects
5. Animations

## How the material is built

- **One idea per game.** Each game adds one concept: coordinates, then input, then collision, gravity, a computer
  opponent, buttons.
- **Games kids already know.** Recreating a dinosaur jump or Flappy Bird gives them an immediate goal and a way to judge
  whether it works.
- **Shapes before sprites.** Part 1 needs no images at all, so nothing can go wrong with files on a phone.
- **Short, readable code** with German comments, meant to be typed in and changed, not just run.

If you know a German-speaking kid with a phone and some curiosity, the tutorial starts with
`Tutorial_Teil1_Grundlagen.md`.
