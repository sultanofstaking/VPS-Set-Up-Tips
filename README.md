# VPS Set-Up Tips

## I created this guide to provide anyone running a node with some basic resources and tips to set up a secure VPS. There are many things you can do to secure your VPS and everyone approaches this a little differenlty. This is meant to give you a starting point. 

## When first learning how to secure a VPS it is not uncommon to lock yourself out. I have included text where possible to help prevent this from occuring, but to be safe I would try this guide on a test machine and get comfortable with your personal set-up before replicating on any production machine. The tips below are my own and not endorsed by any team. Please use the issue feature for fixes here. **Try at your own risk.**

## If you are new to virtual machines then you are probably wondering where to start. Google "VPS provider" and a myriad of results will pop up. You will want to find a provider that is cheap with a friendly user interface to learn on. I launch most of my test instances on Digital Ocean. If you would like to try them out for yourself feel free to use my referral code for a $100 credit over 60 days https://m.do.co/c/ab6f82b1c45f. Another great option is AWS as it has a free tier. AWS is a bit more complex to navigate because they have a ton of great features (features you probably dont need if you are just getting started), but once you figure it out the free tier is well... free. Keep in mind you will likely need to upgrade from free tier however if you wish to run production nodes.

## Set Up SSH Keys on client
We will start with setting up your ssh keys. If you have already done this, skip to setting up the server.

Open Terminal

Generate ssh key (ed25519 is preferred - read Secure SSH resources above for more perspective on this)
`ssh-keygen -t ed25519 -o -a 100`
Or
`ssh-keygen -t rsa -b 4096`

Accept the default path to save.

Use a strong password. Your typing wont be reflected on the screen just a heads up but you will be asked for the password twice to be sure you entered it correclty.

Ensure key is not publicly visible
`chmod 400 ~/.ssh/id_ed25519`
Or 
`chmod 400 ~/.ssh/id_rsa`

**For macos** navigate to SSH files by pressing Command+Shift+G then input `~/.ssh` in the seach bar. You should see your public and private key files. Guard your private key closely. Your public key can be pasted in your VPS providers "SSH Keys" section when you set up a new VPS. You will also need to paste your public key in the authorized keys file of any new users you create (more on this later).

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

## Set Up Server (host)
**I cant stress this enough, try this on a test server and get comfortable with it before trying on any production server**

We will be getting updates, adding a new user, adding your pub key to the new user, disabling root login, disabling password login, setting up a firewall, setting up fail2ban, and setting up a swap space for memory. 

Follow the steps of whatever VPS provider you choose to set up your machine. Add the public key you generated in the last step to the VPS provider autorized keys. This is a bit different for each provider so I cant give exact steps but it should be intuitive. Once you have your machine up, log in as normal and proceed to the next steps. If you were not prompted for a password when logging in to the VPS congrats, it used your key-pair!

## Get updates
First things first, update the server. You should do this regularly, especially whenever there are security updates.
`sudo apt update && sudo apt upgrade -y`

Alternatively you can enable automatic updates.
`sudo apt install unattended-upgrades`

`sudo apt install update-notifier-common`

`sudo nano -w /etc/apt/apt.conf.d/50unattended-upgrades`

We need to make a few changes to this file. In addition to making the changes below you also need to delete the `//` in front of the line. Delete `//` in front of `Remove-Unused-Kernel-Packages "true"` and `Remove-Unused-Dependencies "true"` last to allow automatic reboots scroll down to find `Automatic-Reboot` and set it to `true` then press ctrl+o to save and ctrl+x to exit.

Activate unattended upgrades

`sudo nano /etc/apt/apt.conf.d/20auto-upgrades` 

Paste the following in the file if not already there

```
APT::Periodic::Update-Package-Lists "1";
APT::Periodic::Unattended-Upgrade "1";
```

You can check unattended upgrade logs with `sudo cat /var/log/unattended-upgrades/unattended-upgrades.log`

## Add a new user 

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

**DO NOT LOG OUT OF ROOT. Open a new terminal window and try to log in as the root user. If this worked you will be denied access. Now try to log in with new user you just created ssh username@ip. If all works as planned you will log in with your new user utilizing your ssh keypair.**

Access root with new user
`sudo -s`

## Set up firewall and fail2ban

Enable UFW
`ufw enable`

Check ufw status `ufw status` or `ufw status verbose`

Set firewall permissions (the exact ports you need to open will depend on what you are running on your machine - replace #PORT or #IP with the port or IP you need open in the line below - see resources on UFW and fail2ban above for more tips)

`ufw allow #PORT` or `ufw allow from #IP`

To allow traffic out only `ufw allow out #PORT` 

To delete rules `ufw status numbered` then `ufw delete #NUMBER`

Install fail2ban
`apt install fail2ban`

## Add a swap space

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

## Confirm everything is working

Access sudo
`sudo -s`

Confirm UFW
`sudo ufw status`

Confirm fail2ban
`systemctl status fail2ban`

Confirm swap
`free`

To check CPU / Memory usage on your machine you can use `htop`

To check disk usage on your machine you can use `df -h`

To free up storage space try `journalctl --vacuum-time=1h` to remove log files or `apt autoremove` to remove unused dependencies.

If this all worked congrats. This should be a good base to start from. The best thing you can do from here is continue to learn more about securing your machine and stay on top of it as technology evolves. If this didnt work and you think you messed something up leverage the resources at the top to troubleshoot. If this doesnt work and you think I messed something up submit an issue here and I will fix it. If you want some basic Linux commands check the bottom of my tips guide https://github.com/sultanofstaking/Zenon-Testnet-Node-Tips

## If you found helpful no need to donate, but delegation to SultanOfStaking pillar would be appreciated!
