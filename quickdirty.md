# Introduction #

The Q&D workflow


# Details #

create the booter image of your OS choice

modify the NBImageInfo.plist

package the booter rpm using your rpm package method of choice

package the restorescript rpm using your rpm package method of choice

create a repo that can be local but better is to have it be serverd out on your network for easy update of the

booter image on the netboot server

build the appliance using boxgrinder

move it to your hypervisor (supports vmware, hyper-v and kvm)

boot the appliance.

set the static ip address

change the hostname to reflect the DNS name

add your subnet that will be served at the end of the dhcpd.conf file:

subnet 192.168.56.0 netmask 255.255.255.0 {}

start the dhcpd service:

service dhcpd start

make this permanent

chkconfig dhcpd on

ad the host ip address to the correct subrange using ip-helper function

Boot a mac holding down the N button.

Be amazed!