Changelog
=========

v1.0.0
------
* **Introduced package index**
* **Remove option checks if a file is still needed before deleting it**
* Index support for "remove" and "check"
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