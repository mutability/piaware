# PiAware

This is a repackaging of the official FlightAware version of piaware.
For the original docs, see [README-upstream.md](README-upstream.md).

This version is based on FlightAware's 1.19-3 release.

# Installation

Warning: This package conflicts with the standard piaware package. You must
uninstall the standard piaware package before installing piaware-mutability.

The recommended way to install piaware-mutability is from the mutability.co.uk
repository.

First, manually install the `mutability-repo` package if it is not already
installed:

````
$ wget https://github.com/mutability/mutability-repo/releases/download/v0.1.0/mutability-repo_0.1.0_armhf.deb
$ sudo dpkg -i mutability-repo_0.1.0_armhf.deb
````

Then use apt-get to install piaware-mutability:

````
$ sudo apt-get update && sudo apt-get install piaware-mutability
````

You probably also want a package that provide a FATSV-format feed. Currently
these packages provide one:

 * faup1090: a FlightAware utility that converts Beast-format data from an
   existing source into the FATSV format;
 * dump1090-flightaware: the FlightAware version of dump1090 that directly
   supports FATSV;
 * dump1090-mutability: also directly supports FATSV.

Other software such as modesmixer2 will also work (but isn't packaged).

piaware-mutability will be upgraded to new versions automatically when you
upgrade other packages:

````
$ sudo apt-get update && sudo apt-get upgrade
````

# Changes from upstream

The majority of the changes are policy changes, making different
decisions about how piaware should run, log, and affect other parts
of the system.

## Single-purpose selfcontained packaging

This package is a selfcontained build that can be built
with dpkg-buildpackage without a separate external build step.

This package only contains piaware. faup1090 and tcllauncher are not included -
they are now part of their own packages: faup1090 is built from
[dump1090-flightaware](https://github.com/mutability/dump1090/tree/fa-pi-package)
and
[tcllauncher has its own package](https://github.com/mutability/tcllauncher).

## Configuration

The configuration file for piaware now lives at `/etc/piaware.adept.conf`,
not in `/root`. Settings related to automatic startup of piaware and logging
are in `/etc/default/piaware-mutability`.

Configuration via debconf is also supported
(`dpkg-reconfigure piaware-mutability`)

## Security

piaware no longer insists on running as root (in fact, it will refuse to run as
root).

As a result of this, piaware cannot directly do system upgrades without explicit
sudo configuration. See below.

## Changes to how piaware invokes external commands

Upstream piaware would just run commands as root as desired to affect the local
system.

This package runs scripts contained in `/etc/piaware.hooks/` to run particular
tasks. These scripts will run as the piaware user, not root.

In cases where additional privileges are needed, the hook scripts try to invoke
the needed commands via `sudo`.

The hook scripts are intended to be customized as needed if you want them to
behave differently, as appropriate for your system.

There is a sample `sudoers` file provided that enables the commands that the
default hook scripts want to run. This file is entirely commented out; you must
modify it if you want piaware to have the ability to trigger upgrades etc.

## Limiting information that piaware sends to FlightAware

Upstream piaware sends system health information and logging back to the
FlightAware servers. This can be useful to FlightAware for monitoring dedicated
installations that aren't running anything else and probably won't see any
monitoring otherwise, but it is not so appropriate for a shared machine where
other things are running and someone is looking after it locally.

This package adds configuration options that allow you to disable the sending of
this information.

# Building the package

This package is intended to be built using the normal Debian package building
tools:

````
sudo apt-get install build-essential debhelper tcl8.5
dpkg-buildpackage -b      # or debuild, or pdebuild, etc
````
