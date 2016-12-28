lfs-me
======

Linux from Scratch made (more) easy. A simple fakeroot based package manager for LFS, heavily inspired by Archlinux.

Description
-----------
*lfs-me* is a package manager that is completely written in bash. It was created in order to make installing Linux From Scratch easier and provide the author a learning experience about how package management can be done from scratch.

*lfs-me* can keep track of what packages and files have been installed on the system, for more information about the package index see the **Index** section. Although *lfs-me* supports inter package dependencies, it cannot automatically resolve them, only show you which dependencies are missing.

Packages are created in the form of a PKGBUILD file ( very similar to those used by Archlinux ). PKGBUILD files are bash scripts that get loaded by the package manager and contain variables and functions that describe how the software is to be downloaded, built and installed. You can create a package by copying `PKGBUILD.proto`. The exact process is described in the **PKGBUILD** section.

**WARNING**
You should be aware that you should not under any circumstances use *lfs-me* in a production environment ( outside a virtual machine or chroot ) if you don't now what you are doing and how it works. This might break your system horribly if you try to install packages in there.

Package Format
--------------
The packages are simple tarballs with xz compression that contain some extra files with meta information:

* `sha1sums`: A list of all the files and their SHA1 checksums in order to check the integrity of installed files.
* `DIRS`: A list of every installed directory.
* `FILES`: A list of every installed File.
* `LINKS`: A list of every installed symbolic link.
* `PKGBUILD`: The PKGBUILD file from which the pacakge has been built. This is needed to run postinstall and postremove functions.

Index
-----
To keep track of what packages and what files are installed on the system, the metadata files (`sha1sums`, `PKGBUILD` etc.) get installed into the package index. The package index is a directory structure of the following form:

    $install_prefix/$index_dir
    ├── $pkgname
    │   └── $pkgver
    │       ├── sha1sums
    │       ├── PKGBUILD
    │       ├── DIRS
    │       ├── FILES
    │       └── LINKS
    └── version

Where `$...` are the respective values of the variables. The file `version` contains the version of the index format to prevent corruption when using an older/newer version of lfs-me.

PKGBUILD
--------
Create a new package by copying and modifying `PKGBUILD.proto`. The *PKGBUILD* is loaded by *lfs-me* and then the different functions are executed.

#### Variables:
* `pkgbuild_version`: Version of the PKGBUILD format.
* `pkgname`: The name of the package.
* `pkgver`: The version string for the package.
* `dependencies`: An array containing a list of all dependencies of the package. See more on the *Dependencies* section.
* `backup`: An array containing the names of files that are to be backed up by the lfs-me before installing/upgrading the package. This is useful for config files.
* `overwrite`: An array containing the names of files that get overwritten by this package. This makes the conflict check omit those files.
* `sources`: An array containing the download URLs of the source files.
* `sha1sums`: An array containing the SHA1 checksums of the sources files in order to check the integrity.

#### Predefined variables:
Those variables are provided by *lfs-me* and mustn't be changed in the *PKGBUILD*.
* `fakeroot_dir`: Directory in which the package is to be installed ( instead of the root directory ). The package tarball is created from this directory.
* `build_dir`: Directory to extract the source files to and build the package in.
* `sources_dir`: Directory in which the source tarballs are stored.
* `log_dir`: Directory where logs get stored.

#### Functions:
If you don't include a function it falls back to its default behavior.
The functions are executed in the following order:

1. `lfs_me_prepare()`: Extract the source files and run configure scripts etc.
2. `lfs_me_build()`: Do the actual compilation ( *make* ).
3. `lfs_me_check()`: Run unit tests.
4. `lfs_me_install()`: Install the built files into the *fakeroot_dir*
5. `lfs_me_preinstall()`: This function is executed before a package gets installed to the system.
6. `lfs_me_postinstall()`: This function is executed after a package has been installed ( into the actual filesystem ). It isn't executed when the package is created. You can use this to create users and groups or update info pages for example.
7. `lfs_me_preremove()`: This function is executed before a package gets removed from the system.
8. `lfs_me_postremove()`: This function is executed after a package has been removed from the system.

#### Dependencies:
`dependencies` is an array that contains all the dependencies. A dependency can be a single package name or a package name, followed by a comparator followed by a version number. Valid comparators are `>`, `<`, `>=`, `<=` and `=`. Those dependencies can be inversed to be conflicts by preceding them with `!`. Most packages should be in conflict with themselves for example, so that you can't install multiple conflicting versions in parallel.

Example:
    depencencies=(
        'linux>=3.2'
        'bash'
		'!tmux'	#in conflict with tmux
    )

Command line usage
--------------------
`lfs-me mode file [options]`

#### Modes
|      mode      |              parameter               |                                       description                                           |
|:---------------|:-------------------------------------|---------------------------------------------------------------------------------------------|
|`backuplist`    |                                      |List all backup files.                                                                       |
|`backupmerge`   |                                      |Find all bakup files and ask how to merge them.                                              |
|`build`         |*PKGBUILD-file*                       |Build the package specified by the PKGBUILD                                                  |
|`checkdeps`     |*PKGBUILD-file*                       |Check if all dependencies are met.                                                           |
|`checkdeps`     |*package.pkg*                         |Check if all dependencies are met.                                                           |
|`checkdeps`     |*package*                             |Check if all dependencies are met.                                                           |
|`checkdeps`     |*package* *pkgver*                    |Check if all dependencies are met.                                                           |
|`rebuild`       |*package.pkg*                         |Rebuild a package from an existing one.                                                      |
|`rebuild`       |*pkgname*                             |Rebuild a package from an existing one.                                                      |
|`rebuild`       |*pkgname* *pkgver*                    |Rebuild a package from an existing one.                                                      |
|`install`       |*package.pkg*                         |Install a package to the system                                                              |
|`remove`        |*package.pkg*                         |Remove a package from the system and index.                                                  |
|`remove`        |*pkgname* *pkgver*                    |Remove a package from the system and index.                                                  |
|`remove`        |*pkgname*                             |Remove a package from the system and index.                                                  |
|`indexadd`      |*package.pkg*                         |Add a package to the package index without installing it.                                    |
|`indexremove`   |*package.pkg*                         |Remove a package from the index without removing it from the system.                         |
|`indexremove`   |*pkgname* *pkgver*                    |Remove a package from the index without removing it from the system.                         |
|`indexremove`   |*pkgname*                             |Remove a package from the index without removing it from the system.                         |
|`indexlist`     |                                      |List all packages in the package index.                                                      |
|`indexlist`     |*searchterm*                          |Search for packages in the index.                                                            |
|`list`          |*package.pkg*                         |List all files of a package.                                                                 |
|`list`          |*pkgname* *pkgver*                    |List all files of a package.                                                                 |
|`list`          |*pkgname*                             |List all files of a package.                                                                 |
|`check`         |*package.pkg*                         |Check the installed files                                                                    |
|`check`         |*pkgname* *pkgver*                    |Check the installed files                                                                    |
|`check`         |*pkgname*                             |Check the installed files                                                                    |
|`checksums`     |*PKGBUILD-file*                       |Create checksums for downloaded source files specified in the *sources* array in the PKGBUILD|
|`download`      |*PKGBUILD-file*                       |Download the source files specified in the *sources* array in the PKGBUILD                   |
|`owner`         |*file*                                |List all packages that own a file.                                                           |

#### Options
|  short  |        long        |                    description                      |
|:--------|:-------------------|-----------------------------------------------------|
|`-b`     |`--build-dir`       |Specify build directory                              |
|`-D`     |`--debug`           |Enable debug mode                                    |
|`-f`     |`--fakeroot-dir`    |Specify fakeroot directory (see predefined variables)|
|`-h`     |`--help`            |Show help output                                     |
|`-i`     |`--index-dir`       |Specify index directory                              |
|`-l`     |`--log-dir`         |Specify log directory                                |
|         |`--no-checks`       |Don't run tests                                      |
|         |`--no-color`        |Disable color                                        |
|         |`--no-downloads`    |Don't download sources                               |
|         |`--ignore-checksums`|Don't check checksums                                |
|`-p`     |`--prefix`          |Specify installation prefix                          |
|`-s`     |`--sources`         |Specify directory where sources are stored           |
|`-t`     |`--show-time`       |Show execution time at the end.                      |
|         |`--no-time`         |Don't show execution time at the end.                |
|         |`--no-cert-check`   |Don't check SSL certificates.                        |
|         |`--no-logs`         |Don't log output. (needed for menus during build)    |
|         |`--version`         |Show version number.                                 |

Configuration file
------------------
You can create the configuration file `~/.lfs-me` to set default values for variables used by *lfs-me*. You can use the following variables:
* `build_dir`
* `fakeroot_dir`
* `install_prefix`
* `sources_dir`
* `index_dir`
* `log_dir`
* `do_logs`: *true* or *false*
* `run_checks`: *true* or *false*
* `download_sources`: *true* or *false*
* `verify_checksums`: *true* or *false*
* `show_color`:	*true* or *false*
* `show_time`: *true* or *false*
* `debug`: *true* or *false*
* `merge_tool`: Which mergetool to use for merging backups. Default is `vimdiff`

Example:

    sources_dir=~/src
    install_prefix=~/local/
    index_dir=/var/lfs-me/index
    show_color=false

In this case, the index is stored in `~/local/var/lfs-me/index`

Typical scenario
----------------
Here is a typical scenario on how to create, build and install a package with *lfs-me*.

1. Create the *PKGBUILD* from `PKGBUILD.proto` and save it as `foo-0.1.1`
2. Download the source files into `~/downloas`: `lfs-me download foo-0.1.1 -s ~/downloads`
3. Calculate the checksums for the downloaded sources: `lfs-me checksums foo-0.1.1 -s ~/downloads`
4. Edit the *PKGBUILD* to include the checksums.
5. Build the package: `lfs-me build foo-0.1.1 -s ~/downloads`
6. Install the package into `/mnt/lfs` for example ( you can omit the prefix to install to `/`): `sudo lfs-me install foo-0.1.1.pkg -p /mnt/lfs`
7. Check the installed files: `lfs-me check foo 0.1.1 -p /mnt/lfs`

lfs-me in action
----------------
To see *lfs-me* in action take a look at my collection of packages at https://github.com/FSMaxB/lfs-me-repos .

