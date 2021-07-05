## Introducing Tiger VNC

VNC stands for “Virtual Network Computing” is a sharing system or set of protocols for sharing desktops. There is much software available to access Linux-based desktop remotely including, TigerVNC, TightVNC, Vino, vnc4server, and more.

TigerVNC is a free, open-source, and high-performance VNC server used to control or access Linux-based desktop remotely. It is a client/server application that allows you to interact with graphical applications on remote machines.

## Installing Requirements

By default, Ubuntu Server does not include a Desktop Environment, We will start by getting gnome desktop.

**You should Be logged in to your server terminal using ssh** 

## Hint:

If you are not familiar with nano, 

* `CTRL + X` Will Allow you to save your new file
* `CTRL + SHIFT + V` will paste your copied snippet

## Step 1 - Keep it up to date !

First you will need to update and upgrade your system

```bash
apt update -y && apt upgrade -y 
```

## Step 2 - Install Task Select & Gnome

Task Select will allow you to download and install ubuntu modules without affecting your system, install the Tasksel utility to install a desktop environment:

```bash
apt install tasksel -y 
```

After installing task select, execute it's option menu:

```bash
tasksel 
```

You should see the following interface:

![image](https://user-images.githubusercontent.com/11979856/124403138-1d544800-dd35-11eb-8664-64ed357badf7.png)

Use the arrow key to scroll down the list and find **Ubuntu desktop**. Next, press the `SPACE` key to select it then press the `TAB` key to select OK then hit Enter to install the Ubuntu desktop (Gnome).

Once it finished you need to target your system to boot the graphical interface each time your server reboots:

```bash
systemctl set-default graphical.target  
```

## Step 3 - Install Tiger VNC

Tiger VNC is available in ubuntu default package manager, install it by typing 

```bash
apt install tigervnc-standalone-server -y 
```

Provide you vnc password for your current user:

```bash
vncpasswd 
```

(Optional): you can create a new user for vnc connection which is separated from your current user by typing ``` su - <user name should be here> ```

You should see something similar to this:

```bash
Password:
Verify:
Would you like to enter a view-only password (y/n)? n 
```

Skip the option to enter view-only password by typing `n`.

Start vnc server by typing ``` vncserver -localhost no :<number> ``` replace `<number>` with the instance number you need if you don't know what is that just type

``` vncserver -localhost no ```

Once the VNC server is started, you should get the following output:

```bash
New 'ubuntu2004:1 (tariq)' desktop at :1 on machine ubuntu2004

Starting applications specified in /etc/X11/Xvnc-session
Log file is /home/tariq/.vnc/ubuntu2004:1.log

Use xtigervncviewer -SecurityTypes VncAuth,TLSVnc -passwd /home/tariq/.vnc/passwd ubuntu2004:1 to connect to the VNC server.
```

You can verify your running VNC server using the following command:

``` vncserver -list ```

This will gives you:

```console
TigerVNC server sessions:

X DISPLAY #	RFB PORT #	PROCESS ID
:1		5901		1719
```

## Step 4 - Configure Xterm, xstartup

### Before we start: 
* if you have created a new user you should login to it using ```su - <user name>```
* install xterm using `apt install xterm`


Start by killing your vncserver by typing ``` vncserver -kill :1 ```.

Next, you will need to configure TigerVNC to work with Gnome. You can do it by creating new file xstartup inside .vnc directory:

```nano ~/.vnc/xstartup```

Add the following lines:

```bash
#!/bin/sh
[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
```
then:

``` chmod u+x ~/.vnc/xstartup ```

Next, create the folder by typing `mkdir /etc/vnc` then `nano /etc/vnc/xstartup` and fill it with those lines

```bash
# !/bin/sh

test x"$SHELL" = x"" && SHELL=/bin/bash
test x"$1"     = x"" && set -- default

vncconfig -iconic &
"$SHELL" -l <<EOF
export XDG_SESSION_TYPE=x11
dbus-launch --exit-with-session gnome-session
exec /etc/X11/Xsession "$@"
EOF
vncserver -kill $DISPLAY
```

Then give it executable permission ```sudo chmod u+x /etc/vnc/xstartup```

Next, you will need to create a systemd file for TigerVNC to manage the VNC service. You can create it with the following command:

``` nano /etc/systemd/system/vncserver@.service ```

Add the following lines and make sure to replace `root` with your user name:

```bash
[Service]
Type=forking
User=root
Group=root
WorkingDirectory=/root

PIDFile=/root/.vnc/%H:%i.pid
ExecStartPre=-/usr/bin/vncserver -kill :%i > /dev/null 2>&1
ExecStart=/usr/bin/vncserver -depth 24 -geometry 1360x768 -localhost :%i
ExecStop=/usr/bin/vncserver -kill :%i


[Install]
WantedBy=multi-user.target
```

Save and close your file then reload the configs cache ``` systemctl daemon-reload ```

Next, enable the VNC service to start at system reboot with the following command:

```systemctl enable vncserver@<number>.service``` the phrase `<number>` refers to the instance number you specified when starting your vnc server in step 3.

Next, start the VNC service with the following command:

``` systemctl start vncserver@<number>.service  ```  the phrase `<number>` refers to the instance number you specified when starting your vnc server in step 3

**(Note)**: You should never see any kind of errors after running the previous line, if you do please debug what is happend using ``` systemctl status vncserver@<number>.service  ``` and google it.

keep the `RFB PORT` of the running instance of vnc to use it in the connection step.

To do that run:

``` vncserver -list ```

This will gives you:

```console
TigerVNC server sessions:

X DISPLAY #	RFB PORT #	PROCESS ID
:1		5901		1719
```

## Step 5 - Establish your connection

- SSH to your server terminal with **vnc port forwarding**  ```ssh -i <ssh key location> -L <RFB PORT>:127.0.0.1:<RFB PORT> <VNC User>@<IP>```

- Download VNC Client of your choice I prefer [RealVnc](https://www.realvnc.com/en/connect/download/viewer/)

- Setup a new vnc connection in your vnc client with ```127.0.0.1:<RFB PORT>```.

- Connect and enter your specified vnc password in *step 3*.

![image](https://user-images.githubusercontent.com/11979856/124404971-7031fd80-dd3d-11eb-838a-923d5b342c63.png)

Viola! you got an ubuntu UI in your server!
