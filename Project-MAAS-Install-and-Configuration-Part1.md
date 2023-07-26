# Project-MAAS-Install-and-Configuration-Part1.md

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
            I followed the <em>"How to initialise MAAS for production</em> section to initialize MAAS.

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
             # Add the below user entry to the bottom of the file.
             # TYPE  DATABASE        USER            ADDRESS      METHOD
             host    maascntlrdb     maasadmin       0/0          md5

           ○ postgres@maas-controller:~$ exit
                    
           ○ keith@maas-controller:~$ sudo maas init region+rack --database-uri "postgres://maasadmin:{maasadmin-passwd}@localhost/maascntlrdb"
             [sudo] password for keith:
             MAAS URL [default=http://192.168.1.111:5240/MAAS]:
             MAAS has been set up.             

                If you want to configure external authentication or use
                MAAS with Canonical RBAC, please run
                
                  sudo maas configauth
                
                To create admins when not using external authentication, run
                
                  sudo maas createadmin
                
                To enable TLS for secured communication, please run
                
                  sudo maas config-tls enable

    
           *** Note: For the following steps, just hit enter key here (no value selected)
    
           ○ keith@maas-controller:~$ sudo maas configauth
             [sudo] password for keith: 
             URL for the Canonical RBAC service (leave blank if not using the service): ***
             Path of the Candid authentication agent file (leave blank if not using the service): *** 
                 
           ○ keith@maas-controller:~$ sudo maas createadmin
             Username: maasadmin
             Password:  <--- {maasadmin passwd}
             Again:  <--- {maasadmin passwd} 
             Email: maasadmin@local.domain
                
             Import SSH keys [] (lp:user-id or gh:user-id):  ***
             keith@maas-controller:~$  

           ○ keith@maas-controller:~$ sudo maas status
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
• Step 3: <<<a href="https://maas.io/docs/how-to-do-a-fresh-install-of-maas"> From: How to configure MAAS</a> >>
             # Generate the API-key for the login 
           ○ keith@maas-controller:~$ sudo maas apikey --username=maasadmin > api-key-file
             [sudo] password for keith:
       
           ○ keith@maas-controller:~$ ll api-key-file
             -rw-rw-r-- 1 keith keith   71 Jul 26 16:43 api-key-file

             # Login with the following command
           ○ keith@maas-controller:~$ maas login maasadmin http://192.168.1.111:5240/MAAS < api-key-file
             API key (leave empty for anonymous access): ***
              
             You are now logged in to the MAAS server at
             http://192.168.1.111:5240/MAAS/api/2.0/ with the profile name
             'maasadmin'.
              
             For help with the available commands, try:
              
               maas maasadmin --help
</pre>
<pre>
• Step 4: <<<a href="https://maas.io/docs/try-out-the-maas-cli"> From: Try out the MAAS CLI</a> >>
             # Set the DNS server IP address
           ○ keith@maas-controller:~$ maas maasadmin maas set-config name=upstream_dns value=8.8.8.8

           ○ keith@maas-controller:~$ maas login maasadmin http://192.168.1.111:5240/MAAS/api/2.0/ $(head -1 api-key-file)
              
             You are now logged in to the MAAS server at
             http://192.168.1.111:5240/MAAS/api/2.0/ with the profile name
             'maasadmin'.
              
             For help with the available commands, try:
              
               maas maasadmin --help
              
            ○ keith@maas-controller:~$ maas maasadmin maas set-config name=upstream_dns value=8.8.8.8
              Success.
              Machine-readable output follows:
              OK       
              
            ○ keith@maas-controller:~$ maas maasadmin maas get-config name=upstream_dns
              Success.
              Machine-readable output follows:
              "8.8.8.8"
                     
              # find a usable fabric by picking a valid "Host Only" IP address like this:
            ○ keith@maas-controller:~$ maas maasadmin subnet read 10.1.1.0/24 | grep fabric_id
              "fabric_id": 1,

              # find the name of the primary rack controller:
            ○ keith@maas-controller:~$ maas maasadmin rack-controllers read | grep hostname | cut -d '"' -f 4
              maas-controller

              # turn on DHCP like this:
            ○ keith@maas-controller:~$ maas maasadmin ipranges create type=dynamic start_ip=10.1.1.20 end_ip=10.1.1.50
              Success.
              Machine-readable output follows:
              {
                  "subnet": {
                      "name": "10.1.1.0/24",
                      "description": "",
                      "vlan": {
                          "vid": 0,
                          "mtu": 1500,
                          "dhcp_on": true,
                          "external_dhcp": null,
                          "relay_vlan": null,
                          "fabric": "fabric-1",
                          "id": 5002,
                          "primary_rack": "83nbmc",
                          "fabric_id": 1,
                          "name": "untagged",
                          "space": "undefined",
                          "secondary_rack": null,
                          "resource_uri": "/MAAS/api/2.0/vlans/5002/"
                      },
                      "cidr": "10.1.1.0/24",
                      "rdns_mode": 2,
                      "gateway_ip": null,
                      "dns_servers": [],
                      "allow_dns": true,
                      "allow_proxy": true,
                      "active_discovery": false,
                      "managed": true,
                      "disabled_boot_architectures": [],
                      "id": 4,
                      "space": "undefined",
                      "resource_uri": "/MAAS/api/2.0/subnets/4/"
                  },
                  "type": "dynamic",
                  "start_ip": "10.1.1.20",
                  "end_ip": "10.1.1.50",
                  "user": {
                      "is_superuser": true,
                      "username": "maasadmin",
                      "email": "maasadmin@local.domain",
                      "is_local": true,
                      "resource_uri": "/MAAS/api/2.0/users/maasadmin/"
                  },
                  "comment": "",
                  "id": 4,
                  "resource_uri": "/MAAS/api/2.0/ipranges/4/"
              }
       
              # DHCP switch-on
            ○ keith@maas-controller:~$ maas maasadmin vlan update 1 untagged dhcp_on=True primary_rack=maas-controller
              Success.
              Machine-readable output follows:
              {
                  "vid": 0,
                  "mtu": 1500,
                  "dhcp_on": true,
                  "external_dhcp": null,
                  "relay_vlan": null,
                  "fabric": "fabric-1",
                  "id": 5002,
                  "primary_rack": "83nbmc",
                  "fabric_id": 1,
                  "name": "untagged",
                  "space": "undefined",
                  "secondary_rack": null,
                  "resource_uri": "/MAAS/api/2.0/vlans/5002/"
              }  
</pre>
<pre>
              # Generate an SSH key that will be used by MAAS to access client servers.
            ○ keith@maas-controller:~$ ssh-keygen
              Generating public/private rsa key pair.
              Enter file in which to save the key (/home/keith/.ssh/id_rsa):
              Enter passphrase (empty for no passphrase):  <--- {leave blank/hit enter key}
              Enter same passphrase again:  <--- {leave blank/hit enter key}
              Your identification has been saved in /home/keith/.ssh/id_rsa
              Your public key has been saved in /home/keith/.ssh/id_rsa.pub
              The key fingerprint is:
              SHA256:HCHCOx2e6ndtk2qdzy6W/yDjAVNYs9Fern8mVoHKYBc keith@maas-controller
              The key's randomart image is:
              +---[RSA 3072]----+
              |   .. . . +.     |
              |    .... + E. .  |
              |     + oo o..o.  |
              |    o +. = ..... |
              |     o  S + ..  .|
              |    .    o o.   .|
              |   .     o++.. . |
              |    . . o.@= .+ o|
              |     . o.+.*=o.+ |
              +----[SHA256]-----+
              keith@maas-controller:~$ ll .ssh/id_rsa.pub
              -rw-r--r-- 1 keith keith  575 Jul 26 18:00 id_rsa.pub
</pre>
