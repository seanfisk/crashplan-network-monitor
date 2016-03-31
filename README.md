<!-- -*- coding: utf-8; -*- -->

CrashPlan Network Monitor for OS X
==================================

[CrashPlan][] is a great automated backup tool. In fact, I strongly believe that everyone should use some sort of automated backup. However, there are some instances where CrashPlan should not run.

One of these instances is when your laptop is [tethered][] to your phone and you have a limited data plan. Tethering is great because it allows most applications access to the Internet transparently, as if they were using Ethernet or Wi-Fi. This applies to CrashPlan too, which will happily run large backups while burning through your limited data. This is not CrashPlan's fault, but just how these technologies work together.

[CrashPlan]: https://www.code42.com/crashplan/
[tethered]: https://en.wikipedia.org/wiki/Tethering

What can we do?
---------------

Well, the "easiest" solution to this problem is to simply click on your Crash Plan menu icon and tell it to pause indefinitely or for a certain amount of time, re-enabling when you are done tethering. However, because this requires extra work and extra thought, and considering the conditions under which CrashPlan should not backup are already known, I was not happy with this solution.

Being a programmer, I looked for a way to automate. My solution was to create a program which monitors the system's network configuration. The program disables the CrashPlan backup service when it learns you are on the Wi-Fi network provided by your tether, and enables it when you disconnect from that network.

Installation
------------

The network monitor is OS X-only. It has been tested on OS X 10.10 Yosemite with CrashPlan installed for all users (the default). To install, follow these steps.

First, clone the Git repository:

```bash
git clone https://github.com/seanfisk/crashplan-network-monitor.git
cd crashplan-network-monitor
```

Next, run the installer, passing in the network name under which you would like to disable CrashPlan:

```bash
sudo ./install 'John Doe'
```

Answer yes to the prompts, and you should be good to go!

As I use an iPhone, the network name consists of my first and last name. This may be different for you. The program works by polling the network name, with a default interval of one minute. The program will show notifications to the user; by default, this is the user of the controlling terminal which ran the installer. The default network interface is `en1`, which is AirPort on most Macs. These can all be configured — to see how, run:

```bash
sudo ./install --help
```

The program has been written in such a way as to handle an installation of CrashPlan for the current user only, but this has not been tested. To activate this behavior, do not run the program as `root`:

```bash
./install 'John Doe'
```

As mentioned, *this is untested*. If you have problems, please report them via the [issue tracker][].

[issue tracker]: https://github.com/seanfisk/crashplan-network-monitor/issues

Known Issues
------------

This program was written as a working proof of concept. There are still many issues to be solved:

- The program currently works by polling the Wi-Fi network. It would be nice to be notified when the state of the network changes, and only run the check then.
- The program can't differentiate between a cellular/tethered Wi-Fi network *My Network* and a regular, unlimited-data Wi-Fi network *My Network*. The only way I can think to do this at this time is to use a `traceroute`, attempting to detect known network nodes of the carrier. But using `traceroute` would be unreliable, carrier-specific, slow, and use data when tethered (which is what we *don't* want to do!).
- The program doesn't know when you have paused CrashPlan or stopped the CrashPlan service on your own. It won't start the service if you have stopped it and the connected Wi-Fi network doesn't change, but it will start it when exiting the tethered Wi-Fi network.
- The program doesn't support other forms of tethering, such as through USB or Bluetooth (which the iPhone supports). I haven't yet researched this, but I would like to in the future.

There are also some technical implementation issues:

- The program is currently written in Python using subprocesses to perform various functions (`launchctl`, `osascript`, `networksetup`, `su`). I would like to rewrite it in [Swift][] with native calls replacing subprocesses whenever possible:
    - [CoreWLAN][] to replace `networksetup`
    - [NSUserNotification][] or [terminal-notifier][] to replace `osascript`
    - Replace `su` with a native permission call if necessary and possible
    - See [this thread](http://mac-os-forge.2317878.n4.nabble.com/Programmatic-interface-to-launchctl-and-some-other-questions-OS-X-10-5-td189494.html) for possible programmatic access to `launchctl`
- The program creates a log file for debugging purposes (yay!), but it is not rotated, which means it will continue to grow indefinitely. We can [use newsyslog](http://serverfault.com/questions/352942/equivalent-of-logrotate-on-osx) or logrotate to rotate this file.
- The installation process should be easier, probably with a `.pkg` installer.

[Swift]: http://www.apple.com/swift/
[CoreWLAN]: https://developer.apple.com/library/mac/documentation/CoreWLAN/Reference/CWInterface_reference/index.html#//apple_ref/occ/instp/CWInterface/ssid
[NSUserNotification]: https://developer.apple.com/library/mac/documentation/Foundation/Reference/NSUserNotification_Class/index.html
[terminal-notifier]: https://github.com/julienXX/terminal-notifier
