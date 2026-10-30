---
title: "Bragging About My Projects with Claude"
date: 2026-10-30
draft: false
tags:
  - AI
  - Claude
  - Projects
  - Tools
authors:
  - name: Konner Lester
    link: https://www.linkedin.com/in/konner-lester-925289302/
---

I've been building a lot of small projects lately, like [PlayDetour](https://playdetour.com) and my [Fantasy Football dashboard](https://konnerlester1015.github.io/FantasyFootball/). Building them turned out to be the easy part. Showing them off is where I kept stalling.

A screenshot doesn't capture how a site feels, and recording a screen demo and editing it by hand is a project all by itself. Then I found a Claude skill called **Brag**, and it fixed that problem.

<!--more-->

## What Is Brag?

[Brag](https://latent-spaces.github.io/brag/) is an agent skill that turns a project into a roughly twenty second launch video. You run one command, and it produces the video with music, motion, and share copy included. The tagline on its site is "you built it. now brag."

It was made by [@shunithaviv](https://github.com/shunithaviv), and the source is on [GitHub](https://github.com/latent-spaces/brag).

### Getting Set Up

There are two ways to install it, depending on how you work.

If you use Claude Code, add it as a plugin:

```bash
/plugin marketplace add latent-spaces/brag
/plugin install brag@brag
```

If you use another agent, install it with `npx`:

```bash
npx skills add https://github.com/latent-spaces/brag --skill brag
```

There's also a leaner version called `/brag-slim`, built to work with the tools already on your machine. Per the Brag site, `/brag` switches to it automatically on Opus 5.5, and you can run `/brag --full` if you want the original workflow. Check the Brag site for the current install details, since this is a young project and it may change.

Once it's installed, you point it at a project and tell your agent to brag about it.

## The Results

Here are three videos I made with it.

**Example 1: PlayDetour**
{{< video src="projects/detour/brag.mp4" title="PlayDetour showcase" >}}

**Example 2: Fantasy Football**
{{< video src="projects/fantasy-football/brag.mp4" title="Fantasy Football dashboard showcase" >}}

**Example 3: R6 GeoGuessr**
{{< video src="projects/r6-geoguessr/brag.mp4" title="R6 GeoGuessr showcase" >}}

### The Bonus: Free Infographics

Since a video is just a sequence of frames, brag is a great way to generate a stack of images that match your project. Many of the frames are good enough to use as thumbnails, social cards, or infographics.

## What I Liked and What to Keep in Mind

**What I liked**

- **Low effort.** There's no editing software and no timeline to fiddle with.
- **It fits the way I already work.** It's a skill inside the same agent I'm already using to build things.
- **Dead Simple.** I have never given it a prompt. I just run `/brag-slim` and it does the rest.

**Worth keeping in mind**

- **Review before you post.** It's a draft from an AI, so watch it and check that everything it says about your project is accurate.
- **Twenty seconds is short.** It's built for a quick hook, not a full walkthrough. For that, you still want a real demo.

## Give It a Try

If you've built something and never quite gotten around to showing it off, Brag is worth ten minutes. Install it, point it at a project, and see what comes back.