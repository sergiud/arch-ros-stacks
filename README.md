arch-ros-stacks
---------------

[![Build Status](https://travis-ci.org/bchretien/arch-ros-stacks.png?branch=master)](https://travis-ci.org/bchretien/arch-ros-stacks)
[![Join the chat at https://gitter.im/bchretien/arch-ros-stacks](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/bchretien/arch-ros-stacks?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [How it works](#how-it-works)
- [How to contribute](#how-to-contribute)
  - [PKGBUILD creation and update](#pkgbuild-creation-and-update)
    - [Miscellaneous features](#miscellaneous-features)
    - [PKGBUILD creation](#pkgbuild-creation)
    - [PKGBUILD update](#pkgbuild-update)
    - [AUR upload](#aur-upload)
    - [Pull requests (PR) to arch-ros-stacks](#pull-requests-pr-to-arch-ros-stacks)
    - [Setting push URLs](#setting-push-urls)
    - [Conversion to AUR4 packages (legacy)](#conversion-to-aur4-packages-legacy)
  - [Python version](#python-version)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


This is a fork of [zootboy][zootboy]'s fork of [hauptmech][hauptmech]'s fork of
[moesenle][moesenle]'s very nice arch-ros-stacks.

I am currently maintaining Indigo/Jade packages for Arch Linux. I also accept
pull requests for Groovy and Hydro packages (that I no longer maintain).

Please e-mail me with any suggestions, comments or package requests for
Indigo/Jade:

\<chretien+aur at lirmm dot fr\>

## How it works

Since the AUR now stores PKGBUILDs as Git repositories, each ROS package is
handled as a Git submodule (except some older packages). Such packages contain
two files:

* a `PKGBUILD` script that details the dependencies, versions, how to compile
  the package etc. This file is automatically generated by a Python script.
* a `.SRCINFO` file required for the AUR upload. This file is generated with
  `mksrcinfo`.

Once these files are generated, they simply need to be commited and pushed
to the AUR, so that every Arch user can install the package with his
favorite AUR helper.

To clone this repository, do:

```shell
$ git clone https://github.com/bchretien/arch-ros-stacks.git
$ cd ./arch-ros-stacks
$ ./parallel_submodule_update.py
```

This will initialize/update the (hundreds of) submodules in parallel. This is
equivalent to the usual commands:

```shell
$ git clone --recursive https://github.com/bchretien/arch-ros-stacks.git
```

or

```shell
$ git clone https://github.com/bchretien/arch-ros-stacks.git
$ cd ./arch-ros-stacks
$ git submodule init
$ git submodule update
```

except that it runs on steroids.

## How to contribute

If you want to contribute, simply fork this repository, create or update some
packages, push these packages to the AUR, and make a pull request afterwards.

### PKGBUILD creation and update

The script that automates most of the work is `import_catkin_packages.py`.

Note that the default behavior is to fetch release information from the
[official rosdistro distribution.yaml][distribution.yaml].

#### Miscellaneous features

To list the options:
```shell
$ python2 import_catkin_packages.py --help
```

To list all available Indigo packages:

```shell
$ python2 import_catkin_packages.py --distro=indigo --list | less
```

You can also provide the output directory when listing packages, in order to
see the ones that have not been generated yet:

```shell
$ python2 import_catkin_packages.py --distro=indigo --output-directory=/path/to/arch-ros-stacks/indigo --list | less
```

The Python version can also be chosen:
```shell
$ python2 import_catkin_packages.py --python-version=2.7 --distro=indigo --output-directory=...
```

#### PKGBUILD creation

To add a new official package called `package_foo` recursively:

```shell
$ python2 import_catkin_packages.py --distro=indigo --output-directory=/path/to/arch-ros-stacks/indigo -r package_foo
```

**Note**: packages created that way will be treated as submodules. If the
remote repository is empty (e.g. new AUR package), an initial commit is
automatically created to obtain a valid submodule commit id.

#### PKGBUILD update

To simply update `package_foo`:

```shell
$ python2 import_catkin_packages.py --distro=indigo --output-directory=/path/to/arch-ros-stacks/indigo -u package_foo
```

To recursively "force update" `package_foo`:

```shell
$ python2 import_catkin_packages.py --distro=indigo --output-directory=/path/to/arch-ros-stacks/indigo -rfu package_foo
```

#### AUR upload

PKGBUILDs are no longer explicitly handled in this repository. Instead, new
packages are submodules pointing to the AUR. Thus, updating the AUR package is
as simple as pushing a Git repositories. For this, you will need to create an
account on the [AUR][AUR] (that way, you will also be able to report errors to
package maintainers), and [register an SSH key][AUR key].

Before uploading anything, you need to make sure that the `.SRCINFO` file of
the package is up-to-date, which can be done by running `mksrcinfo`.

Thus, the typical workflow once the PKGBUILD has been updated is:

```shell
# Go to the package directory, e.g. indigo/actionlib
$ cd indigo/actionlib
# Update .SRCINFO
$ mksrcinfo
# Review changes
$ git add -p .
# Commit
$ git commit -m "Update to version ..."
# Push to the AUR
$ git push origin master
```

If this is a newly created package, an initial commit should have been made.
If everything looks good, only the `git push` step is required.

#### Pull requests (PR) to arch-ros-stacks

Now that packages are treated as Git submodules, the pull request should only
contain new submodules (new entries in `.gitmodules`) and updated submodule
commit ids. First, you need to **make sure** that you pushed the new/updated
packages to the AUR. Second, to prepare the actual arch-ros-stacks commit for
the pull request, go to the root of arch-ros-stacks. Then, if for instance you
are dealing with indigo packages:

```shell
$ git add -p indigo
```
You should see differences involving `Subproject commit ...`. Just add the
packages that you updated if you are their maintainer in the AUR, and commit
with a clear message, e.g.:

```shell
# If you added new packages:
$ git commit -m "indigo: add ... and its dependencies"
# or if you updated the packages you maintain
$ git commit -m "indigo: update ... and its dependencies"
```

Finally, push to your GitHub fork and make your PR.

#### Setting push URLs

Git can only store one URL for each submodule in the project. Since most users
do not have SSH access to the AUR repositories, this URL is the read-only HTTPS
URL by default. If you want to set the push URLs, simply run the
`set_pushurl.sh` script, and any `git push` attemp will rely on the SSH URL.


#### Conversion to AUR4 packages (legacy)

The `to_aur4.sh` script can take care of uploading packages that are not
submodules to the AUR. Since it works by creating a subtree of a specific
package from `arch-ros-stacks`, applying `mksrcinfo` to create the required
`.SRCINFO` file, and pushing the result to the AUR, it will only work if all
pushes to the AUR of this specific package were done using it.

Recent versions of `import_catkin_packages.py` create/pull AUR submodules, so
this is no longer required.


### Python version

Starting with Indigo, Python 3 support is added to ROS. However, since most
non-core packages are not Python 3 ready yet, we will continue using Python 2
for Indigo and Jade.

[zootboy]: https://github.com/zootboy/arch-ros-stacks
[hauptmech]: https://github.com/hauptmech/arch-ros-stacks
[moesenle]: https://github.com/moesenle/arch-ros-stacks
[distribution.yaml]: https://github.com/ros/rosdistro/blob/master/indigo/distribution.yaml
[AUR]: https://aur.archlinux.org/
[AUR key]: https://wiki.archlinux.org/index.php/Arch_User_Repository#Submitting_packages
