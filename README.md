# Thank you mitchese. You made all this possible! 
The following code is running in a System with 2 SMA Tripower, 1 SMA Storage Boy 3.6 and a SMA Home Manager 2.0. In this system I added a Victron Multi Plus 2 GX and lifepo4 batteries. ESS is installed and grid feed in is disabled. This is Storage Boys job. The Multiplus has to run my server 24/7 and use only the large lifepo batteries. They will be recharged if there is overproduction or the SoC reachs 0%.


# Victron Faker
This small program emulates the ET340 Energy Meter in a Victron ESS System. It reads
values from an existing SMA Home Manager 2.0, and publishes the result on dbus as
if it were the ET340 meter.

Use this at your own risk, I have no association with Victron or SMA and am providing
this for anyone who already has these components and wants to play around with this.

I use this privately, and it works in Europe, your results may vary.

# Prepare you environment
In case you want to compile the tool by yourself or need some changes later on, you need to install the go Software. Follow the guides on their page according your operation system. 
Otherwise you can use my precompiled .arm file. It is the one without extention. 

Setup your GX device and enable SSH. In my case I enabled the ESS too.

In order to get your arm file for Victron Multiplus II GX you need to compile the file. I have done this on a Windows machine. Follow these steps in order to get your armv7 file.

    Download and install go environment
    Check your env GOPATH= (go env) and take note of the path. Set GOOS=linux GOARCH=arm
    Download the files form mitchese
    In the file is a reference to https://github.com/dmichael/go-multicast 4.1 Download this repo too and copy it in your GOPATH /src folder. --> C:\Users"yourusername"\go\src\go-multicast-master
    You now have to adjust the import folder for go-multicast-master. In my case the GOPATH=C:\Users"yourusername"\go and I had to modify C:\Users"yourusername"\go\src\go-multicast-master\main.go file. In import section (line 13) i corrected the path as follow: "go-multicast-master/multicast".
    You now modify the sma_home_manager_printer part the same way. In my case again "go-multicast-master/multicast".
    Finally you can build your file.Open a command promt and move to the downloaded and previously modified file sma_home_manager_printer.go. Compile it with: go build -o sma_home_manager_printer.armv7

Now you need to prepare your Multiplus:

    Switch on the Multiplus and connect the ethernet port to a router or a network switch. I had a switch only.
    the IP address wil be printed on the screen of the Multiplus. Remember it runs his own DHCP Server.
    Open a browser window and insert the IP Address of the Multiplus.
    You will land on the mainpage. Use your keybaord cursors to navigate. Settings --> General --> Set root passwor. Do so and choos a strong password. Add it to your password manager. If you don't have it, its time to organize one.
    Set the Access Level to User and installer, the password is ZZZ
    Highlight Access Level (don't open the select page!)
    Press and hold the right curser of your keyboard until you see the Access Level change to Superuser.
    Go to Settings → General → Set root password. And create a root password. Note that, for firmware version v2.00 and later, the root password will be reset by a firmware update. The reason is that the passwd file is on the rootfs, which is fully replaced by an update.
    Finally you activate SSH.
    Install first, or if you have it already, open putty and connect to your Multiplus through SSH. You also can use MobaXterm is you like to have more copy paste style.

https://www.victronenergy.com/live/ccgx:root_access

# Setup
1: Test
First ensure that this will work: Try out the ready compiled .arm https://github.com/Schnema1/shm-et340/blob/master/sma_home_manager_printer 
which will run on your Victron GX device and try to connect to the SMA meter. The above test
program does _not_ publish its result on dbus for use by victron, only prints out the result
for your verification. It should be relatively safe to test with.

If the `sma_home_manager_printer` works and shows consistent/reliable result, then you can
install the shm.et340 in the same way.

While this is running, you should see correct values for a grid meter in your Venus UI:

![Venus GX UI](img/meter_sample.gif)

On the console of your GX device, you should see regular updates, around once per second:
```
root@victronvenusgx:~# ./shm-et340
INFO[0000] Successfully connected to dbus and registered as a meter... Commencing reading of the SMA meter
INFO[0000] Meter update received: 6677.15 kWh bought and 3200.45 kWh sold, 681.3 W currently flowing
INFO[0001] Meter update received: 6677.15 kWh bought and 3200.45 kWh sold, 694.1 W currently flowing
INFO[0002] Meter update received: 6677.15 kWh bought and 3200.45 kWh sold, 686.3 W currently flowing
```

If this does not work, try to `export LOG_LEVEL="debug"` first, which should print out significantly more
information on what's happening.

For more details, see the thread on the Victron Energy community forums here:

https://community.victronenergy.com/questions/49293/alternative-to-et340-mqtt-sma-home-manager.html

# Run at boot (not working yet)
The script only works as long as you computer is connected through SSH. This is not what I wanted and probably not what you want. We need to create an config file in order to get the shm-et340 run automaticly at system boot. 

create a file in /data/   called rc.local   -> /data/rc.local
make it executable chmod +x and add the following two lines of code. Just as mitchese did. Remember to change the file name if your one is not called shm-et340

!#/bin/bash
sleep 15
setsid /data/home/root/shm-et340 > /dev/null 2>/dev/null &

After you have saved it, you type reboot and take a look at automatic running energymeter.
