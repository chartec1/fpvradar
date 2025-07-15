# FPV Radar


Quick note! This was originally made by lexfp. I recreated some parts of this github page to keep updating it, and also making the page easier to understand for beginners.


# Example product

![FpvRadar_complete](https://github.com/user-attachments/assets/4bbb4f72-c5fe-47e5-9ea3-b33ea7693e42)


https://youtu.be/YkmsAgEEuzo  

https://youtu.be/ppq6NCjMSJI 

Monitors the nearby airspace for low flying aircraft.  

To clarify, this is a standalone device and does not require wifi/and or data plan to work. Check your country for ADSB requirements. 

# License
The code herein written by the Author is released under the terms of the unlicense. https://unlicense.org/

# Required components:

1) Raspberry pi (zero w used in this guide) https://www.banggood.com/custlink/mvDhZjVbBW (don't buy it at banggood unless you have to since it's expensive there)
2) GPS module (e.g. Beitian 220) https://www.banggood.com/custlink/mGmRZjPN0m
3) Buzzer https://www.banggood.com/custlink/GGKdSoHb0F
4) SDR Dongle (e.g. flightaware, rtl-sdr) https://flightaware.com/adsb/prostick/ 
5) micro usb to USB adapter or HUB (if using pi zero)
6) 5v voltage regulator for external battery power (if using external battery that is not 5v)
7) Antenna - https://www.amazon.com/LM-YN-Omnidirectional-Compatible-Frequency/dp/B01N6GO584/ref=sr_1_4?crid=HYF59H6O5LOO&dib=eyJ2IjoiMSJ9.s_OeoFvC0c7s8LYIGZMQzSGbPPw4DRyqC0XvM45m3BL1FxxWSo5734lvONCCUZJyVfJITSi3Zv49yoLBItw087RS3mbxN7GgDpcIGzu6IGIgkeDIFQ97fcVnGhg2YUnyzK9n1qF069a_2p_P5zURzaMaT7BWIKFClOyYJKsC7ka43DIfqauYItgoeOONqLVqDwkfzh7Q6mUnoHllhc6zKK-FuW-GciHnHDNdwmSvHPBDNpYaXF3cBHF2a-Ps-RtFQltqCh9ykg209CotK_L0aOhtk_or7na6KRyxZvK9jCI.b4eL4aHk0AiuTjCEKTL8NiH7Krg4AiuQoMSsVQAgRV0&dib_tag=se&keywords=1900mhz+antenna&qid=1752536658&s=electronics&sprefix=1900mhz+anten%2Celectronics%2C144&sr=1-4
8) Case (Anything would work. Just something to keep everything toghether while also protecting the pi) - https://www.thingiverse.com/thing:1886598 I snipped away ap portion of it to glue in the gps. Or https://www.thingiverse.com/thing:4756697?fbclid=IwAR1rNRHUoMtD9yErSb5yf0OHlEy4OJyrLd6rC7ygGXJgEToh9D8qGnDaD9E 

# Installation

1) Install piaware (version 4 as of this build) https://flightaware.com/adsb/piaware/build  

2) Follow the optional instructions for enabling wifi (modify piaware-config.txt to put in wifi info)  

3) Enable ssh by creating an empty file on the /boot partition of the SD card with the filename of "ssh"  

4) Enable on/off button - While optional, this is highly recommended so that you don't screw up the file system since you won't be able to ssh into the pie to shut down out in the field.  

If you use Raspbian stretch 2017.08.16 or newer, all that is required is to add a line to /boot/config.txt (and reboot for this to take effect):  
dtoverlay=gpio-shutdown,gpio_pin=3  
see https://www.stderr.nl/Blog/Hardware/RaspberryPi/PowerButton.html  
You will also need to wire a momentary switch/button between GPIO3(pin 5) & Ground. Pressing it once will shutdown the system (wait until all the lights die before unplugging) and pressing it again will start up the system.

5) Buzzer (optional) - wire positive to GPIO17 and black to any ground. 

6) Install the following libraries (script was tested against python 2.7.16):  

sudo apt-get update  
sudo apt-get install python-requests  
sudo apt install python-gpiozero  
sudo apt install python-geopy  
sudo apt install git  

7) Set up GPS:
For reference, you can use the following site (but follow my instructions instead since they differ a bit):    
https://maker.pro/raspberry-pi/tutorial/how-to-use-a-gps-receiver-with-raspberry-pi-4  
You can use various GPS modules, but I have chosen to use the Beitian 220. For wiring, you can use any of the 5v and grounds. I chose pins 4 & 6. The GPS rx will go to the tx on the pi and vice versa. In the case of the beitian 220, the green wire will go to pin 8 (GPIO14) and the white wire will go to pin 10 (GPIO15).

<img width="709" height="554" alt="gps_wiring" src="https://github.com/user-attachments/assets/ae187273-f502-49be-b72d-44e982c2edc5" />


Next, you should ssh into your pi and type "sudo raspi-config". Select Interfacing options and then Serial. Enable the serial interface while keeping the login shell disabled.  

Next install the gps software: "sudo apt-get install gpsd gpsd-clients"  
You can test if it works by "cat /dev/serial0". If you see no output, then chances are you may have to switch the tx/rx wires on the GPS (white and green).  

Next add the pi user to the dialout group: "sudo adduser pi dialout"  

Lastly, in order to have this take effect on startup, "sudo nano /etc/default/gpsd"  
Comment out the DEVICES and GPSD_OPTIONS lines and add these lines to the end of the file:  
GPSD_OPTIONS="/dev/serial0"  
GPSD_SOCKET="/var/run/gpsd.sock"  

After rebooting, you can test if everything works with either "sudo gpsmon" or "sudo cgps -s"  


8) Clone this repo into the home directory.   
Make sure you're in the home directory /home/pi and type "git clone https://github.com/lexfp/fpvradar.git"  
After the command runs, you should have a fpvradar directory with all the files inside.
Make sure the file /home/pi/fpvradar/fpvradar.py exists.   You can type "ls /home/pi/fpvradar/fpvradar.py"  

At this time you should change the code to your liking. Type "nano /home/pi/fpvradar/fpvradar.py" and change the values for the different perimeter alarms along with the altitude at which you want to monitor (INNER_PERIMETER_ALARM_MILES, ALTITUDE_ALARM_FEET, etc...). If you want to play with the other options/settings, be sure to test them as I haven't done much testing other than the defaults. Once you finish editing, you can hit ctrl-x to exit and it will ask you if you want to save first. Just answer yes.

9) Turn fpvradar into a service so it automatically starts when the pi is powered:

/lib/systemd/system/fpvradar.service (move included fpvradar.service file to /lib/systemd/system)    
sudo systemctl daemon-reload  
sudo systemctl enable fpvradar.service    

If you need to check if it is running after you reboot, you can use the status command:  

sudo systemctl status fpvradar.service  

If things appear running but they still don't work, then move on to the next steps and check the logs.  

10) Persistent LOGS (Optional)  
set your time zone (https://en.wikipedia.org/wiki/List_of_tz_database_time_zones):  

sudo timedatectl set-timezone America/New_York  
sudo mkdir -p /var/log/journal
sudo nano /etc/systemd/journald.conf

change/add the following:  

Storage=persistent

To see the logs you can use the following commands (you can also use -2,-3 to go back even further):  

sudo journalctl -b 0 -u fpvradar.service (current boot logs)  
sudo journalctl -b -1 -u fpvradar.service (previous boot logs) 

11) Power  
You can power your pi from the regular USB port or an external battery. If using an external battery, you'll need to use a 5v regulator. There are a few ways to wire up the PI to be powered from a battery. You can google the different options.

12) Screen (optional)  
Since your flying fpv, and your visulizer is focused on the drone. A screen wouldn't be ideal for some use cases. Should you want to add a screen, there is an easy solution. If you carry a cell phone, simply create a hotspot (On your phone) with the same wifi name/password as you used at your home router. While you're out in the field, it will not find your home router, but will connect to your phone instead. Once it connects to your phone, you will need to find the IP address of the Pi. If your phone doesn't let you see it by default, you'll need to install an hotspot manager app such as https://play.google.com/store/apps/details?id=com.catchy.tools.mobilehotspot.dp&hl=en_US&gl=US which will show it to you. Once you find the ip, simply enter it in your browser on the phone and it will show you a map with surrounding planes (default piaware screen). 

## Troubleshooting  
Most of the issues I've experienced are from the GPS not getting signal while testing indoors. You will also need to double check your wiring to make sure everything is hooked up correctly. Try writing code to test each component separately (buzzer, gps).

## Future  
If someone wants to design a better 3d printed case for this, I would be happy to link to it.  

## FAQ: 

How good is the range?  
Depends on your antenna. I use the pro stick which has an internal filter. Coupled with my home made antenna, I can often see aircraft in other states over 100 miles away. This is with the antenna sitting on my desk inside my house. Since you'll be using this outside, I can't imagine that you would have any issues unless you used a really poor antenna.

Can I use other Pis?  
Yes, you should be able to use almost any. My initial prototype (these pics are my 2nd prototype) was a Pi 3 model B from 2016. Unfortunately, I burned that one up by wiring up a voltage regulator to it without checking the inputs/outputs. Since I knew the concept worked, I wanted to build the second one to be as small/compact as possible.  
 

What is the approximate cost?  
For the setup I have, the cost of the components are around $50. This doesn't include the antenna which I made myself.  

How much testing has gone into this?  
Almost none (except for my own). Seriously, I haven't tested a lot of the conditions in there since they are useless to me, so some things may not work. Please test everything yourself and let me know if you find any bugs. 

Are there alternate cases?  
Here's a design by Colby Terry: https://www.thingiverse.com/thing:4756697?fbclid=IwAR1rNRHUoMtD9yErSb5yf0OHlEy4OJyrLd6rC7ygGXJgEToh9D8qGnDaD9E 
