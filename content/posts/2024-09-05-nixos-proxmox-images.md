---
author: Josh Lee
categories:
- Homelab
date: "2024-09-05T00:00:00Z"
excerpt: Ever since I started using Proxmox, I’ve wanted something like a Dockerfile
  to define my VMs. With my developer background, configuration as code just makes
  sense.
tags:
- nixos
- proxmox
title: 'NixOS + Proxmox: A Recipe for a Declarative Homelab'
url: /nixos-proxmox-vm-images/
---
## Using NixOS with Proxmox: A Declarative Approach to Homelab Configuration

Ever since I started using Proxmox, I’ve wanted something like a Dockerfile to define my VMs. With my developer background, configuration as code just makes sense.

For a while, I did what most people do—I created a base image that worked for me and carefully crafted Ansible plays to shape that image into the various setups I needed.

But in 2023, I kept hearing about "Nix," mostly as a magical package manager that made working across different programming languages a breeze. What I didn’t know was that NixOS would soon revolutionize how I define VM configurations for my homelab—entirely declaratively.

## Discovering NixOS

If you’re not familiar with Nix or NixOS, I recommend checking out some excellent [videos](https://www.youtube.com/watch?v=CwfKlX3rA6E) on the topic. It’s a deep rabbit hole, but the rewards are well worth it.

At the core of NixOS is the Nix package manager, which builds each package in isolation and stores them in an append-only structure. This means no dependency conflicts and super easy rollbacks by just updating system-wide symlinks to a previous state.

When combined with the Nix language (imagine "JSON with functions"), you get full declarative definitions for your servers—covering everything from packages to configurations, and even systemd services.

## Building Proxmox VM Images with Nix

Managing VMs gets even better. The `nixos-generators` package can convert a Nix configuration into various formats, like Proxmox `.vma` images.

For example, you can run:

```
nixos-generate -f proxmox -c configuration.nix
```

Where `configuration.nix` is your NixOS configuration file (here's an example [gist](https://gist.github.com/joshleecreates/e6892ca21b0e6b7c24d96ca2a24bf23e)). This command creates a backup image you can upload and deploy to Proxmox.

## Updating VMs with NixOS

After deploying the `.vma` image to Proxmox, you can remotely update the configuration using `nixos-rebuild`. Services are restarted only if necessary, and rollbacks are as easy as rebooting and selecting a previous configuration from the bootloader. It’s like Ansible, but with superpowers:

```
nixos-rebuild --flake .#flakeTarget --target-host user@remote-host --use-remote-sudo
```

You can also use the `--build-host` flag to specify a different machine to build the configuration on, which is handy if you have a more powerful machine for builds or want to skip the upload step.

I’ve been using this approach for configuring my hosts for a few months now, and it’s been working fantastically. Ansible still has its place for certain tasks, but most of my VMs are now stateless, and rollouts happen via `nixos-rebuild`.

## NixOS Configuration Walkthrough

NixOS has some configuration options that simplify this process. Here’s a breakdown of my base template:

### Importing the QEMU Guest Profile

```nix
{ config, pkgs, modulesPath, lib, system, ... }:

{
  imports = [
    (modulesPath + "/profiles/qemu-guest.nix")
  ];
```

This section imports the QEMU Guest profile, which adds drivers for the virtual devices commonly found in KVM/QEMU-based hypervisors like Proxmox. You can find several such profiles in `nixpkgs` depending on your hardware, available on [GitHub](https://github.com/NixOS/nixpkgs/tree/master/nixos/modules/profiles).

### Configuring Hostname and QEMU Guest Service

```bash
  networking.hostName = lib.mkDefault "base"; # Provide a default hostname
  services.qemuGuest.enable = lib.mkDefault true; # Enable QEMU Guest for Proxmox
```

Here, I set the hostname and enable the QEMU Guest service. Setting a single configuration value installs the package, creates necessary services, and sets defaults. The QEMU guest agent allows Proxmox to safely shut down the VM.

The `lib.mkDefault` function makes these values easy to override in other files that import this configuration. It’s the opposite of `!important` in CSS.

### Setting Up the Boot Loader

```bash
  boot.loader.grub.enable = lib.mkDefault true; # Use the boot drive for GRUB
  boot.loader.grub.devices = [ "nodev" ];
```

Next, I enable the bootloader and configure it to use the boot device instead of an EFI partition. This keeps the VM configuration simple since no extra devices are needed. The bootloader lets you roll back configurations from within the Proxmox console if something goes wrong.

### Automatically Growing the Partition

```bash
  boot.growPartition = lib.mkDefault true;
```

This option is critical—it automatically grows the boot partition to match the size of the disk. This allows you to easily add more space to the VM by resizing the drive in Proxmox. NixOS is great, but it can accumulate cruft in the Nix store, so being able to expand disk space when needed is a lifesaver.

### Enabling Remote Updates

```bash
  nix.settings.trusted-users = [ "root" "@wheel" ]; # Allow remote updates
  nix.settings.experimental-features = [ "nix-command" "flakes" ]; # Enable flakes
```

These options allow you to update the host remotely using `nixos-rebuild` as a non-root user. They also enable some experimental but stable Nix features like flakes.

### Essential Packages

```bash
  environment.systemPackages = with pkgs; [
    vim  # for emergencies
    git  # for pulling Nix flakes
    python3  # for Ansible
  ];
```

Out of the box, NixOS only comes with Nano as a text editor. I rarely need to edit files on the host since most configurations are immutable, but if I do, Vim is essential. Git is useful if you’re using the host as a build machine, and Python is required for Ansible.

I keep this base list short and add packages as needed for specific templates.

### Referencing Root Disk by Label

```bash
  fileSystems."/" = lib.mkDefault {
    device = "/dev/disk/by-label/nixos";
    autoResize = true;
    fsType = "ext4";
  };
```

Referencing the root disk by label is super useful in VMs where you won’t know the device ID ahead of time.

### Adding a User and SSH Key

Lastly, if you want to use this template, you’ll need to modify it with a user and SSH key. I also like to enable passwordless SSH and sudo:

```nix
{
  ...
  security.sudo.wheelNeedsPassword = false; # Don't ask for passwords
  services.openssh = {
    enable = true;
    settings.PasswordAuthentication = false;
    settings.KbdInteractiveAuthentication = false;
  };
  programs.ssh.startAgent = true;

  # Add an admin user
  users.users.your_username = {
    isNormalUser = true;
    description = "Your Name";
    extraGroups = [ "networkmanager" "wheel" ];
  };

  users.users.your_username.openssh.authorizedKeys.keys = [
    "YOUR SSH PUBLIC KEY"
  ];
}
```

## Conclusion

This [gist](https://gist.github.com/joshleecreates/e6892ca21b0e6b7c24d96ca2a24bf23e) is just the start of how I’m using NixOS with Proxmox. I encourage you to clone it and make it your own. You may want to include other services by default, such as Avahi for mDNS or Prometheus exporters.

In the future, I’ll write about automating homelab services with NixOS, including Tailscale, Nginx, Grafana, Prometheus, Docker, Portainer, and more. You can follow me on [LinkedIn](https://www.linkedin.com/in/joshuamlee/) or [Mastodon](https://hachyderm.io/@joshleecreates) for updates.
