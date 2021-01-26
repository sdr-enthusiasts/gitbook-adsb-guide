---
description: >-
  If you wish to manage and view data from a remote adsb station, follow the steps
  below before you deploy to the remote location
---

# Managing a remote station using ZeroTier

[`ZeroTier`](https://www.zerotier.com/) is a peer to peer networking tool that is free (for up to 50 devices). Unlike other tools such as [`Openvpn`](https://openvpn.net/) which route all traffic over the VPN, ZeroTier selectively routes only some of your traffic. In practice if both your remote station and local computer are 'members' of the same ZeroTier network then they appear to be on the same local lan. This guide assumes you have physical access to the machine you wish to install ZeroTier on.

## Getting Started - Instal ZeroTier on your local machine

You should create a ZeroTier Account, and install the ZeroTier client on the machine you intend to use to connect to your remote station as per these [`instructions`](https://www.zerotier.com/download/) .

## Create a ZeroTier network

Now you need to create a ZeroTier 'network' for your devices to connect to - this is as simple as clicking on the `Create a Network` button once you have logged in to ZeroTier. If you need help, have a look at this [`guide`](https://www.stratospherix.com/support/setupvpn_01.php) (but stop before the step where you assign IP address ranges).

You should now have a unique 16 character Network ID e.g. `255724d630f06682`. Make a note of this as you will need it in the next steps.

## Deploying `ZeroTier` container

The next step is to deploy a ZeroTier container on your remote station, feel free to choose your own container, but in this example we will use this well maintained container with mult-architecture support by [`bltavares`](https://hub.docker.com/r/bltavares/zerotier).

Open the `docker-compose.yml` file that was created when deploying `readsb`. Append the following lines to the end of the file:

```yaml
  ZeroTier:
    image: bltavares/zerotier:latest
    devices:
     - /dev/net/tun
    container_name: ZeroTier
    restart: always
    environment:
     - net=host
     - cap-add=NET_ADMIN
     - cap-add=SYS_ADMIN
    volumes:
      - '/var/lib/zerotier-one:/var/lib/zerotier-one'
```

Once the file has been updated, issue the command `docker-compose up -d` in the application directory to apply the changes and bring up the `Zerotier` container.

## Joining your Network

Open a shell inside the ZeroTier container with the following command.

```yaml
 sudo docker exec -it ZeroTier bash
```
and then join your network

```yaml
 zerotier-cli join xxxxxxxxxxxxxxxx
```
where xxxxxxxxxxxxxxxx is the 16 digit ID of your network.

finally exit the shell with the command

```yaml
 exit
```

## Authorising your station

Now that youe remote station has joined the network you need to authorise it. Logon to your ZeroTier account in a brower, click on the network you created earlier and scoll down to the `Members` section. Check the box marked `Auth` for your remote station and take note of the `Managed IPs` Section.

You can now use the Managed IPs to communicate as if your devices were on the same network.

More detail on this step is available on this [`link`](https://zerotier.atlassian.net/wiki/spaces/SD/pages/8454145/Getting+Started+with+ZeroTier).
