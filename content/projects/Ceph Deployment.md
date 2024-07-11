---
title: "Ceph Deployment for Working with QEMU VM"
sidebar:
  exclude: true
---
## Setup Ceph Cluster
### <cite>Install Cephadm [^1]</cite>

[^1]: [INSTALL CEPHADM - Ceph Docs](https://docs.ceph.com/en/quincy/cephadm/install/#install-cephadm)

There are two primary methods to install cephadm: using the curl-based installation method or distribution-specific installation methods. This documentation focuses on the curl-based installation method. For distribution-specific installation instructions, please refer to the Ceph documentation.

1. Fetch build of cephadm
   ```bash {filename="bash"}
   curl --silent --remote-name --location https://github.com/ceph/ceph/raw/quincy/src/cephadm/cephadm
   ```
2. Ensure cephadm is executable
   ```bash {filename="bash"}
   chmod +x cephadm
   ```
3. Install cephadm command
   ```bash {filename="bash"}
   ./cephadm add-repo --release quincy
   ./cephadm install
   ```

### <cite>Bootstrap Cluster [^2]</cite>

[^2]: [BOOTSRAP A NEW CLUSTER - Ceph Docs](https://docs.ceph.com/en/quincy/cephadm/install/#bootstrap-a-new-cluster)

Connect to the desired node to run the bootsrapper. Note that running the bootstrapper will do the following:
- Creates a Mon and Mgr daemon for the new cluster on the local host.
- Generates a new SSH key for the Ceph cluster (`/root/.ssh/authorized_keys`).
- Writes a copy of the public key to `/etc/ceph/ceph.pub`.
- Writes a minimal configuration file to `/etc/ceph/ceph.conf`. **Needed to communicate with Ceph daemons**.
- Writes a copy of the `client.admin` administrative key to `/etc/ceph/ceph.client.admin.keyring`.
- Adds the `_admin` label to the local host. Any host with this label will get a copy of `/etc/ceph/ceph.conf` and `/etc/ceph/ceph.client.admin.keyring`.

1. Run Boostrapper.
   ```bash {filename="bash"}
   cephadm bootstrap --mon-ip <Local_Host_IP>
   ``` 
   *Example*
   
   ```bash {filename="bash"}
   cephadm bootstrap --mon-ip 172.18.74.25 --allow-fqdn-hostname --cluster-network 192.168.74.0/24
   ```

### <cite>Set Designated Monitor Subnet [^3]</cite>

[^3]: [DESIGNATING A PARTICULAR SUBNET FOR MONITORS - Ceph Docs](https://docs.ceph.com/en/quincy/cephadm/services/mon/?highlight=set%20mon%20public#designating-a-particular-subnet-for-monitors)

To specify an IP subnet for Ceph monitor daemons, use this command in CIDR notation. This setting ensures that Cephadm deploys new monitor daemons exclusively on hosts within this subnet.

To execute `ceph [options] [arguments]`, launch a ceph shell or use `cephadm shell -- ceph [options] [arguments]`. Run the following commands from the previously bootstrapped host
1. Launch Ceph Shell.
   ```bash {filename="bash"}
   cephadm shell
   ```
2. Set designated public monitor subnet
   ```bash {filename="bash"}
   ceph config set mon public_network 172.18.74.0/24
   ```

### <cite>Add Additional Hosts to the Cluster [^4]</cite>

[^4]: [ADDING HOSTS - Ceph Docs](https://docs.ceph.com/en/quincy/cephadm/host-management/#adding-hosts)

To add a host into the Ceph cluster you must copy the public ssh key to the new host(s) as well as tell ceph to add the new host to the cluster:

#### Copy Cluster's Public SSH Key to New Host
1. Use the following command structure.
   ```bash {filename="bash"}
   ssh-copy-id -f -i /etc/ceph/ceph.pub root@<Hostname>
   ```

   *Example*
   ```bash {filename="bash"}
   ssh-copy-id -f -i /etc/ceph/ceph.pub root@host02.example.com
   ssh-copy-id -f -i /etc/ceph/ceph.pub root@host03.example.com
   ```

#### Tell Ceph Cluster to Add New Host
1. Use the following command structure.
   ```bash {filename="bash"}
   ceph orch host add <hostname> [IP] [--labels]
   ```
   
   *Example*
   ```bash {filename="bash"}
   ceph orch host add host2.example.com 172.18.74.26 --labels _admin
   ceph orch host add host3.example.com 172.18.74.27 --labels _admin
   ```

### <cite>Deploy OSDs [^5]</cite>

[^5]: [DEPLOY OSDS - Ceph Docs](https://docs.ceph.com/en/quincy/cephadm/services/osd/#deploy-osds)
[^6]: [ADVANCED OSD SERVICE SPECIFICATIONS - Ceph Docs](https://docs.ceph.com/en/quincy/cephadm/services/osd/#advanced-osd-service-specifications)

You can deploy <abbr title="Object Storage Daemons">OSDs</abbr> in two ways:

#### Consume all Devices
To instruct Ceph to use all available and unused storage devices automatically, run the following command:
1. Apply OSD
   ```bash {filename="bash"}
   ceph orch apploy osd --all-available-devices
   ```

#### Manually Set Specific Devices [^6]
To manually specify the devices on which OSDs will be deployed on a specific host, use the following command:
1. Create a yaml file (drivespec.yaml)
   ```yaml {filename="drivespec.yaml"}
   service_type: osd
   service_id: osd_using_paths
   placement:
   host_pattern: host0*.example.com
   spec:
   data_devices:  # Specify the data devices for the OSD. Stores primary data
      rotational: 1  # Match to HDDs
   db_devices:  # Specify the db devices for the OSD. Stores metadata
      rotational: 0  # Match to SSDs
   ```
2. Apply OSD
   
   *Orch apply from inside of Ceph shell container*
   ```bash {filename="bash"}
   ceph orch apply -i drivespec.yaml
   ```
   **Note you could apply the yaml from ouside the Ceph shell container with the following command.**

   *Orch apply from outside of Ceph shell container*
   ```bash {filename="bash"}
   cephadm shell --mount /root/drivespec.yaml:/mnt/drivespec.yaml -- ceph orch apply -i /mnt/drivespec.yaml
   ```

### <cite>Create RBD Pool</cite>

[^7]: [CREATING A POOL - Ceph Docs](https://docs.ceph.com/en/quincy/rados/operations/pools/#creating-a-pool)
[^8]: [ASSOCIATING A POOL WITH AN APPLICATION - Ceph Docs](https://docs.ceph.com/en/quincy/rados/operations/pools/#associating-a-pool-with-an-application)
[^9]: [CREATE A BLOCK DEVICE POOL - Ceph Docs](https://docs.ceph.com/en/quincy/rbd/rados-rbd-cmds/#create-a-block-device-pool)

Pools are a logical partition that are used to store objects. Any pool that starts with `.` is a reserved for Ceph operations. Run the following commands from withing the Ceph shell container (`cephadm shell`)

1. Create OSD Pool [^7]
   ```bash {filename="bash"}   
   ceph osd pool create {pool-name} [{pg-num} [{pgp-num}]] [replicated] [crush-rule-name] [expected-num-objects]
   ```
   *Example*
   ```bash {filename="bash"}
   ceph osd pool create rbd 1024 1024
   ```
2. Associate pool with RBD application [^8]
   ```bash {filename="bash"}
   ceph osd pool application enable {pool-name} {application-name}
   ```
   *Example*
   ```bash {filename="bash"}
   ceph osd pool application enable rbd rbd
   ```
3. Initialize the pool for use by RBD [^9]
   ```bash {filename="bash"}
   rbd pool init <pool-name>
   ```
   *Example*
   ```bash {filename="bash"}
   rbd pool init rbd
   ```

### Set Quota [^10]

[^10]: [SETTING POOL QUOTAS](https://docs.ceph.com/en/latest/rados/operations/pools/#setting-pool-quotas)

In cases where you want to allow end-user groups to utilize your Ceph cluster for block storage, it is important to implement a way to limit their storage utilization to prevent overuse. You can achieve this by creating a new pool for each user group and specifying the `max_bytes` size to control the pool's maximum allowable storage. 

1. Run `set-quota` command
   ```bash {filename="bash"}
   ceph osd pool set-quota {pool-name} [max_objects {obj-count}] [max_bytes {bytes}]
   ```
   *Example: Set "rbd" pool to have a max size of 10TB*
   ```bash {filename="bash"}
   ceph osd pool set-quota rbd max_bytes 10000000000000
   ```

### Create Client User [^11]

[^11]: [ADDING A USER](https://docs.ceph.com/en/quincy/rados/operations/user-management/#adding-a-user)

This section covers creating a user called **qemu**, applying capabilities (Ceph's terminology for permissions), and creating a keyring file for the user account. This account will be used on client machines to run and interact with the environment without the possibility of using administrator privileges.

1. Open terminal of a Ceph host
2. Run `get-or-create` command
   *Example*
   ```bash {filename="bash"}
   ceph auth get-or-create client.qemu mon 'profile rbd' osd 'profile rbd pool=rbd' mgr 'profile rbd pool=rbd'
   ```
   *This command creates a user (client.qemu) with read/write permissions to the RBD pool.*
3. Create user keyring file
   ```bash {filename="bash"}
   ceph auth get client.qemu -o /etc/ceph/ceph.client.qemu.keyring
   ```

{{< callout type="warning" >}}
  ***Note**:* 
If you have a lot of pools and users to make you can utilze a bash script that will create the pool, set the quota, and create the users with the correct permissions. Refer to [Bulk Create RBD Pools and Users](/ByteBox-Nexus/docs/ceph/bulk-create-rbd-pools-and-users/)
{{< /callout >}}

## Prepare Client Machine

### Retrieve Keyring and Config files
To interact with the Ceph Cluster from a client node, authentication is required. Typically, this is accomplished using the `ceph.client.{user}.keyring` and `ceph.conf` files. By default, when executing a Ceph command, the system will search for these files in `/etc/ceph/` unless specified otherwise.
1. Copy **ceph.conf**
   ```bash {filename="bash"}
   scp root@host01.example.com:/etc/ceph/ceph.conf .
   ```
2. Copy **ceph.client.{user}.keyring**
   ```bash {filename="bash"}
   scp root@host01.example.com:/etc/ceph/ceph.client.qemu.keyring .
   ```
### Install Ceph Client Package
To utilize Ceph commands a client machine needs to have the `ceph-common` package.
```bash {filename="bash"}
sudo yum install ceph-common
```
### Installing QEMU/KVM Packages
To run/emulate virtual machines the following packages should be installed.
 ```bash {filename="bash"}
sudo yum install qemu-kvm libvirt libvirt-daemon libvirt-daemon-config-network
 ```

## Running Virtual Machines (VMs) with libvirt
There are three scenarios administrators need to support when working with New VMs in the Ceph Cluster:

1. **Using an Existing VM Disk Image that is not stored on Ceph**: If the user already has a virtual machine disk image (e.g., qcow2, raw) stored outside of Ceph.

2. **Using an Existing VM Disk Image Stored on Ceph as a Block Device**: If the user's virtual machine disk image is already stored on Ceph as a block device.

3. **Creating a New Virtual Machine**: If the user doesn't have a virtual machine.

### Attach RBD to Virt-Manager
#### Create Secret [^12]

[^12]: [CONFIGURING THE VM - Ceph Docs](https://docs.ceph.com/en/quincy/rbd/libvirt/#configuring-the-vm)

1. Generate a secret.
    ```bash {filename="bash"}
    cat > secret.xml <<EOF
    <secret ephemeral='no' private='no'>
            <usage type='ceph'>
                    <name>client.qemu secret</name>
            </usage>
    </secret>
    EOF
    ```
2. Define the secret.
   ```bash {filename="bash"}
   sudo virsh secret-define --file secret.xml
   ```
   **Output**
   ```
   <secret UUID>
   ```
3. Save the key string to a file.

   *Note that this command will only work if the client.admin keyring is on the local host. If the admin keyring is not there you can connect to a ceph cluster host and run the following command and then move the `*.key` file to the client machine*
   ```bash {filename="bash"}
   ceph auth get-key client.qemu | sudo tee client.qemu.key
   ```
4. Set the UUID of the secret
   ```bash {filename="bash"}
   sudo virsh secret-set-value --secret <secret UUID> --base64 $(cat client.qemu.key) && rm client.qemu.key secret.xml
   ```
#### Create Pool XML [^13]

[^13]: [Virsh Storage Pool Commands](https://www.ibm.com/docs/en/linux-on-systems?topic=commands-storage-pool)
[^14]: [Libvert RBD Storage](https://libvirt.org/storage.html#rbd-pool)

1. Navigate to **/etc/libvirt/storage**
2. Create new pool **rbd.xml** (nano, vim, etc) [^14]
   ```xml {filename="rbd.xml"}
    <pool type="rbd">
    <name>rbd</name>
    <source>
        <name>rbd</name>
        <host name='host01.example.com' port='6789'/>
        <auth username='qemu' type='ceph'>
        <secret uuid='{uuid of secret}'/>
        </auth>
    </source>
    </pool>
   ```
3. Create a persistent definition of storage pool.
   ```bash {filename="bash"}
   virsh pool-define rbd.xml
   ```
4. Start the RBD storage pool.
   ```bash {filename="bash"}
   virsh pool-start rbd
   ```
5. Verfy RBD pool shows active.
   ```bash {filename="bash"}
   virsh pool-list
   ```

### Move/Convert VM Disk Image to Ceph Cluster [^15]

[^15]: [RUNNING QEMU WITH RBD](https://docs.ceph.com/en/quincy/rbd/qemu-rbd/#running-qemu-with-rbd)

If you have an existing qcow file or another VM disk file that needs to be added to the Ceph Cluster, follow the procedure below to store the disk image on an <abbr title="RADOS Block Device">RBD</abbr> pool. Note Ceph recommends using the `raw` format for VM disk files.
```bash {filename="bash"}
qemu-img convert <file> rbd:<pool_name>/<new_image_name>
```
*Example*
```bash {filename="bash"}
qemu-img convert -f qcow2 -O raw windows_server_2022_2023_12_06.qcow2 rbd:rbd/windows2022-img
```

### Launch VM Image Stored on Ceph Cluster
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


### Create New VM Image
#### Create Image RBD Pool [^16]

[^16]: [SNAPSHOT BASICS](https://docs.ceph.com/en/latest/rbd/rbd-snapshot/#snapshot-basics)

If you want to deploy a new OS image, you need to have a source from which to pull that image. For the best user experience, it is beneficial to have images where the OS is already installed, so the user doesn't need to run through the ISO file installation. To achieve this, we will create a new pool called `OS-Images`. Once we have this pool, we will push our ready-to-deploy OS image to the `OS-Images` pool, create a snapshot of the image, and then delete the original OS image.

1. Create `OS-Image` pool
   ```bash {filename="bash"}
   ceph osd pool create OS-Images
   ceph osd pool application enable OS-Images rbd
   rbd pool init OS-Images
   ```
2. Add OS Image
   ```bash {filename="bash"}
   qemu-img convert -f qcow2 -O raw windows_server_2022_2023_12_06.qcow2 rbd:OS-Images/windows2022
   ```
3. Create Snapshot
   ```bash {filename="bash"}
   rbd snap create OS-Images/windows2022@base
   ```
4. Protect Snapshot
   ```bash {filename="bash"}
   rbd snap protect OS-Images/windows2022@base
   ```

#### Deploy New VM [^17]

[^17]: [CLONING A SNAPSHOT](https://docs.ceph.com/en/latest/rbd/rbd-snapshot/#cloning-a-snapshot)

To request a VM, a user will have the ability to utilize any of the offered operating systems that we have snapshotted in the OS-Images pool. Ensure users have read access to the OS-Images pool or wherever you have the VM snapshot stored. For example, the qemu image has the following capabilities:
```
caps: [mgr] profile rbd pool=rbd, profile rbd-read-only pool=OS-Images
caps: [mon] profile rbd
caps: [osd] profile rbd pool=rbd, profile rbd-read-only pool=OS-Images
```

1. Clone OS Snapshot
   ```bash {filename="bash"}
   rbd clone {pool-name}/{parent-image-name}@{snap-name} {pool-name}/{child-image-name}
   ```
   *Example*
   ```bash {filename="bash"}
   rbd --id=qemu OS-Images/windows2022@base rbd/TestClone
   ```
