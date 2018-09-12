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

<div style="text-align:center">
  <img src ="/demo/images/demo.jpg" alt="demo.png"/>
</div>

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

Install and run DHCP server goes here.

#### DNS

Install and run DNS server goes here.

#### Configure NAT

How to configure NAT goes here.

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

httpd = BaseHTTPServer.HTTPServer(('0.0.0.0', 4443), SimpleHTTPServer.SimpleHTTPRequestHandler)
httpd.socket = ssl.wrap_socket (httpd.socket, certfile='./server.pem', server_side=True)
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

Write the different test scenarions used

## Troublshooting

> Murphy's Law says: "Anything that can go wrong, will go wrong."

What to do in case something is going wrong.


