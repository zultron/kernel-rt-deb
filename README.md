# Xenomai and RTAI kernel Debian packaging for Linux 3.8.13

This builds Xenomai- and RTAI-patched kernel packages for Debian and
Ubuntu.

## How to build:

These instructions assume you have the [Dovetail Automata package
repository][da-repo] configured.

    ### Install build deps
    apt-get install cpio kernel-wedge quilt patchutils openssl xmlto
    ### Install package configure deps
    apt-get install xenomai-kernel-source rtai-source

    ### Set up source package tree
    wget https://www.kernel.org/pub/linux/kernel/v3.x/linux-3.8.13.tar.xz \
	 -O linux_3.8.13.orig.tar.xz
    tar xf linux_3.8.13.orig.tar.xz
    cd linux-3.8.13
    git clone https://github.com/zultron/linux-ipipe-deb.git debian

    ### Configure source package
    # for Wheezy, use gcc-4.7; Trusty, 4.8; Jessie, 4.9
    sed -i config/defines -e '/^compiler:/ s/4.9/4.7/'
    # disable Xenomai kernel build, if desired
    sed -i config/defines -e '/^ xenomai.x86/ s/^/#/'
    # disable RTAI kernel build, if desired
    sed -i config/defines -e '/^ rtai.x86/ s/^/#/'
    # configure package
    debian/rules debian/control  # This will fail; that's normal
    debian/rules clean

    ### Build binary packages; four parallel make jobs
    dpkg-buildpackage -uc -us -b -j4

[da-repo]: http://www.machinekit.io/docs/packages-debian/


# OSC:

This package produces the `linux-libc-dev` package, but that is also a
dependency of the build.  OBS is peculiar in that it will not install
dependencies if they're produced by the build.  To override this, edit
the `prjconf` and add:

       Keep: linux-libc-dev


# Methodology:

Started from a clone of the Debian kernel packaging for Sid, r20131:

    git svn clone \
        svn://anonscm.debian.org/svn/kernel/dists/sid/linux/debian

Followed Debian Linux Kernel Handbook, [4.3 Building a development
version of the Debian kernel package][dlkh-4.3].

Summary of changes:
- Removed a lot of non-x86 architectures (pkg is x86-only for now)
- Removed the rt feature set
- Imported [configs from Ubuntu Raring][raring-configs]
  (Sid configs excluded virtio)
- Added featureset-friendly modifications to packaging
- Added the Xenomai feature set
- Added the r8168 driver
- Added the RTAI feature set

[dlkh-4.3]: http://kernel-handbook.alioth.debian.org/ch-common-tasks.html#s-common-official-vcs
[raring-configs]: http://kernel.ubuntu.com/~kernel-ppa/mainline/v3.8.13.23-raring
