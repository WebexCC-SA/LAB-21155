# Configure Survivability Gateway

## Configure Survivability Gateway (SGW)

* Continuing on Workstation 1, Minimize all applications and open a (SSH) <strong>Putty</strong> session to survivability gateway at 198.18.133.226.  Login as admin and dCloud123!

!!! note "Note"

    All the commands are that are required to configure Local Gateway are put in a text file called <em><strong>Wx</strong></em><em><strong>1</strong></em><em><strong>2026</strong></em><em><strong> – L</strong></em><em><strong>GW</strong></em> <em><strong>-</strong></em> <em><strong>Config.txt</strong></em> on workstation 1 <strong>Desktop</strong> folder.  If you are having trouble copying these commands from this document due to formatting issues, you can copy those commands from this file to <strong>Putty</strong>.  Right click on the file and click Edit with Notepad++.

* Configure the required licenses and throughput values for CUBE using the below commands.

!!! note "Note"

    Ignore the warning about <strong>write </strong>command.

<strong>Configuration:</strong>

```text
configure terminal
license boot level network-essentials addon dna-essentials 
platform hardware throughput level MB 250
end
```

!!! note "Note"

    Licenses commands differ based on the platform. Refer to the official documentation for more information.  In this lab we are suing 250 Mbps throughput. Select the appropriate throughput level for the number of calls that you anticipate.  Ignore any warnings to write/reboot.

![](./assets/image72.png)

Survivability gateway requires publicly signed certificate that will be used during TLS handshake with Webex.  The required certificates are already generated and put on <strong>workstation 1 Desktop</strong> in <strong>certs</strong> &gt; <strong>cube</strong> folder.

!!! note "Note"

    Important! This certificate must meet the following requirements:

    Must be signed by one of the [Webex trusted CA](https://help.webex.com/en-us/article/WBX9000008850/What-Root-Certificate-Authorities-are-Supported-for-Calls-to-Cisco-Webex-Audioand-Video-Platforms) &amp; FQDN which was entered as a Host name while adding Survivability service in Control Hub must be present in the SAN field

* Load/install the certificates using the below command.  The certificate should be imported successfully.

<strong>Configuration:</strong>

```text
crypto pki import webex-sgw pkcs12 ftp://cisco:cisco@198.18.1.36/certs/cube/certificate.pfx password dCloud123!
```

![](./assets/image73.png)

* Configure the above certificate as the default certificate during TLS handshake with Webex.

<strong>Configuration:</strong>

```text
configure terminal 
sip-ua
no remote-party-id 
retry invite 2 
transport tcp tls v1.2
crypto signaling default trustpoint webex-sgw 
handle-replaces
end
```

![](./assets/image74.png)

* Configure the voice service commands for Survivability Gateway using the below commands.

!!! note "Note"

    198.18.1.0/24 and 198.18.133.0/24 represent trusted address ranges. You don't need to enter directly connected subnets as the Survivability Gateway trusts them automatically.

The “registrar server” enables the SIP registrar which allows Endpoints to register to the gateway.

<strong>Configuration:</strong>

```text
configure terminal 
voice service voip
ip address trusted list
ipv4 198.18.1.0 255.255.255.0
ipv4 198.18.133.0 255.255.255.0
allow-connections sip to sip
supplementary-service media-renegotiate 
no supplementary-service sip refer
trace 
sip
asymmetric payload full 
registrar server
end
```

![](./assets/image75.png)

* Configure/Enable the IOS-XE platform for survivability mode using the below commands

<strong>Configuration:</strong>

```text
configure terminal 
voice register global 
mode webex-sgw 
max-dn 50
max-pool 50 
end
```

![](./assets/image76.png)

* Configure NTP server, codec preferences and voice register pool with below commands.  The id network 0.0.0.0 with mask 0.0.0.0 makes devices from any network can register to this gateway.

<strong>Configuration:</strong>

```text
configure terminal
ntp server 198.18.128.1 
end
!
configure terminal 
voice class codec 1
codec preference 1 g711ulaw 
codec preference 2 g711alaw 
end
!
configure terminal 
voice register pool 1
id network 0.0.0.0 mask 0.0.0.0 
dtmf-relay rtp-nte
voice-class codec 1 
end
```

![](./assets/image77.png)

!!! note "Note"

    There are other parts of the configuration which not part of this lab. Refer to the template (you downloaded above) or the official documentation for further details. Some of these configuration elements include:

    Emergency Calling

    Dial-peers for the PSTN dialing

    Music on Hold

    Etc.

This completes configuring Survivability Gateway (SGW).
