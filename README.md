lfs-me
======

Linux from Scratch made (more) easy. A simple fakeroot based package manager for LFS, heavily inspired by Archlinux.

Description
-----------
*lfs-me* is a package manager that is completely written in bash. It was created in order to make installing Linux From Scratch easier and provide the author a learning experience about how package management can be done from scratch.

*lfs-me* doesn't support inter package dependencies, but it can keep track of what packages and files have been installed on the system, for more information about the package index see the **Index** section.

Packages are created in the form of a PKGBUILD file ( very similar to those used by Archlinux ). PKGBUILD files are bash scripts that get loaded by the package manager and contain variables and functions that describe how the software is to be downloaded, built and installed. You can create a package by copying `PKGBUILD.proto`. The exact process is described in the **PKGBUILD** section.

**WARNING**
You should be aware that you should not under any circumstances use *lfs-me* in a production environment ( outside a virtual machine or chroot ) if you don't now what you are doing and how it works. This might break your system horribly if you try to install packages in there.

It isn't recommended to uninstall packages with the builtin *remove* option without checking beforehand if some needed files might get deleted. Otherwise you could brick your system by removing files that have also been installed by other packages and are needed by them.

Package Format
--------------
The packages are simple tarballs with xz compression that contain some extra files with meta information:

* `md5sums`: A list of all the files and their MD5 checksums in order to check the integrity of installed files.
* `DIRS`: A list of every installed directory.
* `FILES`: A list of every installed File.
* `LINKS`: A list of every installed symbolic link.
* `PKGBUILD`: The PKGBUILD file from which the pacakge has been built. This is needed to run postinstall and postremove functions.

Keep in mind that the install option simply overwrites everything without checking if it already exists. And the remove option simply deletes everything listed in those files. The only check that is done is that symbolic links are deleted only when they don't link to any file/directory anymore.

Index
-----
To keep track of what packages and what files are installed on the system, the metadata files (`md5sums`, `PKGBUILD` etc.) get installed into the package index. The package index is a directory structure of the following form:

    $install_prefix/$index_dir
                          |-->$pkgname
                                  |-->$pkgver
                                         |-->md5sums
                                         |-->PKGBUILD
                                         |-->DIRS
                                         |-->FILES
                                         |-->LINKS
Where `$...` are the respective values of the variables. If `$pkgver_postfix` is set, the last directory is `$pkgver-$pkgver_postfix`

PKGBUILD
--------
Create a new package by copying and modifying `PKGBUILD.proto`. The *PKGBUILD* is loaded by *lfs-me* and then the different functions are executed.

#### Variables:
* `pkgname`: The name of the package.
* `pkgver`: The version string for the package.
* `pkgver_postfix`: A postfix string that is added after the version string. This **mustn't** start with `-`!
* `sources`: An array containing the download URLs of the source files.
* `sha1sums`: An array containing the SHA1 checksums of the sources files in order to check the integrity.

#### Predefined variables:
Those variables are provided by *lfs-me* and mustn't be changed in the *PKGBUILD*.
* `fakeroot_dir`: Directory in which the package is to be installed ( instead of the root directory ). The package tarball is created from this directory.
* `build_dir`: Directory to extract the source files to and build the package in.
* `sources_dir`: Directory in which the source tarballs are stored.

#### Functions:
If you don't include a function it falls back to its default behavior.
The functions are executed in the following order:

1. `lfs_me_prepare()`: Extract the source files and run configure scripts etc.
2. `lfs_me_build()`: Do the actual compilation ( *make* ).
3. `lfs_me_check()`: Run unit tests.
4. `lfs_me_install()`: Install the built files into the *fakeroot_dir*
5. `lfs_me_postinstall()`: This function is executed after a package has been installed ( into the actual filesystem ). It isn't executed when the package is created. You can use this to create users and groups for example.
6. `lfs_me_postremove()`: This function is executed after a package has been removed from the system.

Command line usage
--------------------
`lfs-me mode file [options]`

#### Modes
|    mode   |   parameter   |                                    description                                              |
|:---------------|:-------------------------------------|---------------------------------------------------------------------------------------------|
|`build`         |*PKGBUILD-file*                       |Build the package specified by the PKGBUILD                                                  |
|`install`       |*package.pkg*                         |Install a package to the system                                                              |
|`remove`        |*package.pkg*                         |Remove a package from the system and index.                                                  |
|`remove`        |*pkgname* *pkgver* *[pkgver_postfix]* |Remove a package from the system and index.                                                  |
|`indexadd`      |*package.pkg*                         |Add a package to the package index without installing it.                                    |
|`indexremove`   |*package.pkg*                         |Remove a package from the index without removing from the system.                            |
|`indexremove`   |*pkgname* *pkgver* *[pkgver_postfix]* |Remove a package from the index without removing it from the system                          |
|`indexlist`     |                                      |List all packages in the package index.                                                      |
|`check`         |*package.pkg*                         |Check the installed files                                                                    |
|`check`         |*pkgname* *pkgver* *[pkgver_postfix]* |Check the installed files                                                                    |
|`checksums`     |*PKGBUILD-file*                       |Create checksums for downloaded source files specified in the *sources* array in the PKGBUILD|
|`download`      |*PKGBUILD-file*                       |Download the source files specified in the *sources* array in the PKGBUILD                   |

#### Options
|  short  |        long        |                    description                      |
|:--------|:-------------------|-----------------------------------------------------|
|`-b`     |`--build-dir`       |Specify build directory                              |
|`-D`     |`--debug`           |Enable debug mode                                    |
|`-f`     |`--fakeroot-dir`    |Specify fakeroot directory (see predefined variables)|
|`-h`     |`--help`            |Show help output                                     |
|`-i`     |`--index-dir`       |Specify index directory                              |
|         |`--no-checks`       |Don't run tests                                      |
|         |`--no-color`        |Disable color                                        |
|         |`--no-downloads`    |Don't download sources                               |
|         |`--ignore-checksums`|Don't check checksums                                |
|`-p`     |`--prefix`          |Specify installation prefix                          |
|`-s`     |`--sources`         |Specify directory where sources are stored           |

Configuration file
------------------
You can create the configuration file `~/.lfs-me` to set default values for variables used by *lfs-me*. You can use the following variables:
* `build_dir`
* `fakeroot_dir`
* `install_prefix`
* `sources_dir`
* `index_dir`
* `run_checks`: *true* or *false*
* `download_sources`: *true* or *false*
* `verify_checksums`: *true* or *false*
* `show_color`:	*true* or *false*
* `debug`: *true* or *false*

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
