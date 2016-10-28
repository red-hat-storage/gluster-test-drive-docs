# Lab Guide <br/> Gluster Test Drive Module 2 <br/> Volume Setup and Client Access

## Lab Agenda

Welcome to the Gluster Test Drive Module 2 - Volume Setup and Client Access. In this lab we will:

- Create a Gluster trusted pool
- Manually create a Gluster distributed volume
- Automatically create a Gluster replicated volume using gdeploy
- Understand basic Gluster CLI commands
- Write files to our volumes with the Gluter native client, NFS, and SMB
- Observe the data layout on the Gluster bricks


## Creating the Trusted Pool

Connect to the **rhgs1** server instance using its public IP address from the **Addl. Info** tab (Linux/Mac example below).

```bash
ssh gluster@<rhgs1PublicIP>
```

The first step in creating a Gluster trusted pool is node peering. From one node, all of the other nodes should be peered using the gluster CLI. Our lab has 6 Gluster nodes in the local subnet. You will run `peer probe` commands from only node **rhgs1**.

```bash
sudo gluster peer probe rhgs2
sudo gluster peer probe rhgs3
sudo gluster peer probe rhgs4
sudo gluster peer probe rhgs5
sudo gluster peer probe rhgs6
```

Note the success Messages:

``peer probe: success.``

A *trusted pool* is defined as a group of Gluster nodes peered together for the purpose of sharing their local storage and compute resources for one or more logical filesystem namespaces. The state of a trusted pool and its members can be viewed with two important commands: `gluster peer status` and `gluster pool list`.

```bash
sudo gluster peer status
```

Note the `peer status` command only reports the remote peers of the local node from which the command is run, excluding itself (localhost) from the list. This can be confusing for first-time users.

``Number of Peers: 5``

``Hostname: rhgs2``
``Uuid: 64edb288-524e-4918-b421-74b76b80a44b``
``State: Peer in Cluster (Connected)``

``Hostname: rhgs3``
``Uuid: e22261c4-3e34-49e3-b4e3-da9a8c449471``
``State: Peer in Cluster (Connected)``

``Hostname: rhgs4``
``Uuid: fbd4ab10-f50b-4ca8-b973-a9a54dfed47d``
``State: Peer in Cluster (Connected)``

``Hostname: rhgs5``
``Uuid: c351dead-7326-4370-a6fe-90d7b5ac8b45``
``State: Peer in Cluster (Connected)``

``Hostname: rhgs6``
``Uuid: 8b8384ae-cc6e-4403-ab3b-b7cc3c21029f``
``State: Peer in Cluster (Connected)``


The `pool list` command provides similar output in a tabular format and includes the local node, thus giving a complete view of the Gluster trusted pool.

```bash
sudo gluster pool list
```

``UUID					Hostname 	State``
``64edb288-524e-4918-b421-74b76b80a44b	rhgs2    	Connected ``
``e22261c4-3e34-49e3-b4e3-da9a8c449471	rhgs3    	Connected ``
``fbd4ab10-f50b-4ca8-b973-a9a54dfed47d	rhgs4    	Connected ``
``c351dead-7326-4370-a6fe-90d7b5ac8b45	rhgs5    	Connected ``
``8b8384ae-cc6e-4403-ab3b-b7cc3c21029f	rhgs6    	Connected ``
``c0f78e93-ad8f-4849-b8f3-ebdb2f7f4d17	localhost	Connected``



## Creating a Distributed Volume

A unit of storage on a node is referred to as a *brick*. A brick is simply a local filesystem that has been presented to the Gluster system for consumption. Each brick has an associated `glusterfsd` process on its system.

When a Gluster volume is created, its default architecture is *distribution*. A distributed volume simply groups storage from different bricks together into one unified namespace, resulting in a Gluster volume as large as the sum of all of the bricks. This architecture uses a hashing algorithm for pseudo-random distribution of files across the the bricks, resulting in statistically even distribution of files across the bricks.

We will use the gluster CLI to create a 2-brick distributed volume named *distvol*. Note that for the sake of this lab the backing filesystems on the nodes have been pre-configured and mounted to `/rhgs/brick_xvdb`. You can view the LVM and filesystem configurations with the commands below.

```bash
sudo vgdisplay -v /dev/rhgs_vg1
```

``    Using volume group(s) on command line.``
``  --- Volume group ---``
``  VG Name               rhgs_vg1``
``  System ID             ``
``  Format                lvm2``
``  Metadata Areas        1``
``  Metadata Sequence No  7``
``  VG Access             read/write``
``  VG Status             resizable``
``  MAX LV                0``
``  Cur LV                2``
``  Open LV               1``
``  Max PV                0``
``  Cur PV                1``
``  Act PV                1``
``  VG Size               10.00 GiB``
``  PE Size               4.00 MiB``
``  Total PE              2559``
``  Alloc PE / Size       2559 / 10.00 GiB``
``  Free  PE / Size       0 / 0   ``
``  VG UUID               sHGNaI-ODzz-aZV3-j602-aDZQ-DUUU-ddbK7X``
   
``  --- Logical volume ---``
``  LV Name                rhgs_thinpool1``
``  VG Name                rhgs_vg1``
``  LV UUID                A3FlQv-X8K7-IQE0-cYUZ-eEJk-ROVe-hkboxV``
``  LV Write Access        read/write``
``  LV Creation host, time rhgs1.gluster.lab, 2016-07-27 10:56:26 -0400``
``  LV Pool metadata       rhgs_thinpool1_tmeta``
``  LV Pool data           rhgs_thinpool1_tdata``
``  LV Status              available``
``  # open                 2``
``  LV Size                9.97 GiB``
``  Allocated pool data    0.11%``
``  Allocated metadata     0.65%``
``  Current LE             2553``
``  Segments               1``
``  Allocation             inherit``
``  Read ahead sectors     auto``
``  - currently set to     8192``
``  Block device           253:2``
   
``  --- Logical volume ---``
``  LV Path                /dev/rhgs_vg1/rhgs_lv1``
``  LV Name                rhgs_lv1``
``  VG Name                rhgs_vg1``
``  LV UUID                L2f6yD-NhfH-2Mm7-gX5S-b8Ee-2dXL-Pb0HGK``
``  LV Write Access        read/write``
``  LV Creation host, time rhgs1.gluster.lab, 2016-07-27 10:56:49 -0400``
``  LV Pool name           rhgs_thinpool1``
``  LV Status              available``
``  # open                 1``
``  LV Size                10.00 GiB``
``  Mapped size            0.11%``
``  Current LE             2560``
``  Segments               1``
``  Allocation             inherit``
``  Read ahead sectors     auto``
``  - currently set to     8192``
``  Block device           253:4``
   
``  --- Physical volumes ---``
``  PV Name               /dev/xvdb     ``
``  PV UUID               OmPpve-V6qZ-fcvx-DFMq-nrPr-Aeoh-0X2FFg``
``  PV Status             allocatable``
``  Total PE / Free PE    2559 / 0``

```bash
sudo df -h /rhgs/brick_xvdb
```
``Filesystem                     Size  Used Avail Use% Mounted on``
``/dev/mapper/rhgs_vg1-rhgs_lv1   10G   33M   10G   1% /rhgs/brick_xvdb``


Now create the 2-brick *distvol* Gluster volume.


```bash
sudo gluster volume create distvol \
rhgs1:/rhgs/brick_xvdb/distvol \
rhgs2:/rhgs/brick_xvdb/distvol \
rhgs3:/rhgs/brick_xvdb/distvol \
rhgs4:/rhgs/brick_xvdb/distvol \
rhgs5:/rhgs/brick_xvdb/distvol \
rhgs6:/rhgs/brick_xvdb/distvol
```

Note the output, volume created successfuly and we have to start the volume:

``volume create: distvol: success: please start the volume to access data``

Start the volume

```bash
sudo gluster volume start distvol
```

``volume start: distvol: success``

## Automated Creation of a Replicate Volume

Gluster volume configurations can become much more complicated than the basic example above as the scale grows and additional features are leveraged. In order to simplify and automate the deployment process, Red Hat has introduced an **Ansible**-based deployment tool called **gdeploy**. With gdeploy, an end-to-end Gluster architecture can be defined in a configuration file, and the entire deployment can be executed with a single command.

We will now build a *replicated* volume using the gdeploy method. A replicated volume architecture groups Gluster bricks into replica peers, storing multiple copies of the files as they are being written by the Gluster clients.

For this replicated volume deployment, the backing filesystems have *not* been pre-configured as in the above distributed volume example. The provided gdeploy configuration file includes all of the information needed to setup the `/dev/xvdc` block devices with *LVM* thin provisioning and format and mount an *XFS* filesystem. It then further defines the Gluster volume architecture for the rep01 volume.

View the gdeploy configuration file with the below command.

```bash
cat ~/repvol.conf
```

``[hosts]``
``rhgs1``
``rhgs2``

``# Common backend setup for 2 of the hosts.``
``[backend-setup]``
``devices=xvdc``
``vgs=rhgs_vg2``
``pools=rhgs_thinpool2``
``lvs=rhgs_lv2``
``mountpoints=/rhgs/brick_xvdc``
``brick_dirs=/rhgs/brick_xvdc/repvol``

``[volume]``
``action=create``
``volname=repvol``
``replica=yes``
``replica_count=2``
``force=yes``


In order to use `gdeploy`, the node from which it is run requires passwordless ssh access to the root account on all nodes in the Gluster trusted pool (including itself, if the gdeploy node is also a Gluster pool node, as it is in this example).

**NOTE:** *Amazon AWS by default uses only keypairs for SSH authentication and configures no password-based access to the instances. Because of this, we have pre-populated keys to allow the gluster user on each node to login as the root user on all nodes using the `~/.ssh/id_rsa` private key.* **The commands below are for reference only and do not need to be run for this lab.**

``ssh-keygen -f ~/.ssh/id_rsa -t rsa -N ''``
``for i in {1..6}; do ssh-copy-id -i ~/.ssh/id_rsa root@rhgs$i; done``

*With passwordless ssh configured, we can deploy the **repvol** volume using the `gdeploy` command (NOTE because we rely on the ssh keys, we do not need `sudo` for this command).*

```bash
gdeploy -vv -c ~/repvol.conf
```

## Viewing Volume Details

Information about Gluster volumes, including their architecture, configuration specifics, processes, and ports can be viewed with the below two commands.

The `gluster volume info` command shows configuration and operational details about your Gluster volumes. You can also pass a specific volume name at the end of the command to show output for only one volume.

```bash
sudo gluster volume info
```

``Volume Name: distvol``
``Type: Distribute``
``Volume ID: 95b1dc7f-277d-47b4-95db-e1981a8f18b9``
``Status: Started``
``Number of Bricks: 6``
``Transport-type: tcp``
``Bricks:``
``Brick1: rhgs1:/rhgs/brick_xvdb/distvol``
``Brick2: rhgs2:/rhgs/brick_xvdb/distvol``
``Brick3: rhgs3:/rhgs/brick_xvdb/distvol``
``Brick4: rhgs4:/rhgs/brick_xvdb/distvol``
``Brick5: rhgs5:/rhgs/brick_xvdb/distvol``
``Brick6: rhgs6:/rhgs/brick_xvdb/distvol``
``Options Reconfigured:``
``performance.readdir-ahead: on``
 
``Volume Name: repvol``
``Type: Replicate``
``Volume ID: c16eaa3d-7ccc-42ad-bd71-d43bb16229b2``
``Status: Started``
``Number of Bricks: 1 x 2 = 2``
``Transport-type: tcp``
``Bricks:``
``Brick1: rhgs1:/rhgs/brick_xvdc/repvol``
``Brick2: rhgs2:/rhgs/brick_xvdc/repvol``
``Options Reconfigured:``
``performance.readdir-ahead: on``


The `gluster volume status` command provides other details about the operational state of volume, including process IDs and TCP ports. Here we view the output for volume **repvol**.

```bash
sudo gluster volume status repvol
```

``Status of volume: repvol``
``Gluster process                             TCP Port  RDMA Port  Online  Pid``
``------------------------------------------------------------------------------``
``Brick rhgs1:/rhgs/brick_xvdc/repvol         49153     0          Y       13363``
``Brick rhgs2:/rhgs/brick_xvdc/repvol         49153     0          Y       12465``
``NFS Server on localhost                     2049      0          Y       13385``
``Self-heal Daemon on localhost               N/A       N/A        Y       13390``
``NFS Server on rhgs5                         2049      0          Y       11945``
``Self-heal Daemon on rhgs5                   N/A       N/A        Y       11950``
``NFS Server on rhgs3                         2049      0          Y       11945``
``Self-heal Daemon on rhgs3                   N/A       N/A        Y       11950``
``NFS Server on rhgs2                         2049      0          Y       12487``
``Self-heal Daemon on rhgs2                   N/A       N/A        Y       12492``
``NFS Server on rhgs4                         2049      0          Y       11974``
``Self-heal Daemon on rhgs4                   N/A       N/A        Y       11979``
``NFS Server on rhgs6                         2049      0          Y       11833``
``Self-heal Daemon on rhgs6                   N/A       N/A        Y       11838``
``Task Status of Volume repvol``
``------------------------------------------------------------------------------``
``There are no active volume tasks``



## NFS Client Access
A Gluster volume can be accessed through multiple standard client protocols, as well as through specialized methods including the OpenStack Swift protocol and a direct API.

For many common use cases, the well-established NFS protocol is used for ease of implementation and compatibility with existing applications and architectures. For some particular use cases, the NFS protocol may also offer a performance benefit over other access methods.

You can connect to the **client1** system via `ssh` directly from the rhgs1 system.

```bash
ssh gluster@client1
```

Here we mount the Gluster distvol volume we created above on a RHEL client.

```bash
sudo mkdir -p /rhgs/client/nfs/distvol
```

```bash
sudo mount -t nfs rhgs1:/distvol /rhgs/client/nfs/distvol
```

Check the mount and observe the output.

```bash
df -h /rhgs/client/nfs/distvol
```

``Filesystem      Size  Used Avail Use% Mounted on``
``rhgs1:/distvol      20G   66M   20G   1% /rhgs/client/nfs/distvol``

```bash
mount | grep distvol
```

``rhgs1:/distvol on /rhgs/client/nfs/distvol type nfs (rw,relatime,vers=3,rsize=1048576,wsize=1048576,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,mountaddr=10.100.1.11,mountvers=3,mountport=38465,mountproto=tcp,local_lock=none,addr=10.100.1.11)``


Create and set permissions on a directory to hold our data.


```bash
sudo mkdir /rhgs/client/nfs/distvol/mydir
sudo chmod 777 /rhgs/client/nfs/distvol/mydir
```


Add 100 files to the directory.


```bash
for i in {001..100}; do echo hello$i > /rhgs/client/nfs/distvol/mydir/file$i; done
```

List the directory, counting its contents.

```bash
ls /rhgs/client/nfs/distvol/mydir/ | wc -l
```

``100``
  

## Native Client Access

The Gluster native client utilizes *Filesystem in Userspace (FUSE)* technology to implement a client access protocol that is POSIX-compatible and has the distinct advantage that the client is fully aware of the Gluster volume architecture. This means that with a redundant volume architecture (replicate or disperse), the client automatically has *highly-available* access to the Gluster volume data without any special configuration.

Additionally, the native client can benefit from multiple data paths and specific protocol tunings, allowing for greater overall throughput and lower latency under most workloads.

Here on the same client we mount the Gluster **distvol** volume a second time, this time using the Gluster native client.

```bash
sudo mkdir -p /rhgs/client/native/distvol
sudo mount -t glusterfs rhgs1:distvol /rhgs/client/native/distvol/
```

Examine the new mount.

```bash
df -h /rhgs/client/native/distvol/
```

``Filesystem      Size  Used Avail Use% Mounted on``
``rhgs1:distvol       20G   67M   20G   1% /rhgs/client/native/distvol``


We can see here that the Gluster volume is **mounted twice by the two different protocols**.

```bash
mount | grep distvol
```

``rhgs1:/distvol on /rhgs/client/nfs/distvol type nfs (rw,relatime,vers=3,rsize=1048576,wsize=1048576,namlen=255,hard,proto=tcp,timeo=600,retrans=2,sec=sys,mountaddr=10.100.1.11,mountvers=3,mountport=38465,mountproto=tcp,local_lock=none,addr=10.100.1.11)``
``rhgs1:distvol on /rhgs/client/native/distvol type fuse.glusterfs (rw,relatime,user_id=0,group_id=0,default_permissions,allow_other,max_read=131072)``


Listing the files from our new native mount point, we can see that the files we created through the original NFS mount point are visible.

```bash
ls /rhgs/client/native/distvol/mydir/ | wc -l
```

``100``


Now we'll create an additional set of 100 files, this time through the new native client mount point.

```bash
for i in {101..200}; do echo hello$i > /rhgs/client/native/distvol/mydir/file$i; done
```

We can now see that the 200 total files are available through **both** the native and NFS mount points.

```bash
ls /rhgs/client/native/distvol/mydir/ | wc -l
```

``200``

```bash
ls /rhgs/client/nfs/distvol/mydir/ | wc -l
```

``200``


## Windows Client Access

In order to make our Gluster volume available to Windows clients, we need to make a few configuration changes. Re-connect to the **rhgs1** node from your local ssh client.

```bash
ssh -i <PEM_FILE> ec2-user@<PUBLIC_IP>
```

From node **rhgs1**, run the below commands to modify the **rep01** volume.

```bash
sudo gluster volume set rep01 stat-prefetch off
```

``volume set: success``

```bash
sudo gluster volume set rep01 server.allow-insecure on
```

``volume set: success``

```bash
sudo sed -i '/^end-volume/i option rpc-auth-allow-insecure on' /etc/glusterfs/glusterd.vol
sudo systemctl restart glusterd.service
```

```bash
sudo gluster volume set rep01 storage.batch-fsync-delay-usec 0
```

``volume set: success``

```bash
sudo adduser samba-user
echo -ne "redhat\nredhat\n" | sudo smbpasswd -s -a samba-user
```

``Added user samba-user.``


Here we temporarily mount the client interface on the rhgs1 server in order to create a directory for our Windows client to write to.

```bash
sudo mount -t glusterfs rhgs1:rep01 /mnt
sudo mkdir /mnt/mysmbdir
sudo chmod 777 /mnt/mysmbdir
sudo umount /mnt
```


Using your local RDP client, connect to the public IP address of the Windows client system. The username is **Administrator** and the password is **RedHat1**. You can leave the domain field blank.


Using Windows PowerShell, we mount the Gluster volume to the Z: drive.

```
net use Z: \\10.100.1.11\gluster-rep01 redhat /USER:samba-user
```

We create 100 new files in the mysmbdir subdirectory.

```
for($i=1; $i -le 100; $i++){echo hello$i > Z:\mysmbdir\winfile$i}
```

And we can confirm that there are 100 new files in place.

```
dir Z:\mysmbdir | measure-object -line
```


``      Lines Words          Characters          Property``
``      ----- -----          ----------          --------``
``        100``


**NOTE:** *Due to locking incompatibilities, it is **NOT** supported to to use the SMB/CIFS protocol (Windows client) at the same time as another client access method on the same Gluster volume. Doing so will result in data corruption. For the sake of this lab, we do this only to illustrate Gluster’s functionality.*

Connect to *client1* again with your local ssh client.

```bash
ssh -i <PEM_FILE> ec2-user@<PUBLIC_IP>
```


From here, you can mount the **rep01** volume and see the 100 new files added from the Windows client are visible.

```bash
sudo mkdir -p /rhgs/client/native/rep01
sudo mount -t glusterfs rhgs1:rep01 /rhgs/client/native/rep01
```

```bash
ls /rhgs/client/native/rep01/mysmbdir | wc -l
```

``100``


## Analysis of Volume Types

Connect to **rhgs1** again with your local ssh client.

```bash
ssh -i <PEM_FILE> ec2-user@<PUBLIC_IP>
```

Looking at the brick backend for the **distvol** volume on Gluster node **rhgs1**, we can see that only a subset of the 200 files that were created are present on this brick. (Note the numbers you see may be different than the examples below.)

```bash
ls /rhgs/brick_vdb/distvol/mydir/ | wc -l
```

``95``

Now connect to **rhgs2** -- As a convenience, you can do this most simply directly from node rhgs1 (you will need to connect at the root user).

```bash
ssh root@rhgs2
```

Looking at the brick backend for the **distvol** volume on Gluster node **rhgs2**, we can see that the *remainder of the 200 files are located on this brick*.

```bash
ls /rhgs/brick_vdb/distvol/mydir/ | wc -l
```

``105``


This is the natural effect of the *Distributed Hash Algorithm*. Given the bricks in a distribute volume, the files will be pseudo-randomly placed among those bricks in a statistically even pattern.

While still on node **rhgs2**, take a look at the files in the brick backend for the **rep01** volume.

```bash
ls /rhgs/brick_vdc/rep01/mysmbdir/ | wc -l
```

``100``


Above we created 100 files in this directory from the Windows client, and this time we see *exactly 100 files on the brick backend*. This is the effect of the *Automatic File Replication*, which synchronously places copies of the files written by the clients on the replica peer bricks.

Exit from node **rhgs2** (simply type exit at the command line), returning to node **rhgs1**. Look at the **rep01** brick backend on this node and confirm that the file count matches that of node rhgs2.

```bash
ls /rhgs/brick_vdc/rep01/mysmbdir/ | wc -l
```

``100``


# End Your Lab

This concludes Gluster Test Drive Module 2 - Volume Setup and Client Access. Please follow these steps to end your lab and evaluate the experience.

1. [1] Close your remote sessions.
1. In the *qwik*LABS page, click **End Lab**.
1. In the confirmation message, click **OK**.
1. (Optional) Tell us about your lab experience!

# Additional Resources

- https://www.redhat.com/en/technologies/storage/gluster
