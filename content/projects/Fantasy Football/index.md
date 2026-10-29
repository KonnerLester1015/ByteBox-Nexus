---
title: "Fantasy Football"
sidebar:
  exclude: true
---

## Introduction

[**Fantasy Football**](https://konnerlester1015.github.io/FantasyFootball/) is a screenshot friendly dashboard for any Sleeper league. It turns raw league data into a friendly comic-book-style experience with current standings, playoff odds, optimal-lineup and all-play metrics, and weekly recaps.

The site was built to make fantasy football history feel as enjoyable to explore as it is to compete in. Instead of hiding the numbers in a spreadsheet, every metric is presented with plain language explanations, visual comparisons, and a warm inked visual system.

{{< callout type="info" >}}
  **Try it live:** [konnerlester1015.github.io/FantasyFootball](https://konnerlester1015.github.io/FantasyFootball/)  
  **Source code:** [github.com/KonnerLester1015/FantasyFootball](https://github.com/KonnerLester1015/FantasyFootball)
{{< /callout >}}

<video controls preload="metadata" src="brag.mp4" title="Fantasy Football dashboard showcase"></video>

## Objectives

- Turn Sleeper league data into an approachable visual dashboard.
- Support any league through a league ID or Sleeper username lookup.
- Explain derived metrics such as luck, all-play percentage, lineup efficiency, and playoff odds.
- Preserve a league's story across linked seasons with history and record-book views.
- Keep the application static and deployable on GitHub Pages without a backend.
- Make the experience responsive and accessible

## Features

### Current season

The current-season dashboard combines the league's standings with scoring, points against, maximum possible points, lineup efficiency, all-play percentage, schedule luck, and simulated playoff odds. Sort controls let the viewer compare teams by standings order, points, or playoff odds.

### Playoff forecast

The playoff page uses repeated simulations of the remaining schedule to estimate each team's odds of reaching the postseason, winning out, being eliminated, or finishing in a particular record range. The page labels estimates clearly so they are not confused with official results.

### Weekly recap

Each completed week gets a recap with the high and low scores, biggest and closest wins, broken records, score bars, and a team-by-team table. This creates a compact visual summary of what happened without requiring the viewer to reconstruct it from matchup data.

### Record book and team pages

The record book walks through every season Sleeper links to the current league and highlights single-week, career, scoring, luck, injury, and championship records. Individual team pages collect season numbers, weekly results, playoff paths, and simulated finish outcomes in one place.

## Program Architecture

The project is an npm-workspaces monorepo with two layers:

| Layer | Responsibility |
| --- | --- |
| `apps/web` | React 19, Vite, and TypeScript single-page application hosted on GitHub Pages |
| `packages/core` | Pure TypeScript for Sleeper API normalization, metric calculations, playoff simulations, and history |

The browser fetches public Sleeper data directly. The web app passes the raw responses to the framework-independent core package, which computes the dashboard model before React renders the pages. The only persisted browser state is the last league opened and a short-lived cache of Sleeper's player dictionary.

## Routes

Every dashboard route is scoped to a league ID:

| Route | Purpose |
| --- | --- |
| `#/` | Enter a league ID or find a league by Sleeper username |
| `#/<leagueId>` | Current-season standings and metrics |
| `#/<leagueId>/playoffs` | Playoff odds, outlook, and brackets |
| `#/<leagueId>/history` | Cross-season record book and career table |
| `#/<leagueId>/summary` | Weekly recap |
| `#/<leagueId>/team/<rosterId>` | Team detail and weekly metrics |

Hash routing keeps the app compatible with static GitHub Pages hosting while preserving direct navigation within a league.

## Design and Styling

The visual language is intentionally playful: a warm tan ground, paper panels, irregular  corners, hard offset shadows, football and helmet illustrations, and a yellow active state accent. **Londrina Solid** gives headings and large numbers their poster like weight, while **Patrick Hand** keeps labels and explanations conversational.

Motion is used for page load-ins, chart bars, hover states, and the animated football. Every animation respects `prefers-reduced-motion`, and charts keep their numeric values visible so color and motion are never the only way to understand a result.

## Technology Stack

- React 19
- TypeScript
- Vite
- Motion
- Sleeper's public API
- GitHub Pages

There is no backend or environment configuration. A production build is a static site, and the deployment workflow runs the project's lint, type-check, and test gates before publishing it.

## Summary

Fantasy Football combines data-heavy league analysis with a deliberately expressive interface. It is both a reusable dashboard for any Sleeper league and a creative exercise in turning a familiar spreadsheet workflow into an interactive historical narrative.
