# Project-MAAS-Controller-VM-Build.md

Listed below are notes and steps I took to build a lab environment on my laptop to emulate
an enterprise data center solution for hosting Canonical MAAS/Juju environments. I hosted
these trial versions of the software on my existing VMware Workstation Pro virtualization
platform.

My intent is to share this information with anyone who may also have a need to explore these
two products and potentially save them some time, effort and frustration by allowing them
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
  • <a href="https://maas.io/">Canonical MAAS</a>
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
Note: Many of the steps below are taken from the <a href="https://youtu.be/6IRiUbUxbWk">Ubuntu MAAS Openstack install on VMware workstation video (30:37)</a>.
    
• Step 1: Download ISO installation files for the following software:
          ○ <a href="https://ubuntu.com/download/server">Ubuntu Server 23.04 (or later)</a>
</pre>
<pre>
• Step 2: Install and configure the Ubuntu Server product; this will be called the <em>MAAS-Controller</em>. 
          You'll need to create new VM using the <strong>ubuntu-23.04-live-server-amd64.iso</strong> install file.
          Here are VM settings I used:
            Memory: 4 GB
            Processors: 2
            Hard Disk(SCSI): 40 GB
            CD/DVD (IDE): points to the install ISO file
            Network Adapter: Custom(VMnet0) <--- this is a bridged network.
            Network Adapter2: Custom(VMnet1) <--- this is a Host Only network.                
                Make sure you click on the <em>Advanced</em> button and generate a MAC address for the NIC.
            USB Controller: Present
            Display: Auto Detect
                     Note that I use the <em>Display Scaling > Stretch mode > Free</em> stretch option on these
                     VM consoles; otherwise, the console font is too small to easily read when booting up. 
</pre>
<pre>
• Step 3: Post-Install run the below commands from the <a href="https://www.makeuseof.com/configure-network-ubuntu-server/">How to Configure Networking on Ubuntu Servers</a> site
          to establish the static IP address for the bridged NIC.

            ♦ ls /etc/netplan
                # Output
                00-installer-config.yaml
         
            ♦ sudo cp /etc/netplan/00-installer-config.yaml 00-installer-config.yaml.org

            ♦ sudo vim /etc/netplan/00-installer-config.yaml
    
                network:
                  ethernets:
                    ens33:
                      dhcp4: no
                      addresses: [192.168.1.111/24]
                      gateway4: 192.168.1.1
                     #ens34:
                       #dhcp4: no
                       #addresses: [10.1.1.1/24]
                  version: 2

            ♦ sudo netplan apply

            ♦ You can now check the changes using the following command:
                ip addr

            ♦ sudo vim /etc/systemd/resolved.conf
                [Resolve]
                ...
                DNS=8.8.8.8
                FallbackDNS=8.8.4.4
                ...

            ♦ Restart the server to ensure all of the above settings are retained upon reboot.
</pre>
<pre>
• Step 4: Install the <strong>net-tools</strong> utility package:
            ♦ sudo apt install net-tools

• Step 5: Install the <strong>inetutils-traceroute</strong> package:
            ♦ sudo apt install inetutils-traceroute
</pre>
<pre>
• Step 6: Now that we have the bridged network up and running, we need to add an internal <em>Host Only</em> network
          to the MAAS-Controller VM.

          ♦ Using the VMWare Virtual Network Editor, create a Host Only network ensuring the following:
              ○ Name: VMnet1
              ○ Is Checked: "Connect a host virtual adapter to this network
              ○ Is NOT Checked: "Use local DHCP service to distribute IP addresses to VMs
              ○ Subnet IP: 10.1.1.0 
              ○ Subnet Mask: 255.255.255.0
</pre>
<pre>
• Step 7: Add another network adapter (NIC) to the MAAS-Controller VM
          Shutdown the VM, edit the settings and add another NIC that utilizes the newly created <em>Host Only (VMnet1)</em> network above.
          Make sure you click on the <em>Advanced</em> button and generate a MAC address for the NIC.

• Step 8: Power on the MAAS-Controller VM.

• Step 9: Edit the /etc/netplan/00-installer-config.yaml file and unremark the ens34 NIC settings.
            ♦ sudo vim /etc/netplan/00-installer-config.yaml
                network:
                  ethernets:
                    ens33:
                      dhcp4: no
                      addresses: [192.168.1.111/24]
                      gateway4: 192.168.1.1
                    ens34:
                      dhcp4: no
                      addresses: [10.1.1.1/24]
                  version: 2

• Step 10: Apply the netplan changes.
            ♦ sudo netplan apply

            ♦ Run <em>ifconfig -a</em> to verify that ens34 now has the new IP assigned to it.

            ♦ (optional) You may want to reboot the VM to ensure the ens34 NIC retains its newly assigned IP address.
</pre>
<pre>
• Step 11: (Optional) Install the Ubuntu Desktop product
          Post-creation of the Ubuntu MAAS-Controller above, I chose to install the Ubuntu Desktop.
          This makes it easier to simulatneously install and configure software and services, as well as reference
          online manuals and searches without having to constantly switch between VMs and my laptop.
          Note that the install takes about 2GB of storage space and a good bit of time (~30-45 minutes). 
</pre>
