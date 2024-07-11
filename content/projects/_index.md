---
title: Project Showcase ✨
layout: wide
sidebar:
  exclude: true
---
<div class="hx-mt-4"></div>

<p class="hx-mb-12 hx-text-center hx-text-lg hx-text-gray-500 dark:hx-text-gray-400">
Everything you need to know!
</p>

{{< cards >}}

  {{< card
        link="ceph-deployment"
        title="Ceph Deployment"
        subtitle="Full deployment of Ceph Development Cluster, goes over running boot strapper, adding hosts, creating OSDs, and Running VMs off a created RBD pool."
        image="/images/cephdashboard.png"
        imageStyle="object-fit:cover; aspect-ratio:16/9;"
  >}}

  {{< card
        link="pi-cluster"
        title="Raspberry Pi Cluster"
        subtitle="Cluster of Raspberry Pi's created to work with K3"
        image="/images/raspberry-pi-1.jpg"
        imageStyle="object-fit:cover; aspect-ratio:16/9;"
  >}}

  {{< card
        link="subnet-calculator"
        title="Subnet Calculator"
        subtitle="Calculator made to calculate network information provided with host IP and mac and subnet address."
        image="/images/subnetcalculator.png"
        imageStyle="object-fit:cover; aspect-ratio:16/9;"
  >}}

{{< /cards >}}
