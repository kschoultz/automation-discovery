# Project-Juju-Automation-Enablement.md

Listed below are notes and steps I took to utilize the Canonical Juju product to connect
to the vSphere Server and create a VM.

<pre>
<ins>Adding a vSphere cloud to Juju</ins>
The following steps are derived from the <a href="https://juju.is/docs/olm/vmware-vsphere">How to use VMware vSphere with Juju</a> web site.
</pre>

<pre>
• Step 1: Create the <em>vsp-cloud.yaml</em> file
<span style="color:lightblue">
  clouds:
    vsp-cloud:
      type: vsphere
      auth-types: [userpass]
      endpoint: 192.1681.121.129  # the IPAddr for VMware vCenter Server
      regions:
      dc0: {}  # these empty maps are necessary
</span>
</pre>

<pre>
• Step 2: To add cloud ‘vsp-cloud’, assuming the configuration file is vsp-cloud.yaml in the current directory, we would run:
          ○ <strong>juju add-cloud --local vsp-cloud vsp-cloud.yaml</strong>
</pre>

<pre>
• Step 3: Confirm that you’ve added the cloud correctly.
          Ask Juju to report the clouds that it has registered:
            ○ <strong>juju clouds --local</strong>
<span style="color:orange">
              Clouds available on the client:
              Cloud      Regions  Default    Type     Credentials  Source    Description
              localhost  1        localhost  lxd      0            built-in  LXD Container Hypervisor
              vsp-cloud  1        dc0        vsphere  0            local
</span>
</pre>

<pre>
• Step 4: Use the add-credential command to interactively add your credentials to the new cloud:
          ○ <strong>juju add-credential vsp-cloud</strong>
          <span style="color:orange">
            This operation can be applied to both a copy on this client and to the one on a controller.
            No current controller was detected and there are no registered controllers on this client: either bootstrap one or register one.
            Enter credential name: vsp-cloud-creds

            Regions
              dc0</span>

            <span style="color:orange">Select region [any region, credential is not region specific]:</span> dc0

            <span style="color:orange">Using auth-type "userpass".</span>

            <span style="color:orange">Enter user:</span> administrator@vsphere.local

            <span style="color:orange">Enter password:</span> ************

            <span style="color:orange">Enter vmfolder (optional):</span> juju-root

            <span style="color:orange">Credential "vsp-cloud-creds" added locally for cloud "vsp-cloud".</span>
</pre>
