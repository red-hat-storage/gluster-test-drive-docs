# **THIS LAB IS A WORK IN PROGRESS**
# Lab Guide <br/> Gluster Test Drive Module 3 <br/> Volume Operations and Administration

## Lab Agenda

Welcome to the Gluster Test Drive Module 3 - Volume Operations and Administration. In this lab you will:

- Initiate and observe volume self-heal behavior
- Expand a distribute-replicate volume and observe rebalance
- Change volume configurations and view changes
- Understand the use of the `volume profile` command
- Understand the use of the `volume top` command
- Set and observe volume and directory quotas

## Lab Setup
### Connect to the Lab
Connect to the **rhgs1** server instance using its public IP address from the **Addl. Info** tab to the right (Linux/Mac example below).

```bash
ssh gluster@<rhgs1PublicIP>
```

### If Needed, Create the Volume
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
 /rhgs/brick_xvdc/repvol/mydir | wc -l
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