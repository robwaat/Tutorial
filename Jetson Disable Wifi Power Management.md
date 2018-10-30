# Disabling the Wi-Fi Power Management on an Nvidia Jetson TX2
## Introduction
In using the Nvidia Jetson Tx2 for robotics and UAV applications where control and data signals are provided via Wi-Fi, the latency associated with the network is very important.
Extreme cases of latent data can cause the system to act on out of date information, degrading control performance or even causing crashes.
With Wi-Fi power management enabled, as is the default case on most Linux based systems such as the TX2, the network interface hardware periodically enters a low-energy sleep mode, waking up at regular intervals to check for received data waiting to be processed.
This means that data sent to a client can experience a large and variable latency (typically 20 - 120 ms) depending on when it arrives relative to the hardware sleep cycle.
Disabling the Wi-Fi interface power management helps stabilise this effect by preventing the hardware from ever entering low power mode.
This action thereby ensures that the Jetson is always ready to process signals in the lowest time possible.

The process of disabling the Wi-Fi power management on an Nvidia Jetson TX2 module, inserted in an Orbitty Carrier board and running the appropriate Connectech Board Support Package (BSP) is, however, non-trivial.

It is noted that the [standard method](https://askubuntu.com/questions/905273/cant-disable-permanently-power-management-on-wifi) for disabling the Wi-Fi power management in an Ubuntu 16.04 based Linux system does not function as desired in the above scenario. Specifically, there is **no effect** in editing `/etc/NetworkManager/conf.d/default-wifi-powersave-on.conf` to resemble the code below, as would normally solve the issues.
```
[connection]
wifi.powersave = 2
```
Other untested methods are located in another [StackExchange thread](https://askubuntu.com/questions/85214/how-can-i-prevent-iwconfig-power-management-from-being-turned-on) but the method detailed in the [instructions](#Instructions) below is known to perform the job as desired.


## Instructions

### 1. Create a script
This method works by creating a service to run a script that toggles the state of the power management using `sudo iw dev wlan0 set power_saver off`. It was found by experimentation that while the standard method fails, and an alternate using `sudo iwconfig wlan0 power off` encounters issues with the command iwconfig not being properly integrated with the Jetson's Linux for Tegra build, the command above is able to set parameters at the lowest level.

This is performed by switching to the systemd directory
```
cd /etc/systemd
```
Creating a new script file with an appropriate name and opening it for editing using a text editor such as gedit
```
sudo gedit wifi-powersave-off.sh
```
And altering the contents to match the below
```
#!/usr/bin/env bash

iw dev wlan0 set power_save on
iw dev wlan0 set power_save off

```
Before saving the script, closing gedit and granting `wifi-powersave-off.sh` execution permissions with
```
sudo chmod +x wifi-powersave-off.sh
```

The effects of the script can then be tested by calling it with `sudo /etc/systemd/wifi-powersave-off.sh`. A successful execution should see the system ping times stabilise from the perspective of another computer on the network.

It is, however, noted from using the `ping <Jetson-IP-Address` to assess network latency with another computer that the [iw command](http://manpages.ubuntu.com/manpages/trusty/man8/iw.8.html) does not function unless the power management is first turned on, then subsequently turned off. This issue is attributed to the system reporting the state of the Wi-Fi power management doing so incorrectly after retaining the setting before system reboot. Here, the start-up routine automatically turns the power management back on but does not update its recorded state. Thus, running only the `off` command has no effect as the kernel assumes that no changes need to be made.

The shebang line `#!/usr/bin/env bash` is employed automatically to inform the system daemon execution which interpreter to use when running the other commands.


### 2. Define a service to run the script automatically
If the power toggling script is successful, the next step is to automate it to run on boot, preventing the need to manually configure the Wi-Fi on each use of the Jetson. This can be achieved in several ways such as adding a script to `/etc/init.d/` or with a utility such as [cron](https://help.ubuntu.com/community/CronHowto), but the suggested way is to make further use of the systemd start-up manager present in modern Linux distributions.

Here we define a process to run the script on boot, after the appropriate other utilities have loaded, by defining a service using a syntax recognised by the system daemon.

The first step is to change to the directory where the default services are located
```
cd /etc/systemd/system
```
Then to create a new service definition with an appropriate name and open it with gedit, as with the script above
```
sudo gedit wifi-powersave-off.service
```
The contents should then be set to match those below
```
[Unit]
Description=Turn Wifi Power Management Off
After=network.target 
After=network-manager.service
After=network-online.target
After=wpa_supplicant.service
After=multi-user.target
After=graphical.target
After=systemd-networkd-wait-online.service

[Service]
Type=idle
RemainAfterExit=no
ExecStart=/etc/systemd/wifi-powersave-off.sh
Restart=no
RestartSec=20


[Install]
WantedBy=multi-user.target

```
Before saving and exiting gedit.

#### Explanation
Rather than providing an individual script for each start-up daemon's actions like some other methods, systemd works by describing the tasks to be completed in a consistent form and having a single process interpret their ordering and threading based on stated dependencies. It can then provide an optimally parallelised way to execute the required processes on boot.

The definition above contains three main parts; the `[Unit]` which provides a description of the process requirements and sequencing relative to other start-up services; the `[Service]`, which dictates what the daemon actually executes and sets conditions for restarting the process; and `[Install]` which handles the behaviour of the service when it is enabled to start on boot, latching it on to an existing process to control where it is executed in the start-up sequence.

A good resource for more information on the function of the service can be found in [this tutorial](https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files). Further details can be found by consulting the [systemd.unit](http://manpages.ubuntu.com/manpages/cosmic/man5/systemd.unit.5.html) and [systemd.service](http://manpages.ubuntu.com/manpages/cosmic/man5/systemd.service.5.html) man pages.

Key points to note are as follows:
* `After=...` describes the collections of services that must have been run prior to this service for it to function correctly. Those detailed here are selected from a number of sources to ensure that the service starts after the network has been fully configured, thus the power management setting will not be overwritten as part of subsequent boot processes.
* `Type=idle` serves a similar purpose, informing the system daemon that the service should be launched only once all other jobs are dispatched. It is noted that there is much redundancy between these two aspects that could probably be streamlined by more informed dependency configuration. 
* `RemainAfterExit=no` is an explicit declaration of the default behaviour, wherein the service is considered inactive after the process it calls is completed.
* `ExecStart=...` is perhaps the most important line, telling the service what instructions to execute upon starting. Here this links to the earlier wifi-powersave-off.sh script. Note that these services are automatically executed as root and so do not require the use of `sudo` prior to issuing a command.
* `Restart=...` sets the options to restart the service. This is useful if it is found that the power management is being re-enabled and needs to be constantly controlled. As this is not the case here, the restart is disabled.
* `RestartSec=...` defines the time between activations of the service when restart is enabled as above.
* `WantedBy=multi-user.target` determines where the systemctl enabled service should be called during startup by latching it to an existing process or list of services and requirements. The option here is a standard selection used to schedule the service after the majority of boot actions have completed.

### 3. Define a timer to start the service at a given time after boot
The service defined above provides a means to run the earlier script at boot within the systemd framework. However, it was found through experimentation that the process enabling the Wi-Fi power management occurs approximately 20s after reboot.

Therefore, it is necessary to define a timer that will call run the service within systemd at a time after this initial configuration.

The steps to complete define this process are given below.

First, make sure the terminal remains in the correct directory to define the service.
```
cd /etc/systemd/system
```
Then create a new file in gedit with the same root name as the service in Section 2, this time appended by the `.timer` extension.
```
sudo gedit wifi-powersave-off.timer
```
Before modifying the description to resemble the contents below
```
[Unit]
Description=Wait before turning Wifi Power Management Off
#After=network.target 
#After=network-manager.service
#After=network-online.target
#After=wpa_supplicant.service
#After=multi-user.target
#After=graphical.target
#After=systemd-networkd-wait-online.service

[Timer]
OnBootSec=30
#OnUnitActiveSec=10
AccuracySec=1


[Install]
WantedBy=default.target

```
Then saving and closing the gedit file.

#### Explanation
The timer created here has largely the same structure as the service above, describing a program to be executed that defaults to the `.service` file of the same name as the timer.
The difference lies in the acceptable description blocks this file may contain. As before, it includes `[Unit]` and `[Install]` blocks which serve their previous functions, but now contains a `[Timer]` block to dictate execution. Documentation specific to this block can be found in the [systemd.timer](http://manpages.ubuntu.com/manpages/cosmic/man5/systemd.timer.5.html) man pages

The key points to note are as follows:
* The `After=...` declarations are largely unnecessary. The timer itself only depends on the system daemon. Thus, they are commented out.
* `OnBootSec=30` defines the time after boot when the timer will call the service for execution. The units for this default to seconds but can be configured to other units of time using appropriate suffixes given in the documentation.
* `OnUnitActiveSec=10` provides the option to retrigger the service 10 seconds after it previously finished processing. When compared to the use of `Restart=` and `RestartSec=` in the `[Service]` block, this represents a more elegant means of recalling the service should it be necessary to repeatedly control the power management state by considering the execution time of the script and ensuring multiple instances are not launched simultaneously. This is however unnecessary here and more suited to services with a longer execution time.
* `AccuracySec=1` defines the acceptable range of time after in which the system should run the timer's linked service after the timer is triggered. It defaults to 60 seconds. This line ensures the service is run within 1 second of being triggered, reducing the variability of execution times after start-up to aid usability.
* `WantedBy=default.tagret` again links the timer to a process on called on boot to ensure its execution once the service is enabled. The `default.target` ensures that the timer is activated after reboots and not just full shutdowns, it being found the `multi-user.target` has issues here. 

### 4. Implement the solution
The final stage in the implementation of this process is to inform the system daemon that it should run the newly defined service and timer definitions as part of its standard boot activities.

This is performed by use of the `systemctl` command as follows.

First, it can be useful to check that the service works as desired. This is performed by running it manually using the instructions below and observing any change in network ping times from an external computer.
```
sudo systemctl start wifi-powersave-off.service
```
If for any reason this should fail, debugging information logged by the system daemon can be found with the `journalctl` command. Useful approaches were found to be viewing entries specifically related to the service under test, with the explanation flag enabled
```
sudo journalctl -x -u wifi-powersave-off.service
```
Higher-level status reports are also useful for a quick check that the service was in fact executed.
```
sudo systemctl status wifi-powersave-off.service
```
The service can then be enabled to run on boot with
```
sudo systemctl enable wifi-powersave-off.service
```

Finally, the timer definition can be tested with similar calls to systemctl. A delayed change in ping times indicates it functions correctly.
```
sudo systemctl start wifi-powersave-off.timer
```
The functioning timer can be scheduled to run on boot with
```
sudo systemctl enable wifi-powersave-off.timer
``` 

It is worth noting that these instructions will leave both the service and timer enabled to start on boot. The script will thus be run immediately after the network dependencies are met and again 30 seconds later when the timer triggers. This is found to provide the most reliable means of disabling the Wi-Fi power management.

This concludes the steps taken to disable Wi-Fi power management on an Nvidia Jetson.
