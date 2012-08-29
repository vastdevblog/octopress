---
layout: post
title: "Using SyslogAppender on OS X"
date: 2012-04-18 09:48
comments: true
categories: [osx, logback]
author: Alex Moffat <alex.moffat@gmail.com>
---

I'm using [ch.qos.logback.classic.net.SyslogAppender](http://logback.qos.ch/apidocs/ch/qos/logback/classic/net/SyslogAppender.html)
to send messages to syslog and I need to be able to test the configuration on OS X. There are various sources of
information on how to do this but no comprehensive explanation of what to do for OS X 10.7 so here's my four step
process.

<!-- more -->

### Choose the syslog facility to log to

The facility an information field associated with a syslog message. 
It is used, together with the priority, in the syslog configuration to separate the processing of different messages. 
For instance, to send all informational messages from the mail system to a one log file and all critical messages from cron to another.
Choosing the facility to use is important. You want to pick one that isn't used by any, or many, other systems so that you can separate your log messages from others.
I picked LOCAL6 based on [this answer on serverfault](http://serverfault.com/questions/115923/which-program-defaults-uses-syslog-local0-7-facilities). It's easy to change though once you get the system working.

### Configure syslog to log your messages to a file

The SyslogAppender will send logged messages to the syslog daemon (syslogd) and that will use the configuration in [/etc/syslog.conf](http://linux.die.net/man/5/syslog.conf) to decide what to do with them next. 
You'll need to use sudo to edit the config file because only root has write access to it.
I used sudo emacs /etc/syslog.conf.
For my testing purposes I just want to log to a file so I added a line to direct messages from the local6 at all priority levels to a vast.log file.

```
local6.*						/var/log/vast.log
```

At this point you should test your syslog configuration using the [logger](http://www.unix.com/man-page/OSX/1/LOGGER/) command line program.
First you have to get syslogd to reload its configuration using the two commands below.

```
sudo launchctl unload /System/Library/LaunchDaemons/com.apple.syslogd.plist
sudo launchctl load /System/Library/LaunchDaemons/com.apple.syslogd.plist
```

Now you can use logger command to send a message. Here I'm sending an info priority message to the local6 faciltiy.

```
logger -p local6.info "test message"
```

In the log file you should see something like

```
Apr 18 10:32:45 Alexs-MacBook-Pro alexmoffat[3724]: test message
```

### Configure syslog to accept messages via a network socket

SyslogAppender uses the java.net.Datagram class to send messages to syslog. 
As installed on OS X syslog does not accept messages over network connections so the configuration must be changed.
For this you have to edit the /System/Library/LaunchDaemons/com.apple.syslogd.plist file.
This, unfortunately from the point of view of editability, is only writable by root and is in binary not xml format.
In earlier versions of OS X the format was xml and the required elements were present but commented out, in 10.7 they are not present.
I had to use the [PlistBuddy](http://developer.apple.com/library/mac/#documentation/Darwin/Reference/ManPages/man8/PlistBuddy.8.html) program supplied by Apple to make the required edits.
You can see the sequence of commands I used below. The intention is to add a NetworkListener dictionary entry 
with with SockServiceName and SockType properties.

```
sudo /usr/libexec/PlistBuddy /System/Library/LaunchDaemons/com.apple.syslogd.plist

Command: Print :Sockets
Dict {
    AppleSystemLogger = Dict {
        SockPathName = /var/run/asl_input
    	SockPathMode = 438
    }
    BSDSystemLogger = Dict {
        SockPathName = /var/run/syslog
    	SockType = dgram
	    SockPathMode = 438
    }
}
Command: Add :Sockets:NetworkListener dict
Command: Add :Sockets:NetworkListener:SockServiceName string "syslog"
Command: Add :Sockets:NetworkListener:SockType string "dgram"
Command: Print :Sockets
Dict {
    NetworkListener = Dict {
        SockServiceName = syslog
        SockType = dgram
    }
    AppleSystemLogger = Dict {
        SockPathName = /var/run/asl_input
        SockPathMode = 438
    }
    BSDSystemLogger = Dict {
        SockPathName = /var/run/syslog
        SockType = dgram
        SockPathMode = 438
    }
}
Command: Save
```

After making the changes above you have to reload the syslog configuration again as you
did after modifying syslog.conf.

```
sudo launchctl unload /System/Library/LaunchDaemons/com.apple.syslogd.plist
sudo launchctl load /System/Library/LaunchDaemons/com.apple.syslogd.plist
```

### Configure the SyslogAppender

The logback manual has [information on how to configure the SyslogAppender](http://logback.qos.ch/manual/appenders.html#SyslogAppender).
An appropriate example configuration for my testing is.

```
  <appender name="SYSLOG" class="ch.qos.logback.classic.net.SyslogAppender">
    <syslogHost>localhost</syslogHost>
    <facility>LOCAL6</facility>
    <suffixPattern>%-5level[%thread] %logger{1} - %msg</suffixPattern>
  </appender>
```

### Test

Finally, give this a test this by running you app and you should see messages appearing in the
log file you configured in /etc/syslogd.conf

### Acknowledgements

I pulled together the information for this post from a number of sources on the web.
In addition to those linked to in the post the original inspiration and flow is from
[Fun with SyslogAppender](http://kreskasnotes.blogspot.com/2010/03/fun-with-syslogappender.html), 
information about what needs to be added to the com.apple.syslogd.plist came from
[Syslogd on 10.6: enabling UDP listening on 514](http://compgroups.net/comp.sys.mac.system/syslogd-on-10.6-enabling-udp-listening-on/199907)


