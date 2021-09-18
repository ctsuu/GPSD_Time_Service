# GPSD_Time_Service
GPSD Time Service with Garmin GPS19x

## Introduction

This project is to using GPSD, NTP and Garmin GPS 19x HVS to setup a high-quality NTP time server for on going autonomous driving projects. The following tasks are in certain sequences.
1.  Connect a GPS receiver that supports PPS(one pulse per second) to serial or USB port.
3.  Install the GPSD 3.22, NTP or Chronyd on the target system.
4.  Config the GPSD daemon.
5.  Config the Chronyd daemon.
6.  Create GPS_startup script.
7.  Start the GPS time server from booting the computer.
8.  Summary, future work

The NTP service is designed to solve the Latency, Jitter, Wobble and Accuracy problems for the time services. Latency is delay from a time measurement until a report on it arrivs where it is needed. Jitter is short-term variation in latency. Wobble is a jitter-like variation that is long compareed to typical measurement periods. Accuracy is the traceable offset from 'true' time as defined by a national standard institute.

## Install GPSD, NTP, Chronyd

In order to make high-quality time available for the sub-systems on my network, I am going to use GPS receiver as the reference clocks to build the system. GPSD use the 1 PPS pulse delivered by Garmin GPS 19x receiver to discipline (correct) a local NTP instance. The concept is just by timestamping the arrival of the first character in the first character in the report on each fix and correcting for a relatively small fix latency composed of fix-processing and RS232 transmission time. I use the Rs-232 control line (the Carrier Detect) to ship the 1PPS edge of second to the host system. Satellite top-of-second loses some accuracy on the way down due mainly to variable delays in the ionosphere; processing overhead in the GPS receiver itself adds a bit more latency, and the local host detecting that pulse adds more latency and jitter. But it's still often accurate to on the order of 1uSec. . 

## Install GPSD, NTP, Chronyd
Now, install the following packages for GPS daemon and ntp or alternative ntp server:
~~~
sudo apt-get update
sudo apt-get -y install gpsd gpsd-clients python-gps chrony ntp
~~~
## Config the gps daemon
~~~
sudo nano /etc/default/gpsd
~~~
In the file that opens, add or amend lines to make sure the following is present:
~~~
START_DAEMON=”true”

# Automatically hot add/remove USB GPS devices via gpsdctl
USBAUTO=”true” # if you use Serial to USB adapter

#They need to be read/writeable, either by user gpsd or the group dialout
#DEVICES=”/dev/ttyACM0″
DEVICES=”/dev/ttyUSB0″ # Using Serial to UBS adapter
GPS_BAUD=38400

# don't wait for client connects to poll GPS
GPSD_OPTIONS=”-n”  
~~~
Hit ctl-x followed by y to close and save the file.

Reboot the system and check that the following services are active:
~~~
sudo systemctl is-active gpsd

sudo systemctl is-active chronyd
~~~

Use the following client to check the gps status:
~~~
cgps -s
~~~
PPS is live. Time offset is calculated around 1.05 secound. 
~~~
gpsmon -n
~~~

## Make a change to the chrony configuration file:
If you are connected to the network, you will see a list of available time servers plus the GPS source which will be shown as NMEA. If you’re not network connected, you will just see the NMEA source listed. The two punctuation characters immediately before NMEA, indicate its status and you need to see #* where # means thge GPS is recognised as a local clock and * means that it’s being used to synchronise the Pi system time.
~~~
sudo nano /etc/chrony/chrony.conf
~~~


## Reference
~https://photobyte.org/raspberry-pi-stretch-gps-dongle-as-a-time-source-with-chrony-timedatectl/

~https://gpsd.gitlab.io/gpsd/gpsd-time-service-howto.html

~https://medium.com/tech-guides/run-a-script-on-ubuntu-18-at-startup-7d8b650728e5

~https://mythopoeic.org/pi-ntp/
