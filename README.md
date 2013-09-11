# FHS++

An approach that offers significant improvements over FHS without radically altering its basic structure and naming scheme.

We can divide the improvements into two layers: *User Centric* and *Vendor Centric* changes. We will look at each in turn.


## User Centricity

To make the system *user centric*, the pertinent root directories are divided between users (and user groups), in the same fashion as `home`. For example, if we had two users, `tom` and `mary`, then the `etc` directory would be subdivided as follows:

    /etc/
      all/
      tom/
      mary/

Unfortunately, in order to maintain backward compatibility, we cannot use `etc` in this manner because too many applications are preset to use `etc` and would thus not be able to utilize `/etc/all` in its place. So we have to use a different directory, in the case of `etc` will be caledl `ctrl`, and symlink `ln -s /ctrl/all /etc`.

Similarly, we must do the same with `var`, which is mapped to `data`.

So we have:

    /ctrl
    /data
    /home
    /tmp
    /etc -> /ctrl/all
    /var -> /data/all

And each of the main directories follow that same user-centric subdivisions, e.g.

    /ctrl/
      all/
      mary/
      tom/
    /data/
      ...
    /home/
      ...
    /tmp/
      ...

It is not necessary to have an alternate `tmp` directory to link to the `all` entry, as we have done with `etc` and `var` b/c in the case of tmp, it does not matter so much. Nonetheless we recommend using `/tmp/all` and `/tmp/$USER` instead of `tmp` in applications.

To make it clear how this effects the general structure of the system, consider how it would alter XDG Base Directory environment variables.

    $XDG_CONFIG_HOME="/ctrl/$USER/"
    $XDG_DATA_HOME="/data/$USER/"
    $XDG_CACHE_HOME="/tmp/$USER"

However, we recommend symlinking `~/.config` to `/ctrl/$USER/` instead to make it easier to, say, backup all a users files.


## Vendor Centricity

The second layer of FHS++ is the organization of software installation.
An additional directory called `org` is added to the root directory. Where `org` is the location of programs subdivided into vendors. (The name `org` comes from [objectroot](http://objectroot.org), we are open to suggestions for a better one, maybe `pkg`?) 

In this design, `usr` becomes the equivalent of GoboLinux's `Links` and/or `Index` directories. It stores symlinks to files in `org`, and unionfs is used with it to do safe installs. Read [A UnionFS-based Package System](http://www.linuxfromscratch.org/hints/downloads/files/pkg_unionfs.txt).

The `org` directory too might be sub-divided into users, with system-wide installs being placed under `/org/all`. However, it might be the exception *if* it is possible to allow users to securely install software into a shared location and still finely control access rights. (Is it?)


## Closing Statements

This new hierarchy sits fits well with the current FHS, the addition of `sys`, `dev` and `proc` etc., would not stand out in the least, and thus not require anything like the GoboHide kernel patch. This design might not bring with it the beauty of Gobolinux' design, but it does have most, if not all of, the technical merits.


