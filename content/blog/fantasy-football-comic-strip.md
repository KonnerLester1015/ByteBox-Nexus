---
title: "Giving My Fantasy Football League a Statistical Makeover"
date: 2026-10-29
draft: false
tags:
  - Projects
  - APIs
  - AI
image: content/projects/Fantasy Football/brag.jpg
authors:
  - name: Konner Lester
    link: https://www.linkedin.com/in/konner-lester-925289302/
---

I started playing fantasy football this year. Within a few weeks, a few people in my league were asking the question: *"Is there a better way to see what's actually going on?"*

The Sleeper app is great at running matchups and keeping track of scores, but it doesn't give a comprehensive view of the season. It doesn't show how your team is doing compared to the rest of the league, how much luck is affecting your record, or what your playoff odds are.

So I built something to fix that.

<!--more-->

## The Idea

**[Fantasy Football](/projects/fantasy-football/)** is a dashboard for Sleeper leagues, drawn like a comic book. You enter a league ID or a Sleeper username, pick your league, and get a warm, inked, paper-panel view of your season: standings, weekly recaps, playoff odds, and a record book that follows your league across the season.

You can try it here: [https://konnerlester1015.github.io/FantasyFootball/](https://konnerlester1015.github.io/FantasyFootball/)

I originally built it for my own league. Then I realized nothing about it was specific to my league. If someone else entered their own ID, it should just work. So I added the front door, and now it works for anyone with a Sleeper league.

---

{{< video src="projects/fantasy-football/brag.mp4" title="Fantasy Football dashboard showcase" >}}

## Stats Sleeper Doesn't Show You

The part I enjoyed most was digging into numbers that don't appear in the app. A few of the favorites:

- **All-play record.** How your team would have done if it played every other team every week. It's a much more honest picture than your actual record.
- **Lineup efficiency.** Your score compared to the best lineup you could have set.
- **Schedule luck.** How much your record was helped or hurt by *who* you happened to face.
- **Playoff odds.** The remaining schedule is simulated over and over to estimate each team's chances.

## Why a Comic Book?

Since fantasy football is supposed to be fun, and most stat pages look like a tax form.

The design leans into that on purpose: a warm tan background, paper panels, hard offset shadows, and a lot of footballs and helmets. I took a lot of inspiration for The Peanuts comic strip.

## The Sleeper API Made This Easy

The best thing about this project is that **Sleeper's API is completely public**. No API keys, no tokens, no secrets to hide or leak. The browser asks for league data and gets it.

That shaped the whole architecture:

- A **React + TypeScript + Vite** front end.
- A separate, framework free **core package** that turns raw Sleeper responses into metrics, simulations, and league history.
- **No backend at all.** It's a static site on GitHub Pages, and the only thing stored in your browser is the last league you opened.

Since there's no server and nothing sensitive, I don't have to worry about protecting anyone's data. It also costs nothing to host.

I've spent plenty of time working with APIs in enterprise environments, where authentication, permissions, and change control are most of the job. Working with an open API where I could just *build the thing* was a refreshing change of pace.

## About That Contributors List

If you look at the [repository](https://github.com/KonnerLester1015/FantasyFootball), you'll see Claude listed as a contributor. That's accurate, and I'm happy to say so.

I'm not a professional programmer. My background is in infrastructure and enterprise systems, not React and TypeScript. Claude was a big part of turning what was in my head into a working app. The useful part wasn't just generating code. It was learning how to *direct* it: giving clear constraints, showing examples of what I wanted, and keeping it on track when it drifted. Prompting turned out to be a skill in its own right, and I got better at it with every iteration.

## What I Took Away

This was a small project with some surprisingly good lessons:

1. **Build for a real audience, even a tiny one.** My friends and family kept asking for things, and that made the project better than anything I'd have designed in isolation.
2. **A public API removes a lot of friction.** No keys and no backend meant I spent my time on the interesting parts.
3. **Good explanations matter as much as good numbers.** A stat nobody understands is just decoration.
4. **AI works best with a clear vision behind it.** The more specific I was, the better the results. The speed of getting from idea to working feature is something I couldn't have managed alone.

## Take It for a Spin

If you play in a Sleeper league, give it a try with your own league ID and see how your season really looks. If you find a stat you wish it had, or something that looks off, I'd like to hear about it. Issues and ideas are welcome on [GitHub](https://github.com/KonnerLester1015/FantasyFootball).