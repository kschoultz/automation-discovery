# Project-VSphere-ESXi-Lab-Build.md

Listed below are notes and steps I took to build a lab environment on my laptop to emulate
an enterprise data center solution for hosting VMWare ESXi/vSphere environments. I hosted
these trial versions of the software on my existing VMware Workstation Pro virtualization
platform.

My intent is to share this information with anyone who may also have a need to explore these
VMWare products and potentially save them some time, effort and frustration by allowing them
to cheat off my homework.

<pre>
###--- Platform Used ---###
Hardware:
• Laptop
    Make/Model: Asus ProArt Studiobook
    CPU(s): 1 x 12th Gen Intel(r) Core(TM) i7-12700H
              Physical Cores: 14
              Logical Cores: 20
              P-Cores: 6 x 4.70 GHz(max); hyperthreaded
              E-Cores: 8 x 3.50 GHz(max); single threaded
    RAM: 32GB (2 x 16GB DDR-5)
    HDD: SSD/EMMC 1 - 953GB, Samsung MZVL21T0HCLR-00B00
    OS: Microsoft Windows 11 Pro 64-bit 10.0.22621
</pre>
<pre>
Software:
  • <a href="https://coderbag.com">Quick CPU x64 - version 4.6.0.0</a>
  • <a href="https://www.vmware.com/products/workstation-pro.html">VMware® Workstation 17 Pro (17.0.2 build-21581411) [Licensed Product]</a>
  • <a href="https://www.vmware.com/products/esxi-and-esx.html">VMware ESXi 8.0.1/Build 21813344 [60-day trial license]</a>
  • <a href="https://www.vmware.com/products/vsphere.html">VMware vSphere version 8.0.1.00000 [60-day trial license]</a>
</pre>
<pre>
###--- Installation and Configuration ---###
The following steps assume that you have already installed VMware Workstation 17 Pro and are ready
to build the VMs required to form the lab environment.
Note: Even though I haven't made the attempt, I strongly suspect that the same build process could be
      accomplished using a freeware virtualization product such as Oracle's VirtualBox solution.
      The key would be setting up virtual networks that would perform NATing and DNS IP assignments.
</pre>
<pre>
<ins>Windows Prerequisite Steps</ins>
• Resolving the "VT-X/EPT is not supported on this platform" error message:
  When attempting to create nested VMs inside of VMware Workstation Pro (or any other virtualization platform) on 
  a Windows OS, you may receive an error message like th eone listed above. This <ins>must</ins> be remediated before
  you can continue with the installation and configurations listed below.

  Luckily, I found the video listed below which does a really good job of explaining why you're likely receiving this
  message and how to go about remediating it.
    ♦ <a href="https://youtu.be/6f1Qckg2Zx0">Solved: Virtualized Intel VT-X/EPT is not supported on this platform (video 5:19)</a>

  Note: Fully disabling the Microsoft Hypervisor will also <ins>disable</ins> the <strong>Windows Subsystem for Linux (WSL)</strong> if you happen
  to have it installed.

• Quick CPU x64:
  My intial install attempts of the VMware ESXi product performed abismally. Viewing the Windows Resource
  Manager app, I discovered that several of the vCPUs were "Parked" and not being utilized. This is largely
  because each OS vendor sets specific criteria as to when and how many cores are spun up and deployed depending
  on a number of performance indicators.

  To overcome these built-in limitations, I downloaded and deployed the Quick CPU x64 product for Windows.
  This producte helped me force the utilization of all the vcores on my laptop and maximize the performance by
  implementing the Turbo Boost functionality on the P-cores. This greatly enhanced the overall performance of my
  laptop while running all the required lab VMs at once.

  This software is basically freeware with the option of providing a monetary gift for support.
</pre>
<pre>
<ins>Software Installs and VM Configurations</ins>
• Step 1: Download ISO installation files for the following software:
          ○ <a href="https://www.vmware.com/products/esxi-and-esx.html">VMware ESXi 8.0.1 or later [60-day trial license]</a>
          ○ <a href="https://www.vmware.com/products/vsphere.html">VMware vSphere version 8.0.1.00000 or later [60-day trial license]</a>
</pre>
<pre>
• Step 2: Install and configure the VMware ESXi Hypervisor product 
          You'll need to create new VM using the ESXi Hypervisor ISO install file.
          Here are VM settings I used:
            Memory: 16 GB
            Processors: 8
            Hard Disk(SCSI): 40 GB {where the ESXi binaries get installed}
            Hard Disk 2 (SCSI): 142 GB {for establishing a Data Store to be used by hosted VMs}
            CD/DVD (IDE): points to the install ISO file
            Network Adapter: NAT
            USB Controller: Present
            Display: Auto Detect
                     Note that I use the <em>Display Scaling > Stretch mode > Free</em> stretch option on these
                     VM consoles; otherwise, the console font is too small to easily read when booting up. 

            Here is a good video that walks you through the process of installing and configuring ESXi on a
            VM running on VMware Workstation Pro:
              ♦ <a href="https://youtu.be/HDpPOx7g0Lk">Installation of ESXi 8.0 Step by Step on VMware Workstation Pro (video 15:54)</a>
            And here's a web page that also provides a good step-by-step process for installing ERSXi on Workstation Pro:
              ♦ <a href="https://www.sysprobs.com/vmware-esxi-on-windows-10">Install VMware ESXi 7.x on Windows 10/11 by VMware Workstation</a>
</pre>
<pre>
• Step 3: Configure the ESXi Datastore
          Log into the ESXi console (as root) and perform the following actions:
            a) On the left side panel, click on the <ins>Storage</ins> icon.
            b) On the right panel, you should see the <ins>Datastores</ins> link; click on it.
            c) You should now see the <ins>New Datastores</ins> icon below; click on it to create a new datastore using
               the second virtual 142GB hard disk that was created above during the ESXi VM creation. I named mine
               "ESXi-DataStore01".

               This datastore will be utilized by ESXi to host new VMs created by the vSphere Server down the road.
</pre>
<pre>
• Step 4: Install the VMware vSphere Server Appliance
          Now that the ESXi Vm is installed and working, we'll proceed to install the vSphere Server appliance inside of ESXi
          by following the steps outlined in the below video:
            ♦ <a href="https://youtu.be/GE0uE2XuJmw">Installation of VMware vCenter server appliance VCSA 8 0 - step by step (video 15:37) </a>

          Here are VM settings I used during the install process:
            ESXi Host or vCenter Server name: {I used the ESXi IPv4 Address in which it's being installed}
            HTTPS Port: 443 (default)
            User name: root {the root login for ESXi}
            Password: {the root password for ESXi}
            VM name: VMware vCenter Server (default)
            Set root password: {a new root password for the vSphere appliance; not ESXi}
            Confirm root password: {same as above}
            Deployment Size: Tiny
            Storage Size: default
            Select Datastore: ESXi-DataStore01 {the datastore created in ESXi above}
              ○ Enable Thin Disk Mode: {I selected/checked this option}
            Network: VM Network
            IP version: IPv4
            IP assignment: DHCP
            Time synchronization mode: Synchronize time with the ESXi host
            SSH access: Activated
            Single Sign-On domain name: vsphere.local
            Single Sign-On username: administrator
            Single Sign-On password: {whatever you choose}
            Confirm password: {same as above}
            
            Host Resources used by the vSphere appliance install:
            Memory: 16 GB 
              {I intially selected the 14GB default; however, I received an error message during the first install attempt
               stating that there wasn't enough memory. So, I bumped it up to 16GB and reran the install - which succedded.
               It appears that the bare minimum install footprint actually uses 14.77GB or memory.}
            Processors: 2

          Notes:
          1. Bootup time: When starting/restarting the vSphere Server appliance from within the ESXi VM, you'll need to be
             extremely patient - as it seems to take an exhorbant amount of time for the appliance to completely spin up
             and enable its services to allow login via https.

             I took a look at the vCPU utilization of the vSphere Server during bootup from within ESXi. It seems to stay
             pegged at or near 100 percent utilization during the boot process. I suspect that adding additional vCPUs may
             help with this time delay; but, I haven't tried that solution yet.

          2. Login prompt on the vSphere Server console inside of ESXi:
             When starting/restarting the vSphere appliance inside of ESXi, you'll see a login prompt on the console. You
             can safely ignore this because the Single Sign-On background process will take care of the required logins
             in the background. Attempting to login to the console during the bootup process will likely fail.

          3. ESXi error message for vSphere appliance can be ignored:
             Once the vShpere appliance has been deployed, you'll likely see the below error message from ESXi about the 
             wrong guest host OS for vSphere.
             "The configured guest OS (Other 3.x Linux (64-bit)) for this virtual machine does not match the guest that
              is currently running (VMware Photon OS (64-bit))."

             According to the VMware article listed below, you can safely ignore this.
             ♦ <a href="https://kb.vmware.com/s/article/67671">vCenter Server Appliance displays a warning message on ESXi Host Client (67671) </a>
</pre>
<pre>
• Step 5: Create a Cluster and Data Center from within vSphere Center
          Follow the steps outlined in the below video to create your first data center and cluster:
            ♦ <a href="https://youtu.be/M3dOX_8wQrs">Creating datacenter clusters and hosts using VMware vCenter Server 8.0 (video 11:03) </a>

          I used:
            Data Center name: DC0
            Cluster name: Cluster-01     
</pre>