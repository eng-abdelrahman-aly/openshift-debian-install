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

 
