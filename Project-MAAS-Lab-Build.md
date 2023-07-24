# Project-MAAS-Lab-Build.md

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
• Step 1: Download ISO installation files for the following software:
          ○ <a href="https://ubuntu.com/download/server">Ubuntu Server 23.04 (or later)</a>
</pre>
<pre>
• Step 2: Install and configure the Ubuntu Server product; this will be called the <em>MAAS-Controller</em>. 
          You'll need to create new VM using the <strong>ubuntu-23.04-live-server-amd64.iso</strong> install file.
          Here are VM settings I used:
            Memory: 16 GB
            Processors: 8
            Hard Disk(SCSI): 160 GB
            CD/DVD (IDE): points to the install ISO file
            Network Adapter: VMnet8(NAT)
            USB Controller: Present
            Display: Auto Detect
                     Note that I use the <em>Display Scaling > Stretch mode > Free</em> stretch option on these
                     VM consoles; otherwise, the console font is too small to easily read when booting up. 
</pre>
<pre>
• Step 3: (Optional) Install the Ubuntu Desktop product
          Post-creation of the Ubuntu MAAS-Controller above, I chose to install the Ubuntu Desktop.
          This makes it easier to simulatneously install and configure software and services, as well as reference
          online manuals and searches without having to constantly switch between VMs and my laptop.
          Note that the install takes about 2GB of storage space and a good bit of time (~30-45 minutes). 
</pre>
<pre>
• Step 4: MAAS Install
          Follow the steps outlined on the <a href="https://maas.io/docs/how-to-get-started-with-maas">How to get started with MAAS</a> documentation.

          <ins>Notes and Observations</ins>
          ○ PostgreSQL: <strong>Do this first!</strong>
                        The <strong>sudo apt install -y postgresql</strong> command will likely install the
                        latest version (in my case 15.3).

                            # Postgres v15.3 Initial Setup
                            # Necessary steps from: https://stackoverflow.com/questions/1471571/how-to-configure-postgresql-for-the-first-time
                            
                            postgres@maas-controller:~$ psql template1
                            psql (15.3 (Ubuntu 15.3-0ubuntu0.23.04.1))
                            Type "help" for help.
                            
                            template1=# ALTER USER postgres with encrypted password '{postgres-passwd}';
                            ALTER ROLE
                                                        
                            keith@maas-controller:~$ sudo vi /etc/postgresql/15/main/pg_hba.conf                            
                                # Database administrative login by Unix domain socket
                                local   all             postgres                                md5
                            
                            keith@maas-controller:~$ sudo /etc/init.d/postgresql restart
                            Restarting postgresql (via systemctl): postgresql.service.
                            
                            sudo createuser -U postgres -d -e -E -l -P -r -s keith
    
          ○ MAAS Install: <strong>Do this second!</strong>
                sudo snap install --channel=3.3 maas

          ○ MAAS Initialization: <stong> Do this third!</stong>
            I followed the <em>"How to initialise MAAS for productionC</em> section to initialize MAAS.

                $MAAS_DBUSER = maasadmin
                $MAAS_DBPASS = {maasadmin-passwd}
                $MAAS_DBNAME = maascntlrdb
                $HOSTNAME = localhost

                $ sudo -i -u postgres psql -c "CREATE USER \"maasadmin\" WITH ENCRYPTED PASSWORD 'mAAsD3ployment'"  

                $ sudo -i -u postgres createdb -O "maasadmin" "maascntlrdb"

                $ vi /etc/postgresql/15/main/pg_hba.conf
                    # TYPE  DATABASE        USER            ADDRESS                 METHOD
                    host    maascntlrdb    	maasadmin       0/0                     md5

                keith@maas-controller:~$ sudo maas init region+rack --database-uri "postgres://maasadmin:mAAsD3ployment@localhost/maascntlrdb"
                MAAS URL [default=http://192.168.121.137:5240/MAAS]: 
                MAAS has been set up.             

                If you want to configure external authentication or use
                MAAS with Canonical RBAC, please run
                
                  sudo maas configauth
                
                To create admins when not using external authentication, run
                
                  sudo maas createadmin
                
                To enable TLS for secured communication, please run
                
                  sudo maas config-tls enable

    
                *** = Just hit enter key here (no value selected)
    
                keith@maas-controller:~$ sudo maas configauth
                [sudo] password for keith: 
                URL for the Canonical RBAC service (leave blank if not using the service): ***
                Path of the Candid authentication agent file (leave blank if not using the service): *** 
                 
                keith@maas-controller:~$ sudo maas createadmin
                Username: maasadmin
                Password: 
                Again: 
                Email: maasadmin@local.domain
                
                Import SSH keys [] (lp:user-id or gh:user-id):  ***
                keith@maas-controller:~$  

    
                keith@maas-controller:~$ sudo maas status
                bind9                            RUNNING   pid 39855, uptime 0:05:41
                dhcpd                            STOPPED   Not started
                dhcpd6                           STOPPED   Not started
                http                             RUNNING   pid 40321, uptime 0:05:23
                ntp                              RUNNING   pid 39987, uptime 0:05:38
                proxy                            RUNNING   pid 40294, uptime 0:05:26
                rackd                            RUNNING   pid 39858, uptime 0:05:41
                regiond                          RUNNING   pid 39860, uptime 0:05:41
                syslog                           RUNNING   pid 40267, uptime 0:05:28
                
</pre>

<pre>
<<<a href="https://maas.io/docs/how-to-do-a-fresh-install-of-maas"> From: How to configure MAAS</a> >>

# Generate the API-key for the login 
<strong>sudo maas apikey --username=maasadmin > api-key-file</strong>

# Login with the following command
<strong>maas login maasadmin http://192.168.121.137:5240/MAAS < api-key-file</strong>

<<<a href="https://maas.io/docs/try-out-the-maas-cli"> From: Try out the MAAS CLI</a> >>
# Set the DNS server IP address
<strong>maas maasadmin maas set-config name=upstream_dns value=8.8.8.8</strong>

# find a usable fabric by picking a valid bridge IP address like this:
<strong>maas maasadmin subnet read 192.168.121.0/24 | grep fabric_id</strong>
    "fabric_id": 0,

# find the name of the primary rack controller:
<strong>maas maasadmin rack-controllers read | grep hostname | cut -d '"' -f 4</strong>
maas-controller

# turn on DHCP like this:
<strong>maas maasadmin ipranges create type=dynamic start_ip=192.168.121.150 end_ip=192.168.121.170</strong>
Success.
Machine-readable output follows:
{
    "subnet": {
        "name": "192.168.121.0/24",
        "description": "",
        "vlan": {
            "vid": 0,
            "mtu": 1500,
            "dhcp_on": false,
            "external_dhcp": null,
            "relay_vlan": null,
            "fabric_id": 0,
            "space": "undefined",
            "name": "untagged",
            "primary_rack": null,
            "fabric": "fabric-0",
            "secondary_rack": null,
            "id": 5001,
            "resource_uri": "/MAAS/api/2.0/vlans/5001/"
        },
        "cidr": "192.168.121.0/24",
        "rdns_mode": 2,
        "gateway_ip": "192.168.121.2",
        "dns_servers": [],
        "allow_dns": true,
        "allow_proxy": true,
        "active_discovery": false,
        "managed": true,
        "disabled_boot_architectures": [],
        "space": "undefined",
        "id": 1,
        "resource_uri": "/MAAS/api/2.0/subnets/1/"
    },
    "type": "dynamic",
    "start_ip": "192.168.121.150",
    "end_ip": "192.168.121.170",
    "user": {
        "is_superuser": true,
        "username": "maasadmin",
        "email": "maasadmin@local.domain",
        "is_local": true,
        "resource_uri": "/MAAS/api/2.0/users/maasadmin/"
    },
    "comment": "",
    "id": 1,
    "resource_uri": "/MAAS/api/2.0/ipranges/1/"
}

# DHCP switch-on
<strong>maas maasadmin vlan update 0 untagged dhcp_on=True primary_rack=maas-controller</strong>
Success.
Machine-readable output follows:
{
    "vid": 0,
    "mtu": 1500,
    "dhcp_on": true,
    "external_dhcp": null,
    "relay_vlan": null,
    "fabric_id": 0,
    "space": "undefined",
    "name": "untagged",
    "primary_rack": "y7sbwq",
    "fabric": "fabric-0",
    "secondary_rack": null,
    "id": 5001,
    "resource_uri": "/MAAS/api/2.0/vlans/5001/"
}
  
</pre>
