---
layout: default
---

[Table of Contents](toc.md) > Your first package

# Your First Package

We will begin walking you through building a simple package.  This will introduce you to a basic software build from source, and then convert that into a package to be installed by the Pacman package manager.

## Hello!

All good programming experiences start with a simple "Hello, world!".  So will your packaging journey.

[GNU Hello](https://www.gnu.org/software/hello/) is an almost-trivial free software program that prints the phrase "Hello, world!" or a translation thereof to the screen. This is primarily to serve as an example of the GNU coding standards, and serve as template for more serious software projects. We shall use it as an introduction to the concept of package software.

## Building From Source

We will first walk through the steps of manually building the software.

### Obatining the Source

From the project website, we can follow the [link](https://ftp.gnu.org/gnu/hello/) through to download the source. We will select the latest version of the software to build, which at the time of writing is hello 2.10.

````shell
$ wget https://ftp.gnu.org/gnu/hello/hello-2.10.tar.gz
````

We have downloaded the source file (`hello-2.10.tar.gz`). Note that a cryptographic signature (`hello-2.10.tar.gz.sig`) is also available for download and could be use to verify the integrity of the source file. We will skip this for now, but verifying source files will be covered in more detail later.

Now we can move onto building the software.

### Building the Software

The first step is to extract the source tarball and change into the source directory:

````shell
$ tar -xf hello-2.10.tar.gz
$ cd hello-2.10
````

For this example, inside the source directory you will find a file named `INSTALL` with instructions of how to build and install the software. Other time you may need to follow instructions on a software's wiki page. Following these instructions we can build the software with the following commands:

````shell
$ ./configure
$ make
````

We can then test the build worked correctly with:

````shell
$ ./hello
Hello, world!
````
We could then install the software to our system.  It is not advisable to install software to the system without a package manager, so **do not run** this example command.  It would also need run as the `root` user to have access to the system directory.

````shell
$ make install
````

That is all it takes to build this piece of software. Most (but not all) will be slightly more complicated than this!

## Creating a PKGBUILD

Now we know how to build our software from source, it is time to create a pacman package from it.  To do this we create a file called `PKGBUILD`.  This file contains various fields annotating the software and the build commands.

### Boilerplate

There are four fields that every `PKGBUILD` file must minimally contain. These fields will be described in more detail elsewhere, but briefly we need:

- **pkgname**: the name of the software
- **pkgver**: the version of the software
- **pkgrel**: the release number of the package
- **arch**: the architecture the package can be built on

In the `PKGBUILD` file, this boilerplate for the GNU Hello package would look like:

````bash
pkgname=hello
pkgver=2.10
pkgrel=1
arch=('x86_64')
````

Note we have used the `x86_64` architecture as an example. You will need to adjust that to the architecture of your system as given in `makepkg.conf` to follow this example.

### Downloading the source

To download the source with a PKGBUILD we specify a `source` array:

````bash
source=("https://ftp.gnu.org/gnu/hello/hello-2.10.tar.gz")
````

Note that we can use any bash syntax here, including the variables as defined in our boilerplate.  So this can be written as:

````bash
source=("https://ftp.gnu.org/gnu/hello/hello-${pkgver}.tar.gz")
````

Note we could have also used `${pkgname}` in place of `hello` in the source line, but this has no advantage.  Using `${pkgver}` in the source allows us to change the version of the software by just altering the `pkgver` variable in the boilerplate.

Note that for this minimal first example, we are not following best practices for source verification. Instead we will add the `md5sum` of the source file to satisfy the build system.

````bash
md5sums=('6cd0ffea3884a4e79330338dcc2987d6')
````

### Adding build commands

Now we can add the build commands to the PKGBUILD.  This is separated into two functions, those run as the normal user, and those run as the "`root`" user.  We will start with the exact commands use to build the software above:

````bash
build() {
  cd hello-2.10
  ./configure
  make
}

package() {
  make install
}
````

There are a few adjustments that need to be made.  As in the `source` array, we can use the `pkgver` variable rather than a specific version.  Also, reading the `INSTALL` file, we see that running `./configure` on its own prepares files to be placed in `/usr/local`. When packaging software to be installed on your system, you will generally want that directly in `/usr`. We can add that information to the `configure` command:

````bash
build() {
  cd hello-${pkgver}
  ./configure --prefix=/usr
  make
}
````

Now to adjust the packaging step. The packaging system takes us back to the source root between each function, so we will need to repeat changing into the source directory. The current `make install` command, trys installing files directly into `/usr`. This misses the entire point of packaging software - we want all files in the system directories to be managed by the package manager. To avoid this, we install to a temporary directory, referred to by the variable `${pkgdir}`, and using the `DESTDIR` arguement to make. The packaging script takes all files installed to this directory and creates the final package from it.  So our final packaging function looks like:

````bash
package() {
  cd hello-${pkgver}
  make DESTDIR="${pkgdir}/" install
}
````

### The final PKGBUILD

If you have followed the instructions above, your first `PKGBUILD` file should contain the following:

````bash
pkgname=hello
pkgver=2.10
pkgrel=1
arch=('x86_64')
source=("https://ftp.gnu.org/gnu/hello/hello-${pkgver}.tar.gz")
md5sums=('6cd0ffea3884a4e79330338dcc2987d6')

build() {
  cd hello-${pkgver}
  ./configure --prefix=/usr
  make
}

package() {
  cd hello-${pkgver}
  make DESTDIR="${pkgdir}/" install
}
````

## Building the package

Now we have a complete `PKGBUILD` file, we can proceed to create the package.  In the same directory as the `PKGBUILD` file, run the command:

````shell
$ makepkg
````

If all is successful, `makepkg` will run though downloading the source, building the code and creating the package. The final package file will have a name like `hello-2.10-1-x86_64.pkg.tar.xz` (the extension may vary depending on your default compression setting).

Congratulations!  You have successfully created your first package. This can now be installed with `pacman` by running the following as the `root` user:

````shell
$ pacman -U hello-2.10-1-x86_64.pkg.tar.xz
````

You can run the `hello` command to verify it is installed and working.
