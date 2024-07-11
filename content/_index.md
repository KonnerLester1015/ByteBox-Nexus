---
title: Homepage
layout: hextra-home
disableSidebar: false
type: default
width: normal
---

{{< hextra/hero-badge link="https://github.com/KonnerLester1015/ByteBox-Nexus">}}
  <div class="hx-w-2 hx-h-2 hx-rounded-full hx-bg-primary-400"></div>
  ByteBox Github Repo
  {{< icon name="arrow-circle-right" attributes="height=14" >}}
{{< /hextra/hero-badge >}}

<div class="hx-mt-6 hx-mb-6">
{{< hextra/hero-headline >}}
  Welcome to My Website!<br class="sm:hx-block hx-hidden" />
{{< /hextra/hero-headline >}}
</div>

<div class="hx-mb-12">
{{< hextra/hero-subtitle >}}
  Explore my projects and documentation<br class="sm:hx-block hx-hidden" />covering a wide range of technologies
{{< /hextra/hero-subtitle >}}
</div>


<div class="hx-mb-6">
{{< hextra/hero-button text="View Documentation" link="docs" >}}
{{< hextra/hero-button text="Project Showcase" link="projects" >}}
</div>


<div class="hx-mt-6"></div>

{{< hextra/feature-grid >}}
  {{< hextra/feature-card
    title="Project Showcase"
    subtitle="Explore detailed descriptions and demonstrations of my major projects and full deployments."
    class="hx-aspect-auto md:hx-aspect-[1.1/1] max-md:hx-min-h-[220px]"
    link="/projects"
    image="images/project.png"
    imageClass="hx-top-[100%] hx-left-[36px] hx-w-[140%] sm:hx-w-[110%] dark:hx-opacity-80"
    style="background: radial-gradient(ellipse at 60% 70%,rgb(177, 204, 240),hsla(0,0%,100%,0));"
  >}}
  {{< hextra/feature-card
    title="About Me"
    subtitle="Learn more about me and how to get in touch."
    class="hx-aspect-auto md:hx-aspect-[1.1/1] max-lg:hx-min-h-[340px]"
    link="/about"
    image="images/profile.png"
    imageClass="hx-top-[80%] hx-left-[36px] hx-w-[140%] sm:hx-w-[110%] dark:hx-opacity-80"
    style="background: radial-gradient(ellipse at 50% 90%,rgb(177, 204, 240),hsla(0,0%,100%,0));"
  >}}
  {{< hextra/feature-card
    title="Full Text Search"
    subtitle="Utilize the built-in text search to quickly and easily find the information you need within the documentation."
    class="hx-aspect-auto md:hx-aspect-[1.1/1] max-md:hx-min-h-[100px]"
    image="images/search.png"
    imageClass="hx-top-[100%] hx-left-[20px] hx-w-[180%] sm:hx-w-[110%] dark:hx-opacity-80"
    style="background: radial-gradient(ellipse at 45% 80%,rgb(177, 204, 240),hsla(0,0%,100%,0));"
  >}}
{{< /hextra/feature-grid >}}

<div class="hx-mt-6 hx-mb-6">
{{< hextra/hero-headline >}}
  References<br class="sm:hx-block hx-hidden" />
{{< /hextra/hero-headline >}}
</div>

<div class="hx-mt-6"></div>

{{< hextra/feature-grid >}}
  {{< hextra/feature-card
    title="Hugo"
    icon="template"
    subtitle="The world’s fastest framework for building websites"
    link="https://gohugo.io/"
  >}}
  {{< hextra/feature-card
    title="Hextra Hugo Theme"
    icon="color-swatch"
    subtitle="Powered by Hugo, one of *the fastest* static site generators, building your site in just seconds with a single binary."
    link="https://github.com/imfing/hextra"
  >}}
{{< /hextra/feature-grid >}}