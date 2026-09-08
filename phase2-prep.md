To prepare for phase 2 lessons we need to use a previous OS called Bookworm instead of Trixie.  Here are the steps to modify the card.

1. Boot the cloned card and SSH in as the original user
   ```BASH
   ssh lhsengr03a@<ip>
   ```
   
2. Reset machine identity (do this first, before anything else)
   ```BASH
   sudo rm -f /etc/machine-id
   sudo systemd-machine-id-setup
   ```

   ```BASH
   sudo rm -f /var/lib/dbus/machine-id
   sudo ln -s /etc/machine-id /var/lib/dbus/machine-id
   ```

   ```BASH
   sudo rm -f /etc/ssh/ssh_host_*
   sudo dpkg-reconfigure openssh-server
   ```

   ```BASH
   sudo reboot
   ```
   
3. Set the new hostname
   ```BASH
   ssh lhsengr03a@<PI_IP>
   sudo raspi-config nonint do_hostname lhsengr03a
   sudo reboot
   ```
   
4. Create the new user
   ```BASH
   ssh lhsengr03a@<IP>
   sudo adduser <new user>
   ```
      
5. Copy group memberships from the old user to the new one
   ```BASH
   groups lhsengr03a
   ```
   
   Copy the output (excluding the username itself), then run:
   ```BASH
   sudo usermod -aG <paste-group-list-here> <new user>
   ```
   
6. Log out and log back in as the new user

   ```BASH
   exit
   ssh <newusername>@ip>
   ```
   9. Set up RPi Connect fresh (as the new user)

   ```BASH
   rpi-connect signin
   ```

   Give me the unique code at the end of the url
   
7. Verify the camera and environment
   ```BASH
   rpicam-hello --list-cameras
   ```
