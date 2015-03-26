# Introduction #

So you wantto build netboot virtual appliances?


# Details #

This is one method that I find is easy to maintain due to the fact that it is a modular build method. Also one of the goals was to make the boot process as fast as possible so I based the booter-image on the most excellent work made of my ex-collegue Daniel Svensson. He strips out the booter image to save bandwidth when distributing it to approx 330 linux servers. He also rocks!

The boot-image is all in all 483 Mb, that is not much to transfer from the boot server to the netbooting client.

The setup of the working netboot server is like follows:

Virtual instance running CentOS 6 and with three services running:

dhcpd

httpd

xinet tftpd

Add the ip-address of the VM as an ip-helper on the subnet you need to netboot macs on. This subnet should already have a dhcp server (most likely the current setup)

Now when a mac boots with N it will call for a dhcp address, and later for a dhcp Inform. The current dhcp server most likely have no idea what the mac wants but the mac will also ask via the ip-helper the VM. This server knows and will send two options:

kernel path: i386/booter

and booter image path: http://theipadressofyourVM/boot/booter.dmg

so in this step the three services are involved in this sequence:

dhcpd

serves out the options required for the mac to get all required info to net boot

tftpd

serves out the kernel and kernelcache files required to boot the hardware

httpd

serves out the booter image required to load a complete OSX graphical user interface

Now when the mac has netbooted on the booter image it will load terminal and curl down the restore.sh script from the VM. This script is a regular bash script and hence is quite easily customized for your needs if you speak bash.

The script as it is shiped will look for a restore image called restore.dmg that will be located at:

http://theipadressofyourVM/restore/restore.dmg

This file will be restored to the first available partition on the local disk.

The last step is to set the boot-volume and then it reboots.

So the build process has the following ideas:

it installs all required rpms to get a working centos server with httpd and tftpd services running out of the box.

It then copies the configure.sh script that later runs as the last step before your appliance is done.

The configure script sets your appliance up with a swedish keyboard layout. You can easly change this to reflect your own layout. The script also sets the hostname to netbooter, change this now or later to reflect the DNS name of your server. It will turn selinux off and create a ifcfg-eth0 file located in the /root directory for reference if you need (and you should) static ip address on the VM. It also adds a local user called imager with password imager that you can use for uploading images to the restore path: /var/www/html/restore/

As a last step it adds a script intended to setup the server with all the local parameters. This script is located at /root/fixit.sh. You will need to change several occurrences of theipaddress to the correct ip-address and this script will fix that for you. It will also copy the tftp files to the CentOS 6 default location from the CentOS 5 location.

The rc.local script will run when the Vm is started for the first time and will do the same copy of tftp files to the new default location, I added in this because I kept forgetting to do this step.

The fixipaddress.sh is basically the same script as fixit.sh. Cleanup is a good idea.

The build process will also add the webmin repo to the default repos for your webmin needs. It is a great web frontend to manage your server, use if you like.

To create the booter image you will need the most current version of the InstallESD.dmg file that you can get from the Install Lion app that you can by from the App-store.

Get the entire directory from the source. It is imperative that you have two directories within the build directory, source and nestedsource.

Make sure you have the InstallESD.dmg in the same directory and run the creator.sh script as root (use sudo)

The script will mount the InstallESD image into the source directory and remove a lot of un-needed stuff. The next step is to mount the BaseSystem.dmg at the nestedsource directory. Now it will copy from the local mac that is running the build some needed binaries as well as the needed code for the restore feature.¨

This is why you need to build the booter image on a machine with the same OS version and preferably the same build as you are using InstallESD.

You can speed this process up significantly by building on a mac with SSD or flash based storage, I used my MacBookAir for this step and it was at least four times as fast as when building on my MacBookPro with a spinning disk.

In the end you will have booter.dmg file that will boot all macs that the InstallESD file supports. And it will be around 480 Mb in size, nice if you have limited bandwidth.

Now the best option would be to package only the booter.dmg and the NBImageInfo.plist into a rpm that you later add to a http repo that you add to the appliance, see how I added the webmin repo in the netbooter.appl file. This way you can handle the booter image separately from the kernel/kernelcache files that gets served using tftp earlier in the boot process.

Lastly do not forget to add the subnet that the VM is located on into the dhcpd.conf file before starting the service. Also add any subnets that you want to restore macs on.