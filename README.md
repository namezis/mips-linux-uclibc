# mips-linux-uclibc cross-toolchain on Gentoo (WIP)

This document aims to guide the reader to creating their own mips-linux-uclibc toolchain using tools provided by Gentoo linux.
The versions and configurations used here are not totally vanilla and thus need more hands-on involvement than
just running a long magic command.

Resulting toolchain will contain following versions of various packages (excluding gentoo revisions):

- `gcc-6.4.0` (different/more recent version may probably be used freely)
- `binutils-2.29.1` (different/more recent version may probably be used freely)
- `uclibc-0.9.33.2` (must mostly match TARGET uclibc version)
- `linux-headers-2.6.10` (must mostly match TARGET kernel version)

**Caveat:**
Originally the TARGET would have needed `uclibc-0.9.30` but as that does not work with recent `binutils` and
sufficiently old version (i.e. `binutils-0.20`) is not easily found, `0.9.33.2` is used here.
It may work with `0.9.30` libc on TARGET, but ymmv.

This documentation is licensed under [CC-BY-SA-4.0](LICENSE).
The `files/1-makefile.patch` is licensed under [GNU General Public License 2](LICENSE-patch).

## Requirements

- Gentoo linux with root shell. `sudo` works too if that is installed and preferred.
  All commands here unless otherwise noted are to be run as root.
- `sys-devel/crossdev`. Version `20171230` is used in this guide, others may work, ymmv.

## Assumptions

- `sys-devel/binutils` is at version `2.29.1-r1`, and `sys-libs/binutils-libs` has same ('ish) or more recent version.
- Portage contains `sys-libs/uclibc-0.9.33.2-r15`. The `0.9.33.9999` version can also be used.
- The host Gentoo has local portage overlay configured.
  If not, follow [Defining a custom repository](https://wiki.gentoo.org/wiki/Handbook:AMD64/Portage/CustomTree#Defining_a_custom_repository) in Gentoo handbook.


## Setting up the custom ebuilds
### linux-headers

The required `2.6.10` is not found on portage anymore, but `2.4.36` still is.
Here we use that as a base for the required `2.6.10` version.

```sh
# Prepare category and aux files directory.
cd /usr/local/portage
mkdir -p sys-kernel/linux-headers/files

# Copy the ebuild for base.
cd sys-kernel/linux-headers
cp -av /usr/portage/sys-kernel/linux-headers/linux-headers-2.4.36.ebuild ./linux-headers-2.6.10.ebuild
```

Next we need a patch file to fix the `linux-headers-2.6.10` source package.
The [1-makefile.patch](files/1-makefile.patch) (in raw form) must be copied in the previously created `files`-directory,
or more specifically, into `/usr/local/portage/sys-kernel/linux-headers/files` .
After that, the `2.6.10` ebuild must be modified of three variables so that the patch is actually applied,
and no other patches are tried:

```
PATCHES_V=""

SRC_URI="${KERNEL_URI}"

UNIPATCH_LIST="${FILESDIR}/1-makefile.patch"
```

To finish this step, build Manifest of the ebuild, patch, and source package:

```sh
ebuild linux-headers-2.6.10.ebuild digest
```

**Checkup:** Ensure that running the `find`-command below results in list below (order may vary).
This may be done as regular user.

```
$ find /usr/local/portage/sys-kernel
/usr/local/portage/sys-kernel
/usr/local/portage/sys-kernel/linux-headers
/usr/local/portage/sys-kernel/linux-headers/linux-headers-2.6.10.ebuild
/usr/local/portage/sys-kernel/linux-headers/files
/usr/local/portage/sys-kernel/linux-headers/files/1-makefile.patch
/usr/local/portage/sys-kernel/linux-headers/Manifest
```
