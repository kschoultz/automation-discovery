# Project-Automation-VM-Creation.md

Listed below are notes and steps I took to utilize the ESXi/vSphere lab environment on my
laptop, in conjunction with the Canonical Juju product to emulate automating the creation
and configuration of a Red Hat Enterprise Linux VM.

<pre>
<ins>Prerequisites</ins>
Please review the <a href="https://github.com/kschoultz/automation-discovery/blob/main/Project-VSphere-ESXi-Lab-Build.md">Project-VSphere-ESXi-Lab-Build.md </a> document for the step-by-step
details of how to build out the ESxi/vSphere lab environment.
</pre>
<pre>
Software:
  • <a href="https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux">Red Hat Enterprise Linux 9.2 (Plow) - 64bit [60-day trial license]</a>
  • <a href="https://juju.is/docs/olm">Canonical Juju</a>
</pre>

<pre>
<ins>Create the Automation Controller VM</ins>
We'll use the RHEL9 ISO file downloaded above to create a simple VM that will be the central control center
where we automate the creation, configuration and software deployments of other RHEL9 VMs inside the ESXi/vSphere
data center and clusters.

• Step 1: Create the Automation VM on the VMware Workstation Pro level (not inside the ESXi/vSphere environment).
           Here are VM settings I used. I also leveraged the <ins>Typical (recommended)</ins> option
           to create this VM instead of manually assigning each of the resource values below.
            Memory: 2 GB
            Processors: 2
            Hard Disk(SCSI): 20 GB
            CD/DVD (SATA): points to the install ISO file
            Network Adapter: NAT
            USB Controller: Present
            Sound Card: Auto Detect
            Printer: Present
            Display: Auto Detect
                     Note that I use the <em>Display Scaling > Stretch mode > Free</em> stretch option on these
                     VM consoles; otherwise, the console font is too small to easily read when booting up. 
</pre>
<pre>
• Step 2: Here we'll need to follow the process outlined by Canonical to install and update OS-sepcific packages
          and utilities required to implement Juju. All of these steps that follow assume that the VM has internet
          access via the NATed network.

            a) Reviewing the steps outlined on the <a href="https://snapcraft.io/install/juju/rhel">Install juju on Red Hat
               Enterprise Linux</a> web site, we'll need 
               to first install the latest EPEL repository. The <strong>epel-release-latest-9.noarch.rpm</strong> package can be
               downloaded from the <a href="https://dl.fedoraproject.org/pub/epel/">Fedora EPEL</a> web site.

               Run the following commands from the command prompt:
                ○ <strong>sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-9.noarch.rpm</strong>
                ○ <strong>sudo dnf upgrade</strong>

            b) Snap can now be installed as follows:
                ○ <strong>sudo yum install snapd</strong>

            c) Once installed, the systemd unit that manages the main snap communication socket needs to be enabled:
                ○ <strong>sudo systemctl enable --now snapd.socket</strong>

            d) To enable classic snap support, enter the following to create a symbolic link between <em>/var/lib/snapd/snap</em>
               and <em>/snap</em>:
                ○ <strong>sudo ln -s /var/lib/snapd/snap /snap</strong>

            e) The web site suggests we now "Either log out and back in again or restart your system to ensure snap’s paths
               are updated correctly". We'll reboot the VM to ensure everything works as expected.

            f) Once the VM has completely recovered from the restart, we move forward with installing Juju:
                ○ <strong>sudo snap install juju --classic</strong>

            g) We can now verify that Juju is in fact installed by executing the following:
                ○ <strong>juju help</strong>

                The output should look similar to the following:

                    Juju provides easy, intelligent application orchestration on top of Kubernetes,
                    cloud infrastructure providers such as Amazon, Google, Microsoft, Openstack,
                    MAAS (and more!), or even your local machine via LXD.

                    See https://juju.is for getting started tutorials and additional documentation.

                    Starter commands:

                        bootstrap           Initializes a cloud environment.
                        add-model           Adds a hosted model.
                        deploy              Deploys a new application.
                        status              Displays the current status of Juju, applications, and units.
                        add-unit            Adds extra units of a deployed application.
                        relate              Adds a relation between two applications.
                        expose              Makes an application publicly available over the network.
                        models              Lists models a user can access on a controller.
                        controllers         Lists all controllers.
                        whoami              Display the current controller, model and logged in user name.
                        switch              Selects or identifies the current controller and model.
                        add-k8s             Adds a k8s endpoint and credential to Juju.
                        add-cloud           Adds a user-defined cloud to Juju.
                        add-credential      Adds or replaces credentials for a cloud.

                    Interactive mode:

                    When run without arguments, Juju will enter an interactive shell which can be
                    used to run any Juju command directly.

                    Help commands:

                        juju help           This help page.
                        juju help <command> Show help for the specified command.

                    For the full list of supported commands run:

                        juju help commands                
</pre>