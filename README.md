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

## Connect the GPS to the Raspberry 4 Board

Pin connections:

    GPS PPS to RPi pin 12 (GPIO 18)
    GPS VIN to RPi pin 2 or 4
    GPS GND to RPi pin 6
    GPS RX to RPi pin 8
    GPS TX to RPi pin 10

## Install GPSD, NTP, Chronyd

In order to make high-quality time available for the sub-systems on my network, I am going to use GPS receiver as the reference clocks to build the system. GPSD use the 1 PPS pulse delivered by Garmin GPS 19x receiver to discipline (correct) a local NTP instance. The concept is just by timestamping the arrival of the first character in the first character in the report on each fix and correcting for a relatively small fix latency composed of fix-processing and RS232 transmission time. I use the Rs-232 control line (the Carrier Detect) to ship the 1PPS edge of second to the host system. Satellite top-of-second loses some accuracy on the way down due mainly to variable delays in the ionosphere; processing overhead in the GPS receiver itself adds a bit more latency, and the local host detecting that pulse adds more latency and jitter. But it's still often accurate to on the order of 1uSec. . 
~~~
$ ls -l /dev/ttyU*
crw-rw---- 1 root dialout 188, 0 Jun 12 13:28 /dev/ttyUSB0
$ sudo usermod -a -G dialout [user]
$ sudo chmod 777 /dev/ttyUSB0
to add 'rw_' for all users.

~~~
To check serial port signal, 
~~~
sudo stty -F /dev/ttyUSB0 38400 
sudo cat /dev/ttyUSB0
~~~

## Install GPSD, NTP, Chronyd
Now, install the following packages for GPS daemon and ntp or alternative ntp server:
~~~
sudo apt-get update
sudo apt-get -y install gpsd gpsd-clients chrony
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

Add the following lines at the end of the file
~~~
allow

refclock SHM 0 refid GPS precision 1e-2 offset 0.9999 delay 0.2
refclock SOCK /run/chrony.ttyUBS0.sock refid PPS precision 1e-3 offset 0.9999
~~~
If you start the system without network connection, the chrony will poll the GPS time in few minutes as soon as GPS get a good 3D fix. The time offset will show around 1 second with 30uSec variances. 

If your network is back, the time offset may slowly move to the true time. 

If you lost the GPS fix, the system time will keep going on it is own. 

If you lost the internet, the system time will walk back slowly to the time offset around 1 second. 

For best result, the system is better run without internet connection at all. 

Check the NTP status:
~~~
sudo chronyc sources -v

sudo chronyc tracking

sudo chronyc makestep

sudo timedatectl
~~~

## Create GPS_startup script

Goto the the folder /etc/systemd/system, create a new file with gps.service, copy & paste the following line into the file. 
~~~
[Unit]
Description=My custom startup script

[Service]
ExecStart=/home/rp4/gps_startup.sh start

[Install]
WantedBy=multi-user.target
~~~
Hit ctl-x followed by y to close and save the file.

Then, add the service at start-up:
~~~
sudo systemctl enable gps.service
~~~
Start or Stop the gps.service as needed. 
~~~
sudo systemctl start gps.service
sudo systemctl stop gps.service
~~~

## Start the GPS time server from booting the computer
Now we can edit the gps_startup.sh script. 
```
sudo nano /home/rp4/gps_startup.sh
```
Add the following lines into the file. 
```
sudo su-
killall -9 gpsd chronyd
chronyd -f /etc/chrony/chrony.conf
sleep 2

gpsd -n /dev/ttyUBS0
sleep 2
```

Reboot the system, wait 5 minutes to see the effect. Once the GPS device get 3D fix, the system clock will be synchronized to the gps time.
```
Sudo timedatectl
```

## Performance tuning
At this point, the chrony time offset 0.9999 is a place holder. let chronyd run for at least 4 hours and observe the offset reported in the chronyc sourcestats output. 
~~~
Sudo chronyc sourcestats
~~~

In this case (Garmin 19x) the offset specified in the config for the GPS source should be around 0.083 Second.


## Reference
~https://photobyte.org/raspberry-pi-stretch-gps-dongle-as-a-time-source-with-chrony-timedatectl/

~https://gpsd.gitlab.io/gpsd/gpsd-time-service-howto.html

~https://medium.com/tech-guides/run-a-script-on-ubuntu-18-at-startup-7d8b650728e5

~https://mythopoeic.org/pi-ntp/

https://weberblog.net/ntp-server-via-gps-on-a-raspberry-pi/

https://austinsnerdythings.com/2021/04/19/microsecond-accurate-ntp-with-a-raspberry-pi-and-pps-gps/

