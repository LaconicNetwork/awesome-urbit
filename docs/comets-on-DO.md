# Deploy an Urbit Comet on Digital Ocean

The following tutorial explains how to deploy a comet on Digital Ocean.

It assumes that you:
- have a Digital Ocean account and familiarity with creating Droplets
- some familiarty with the command line terminal

* Create a droplet with the latest Ubuntu
* Choose the Shared CPU: Basic plan with 2GB RAM as a minimum.
* NOTE: Urbit needs at least 2GB RAM to run.


## Setup local SSH key

Skip this step if you already have one located at `~/.ssh/id_rsa.pub`. Otherwise, in your local terminal, run: 

```
$ ssh-keygen
```

Get the public key contents with:

```
$ cat ~/.ssh/id_rsa.pub
```

Copy the output and add it to your Digital Ocean account. 

## Setup your Droplet

- choose a Shared CPU: Basic Plan with at least 2GB of RAM
- ensure your SSH key is set
- give the droplet a unique hostname (`<YOUR_DROPLET_NAME>`)
- click on Additional Options, check **Add Initialization Scripts (Free)**, and paste the following script:
- finally, create your droplet and note its IP (`<YOUR_IP>`) for later

```
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
```

## Connect to your Droplet


On your local machine, open `~/.ssh/config` with the editor of your choice and add the following lines to the file:

```
Host <YOUR_DROPLET_NAME> 
HostName <YOUR_IP> 
User urbit
IdentityFile ~/.ssh/id_rsa
IdentitiesOnly yes
```

save and exit, then:

```
$ ssh <YOUR_DROPLET_NAME>
```

## Booting Your Comet

Set an env var for you comet name.

```
$ export COMET_NAME=my-fist-comet
```

To activate and boot a new comet, run:

```
$ ./urbit -c $COMET_NAME
```

**Note:** This will create a directory (i.e., a `pier`) at `$COMET_NAME`.,

This step can take several minutes to initialize. The tail end of the output should look like:

```
TODO add full output
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
```

Finally, mount the filesystem with:

```
$ |mount %
```

## Dock And Run Your Comet

Now hit Ctrl-D to exit your Comet. We then need to dock it before running it on port 34543, to access it from a browser.

Do so with the following:

```
$ ./urbit dock $COMET_NAME
$ sudo setcap 'cap_net_bind_service=+ep' $COMET_NAME/.run
```

Use `screen` to run Urbit in the background

```
$ screen -S urbit
```

Start your Comet up with:

```
$ ./$COMET_NAME/.run -p 34543
```

## Connect to your Comet in the Browser

Now that you have your Comet up and running you should be able to connect directly by typing your into your browser
but first you need your login info which can be found by typing:

```
$ +code
```

Once it's up and running press Ctrl + A + D to close the screen.
If you ever need to get back to this screen to find your login code or to make changes use the following:

```
$ screen -r urbit
```

Open in your browser and login with you web login code from above.  This will take you to the Urbit homescreen loaded with
default apps (Groups, Talk, Terminal).

That's it!  You're all set!


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

## Next Steps

Apps, TODO, osmosis
