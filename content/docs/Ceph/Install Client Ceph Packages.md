---
title: "Install Client Ceph Packages"
---
## Introduction
To interact with a Ceph cluster from client machines, it is necessary to install the appropriate Ceph client packages. This guide provides step-by-step instructions for installing the `ceph-common` package on Ubuntu and Fedora systems. If your Linux distribution does not include the `ceph-common` package, additional steps are provided to manually add the RPM package repository.


## Install Ceph-Common
**In Ubuntu**

```bash {filename="bash"}
$ apt install -y ceph-common
```

**In Fedora**

```bash {filename="bash"}
$ dnf install -y ceph-common
```

## RPM Package
If your Linux distribution cannot locate the `ceph-common` package, you may need to add the RPM manually.

1. Create `ceph.repo` under `/etc/yum.repos.d`
2. Add the following content
   ```bash {filename="ceph.repo"}
   [ceph]
    name=Ceph packages for $basearch
    baseurl=https://download.ceph.com/rpm-quincy/el8/$basearch
    enabled=1
    priority=2
    gpgcheck=1
    gpgkey=https://download.ceph.com/keys/release.asc
   ``` 
3. Run apt/dnf install again
   