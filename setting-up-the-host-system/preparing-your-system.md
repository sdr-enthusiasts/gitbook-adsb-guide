# Preparing Your System

## Raspberry Pi

Raspberry Pi single-board computers (SBCs) are a popular choice for flight data enthusiasts running this software.

If you're generally unfamiliar with Linux or working with Raspberry Pi, the [official Getting started guide](https://www.raspberrypi.com/documentation/computers/getting-started.html) is a great place to start, and will walk you through the process of installing Raspberry Pi OS - you'll want to enable SSH.

If SSH'ing into a server and editing a text file with nano or vi is something you've done before, you might consider installing [DietPi](https://dietpi.com/docs/install/).

## Hints for Linux Newbies

Collecting ADS-B data with an SDR is a great project to learn about Linux. While this guide isn't a Linux crash course, it's worth pointing out a few basic concepts to ease into things.

### Entering Commands

Throughout this guide, you'll be presented with commands to copy/paste into your new device. If you're accessing your new Linux device directly with a keyboard, mouse, and display like a traditional desktop PC, you'll be entering these via the "terminal," "command line," or "command prompt." If you're using something with a fancy graphical user interface, such as Raspberry Pi OS with a mouse, keyboard, and display plugged in like a traditional desktop computer, it's likely there's a "Terminal" app you can access with a few clicks.

### Remote Access: SSH

Once you've connected your Linux device to your network, it's possible to access and manage it with SSH. On macOS, you can do this via the Terminal app, on Windows, you can use [PuTTY](https://www.putty.org/). You can connect to your new device via SSH entering this command `ssh username@new.device.ip.addresss` in Terminal/PuTTY.

### Raspberry Pi DHCPCD Issue

Some users have reported issues with Raspberry Pi devices loosing network connectivity when running multiple containers. In effort to prevent this, run the following command `echo "denyinterfaces veth*" >> /etc/dhcpcd.conf`. This will append the line `denyinterfaces veth*` to end of the file. The file will look something like this (note the last line):

```
# A sample configuration for dhcpcd.
# See dhcpcd.conf(5) for details.

# Allow users of this group to interact with dhcpcd via the control socket.
#controlgroup wheel

# Inform the DHCP server of our hostname for DDNS.
hostname

# Use the hardware address of the interface for the Client ID.
clientid
# or
# Use the same DUID + IAID as set in DHCPv6 for DHCPv4 ClientID as per RFC4361.
# Some non-RFC compliant DHCP servers do not reply with this set.
# In this case, comment out duid and enable clientid above.
#duid

# Persist interface configuration when dhcpcd exits.
persistent

# Rapid commit support.
# Safe to enable by default because it requires the equivalent option set
# on the server to actually work.
option rapid_commit

# A list of options to request from the DHCP server.
option domain_name_servers, domain_name, domain_search, host_name
option classless_static_routes
# Respect the network MTU. This is applied to DHCP routes.
option interface_mtu

# Most distributions have NTP support.
#option ntp_servers

# A ServerID is required by RFC2131.
require dhcp_server_identifier

# Generate SLAAC address using the Hardware Address of the interface
#slaac hwaddr
# OR generate Stable Private IPv6 Addresses based from the DUID
slaac private

# Example static IP configuration:
#interface eth0echo "denyinterfaces veth*" >> /etc/dhcpd.conf
#static ip_address=192.168.0.10/24
#static ip6_address=fd51:42f8:caae:d92e::ff/64
#static routers=192.168.0.1
#static domain_name_servers=192.168.0.1 8.8.8.8 fd51:42f8:caae:d92e::1

# It is possible to fall back to a static IP if DHCP fails:
# define static profile
#profile static_eth0
#static ip_address=192.168.1.23/24
#static routers=192.168.1.1
#static domain_name_servers=192.168.1.1

# fallback to static profile on eth0
#interface eth0
#fallback static_eth0
denyinterfaces veth*
```

After saving this file, restart DHCPCD

```
sudo systemctl restart dhcpcd
```
