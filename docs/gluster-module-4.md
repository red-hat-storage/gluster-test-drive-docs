# **THIS LAB IS A WORK IN PROGRESS**
# Lab Guide <br/> Gluster Test Drive Module 4 <br/> Disperse Volumes (Erasure Coding)

## Lab Agenda

Welcome to the Gluster Test Drive Module 4 - Disperse Volumes (Erasure Coding). In this lab you will:

- Understand the basic concepts of erasure coding
- Create a gdeploy configuration for a disperse volume
- Deploy a disperse volume using gdeploy
- Manually fail bricks and observe the high availability of erasure coding


## Getting Started

If you have not already done so, click the <img src="http://us-west-2-aws-training.s3.amazonaws.com/awsu-spl/spl02-working-ebs/media/image005.png"> button in the navigation bar above to launch your lab. If you are prompted for a token, use the one distributed to you (or credits you've purchased).

> **NOTE** It may take **up to 10 minutes** for your lab systems to start up before you can access them.

## Lab Setup
### Connect to the Lab
Connect to the **rhgs1** server instance using its public IP address from the **Addl. Info** tab to the right (Linux/Mac example below).

```bash
ssh gluster@<rhgs1PublicIP>
```

## About Disperse Volumes

Gluster disperse volumes utilize **erasure coding** technology to provide data protection with a lower overall infrastructure investment. Unlike standard replication, in which to protect *n* bricks against *r* failures you must invest in a total of *n+(n\*r)* bricks, with erasure coding you can design for the same level of protection with an investment of only *n+(n/r)* or even less.

For example, assume you need *n=4* bricks to hold your data set and you require protection from failure of *r=2* bricks. If you use standard **replication**, your investment in bricks will be *4+(4\*2)* or **12 total bricks**. If instead you utilize **erasure coding**, your investment in bricks can be *4+(4/2)* or **6 total bricks**.

![](images/RHS_Gluster_Test_Drive-Module_1-dblack-201610-19.png)

### Erasure Coding and Parity

Instead of writing multiple exact copies of data, erasure coding algorithms write a combination of *data* and *parity* across bricks in the volume. A number of encoding algorithms are available offering varying levels of protection in terms of a *redundancy-to-data* ratio. For our lab, you will be using a 4:2 ratio -- 2 levels of redundancy for every 4 data bricks, meaning that each disperse set requires exactly 6 bricks.

> **NOTE** This functionality is very similar to how RAID works at the block level. You can relate our 4:2 erasure coding ratio to RAID level 6.

When a file is written to the disperse volume, it is broken into calculated chunks of data and written across all of the bricks in the disperse set. When a file is read, it can be **algorithmically reassembled from any n of the bricks where n is the number of data bricks in the set**. Here with our 4:2 ratio, therefore, any 4 of the 6 bricks can be used to retrieve the files.

### Use Cases

Disperse volumes offer *space-efficient* and *capacity-optimized* storage architectures. However, there are a couple of primary tradeoffs when considering this method of data protection.

- The overhead of the algorithms can lead to decreased performance for your data set, particularly for reads and for small files.
- The files are no longer stored in their whole original form on the brick backends, making offline or backend retrieval of data impossible.

> **NOTE** Interestingly, it has been shown that, under most conditions with the Gluster native client, write performance is unaffected by erasure coding and is even sometimes modestly improved.

Disperse volumes should be used when capacity is of greater value than performance. Larger file workloads (1GB+) will experience the least performance degredation versus replicated volumes. *You should avoid using the NFS client with disperse volumes.*
