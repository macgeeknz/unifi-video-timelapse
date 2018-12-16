# unifi-video-timelapse

This repository is the reponse to a request for me to share my code after this posting on the Ubiquity Unifi-Video forums:

https://community.ubnt.com/t5/UniFi-Video-Stories/My-5-month-NVR-solution-for-two-G3-flex-cameas/cnc-p/2598278

The tar file contains an empty folder structure and symcbolic link, plus 4 shell scripts called snapshotter-via-api.sh, continuous-snapshots.sh, generateTimelapse.sh, and storage-purger.sh.

The full list of pre-requisite steps you need to take to get this working all are as follows:

- sign up to Linode, create a new 'nanode' VM using their Ubuntu 16.04 LTS template, then log in via SSH
 (note - any hosting platform will work; I use Linode becuase they have free inbound traffic - perfect for an NVR)
 
- run software updates:

	apt update && apt -y upgrade

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

	apt -y install unifi-video

- install ffmpeg

	apt -y install ffmpeg

- install jq

	apt -y install jq

- install imagemagick
	
	apt -y install imagemagick

- install apache2

	apt -y install apache2

- clone the tar file from this repository to the root-level of the VM and un-tar:

	cd /root ; git clone https://www.github.com/macgeeknz/unifi-video-timelapse
	cd / ; tar -xvf /root/unifi-video-timelapse/unifi-bundle.tar

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
	[save]

- Append /usr/local/bin/continuous-snapshots.sh to the end of rc.local so that it launches at startup:

	echo "#!/bin/sh -e" > /etc/rc.local
	echo "sleep 30 ; /usr/local/bin/continuous-snapshots.sh &" >> /etc/rc.local
	echo "exit 0" >> /etc/rc.local

- reboot the VM
	
	reboot

- wait about 3-4 minutes for the VM to restart

- connect to the VM through apache and check that new still frames are being added here every 10 seconds:

	http://nvr.vm.ip.address/timelapse/scratch/jpegs/

- after an hour, connect to the VM through apache and check that the an MP4 was created:

	- connect to the VM through apache and check that new still frames are being added here:

	http://nvr.vm.ip.address/timelapse/movies/

# Note that the apache access is not encrypted, nor authenticated.
Anyone who knows where to look could view the contents of your timelapse folder! There are plenty of other tutorials on the 'net about how to achieve security in apache so the steps for that are not covered here.
