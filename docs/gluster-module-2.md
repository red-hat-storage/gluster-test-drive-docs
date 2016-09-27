# Lab Guide - Gluster Cluster Setup

## Overview

### What is Gluster?

Gluster provides a scalable, reliable, and cost-effective data management platform, streamlining file and object access across physical, virtual, and cloud environments. You can read more about gluster here https://www.redhat.com/en/technologies/storage

### Lab the plan

Welcome to the Gluster Cluster setup, the first module in the Red Hat Gluster Storage Test Drive. In this lab we will

- Setup a Red Hat Gluster storage primary cluster using gdeploy automation
- The gdeploy script will also create a distributed-replicated Gluster volume
- Verify the client access to the gluster volume and observe the layout on the backend bricks
- Write files to the gluster volume and observe the layout of the backend bricks
- On the secondary storage nodes, use command line for backend initialization and volume creation 

# Prerequisites

If you are a Windows user, you will need a Secure Shell client like PuTTY to connect to your instance. If you do not have it already, you can download the PuTTY client here: http://the.earth.li/~sgtatham/putty/latest/x86/putty.exe

Mac and Linux users, you will use the Terminal application to connect to EC2 (this should already be installed on your machine). 

# Start your lab

On the **Lab Details** tab, notice the lab properties:

![](http://us-west-2-aws-training.s3.amazonaws.com/awsu-spl/spl02-working-ebs/media/image004.png)

- **Setup Time -** The estimated time for the lab to start your instance so you can access the lab environment.
- **Duration -** The estimated time the lab should take to complete.
- **Access -** The time the lab will run before automatically shutting down.

1. Click **Start Lab** to launch your *qwik*LABS. If you are prompted for a token, use the one distributed to you (or credits you've purchased).

![](http://us-west-2-aws-training.s3.amazonaws.com/awsu-spl/spl02-working-ebs/media/image005.png)

A status bar shows the progress of the lab environment creation process. Your lab resources may not be fully available until the process is complete.

![](http://us-west-2-aws-training.s3.amazonaws.com/awsu-spl/spl02-working-ebs/media/image006.png)

1. [2] The **Connect** panel is automatically opened when the lab starts. Practice closing and re-opening it. While open it may obscure some of these lab instructions temporarily. 

**Tip** If the **Connect** tab is unavailable, make sure you click **Start Lab** at the top of your screen.

1. [3] On the open **Connect** panel, copy the **Endpoint** of the instance that was created for you. You will need this information later. Close the **Connect** tab to resume reading and follow the rest of these lab instructions.

# Log into your instance

Windows users, proceed to the next step. Mac and Linux users, skip to [the next section](#maclinux).

## Windows Users: Connecting to your Amazon EC2 Instance via SSH

In this section, you will use the PuTTY Secure Shell (SSH) client and your serverÃ¢â‚¬â„¢s public DNS address to connect to your server.

**Note** This section is for Windows users only. If you are running OSX or Linux, skip to the next section.

### Download your key

1. [4] On the *qwik*LABS web page, click the **Connect** tab.
1. Copy the endpoint of your instance to your clipboard if you have not done so already.
1. Under **Connection Details,** open the **Download PEM/PPK** drop-down list, and click **Download PPK**.
1. Save the file to the directory of your choice.

### Connect to the server using SSH and PuTTY

1. [8] Open PuTTY.exe. 

**Note** You may have downloaded it at the beginning of this lab. You can download it here: http://the.earth.li/~sgtatham/putty/latest/x86/putty.exe)

1. [9] For **Host Name**, type **ubuntu@**, then paste the endpoint of your instance from your Clipboard. Your host name should look something like this: *ubuntu@ec2-00-00-000-00.compute-1.amazonaws.com*.
1. In the **Category** list, click **+** to expand **SSH**.
1. Click **Auth** (don't expand it).
1. In the **Private key file for authentication** field, browse to the PPK file that you downloaded and double-click it.
1. Click **Open**.
1. If you are prompted, click **Yes** to allow a first connection to this remote SSH server.

**Note** Because you are using a key pair for authentication, you will not be prompted for a password.

When you see a terminal screen and Linux command line prompt, it means that you are connected to your Ubuntu instance. Congratulations! If you know any Linux commands, feel free to explore. 
<a name="maclinux"></a>
### Troubleshooting

If PuTTY fails to connect to your instance, verify that:

* You typed **ubuntu@**\<Public DNS\> as your Host Name.
* You downloaded the PPK file for this lab from *qwik*LABS (not the PEM file).
* You are using the downloaded PPK file in PuTTY.
* The network you are on allows for outbound TCP connections to destination port 22.

## OSX and Linux Users: Connecting to your instance via SSH

This section is for OSX and Linux users only. If you are running Windows, skip to the next section.

### Download your key

1. [15] On the *qwik*LABS page, open the **Connect** tab.
1. Copy the **Endpoint** of your instance to your clipboard.
1. Under Connection Details, open the **Download PEM/PPK** drop-down list, and click **Download PEM**.
1. Save the file to the directory of your choice.

### Connect to the RHEL instance using the OpenSSH CLI client

1. [19] Open the Terminal application.
1. Enter the following command, substituting **\<path-to-pem\>** for the path to the key you downloaded: 

```
chmod 600 <path-to-pem>
```

1. [21] Enter the following command. Substitute **\<path-to-pem\>** for the PEM file you downloaded, and **\<endpoint\>** from your clipboard: 
```
ssh -i <path-to-pem> ubuntu@<endpoint>
```

**Note** Do not include the "\< \>" brackets in your commands.

When you see a terminal screen and Linux command line prompt, it means that you are connected to your Ubuntu instance. Congratulations! If you know any Linux commands, feel free to explore.

# End Your Lab

Follow these steps to end your lab and evaluate the experience.

1. [55] Close your remote session.
1. In the *qwik*LABS page, click **End Lab**.
1. In the confirmation message, click **OK**.
1. (Optional) Tell us about your lab experience!

# Additional Resources

- https://www.redhat.com/en/technologies/storage/gluster
