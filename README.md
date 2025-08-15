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
