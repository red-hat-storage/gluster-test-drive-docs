# **THIS LAB IS A WORK IN PROGRESS**
# Lab Guide <br/> Gluster Test Drive Module 3 <br/> Volume Operations and Administration

## Lab Agenda

Welcome to the Gluster Test Drive Module 3 - Volume Operations and Administration. In this lab you will:

- Initiate and observe volume self-heal behavior
- Expand a distribute-replicate volume and observe rebalance
- Understand the use of the `volume profile` command
- Understand the use of the `volume top` command
- Set and observe volume and directory quotas

## Lab Setup
### Connect to the Lab
Connect to the **rhgs1** server instance using its public IP address from the **Addl. Info** tab to the right (Linux/Mac example below).

```bash
ssh gluster@<rhgs1PublicIP>
```

### If Needed, Create the repvol Volume
If you have not already done so as part of **Module 2**, deploy the **repvol** volume using the provided gdeploy configuraiton file.

```bash
gdeploy -c ~/repvol.conf
```
Confirm the volume configuration.

```bash
sudo gluster volume info repvol
```

``Volume Name: repvol``
``Type: Replicate``
``Volume ID: 6fb61bd8-4642-44e9-a5ec-4f15c8740b6f``
``Status: Started``
``Number of Bricks: 1 x 2 = 2``
``Transport-type: tcp``
``Bricks:``
``Brick1: rhgs1:/rhgs/brick_xvdc/repvol``
``Brick2: rhgs2:/rhgs/brick_xvdc/repvol``
``Options Reconfigured:``
``performance.readdir-ahead: on``


## Volume Self-Healing

### About Self-Healing
A Gluster replicated volume maintains multiple copies of files synchronously on the volume bricks. This can provide high availability, load balancing, and increased read throughput. When a member of a replica set becomes unavailable for any reason, Gluster tracks the changes made to the online bricks in order to facilitate a set of self-healing processes when the offline bricks return to service.

There are two types of self-heal that operate concurrently: *client-side* and *server-side* (also known as *proactive*). Client-side heals are triggered when a client performs a file operation on a file or directory marked as needing healed. Server-side heals are managed by background processes that run on each Gluster node, periodically scouring the bricks and healing any files that they find as marked.

### Offlining a Brick
Your **repvol** volume has two bricks and a replication value of 2, and therefore a single replica set between bricks on nodes **rhgs1** and **rhgs2**. On node **rhgs1** you will stop all Gluster services and processes to ensure its bricks are offline.

```bash
sudo systemctl stop glusterd.service
sudo pkill glusterfs
sudo pkill glusterfsd
```

Confirm there are no gluster processes running. The below command should return nothing.
```bash
sudo ps -ef |grep glusterfs | grep -v grep
```

### Writing Files to the Degraded Volume

From **rhgs1** connect via SSH to **client1**.

```bash
ssh gluster@client1
```

If you did not already mount the **repvol** volume as part of Module 2, do it now.

> *NOTE* Below we mount the volume on **client1** using node **rhgs2** as the server because the Gluster services on node **rhgs1** were offlined above.

```bash
sudo mkdir -p /rhgs/client/native/repvol
sudo mount -t glusterfs rhgs2:repvol /rhgs/client/native/repvol
```

Confirm the volume is mounted.
```bash
df -h | grep repvol
```

``rhgs2:repvol     10G   34M   10G   1% /rhgs/client/native/repvol``

Create a directory to hold your files and set permissions appropriately.
```bash
sudo mkdir /rhgs/client/native/repvol/mydir
sudo chmod 777 /rhgs/client/native/repvol/mydir
```

Write 10 new files to the directory you created in the mounted volume.

> **NOTE** There is a default timeout of 42 seconds before which the client will continue trying to contact the offline brick. If that timeout has not expired before you issue a file operation on the mounted volume, you may experience a delay at the client.

```bash
for i in {001..010}; do echo hello$i > /rhgs/client/native/repvol/mydir/healme$i; done
```

Confirm the new files were written.
```bash
ls /rhgs/client/native/repvol/mydir/ | wc -l
```

``10``


### Viewing the Volume State 

Return to lab node **rhgs1**.

```bash
exit
```

On node **rhgs1**, note that the new directory you just created is not visible on the brick backend because this brick was offline at the time of the write.

```bash
ls /rhgs/brick_xvdc/repvol/mydir
```

``ls: cannot access /rhgs/brick_xvdc/repvol/mydir: No such file or directory``

Connect to node **rhgs2** via SSH.

```bash
ssh gluster@rhgs2
```

On node **rhgs2**, note that the `volume status` output only shows the processes for itself and nothing for node **rhgs1**.

```bash
sudo gluster volume status repvol
```

``Status of volume: repvol``
``Gluster process                             TCP Port  RDMA Port  Online  Pid``
``------------------------------------------------------------------------------``
``Brick rhgs2:/rhgs/brick_xvdc/repvol         49152     0          Y       11468``
``NFS Server on localhost                     2049      0          Y       11490``
``Self-heal Daemon on localhost               N/A       N/A        Y       11495``
`` `` 
``Task Status of Volume repvol``
``------------------------------------------------------------------------------``
``There are no active volume tasks``

Confirm that the files you created at the client are visible on the brick backend.

```bash
ls /rhgs/brick_xvdc/repvol/mydir | wc -l
```

``10``

You can view the volume's knowledge of the pending file heals with the below command.

```bash
sudo gluster volume heal repvol info
```

``Brick rhgs1:/rhgs/brick_xvdc/repvol``
``Status: Transport endpoint is not connected``
`` ``
``Brick rhgs2:/rhgs/brick_xvdc/repvol``
``/ ``
``/mydir ``
``/mydir/healme001 ``
``/mydir/healme002 ``
``/mydir/healme003 ``
``/mydir/healme004 ``
``/mydir/healme005 ``
``/mydir/healme006 ``
``/mydir/healme007 ``
``/mydir/healme008 ``
``/mydir/healme009 ``
``/mydir/healme010 ``
``Number of entries: 12``

Return to lab node **rhgs1**.

```bash
exit
```


### Triggering the Self-Heal

Re-start the gluster services. Note that starting the `glusterd` management daemon will automatically start the `glusterfsd` brick and `glusterfs` supporting processes.

```bash
sudo systemctl start glusterd.service
```

Connect again to **client1** via SSH.

```bash
ssh gluster@client1
```

Stat a file that you created above. This will trigger client-side self-heal.

```bash
stat /rhgs/client/native/repvol/mydir/healme001
```

Return to lab node **rhgs1**.

```bash
exit
```

Looking again at the brick backend for the **repvol** volume on node **rhgs1** we can now see the healed files are present.

```bash
ls /rhgs/brick_xvdc/repvol/mydir/ | wc -l
```

``10``


## Volume Expansion and Rebalance

### About Rebalance
When a Gluster volume is scaled horizontally by adding additional bricks, the overall architecture of the volume changes fundamentally and affects data placement by the *Distributed Hash Algorithm*. New file writes can easily account for the new bricks by including them in the random file placement calculation. However, any existing files will remain on their original bricks until a **rebalance** is initiated by the administrator.

A rebalance triggers a re-calculation of data placement for existing files in the volume. A background operation then takes responsibility for moving the files to their new bricks, as needed. This, of course, is one of the heavier operations that Gluster performs, consuming additional system resources across the trusted pool until the rebalance is complete.

### About Distributed-Replicated Volumes
In the lab modules so far you have worked separately with *Distributed* and *Replicated* volumes. Here you will expand your **repvol** volume by adding additional bricks. The replica count remains 2, meaning that every file is written synchronously to two bricks. By adding additional bricks (which you must do in sets of 2 to maintain the replica count), you are creating a distribution set out of multiple replica sets. Upon a file write, first a hashing calculation will be made to determine under which branch of the distribute set to place the file, then the write is made synchronously to the replica peers in that branch. This will be further illustrated in the commands below.

### Add Files to the repvol Volume

From **rhgs1** connect via SSH to **client1**.

```bash
ssh gluster@client1
```

Add 200 new files to the **repvol** volume.

```bash
for i in {001..200}; do echo hello$i > /rhgs/client/native/repvol/mydir/rebalanceme$i; done
```

Confim the file count from the client. Note that we added 10 files in the self-heal section above, so the total file count should be 210.

```bash
ls /rhgs/client/native/repvol/mydir | wc -l
```

``210``

Exit **client1**, returning to node **rhgs1**.

```bash
exit
```

On node **rhgs1**, list the backend brick contents and notice that the file count matches that of the client's view of the volume. Currently, there is only one branch to the distribute set, so all files will be located on this brick and on its replica peer, **rhgs2**.

```bash
ls /rhgs/brick_xvdc/repvol/mydir | wc -l
```

``210``


### Expand the repvol Volume

First, examine the existing configuration for the **repvol** volume. Notice in particular the **Type** and **Number of Bricks** fields.

```bash
sudo gluster volume info repvol
```
 
``Volume Name: repvol``
``Type: Replicate``
``Volume ID: 2ec69e5b-0d04-4a3e-94c3-337b4302fbe8``
``Status: Started``
``Number of Bricks: 1 x 2 = 2``
``Transport-type: tcp``
``Bricks:``
``Brick1: rhgs1:/rhgs/brick_xvdc/repvol``
``Brick2: rhgs2:/rhgs/brick_xvdc/repvol``
``Options Reconfigured:``
``performance.readdir-ahead: on``


You will use the `gdeploy` command to create the new brick backends for nodes rhgs3 through rhgs6, and to add these new bricks to the layout of the **repvol** volume. Take a look at the contents of the configuration file.

```bash
cat ~/repvol-expand.conf
```

``[hosts]``
``rhgs3``
``rhgs4``
``rhgs5``
``rhgs6``
`` ``
``[backend-setup]``
``devices=xvdc``
``vgs=rhgs_vg2``
``pools=rhgs_thinpool2``
``lvs=rhgs_lv2``
``mountpoints=/rhgs/brick_xvdc``
``brick_dirs=/rhgs/brick_xvdc/repvol``
`` ``
``[volume]``
``action=add-brick``
``volname=rhgs1:repvol``
``bricks=rhgs3:/rhgs/brick_xvdc/repvol,rhgs4:/rhgs/brick_xvdc/repvol,rhgs5:/rhgs/brick_xvdc/repvol,rhgs6:/rhgs/brick_xvdc/repvol``

Use `gdeploy` to make the volume change.

```bash
gdeploy -c ~/repvol-expand.conf 
```

Take a look at the updated volume configuration. Note the changes to **Type** and **Number of Bricks**, as well as the additional bricks listed.

```bash
sudo gluster volume info repvol
```
 
``Volume Name: repvol``
``Type: Distributed-Replicate``
``Volume ID: 2ec69e5b-0d04-4a3e-94c3-337b4302fbe8``
``Status: Started``
``Number of Bricks: 3 x 2 = 6``
``Transport-type: tcp``
``Bricks:``
``Brick1: rhgs1:/rhgs/brick_xvdc/repvol``
``Brick2: rhgs2:/rhgs/brick_xvdc/repvol``
``Brick3: rhgs3:/rhgs/brick_xvdc/repvol``
``Brick4: rhgs4:/rhgs/brick_xvdc/repvol``
``Brick5: rhgs5:/rhgs/brick_xvdc/repvol``
``Brick6: rhgs6:/rhgs/brick_xvdc/repvol``
``Options Reconfigured:``
``performance.readdir-ahead: on``

Start the rebalance operation on the **repvol** volume.

```bash
sudo gluster volume rebalance repvol start
```

``volume rebalance: repvol: success: Rebalance on repvol has been started successfully. Use rebalance status command to check status of the rebalance process.``
``ID: e13d3c36-9531-4411-98aa-c974de5e2219``

Take a look at the status of the rebalance. If you run this command quickly after the `rebalance start` above you may catch the *localhost* in the *in progress* status, but due to the limited scale of the lab the rebalance may complete before you run this command and instead show the *completed* status for this node.

```bash
sudo gluster volume rebalance repvol status
```

``                                    Node Rebalanced-files          size       scanned      failures       skipped               status   run time in secs``
``                               ---------      -----------   -----------   -----------   -----------   -----------         ------------     --------------``
``                               localhost               79         1.1KB           210             0             0          in progress               1.00``
``                                   rhgs2                0        0Bytes             0             0             0            completed               0.00``
``                                   rhgs3                0        0Bytes             0             0             0            completed               0.00``
``                                   rhgs4                0        0Bytes             0             0             0            completed               0.00``
``                                   rhgs5                0        0Bytes             0             0             0            completed               0.00``
``                                   rhgs6                0        0Bytes             0             0             0            completed               0.00``
``volume rebalance: repvol: success``

Now take a look again at the count of files in the brick backend for **repvol** on node **rhgs1**. You will find that the number of files has reduced considerably.

```bash
ls /rhgs/brick_xvdc/repvol/mydir | wc -l
```

``75``

To further illustrate the effect of the rebalance, connect to node **rhgs5** via SSH and look at the file count in the brick backend there.

```bash
ssh gluster@rhgs5
```

```bash
ls /rhgs/brick_xvdc/repvol/mydir | wc -l
```

``66``

Return to node **rhgs1**.

```bash
exit
```

### The gstatus Command

A good summary view of your volume is available through the `gstatus` command. Passing the `-l` flag to this command will also provide a visual of the volume layout in ASCII format, which can be very handy for understanding the distribute branching and replica pairing.

```bash
sudo gstatus -v repvol -l -w
```

<p><code> 
     Product: RHGS Server v3.1Update3  Capacity:  60.00 GiB(raw bricks)
      Status: HEALTHY                      201.00 MiB(raw used)
   Glusterfs: 3.7.5                         30.00 GiB(usable from volumes)
  OverCommit: No                Snapshots:   0

Volume Information
	repvol           UP - 6/6 bricks up - Distributed-Replicate
	                 Capacity: (0% used) 100.00 MiB/30.00 GiB (used/total)
	                 Snapshots: 0
	                 Self Heal:  6/ 6
	                 Tasks Active: None
	                 Protocols: glusterfs:on  NFS:on  SMB:on
	                 Gluster Connectivty: 7 hosts, 78 tcp connections

	repvol---------- +
	                 |
                Distribute (dht)
                         |
                         +-- Replica Set0 (afr)
                         |     |
                         |     +--rhgs1:/rhgs/brick_xvdc/repvol(UP) 33.00 MiB/10.00 GiB 
                         |     |
                         |     +--rhgs2:/rhgs/brick_xvdc/repvol(UP) 33.00 MiB/10.00 GiB 
                         |
                         +-- Replica Set1 (afr)
                         |     |
                         |     +--rhgs3:/rhgs/brick_xvdc/repvol(UP) 33.00 MiB/10.00 GiB 
                         |     |
                         |     +--rhgs4:/rhgs/brick_xvdc/repvol(UP) 33.00 MiB/10.00 GiB 
                         |
                         +-- Replica Set2 (afr)
                               |
                               +--rhgs5:/rhgs/brick_xvdc/repvol(UP) 33.00 MiB/10.00 GiB 
                               |
                               +--rhgs6:/rhgs/brick_xvdc/repvol(UP) 33.00 MiB/10.00 GiB 
</code></p>



## Analyzing Volume Performance

The `volume profile` command provides an interface to get the per-brick or NFS server I/O information for each File Operation (FOP) of a volume. This information helps in identifying the bottlenecks in the storage system.

Start profiling for the **repvol** volume.

sudo gluster volume profile repvol start
Starting volume profile on repvol has been successful



## Administration of Volume Quotas


# End of Module 3

This concludes **Gluster Test Drive Module 3 - Volume Operations and Administration**. You may continue now with Module 4, or return at any time to access the modules in any order you wish.
