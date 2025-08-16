# Minecraft On Raspberry Pi
Tutorial for running minecraft on the raspberry pi

---

## Step 1
### Setup the raspberry pi

For this project we're using the **Raspberry Pi OS Lite**, First go to https://www.raspberrypi.com/software/ and downlod the imager tool.

When setting up the pi, select the correct model and than select the lite version with no desktop environment. Than Make sure u set a password, wifi and enable SSH.

After the imager is finished boot the pi, now we need to find the IP adress of the pi. u can do this through your routers control pannel or by plugging the pi into a monitor, it should say the IP at the top.
Alternatively u can type ifconfig on the pi's terminal.

Now lets connect to the server by opening the control panel and using ssh:
```
ssh [IP]
```

Replace `IP` with your own ip.

## Step 2
### Install JAVA on the pi

Beceause the pi has an arm processor and uses a special OS we cant install java normally, so we'll need to use something else. And that is [Liberica JDK](https://bell-sw.com/pages/downloads/#jdk-21-lts).

The easiest way to install this is by adding Bellsofts APT repository using:
```
wget -q -O - https://download.bell-sw.com/pki/GPG-KEY-bellsoft | sudo apt-key add -
echo "deb https://apt.bell-sw.com/ stable main" | sudo tee /etc/apt/sources.list.d/bellsoft.list
```

and than install it using:

```
sudo apt update
sudo apt install bellsoft-java21
```

Replace `java21` with the actual version u need, for the latest minecraft versions JDK21 should be fine.

## Step 3
### Setup the server

First we need to acquire the right files, we can get them from https://mcversions.net/.

Download the correct versions server.jar file.

To get the file to your pi were using scp:
```
scp  [filepath] [ip]:[location]
```

For example:
```
scp c://users/user/downloads/server.jar 192.168.0.0:/home/user
```

## Step 4
### Start using the server

Now we start the server using :
```
java -Xmx1024M -Xms1024M -jar server.jar nogui
```

U will get an error saying u should accept the eula first.

We need to accept the eula by using:
```
nano eula.txt
```

Change the value from `False` to `True`

Than save file using `CTRL + O`
Press `Enter`
And exit using `CTRL + X`

If u restart the last command again:
```
java -Xmx1024M -Xms1024M -jar server.jar nogui
```

U should be able to join the server by entering this into minecraft:
```
[IP]:25565
```

---
# Extra
If u wanna host the server online follow these steps.

### Step 5
Forward your ip adress and port using the instructions for your own router. once forwarded u need to find the public ip adress of your raspberry pi using:
```
curl -4 ifconfig.me
```

Once u have that u can go to your domain registar and go to your dns setting.
Create a new A record with the name being your subdomain or `www` if u want to use your main domain.

|Type|Name|IP4|ect...|
|----|----|---|------|
|A|subdmain|XXX.XXX.XXX.XXX (public ip4 adress)||

If u are using cloudflare be sure to disable proxy and use dns only beceause otherwise it will not work.

After that u are going to need a `SRV` record like this:

|Type|Name|Priority|Weight|Port|Target|
|----|----|--------|------|----|------|
|SRV|_minecraft._tcp|0|0|25565 (default)|subdomain.domain.tld|

### Example
I'm using cloudflare so for me it is:
|Type|Name|Priority|Weight|IPv4|Proxy Status|TTL|Port|Target|
|----|----|--------|------|----|------------|---|----|------|
|A|server|-|-|127.0.0.1 (your public adress here)|disabled|2 min (default)|-|-|
|SRV|_minecraft._tcp|0 (default)|0 (default)|-|-|Auto (default)|25565 (default)|server.domain.com|

### Step 6
Auto start server when pi starts.

First install screen:
```
sudo apt update
sudo apt install screen
```

Than lets create the start script:
```
nano start.sh
```

And add following content:
```sh
#!/bin/bash
cd /home/jonas
# Start Minecraft server inside a detached screen session named "mc"
exec screen -DmS mc bash -lc 'exec java -Xmx1024M -Xms1024M -jar server.jar nogui'
```

And make it executable:
`chmod +x [path]/start.sh`

Than create the screen service:
`sudo nano /etc/systemd/system/minecraft.service`

And add:
```sh
[Unit]
Description=Minecraft Server (screen session)
After=network-online.target
Wants=network-online.target

[Service]
User=[username]
Group=[username]
WorkingDirectory=[path]

Type=oneshot
RemainAfterExit=yes

# Clean up any stale "mc" screen (ignore errors if none exist)
ExecStartPre=-/usr/bin/screen -S mc -X quit

# Run the server using the start script
ExecStart=[path]/start.sh

# Stop cleanly by sending "stop<Enter>"
ExecStop=/usr/bin/screen -S mc -p 0 -X stuff "stop$(printf \\r)"
ExecStopPost=-/usr/bin/screen -S mc -X quit

StartLimitIntervalSec=0

[Install]
WantedBy=multi-user.target
```

To enable and start the service add:
```
sudo systemctl daemon-reload
sudo systemctl enable minecraft
sudo systemctl start minecraft
systemctl status minecraft --no-pager
```

Now when u boot the pi the server automatticly starts and whern u use SSH to enter your pi u can manage the server by using `screen -r mc`

And to exit without shutting it down use `CTRL + A` and than press `d`.

---
And thats it now u should be good to go to make your own minecraft server on your raspberry pi!
