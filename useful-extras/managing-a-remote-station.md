---
description: >-
  If you wish to manage and view data from a remote adsb station, follow the steps
  below before you deploy to the remote location
---

# Managing a remote station using ZeroTier

[`ZeroTier`](https://www.zerotier.com/) is a peer to peer networking tool that is free (for up to 10 devices). Unlike other tools such as [`OpenVpn`](https://openvpn.net/) which route all traffic over the VPN, ZeroTier selectively routes only some of your traffic. In practice if both your remote station and local computer are 'members' of the same ZeroTier network then they appear to be on the same local LAN. This guide assumes you have physical access to the machine you wish to install ZeroTier on.

## Getting Started - Install ZeroTier on your local machine

You should create a ZeroTier Account, and install the ZeroTier client on the machine you intend to use to connect to your remote station as per these [`instructions`](https://www.zerotier.com/download/).

Note - it is possible to deploy ZeroTier as a container, but we recommend against it. If your station is remote-managed only, you'd want your VPN network to be managed and established as close as possible to the hardware. Putting it in a Docker Container is risky, as your station may become unreachable via ZT if the docker service crashes or exhibits issues.

## Create a ZeroTier network

Now you need to create a ZeroTier 'network' for your devices to connect to - this is as simple as clicking on the `Create a Network` button once you have logged in to ZeroTier. If you need help, have a look at the [`Getting Started`](https://docs.zerotier.com/start) guide.

You should now have a unique 16 character Network ID e.g. `253ef24d630f06682`. Make a note of this as you will need it in the next steps.

## Joining your Network

Join your network from your machine's command line:

```bash
sudo zerotier-cli join xxxxxxxxxxxxxxxx
```

## Authorising your station

Now that your remote station has joined the network you need to authorise it. Logon to your ZeroTier account in a browser, click on the network you created earlier and scroll down to the `Members` section. Check the box marked `Auth` for your remote station and take note of the `Managed IPs` Section.

You can now use the Managed IPs to communicate as if your devices were on the same network.

More detail on this step is available on this [`link`](https://docs.zerotier.com/start#authorize-your-device).
