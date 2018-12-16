# unifi-video-timelapse
This archive contains the files overlay for Ubiquiti Unifi-Video NVR on Ubuntu VM for timelapse.

These are the pre-requisite steps I used to create a Unifi-Video NVR running on a USD$5/month Linode VM:

- sign up to Linode, create a new 'nanode' VM using their Ubuntu 16.04 LTS template, then log in via SSH

- run software updates:

	apt update && apt upgrade

- set local timezone:

	dpkg-reconfigure tzdata
	[set timezone]

- start NTP:

	service ntp start

- add the unifi repository:

	echo "deb [arch=amd64] http://www.ubnt.com/downloads/unifi-video/apt-3.x xenial ubiquiti" > /etc/apt/sources.list.d/unifi-video.list
	apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 97B46B8582C6571E
	apt update

- install unifi-video package and dependancies:

	apt install unifi-video

- install ffmpeg

	apt install ffmpeg

- install jq

	apt install jq

- install imagemagick
	
	apt install imagemagick

- install apache2

	apt install apache2

- Create the blank directory structure:

	mkdir -p /timelapse/movies /timelapse/scratch/jpegs /timelapse/scratch/moviecreation

- Create a symbolic link for apache:

	ln -s /timelapse /var/www/html/

- clone these from github(?):

	/usr/local/bin/snapshotter-via-api.sh
	/usr/local/bin/generateTimelapse.sh
	/usr/local/bin/continuous-snapshots.sh
	/usr/local/bin/storage-purger.sh

- log in to the NVR webUI & complete initial configuration:

	https://nvr.vm.ip.address:7443/
	Log in, accept ToS, create admin account
	Connect camera via normal methods & make sure live view works
	(ie configure camera with NVR server IP & adoption token)
	Cameras -> [Camera Name] -> Recording -> Record Mode = "Record only motion" 
	Settings -> System Configuration -> set 'Space To Keep Free' slider to 10GB (smallest) -> Save

- Back on the the VM console again, edit /usr/local/bin/snapshotter-via-api.sh to contain your NVR admin username & password

	username="YOUR-USERNAME"
	password="YOUR-PASSWORD"

- Create crontab entries to trigger creating the movies every hour and cleaning up old files

	crontab -e
	[add the following two lines]
	58 * * * * /usr/local/bin/storage-purger.sh
	0 * * * * /usr/local/bin/generateTimelapse.sh
	save

- Append /usr/local/bin/continuous-snapshots.sh to the end of rc.local so that it launches at startup:

	echo "#!/bin/sh -e" > /etc/rc.local
	echo "sleep 30 ; /usr/local/bin/continuous-snapshots.sh &" >> /etc/rc.local
	echo "exit 0" >> /etc/rc.local

- reboot the VM
	
	reboot

- wait about 3-4 minutes for the VM to restart

- connect to the VM trough apache and check that new still frames are being added here:

	http://nvr.vm.ip.address/timelapse/scratch/jpegs/

- after an hour, connect to the VM through apache and check that the an MP4 was created:

	- connect to the VM trough apache and check that new still frames are being added here:

	http://nvr.vm.ip.address/timelapse/movies/

Note that the apache access is not encrypted, nor authenticated. Therefore anyone who knows where to look could view the contents of your timelapse folder. There are plenty of other tutorials on the 'net about how to achieve security in apache so the steps for that are not covered here.
