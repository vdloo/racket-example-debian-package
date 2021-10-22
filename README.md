# racket-example-debian-package

This repository is an example of how you could package software written in Racket for Debian.

# Description

This repository assumes building on Debian Buster for Debian Buster but could be easily adapted for any other release. [Raco](https://docs.racket-lang.org/raco/exe.html) is used to create a [stand-alone executable](https://docs.racket-lang.org/raco/exe-dist.html). This means that the package will install a binary executable but won't have the Racket system package as a requirement. The Racket system package is only used as a build dependency. Everything needed for distribution is bundled inside the deb (including Racket).

Note that the binary is stand-alone, but not statically compiled. This means that you wouldn't be able to run the binary inside the .deb on another Linux distribution for example because you'd get errors like `error while loading shared libraries: libffi.so.6: cannot open shared object file`.

## To create a new project with this as a template

1. Make sure you have the required packages installed:
```
apt-get install git-buildpackage debhelper
```

2. Clone and rename this repository

3. Find and replace all mentions of `racket-example-debian-package` with your project name:
```
git grep racket-example-debian-package
```

4. Edit and replace the maintainer name and other information in `debian/control` and `debian/changelog`.

5. Commit your changes

## Build the Debian package (.deb)

These instructions assume you're running Debian.

1. First ensure you have a cow environment
```
export ARCH=amd64
export DIST=buster
cowbuilder --create --distribution buster --architecture amd64 --basepath /var/cache/pbuilder/base-$DIST-amd64.cow --mirror http://deb.debian.org/debian --components="main"  --othermirror 'deb http://deb.debian.org/debian buster-backports main'
```

2. Build the .deb
```
cd racket-example-debian-package
gbp buildpackage --git-pbuilder --git-dist=$DIST --git-arch=$ARCH
```

On success a .deb should have been made in the parent directory.

```
ls ../ | grep -E ".deb$"
racket-example-debian-package_20211022.145506_amd64.deb
```

## Installing the package

You can install the package with `dpkg`
```
dpkg -i ../racket-example-debian-package_20211022.145506_amd64.deb
(Reading database ... 95715 files and directories currently installed.)
Preparing to unpack .../racket-example-debian-package_20211022.145506_amd64.deb ...
Unpacking racket-example-debian-package (20211022.145506) over (20211022.145506) ...
Setting up racket-example-debian-package (20211022.145506) ...
```

And then inspect the contents like:
```
dpkg -L racket-example-debian-package
/.
/usr
/usr/bin
/usr/lib
/usr/lib/racket-example-debian-package
/usr/lib/racket-example-debian-package/usr
/usr/lib/racket-example-debian-package/usr/bin
/usr/lib/racket-example-debian-package/usr/bin/racket-example-debian-package
/usr/lib/racket-example-debian-package/usr/lib
/usr/lib/racket-example-debian-package/usr/lib/plt
/usr/lib/racket-example-debian-package/usr/lib/plt/racket3m-7.2
/usr/share
/usr/share/doc
/usr/share/doc/racket-example-debian-package
/usr/share/doc/racket-example-debian-package/README.md
/usr/share/doc/racket-example-debian-package/changelog.gz
/usr/bin/racket-example-debian-package
# Also interesting: apt-cache show racket-example-debian-package
```

If you run your own APT repository you could include package into the repo like:
```
reprepro -C yourcomponent includedeb buster racket-example-debian-package_20211022.145506_amd64.deb
```

And then `apt-get update` to be able to install it like `apt-get install racket-example-debian-package`.


## Running the executable

Your executable should now be available in the default PATH.

```
# racket-example-debian-package 
Hello World
# which racket-example-debian-package 
/usr/bin/racket-example-debian-package
# ls -la /usr/bin/ | grep racket-example
lrwxrwxrwx  1 root   root          74 Oct 22 13:16 racket-example-debian-package -> ../lib/racket-example-debian-package/usr/bin/racket-example-debian-package
```

## Updating the package

When you make a change to the source code, commit the changes and then when you want to create a new release run:
```
export VERSION=$(date "+%Y%m%d.%H%M%S")
gbp dch --debian-tag="%(version)s" --new-version=$VERSION --release --commit
```

And don't forget to tag and push the new version:
```
git tag $VERSION
git push
git push --tags
```
