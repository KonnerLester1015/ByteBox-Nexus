# ByteBox Nexus

## Overview

This repository hosts the source code for **ByteBox Nexus**, a personal portfolio and technical documentation website. The site is designed to showcase projects and provide detailed, organized documentation (how-tos, guides, and technical notes) on various topics and technologies I've worked with.

## Key Features

* **Technology Documentation:** Organized guides and how-tos on a wide range of technologies.
* **Project Showcase:** Dedicated pages detailing significant projects and complex deployments (e.g., Ceph Cluster Deployment, n8n Automation).
* **Static Site Performance:** Built with Hugo, ensuring fast load times and high performance.

## Tech Stack

* **Framework**: [Hugo](https://gohugo.io/) (Static Site Generator)
* **Theme**: [Hextra](https://github.com/imfing/hextra) (Hugo Module)
* **Language**: Go, HTML, CSS, JavaScript
* **Deployment**: GitHub Pages via GitHub Actions

## Deployment

The site is automatically built and deployed to **GitHub Pages** whenever changes are pushed to the `main` branch. The deployment process is managed by the `.github/workflows/pages.yaml` file.

## Updating Hugo and the Theme

To update Hugo and the Hextra theme, follow these steps:

1. **Check for Hugo Updates**: Run `hugo version` to see the current Hugo version installed. If an update is available, download and install the latest version from [Hugo Releases](https://github.com/gohugoio/hugo/releases). Example command to update Hugo on Windows using Winget: `winget upgrade Hugo.Hugo.Extended`
2. **Update Hextra Theme**: Update the Hextra theme by running `hugo mod get -u github.com/imfing/hextra` in your project directory. This command updates the theme module to the latest version available in the repository.
