---
title: "Bulk Create RBD Pools and Users"
---
## Introduction
In scenarios where an administrator needs to create several RBD (RADOS Block Device) pools and corresponding users, executing individual commands for each task can be time-consuming and error-prone. To streamline this process, a Bash script can be used to automate the creation of RBD pools, setting of quotas, and assignment of user permissions.

The script below simplifies these operations by accepting specific inputs, thereby reducing the need for repetitive manual command execution. This not only speeds up the process but also ensures consistency and accuracy.

## Script Functionality
The provided Bash script takes the following inputs:

- `-n` (**pool name**): The name of the RBD pool to be created.
- `-u` (**users**): A comma-separated list of usernames to be created and granted access to the pool.
- `-q` (**quota**): The quota (in bytes) to be set for the specified pool.
  
{{< callout emoji="🔓" >}}
  By default, the specified users will receive read and write access to the created RBD pool.
{{< /callout >}}

## Detailed Steps Performed by the Script
1. **Create RBD Pool:**
   - The script creates a new RBD pool with the specified name.
   - It enables the RBD application on the pool.
   - It initializes the RBD pool.

2. **Set Quota:**
   - The script sets a quota on the pool to limit its maximum byte usage to the specified value.

3. **Create Users and Set Permissions:**
   - The script iterates through the provided list of usernames.
   - For each user, it creates a Ceph authentication key and sets the appropriate permissions, granting read and write access to the specified pool.


## BulkRBDPoolCreation.sh
```bash {filename="BulkRBDPoolCreation.sh"}
#!/bin/bash

# Function to display the usage of the script
usage() {
  echo "Usage: $0 -n <poolname> -u <username1,username2,...> -q <quota_in_bytes>"
  exit 1
}

# Parsing command-line arguments
while getopts "n:u:q:" opt; do
  case $opt in
    n) POOL_NAME=$OPTARG ;;
    u) USERNAMES=$OPTARG ;;
    q) QUOTA=$OPTARG ;;
    *) usage ;;
  esac
done

# Check if all required arguments are provided
if [ -z "$POOL_NAME" ] || [ -z "$USERNAMES" ] || [ -z "$QUOTA" ]; then
  usage
fi

# Create RBD pool
echo "Creating RBD pool: $POOL_NAME"
ceph osd pool create "$POOL_NAME"
ceph osd pool application enable "$POOL_NAME" rbd
rbd pool init "$POOL_NAME"

# Set quota on the pool
echo "Setting quota of $QUOTA bytes on pool: $POOL_NAME"
ceph osd pool set-quota "$POOL_NAME" max_bytes "$QUOTA"

# Create users and set permissions
IFS=',' read -ra USER_ARRAY <<< "$USERNAMES"
for USER in "${USER_ARRAY[@]}"; do
  echo "Creating user: client.$USER and setting permissions for pool: $POOL_NAME"
  ceph auth add client."$USER" mon 'profile rbd' osd "profile rbd pool=$POOL_NAME" mgr "profile rbd pool=$POOL_NAME"
done

echo "Script execution completed."
```

## Usage
The script can be executed with the necessary arguments as shown in the example below:
```bash
$ ./BulkRBDPoolCreation.sh -n Marvel -u Hulk,Spiderman,ironman -q 5000000000000
```

This command will create an RBD pool named **Marvel**, set its quota to **5,000,000,000,000** bytes or **5.6TiB**, and create users **Hulk**, **Spiderman**, and **ironman** with the necessary permissions for the pool.

By using this script, administrators can efficiently manage multiple pools and user permissions, ensuring a streamlined and error-free setup process.

```bash {filename="Output"}
Creating RBD pool: Marvel
pool 'Marvel' created
enabled application 'rbd' on pool 'Marvel'
Setting quota of 5000000000000 bytes on pool: Marvel
set-quota max_bytes = 5000000000000 for pool Marvel
Creating user: client.Hulk and setting permissions for pool: Marvel
added key for client.Hulk
Creating user: client.Spiderman and setting permissions for pool: Marvel
added key for client.Spiderman
Creating user: client.ironman and setting permissions for pool: Marvel
added key for client.ironman
Script execution completed.
```