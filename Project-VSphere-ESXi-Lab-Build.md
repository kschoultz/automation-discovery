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
• Step 1: Download ISO installation files for the following software:
          ○ VMware ESXi 8.0.1 or later [60-day trial license]
          ○ VMware vSphere version 8.0.1.00000 or later [60-day trial license]
          ○ Red Hat Enterprise Linux 9.2 (Plow) - 64bit [60-day trial license]
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
              ♦ <a href="https://youtu.be/HDpPOx7g0Lk">Installation of ESXi 8.0 Step by Step on VMware Workstation Pro</a>
</pre>