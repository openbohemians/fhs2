# FHS++

An approach that offers significant improvements over FHS without radically altering its basic structure and naming scheme.

We can divide the improvements into two layers: *User Centric* and *Vendor Centric* changes. We will look at each in turn.


## User Centricity

To make the system *user centric*, first `usr` is renamed `idx` (better name?). Then `usr` is repurposed for use as the per-user store, in place of `home`.

For example, if we had two users, `tom` and `mary`, then the `usr` directory would be subdivided as follows:

    /usr/
      all/
      tom/
      mar/

Each pertinent root directory is linked the `usr/all/{dir}`. So we have:

    /usr/
      all/
        etc
        tmp
        var
    /etc -> /usr/all/etc
    /tmp -> /usr/all/tmp
    /var -> /usr/all/var

The `home` directory can be left as is, or rerouted via `usr`.

    /home/
      tom -> /usr/tom/home
    /usr/
      tom/
        home

To make it clear how this effects the general structure of the system, consider how it would alter XDG Base Directory environment variables.

    $XDG_CONFIG_HOME="/usr/$USER/home"
    $XDG_CACHE_HOME="/usr/$USER/tmp"
    $XDG_DATA_HOME="/usr/$USER/data"
    
However, we recommend symlinking `~/.config` to `/usr/$USER/etc` and `~/.cache` to `/usr/$USER/tmp` to be on the safe side.

NOTE: If for backward compatability repurposing `usr` is not acceptable, then we just need to use a different name, e.g. `user`.


## Vendor Centricity

The second layer of FHS++ is the organization of software installation. This can proceed in one of two ways depending on feasability.

An additional directory called `org` is added either to the root directory or to each `usr` directory, where `usr/all/org`. Where `org` is the location of programs subdivided into vendors. (The name `org` comes from [objectroot](http://objectroot.org), we are open to suggestions for a better one, maybe `pkg`?) 

Whether the `org` directory is divyed up or a single entry in root depends on whether it is possible to allow users to securely install software into a shared location and still finely control access rights. (Is it?)
In either case a per-user `link` directory is available so that personal variation of packages can be installed.

In this design, `link` becomes the equivalent of GoboLinux's `Links` and `Index` directories. It stores symlinks to files in `org`, and unionfs is used with it to do safe installs. Read [A UnionFS-based Package System](http://www.linuxfromscratch.org/hints/downloads/files/pkg_unionfs.txt).

The procedure for installing software is outlined as follows:

1. Create an entry for the program under `org/{vendor}/{name}/{version}`. Optionally the version can have an appended build hash, e.g. `1.1.0--A45211BF`.
2. Create a unionfs of the program's directory overlaying `/usr/all/link` or `/usr/$USER/link` as appropriate.
3. Install package using distribution tool (apt-get), or manually (make) ensuring to install to `usr/all/link` or `usr/$USER/link`.
4. Unmount unionfs.
5. Symlink entries from program's `org` directory to link location.


## Closing Statements

This new hierarchy sits fits well with the current FHS, the addition of `sys`, `dev` and `proc` etc., would not stand out in the least, and thus not require anything like the GoboHide kernel patch. This design might not bring with it the beauty of Gobolinux' design, but it does have most, if not all of, the technical merits.


