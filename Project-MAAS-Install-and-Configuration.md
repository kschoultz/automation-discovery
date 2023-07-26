# Project-MAAS-Install-and-Configuration.md

<pre>
• Step 1: <strong>PostgreSQL Install</strong>
       ○ sudo apt install -y postgresql
          Note: The command will likely install the latest version (in my case 15.3).
          # Postgres v15.3 Initial Setup
          # Necessary steps from: https://stackoverflow.com/questions/1471571/how-to-configure-postgresql-for-the-first-time

       ○ sudo su postgres
        
       ○ postgres@maas-controller:~$ psql template1
          psql (15.3 (Ubuntu 15.3-0ubuntu0.23.04.1))
          Type "help" for help.
        
       ○ template1=# ALTER USER postgres with encrypted password '{postgres-passwd}';
         ALTER ROLE

       ○ template1=# \q
       
       ○ postgres@maas-controller:~$ vi /etc/postgresql/15/main/pg_hba.conf                            
         # Database administrative login by Unix domain socket
         local   all             postgres                                md5
       
       ○ postgres@maas-controller:~$ exit
       
       ○ keith@maas-controller:~$ sudo /etc/init.d/postgresql restart
         Restarting postgresql (via systemctl): postgresql.service.

       ○ keith@maas-controller:~$ psql template1 -U postgres
         Password for user postgres:
         psql (15.3 (Ubuntu 15.3-0ubuntu0.23.04.1))
        
       ○ keith@maas-controller:~$ sudo createuser -U postgres -d -e -E -l -P -r -s keith
         Enter password for new role:  <--- {keith's passwd}
         Enter it again:  <--- {keith's passwd}
         Password:  <--- {postgres' passwd}

       ○ keith@maas-controller:~$ psql -U postgres
         Password for user postgres:
         psql (15.3 (Ubuntu 15.3-0ubuntu0.23.04.1))
         Type "help" for help.

       ○ postgres=# \du
                                           List of roles
         Role name |                         Attributes                         | Member of
         -----------+------------------------------------------------------------+-----------
         keith     | Superuser, Create role, Create DB                          | {}
         postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}

       ○ postgres=# \q
                
• Step 2: <strong>MAAS Install</strong>
          Follow the steps outlined on the <a href="https://maas.io/docs/how-to-get-started-with-maas">How to get started with MAAS</a> documentation.

          ○ keith@maas-controller:~$ sudo snap install --channel=3.3 maas
            maas (3.3/stable) 3.3.4-13189-g.f88272d1e from Canonical✓ installed

          <ins>MAAS Initialization</ins>
            I followed the <em>"How to initialise MAAS for productionC</em> section to initialize MAAS.

                $MAAS_DBUSER = maasadmin
                $MAAS_DBPASS = {maasadmin-passwd}
                $MAAS_DBNAME = maascntlrdb
                $HOSTNAME = localhost

           ○ keith@maas-controller:~$ sudo su - postgres
             [sudo] password for keith:  <--- {keith's passwd}

           ○ postgres@maas-controller:~$ psql -c "CREATE USER \"maasadmin\" WITH ENCRYPTED PASSWORD '{maasadmin passwd}'"
             Password for user postgres:  <--- {postgres' db passwd}
             CREATE ROLE

           ○ postgres@maas-controller:~$ psql
             Password for user postgres:  <--- {postgres' db passwd}
             psql (15.3 (Ubuntu 15.3-0ubuntu0.23.04.1))
             Type "help" for help.

           ○ postgres=# \du
                                               List of roles
             Role name |                         Attributes                         | Member of
             -----------+------------------------------------------------------------+-----------
             keith     | Superuser, Create role, Create DB                          | {}
             maasadmin |                                                            | {}
             postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
                
           ○ postgres=# \q

           ○ postgres@maas-controller:~$ createdb -O "maasadmin" "maascntlrdb"
             Password:  <--- {postgres' db passwd}
              
           ○ postgres@maas-controller:~$ psql
             Password for user postgres:  <--- {postgres' db passwd}
             psql (15.3 (Ubuntu 15.3-0ubuntu0.23.04.1))
             Type "help" for help.
              
             postgres=# \l
                                                                List of databases
                 Name     |   Owner   | Encoding |   Collate   |    Ctype    | ICU Locale | Locale Provider |   Access privileges
             -------------+-----------+----------+-------------+-------------+------------+-----------------+-----------------------
              maascntlrdb | maasadmin | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            |
              postgres    | postgres  | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            |
              template0   | postgres  | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
                          |           |          |             |             |            |                 | postgres=CTc/postgres
              template1   | postgres  | UTF8     | en_US.UTF-8 | en_US.UTF-8 |            | libc            | =c/postgres          +
                          |           |          |             |             |            |                 | postgres=CTc/postgres
              (4 rows)

           ○ postgres=# \q
                    
           ○ postgres@maas-controller:~$ vi /etc/postgresql/15/main/pg_hba.conf
             # TYPE  DATABASE        USER            ADDRESS      METHOD
             host    maascntlrdb     maasadmin       0/0          md5


                    
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
