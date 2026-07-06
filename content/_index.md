---
title: Homepage
layout: hextra-home
---

{{< hextra/hero-badge link="https://github.com/KonnerLester1015/ByteBox-Nexus" >}}
  <div class="hx:w-2 hx:h-2 hx:rounded-full hx:bg-primary-400"></div>
  <span>ByteBox GitHub Repo</span>
  {{< icon name="arrow-circle-right" attributes="height=14" >}}
{{< /hextra/hero-badge >}}

<div class="hx:mt-6 hx:mb-6">
{{< hextra/hero-headline >}}
  Welcome to My Website!<br class="hx:sm:block hx:hidden" />
{{< /hextra/hero-headline >}}
</div>

<div class="hx:mb-12">
{{< hextra/hero-subtitle >}}
  Explore my projects and documentation<br class="hx:sm:block hx:hidden" />covering a wide range of technologies.
{{< /hextra/hero-subtitle >}}
</div>

<div class="hx:mb-6">
{{< hextra/hero-button text="View Documentation" link="docs" >}}
{{< hextra/hero-button text="View Project Showcase" link="projects" >}}
</div>

<div class="hx:mt-6"></div>

---

{{< hextra/feature-grid >}}

  {{< hextra/feature-card
    title="Technical Documentation"
    subtitle="Comprehensive guides, tutorials, and reference materials for various technologies and frameworks."
    class="hx:aspect-auto hx:md:aspect-[1.1/1] hx:max-md:min-h-[340px]"
    link="docs"
    image="/docs.png"
    imageClass="hx:top-[35%] hx:left-[31px] hx:w-[110%] hx:sm:w-[110%] hx:dark:opacity-80"
    style="background: radial-gradient(ellipse at 45% 80%,rgb(177, 204, 240),hsla(0,0%,100%,0));"
  >}}

  {{< hextra/feature-card
    title="Project Showcase"
    subtitle="Explore detailed descriptions and demonstrations of my major projects and full deployments."
    class="hx:aspect-auto hx:md:aspect-[1.1/1] hx:max-md:min-h-[340px]"
    link="projects"
    image="/project.png"
    imageClass="hx:top-[60%] hx:left-[22px] hx:w-[180%] hx:sm:w-[110%] hx:dark:opacity-80"
    style="background: radial-gradient(ellipse at 60% 70%,rgb(177, 204, 240),hsla(0,0%,100%,0));"
  >}}

  {{< hextra/feature-card
    title="About Me"
    subtitle="Learn more about my background, skills, and how to get in touch for collaboration opportunities."
    class="hx:aspect-auto hx:md:aspect-[1.1/1] hx:max-lg:min-h-[340px]"
    link="about/"
    image="/profile.png"
    imageClass="hx:top-[35%] hx:left-[36px] hx:w-[180%] hx:sm:w-[110%] hx:dark:opacity-80"
    style="background: radial-gradient(ellipse at 50% 90%,rgb(177, 204, 240),hsla(0,0%,100%,0));"
  >}}

  {{< hextra/feature-card
    title="Certifications"
    subtitle="View my verified certifications and achievements in various technologies and domains."
    class="hx:aspect-auto hx:md:aspect-[1.1/1] hx:max-lg:min-h-[340px]"
    link="certifications/"
    image="/certifications.png"
    imageClass="hx:top-[35%] hx:left-[27px] hx:w-[130%] hx:sm:w-[110%] hx:dark:opacity-80"
    style="background: radial-gradient(ellipse at 50% 90%,rgb(177, 204, 240),hsla(0,0%,100%,0));"
  >}}

  {{< hextra/feature-card
    title="Blog"
    subtitle="Read my latest articles on software development, tutorials, and insights from my journey in tech."
    class="hx:aspect-auto hx:md:aspect-[1.1/1] hx:max-md:min-h-[340px]"
    link="blog"
    image="/blog.png"
    imageClass="hx:top-[-90px] hx:left-[35px] hx:w-[150%] hx:sm:w-[110%] hx:dark:opacity-80"
    style="background: radial-gradient(ellipse at 50% 70%,rgb(177, 204, 240),hsla(0,0%,100%,0));"
  >}}

{{< /hextra/feature-grid >}}

<div class="hx:mt-6 hx:mb-6">
{{< hextra/hero-section >}}
  Powered By
{{< /hextra/hero-section >}}
</div>

{{< hextra/feature-grid >}}
  {{< hextra/feature-card
    title="Hugo"
    icon="template"
    subtitle="The world's fastest framework for building websites - generating sites in milliseconds with a single binary."
    link="https://gohugo.io/"
  >}}

  {{< hextra/feature-card
    title="Hextra Theme"
    icon="color-swatch"
    subtitle="A modern, responsive Hugo theme designed for beautiful documentation and project showcases."
    link="https://github.com/imfing/hextra"
  >}}
{{< /hextra/feature-grid >}}
