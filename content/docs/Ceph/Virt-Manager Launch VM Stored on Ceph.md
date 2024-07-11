---
title: "Virt-Manager: Launch VM Stored on Ceph"
---

## Introduction
This document provides essential steps for verifying the installation of Ceph client packages, converting and moving virtual machine disk images to a Ceph cluster, and launching these images from Virt-Manager. By following this guide, you will ensure a seamless process of managing VM disk files within your Ceph environment and leveraging the powerful features of Ceph's RBD pools for efficient storage and retrieval.


## Verify Ceph Client Package is Installed

To verify that the `ceph-common` is installed run the `which` command on a command from that package.
   ```bash {filename="bash"}
   which rbd
   ```
   **Output**
   ```
   /usr/bin/rbd
   ```
If the `ceph-common` is not installed then refer to [Install Client Ceph Packages](https://konnerlester1015.github.io/ByteBox-Nexus/docs/ceph/install-ceph-packages/) for getting that package added to your linux environment. 

## Move/Convert VM Disk Image to Ceph Cluster
If you have a existing qcow file or some other vm disk file that needs to be added onto the Ceph Cluster, follow the below procedure in order to store the disk image on an <abbr title="RADOS Block Device">RBD</abbr> pool.

*Note Ceph recommends using the `raw` format for VM disk files.*
```bash {filename="bash"}
qemu-img convert <file> rbd:<pool_name>/<new_image_name>
```
*Example*
```bash {filename="bash"}
qemu-img convert -f qcow2 -O raw windows_server_2022_2023_12_06.qcow2 rbd:rbd/windows2022-img
```


## Launch Image Stored on Ceph Cluster
Once an image exists on the Ceph Cluster an end user will be able to launch it from Virt-Manager. Refer to [Attaching RBD Storage Pool to Virt-Manager](/doc/ceph/attach-rbd-to-virt-manager/) for steps to access the Ceph pool from Virt-Manager.
1. Launch **virt-manager**
   ```bash {filename="bash"}
   sudo virt-manager
   ```
2. Click **Create a new virtual machine**
3. Select **Import existing disk image**
4. Click **Forward**
5. Click **Browse...**
6. Click **rbd**
7. Select **windows2022-img**
8. Click **Choose Volume**
9. Enter **Microsoft Windows Server 2022** in "Choose the operating system you are installing" field.
10. Click **Forward**
11. Click **Forward**
12. Click **Finish**