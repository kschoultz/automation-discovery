# Project-MAAS-Install-and-Configuration-Part2.md

The MAAS configuration continues; however, for the sake of saving time we're going to reference the MAAS docuemntation
and/or the HTTP User interface when completing steps or applying changes.

• Step 1: Import the SSH public key that was generated previously.

<img hspace="50" width=900 src="images/MAAS-Upload-SSH-Key01.png">

<img hspace="50" width=900 src="images/MAAS-Upload-SSH-Key02.png">

<img hspace="50" width=900 src="images/MAAS-Upload-SSH-Key03.png">

• Step 2: Set the Netwwork Discovery fabric to the <em>Host Only</em> VLAN (10.1.1.0/24)

<img hspace="50" width=900 src="images/Network-Discovery-Changes01.png">
<img hspace="50" width=900 src="images/Network-Discovery-Changes02.png">

• Step 3: Remove (delete) any previously discovered machines/devices on the public bridged VLAN (192.168.1.0/24)

<img hspace="50" width=900 src="images/Network-Discovery-Changes03.png">
