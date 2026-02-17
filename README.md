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

## Folder Structure

```
ByteBox-Nexus/
├── .github/
│   └── workflows/        # GitHub Actions CI/CD pipeline (auto-build & deploy)
├── archetypes/           # Content templates for new pages (hugo new ...)
├── content/              # All site content written in Markdown
├── data/                 # Structured data files (JSON/YAML/TOML) used by templates
├── i18n/                 # Internationalization and translation strings
├── layouts/              # Custom HTML templates that override the Hextra theme
├── static/               # Static assets (images, files) served directly as-is
├── .gitignore
├── go.mod                # Go module definition (manages the Hextra theme dependency)
├── go.sum                # Go module checksum file
├── hugo.yaml             # Main Hugo site configuration
└── README.md
```

Most day-to-day work happens in the `content/` directory. Adding or editing Markdown files here is all that's needed to publish new pages or guides.

## Running Locally

To preview the site on your local machine:

1. Install [Hugo Extended](https://gohugo.io/installation/) (Extended version required for SCSS support).
2. Clone the repository and navigate into it:
   ```bash
   git clone https://github.com/KonnerLester1015/ByteBox-Nexus.git
   cd ByteBox-Nexus
   ```
3. Start the local dev server:
   ```bash
   hugo server
   ```
4. Open `http://localhost:1313` in your browser. The site will live-reload as you make changes.

## Deployment

The site is automatically built and deployed to **GitHub Pages** whenever changes are pushed to the `main` branch. The deployment process is managed by the `.github/workflows/pages.yaml` file. No manual deployment steps are needed.

## Updating Hugo and the Theme

To update Hugo and the Hextra theme, follow these steps:

1. **Check for Hugo Updates**: Run `hugo version` to see the current Hugo version installed. If an update is available, download and install the latest version from [Hugo Releases](https://github.com/gohugoio/hugo/releases). Example command to update Hugo on Windows using Winget:
   ```bash
   winget upgrade Hugo.Hugo.Extended
   ```
2. **Update Hextra Theme**: Update the Hextra theme by running the following in your project directory:
   ```bash
   hugo mod get -u github.com/imfing/hextra
   ```
   This updates the theme module to the latest available version.

## Contributions

![GitHub Contributions](https://isometric-contributions-spectrewolf8.onrender.com/api/graph?username=konnerlester1015&credit=true&stats=true)
