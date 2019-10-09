All Debian packages
-------------------

There will be a new subdirectory under the program's source directory called `debian`. The most important files are `control`, `changelog`, `copyright` and `rules` which are required for all packages

The original tool was `dh-make`. The more modern tool is `debmake`.

Within the package directory you run `debmake`. It will provide good template files. These template files must be manually adjusted to their perfection to comply with the strict quality requirements of the Debian archive. 

It will also generate the upstream tarball and its required symlink if they are missing

Please make yourself ready to read the pertinent part of the official Debian documentation together with this guide as needed to generate perfect Debian packages:

    “Debian Policy Manual”
        [“must follow” rules](https://www.debian.org/doc/devel-manuals#policy)

    “Debian Developer’s Reference”
        [“best practice” document](https://www.debian.org/doc/devel-manuals#devref)

Before you decide to ask your question in some public place, please do your parts of efforts, i.e., read the fine documentation:

    package information available through the aptitude, apt-cache, and dpkg commands.
    files in /usr/share/doc/package for all pertinent packages.
    contents of man command for all pertinent commands.
    contents of info command for all pertinent commands.
    contents of debian-mentors@lists.debian.org mailing list archive.
    contents of debian-devel@lists.debian.org mailing list archive.

- Do not forget to inspect existing well maintained packages


    The program should be useful.
    The program should not introduce security and maintenance concerns to the Debian system.
    The program should be well documented and its code needs to be understandable (i.e. not obfuscated).
    The program’s authors agree with the packaging and are amicable to Debian. [5]
    It, and its dependencies, must meet Debian Free Software Guidelines (DFSG).

    - We need to [file an ITP](https://www.debian.org/doc/manuals/developers-reference/pkgs.html#newpackage)

The `debsign` command, included in the devscripts package, is used to sign the Debian package with your private GPG key.

The `debuild` command, included in the devscripts package, builds the binary package and check them with the `lintian` command. It is useful to have verbose outputs from the `lintian` command.

Ensure that the install script (Makefile in their example) supports GNU Coding Standards and FHS.

debian/
-------
- The `debian/rules` file is the build script provided by the package maintainer
- The `debian/control` file provides the main meta data for the Debian package
- The `debian/copyright` file provides the copyright summary data of the Debian package
More specifics on the configuration files
https://www.debian.org/doc/manuals/debmake-doc/ch05.en.html

Python specific
---------------
READ THIS
https://wiki.debian.org/Python/LibraryStyleGuide

- Public python3 modules must be installed in the system Python 3 modules directory `/usr/lib/python3/dist-packages

- PEP 427 defines a built-package format called 'wheels' which is a Zip format archive containing Python code and a `star.dist-info` metadata directory, in a single file named with a `.whl` suffix.
- Packages must not build or provide wheels. They are redundant to the established way of providing Python libraries to Debian users, take no advantage of distro-based tools, and are less convenient to use. E.g. they must be explicitly added to the `sys.path`, cannot be easily grepped, and stack traces through Zip files are more difficult to debug.

- The `debian/control` source paragraph may contain optional fields to specify the versions of Python the package supports
`X-Python3-Version: >= X.Y`

- Any package that installs modules for the default Python version must declare a dependency on the default Python runtime package. If it requires other modules to work, the package must declare dependencies on the corresponding packaged modules. The package must not declare dependency on any version-specific Python runtime or module package.

For Python 3, the correct dependencies are `Depends: python3 (>= 3.Y)` and any corresponding `python3-foo` packages

`Build-Depends: python3-all-dev (>= 3.4)

- If a binary package provides any binary-independent modules (`foo.py` files), the correspnding byte-compiled modules (`foo.pyc` files) and optimised modules (`foo.pyo` files) must not ship in the package. Instead, they should be generated in the package's post-install script, and removed in the package's pre-remove script. The package's prerm has to make sure that both `foo.pyc` and `foo.pyo` are removed.

- A binary package should only byte-compile the files which belong to the package

- The file `/etc/python/debian_config` alls configuration how modules should be byte-compiled. The post-install scripts should respect these settings

- A program that specifies `python3` as its interpreter may require its own private Python modules. These modules should be installed in `/usr/share/module` or `/usr/lib/module` if the modules are architecture depdendent. 

'Default to making packages non-native. You should only use a native Debian package when it is clear that the package would not be useful outside the context of a Debian system, and would never be distributed except packaged for Debian or its derivatives.'

A non-native Debian source package contains: a source control file, the Debian packaging files and one or more source tarballs

1. The maintainer obtains the upstream tarball debhello-0.0.tar.gz and untars its contents to 'debhello-0.0' directory.

```
tar -xzmf example-1.1.tar.gz
cd example-1.1
```

2. The `debmake` command debianizes the upstream source tree by adding template files only in the `debian` directory
    - Note that it generates template files based on the command line options that you pass

```
debamke -b':py3'
```

3. The maintainer customises the template files

rules needs to contain:

```
export PYBUILD_NAME=visidata

%:
    dh $0 --with python3 --buildsystem=pybuild
```

control needs to contain:

```
Build-Depends: debhelper (>=9), dh-python, python3-all
X-Python3-Version: >= 3.4
Multi-Arch: foreign
Depends: ${misc:Depends}, ${python3:Depends}
```

Update the copyright information as it changes

Each time you update the package, you need to make changes to `changelog`.

```
dch -v version-revision
or
dch -i
```

4. The `debuild` command builds the binary package from the debianized source tree
    - `debhello-0.0-1.debian.tar.xz` is created containing the `debian` directory
    - It is a wrapper script of `dpkg-buildpackage` -> whose manpage you can check out

```
You see all the generated files.

    The debhello_0.0.orig.tar.gz is a symlink to the upstream tarball.
    The debhello_0.0-1.debian.tar.xz contains the maintainer generated contents.
    The debhello_0.0-1.dsc is the meta data file for the Debian source package.
    The debhello_0.0-1_amd64.deb is the Debian installable binary package.
    The debhello_0.0-1_amd64.changes is the meta data file for the Debian binary package.


3. `dh-python`
`dh-python` provides extensions for `debhelper` to make it easier to package Python modules and extensions. They calculate Python dependencies, add maintainer scripts to byte compile files, etc. Their use is recommended by the Debian python maintainers.

See `man dh_python3` for details

4. pybuild

Pybuild is a Debian python specific build system that invokes various build systems for requested Python versions in order to build modules and extensions. It supports automatically building for multiple Python versions.

------------------------

Maintaing a Debian repository
http://www.hackgnar.com/2016/01/creating-remote-apt-package.html (NOTE THAT THE INFORMATION IS OUTDATED)
https://wiki.debian.org/DebianRepository/SetupWithReprepro
http://blog.jonliv.es/blog/2011/04/26/creating-your-own-signed-apt-repository-and-debian-packages/
