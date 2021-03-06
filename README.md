osx-trix
========

Compilation of Patches, Fixes, Tips and Tricks for Apples OS X Platform.

**Pull Requests are welcome.**

# Notes
``$`` means your normal user account.

``#`` means your root account.

## [Finder related](#finder)
## [Display related](#display)
## [Networking related](#networking)
## [Animation related](#animations)
## [Free Apps](#useful-apps-free)
## [Terminal related](#terminal)
## [git related](#git)

# Finder
After any change you made here you have to type *killall Finder*
## show 'Quit' in the Menubar
This will enable CMD + Q and in the menubar Finder -> Quit Finder

    $ defaults write com.apple.Finder QuitMenuItem 1

tested for 10.9

## show extended save dialog via default
This will always bring up the extended save dialog in all applications.

```
$ defaults write -g NSNavPanelExpandedStateForSaveMode -bool TRUE
```
You need to restart your machine afterwards.


## show extended print dialog via default
This will always bring up the extended print dialog in all applications.

```
$ defaults write -g PMPrintingExpandedStateForPrint -bool TRUE
```
You need to restart your machine afterwards.

## show hidden files in Finder
This wil show all files in the Finder.

```
$ defaults write com.apple.finder AppleShowAllFiles TRUE
```

tested for 10.4 - 10.9

## show a specific hidden file/directory in Finder
This will show a specific file/directory in the Finder. e.g. The directory ~/Library/.

```
$ chflags nohidden ~/Library/
```

necessary since 10.7

## show full POSIX Path in title
This will enable the full POSIX path in any Finder menu.

    $ defaults write com.apple.finder _FXShowPosixPathInTitle -bool true
    
tested for 10.7 - 10.9

## prevent .DS_Store file creation over network connections
This will prevent the Finder from creating .DS_Store over network connections.

    $ defaults write com.apple.desktopservices DSDontWriteNetworkStores true
    
tested for 10.4 - 10.9

# Preparing your System for HDD + SSD usage
This will disable Spotlight on your selected Drive and it will stop spinning after 1 minute (for this the option must be first activated in energy preferences). Useful for Macs with HDD and SSD.
```
# mdutil -d /Volumes/diskX
# mdutil -i off /Volumes/diskX
# pmset -a disksleep 1
```

# Testing your Memory
Source: http://rampagedev.wordpress.com/os-x-tweaks/run-memtest-under-mac-os-x/
## with normal RAM usage
1. Download Memtest and install
2. type *memtest all 2* this will test your whole unused memory 2 times

## with minimal RAM usage (recommend)
1. Download Memtest and install
2. restart your Mac and hold down cmd+s while starting
3. Type *memtest all 6* into prompt. To save the results into .txt file you need to mount your volume with 
*# /sbin/mount -uw /* and then run *memtest all 6 > output.txt* this will save all results to the root of the volume.

# Display
Source: https://pikeralpha.wordpress.com/2017/01/30/4398/

enable Night Shift on older machines.
```

# cd /System/Library/PrivateFrameworks/CoreBrightness.framework/Versions/Current
# nm CoreBrightness |grep _ModelMinVersion
000000000001e260 S _ModelMinVersion
```
0x1e260 is the starting position add 4 to the offset to access all machine types. offset 0 is MacBookPro, followed by iMac, Macmini, MacBookAir, MacPro and MacBook. We use node to rewrite the CoreBrightness binary for the MacBook Pro (offset 0).
```
# node
> const a = fs.readFileSync('CoreBrightness');
> a[0x1e260]
9
> a[0x1e260] = 8;
> fs.writeFileSync('CoreBrightness', a);
> .exit
```
Now we resign the binary:
```
# codesign -f -s - /System/Library/PrivateFrameworks/CoreBrightness.framework/Versions/Current/CoreBrightness
```
Don't forget to reboot your machine.

# Networking

## SMB / CIFS (samba)

generally use `cifs://` instead of `smb://` <https://discussions.apple.com/thread/6635652?start=0&tstart=0>

### speedup connection
This will speedup smb (samba) connections. Write a file (as root) with:

```
[default]
minauth=none
streams=no
soft=yes
notify_off=yes
port445=no_netbios
```

to

```
/private/etc/nsmb.conf
```

tested for 10.9

## throttle TCP/IP Port throughput
* (src|dst) means src-port xor dst-port
* replace {port} with the port number you want to throttle

```
# ipfw pipe 1 config bw 500KByte/s
# ipfw add 1 pipe 1 (src|dst)-port {port}
# ipfw delete 1
```
tested for 10.8 - 10.9

## disable delayed tcp ack
Source: <http://www.jeremycole.com/blog/2010/01/13/delayed-ack-in-os-x-is-incomprehensible/>
```
# echo 'sysctl -w net.inet.tcp.delayed_ack=0' >> /etc/sysctl.conf
```

## keep bluetooth disable on startup
Source: http://www.kernelcrash.com/blog/os-x-disabling-bluetooth/2007/11/12/
```
# defaults write /Library/Preferences/com.apple.Bluetooth “ControllerPowerState” -bool FALSE
```


# Editing Apple-Apps

## Enable diskutility debug-menu

    $ defaults write com.apple.DiskUtility DUDebugMenuEnabled 1
    
## codesign
As soon as you edit any .plist etc. in an original Apple-App it crashes after editing the .plist, because every App is signed by Apple. To sign the edited App (e.g *Boot Camp Assistant*) get root and type:

    # codesign -fs - /Applications/Utilities/Boot\ Camp\ Assistant.app

only necessary for 10.9

## Make BootCamp Assistant create a bootable USB-drive on Mac's with optical drive
Source: https://www.youtube.com/watch?v=5A5FG8EMEGc

1. Edit the info.plist located in: */Applications/Utilities/Boot Camp Assistant.app/Contents*.
2. Search for the string *<key>PreESDRequiredModels</key>* and paste your Model Identifier e.g. *<string>MacBookPro7,2</string>* (You can find the Identifier in Mactracker or by left clicking the -Menu while holding down the "alt" Key and clicking "Systeminformation")
3. Copy your Boot-Rom-Version form the System-Profiler, which you start by left clicking the -Menu while holding down the "alt" Key and clicking "Systeminformation" and paste it in the info.plist at *<key>DARequiredROMVersions</key>*
4. To make your Mac boot from USB just delete the "Pre" from *<key>PreUSBBootSupportedModels</key>* and add your Model Identifier
5. Codesign your BootCamp Assistant

# Animations

## disable Mission Control animation
This disables the swoosh animation when you start Mission Control.
```
$ defaults write com.apple.dock expose-animation-duration -float 0
$ killall Dock
```

## disable Launchpad  animations
source: http://apple.stackexchange.com/questions/117223/how-do-i-disable-launchpad-animation-on-mac
```
$ defaults write com.apple.dock springboard-page-duration -float 0
$ killall Dock
```

## Install 10.x.app to USB thumb drive
source: http://coolestguidesontheplanet.com/making-a-boot-usb-disk-of-osx-10-9-mavericks/

```
10.x.app/Contents/Resources/createinstallmedia --volume /Volumes/{yourVolume} --applicationpath 10.x.app --nointeraction
```

## Install 10.11 installer to USB thumb drive
source: http://www.macworld.com/article/2981585/operating-systems/how-to-make-a-bootable-os-x-10-11-el-capitan-installer-drive.html

`/Applications/Install\ OS\ X\ El\ Capitan.app/Contents/Resources/createinstallmedia --volume /Volumes/{yourVolume}/ --applicationpath /Applications/Install\ OS\ X\ El\ Capitan.app/ --nointeraction`

## get TeamViewer.app from Teamvier.dmg
<https://apple.stackexchange.com/questions/15658/how-can-i-open-a-pkg-file-manually> <http://www.linuxquestions.org/questions/linux-newbie-8/how-to-extract-a-ascii-cpio-archive-svr4-with-no-crc-4175436617/>

* copy `Install TeamViewer.pkg` to a writeable place
* `cd` to the directory where `Install TeamViewer.pkg` is located
* exec `pkgutil --expand Install\ TeamViewer.pkg viewer` this will extract the pkg
* exec `cd viewer/TeamViewerApp.pkg`
* exec `mv Payload Payload.gz` rename Payload to Payload.gz
* exec `gunzip Payload.gz` gunzip the Payload
* exec `cpio -i -F Payload` finaly unpack the App
* exec `mv TeamViewer.app ~/Downloads` move the extracted App to the Downloads directory

# Useful Apps (free)
## Boot Camp 5
http://support.apple.com/kb/dl1638

## flux
http://justgetflux.com/

Changes the kelvin-temperature of your display when the sun goes down.

> Ever notice how people texting at night have that eerie blue glow?

> Or wake up ready to write down the Next Great Idea, and get blinded by your computer screen?

> During the day, computer screens look good—they're designed to look like the sun. But, at 9PM, 10PM, or 3AM, you probably shouldn't be looking at the sun.

> f.lux fixes this: it makes the color of your computer's display adapt to the time of day, warm at night and like sunlight during the day.

> It's even possible that you're staying up too late because of your computer. You could use f.lux because it makes you sleep better, or you could just use it just because it makes your computer look better.

## img2icns
Transforms images to icons.

## Rufus
Create bootable USB-drives form any .iso.
https://rufus.akeo.ie/

## iTerm (Version 2)
http://www.iterm2.com/

> What is iTerm2?

> iTerm2 is a replacement for Terminal and the successor to iTerm. It works on Macs with OS 10.5 (Leopard) or newer. Its focus is on performance, internationalization, and supporting innovative features that make your life better.

## TextMate
http://macromates.com/download

Improved text displaying with syntax highlighting. It's the Notepad++ equal for OS X.

## 0xED, HexEditor
http://www.suavetech.com/0xed/
> 0xED is a native OS X hex editor based on the Cocoa framework.


## BetterTouchTool
http://www.boastr.de
> BetterTouchTool is a great, feature packed FREE app that allows you to configure many gestures for your Magic Mouse, Macbook Trackpad and Magic Trackpad. It also allows you to configure actions for keyboard shortcuts, normal mice and the Apple Remote. In addition to this it has an iOS companion App (BTT Remote) which can also be configured to control your Mac the way you want.

> BetterTouchTool includes many goodies, like window snapping or an integrated window switcher.

## Memtest
http://cdn.command-tab.com/2008/memtest_422.zip (direct)

## buildin http server
source: <http://osxdaily.com/2010/05/07/create-an-instant-web-server-via-terminal-command-line/>

just run
```
$ python -m SimpleHTTPServer 4104
```
in your terminal. It will serve the files and directories in the current directory.

## VLC (media player)
https://videolan.org

> VLC is a free and open source cross-platform multimedia player and framework that plays most multimedia files as well as DVD, Audio CD, VCD, and various streaming protocols.

## KeepassX (secure password storage)
https://www.keepassx.org/downloads/

> KeePassX is an application for people with extremly high demands on secure personal data management. It has a light interface, is cross platform and published under the terms of the GNU General Public License.

# Peripherals

## Prevent Volume from mounting
Source: http://www.cnet.com/how-to/prevent-a-partition-from-mounting-in-os-x/

1. Get the UUID from the Volume by running: *diskutil info {pathToVolume}*.
2. *# pico /etc/fstab*
3. Type: *# UUID=$NUMBER none hfs rw,noauto* for any other formatted drive you replace "hfs" with for example "FAT".

## Make the SuperDrive work on any Mac
Source: http://www.maclife.de/tipps-tricks/hardware/externes-superdrive-fast-jedem-mac-verwenden

1. Copy the file */Library/Preferences/SystemConfiguration/com.apple.Boot.plist* to your desktop and open it with TextEdit.
2. paste "mbasd=1" bewteen *&lt;string&gt;* and *&lt;/string&gt;*. Save the document and paste it into */Library/Preferences/SystemConfiguration/* replace the old file. Root is needed.
3. Restart your Mac.

## Keyboard changes &gt;/&lt; with ^° Keys
```
# rm /Library/Preferences/com.apple.keyboardtype.plist
```
then reboot

# Clearing Caches
None of these steps will harm your settings or something like that. Everything listet below is to delete temporary files which can removed without harming the System or other applications.
## Rebuild Kext-cache
Source: http://forums.macrumors.com/archive/index.php/t-1654766.html

This usually fixes slow boot ups.
```
# kextcache -system-prelinked-kernel
# kextcache -system-caches
```

## Clearing sandbox containers
Note: this will delete all configs from sandboxed apps

Empty the directory: *~/Library/Containers*

## Removing User-Cache
Just delete the content of the directory: *~/Library/Caches*

## Clearing Font-Cache
Source: http://www.extensis.com/support/de/knowledge-base/clearing-the-mac-os-x-font-cache/
```
# atsutil databases -remove
```

## Clearing DNS-Cache
Source: http://reviews.cnet.com/8301-13727_7-57493543-263/how-to-reset-the-dns-cache-in-os-x/

```
# dscacheutil -flushcache
# killall -HUP mDNSResponder
```

## Clearing User-temp
Just delete the content of the directory: */private/var/folders*

## Clearing application-states
Empty the directory *~/Library/Saved Application State*


# disable bluetooth via terminal
Source: https://discussions.apple.com/message/12448781
```
#set bluetooth pref to off
defaults write /Library/Preferences/com.apple.Bluetooth.plist ControllerPowerState 0

#set bluetooth pref to on 
defaults write /Library/Preferences/com.apple.Bluetooth.plist ControllerPowerState 1

#kill the bluetooth server process 
killall blued

#unload the daemon 
launchctl unload /System/Library/LaunchDaemons/com.apple.blued.plist

#reload the daemon
launchctl load /System/Library/LaunchDaemons/com.apple.blued.plist

#restart blued daemon
launchctl start com.apple.blued
```

## Rebuild Spotlight index

Open the system preferences go to Spotlight and add your whole volume to the exclude list. Then wait a few seconds and delete it again. Open Spotlight to see the indexing-process.


# Terminal

## send display to sleep
works since 10.9
```
$ pmset displaysleepnow
```

## remove file secure

Source: http://www.macworld.com/article/3005796/operating-systems/how-to-replace-secure-empty-trash-in-os-x-el-capitan.html

    srm -zv /{path-to-file}



## show my group memberships
```
$ groups {username}
```
The admin user has among others the group ``admin``

The root user has among others the groups ``wheel`` ``daemon`` ``kmem`` ``sys`` ``operator``


## getting Display EDID as hex
```
$ ioreg -l | grep -5 IODisplayEDID
```
The EDID informations are between &lt;&gt; and decoded in hex

# Performance issues

## Finder is slow on user actions / coreservicesd high cpu usage
source: <http://blog.hsoi.com/2014/02/25/my-slow-mac-mavericks-coreservicesd-iconservicesagent-and-how-fs_usage-saved-me/>

see which app is called do get icon information

    # fs_usage -f pathname -w com.apple.IconServicesAgent | grep open


# Security
## Configuring System Integrity Protection
<https://developer.apple.com/library/mac/documentation/Security/Conceptual/System_Integrity_Protection_Guide/ConfiguringSystemIntegrityProtection/ConfiguringSystemIntegrityProtection.html>
reboot with `Cmd + R`, open up terminal and type `csrutil enable` to enable and `csrutil disable` to disable. Status via `csrutil status`

# git
## detect case in filename change

    $ git config core.ignorecase false

now git detects casechanges
