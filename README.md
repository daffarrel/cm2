CM2 Realtime CMS Capture Application
====================================



What is CM2?
------------
####CM2 captures real-time CMS data and sends it to the endpoints we want  

Built as a bridge between the Avaya CMS and our various systems for routing, dashboarding, reporting, and peace of mind, CM2 runs arbitrary CMS reports and feeds them to configurable endpoints on the schedules the data demands. It is built in five pieces, the CMS report daemon, the CMS report runner, the CMS report spooler, the CM2 handler daemon, and the CM2 export loader.  

Each piece is written in the perl programming language, targeted with versions compatible with the native installations on each CMS, allowing for deployment without dependency troubles.

 
Architecture
------------
Each piece is engineered to prevent resource locking on the CMS server when reports run haywire. The daemon supervises and maintains timeout alarms on each report and the spooler. The reports output to a file to be watched by the spooler to prevent socket misbehavior and Unix file handle issues. The spooler is then alerted to move the files when it receives the signal SIGUSR1, and will in turn alert the handler daemon that things are moving happily along.  

The receiver daemon places an inotify on the directory receiving the files, and sends them to the processors when the write is closed. The processors can be tailored for each endpoint, such as a database or report emailer.


CMS Daemon Management
---------------------
The CMS server-side CM2 software is managed by a daemon, and should be installed by default under the `/export/home/rtdata/cm2` directory. This application will be owned by the rtdata user under a standard installation.

The application executables are inside the `./cm2/bin` directory. There are several commands used to manage the processes:
- `./cm2/bin/cm2 status`
- `./cm2/bin/cm2 start`
- `./cm2/bin/cm2 stop`
- `./cm2/new`

Status will report back if the service is currently running and force a check on the configured reports. Start and Stop will, as expected, start and stop the service. Note however, stop will not currently disable the cron entry to check for the daemon’s status, and will restart on the next check.

Disabling the Daemon
--------------------
There are a few commands and steps to take to completely turn off and disable the CM2 CMS daemon.
1. Log in as the `rtdata` user
	- The best way would be to execute: `sudo su – rtdata`
	- If direct login access is needed, an ssh key can be acquired from development
2. Modify the crontab file  
	- Run the command `crontab –e`  
	- Find the following line:  
			`* * * * * /export/home/rtdata/cm2/bin/cm2 start > /export/home/rtdata/cm2/log/start.log`  
	- Comment out the line to look like follows:  
			`#* * * * * /export/home/rtdata/cm2/bin/cm2 start > /export/home/rtdata/cm2/log/start.log`  
	- Save the crontab (by default hit `esc`, followed by `:wq`)  
3. Stop the daemon  
	- In the rtdata user home run the command: `./cm2/bin/cm2 stop`  
	- The service is now stopped  
4. (optional) Verify the data has stopped  
	- Checking the timestamps on the files in `~/cm2/outfiles` should show they’ve not been modified since the service stopped  
	```
	$ ls -lathr ~/cm2/outfiles
	total 108K
	drwxr-xr-x. 7 rtdata cms 4.0K Jan 13 14:21 ..
	drwxr-xr-x. 2 rtdata cms 4.0K Apr 26 14:43 .
	-rw-rw-rw-. 1 rtdata cms 100K Apr 26 14:43 NORTHEAST_custom:real-time:TWCRTAGEN_2
	```

5. (optional) If they continue, a second execution of ./cm2/bin/cm2 stop would force all copies to terminate

Starting the Daemon
-------------------

Much like disabling the daemon, but in reverse:
1. Start the daemon by executing: `./cm2/bin/cm2 start`
2. Modify the crontab file to enable the process checking
	- Run the following command: `crontab -e`
	- Ensure the crontab entry below is uncommented by removing the ‘#’ symbol  
		`#* * * * * /export/home/rtdata/cm2/bin/cm2 start > /export/home/rtdata/cm2/log/start.log`  


Checking Daemon Status
----------------------

A simple status option will force cm2 to run a check and return the daemon status.
```
$ ./cm2/bin/cm2 status
Running status check...Process running
```

CMS Daemon Installation and Configuration
-----------------------------------------

The CM2 CMS daemon installation process requires no privilege escalation or system-wide library installation. There is a 4 step process that unarchives the install into a specially-created rtdata user, ready to execute and run.
1. Create a user rtdata and grant all required ACD, Skill, and VDN privileges to this user in CMS
2. Acquire the `cm2.tar.gz` installation archive
3. Place a copy of this into the rtdata user’s home on the CMS server. FTP, scp, or any other Unix file copy method will work
4. Login to this user via ssh, PuTTY, or your favorite remote terminal application
5. Extract the archive in the home directory with `tar`
```
	   $ tar xjvf cm2.tar.gz
```
6. Alter the cm2 script or the cm2 settings file to add the report settings you wish to run on this CMS
