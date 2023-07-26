# Project-MAAS-Install-and-Configuration.md

<pre>
• Step 1: <ins>PostgreSQL Install</ins>
       ○ sudo apt install -y postgresql
          Note: The command will likely install the latest version (in my case 15.3).
          # Postgres v15.3 Initial Setup
          # Necessary steps from: https://stackoverflow.com/questions/1471571/how-to-configure-postgresql-for-the-first-time

       ○ sudo su postgres
        
       ○ postgres@maas-controller:~$ psql template1
          psql (15.3 (Ubuntu 15.3-0ubuntu0.23.04.1))
          Type "help" for help.
        
       ○  template1=# ALTER USER postgres with encrypted password '{postgres-passwd}';
          ALTER ROLE
                                    
       ○  keith@maas-controller:~$ sudo vi /etc/postgresql/15/main/pg_hba.conf                            
            # Database administrative login by Unix domain socket
            local   all             postgres                                md5
        
       ○  keith@maas-controller:~$ sudo /etc/init.d/postgresql restart
          Restarting postgresql (via systemctl): postgresql.service.
        
       ○  sudo createuser -U postgres -d -e -E -l -P -r -s keith
    
• Step 2:<ins>MAAS Install</ins>
         Follow the steps outlined on the <a href="https://maas.io/docs/how-to-get-started-with-maas">How to get started with MAAS</a> documentation.

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
