# OpenShift (CRC) Installation on Debian - Fix Guide

This repository documents how to install Red Hat CodeReady Containers (CRC) on Debian and how to fix the common `Permission denied` error with `libvirt`.

## The Error
When running `crc start`, the process fails with:
`ERRO Error starting machine: ... Could not open 'crc.qcow2': Permission denied`

## The Solution
The issue is caused by the Libvirt security driver (AppArmor) on Debian. Follow these steps to fix it:

Step 1: Clean up hung processes

Before applying the fix, ensure no background CRC daemons are stuck:
Bash

# Check for the process
ps aux | grep -E '[c]rc daemon|[c]rcd'

# Kill the process (replace <PID> with the number found above)
kill <PID>

Step 2: Disable Libvirt Security Driver

You must tell Libvirt not to use a security driver for QEMU.

    Open the configuration file:
    Bash

    sudo vim /etc/libvirt/qemu.conf

    Find the security_driver setting. It is usually commented out like this: #security_driver = "selinux".

    Change it to:
    security_driver = "none"

    Save and exit (in Vim, type :wq and press Enter).

Step 3: Restart Services

Apply the new configuration by restarting the libvirt daemon:
Bash

sudo systemctl restart libvirtd

Step 4: Verify and Start

Verify that the qcow2 file is accessible and start CRC:
Bash

# Verify file existence
ls -l /home/abdo/.crc/cache/crc_libvirt_4.21.4_amd64/crc.qcow2

# Start CRC
crc start

Finalized detailed troubleshooting guide



# Red Hat CRC: The "Persistent Socket" Fix for Debian

This guide resolves the common "Address already in use" loop where the CRC daemon keeps respawning even after being killed.
# What is VSOCK?

VSOCK (Virtual Socket) is a specialized network family (AF_VSOCK) used for communication between a Hypervisor (Host) and Virtual Machines.

Unlike standard TCP/IP which uses IP addresses (like 127.0.0.1), VSOCK uses a CID (Context Identifier) and a Port.

    CRC standard port: 1024

    The Problem: If a CRC process crashes or is managed by systemd, the kernel locks port 1024. Because it’s a hardware-level communication channel, standard networking tools often can't "see" why the port is busy.

# Complete Reset Procedure
1. Clear Global (Root) Conflicts

If sudo crc was ever run, a system-wide socket might be "squatting" on the VSOCK port. We must disable these first.
Bash

# Stop any system-wide services
sudo systemctl stop crc-vsock.socket 2>/dev/null
sudo systemctl stop crc-http.socket 2>/dev/null
sudo systemctl stop crc-daemon.service 2>/dev/null

# Disable them so they don't auto-start on boot
sudo systemctl disable crc-vsock.socket crc-http.socket 2>/dev/null

# Refresh the global systemd manager
sudo systemctl daemon-reload

2. Break the User-Level Respawn Loop

Systemd "Socket Activation" will try to restart CRC the moment you kill it. We must move the configuration files to stop this.
Bash

# 1. Back up and move user unit files
mkdir ~/crc-backup
mv ~/.config/systemd/user/crc-* ~/crc-backup/

# 2. Force systemd to flush its internal memory
systemctl --user daemon-reload
systemctl --user daemon-reexec

# 3. Kill any remaining "ghost" processes
pkill -9 crc

3. Force-Release the Hardware Port

Verify port 1024 is empty. If it still shows a PID, use fuser to "kick" the process off the hardware.
Bash

# Check port status
sudo ss -l --vsock -p | grep 1024

# If NOT empty, run:
sudo fuser -k 1024/vsock

4. Restore and Re-Initialize

Now that the path is clear, restore the environment to a clean, working state.
Bash

# 1. Restore unit files
mv ~/crc-backup/* ~/.config/systemd/user/
systemctl --user daemon-reload

# 2. Run CRC setup (Fixes libvirt permissions & groups)
crc setup

# 3. Start the cluster
crc start

# Summary of Key Commands
Command	Why it's needed
daemon-reexec	Restarts the systemd manager to clear "ghost" jobs from memory.
fuser -k 1024/vsock	Forcefully releases the VSOCK communication channel.
systemctl disable	Prevents Root-level sockets from fighting your User-level sockets.
Pro-Tip

Ensure your user is in the libvirt group to avoid permission issues with the VM driver:
Bash

sudo usermod -aG libvirt $USER && newgrp libvirt

 
