# Rsensor - sensor node
**We are totally in Work In Progress  mode ;)**
# Parts
* Raspberry Pi Zero W
* DS3231
# Wiring
# Installation
## Sensor node
### Install OS
I will not cover in details the installation of Raspbian on the Pi board.
In short: 
- Download RASPBIAN STRETCH LITE version from https://www.raspberrypi.org/downloads/raspbian/
- Perpare the SD card following official guide: https://www.raspberrypi.org/documentation/installation/installing-images/
- Configure networking (wifi or ethernet)
- Do whatever you want (setup own user, install some packages)
### Install prerequisites
#### Setup your Pi
Enter configuration tool by running as root:
```
root@raspberrypi:~# raspi-config
```
Now do the following:
1. **Interfacing Options**
* Enable SSH to get remote login via SSH.
* Enable I2C, on this bus we have the BME280 sensor and the DS3231 clock.
2. **Serial**
* Disable Serial Login Shell, our software will use the serial line instead.
* Enable Serial Port, we have the PMS5003 sensor attached to it.
3. **Localisation Options**
* Change Locale, choose the locales you nedd (e.g. en_US.UTF-8).
* Change Timezone, set the proper timezone. Data timestamps will be stored in UTC time.
* Change Wi-fi Country, to use the proper frequencies, etc.
4. **Advanced Options**
* Memory Split: 16, we use only the text interface of the GPU.

Now **reboot** your Pi.
#### Setup hardware clock (DS3231)
First, let's install some basic tools
```
root@raspberrypi:~# apt -y install vim i2c-tools ntpdate python-smbus
```
Load the kernel driver for hw_clock and let's see if it finds the device.
```
root@raspberrypi:~# modprobe i2c-dev
root@raspberrypi:~# i2cdetect -y 1
```
You should see DS3231 device at code 68 like the following:
```
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:          -- -- -- -- -- -- -- -- -- -- -- -- -- 
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
60: -- -- -- -- -- -- -- -- UU -- -- -- -- -- -- -- 
70: -- -- -- -- -- -- -- --                         
```
If you do not have it listed, please re-check wiring and start again with setting up your Pi.
The last step is to enable autoloading the modules and initiating the clock at boot.
In the **/etc/rc.local** add before "exit 0 the following line so the file looks like:
```
echo ds1307 0x68 > /sys/class/i2c-adapter/i2c-1/new_device
exit 0
```
In the **/etc/modules** make sure you have those 2 lines:
```
i2c-dev
rtc-ds1307
```
Now reboot your Pi, when it's up connect to it and check if everything is working by issuing:
```
root@raspberrypi:~# hwclock --show
```
Above command should return date and time.
The last step is to setup NTP client to keep our Pi always in sync with the NTP time servers.
```
root@raspberrypi:~# systemctl start ntp.service
root@raspberrypi:~# systemctl enable ntp.service
```
We are done with this part ;)

# Credits:
Based on project:
https://www.rigacci.org/wiki/doku.php/doc/appunti/hardware/raspberrypi_air