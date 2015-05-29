# Xenomai and RTAI kernel Debian packaging for Linux 3.16.7

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
    wget https://www.kernel.org/pub/linux/kernel/v3.x/linux-3.16.7.tar.xz \
	 -O linux_3.16.7.orig.tar.xz
    tar xf linux_3.16.7.orig.tar.xz
    cd linux-3.16.7
    git clone https://github.com/zultron/linux-ipipe-deb.git debian

    ### Configure source package
    # for Wheezy, use gcc-4.7; Trusty, 4.8; Jessie, 4.8
    sed -i config/defines -e '/^compiler:/ s/4.9/4.7/'
    # disable Xenomai kernel build, if desired; set 'enabled: false'
    sed -i config/defines \
        -e '/featureset-xenomai.x86_base/,/^[[]/ s/enabled:.*/enabled: false/'
    # disable RTAI kernel build, if desired
    sed -i config/defines \
        -e '/featureset-rtai.x86_base/,/^[[]/ s/enabled:.*/enabled: false/'
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

Started from a clone of the Debian kernel packaging for Sid, r22524:

    git svn clone \
        svn://anonscm.debian.org/svn/kernel/dists/sid/linux/debian

Followed Debian Linux Kernel Handbook, [4.3 Building a development
version of the Debian kernel package][dlkh-4.3].

Summary of changes:
- Added featureset-friendly modifications to packaging
- Added the Xenomai feature set
- Added the RTAI feature set
- Disabled `none` and `rt` featuresets
- Backed out patches that interfered with i-pipe patch

[dlkh-4.3]: http://kernel-handbook.alioth.debian.org/ch-common-tasks.html#s-common-official-vcs

# Updating:

Locate the last commit from upstream SVN:

    $ git log -1 --pretty=oneline --grep=git-svn-id:
	cb28f5791d75dc9dab9f74a3aeb17c66d7721beb [...]

Check it out into a new branch:

    $ git checkout -b svn-sid cb28f579

Add a line like this to `.git/config`:

	[svn-remote "svn-sid"]
		url = svn://anonscm.debian.org/svn/kernel/dists/sid/linux/debian
		fetch = :refs/remotes/git-svn/sid

And put that git rev into the file `.git/refs/remotes/git-svn/sid`.

The remote should now be visible from `git branch -a`.

Now rebuild the index:

    $ git svn fetch svn-sid
	Migrating from a git-svn v1 layout...
	Data from a previous version of git-svn exists, but
		.git/svn
		(required for this version (1.7.10.4) of git-svn) does not exist.
	Done migrating from a git-svn v1 layout
	Rebuilding .git/svn/refs/remotes/git-svn/sid/.rev_map.510b9475-[...]
	[...]
	r22524 = cb28f5791d75dc9dab9f74a3aeb17c66d7721beb
	Done rebuilding .git/svn/refs/remotes/git-svn/sid/.rev_map.510b9475-[...]

Be sure the output includes messages like the last.  After this, `git
svn` will spend a long time searching for relevant changes on the
upstream SVN server.

When it finishes, check out the desired revision from this remote, and
rebase the changes in this repo on top.
