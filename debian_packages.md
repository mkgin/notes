### Building

* make in parallel ( -j4 )
 * ```export DEB_BUILD_OPTIONS="parallel=4"```
* build a binary package 
 * ```dpkg-buildpackage -b -rfakeroot -us -uc```
* build package without cleaning... (-nc)
 * ```dpkg-buildpackage -rfakeroot -D -nc -us -uc```
