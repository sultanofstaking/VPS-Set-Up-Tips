# VPS-Set-Up

VPS Comparison (Ubuntu 20.04)
* Vultr (high frequency 16gb seems best deal for pillar)
    * Pro
        * Simple clean layout
        * Lots of data centers
        * Simple to enable pre-set alerts
    * Con
        * Limited native alerts from Vultr (can use external solutions)
* Digital Ocean 
    * Pro
        * Simple clean layout
        * Easy to set alerts
    * Con
        * Seemingly fewer data centers
* OVH 
    * Pro
        * TBD
    * Con
        * Doesn't natively support ED25519 keys (can embed in VPS)
        * Seemingly fewer data centers
        * Not intuitive to set up alerts
        * Allows Ubuntu login after root disable
* AWS t4g (better than t3 and reservable) https://www.learnaws.org/2020/12/19/t3-t3a-t4g/ T2. Large $70 per month
    * Pro
        * Lots of data centers
        * Easy to set alerts
    * Con
        * Expensive
        * Doesn't natively support ED25519 keys (can embed in VPS)
        * Must change hostname through CLI (uses private IPv4 for hostname)
        * Allows Ubuntu login after root disable

Mac (client) Commands
#Generate ssh key
ssh-keygen -t ed25519 -o -a 100
#OR ssh-keygen -t rsa -b 4096

#Ensure key is not publicly visible
chmod 400 ~/.ssh/id_ed25519 
#OR chmod 400 ~/.ssh/id_rsa

#Remove keys as needed
rm ~/.ssh/id_rsa // rm ~/.ssh/id_rsa.pub
#chmod 600 if cant delete

#Copy pub key as needed
#pbcopy < ~/.ssh/id_rsa.pub

#Navigate to SSH file mac only
Command+Shift+G ~/.ssh
#Navigate to Library file mac only
Command+Shift+G ~/Library/

#Edit SSH Config mac only
Host *
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519
  IdentityFile ~/.ssh/id_rsa

#Add to keychain mac only
	ssh-add --apple-load-keychain

Linux Best Practices
    * Never use the root user.
    * Always update the security patches for your OS.
    * Enable and set up a firewall.
    * Never allow password-based SSH, only use key-based access.
    * Disable non-essential SSH subsystems (banner, motd, scp, X11 forwarding) and harden your SSH configuration (reasonable guide to begin with).
    * Back up your storage regularly.

Server (host) Commands
#Get updates
sudo apt update && sudo apt upgrade -y

#Add User
adduser username

#Give sudo permissions to new user
usermod -aG sudo username

#Switch user
su - username

#Add pub key to new user
cd $HOME
mkdir ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys

ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAICaUbJN7isDUJQYatZmc8ToDgGXLA1/lQTAMeSQEsug>
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIApzg+J6DyY7EuYQEoUwqdiIVZ8P1VihfbW47H+NrQy>

#copy paste pub key into this file and save with CTRL+O - path should show /home/username/.ssh/authorized_keys - if right press enter then exit with CTRL+X
chmod 600 ~/.ssh/authorized_keys
exit

#Update ssh config
nano /etc/ssh/sshd_config
#Use CTRL+W to search # means something is commented out, need to delete # to “activate”
PermitRootLogin no
PubkeyAuthentication  yes
PasswordAuthentication  no
ChallengeResponseAuthentication  no
CTRL+O to save then CTRL+X to exit

#Reload ssh 
systemctl reload sshd

#DO NOT LOG OUT OF ROOT try to log in with new user in a new window ssh username@ip

#Access root with new user
sudo -s

#Access root user
sudo su - root

#Install fail2ban
apt install fail2ban

#Add swap space (4 GB)
fallocate -l 4G /swapfile
#use the following if fallocate doesnt work dd if=/dev/zero of=/swapfile bs=1024 count=4194304
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
nano /etc/fstab
#Append the line below, save, and exit
/swapfile swap swap defaults 0 0
sysctl vm.swappiness=25
cat /proc/sys/vm/swappiness

#Reboot server
reboot

#Access sudo
sudo -s

#Confirm fail2ban
systemctl status fail2ban

#Confirm swap
free

Basic Linux Commands: https://linuxcommand.org/lc3_man_pages/ssh1.html 
    * Escape (END) = press q to get back to cli
    * Save = CTRL+O
    * Exit = CTRL+X
    * Remove = rm 
    * Remove files in directory = rm -r
    * Remove directory = rm -rf 
    * Terminate program = ctrl+c
    * View free memory = free -m
    * View memory usage = ps aux
    * View logs = 
        * cd /var/log 
        * ls
        * sudo nano /var/log/log you want to view
        * sudo tail -f /var/log/syslog
    * Exit log = ctrl+c
System utilities
If you want to avoid installing additional software on your system, there are other simple methods to view performance. The tools mentioned here work across most Unix/Linux/BSD platforms.
* 		Memory usage with vmstat, top, free, htop.
* 		CPU/memory/disk with sar.
* 		Bandwidth with iftop.
* 		Network protocols with ntopng.
Basic Linux “Screen” Usage
Below are the most basic steps for getting started with screen:
* On the command prompt, type screen.
* Name screen = screen -S session_name
* List screens = screen -ls
* Use the key sequence Ctrl-a + Ctrl-d to detach from the screen session.
* Reattach to the screen session by typing screen -r number
* exit will exit screen / Ctrl+A, and then K to forcibly kill a window
* Ctrl+a c Create a new window (with shell).
* Ctrl+a " List all windows.
* Ctrl+a 0 Switch to window 0 (by number).
* Ctrl+a A Rename the current window.
* Ctrl+a S Split current region horizontally into two regions.
* Ctrl+a | Split current region vertically into two regions.
* Ctrl+a tab Switch the input focus to the next region.
* Ctrl+a Ctrl+a Toggle between the current and previous windows
* Ctrl+a Q Close all regions but the current one.
* Ctrl+a X Close the current region.

Other Things to Try 
* Firewalls UFW / IPT 
* Ports
* Host side keys 

Resources
* General Best Practices 
    * https://docs.ovh.com/us/en/vps/tips-for-securing-a-vps/ 
    * https://www.howtogeek.com/412055/37-important-linux-commands-you-should-know/
    * https://dzone.com/articles/13-ways-to-secure-your-cloud-vps 
* Secure SSH
    * https://medium.com/bld-nodes/securing-ssh-access-to-your-server-cc1324b9adf6 
* Firewalls 
    * UFW https://www.vultr.com/docs/initial-secure-server-configuration-of-ubuntu-18-04 
    * https://www.digitalocean.com/community/tutorials/ufw-essentials-common-firewall-rules-and-commands 
    * IP Tables https://upcloud.com/community/tutorials/configure-iptables-ubuntu/ 
* Fail2ban 
    * https://linuxhint.com/install_fail2ban_on_ubuntu_20-04/  
* Simplified log-in (bottom of article) https://medium.com/risan/upgrade-your-ssh-key-to-ed25519-c6e8d60d3c54 
* Monitoring (sample from Plasm https://docs.plasmnet.io/build/validator-guide/secure-validator/installation-node-monitoring)
* Courses: https://acloudguru.com/browse-training?vendors%5B%5D=amazon-web-services&vendors%5B%5D=linux&role%5B%5D=sys-ops
* Mac SSH: https://www.getpagespeed.com/work/proper-use-of-ssh-client-in-mac-os-x 
* SSH: https://www.ssh.com/academy/ssh/keygen 
* AWS Users (maybe not helpful…): https://aws.amazon.com/premiumsupport/knowledge-center/new-user-accounts-linux-instance/ 
* UFW Message: https://handyman.dulare.com/ufw-block-messages-in-syslog-how-to-get-rid-of-them/ 
    * Restart after: sudo service rsyslog restart
* SWAP
    * https://www.digitalocean.com/community/tutorials/how-to-configure-virtual-memory-swap-file-on-a-vps
    * https://www.digitalocean.com/community/questions/ubuntu-2gb-swap-file 
    * https://linuxtechlab.com/create-swap-space-linux-system/
    * https://help.ubuntu.com/community/SwapFaq
    * https://www.cyberciti.biz/faq/linux-add-a-swap-file-howto/
    * https://linuxize.com/post/how-to-add-swap-space-on-ubuntu-20-04/
        * bs=1024 count=1048576
* Systemd
    * https://www.freedesktop.org/software/systemd/man/systemd.service.html 

