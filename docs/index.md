# Introduction

Welcome to the **Red Hat Gluster Storage** Hands-on Test Drive. To make your Gluster experience awesome, the contents of this test drive have been divided into following modules.

- <a href="gluster-module-1/">**Module 1 :** Introduction to Gluster concepts</a>
- <a href="gluster-module-2/">**Module 2 :** Volume Setup and Client Access</a>
- ***Coming Soon*** **Module 3 :** Volume Operations and Administration
- ***Coming Soon*** **Module 4 :** Disperse Volumes (Erasure Coding)
- ***Coming Soon*** **Module 5 :** Tiered Volumes (Cache Tiering)
- ***Coming Soon*** **Module 6 :** Gluster Internals

## What is Gluster?

Gluster provides open, software-defined file storage that scales out as much as you need. You can easily and securely manage large, unstructured, and semi-structured data at a fraction of the cost of traditional, monolithic storage. And only Red Hat lets you deploy the same storage on premise; in private, public, or hybrid clouds; and in Linux® containers. You can read more about gluster here: <https://www.redhat.com/en/technologies/storage/gluster>

## Test-Drive Prerequisites

### SSH and RDP
If you are a Windows user, you will need a Secure Shell client like *PuTTY* to connect to your instance. If you do not have it already, you can download the PuTTY client here: <http://the.earth.li/~sgtatham/putty/latest/x86/putty.exe>

Mac and Linux users, you will use your preferred terminal application (this should already be installed on your machine). For accessing the Windows lab client system interface via RDP, you will need a RDP client program such as *Vinagre* on Linux or the *Microsoft Remote Desktop* client for Mac. Vinagre is likely available for your Linux distribution using your standard package manager. You can download the Mac RDP client here: <https://www.microsoft.com/en-us/download/details.aspx?id=18140>

## Getting to know your lab environment

### Starting the lab

1. On the **Lab Details** tab to the right, notice the lab properties:

   ![](http://us-west-2-aws-training.s3.amazonaws.com/awsu-spl/spl02-working-ebs/media/image004.png)

   - **Setup Time -** The estimated time for the lab to start your instance so you can access the lab environment.
   - **Duration -** The estimated time the lab should take to complete.
   - **Access -** The time the lab will run before automatically shutting down.

<ol start="2"><li>Click the <img src="http://us-west-2-aws-training.s3.amazonaws.com/awsu-spl/spl02-working-ebs/media/image005.png"> button in the navigation bar above to launch your *qwik*LABS. If you are prompted for a token, use the one distributed to you (or credits you've purchased).</li></ol>

   A status bar will then show the progress of the lab environment creation process. Your lab resources may not be fully available until the process is complete.

   ![](http://us-west-2-aws-training.s3.amazonaws.com/awsu-spl/spl02-working-ebs/media/image006.png)

<ol start="3"><li>The **Connect** panel to the right is automatically opened when the lab starts. Practice closing and re-opening it. While open it may obscure some of these lab instructions temporarily.</li></ol>

> **Tip** If the **Connect** tab is unavailable, make sure you click **Start Lab** at the top of your screen.


## Accessing the lab

All of the information you need to access your test drive lab instances is available via the **Addl. Info** tab to the right. There you will find relevant public IP addresses, usernames, and passwords for your personal lab environment.

All Linux instances may be accessed via your local SSH client.

All Windows instances may be accessed via your local RDP client. **Windows instances will be logged into with the** ***Administrator*** **username and with the password available in the** ***Addl. Info*** **tab.**
