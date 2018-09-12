# MUD Implementation Demo

In this guide, we're going to describe the setup used for the MUD implementation demo.

## Setup Environement and Hardware

### Hardware Required

For the initial demo, we used the following pieces of hardware:

- 1x Omnia Turris router
- 5x Raspberry PIs
- 1x Server
- 1x Laptop
- 2x L2 Switch
- Monitors, Keyboard, Mouse, and HDMI Switcher
- Ethernet Cables
- USB Dock and Cables (to power the RPis)

The devices listed above were connected following the topology shown in the next section.

### Topology
The topology that was used in this demo is as follows:

<p align="center">
  <img src ="/demo/images/demo.jpg" alt="demo.jpg" height="auto" width="85%"/>
</p>

> The HDMI switcher and monitors are to makes the demoer's life easier, by avoiding switching the monitor, keyboard and mouse between every PI. 

### How Does It Work?

The Turris Omnia router in the core of our demo. It's where OVS will be running and it's where the MUD rules will be pushed. 
To the switch are connected a bunch of Raspberry PIs. These PIs are used to mimic the different devices in an IoT setup (e.g. Smart Home).
Then we have the ODL server. In this server we will run our OpenDaylight app, which will be the SDN controller in our setup.
This controller is responsible of parsing and pushing the different MUD rules to the OVS.
A laptop was used to ssh to the Turris Omnia router and manage it. Layer 2 switches were used to conenct the different devices.

## Configuration and Scripts

In this section, we describe the different configurations we had to do and the different scripts used.

### Network Configuration

Network configuration goes here.

### Services Configuration (DHCP, DNS, NAT, Web Server)

In our demo, we are running different services, namely DHCP, DNS, NAT, and a Web Server.
We chose to run DHCP, DNS on a RPi that we called **Services**, the web server on another RPi that we called simply **Web Server**.
The web server (203.0.113.2) is running on a fake public network (203.0.113.0/24) with a fake domain name (nist.local), just for demo purposes. 
The services RPi is connected to both network: the private network 10.0.41.0/24 where it's acting as a GW (10.0.41.254), and the public network 203.0.113.0/24 
having the address (203.0.113.1).
To connect both network we configured NAT on the services RPI.

#### DHCP

DHCP is required for our demo. The only reason we need it to be able to communication the MUD URL to the controller.
In fact, as explained in the [ietf-draft](https://datatracker.ietf.org/doc/draft-ietf-opsawg-mud/), there are different ways for a MUD device to communicate its URL to the controller. One way is embedding that URL in a DHCP request. Our current implementation supports the DHCP method and 2 others.

To set up a DHCP server in ourinternal network, we used [dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html).

```
# apt-get update
# apt-get install dnsmasq
```

After dnsmasq has been installed, it an be configured by editing the file `/etc/dnsmasq.conf`. The file has detailed comments with examples and is self-explainatory.

Open file:

```
# nano /etc/dnsmasq.conf 
```

Since we have a finite set of RPIs and we want to map their names to their IP addresses (e.g. RPI_1 @ 10.0.41.101, RPI_2 @ 10.0.41.102, and so on..), so we opted for static DHCP leases configuration.

Add static leases:
```
# generic format
# dhcp-host=<mac address>,<ip address>
# rpi_1 (mud device)
dhcp-host=b8:27:eb:68:30:2d,10.0.41.101
# rpi_2
dhcp-host=todo,10.0.41.102
# rpi_3 (non-mud device)
dhcp-host=b8:27:eb:96:84:44,10.0.41.103
# rpi_4
dhcp-host=todo,10.0.41.104
```

A DHCP range, the gateway and the dns server options can be configured as follows:

```
# generic format
# dhcp-range=<start-ip>,<end-ip>,<netmask>,<lease>
# range
dhcp-range=10.0.41.100,10.0.41.150,255.255.255.0,48h
# gateway
dhcp-option=3,10.0.41.254
# dns server
dhcp-option=6,10.0.41.254
```

#### DNS

Install and run DNS server goes here.

#### Configure NAT

NAT is needed to allow our internal private network (10.0.41.0/24) to connect to the public network (203.0.113.0/24). We decided to put the NAT functionality on the **services** RPi. Following is the complete configuration.

<p align="center">
  <img src ="/demo/images/nat.jpg" alt="nat.jpg" height="50%" width="50%"/>
</p>

First step is to enable IPv4 forwarding. Most linux distros come with IPv4 forwarding disabled. In our case, since we need to route traffic from our private network to the public network, we need to enable it on the RPi.

- enable IP forwarding

Let's first check if IPv4 is enabled:

```
# sysctl net.ipv4.ip_forward
net.ipv4.ip_forward = 0
```

> 0: disabled
> 1: enabled

There are different ways to configure set this parameter to 1. The permanent way to do this is:

```
# nano /etc/sysctl.conf
```

Look for `net.ipv4.ip_forward=0` and set it to 1.

- configure iptables

Most Raspberry PIs come with empty iptables and ACCEPT policy by default. Just in case we can delete all rules using the command:

```
# iptables -F
```

We are trying to bridge both interfaces eth0 (10.0.41.254) and eth1 (203.0.113.1) on the **services** RPi. We start by setting up MASQUERADING (i.e. replacing the sender's address by the router's address):

```
# iptables -t nat -A POSTROUTING -o eth1 -j MASQUERADE
```

Then, we need to forward incoming packets (from eth1 to eth0) that have connections ESTABLISHED and RELATED:

```
# iptables -A FORWARD -i eth1 -o eth0 -m state --state RELATED,ESTABLISHED -j ACCEPT
```

Then, forward all outgoing packets (from eth0 to eth1):

```
# iptables -A FORWARD -i eth0 -o wlan0 -j ACCEPT
```

Save the firewall rules:

```
# sh -c "iptables-save > /etc/iptables.ipv4.nat"
```

And to apply them from boot, add the following line at the end of `/etc/network/interfaces`:

```
# up iptables-restore < /etc/iptables.ipv4.nat
```

Configuration should be applied by now. You can check it using:

```
# iptables -L
```

And then run tests to check if devices from 10.0.41.0/24 can access devices on 203.0.113.0/24.

#### Web Server

First generate `cert.pem` with the command:
```
# openssl req -new -x509 -keyout cert.pem -out cert.pem -days 365 -nodes
```
It will generate a CA certificate that will be used by the web server.

- Code for the HTTPS server

```python
import BaseHTTPServer, SimpleHTTPServer
import ssl

httpd = BaseHTTPServer.HTTPServer(('0.0.0.0', 443), SimpleHTTPServer.SimpleHTTPRequestHandler)
httpd.socket = ssl.wrap_socket (httpd.socket, certfile='./cert.pem', server_side=True)
httpd.serve_forever()
```
Copy and paste above code in a file and save it, e.g. https-server.py

 - Run the web server
 
 ```
 # python https-server.py
 ```

> **Note**: Create an `index.html` file and put it in the root directory of your web server. This is optional.

### OVS and OpenFlow

For more details about OpenVSwitch and OpenFlow check the officila documentation. In what follows, we list only the commands that were used in the demo.

#### Install OpenVSwitch

Our Turris Omnia router is running the [latest version of OpenWrt](https://repo.turris.cz/omnia/medkit/omnia-medkit-latest.tar.gz) available on the [Turris Omnia docs page](https://doc.turris.cz/doc/en/start).

> To flash or factory reset your Turris Omnia follow instructions [here](https://doc.turris.cz/doc/en/howto/omnia_factory_reset).
> If you have any issues, send an email to the support...yes, they reply!

```
# opkg update
# opkg install openvswitch
```

#### Start OpenVSwitch

```
# /etc/init.d/openvswitch start
```

OpenVSwitch might run automatically after installing it, so you can check if it's running using the command
```
# ps w | grep ovs
```
or
```
# ovs-vsctl show 
```
For this second command, you should see somthing random like the following:
```
123abc1b-d10b-3268-a77c-795aef39
```

#### Create OVS switch

```
$ ovs-vsctl add-br <bridge-name>
```

#### Create OVS ports

To create OVS ports, we need to attach the Turris Omnia router ports to the OVS that was created. To do so, run the command:
```
# ovs-vsctl add-port <bridge-name> <port-name>
```

> The `<port-name>` is the name of the port on the router, e.g. eth0, wlan1.

For more information on how to configure the router ports, check the [network config file]().

#### The Complete Demo Configuration

Put full configuration here.

### Different Files and Scripts

#### Network
TBD

#### Wireless
TBD


## Test Scenarios

Write the different test scenarios used.

## Troublshooting

> Murphy's Law says: "Anything that can go wrong, will go wrong."

What to do in case something is going wrong? STAY CALM!

_Just kidding_, but yes, in any case stay calm.
