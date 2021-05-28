Change every instance of YourUserName to your username. 

Everything in quote format is run within the terminal, everything in code format has to be entered into the file through nano commands.

Find the IP address of the server:

> ip addr show


Update your server software to the newest version and reboot:

> sudo apt update

> sudo apt upgrade

> sudo reboot now

Install python and set up a virtual environment

> cd ~
> 
> sudo apt install python-pip python-dev python-setuptools python-virtualenv git libyaml-dev build-essential
> 
> mkdir OctoPrint && cd OctoPrint
> 
> virtualenv venv
> 
> source venv/bin/activate
> 
> pip install pip --upgrade
> 
> pip install https://get.octoprint.org/latest
> 
> sudo usermod -a -G tty YourUserName
> 
> sudo usermod -a -G dialout YourUserName
> 
> ~/OctoPrint/venv/bin/octoprint serve

Now test to make sure you can get to octoprint on the server, use your IP in a browser:
**http://< your lan ip >:5000**
## Automatic start up
Download the init script files from OctoPrint's repository, move them to their respective folders and make the init script executable:

> wget https://github.com/OctoPrint/OctoPrint/raw/master/scripts/octoprint.service && sudo mv octoprint.service /etc/systemd/system/octoprint.service
> 
> nano /etc/systemd/system/octoprint.service

Adjust the paths to your octoprint binary in /etc/systemd/system/octoprint.service. If you set it up in a virtualenv as described above make sure your /etc/systemd/system/octoprint.service looks like this:
```
ExecStart=/home/YourUserName/OctoPrint/venv/bin/octoprint
```
Then add the script to autostart using:

> sudo systemctl enable octoprint.service

This will also allow you to start/stop/restart the OctoPrint daemon via:

> sudo service octoprint {start|stop|restart}

## Add Sudoer Privileges to Your User

Add your users to sudoers file so it can run shutdown commands:

> sudo nano /etc/sudoers.d/octoprint-shutdown

Enter the following and save.

`YourUserName ALL=NOPASSWD: /sbin/shutdown`

Update octoprint service as well:

> sudo nano /etc/sudoers.d/octoprint-service

Enter the following and save.

`YourUserName ALL=NOPASSWD: /usr/sbin/service`

## Install Webcam Support

Now we install webcam support:
> cd ~
> 
> sudo apt install subversion libjpeg8-dev imagemagick ffmpeg libv4l-dev cmake
> 
> git clone https://github.com/jacksonliam/mjpg-streamer.git
> 
> cd mjpg-streamer/mjpg-streamer-experimental
> 
> export LD_LIBRARY_PATH=.
> 
> make

Test mjpg streamer:

> sudo ./mjpg_streamer -i "./input_uvc.so" -o "./output_http.so"

## Create the webcam startup scripts

Create webcam startup scripts don’t use sudo:
> cd ~
>
> mkdir scripts
> 
> nano /home/YourUserName/scripts/webcam

```
#!/bin/bash
# Start / stop / Restart streamer daemon

case "$1" in
    start)
        /home/YourUserName/scripts/webcamDaemon >/dev/null 2>&1 &
        echo "$0: started"
        ;;
    stop)
        pkill -x webcamDaemon
        pkill -x mjpg_streamer
        echo "$0: stopped"
        ;;
    restart)
        pkill -x webcamDaemon
        pkill -x mjpg_streamer
        echo "$0: stopped"
        /home/YourUserName/scripts/webcamDaemon >/dev/null 2>&1 &
        echo "$0: started"
        ;;
    *)
        echo "Usage: $0 {start|stop}" >&2
        ;;
esac
```

Create webcam Daemon script don’t use sudo:

> nano /home/YourUserName/scripts/webcamDaemon

Note that I have changed camera_usb_options setting from Chris' document. The default is: **camera_usb_options="-r 640x480 -f 10"**

```
#!/bin/bash

MJPGSTREAMER_HOME=/home/YourUserName/mjpg-streamer/mjpg-streamer-experimental
MJPGSTREAMER_INPUT_USB="input_uvc.so"
MJPGSTREAMER_INPUT_RASPICAM="input_raspicam.so"

# init configuration
camera="auto"
camera_usb_options="-r 1280x720 -f 20 -wb 5000 -ex 300”
camera_raspi_options="-fps 10"

if [ -e "/boot/octopi.txt" ]; then
    source "/boot/octopi.txt"
fi

# runs MJPG Streamer, using the provided input plugin + configuration
function runMjpgStreamer {
    input=$1
    pushd $MJPGSTREAMER_HOME
    echo Running ./mjpg_streamer -o "output_http.so -w ./www" -i "$input"
    LD_LIBRARY_PATH=. ./mjpg_streamer -o "output_http.so -w ./www" -i "$input"
    popd
}

# starts up the RasPiCam
function startRaspi {
    logger "Starting Raspberry Pi camera"
    runMjpgStreamer "$MJPGSTREAMER_INPUT_RASPICAM $camera_raspi_options"
}

# starts up the USB webcam
function startUsb {
    logger "Starting USB webcam"
    runMjpgStreamer "$MJPGSTREAMER_INPUT_USB $camera_usb_options"
}

# we need this to prevent the later calls to vcgencmd from blocking
# I have no idea why, but that's how it is...
vcgencmd version

# echo configuration
echo camera: $camera
echo usb options: $camera_usb_options
echo raspi options: $camera_raspi_options

# keep mjpg streamer running if some camera is attached
while true; do
    if [ -e "/dev/video0" ] && { [ "$camera" = "auto" ] || [ "$camera" = "usb" ] ; }; then
        startUsb
    elif [ "`vcgencmd get_camera`" = "supported=1 detected=1" ] && { [ "$camera" = "auto" ] || [ "$camera" = "raspi" ] ; }; then
        startRaspi
    fi

    sleep 120
done
```

Edit webcam file startup permissions:

>sudo chmod +x /home/YourUserName/scripts/webcam
>
>sudo chmod +x /home/YourUserName/scripts/webcamDaemon

Add webcams to startup:

> sudo nano /etc/rc.local

```
#!/bin/sh -e
#
# rc.local
#
# This script is executed at the end of each multiuser runlevel.
# Make sure that the script will "" on success or any other
# value on error.
#
# In order to enable or disable this script just change the execution
# bits.
#
# By default this script does nothing.

/home/YourUserName/scripts/webcam start

exit 0
```

Start webcam:

> sudo /home/YourUserName/scripts/webcam start

Change rc.local permissions:

> sudo chmod +x /etc/rc.local

## Finishing Up

Now reboot:
> sudo reboot now

Add the following shutdown commands within Octoprint:

**Restart OctoPrint: sudo service octoprint restart**

**Restart system: sudo shutdown -r now**

**Shutdown system: sudo shutdown -h now**

##

The webcam links are as follows:

Stream URL: **/webcam/?action=stream**

Snapshot URL: **http://YourIPAddress:8080/?action=snapshot**

Path to FFMPEG: **/usr/bin/ffmpeg**
