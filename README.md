# FHS++

An approach that offers significant improvements over FHS without radically altering its basic structure and naming scheme.

We can divide the improvements into two layers: *User Centric* and *Vendor Centric* changes. We will look at each in turn.

## System Centricity

*WHAT ABOUT SIFFERENT HOSTS?*

Most of the top level directories in the FHS standard are system centric locations.

    boot
    dev
    mnt
    proc
    run
    var
    sys
    
A few others are also system centric, but have a more direct interface with users. With the exception of `root` they are all related to the software installed on a system.

    bin
    etc
    lib
    root
    sbin
    usr
    
Because FHS++ is user and vendor centric, it moves all of the first set into a single top level directory.

    system/
      boot/
      dev/
      mnt/
      proc/
      run/
      var/
      sys/

To retain compatability with legacy systems, until such time as it is uncessary, each legacy directory is linked the corresponding system directory. eg,

    /etc -> /system/etc
    /tmp -> /system/tmp
    /var -> /system/var
   
    
## User Centricity

In the current FHS, user directories fall under `home`. To make the standard *user centric*, we move those directories to the top level.

So, for example, if we had two users, `tom` and `mary`, then the root directory would be subdivided as follows:

    / 
      admin/
      all/
      tom/
      mary/
      system/

At first glance this will probably seem a bit *crazy*, as it would appear to make it impossible to mount user data to a separate partition -- the most common use of partiion separation. But in actuality it is not a problem. The technique for doing the same is simply reversed. Instead of mounting the root directory ('`/`') to a systems partition and `home/` to a secondary data partition, the the root directory is mapped to the data partition and `system/` is mapped to the systems partition. With advances is OS design this is quite doable, and has actually been so for a while.

The `all` directory is a *user group* that is shared by all users. 

The `home` directory can be left as is, or rerouted via `usr` as well.

    /home/
      tom -> /tom/home
    /tom/
      home

Of course it is not necessary to make these links from `home` since we can just adjust the `passwd` directory and $HOME environment variable. Then `home` can be removed.

To make it clear how this effects the general structure of the system, consider how it would alter XDG Base Directory environment variables.

    $XDG_CONFIG_HOME="/$USER/home"
    $XDG_CACHE_HOME="/$USER/tmp"
    $XDG_DATA_HOME="/$USER/data"  (or var?)
    
However, we recommend symlinking `~/.config` to `/$USER/etc` and `~/.cache` to `/$USER/tmp` to be on the safe side.


## Vendor Centricity

The second layer of FHS++ is the organization of software installation. 

An additional directory called `app` is added to each users directory, where `all/app` is the location for softwares shared by all users. The `app` directoires are the location of programs subdivided into vendors. (The idea for `app` comes from [objectroot](http://objectroot.org)'s `org`).

Then a per-user `link` directory is available so that personal variation of packages can be installed.
In this design, `link` becomes the equivalent of GoboLinux's `Links` and `Index` directories. It stores symlinks to files in `app`. Potentially unionfs can be used with it to do safe installs. Read [A UnionFS-based Package System](http://www.linuxfromscratch.org/hints/downloads/files/pkg_unionfs.txt).

The procedure for installing software is outlined as follows:

1. Create an entry for the program under `app/{vendor}/{name}/{version}`. If the app already exists but has variant build  configuration then the version can be appended with a build hash, e.g. `1.1.0--A45211BF`.
2. Create a unionfs of the program's directory overlaying `/usr/$USER/link` or `/usr/all/link` as appropriate.
3. Configure a chroot jail out of the `link` driectory and then chroot to it.
4. Install package via distribution tool (e.g. apt-get), or manually (e.g. make).
5. Close chroot and unmount unionfs.
6. Symlink entries from the application to the `link` directory.

These steps would be handled automatically by a wrapper script around a target package tool. (Can we can use PackageKit to handle this is a universal fashion?)


## Closing Statements

This new hierarchy fits well with the current FHS, the addition of `sys`, `dev` and `proc` etc., would not stand out in the least, and thus not require anything like the GoboHide kernel patch. This design might not bring with it the beauty of Gobolinux' design, but it does have most, if not all of, the technical merits.


