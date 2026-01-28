---
title: "Metasploitable Tikiwiki"
date: 2025-02-25 10:00:00 +0000
categories: [Writeups, Labs]
tags: [metasploit, tikiwiki, intrusion penetration]
author: "JT"
pin: false
toc: true
comments: true
---

## Setting up metasploitable box
### Getting it to boot
Fixing the kernel panic when launching metasploitable box

1. Edit boot options by pressing `esc` before boot
2. Press `e` to edit boot options in grub
3. Modify kernel boot options by pressing `e` on the line beginning with kernel and adding `noapic` to the end after a space
4. Press `b` to boot

Login with
User: `msfadmin`
Password: `msfamin`

### Fixing it permanently
After you have logged in let's change the grub boot options permanently.

Use your preferred text editor to open the following file:

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250213135635.png)


Modify the kernel line, appending `noapic` to the end:

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250213141531.png)


Save the changes and the problem has been solved!

## Fixing Network Configuration Issues
After getting both boxes online, I ran into another problem... both boxes had the same internal IP.

To fix this follow the next steps in order to create a subnet for our two VMs.

Go to the tools section on VirtualBox

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250213141036.png)

Create a NatNatwork

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250213141029.png)


Go to the network settings of **both** of your VMs and add them to your new NatNetwork. 

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250213141207.png)


Relauch your boxes and confirm that they have different IPs, and you can reach them from the other box.

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250213140925.png)


![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250213140940.png)


![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250213141341.png)


Now we should be all setup and can actually start the lab.

## Starting the lab
Let's start by enumerating the attack surfaces. 

Let's use `nmap` to scan all open ports on the subnet.


![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250213141712.png)


![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250213141724.png)


With this the open `80/http` port on our target IP `10.0.2.5` is very interesting. Let's check if we can access it via firefox:

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250213141952.png)



![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250213141938.png)


It works! Now let's continue enumerating the attack surface with sub-domain enumeration using `dirbuster`. 

In kali we can run this very simply:

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250213142320.png)


Once we are in dirbuster we can select the target IP and the wordlist we will use for enumeration. We will also untick the `recursive` and `Brute force files` settings for the sake of time.

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250213143055.png)



After letting dirbuster run for a few minutes we get some hits that are worth investigating 

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250213143547.png)


Let's start with the main tikiwiki directory

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250213142828.png)



![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250213143654.png)


It opened up! That means that tikiwiki is present on our target machine. Let's now proceed using the `msfconsole` to exploit it.

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250213143803.png)


Let's search for any known modules that we can use on tikiwiki

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250213143917.png)


This is good, there are some modules we can possible use to exploit the target machine.

Let's use the `auxiliary/admin/tikiwiki/tikidblib` module

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250213144124.png)


Now it is as simple as setting our remote host using `RHOST <target-ip>`. Once it is set we can just run `exploit`. 

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250213144220.png)


Awesome, now we know the type of the database, the name, where it is hosted and the login credentials. That was easy..

Now with all this info let's login with `root:root`

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250213144533.png)


It worked and we can see all the databases.

Let's change our db to `tikiwiki195` and see the tables.

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250213145334.png)


The `users_users` table stands out as something that could be useful, lets take a look.

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250213145503.png)


This is good, but let's make a better query to get some more useful information. Let's look for a login we can use 

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250213145550.png)


After login with these credentials we are prompted to change our password, and so I did to `admin123`.

Now we are logged in as admin:

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250213145818.png)


The next step is to do privilege escalation to become `root`

We will be using the root shell found at: https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php



![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250213150117.png)



![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250213150301.png)


Modify the reverse shell so it points back to our attacker machine.

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250214141036.png)


Upload the file to tikiwiki. We can get it to run by vising the IP that it is hosted at `http://<target-ip>/tikiwiki/backups/<reverse-shell-file>`

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250213164535.png)


Start netcat to listen for incoming connection on the port specified in the reverse shell, this will give us our reverse shell.

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250214141244.png)


![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250214141231.png)


![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250214141434.png)


This is good we have a reverse shell into the system, however we are not root, so let's escalate our privilege. Start by going back to the `msfconsole` and searching for tikiwiki. 

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250214141825.png)

We will want to use the `tikiwiki_graph_formula_exec` for escalation

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250214142016.png)

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250214142048.png)


![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250214142129.png)

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250214142203.png)


Just like that we have a reverse shell using metasploit, however we are still not root. Lets leak the **ssh private key**, which when paired with the public key could give us root acces.

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250214142245.png)


Now we will download a list of public and private key pairs from: https://gitlab.com/exploit-database/exploitdb-bin-sploits/-/raw/main/bin-sploits/5622.tar.bz2

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250214144442.png)


It downloaded a tar archive, lets unarchive it and grep to look for our known private keys public key.

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250214144521.png)


Now we can grep the key pairs for our private key. (I could have done the whole key, but couldn't copy and paste so I just used the start)

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250214144743.png)


We found a public key associated with our private key. Now we can attempt to connect via `ssh` and see what happens

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250214144853.png)

![Alt Text](/assets/images/SeedsLabs12Imgs/Pasted image 20250214144913.png)


Magical, just like that we have root access to the machine and it is fully exploited. If this was a real target we could now execute our post exploitation to ensure persistence. 
