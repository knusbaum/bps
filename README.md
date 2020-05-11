# bps - package manager for 9bps

bps contains simple scripts to install and remove packages build with [9bps](https://github.com/knusbaum/9bps)

This is still an experiment, and bps does virtually nothing to ensure packages are installed/uninstalled properly.

## Installation

### By script
To try out bps, you can run the install script that is in the root of this repository on a 9front system.

If you find this scary, please have a look at the script. It is a short script does a small set of tasks you can do manually if you like.

1. mount `tcp!pkg.9project.net!565` on `/n/bps`. This serves the experimental package repository at 9project.net
2. create `/sys/lib/bps`, which is where installed packages and their metadata will be stored
3. unpack `/n/bps/noarch/bps-$version_$rev.pkg` into `/sys/lib/bps/bps`.
4. install `bps` from `/sys/lib/bps/bps/root.tar` into the current namespace. This is done by untarring `root.tar` into `/`

### Manually

Feel free to install manually. All that is required is for the bps directory in this repo to be mounted under /bin/ and for
/sys/lib/bps to exist and be writable.


## Use

Currently bps does extremely basic tasks like installing and uninstalling packages.

#### `bps/find [pkg name]`
looks for a package by name in `/n/bps` suitable for installation on `$objtype` systems and returns
the path to the `.pkg` file if one is found. It looks first in `/n/bps/$objtype` and then in `/n/bps/noarch`. To browse
packages, mount `tcp!pkg.9project.net!565` and look through the directories. There is not yet a tool to assist with that.
The root of that filesystem contains a directory per architecture. The `noarch` directory contains architecture-independent
packages.

If `/n/bps` is empty, `bps/find` will mount `tcp!pkg.9project.net!565` before searching. Other repositories created with
[9bps](https://github.com/knusbaum/9bps) can be bound/mounted to `/n/bps` instead.


#### `bps/install [pkg name]`
will `bps/find` a package, unpack the metadata and `root.tar` into `/sys/lib/bps/`, and then unpack the
`root.tar` into the current namespace by untarring `root.tar` into `/`. It should be possible to install packages into
custom locations by mounting or binding directories over the install directories.


#### `bps/remove [pkg name]`
will find a package in `/sys/lib/bps/` and deletes everything listed in its `manifest` file. It then
deletes all package metadata and `root.tar` from `/sys/lib/bps`. One issue with `bps/remove` is that it makes no attempt to
delete directories that were created by the package installation.


## TODO

There is a lot to do.

* Better validation of installs
* Some sort of mechanism for rolling-back during failures on install or removal
* Improve remove mechanism 
* Add search
* Add `bps/activate` which will serve a new namespace that has a package installed, without copying files anywhere.
  It should serve a union of a package's `root.tar` over `/`. This is trickier than it sounds, and requires unionfs as a dependency
* Add `bps/install_user` (probably need a better name) to install a package for a given user. This is problematic for non-program files.
  We can install binaries into `/usr/$user/bin/$arch`, for example, but there isn't a clear place to install data destined for other
  places such as `/sys/lib`.
