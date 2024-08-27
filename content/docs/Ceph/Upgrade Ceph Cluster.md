---
title: "Upgrade Ceph Cluster"
---
## Introduction
Upgrading your Ceph cluster is a crucial process to ensure you are leveraging the latest features, security updates, and performance improvements. This document provides a step-by-step guide to upgrading your Ceph cluster, focusing on transitioning to version 18.2.2. The process involves verifying cluster health, initiating the upgrade, and monitoring the progress to ensure a smooth and successful upgrade. By following these procedures, you can maintain the robustness and efficiency of your Ceph storage solution.

## Procedure [^1]

[^1]: [STARTING THE UPGRADE](https://docs.ceph.com/en/quincy/cephadm/upgrade/#starting-the-upgrade)

1. Ensure your cluster is healthy and all nodes are online

   ```bash {filename="bash"}
   ceph -s
   ```
2. Run `ceph orch upgrade start` command [^2]

[^2]: [CEPH RELEASE INDEX](https://docs.ceph.com/en/latest/releases/#ceph-releases-index)

   ```bash {filename="bash"}
   ceph orch upgrade start --ceph-version <version>
   ```
   *Example: upgrade to version 18.2.2*
   ```bash {filename="bash"}
   ceph orch upgrade start --ceph-version 18.2.2
   ```

## Monitor Upgrade
You can use the `ceph orch upgrade status` command to check if the upgrade is in progress.
You can also utlize the `ceph -s` (`ceph -w`) command to watch the progress bar and receive updates as to the progress of the upgrade.
   ```bash {filename="bash"}
   ceph -w
   ```
   *Output*
   ```bash {filename="bash"}
   cluster:
     id: ad99432c-437e-11ef-8f76-86577d660b4a
     health: HEALTH_OK
   
   services:
     mon: 3 daemons, quorum host01, host02, host03 (age 5w)
     mgr: host01.fhdyeg(active, since 13d), standbys: host2.vhndye, host03.avdefo
     osd: 36 osds: 36 up (since 4w), 36 in (since 4w)
   
   data:
     pools:   6 pools, 161 pgs
     objects: 8.19k objects, 32 GiB
     usage:   18 TiB used, 655 TiB / 672 TiB avail
     pgs :    161 active+clean

   progress:
     Upgrade to 18.2.2 (21s)
       [===.........................]

    2024-07-16T10:16:44.270101-0400 mon. host01 [INF] Manager daemon host01.fhdyeg is now available
    2024-07-16T10:18:34.067874-0400 mon. host01 [INF] Manager daemon host2.vhndye is now available
    2024-07-16T10:19:41.460139-0400 mon. host01 [INF] mon. host01 calling monitor election
   ```

