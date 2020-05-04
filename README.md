# pkg - package manager for 9

pkg contains simple scripts to install and remove packages build with [9bps](https://github.com/knusbaum/9bps)

This is still an experiment, and pkg does virtually nothing to ensure packages are installed/uninstalled properly.

## Installation

### By script
To try out pkg, run the install script that is in the root of this repository on a 9front system.

If you find this scary, please have a look at the script. It does a small set of tasks:

1. mount `tcp!pkg.9project.net!565` on `/n/pkg`. This serves the experimental package repository at 9project.net
2. create `/sys/lib/pkg`, which is where installed packages and their metadata will be stored
3. unpack `/n/pkg/noarch/pkg-$version_$rev.pkg` into `/sys/lib/pkg/pkg`.
4. install `pkg` from `/sys/lib/pkg/pkg/root.tar` into the current namespace. This is done by untarring `root.tar` into `/`

### Manually

Feel free to install manually. All that is required is for the pkg directory in this repo to be mounted under /bin/ and for
/sys/lib/pkg to exist and be writable.


## Use

Currently pkg does extremely basic tasks like installing and uninstalling packages.

`pkg/find [pkg name]` looks for a package by name in `/n/pkg` suitable for installation on `$objtype` systems and returns
the path to the `.pkg` file if one is found. It looks first in `/n/pkg/$objtype` and then in `/n/pkg/noarch`. To browse
packages, mount `tcp!pkg.9project.net!565` and look through the directories. There is not yet a tool to assist with that.
The root of that filesystem contains a directory per architecture. The `noarch` directory contains architecture-independent
packages.

If `/n/pkg` is empty, `pkg/find` will mount `tcp!pkg.9project.net!565` before searching. Other repositories created with
[9bps](https://github.com/knusbaum/9bps) can be bound/mounted to `/n/pkg` instead.


`pkg/install [pkg name]` will `pkg/find` a package, unpack the metadata and `root.tar` into `/sys/lib/pkg/`, and then unpack the
`root.tar` into the current namespace by untarring `root.tar` into `/`. It should be possible to install packages into
custom locations by mounting or binding directories over the install directories.


`pkg/remove [pkg name]` will find a package in `/sys/lib/pkg/` and deletes everything listed in its `manifest` file. It then
deletes all package metadata and `root.tar` from `/sys/lib/pkg`. One issue with `pkg/remove` is that it makes no attempt to
delete directories that were created by the package installation.


## TODO

There is a lot to do.

* Better validation of installs
* Some sort of mechanism for rolling-back during failures on install or removal
* Improve remove mechanism 
* Add search
* Add `pkg/activate` which will serve a new namespace that has a package installed, without copying files anywhere.
  It should serve a union of a package's `root.tar` over `/`. This is trickier than it sounds, and requires unionfs as a dependency
* Add `pkg/install_user` (probably need a better name) to install a package for a given user. This is problematic for non-program files.
  We can install binaries into `/usr/$user/bin/$arch`, for example, but there isn't a clear place to install data destined for other
  places such as `/sys/lib`.
