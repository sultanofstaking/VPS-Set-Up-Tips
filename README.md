# VPS Set-Up Tips

## Resources
* General Best Practices 
    * https://docs.ovh.com/us/en/vps/tips-for-securing-a-vps/ 
    * https://www.howtogeek.com/412055/37-important-linux-commands-you-should-know/
    * https://dzone.com/articles/13-ways-to-secure-your-cloud-vps 
* Secure SSH
    * https://medium.com/bld-nodes/securing-ssh-access-to-your-server-cc1324b9adf6 
* Courses: https://acloudguru.com/browse-training?vendors%5B%5D=amazon-web-services&vendors%5B%5D=linux&role%5B%5D=sys-ops
* Mac SSH: https://www.getpagespeed.com/work/proper-use-of-ssh-client-in-mac-os-x 
* SSH: https://www.ssh.com/academy/ssh/keygen 
* Systemd
    * https://www.freedesktop.org/software/systemd/man/systemd.service.html 
* Firewalls 
    * UFW https://www.vultr.com/docs/initial-secure-server-configuration-of-ubuntu-18-04 
    * https://www.digitalocean.com/community/tutorials/ufw-essentials-common-firewall-rules-and-commands 
    * IP Tables https://upcloud.com/community/tutorials/configure-iptables-ubuntu/ 
* UFW Message: https://handyman.dulare.com/ufw-block-messages-in-syslog-how-to-get-rid-of-them/ 
    * Restart after: sudo service rsyslog restart
* Fail2ban 
    * https://linuxhint.com/install_fail2ban_on_ubuntu_20-04/  
* Simplified log-in (bottom of article) https://medium.com/risan/upgrade-your-ssh-key-to-ed25519-c6e8d60d3c54 
* Monitoring (sample from Plasm https://docs.plasmnet.io/build/validator-guide/secure-validator/installation-node-monitoring)
* SWAP
    * https://www.digitalocean.com/community/tutorials/how-to-configure-virtual-memory-swap-file-on-a-vps
    * https://www.digitalocean.com/community/questions/ubuntu-2gb-swap-file 
    * https://linuxtechlab.com/create-swap-space-linux-system/
    * https://help.ubuntu.com/community/SwapFaq
    * https://www.cyberciti.biz/faq/linux-add-a-swap-file-howto/
    * https://linuxize.com/post/how-to-add-swap-space-on-ubuntu-20-04/
        * bs=1024 count=1048576

## Mac (client) Commands
Generate ssh key (ed25519 is preferred)
`ssh-keygen -t ed25519 -o -a 100`
Or
`ssh-keygen -t rsa -b 4096`

Ensure key is not publicly visible
`chmod 400 ~/.ssh/id_ed25519`
Or 
`chmod 400 ~/.ssh/id_rsa`

Remove keys as needed
`rm ~/.ssh/id_rsa // rm ~/.ssh/id_rsa.pub`
If you cant delete grant access with
`chmod 600` 

Copy pub key as needed
`pbcopy < ~/.ssh/id_rsa.pub`

Navigate to Library file on macos using Command+Shift+G then input ~/Library/

Navigate to SSH files on macos using Command+Shift+G then input ~/.ssh

To edit SSH Config on macos navigate to .ssh using the command above then add the text below to the ssh config file. If file doesnt exist creat it using text editor.
```
Host *
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519
  IdentityFile ~/.ssh/id_rsa
```

Add keys to keychain in terminal so you dont have to enter password for keys every time you log in VPS
`ssh-add --apple-load-keychain`

## Linux Best Practices
- Never use the root user.
- Always update the security patches for your OS.
- Enable and set up a firewall.
- Never allow password-based SSH, only use key-based access.
- Disable non-essential SSH subsystems (banner, motd, scp, X11 forwarding) and harden your SSH configuration (reasonable guide to begin with).
- Back up your storage regularly.

## Server (host) Commands to set up new server
Get updates
`sudo apt update && sudo apt upgrade -y`

Add User
`adduser username`

Give sudo permissions to new user
`usermod -aG sudo username`

Switch user
`su - username`

Add pub key to new user
```
cd $HOME
mkdir ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
```

Copy paste pub key into this file and save with CTRL+O - path should show /home/username/.ssh/authorized_keys - if right press enter then exit with CTRL+X
`chmod 600 ~/.ssh/authorized_keys` then `exit` to get back to root user

Update ssh config
`nano /etc/ssh/sshd_config`

Use CTRL+W to search # means something is commented out, need to delete # to “activate”. Search for the lines below and update as needed.
```
PermitRootLogin no
PubkeyAuthentication  yes
PasswordAuthentication  no
ChallengeResponseAuthentication  no
```

Once changed CTRL+O to save then CTRL+X to exit

Reload ssh 
`systemctl reload sshd`

**DO NOT LOG OUT OF ROOT try to log in with new user in a new window ssh username@ip**

Access root with new user
`sudo -s`

Access root user
`sudo su - root`

Install fail2ban
`apt install fail2ban`

Add swap space (4 GB)
`fallocate -l 4G /swapfile`

Use the following if fallocate doesnt work `dd if=/dev/zero of=/swapfile bs=1024 count=4194304`

Enter the commands below to configure the swap space
```
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```

Edit the fstab file `nano /etc/fstab`

Append the line below, save, and exit
```
/swapfile swap swap defaults 0 0
sysctl vm.swappiness=25
cat /proc/sys/vm/swappiness
```

Reboot server
`reboot`

Access sudo
`sudo -s`

Confirm fail2ban
`systemctl status fail2ban`

Confirm swap
`free`

## Basic Linux Commands: https://linuxcommand.org/lc3_man_pages/ssh1.html 
Escape (END) = press q to get back to cli
- Save = CTRL+O
- Exit = CTRL+X
- Remove = `rm`
- Remove files in directory = `rm -r`
- Remove directory = `rm -rf`
- Terminate program = `ctrl+c`
- View free memory = `free -m`
- View memory usage = `ps aux`
- View logs = 
`cd /var/log`
`ls`
`sudo nano /var/log/log` replace log with what you want to view
- Exit log = ctrl+c

## System utilities to view performance
- Memory usage with `vmstat` `top` `free` `htop`
- CPU/memory/disk with `sar`
- Bandwidth with `iftop`
- Network protocols with `ntopng`

## Basic Linux “Screen” Usage
- On the command prompt, type `screen`
- Name screen = `screen -S session_name`
- List screens = `screen -ls`
- Use the key sequence Ctrl-a + Ctrl-d to detach from the screen session.
- Reattach to the screen session by typing `screen -r number`
- `exit` will exit screen / Ctrl+A, and then K to forcibly kill a window
- Ctrl+a c Create a new window (with shell).
- Ctrl+a " List all windows.
- Ctrl+a 0 Switch to window 0 (by number).
- Ctrl+a A Rename the current window.
- Ctrl+a S Split current region horizontally into two regions.
- Ctrl+a | Split current region vertically into two regions.
- Ctrl+a tab Switch the input focus to the next region.
- Ctrl+a Ctrl+a Toggle between the current and previous windows
- Ctrl+a Q Close all regions but the current one.
- Ctrl+a X Close the current region.




