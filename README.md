# Omnia Turris Switch Setup
These are the steps for the Omnia Turris switch setup, used in the SDN/MUD project.

## Omnia Turris Switch 
The front panel of the turris switch that I have looks like:
<p align="center"> 
<img src="https://github.com/elmiomar/OmniaTurrisSetup/blob/master/img/turris-front.jpg">
</p>

The back panel of the turris switch that I have looks like:
<p align="center"> 
<img src="https://github.com/elmiomar/OmniaTurrisSetup/blob/master/img/turris-back.jpg">
</p>




## Download and install latest OS imagne
The Turris Omnia router has a reset button on its back panel. When you press and immediately release the reset button the router resets and boots into ordinary Turris.

To enter a specific reset mode, press the button and hold until the LED turns orange. There are different reset modes:
- 1 LED: Standard (re)boot
- 2 LEDs: Rollback to latest snapshot
- 3 LEDs: Rollback to factory reset
- 4 LEDs: Re-flash router from flash drive
- 5 LEDs or more: Boot to rescue shell

>The ones that I used the most (except mode 1 of course) are 3 and 4.

If you want to reset your switch to factory settings, use mode 3. If the switch is broken beyond repair use mode 4.

In mode 4, [download the OS image](https://repo.turris.cz/omnia/medkit/omnia-medkit-latest.tar.gz). Save the file omnia-medkit-latest.tar.gz to USB flash to the root directory and put the USB flash to the front panel USB connector of the Turris Omnia router. The Turris Omnia router supports following filesystems: ext2/3/4, BtrFS, XFS and FAT. After that use reset button to select mode 4 (4 LEDs)
Once the process has completed writing the OS, remove the USB flash and access the switch on IP address 192.168.1.1.

(source [here](https://doc.turris.cz/doc/en/howto/omnia_factory_reset).)

If you enter the switch from mode 4 or for the first time, you will be prompted to a wizard that will help you configure the switch. There are 10 or 11 steps and are very clear. There should be no issue with this step.

Configuration I used:
- **WAN IP**: 10.0.40.26/24
- **Gateway**: 10.0.40.254
- **DNS servers**: 10.0.20.200, 8.8.8.8
- **LAN**: 192.168.1.0/24
- Disabled both WLAN1 and WLAN2 (temporarly)

## Other things to consider

- **Update the packages list**: go to System->Software and update the packages.
- **OpenVSwitch**: find and install all openvswitch packages.

## Experiment environment
<p align="center"> 
<img src="https://github.com/elmiomar/OmniaTurrisSetup/blob/master/img/omnia-pis.jpg">
</p>

### Wired
First thing is to test using wired connections. I have 2 Raspberry Pis that I need to connect to the switch.

>In this part the wireless APs are disabled.

### Wireless

Enable at least one WLAN.
