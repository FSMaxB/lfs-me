Changelog
=========

v3.1.3
------
* parallel xz compression
* Several fixes and improvements, thanks @faithust
    * fix cleaning of fakeroot dir after installing
    * use parallel make
    * fix missing install prefix
    * fix deleting fakroot_dir and build_dir
    * fix help message line

v3.1.2
------
* --version flag
* significant performance improvement
* LC_ALL=C for the build process
* fix: bug in dependency handling when multiple versions of a package are installed

v3.1.1
------
* new flag --no-logs ( to fix an issue with the linux package)
* performance improvements ( less forks )

v3.1.0
------
* PKGBUILD format version 5
* "overwrites" with files that a package overwrites
* bashcompletion file

v3.0.2
------
* fix creation of checksums when downloading to other filenames than in the URL

v3.0.1
------
* fix regex
* fix escaping strings

v3.0.0
------
* PKGBUILD format version 4
* Download to other filenames than in the URL.
* Check for file conflicts before installing.
* Backup config files before installing/removing
* New commands `backuplist` and `backupmerge`
* New example files
* Packages should now be in conflict with themselves
* several fixes
* some cleanup

v2.1.1
------
* Ignore SSL certificates with `--no-cert-check`
* clenup
* fix dependency check

v2.1.0
------
* fix problem with logging
* Therefore new PKGBUILD format version 3
* now logging output of lfs_me_prepare
* `--no-time` flag
* fix a problem with special characters in package names

v2.0.1
------
* Show execution time with `-t` or `--show-time`
* fix help output

v2.0.0
------
* **Support for dependencies**
* SHA1 sums for index
* Format version checks for index and PKGBUILD format
* removed the `pkgver_postfix` variable
* log directory
* `lfs_me_preinstall` and `lfs_me_preremove`
* New commands: `list`, `owner`, `rebuild` and `checkdeps`
* General code cleanup
* fixes

v1.0.1
------
* search function for `indexlist`
* changed license to GPLv3 **without** any later versions
* fixes

**This is the last version before breaking the index, package and PKGBUILD format, so use this for older systems and PKGBUILDS!**

v1.0.0
------
* **Introduced package index**
* **Remove option checks if a file is still needed before deleting it**
* Index support for `remove` and `check`
* File removal is much cleaner
* Various fixes

v0.1.2
------
* Fix potential issue with relative pathnames
* Critical fix: Remove now respects install_prefix

v0.1.1
------
* Config file `~/lfs-me`
* Fix checking files when a prefix is enabled
* Introduce `Changelog.md`

v0.1
----
Inital version with the following features:

* Building packages
* Installing packages
* Checking checksums of installed packages
* Download source files
* Creating checksums for downloaded source files
* Specifying directory to download sources to
* Specifying installation prefix
