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
2. Change the security driver

Search for the security_driver line. It might be commented out with a #. Remove the # and change it to:
Code snippet

security_driver = "none"

3. Restart Libvirt

Apply the changes by restarting the service:
Bash

sudo systemctl restart libvirtd

4. Start CRC

Now you can start the cluster:
Bash

crc start


### 3. Save (Commit) your work
* Click the green **Commit changes...** button at the top right of the screen.
* In the "Commit message" box, type: `Add libvirt troubleshooting steps`
* Click the final **Commit changes** button.

---

### You're finished!
Now, when you go back to your repository's main page, you will see all those instructions beautifully formatted. 

**One last tip:** If you want to see what it looks like *before* you save, you can click the **Preview** tab at the top of the editor while you are pasting the text. 

Does the formatting look correct on your page now?
