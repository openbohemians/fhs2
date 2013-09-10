# FHS++

An approach that offers significant improvements over FHS without radically altering its basic structure and naming scheme.

We can divide the improvements into two layers: *User Centric* and *Vendor Centric* changes. We will look at each in turn.


## User Centricity

Each pertinent main directory is divided between users (and user groups), in the same fahsion as `home`. For example, if we had two users, `tom` and `mary`, then the `etc` directory would be subdivided as follows:

    /etc/
      main/
      tom/
      mary/

Where `main` holds the system-wide shared configuration files.

In order to maintain backward compatibility however, we cannot use `etc` in this manner because too many applications are preset to use `etc` and would thus not be able to utilize `/etc/all` in its place. So we have to use a different directory, which we will call `ctrl`, and symlink `ln -s /ctrl/all /etc`. Similarly, the same pattern is followed for `var`, which is mapped to `/cache/all`.

In addition is added a `data` directory to store the equivalent of XDG's `.local/share`. Accordingly `/usr/share` links to `/data/all` (but more on `usr` below). So we have:

    /cache
    /ctrl
    /data
    /home
    /tmp
    /etc -> /ctrl/all
    /var -> /cache/all

And each of the main directoies follow that same user-centric subdivisions, e.g.

    /cache/
      all/
      mary/
      tom/
    /ctrl/
      ... ditto ...
    /data/
      ... ditto ...
    /home/
      ... ditto ...

We do not worry about have an alternate `tmp` directory to link to the `all` entry, as we have done with `etc` and `var` b/c in the case of tmp, it does not matter so much. Nonetheless it is advisable to use `/tmp/all` instead of `tmp` in applications and such.

To make it clear how this effects the general structure of the system, consider how it would alter XDG Base Directory environment variables.

    $XDG_CONFIG_HOME="/ctrl/{user}/"

    ...TODO...

However, administrators may choose to symlink `~/.config` to `/ctrl/~/` instead to make it easier to, say, backup all a users files.


## Vendor Centricity

The second layer of FHS++ is the organization of software installation.
An additional directory called `org` is added to 

Where `org` is the location of programs subdivided into vendors. (The name `org` comes from [objectroot](http://objectroot.org), we are open to suggestions for a better one). 

In this design, `usr` becomes the equivalent of Gobolinux's `Index` directory. It is a unionfs of the directories in `org`. Read [A UnionFS-based Package System](http://www.linuxfromscratch.org/hints/downloads/files/pkg_unionfs.txt).

The `org` directory too might be subdivied into users, with system-wide installs being placed under `/org/all`. However, it might be the exception *if* it is possible to allow users to securely install software into a shared location and still finely control access rights. (Is it?)


## Closing Statements

This hierarchy sits well with the current FHS, the addition of `sys`, `dev` and `proc` as well as `mnt` would not stand out in the least, and thus not require the GoboHide kernel patch.

This design might not bring with it the beauty of Gobolinux' design, but it does have all the technical merits.


