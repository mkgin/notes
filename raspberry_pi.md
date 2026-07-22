# raspberry pi

* https://www.raspberrypi.com/documentation/computers/configuration.html

## headless setup ...

* https://www.raspberrypi.com/documentation/computers/remote-access.html#enabling-the-server

* enable ssh ... 
    * `touch /boot/ssh` ... empty file named ssh in boot partition....
    * ssh keys, users.
	* #PermitRootLogin prohibit-password
	*
	
* userconf.txt file, which contains a string username:encryptedpassword.


* /etc/apt/apt.conf.d/20proxy
    * `Acquire::http::Proxy "http://aptcache.host.or.ip:3142";`

* basic packages.. (emacs-nox) 
    * 

## hardware

### LCD (i2c ...)
* https://github.com/dbrgn/RPLCD

### RTC (i2c)
* https://github.com/rgl/rtc-i2c-ds3231-rpi

# rasbian bookworm 32 bit

* Error on `apt-update`: `http://raspbian.raspberrypi.com/raspbian/dists/bookworm/InRelease: Key is stored in legacy trusted.gpg keyring (/etc/apt/trusted.gpg), see the DEPRECATION section in apt-key(8) for details.`

* https://github.com/RPi-Distro/repo/issues/348#issuecomment-1856137060
