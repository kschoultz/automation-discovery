# Project-Juju-Automation-Enablement.md

Listed below are notes and steps I took to utilize the Canonical Juju product to connect
to the vSphere Server and create a VM.

<pre>
<ins>Adding a vSphere cloud to Juju</ins>
The following steps are derived from the <a href="https://juju.is/docs/olm/vmware-vsphere">How to use VMware vSphere with Juju</a> web site.
</pre>

<pre>
• Step 1: Create the <em>vsp-cloud.yaml</em> file

  clouds:
    vsp-cloud:
      type: vsphere
      auth-types: [userpass]
      endpoint: 192.168.121.129  # the IPAddr for VMware vCenter Server
      regions:
      DC0: {}  # these empty maps are necessary
</pre>

<pre>
• Step 2: To add cloud ‘vsp-cloud’, assuming the configuration file is vsp-cloud.yaml in the current directory, we would run:
          ○ <strong>juju add-cloud --local vsp-cloud vsp-cloud.yaml</strong>
</pre>

<pre>
• Step 3: Confirm that you’ve added the cloud correctly.
          Ask Juju to report the clouds that it has registered:
            ○ <strong>juju clouds --local</strong>

              Clouds available on the client:
              Cloud      Regions  Default    Type     Credentials  Source    Description
              localhost  1        localhost  lxd      0            built-in  LXD Container Hypervisor
              vsp-cloud  1        dc0        vsphere  0            local
</pre>

<pre>
• Step 4: Use the add-credential command to interactively add your credentials to the new cloud:
          ○ <strong>juju add-credential vsp-cloud</strong>

            This operation can be applied to both a copy on this client and to the one on a controller.
            No current controller was detected and there are no registered controllers on this client: either bootstrap one or register one.

            Enter credential name: vsp-cloud-creds

            Regions
              DC0

            Select region [any region, credential is not region specific]: DC0

            Using auth-type "userpass".

            Enter user: administrator@vsphere.local

            Enter password: ************

            Enter vmfolder (optional):

            Credential "vsp-cloud-creds" added locally for cloud "vsp-cloud".
</pre>

<pre>
• Step 5: Confirm that you’ve added the credential correctly
          To view the credentials that Juju knows about, use the credentials command and inspect both
          remote and locally stored credentials:

          ○ <strong>juju credentials --local</strong>

            Client Credentials:
            Cloud      Credentials
            vsp-cloud  vsp-cloud-creds

          ○ <strong>juju show-credential</strong>

            client-credentials:
              vsp-cloud:
                vsp-cloud-creds:
                  content:
                    auth-type: userpass
                    user: administrator@vsphere.local
</pre>

<pre>
• Step 6: Create a controller
          You are now ready to create a Juju controller for cloud ‘vsp-cloud’:

          ○ <strong>juju bootstrap vsp-cloud vsp-controller</strong>

          Creating Juju controller "vsp-controller" on vsp-cloud/DC0
          Looking for packaged Juju agent version 2.9.43 for amd64
          Located Juju agent version 2.9.43-ubuntu-amd64 at https://streams.canonical.com/juju/tools/agent/2.9.43/juju-2.9.43-linux-amd64.tgz
          WARNING empty string passed as vm-folder, using Datacenter root folder instead
          Launching controller instance(s) on vsp-cloud/DC0...
          WARNING empty string passed as vm-folder, using Datacenter root folder instead
          - juju-4efe93-0 (arch=amd64 mem=3.5G)
          Installing Juju agent on bootstrap instance
          Fetching Juju Dashboard 0.8.1
          Waiting for address
          Attempting to connect to 192.168.121.132:22
          Attempting to connect to fe80::250:56ff:fe3a:72e7:22
          Connected to 192.168.121.132
          Running machine configuration script...
          Bootstrap agent now started
          Contacting Juju controller at 192.168.121.132 to verify accessibility...

          Bootstrap complete, controller "vsp-controller" is now available
          Controller machines are in the "controller" model
          Initial model "default" added
</pre>

https://dl.fedoraproject.org/pub/fedora/linux/releases/36/Everything/source/tree/Packages/r/