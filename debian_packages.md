### Building

* make in parallel ( -j4 )
 * ```export DEB_BUILD_OPTIONS="parallel=4"```
* build a binary package 
 * ```dpkg-buildpackage -b -rfakeroot -us -uc```
* build package without cleaning... (-nc)
 * ```dpkg-buildpackage -rfakeroot -D -nc -us -uc```

* update release info for APT repositories
```
Reading package lists... Done
E: Repository 'https://dl.ubnt.com/unifi/debian stable InRelease' changed its 'Codename' value from 'unifi-7.1' to 'unifi-7.2'
N: This must be accepted explicitly before updates for this repository can be applied. See apt-secure(8) manpage for details.
```
apt-get update --allow-releaseinfo-change
