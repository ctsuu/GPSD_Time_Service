# GPSD_Time_Service
GPSD Time Service with Garmin GPS19x

## Introduction

This project is to using GPSD, NTP and Garmin GPS 19x HVS to setup a high-quality NTP time server for on going autonomous driving projects. The following tasks are in certain sequences.
1.  Install the GPSD 3.22, NTP or Chronyd on the target system.
2.  Connect a GPS receiver that supports PPS(one pulse per second) to serial or USB port.
3.  Config the GPSD daemon.
4.  Config the Chronyd daemon.
5.  Create GPS_startup script.
6.  Start the GPS time server from booting the computer.
7.  Summary, future work

The NTP service is designed to solve the Latency, Jitter, Wobble and Accuracy problems for the time services. Latency is delay from a time measurement until a report on it arrivs where it is needed. Jitter is short-term variation in latency. Wobble is a jitter-like variation that is long compareed to typical measurement periods. Accuracy is the traceable offset from 'true' time as defined by a national standard institute.

## Install GPSD, NTP, Chronyd

In order to make high-quality time available for the sub-systems on my network, I am going to use GPS receiver as the reference clocks to build the system.  


## Reference
~https://photobyte.org/raspberry-pi-stretch-gps-dongle-as-a-time-source-with-chrony-timedatectl/

~https://gpsd.gitlab.io/gpsd/gpsd-time-service-howto.html

~https://medium.com/tech-guides/run-a-script-on-ubuntu-18-at-startup-7d8b650728e5

~https://mythopoeic.org/pi-ntp/
