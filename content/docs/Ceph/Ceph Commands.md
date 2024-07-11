---
title: "Ceph Commands"
---

## Show Cluster Status
Used to show the status of the cluster.
```bash {filename="bash"}
ceph -s
```
Used to show the status of the cluster in real time
```bash {filename="bash"}
ceph -w
```

**Output**
```bash
cluster:
    id:     f41wsd8-9u6e-123f-9a8b-nGtsw33a293d
    health: HEALTH_OK

  services:
    mon: 3 daemons, quorum host01,host02,host03 (age 3w)
    mgr: host01.rdffgq(active, since 16h), standbys: host02.mvjfib, host03.pfurbb
    osd: 36 osds: 36 up (since 2w), 36 in (since 2w)

  data:
    pools:   4 pools, 97 pgs
    objects: 8.35k objects, 32 GiB
    usage:   18 TiB used, 655 TiB / 672 TiB avail
    pgs:     97 active+clean

  io:
    client:   3.7 KiB/s wr, 0 op/s rd, 0 op/s wr
```

## Display Pool Information
[Understanding the storage clusters usage stats](https://docs.redhat.com/en/documentation/red_hat_ceph_storage/4/html/administration_guide/monitoring-a-ceph-storage-cluster#understanding-the-storage-clusters-usage-stats_admin)
```bash {filename="bash"}
ceph df detail
```


Run `ceph df detail` with specific user. - [Cephx](https://docs.ceph.com/en/quincy/rbd/rbd-snapshot/#cephx-notes)
```bash
ceph --name client.qemu --keyring /etc/ceph/ceph.client.qemu.keyring df detail
```


**Output**
```bash
--- RAW STORAGE ---
CLASS     SIZE    AVAIL    USED  RAW USED  %RAW USED
hdd    672 TiB  655 TiB  18 TiB    18 TiB       2.61
TOTAL  672 TiB  655 TiB  18 TiB    18 TiB       2.61

--- POOLS ---
POOL  ID  PGS   STORED   (DATA)  (OMAP)  OBJECTS     USED   (DATA)  (OMAP)  %USED  MAX AVAIL  QUOTA OBJECTS  QUOTA BYTES  DIRTY  USED COMPR  UNDER COMPR
.mgr   1    1  1.0 MiB  1.0 MiB     0 B        2  3.0 MiB  3.0 MiB     0 B      0    207 TiB            N/A          N/A    N/A         0 B          0 B
rbd    2   32   32 GiB   32 GiB   217 B    8.34k   95 GiB   95 GiB   217 B   0.01    207 TiB            N/A       32 GiB    N/A         0 B          0 B
```

## List users in the storage cluster
[List Users](https://docs.redhat.com/en/documentation/red_hat_ceph_storage/3/html/administration_guide/user_management#list_users)
```bash {filename="bash"}
$ ceph auth list
```
**Output**
```
osd.0
        key: AQDCi2hmdCj4DhAAdRzfeHKOcu+5TTVGP94zhA==
        caps: [mgr] allow profile osd
        caps: [mon] allow profile osd
        caps: [osd] allow *
osd.1
        key: AQDCi2hmA8QwERAAjK/svMf6m+SXlF6N5h4X1g==
        caps: [mgr] allow profile osd
        caps: [mon] allow profile osd
        caps: [osd] allow *
client.Dobby
        key: AQDxD4NmI8MJABAAvH3diXZ6KHkLxMaMw4pKFQ==
        caps: [mgr] profile rbd pool=Hogwarts
        caps: [mon] profile rbd
        caps: [osd] profile rbd pool=Hogwarts
client.HarryPotter
        key: AQDvD4NmCM7BMBAAVSxI65bbIVmcfmJJUdC30w==
        caps: [mgr] profile rbd pool=Hogwarts
        caps: [mon] profile rbd
        caps: [osd] profile rbd pool=Hogwarts
```

## List cluster pool names
[LIST POOLS](https://docs.ceph.com/en/latest/rados/operations/pools/#list-pools)

```bash {filename="bash"}
$ ceph osd pool ls
```

**Output**
```
.mgr
rbd
Hogwarts
Marvel
```

## List Block Device Images
[LISTING BLOCK DEVICE IMAGES](https://docs.ceph.com/en/quincy/rbd/rados-rbd-cmds/#listing-block-device-images)

```bash {filename="bash"}
$ rbd ls [poolname]
```

Run with specific user:
```bash {filename="bash"}
$ rbd --id=qemu ls OS-Images
```

**Output**
```
windows2022-img
windows2022-img2
windows2022-img3
```
