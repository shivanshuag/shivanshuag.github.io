+++
Categories = ["Hacking"]
Description = "Switching to a tiling window manager"
Tags = ["i3wm", "i3", "tiling window manager", "gnome"]
date = "2014-01-27T13:35:54+05:30"
menu = "main"
title = "Gnome 3 to i3"
+++

I have been a linux user for more than two years now and the search for a perfect desktop environment is still on. I have tried many of the popular ones - Unity, Gnome 2 and 3, KDE, Cinnamon and MATE. Each of them have a different philosophy which is suitable for some type of users but all of them are either very heavy-weight or have a some very frustrating bugs. So, I decided to try a new window manager instead of a Desktop Environment and switched to [i3](http://i3wm.org/).

## Why I switched to i3

i3 is a very light-weight tiling window manager. Since quite some time I had started feeling all the  panels, the system menus, the starters, the status applets, desktop background, drag-drop functionality, border decoration and animations which Desktop environments provide to be useless. These functionalities consumed a lot of resources and slowed down my system. Moreover, more the functionalities, more are the bugs. I could do away with all those functionalities. I just needed a Window Manager. [This](http://adereth.github.io/blog/2013/10/02/why-you-should-try-a-tiling-window-manager/) blog post on using a tiling wm also motivated me.

Tiling Window manager can increase your efficiency by providing a lot of keyboard shortcuts. They make you less dependent on mouse. It is hard to get used to at first but you get used to it eventually.

They are heavily customizable. To start with, you can provide your own keyboard shortcuts but essentially you can make your own environment using all the apis and addons available for the window manager. 


There are many tiling wm available out there like [awesome](http://awesome.naquadah.org/), [notion](http://notion.sourceforge.net/) but I chose i3 because of its popularity and because some of my friends use it.

Following is a breif tutorial on things I found necessary to learn being a newbie to tiling wm.

## Installing i3
For ubuntu,

{{< highlight bash >}}
echo "deb http://debian.sur5r.net/i3/ $(lsb_release -c -s) universe" >> /etc/apt/sources.list
apt-get update
apt-get --allow-unauthenticated install sur5r-keyring
apt-get update
apt-get install i3
{{< /highlight >}}

## Getting Used to i3

A good place to start is [i3 User's Guide](http://build.i3wm.org/docs/userguide.html). But it contains a lot of information which can be overwhelming at times.

When you first start i3, it asks if you want to use the default config(default config file is located in `/etc/i3`) or create a new config in `~/.i3`. I chose new config and it created a new config file. To edit the default config, this file needs to be updated. Then it asks for a mod key giving Alt key and Window key options. I preferred Alt Key. 

### Some Indispensable Shortcuts

* Alt + Enter - open Terminal
* Alt + D - menu to open applications(it lists only those apps which are in your path)
* Alt + Shift + Q - Close a window
* Ctrl + W - Close a window which has a cross on the top right to close it(generally browser tabs)
* Alt + 2 - a new workspace(use any number instead of 2)
* Alt + Shift + 2 - move the focused window to second workspace
* Alt + Shift + C - reload config file
* Alt + arrow keys - switch through windows in the same workspace

These are default shortcuts and can be changed by updating the config file.

### Tabbed Layout
By default when you open a new window, the current window splits either horizontally or vertically to accommodate the new one. I prefer tabbed layout to horizontal/vertical or stacked layout. Use `Alt + W` to switch to tabbed layout.

### Volume Control
There is no volume control panel by default in i3. You can bind your own shortcuts to volume control. Modify your config to include the following

{{< highlight bash >}}
bindsym XF86AudioLowerVolume exec /usr/bin/pactl set-sink-volume 0 -- '-5%'
bindsym XF86AudioRaiseVolume exec /usr/bin/pactl set-sink-volume 0 -- '+5%'
{{< /highlight >}}

Here the XF86AudioLowerVolume and XF86AudioRaiseVolume are the multimedia keys on your keyboard for volume control. `bindsym` is used to bind a key to a command. `exec /usr/bin/pactl set-sink-volume 0 -- '-5%'` is to execute the `pactl` command on keypress. `pactl` command is for controlling a running pulseaudio server. So, above command only works when you have pulseaudio installed which is default in ubuntu.

*To reload the config file after making changes, use Alt + Shift + C*

### Screen Lock

For screen lock, I use `gnome-screensaver`. Gnome Screensaver locks the screen on  `gnome-screensaver-command -l` command. I want my screen to lock automatically after 1 minute of inactivity. For this, I use `xautolock`.

{{< highlight bash >}}
exec xautolock -time 1 -locker screenlock
{{< /highlight >}}

The above line in my config starts `xautolock` which executes the screenlock script after 1 minute of inactivity. Screenlock script contains the `gnome-screensaver-command -l` command.

I also  to bind `Alt + Ctrl + l` shortcut to lock the screen. 
{{< highlight bash >}}
bindsym Control+Mod1+l exec gnome-screensaver-command -l
{{< /highlight >}}

### Reboot, Suspend and Shutdown

The commands for shutdown is `sudo shutdown -Ph now`, for reboot is `sudo reboot` and for suspend is `sudo pm-suspend`.
Since all these commands require root access and hence password, it will not be possible for i3 to bind them to keys normally. We need to create a script for each of these commands and add a no-password entry for them in sudoers file. Run `sudo visudo` and add the following lines at the end of the file. Adding them at the end is important.

{{< highlight bash >}}
shivanshu ALL=(ALL) NOPASSWD: /home/shivanshu/.i3/shutdown
shivanshu ALL=(ALL) NOPASSWD: /home/shivanshu/.i3/reboot
shivanshu ALL=(ALL) NOPASSWD: /home/shivanshu/.i3/suspend
{{< /highlight >}}

Here, shutdown, reboot and suspend are scripts which contain the corresponding commands. Then, we can bind these scripts to keyboard shortcuts in the config file

{{< highlight bash >}}
bindsym Control+Mod1+s exec gnome-screensaver-command -l && sudo /home/shivanshu/.i3/suspend
bindsym Control+Mod1+x exec sudo /home/shivanshu/shutdown
bindsym Control+Mod1+r exec sudo /home/shivanshu/reboot
{{< /highlight >}}

Notice that I have also locked the screen before suspending the system so that it requires password on wakeup.

### Battery Low Warning
I want a warning when my battery is low. For this, I have used a script I found on archlinux forum's i3 thread. This script uses i3-nagbar to give warnings and needs acpi installed. It also need root permissions to run, so add a no-password entry for it in the sudoers file. Then add it in the config file.

<script src="https://gist.github.com/shivanshuag/8648921.js"></script>

I have added line 34 and 51 to the script to enable the power saving mode when my laptop is running on battery. You need `pm-utils` for these commands. 

You can find my config file [here](https://gist.github.com/shivanshuag/8614576). I will keep on updating the changes I make to my config.
I hope that i3 proves to be that perfect desktop environment I have been searching for although I think I can customize it to do whatever I want :D