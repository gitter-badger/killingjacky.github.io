---
title: Work around for macOS Mail.app eating high CPU
tags:
  - tech
  - mac
  - Mail.app
date: 2017-03-09 14:13:31
---
The work around is based on a software named `monit`. Let's see what monit do.

[Monit](https://mmonit.com/monit/) is a small Open Source utility for managing and monitoring Unix systems. Monit conducts automatic maintenance and repair and can execute meaningful causal actions in error situations.

![](https://mmonit.com/monit/img/logo.png)

<!-- more -->

#### Step-by-step

##### I. Install monit

- Download monit here for 64bits macOS https://mmonit.com/monit/dist/binary/5.21.0/monit-5.21.0-macos-x64.tar.gz
- Extract the archive, you will get file like this:

    ![](https://cloud.githubusercontent.com/assets/5130185/23732137/445b8186-04ac-11e7-8b77-947379bf84f5.png)

- Copy files into their locations
  - `sudo cp -f bin/monit /usr/local/bin`
  - `sudo chmod a+x /usr/local/bin/monit`
  - `sudo cp -f conf/monitrc /usr/loca/etc`
  - `sudo cp -rf man/man1/monit.1 /usr/local/share/man/man1`
- Test your installation
  - `monit --version`

##### II. Configure monit

- Add the following lines into `/usr/loca/etc/monitrc`

```shell
check process Mail matching "/Applications/Mail.app/Contents/MacOS/Mail"
     start program = "/usr/bin/open -a /Applications/Mail.app"
     stop program  = "/usr/bin/pkill -9 Mail"

     if cpu > 80% for 2 cycles then alert
     if cpu > 100% for 2 cycles then restart
```

##### III. Configure the startup script for LaunchDaemon

LaunchDaemon is the startup manager for macOS, and the startup items for LaunchDaemon can be started when system boot up before user login. It's system level.

- `sudo vim /Library/LaunchDaemons/com.tildeslash.monit.plist`
- Parse the following lines into that plist file
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
     <key>Label</key>             <string>com.tildeslash.monit</string>
     <key>ProcessType</key>       <string>Adaptive</string>
     <key>Disabled</key>          <false/>
     <key>RunAtLoad</key>         <true/>
     <key>LaunchOnlyOnce</key>    <false/>
     <key>ProgramArguments</key>
     <array>
             <string>/usr/local/bin/monit</string>
             <string>-I</string>
     </array>
</dict>
</plist>
```
- Now load monit
  - `sudo launchctl load -w /Library/LaunchDaemons/com.tildeslash.monit.plist`

- Final test
  - The monit daemon should be running now, test its status using command:
  - `sudo monit status`, if everything was done right, you should see the diagnosis information of the Mail process.
  - Besides, monit serves a http server at address http://localhost:2812

##### IV. Sum up

The basic idea of this workaround is to monitor the CPU usage of process Mail, if it's eating over 100% CPU for more than 1 minute, monit will restart it. Monit will check the status of Mail every 30 seconds (the default configuration), and it prints logs into the system log, you can access the log with system applicaton `Console`/`控制台`, e.g. To investigate if your Mail.app lose its mind for the last few days, open Console application and search `monit`. If any entry out there, bla bla 1000 words for the excellent Mail.app!!! Also, you can check the uptime of Mail.app in the web page of monit with default user/password - admin/monit.

![](https://cloud.githubusercontent.com/assets/5130185/23732798/3f38ece4-04b0-11e7-96d9-758be9d866cc.png)