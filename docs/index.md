# Introduction

Welcome to the Red Hat Gluster Storage Hands-on Lab. To make your Gluster experience awesome, the contents of this test drive have been divided into following modules.

- [**Module 1 :** Introduction to Gluster concepts](gluster-module-1)
- [**Module 2 :** Volume Setup and Client Access](gluster-module-2)
- **Module 3 :** Volume Operations and Administration
- **Module 4 :** Disperse Volumes (Erasure Coding)
- **Module 5 :** Tiered Volumes (Cache Tiering)
- **Module 6 :** Gluster Internals

## What is Gluster?

Gluster provides a scalable, reliable, and cost-effective data management platform, streamlining file and object access across physical, virtual, and cloud environments. You can read more about gluster here https://www.redhat.com/en/technologies/storage

## Prerequisites

If you are a Windows user, you will need a Secure Shell client like PuTTY to connect to your instance. If you do not have it already, you can download the PuTTY client here: http://the.earth.li/~sgtatham/putty/latest/x86/putty.exe

Mac and Linux users, you will use your preferred terminal application to connect to EC2 (this should already be installed on your machine). For accessing the Windows system interface via RDP, you will need a RDP client program such as *Vinagre* on Linux or the *Microsoft Remote Desktop* client for Mac. Vinagre is likely available for your Linux distribution using your standard package manager. You can download the Mac RDP client here: https://www.microsoft.com/en-us/download/details.aspx?id=18140

## Getting to know your lab environment

### Starting the lab

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


## Accessing the lab

Windows users, please proceed to the next step. Mac and Linux users, skip to [the next section](#maclinux).

<a name="windows"></a>
### **Windows Users**: Connecting to your Amazon EC2 Linux Instance via SSH

This section describes how to use PuTTY Secure Shell (SSH) client to connect to your Linux lab systems. *The connection you are making to a lab system under these instructions are for testing purposes only to confirm functionality and is not part of the lab itself.*

**Note** This section is for Windows users only. If you are running OSX or Linux, please skip to the next section.

#### Download your key

1. [3] On the *qwik*LABS web page, click the **Addl. Info** tab.
1. Copy the **IP address** of the instance to which you are connecting to your clipboard if you have not done so already.
1. Under the **Connect** tab, open the **Download PEM/PPK** drop-down list, and click **Download PPK**.
1. Save the file to the directory of your choice.

#### Connect to the server using SSH and PuTTY

1. [7] Open PuTTY.exe. 

**Note** If you have not already downloaded PuTTY, you can do so here: http://the.earth.li/~sgtatham/putty/latest/x86/putty.exe)

1. [8] For **Host Name**, type **ec2-user@\<public_ip\>**, where \<public_ip\> is the IP address of your lab instance from the **Addl. Info** tab. Your host name should look something like this: *ec2-user@54.209.158.243*.
1. In the **Category** list, click **+** to expand **SSH**.
1. Click **Auth** (don't expand it).
1. In the **Private key file for authentication** field, browse to the PPK file that you downloaded and double-click it.
1. Click **Open**.
1. If you are prompted, click **Yes** to allow a first connection to this remote SSH server.

**Note** Because you are using a key pair for authentication, you will not be prompted by the lab system for a password.

When you see a terminal screen and Linux command line prompt, it means that you are connected to your lab instance. Congratulations! If you know any Linux commands, feel free to explore. 

#### Troubleshooting

If PuTTY fails to connect to your instance, verify that:

* You typed **ec2-user@**\<public_ip\> as your Host Name.
* You downloaded the PPK file for this lab from *qwik*LABS (**not** the PEM file).
* You are using the downloaded PPK file in PuTTY.
* The network you are on allows for outbound TCP connections to destination port 22.


<a name="maclinux"></a>
### **OSX and Linux Users**: Connecting to your Amazon EC2 Linux Instance via SSH

**Note** This section is for OSX and Linux users only. If you are running Windows, please see [the previous section](#windows).

#### Download your key

1. [14] On the *qwik*LABS page, open the **Addl. Info** tab.
1. Copy the **IP address** of the instance to which you are connecting to your clipboard if you have not done so already.
1. Under **Connect** tab, open the **Download PEM/PPK** drop-down list, and click **Download PEM**.
1. Save the file to the directory of your choice.

#### Connect to the RHEL instance using the OpenSSH CLI client

1. [18] Open your terminal application.
1. Enter the following command, substituting **\<path-to-pem\>** for the path to the key you downloaded: 

```bash
chmod 600 <path-to-pem>
```

1. [20] Enter the following command. Substitute **\<path-to-pem\>** for the PEM file you downloaded, and **\<public_ip\>** for the IP address of your lab instance from the **Addl. Info** tab.

```bash
ssh -i <path-to-pem> ec2-user@<public_ip>
```

**Note** Do not include the "\< \>" brackets in your commands.

When you see a terminal screen and Linux command line prompt, it means that you are connected to your Linux instance. Congratulations! If you know any Linux commands, feel free to explore.
