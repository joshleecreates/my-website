---
author: Josh Lee
categories:
- Homelab
date: "2025-03-16T00:00:00Z"
excerpt: In this post we'll use Tailscale to connect Proxmox VE hosts across the public internet. We'll do this by combining Proxmox SDNs (Software Defined Networks) with Tailscale for overlay networking. 
tags:
- nixos
- proxmox
title: 'NixOS + Proxmox Part 2: Overlay Networking with Tailscale and Proxmox SDNs'
url: /nixos-proxmox-tailscale/
---

> **Note:** I no longer use this setup. This post is kept for reference but the approach described here has been deprecated in my homelab.

In this post we'll use Tailscale to connect Proxmox VE hosts across the public internet. We'll do this by combining Proxmox SDNs (Software Defined Networks) with Tailscale for overlay networking.

## Prerequisites
In order to follow along, you will need access to one or more Proxmox VE hosts, a free tailscale account, and nix installed locally. 

## Why do this?
My Homelab is spread across three physical locations, each running on a consumer-grade router without any VPN capabilities. So for this project my goals were:

1. Connect all of my Proxmox hosts and VMs to a single subnet for management and monitoring.
2. Avoid modifying the Proxmox Host itself, as much as possible.
3. Allow for some VMs to be connected to the tailnet without an internet connection (behind the NAT of the Proxmox SDN).

## The Network Layout
For this demonstration we'll pretend we're only connecting two Proxmox hosts: `Proxmox Main` and `Proxmox Remote`. Each will run a couple of VMs.

I've chosen the private subnet `10.10.0.0/16` to use for my overlay network. Each Proxmox host will create a `10.10.x.0/24` subnet, and the VMs will receive a `10.10.x.0/24` IP.

The first VM we will create on each host will be a Tailscale tailnet router — I call these "tailgates" — "tailnet" + "gateway"...

I like to save the `.100` IP address for the tailgate, so it will always be `10.10.x.100`.

Each tailgate VM will advertise its local `10.10.x.0/24` subnet to the tailnet, and also act as a router to the entire `10.10.0.0/16` subnet.

So we end up with a network that looks like this: 

```
10.10.0.0/16
├── 10.10.1.0/24 - Proxmox Main SDN
│   ├── 10.10.1.1/24 - Proxmox Host
│   ├── 10.10.1.72/24 - Some other DHCP VM
│   └── 10.10.1.100/24 - Proxmox Main Tailgate
├── 10.10.2.0/24 - Proxmox Remote SDN
│   ├── 10.10.2.1/24 - Proxmox Host
│   └── 10.10.2.100/24 - Tailgate VM
```

## Step 1: Create the SDN on each Proxmox Host

First, we need to create a Proxmox SDN on each host. This will be the subnet that the VMs will use for communication.

SDNs are available as of Proxmox 8, under the `Datacenter` section of the admin panel menu.

### Create a Zone
Under `Zones`, create a zone called `tail0`. You can choose a simple zone for a single-node setup and VLAN for a multi-node Proxmox Cluster. You'll want to enable the PVE IPAM for the SDN.

Also, click on the "Advanced" checkbox and then check the "Use DHCP" checkbox — without this, your VMs will not be assigned an IP address automatically.

### Install dnsmasq
Somewhat annoyingly, the settings we chose above require dnsmasq to be installed on the Proxmox host. It is not installed by default. On each proxmox host console, install dnsmasq:
```bash
apt install -y dnsmasq
```

### Create a vnet
Back in the Proxmox UI, create a vnet. You can call it `tail0` again and select the zone you created in the previous step. Choose the appropriate VLAN settings for your setup.

### Create a Zone
Under `Zones`, create a new zone. For example, `10.10.1.0/24`, with a gateway `10.10.1.1`. If you enable SNAT, VMs will be able to use this network to make outgoing connections to the internet, through your Proxmox Host. Whether you want that enabled or not is up to you.

The Proxmox host itself will join the network at the `10.10.1.1` IP address and act as a software router.

Repeat these steps for each Proxmox host you want to connect.

## Step 2. Create the Tailgate VMs
We'll use NixOS to create the tailgate VMs. As a starting point, checkout [Part I: Nixos + Proxmox](/nixos-proxmox-vm-images/).

We'll add the following config to the template from that post:

```nix
services.tailscale.enable = true;
services.tailscale.useRoutingFeatures = "both";
```

### Build the VM
Build the image using `nixos-generate`:

```bash
nixos-generate -f proxmox -c /path/to/configuration.nix
```

The output of this command will include a path to a `.vma` file in your nix-store. You can upload this file to Proxmox and create a new VM from it:

```bash
scp /path/to/.vma root@proxmox:/var/lib/vz/dump
```

### Create the VM
In the Proxmox UI, choose your host's local storage (`local` by default) and navigate to `Backups`. You should see your recently uploaded image. Select it and click `Restore`. You don't need to dedicate more than 1 core and 512MB of RAM to this VM.

Do not boot the VM on creation. First, navigate to the VM configuration:

- Navigate to "Hardware"
- Click "Add" and choose "Network Device"
- Under "Bridge", select the vnet you created in the previous step.

### Assign a Static IP
Now boot the VM. Then, to assign a specific IP address, navigate to `Datacenter > SDN > IPAM` in the Proxmox admin UI. Find the VM you just created and assign it to the IP address, e.g. `10.10.1.100`.

Reboot the VM so that it will pick up the new network settings.

### Configure Tailscale
Though we've installed Tailscale on each of our tailgate VMs, it isn't configured. We can configure it declaratively with NixOS or Ansible, but that's beyond the scope of this post.

Instead, simply login into each host and run the command:

```bash
sudo tailscale up --advertise-routes=10.10.1.0/24 --accept-routes
```

Next, you'll need to login to the Tailscale Admin panel and approve each new route advertisement.

Repeat these steps and create a tailgate VM on each Proxmox host, using incrementing subnets.

## Step 3. Configure the "Client" VMs
To add a Proxmox VM to this network, add a new network device to the tail0 subnet, the same way that we did for the Tailgate VMs. Then, for the VM to be able to speak back to tailnet hosts, it needs a route to the tail0 subnet. You can do this on the command line:

```bash
sudo ip route add 10.10.0.0/16 via 10.10.1.100
```

Or of course by editing your network configuration. On NixOS hosts you can add this to your configuration.nix:

```nix
networking.interfaces.eth0.routes = [
  { to = "10.10.0.0/16"; via = "10.10.1.100"; }
];
```

Since we're using a broad subnet here (`10.10.0.0/16` instead of `10.10.x.0/24`), the route will apply to all VMs on any Proxmox host, even new ones that haven't yet joined our tailnet. This saves us from having to go back and update VMs as we add or remove Proxmox hosts.

## Tada
That's it! Now we can use the 10.10.0.0/16 subnet for management and monitoring of all VMs across all of our connected Proxmox Hosts. For example, our Prometheus node can scrape node exporters from all VMs, and an Nginx instance on one host can serve resources from any of the others.

### NB: Proxmox and DHCP
This blog post was slightly delayed because originally I was trying to run all of the networking and tailscale inside Proxmox instead of a VM. For various reasons, this wasn't stable. 

It was possible to use DHCP with my Proxmox hosts, but it made the SDNs unstable.

The final killer for me was conflicts between the tailscale client's DNS settings and the Proxmox VE DNS settings, which I could not prevent from clobbering eachother. 

In the end, I went with this layout using a VM to handle all of the networking and tailscale.

## Next Steps
This is pretty cool, but it isn't perfect. One major piece is missing: DNS. For my own set up, I'm running dnsmasq and nginx inside an ingress VM. 

I have the `.internal` tld configured in my tailscale settings to point to this DNS server, and I update the hosts file on the ingress VM to point to the Proxmox hosts and VMs. 

Ideally, I'd like to update this to be more automated so that I don't have to update my DNS config every time I add a new VM or Proxmox host. But that's a topic for another post.

**This is Part II in "NixOS+Proxmox"**:

 - [Part I: Building NixOS Images for Proxmox](https://www.joshuamlee.com/nixos-proxmox-vm-images/)  
 - [Part II: Using NixOS as a Tailnet Router for Proxmox SDNs](https://www.joshuamlee.com/nixos-proxmox-tailscale/)  
 - [NixOS+Proxmox Homelab Example Repository](https://github.com/joshleecreates/nixos-proxmox)
