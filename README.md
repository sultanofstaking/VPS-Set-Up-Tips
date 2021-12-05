# VPS Set-Up Tips

## I created this guide to provide anyone running a node with some basic resources and tips to set up a secure VPS. There are many things you can do to secure your VPS and everyone approaches this a little differenlty. This is meant to give you a starting point and some resources to learn more. When first learning how to secure a VPS it is not uncommon to lock yourself out. I have included text where possible to help prevent that, but to be safe I would try this guide on a test machine and get comfortable with your personal set-up before replicating on any production machine. The tips below are my own and not endorsed by any team.  Please use the issue feature for fixes here. **Try at your own risk.**

## If you are new to virtual machines you are probably wondering where to start. You are going to want to find a provider that is cheap with a simple interface to learn on. I launch most of my test instances on Digital Ocean. If you would like to try them out use my referral code for a $100 credit over 60 days https://m.do.co/c/ab6f82b1c45f. Another option is AWS as it has a free tier. AWS is a bit more complex to get started because it has a ton of great features you dont need if you are just getting started but once you figure it out the free tier is well... free. You will likely need to upgrade however if you wish to run production nodes so in that case, I would recommend you pick a provider you are comfortable with and start there.

## I am providing resources up front instead of at the end because the best way to secure your machine is by learning. As ironic as it is in this guide, you should never blindly trust copy paste guides you find on the internet. The more your know the better protected you will be.
* General Best Practices 
    * https://docs.ovh.com/us/en/vps/tips-for-securing-a-vps/ 
    * https://www.howtogeek.com/412055/37-important-linux-commands-you-should-know/
    * https://dzone.com/articles/13-ways-to-secure-your-cloud-vps 
    * https://www.vultr.com/docs/initial-secure-server-configuration-of-ubuntu-18-04
* Secure SSH
    * https://medium.com/bld-nodes/securing-ssh-access-to-your-server-cc1324b9adf6 
    * https://stribika.github.io/2015/01/04/secure-secure-shell.html
    * https://medium.com/risan/upgrade-your-ssh-key-to-ed25519-c6e8d60d3c54 
    * https://www.ssh.com/academy/ssh/keygen 
* Mac SSH 
    * https://www.getpagespeed.com/work/proper-use-of-ssh-client-in-mac-os-x 
* Systemd
    * https://www.freedesktop.org/software/systemd/man/systemd.service.html 
* Firewalls 
    * UFW https://www.digitalocean.com/community/tutorials/ufw-essentials-common-firewall-rules-and-commands 
    * IP Tables https://upcloud.com/community/tutorials/configure-iptables-ubuntu/ 
* Fail2ban 
    * https://linuxhint.com/install_fail2ban_on_ubuntu_20-04/  
* Monitoring (sample from Plasm) 
    * https://docs.plasmnet.io/build/validator-guide/secure-validator/installation-node-monitoring)
* SWAP
    * https://www.digitalocean.com/community/tutorials/how-to-configure-virtual-memory-swap-file-on-a-vps
    * https://www.digitalocean.com/community/questions/ubuntu-2gb-swap-file 
    * https://linuxtechlab.com/create-swap-space-linux-system/
    * https://help.ubuntu.com/community/SwapFaq
    * https://www.cyberciti.biz/faq/linux-add-a-swap-file-howto/
    * https://linuxize.com/post/how-to-add-swap-space-on-ubuntu-20-04/
        * bs=1024 count=1048576

## Set Up SSH Keys on Mac (client)
Open Terminal

Generate ssh key (ed25519 is preferred - read Secure SSH resources for more perspectives on this)
`ssh-keygen -t ed25519 -o -a 100`
Or
`ssh-keygen -t rsa -b 4096`

Accept the default path to save.

Use a strong password. Your typing wont be reflected on the screen just a heads up but you will be asked for the password twice to be sure you entered it correclty.

Ensure key is not publicly visible
`chmod 400 ~/.ssh/id_ed25519`
Or 
`chmod 400 ~/.ssh/id_rsa`

Navigate to SSH files on macos using Command+Shift+G then input `~/.ssh` in the seach bar. You should see your public and private keys. Guard your private key closely. Your public key can be pasted in your VPS providers "SSH Keys" section when you set up a new VPS. You will also need to paste your public key in the authorized keys file of any new users you create (more on this later).

To edit SSH Config on macos navigate to .ssh using the command above then add the text below to the ssh config file. If file doesnt exist create it using text editor. Include ed25519 or rsa as appicable i.e., if you only set up the ed25519 key dont insert the rsa line.
```
Host *
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519
  IdentityFile ~/.ssh/id_rsa
```

Add keys to keychain in terminal so you dont have to enter password for keys every time you log in VPS. Note, you will need to do this every time you restart your computer.
`ssh-add --apple-load-keychain`

Congrats, you should now have a ssh key-pair. Now time to set up the VPS.

## Server (host) Commands to set up new server
**I cant stress this enough, try this on a test server and get comfortable with it before trying on any production server**

We will be getting updates, adding a new user, adding your pub key to the new user, disabling root login, disabling password login, setting up a firewall, setting up fail2ban, and setting up a swap space for memory. **If you are running a node on Zenon know the service file points to root, so you need to perform zenon actions as the root user. If you follow this guide when you log into your machine, you will log in as the new user. This means when you download zenon, you need to switch to root otherwise your data will not be where zenon expects it to be. You can switch to the root user at any time using the following command `sudo su - root`** 

Follow the steps of whatever VPS provider you choose to set up your machine. Add the public key you generated in the last step to the VPS provider autorized keys. This is a bit different for each provider so I cant give exact steps but it should be intuitive. Once you have your machine up, log in as normal and proceed to the next steps. If you were not prompted for a password when logging in to the VPS congrats, it used your key-pair!

First things first, update the server. You should do this regularly, especially whenever there are security updates.
`sudo apt update && sudo apt upgrade -y`

Alternatively you can enable automatic updates.
`sudo apt install unattended-upgrades`

`sudo apt install update-notifier-common`

`sudo nano -w /etc/apt/apt.conf.d/50unattended-upgrades`

We need to make a few changes to this file. In addition to making the changes below you also need to delete the `//` in front of the line. To set up email notification search for `Unattended-Upgrade: :Mail` and insert your email between the quotations. Next there should be a `MailReport` section and be sure that is set to `on-change` last to allow automatic reboots scroll down to find `Automatic-Reboot` and set it to `true` then press ctrl+o to save and ctrl+x to exit.

Now add your email to the listchanges.conf file
`sudo nano -w /etc/apt/listchanges.conf`

Insert
`email_address=YOUREMAILADDRESS@YOURDOMAIN`

Activate unattended upgrades

`sudo dpkg-reconfigure -plow unattended-upgrades`

Select Yes

To check if this all worked run `sudo nano /etc/apt/apt.conf.d/20auto-upgrades` if the two lines end with `1` that means you are good. `0` means unattended upgrades are disabled.

Now add a user (replace username with whatever name you please)
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

Set firewall permissions (the exact ports you need to open will depend on what you are running on your machine - replace #PORT with the port you need open in the line below - see resources on UFW for more tips here)
`ufw allow #PORT`

Enable UFW
`ufw enable`

Install fail2ban
`apt install fail2ban`

Add swap space (16 GB)
`fallocate -l 16G /swapfile`

Enter the commands below to configure the swap space
```
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```

Edit the fstab file `nano /etc/fstab`

Append the line below, save, and exit
`/swapfile swap swap defaults 0 0`

Check swappiness
`cat /proc/sys/vm/swappiness`

Set swappiness if needed
`sudo sysctl vm.swappiness=25`

Reboot server
`reboot`

Access sudo
`sudo -s`

Confirm UFW
`sudo ufw status`

Confirm fail2ban
`systemctl status fail2ban`

Confirm swap
`free`

If this all worked congrats. This should be a good base to start from. The best thing you can do from here is learn more about securing your machine and stay on top of it as technology evolves. If this didnt work and you think you messed something up leverage the resources to troubleshoot. If this doesnt work and you think I messed something up submit an issue here. If you want some basic Linux commands check the bottom of my tips guide https://github.com/sultanofstaking/Zenon-Testnet-Node-Tips

## If you found helpful no need to donate, but delegation to SultanOfStaking pillar would be appreciated https://explorer.znn.space/pillar/z1qpgdtn89u9365jr7ltdxu29fy52pnzwe4fl7zc
