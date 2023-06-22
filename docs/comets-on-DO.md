**!!!!!!	THIS FILE IS ONLY FOR BOOTING AND RUNNING COMETS   !!!!!!**
**!!!!!!	COMETS ARE TEMP AND USED FOR TESTING PURPOSES      !!!!!!**

---

# DIGITALOCEAN HOSTING
* **->VARIABLES:**
	* **$COMETNAME** = Folder name
	* **$DIGITALOCEANNAME** = This is what you named your droplet
	* **$DIGITALOCEANIPV4** = Your droplet IP

From here we can take our Urbit Comet and host it on Digital Ocean for testing purposes.  For a permanent solution you will want to be using a Planet.

* Create a droplet and load Ubuntu 22.10 x64
* Choose the Shared CPU: Basic plan with 2GB RAM as a minimum.
* NOTE: Urbit needs at least 2GB RAM to run.

In the Authentication field select SSH keys and add new SSH key.  Run the following command in your *LOCAL TERMINAL*.

	$ ssh-keygen

This will generate a public and private SSH key you can use in your droplets.
By default, the keys are stored in the ~/.ssh directory with the filenames id_rsa for the private key and id_rsa.pub for the public key.

Read the file contents from your id_rsa.pub by typing:

	$ cat ~/.ssh/id_rsa.pub

Copy and paste the key in SSH Key Content.  In the name field enter your **$DIGITALOCEANNAME**.

Click on Additional Options and check Add Initialization Scripts (Free) and paste the following script and create your Droplet:

	#!/bin/bash
	
	# configure swap
	fallocate -l 2G /swapfile
	chmod 600 /swapfile
	mkswap /swapfile
	swapon /swapfile
	echo "/swapfile swap swap defaults 0 0" >> /etc/fstab
	
	# setup firewall
	ufw allow OpenSSH
	ufw allow www
	ufw allow https
	ufw allow 34543/udp
	ufw enable
	
	# create and configure user
	useradd -s /bin/bash -d /home/urbit -m -G sudo urbit
	passwd -d urbit
	echo "urbit ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

	# configure ssh keys for user
	mkdir -p /home/urbit/.ssh
	chmod 700 /home/urbit/.ssh
	cp /root/.ssh/authorized_keys /home/urbit/.ssh/authorized_keys
	chmod 600 /home/urbit/.ssh/authorized_keys
	chown -R urbit:urbit /home/urbit/.ssh
	
	# configure sshd
	mkdir -p /etc/ssh/sshd_config.d
	cat > /etc/ssh/sshd_config.d/override.conf <<EOF
	PermitRootLogin no
	PubkeyAuthentication yes
	PasswordAuthentication no
	EOF
	
	# fetch and extract urbit binary
	curl -L https://urbit.org/install/linux-x86_64/latest | tar xzk --transform='s/.*/urbit/g' -C /home/urbit/
	chown urbit:urbit /home/urbit/urbit
	
	# install tmux
	apt -y update
	apt install -y tmux
	
	# reboot
	systemctl reboot

 ---

# CONNECT TO YOUR DROPLET
To connect to your droplet you need to add an alias to ssh/config file.  Open the file in your editor.  I am using Sublime Editor in the example below:

	$ subl ~/.ssh/config

Now add the following to the file and save.

	Host $DIGITALOCEANNAME
  	  HostName $DIGITALOCEANIPV4
  	  User urbit
  	  IdentityFile ~/.ssh/id_rsa
  	  IdentitiesOnly yes

Run the following to ssh into your droplet:

	$ ssh $DIGITALOCEANNAME

---

# BOOTING YOUR COMET IN DIGITAL OCEAN
We are going to activate and run a Comet, on Digital Ocean for testing purposes.
A Comet can be booted with the -c argument.

	$ ./urbit -c $COMETNAME

**$COMETNAME** will be the name given to the folder (AKA "PIER") it will create.
It could take several minutes to initalize.

	->Output
	---------------- playback complete ----------------
	vere: checking version compatibility
	loom: image backup complete
	ames: live on 40019
	conn: listening on /home/urbit/mycomet/.urb/conn.sock
	http: web interface live on http://localhost:8080
	http: loopback live on http://localhost:12321
	pier (26): live
	docket: fetching %http glob for %garden desk
	ship: loading azimuth snapshot (120.197 points)
	ship: processing azimuth snapshot (120.197 points)
	ames: czar zod.urbit.org: ip .35.247.119.159
	ames: metamorphosis
	; ~zod is your neighbor
	~palnyd_binzod:dojo> 

Now we want to mount the filesystem with:

	$ |mount %

---

# DOCK YOUR COMET
Now hit Ctrl-D to exit your Comet.  We need to dock it and run it on port 34543 so we can access it from a browser.
Do so with the following:

	$ ./urbit dock $COMETNAME
	$ sudo setcap 'cap_net_bind_service=+ep' $COMETNAME/.run

# CREATE A SCREEN SO URBIT CAN RUN CONTINUOUSLY
Now that everything is set up we need to create a new screen to keep Urbit running in the background:

	$ screen -S urbit

And start your Comet up with:

	$ ./$COMETNAME/.run -p 34543

# CONNECT TO YOUR COMET VIA BROWSER
Now that you have your Comet up and running you should be able to connect directly by typing your Droplets IpV4 address into your browser
but first you need your login info which can be found by typing:

	$ +code

Once it's up and running press Ctrl + A + D to close the screen.
If you ever need to get back to this screen to find your login code or to make changes use the following:

	$ screen -r urbit

Open in your browser and login with you web login code from above.  This will take you to the Urbit homescreen loaded with
default apps (Groups, Talk, Terminal).

That's it!  You're all set!

---

* Some recommended Groups to join: 

	* ~tocsem-patlet-tobfer-tolmyr--havped-migren-lashes-wanzod/tha-bois ***<- This is a group I started!***
	* ~bitbet-bolbel/urbit-community ***<- General Info***

---

# A TO Z LINUX CLI
	#Setup SSH
		$ ssh-keygen
		$ cat ~/.ssh/id_rsa.pub
		$ subl ~/.ssh/config
	
	#Login and create
		$ ssh $DIGITALOCEANNAME
		$ ./urbit -c $COMETNAME
	
	#Setup Urbit and run
		$ |mount %
		$ ./urbit dock $COMETNAME
		$ sudo setcap 'cap_net_bind_service=+ep' $COMETNAME/.run
		$ screen -S urbit
		$ ./$COMETNAME/.run -p 34543
	
	#Get login code
		$ +code
	
	#Restart Urbit screen
		$ screen -r urbit

---

# ERRORS
It's slow to load!  I believe this is because of my DO droplet settings and I'm not running a planet!
I have pointed my website with a valid SSL cert to DigitalOcean nameservers.
Currently when I open the comet in browser https:// isnt working.  This might be because I'm not using a planet.
This is an issue because certain planets and ships will not communicate with unsecured instance.
Some apps also don't seem to be installing.  From the errors I'm getting in my console I'm assuming this is related to the SSL Certs as well.

---
