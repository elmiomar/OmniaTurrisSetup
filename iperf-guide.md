# Using Iperf for Measurements

## What is Iperf
>[Iperf](https://en.wikipedia.org/wiki/Iperf) is a widely used tool for network performance measurement and tuning. It is significant as a cross-platform tool that can produce standardized performance measurements for any network. Iperf has client and server functionality, and can create data streams to measure the throughput between the two ends in one or both directions. Typical Iperf output contains a time-stamped report of the amount of data transferred and the throughput measured. The data streams can be either TCP or UDP.
>
>-wikipedia

## Install Iperf

To install Iperf, log into the RPi, open a terminal, and run the command:

`sudo apt-get install iperf`

## Tests

We suppose we have 2 RPis, one acting as a server and the other one as a client.

#### TCP

The TCP test pushes as much traffic as possible between the two RPis, trying to saturate the bandwidth.

To start a TCP server, log into one of the RPis, open a terminal, and run the command:

`iperf -s`

To start the client, log into the other RPi, open a terminal, and run the command:

`iperf -c <SERVER_IP>`

A report will be generated at the end.

#### UDP

With you UDP we can control the amount of traffic that is pushed from the client to the server.

Server side:

`iperf -s -u`

Client side:

`iperf -c <SERVER_IP> -u [OPTIONS]`

> **Note**: it is preferable to run the server first then the client. 
