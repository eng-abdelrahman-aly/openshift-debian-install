# OpenShift (CRC) Installation on Debian - Fix Guide

This repository documents how to install Red Hat CodeReady Containers (CRC) on Debian and how to fix the common `Permission denied` error with `libvirt`.

## The Error
When running `crc start`, the process fails with:
`ERRO Error starting machine: ... Could not open 'crc.qcow2': Permission denied`

## The Solution
The issue is caused by the Libvirt security driver (AppArmor) on Debian. Follow these steps to fix it:

1. **Kill hung processes:**
   ```bash
   ps aux | grep -E '[c]rc daemon|[c]rcd'
   # kill the process ID found
